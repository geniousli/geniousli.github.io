---
title: Crafting rails
date: 2017-07-28
tags: rails, ruby
---

Crafting rails application
--------

### 创建自己的render

  * rails plugins new pdf_render
  * gemspec, 之盾依赖， 作者、version， lib/pef_render.rb会被自动require（详细看bundler.io中的解释）
  * Gemfile, 直接引入gemspec，生命依赖关系

  rails render 解析

  ```ruby
  # rails/actionpac k/lib/action_controller/metal/rendcers.rb

  add :json do |json, options|
    json = json.to_json(options) unless json.kind_of?(String)

    if options[:callback].present?
      self.content_type ||= Mime::JS
      "#{options[:callback]}(#{json})"
    else
      self.content_type |||= Mime::JSON
      self
    end
  end

  render json: @post
  json  => @post# json 指的是， block中的json的变量
  # 我们想提供的为
  render pdf: 'contents', template: 'path/to/template'

  require 'prawn'  # prawn 提供 pdf的生成
  pdf = Prawn::Document.new
  pdf.text('a string to pdf')
  pdf.render_file('sample.pdf')


  # lib/pdf_render.rb
  require 'prawn'
  ActionController::Renderers.add :pdf do |filename, options|
    pdf = Prawn::Document.new
    pdf.text = render_to_string(options)
    send_data(pdf.render, filename: "#{filename}.pdf", diposition: "attachment")
  end

  # rails 如何设定正确的respon中的content type？
  Mime::Type.register "application/pdf", :pdf, [], %w(pdf)
  ```

  rails render stack

  ```text
   AbstractController::Rendering.render
   |
   |-- _normalize_render
   |      |-- _normalize_args
   |      |-- _normalize_opions
   |-- ActionView::Rending.render_to_body
          |-- _proccess_options
          |-- _render_template
                  |-- context = view_context_class.new(view_renderer, view_assigns, self)
                  |-- ActionView::Renderer.new(lookup_context).render(context, option)
                      |-- Renderer.render_template(context, options)
                          |-- TemplateRenderer.new(@lookup_context).render(context, options)

  其中, 大部分的都是在ActionController::Base 中include进去的，所以。所有方法都是在controller中执行的
    def view_assigns
      protected_vars = _protected_ivars
      variables      = instance_variables

      variables.reject! { || sprotected_vars.include?  }
      svariables.each_with_object({}) { |name, hash|
        hash[name.slice(1, name.length)] = instance_variable_get(name)
      }
    end
    获取controller中所有的实例变量, 传递到 context中
  ```

### 通过Active Model建立自己的模型

  1. form_helper

  ```
  form_for(record, options)
  |-- builder = instantiate_builder(object_name, record, options)
      |-- builder = options[:builder] || default_form_builder_class # ActionView::Helpers::FormBuilder
      |-- builder.new(object_name, object, self, options) # self is ActionView::Base instance
  |-- output = capture(builder, &block) # form_for中的内部dom
      |-- yield(builder)
        |-- Tags::TextField.new("data_bank", :title, self, {object: object}).render
          |-- options["value"] = options.fetch("value") { value_before_type_cast(object) }
            |-- value_before_type_cast # 从object获取method_name的数值
  |-- form_tag_with_body(html_options, output) # 构建真正的form dom
  ```

  **所以rails中的文档有很明确的拓展方法:**

  ```ruby
  class MyFormBuilder < ActionView::Helpers::FormBuilder
    def div_radio_button(method, tag_value, options = {})
      @template.content_tag(:div,
        @template.radio_button(
          @object_name, method, tag_value, objectify_options(options)
        )
      )
    end
  end

  # The above code creates a new method +div_radio_button+ which wraps a div
  # around the new radio button. Note that when options are passed in, you
  # must call +objectify_options+ in order for the model object to get
  # correctly passed to the method. If +objectify_options+ is not called,
  # then the newly created helper will not be linked back to the model.
  #   <%= form_for @person, :builder => MyFormBuilder do |f| %>
  #     I am a child: <%= f.div_radio_button(:admin, "child") %>
  #     I am an adult: <%= f.div_radio_button(:admin, "adult") %>
  #   <% end -%>

  ```

  使用继承类，定义自定义的方法， 其他的自然使用继承的方法调用

  > 1. 如何扩展 FormBuilder， 2： FormBuilder 在编辑时候的，默认值，是如何取到的
  
  
  2. 完善的ActiveModel
     1. ActiveModel::AttributeMethods
     ```ruby
     moduel MailForm
        class Base
            include ActiveModel::AttributeMethods
            attribute_method prefix: "clear_"
            
            def self.attributes(* names)
                attr_accessor(*names)
                define_attribute_methods(names)
            end
            
            private
            def clear_attribute(name)
                send("#{name}=", nil)
            end
        end
     end
     ```
     
     * 当rails controller, view helper接到一个 model,首先调用to_model，操纵结果，而不是直接使用model,　同样我们也不会自己实现，而是通过include ActiveModel::Conversion来实现, 其中主要实现为：　
     
       1. to_params, 用来生成唯一路由，　
       2. to_key, 返回唯一表示model的数组，供诸如dom_id, dom_class, div_for
       3. to_partial_path，　在vieｗ中使用render(model)时候调用，来计算渲染使用的模板，
   　　> 重要的不仅仅是　这些方法的作用，还有这些方法帮助我们所实现的，取得的成就。　比如实现to_params 可以轻松的改变object 的url,
    def to_params
        "#{id}-#{title.parameterize}"
    end
    假设每个POST有一个不同的方式，可能是video, link, or text, 需要不同的渲染方式，　
    @posts.each do |post| 
       render partial: "post/post_#{post.format}", locals: {post: post}
    end
    可以这样：　
    def to_partial_path
       "post/post_#{format}"
    end
    render @posts
    
    * 最终的结果是这个样子：
    ```ruby
    module ActiveModel
        class Base
            def self.included(base)
                base.class_eval do 
                    extend ActiveModel::Naming
                    extend ActiveModel::Translation
                    include ActiveModel::Validations
                    include ActiveModel::Conversions
                    
                end
            end
            
            def persisted?
                true
            end
        end
    end
    ```
    > 可以实现一个自己的model来自定义，自己重写sequel中定义的model，而不需要改动interface来定义, cms中可以通过两种方式结合来，重新重构：１：自定义form_for，　２：自定义model
    

### 自定义视图模板
    rails在渲染模板，必须通过某种方式定位模板位置，默认rails的模板在文件系统中，但是却不是必须的，rails提供的钩子允许我们从任何地方来提供模板，让我们从数据库中提供模板，让我们深入了解rails的render stack。
    
    *　rails 渲染：　主要职责为：normalize　选项然后传递给ActionView::Renderer 的实例变量，renderer接受一个ActionView::Base的实例变量和hash，来寻找，编译，渲染模板。无论合适，我们渲染模板，源代码必须给编译成可执行的ruby代码，每当代码被执行，发生在给定的环境变量中，　view context，所有的helper方法都在，　包括form_for, link_to，　在view context中，　view renderer还有ActionView::LookupContext的实例变量，这个变量是在controller和view中共享的，它包含所有的为了找到模板的信息，　比如，当渲染jsoｎ请求来是，request的format　就被存储在lookup context变量中，所以rails只寻找json格式的模板。变量在view中也是可用的，包含模板名称，locale, format, 
    
request---> controller---->(render) view_renderer ------->(find) looup_context ------>(find) view_path ----------------------------|
                                                                                                                                   |
                                                                                                                                   |
response <- controller <-(rendered template)<-----view_renderer<--------(template)looup_context<-------(template)view_path<--------|


    * resolver API, 
      > resolver API 只有find_all一个接口，返回包含template,等的数组。
      
> 1. 如何扩展 FormBuilder， 2： FormBuilder 在编辑时候的，默认值，是如何取到的


### Server 异步消息到 client
  1. 当样式表改动时候，rails发送data到浏览器，浏览器根据data 重新加载当前页面的样式表， 从而达到不需要重新刷新页面累加载样式的目的.
  2. 使用websocket(但是不知道为啥不能保持很长时间)， puma多线程, 自定义subscribe 使用queue作为数据结构， 体统轮训， 来分发到个个subscribe， 使用listener，监听个个文件的通知提供事件。
  3. 涉及到线程概念
  4. 代码加载， autoload是rails提供，而非ruby， ruby中的require是存在缺陷的，不是原子性的require， 在多线程加载中，存在问题， 可能存在A加载中class， B看到了class但是却是残缺不全的， 所以提供了eager load技术，加载所有的代码，而不需要动态加载代码。可以通过config.eager_load_namespaces 来配置，或者，使用代码 eager_autoload {autoload: SSESubscriber}， eager load受益的不仅仅是 puma这样的多线程，还有基于fork的unicorn，
  5. listener， linux实现机制

### Responders
  **使用ActionController::Responder 最大的好处是 集中处理 每种请求的format**

  1. 影响条件， type, http verb, resource status

  rails 中responder 的实现
  ```ruby
  # rails/actionpack/lib/action_controller/metal/responder.rb
  def self.call(*args)
    new(*args).respond
  end

  def respond
    method = "to_#{format}"
    respond_to?(method) ? send(method) : to_format
  end

  def to_html
    default_render
  rescue ActionView::MissingTemplate => e
    navigation_behavior(e)
  end

  def to_js
    default_render
  end

  def to_format
    if get? || !has_errors? || response_overridden?
      default_render
    else
      display_errors
    end
  rescue ActionView::MissingTemplate => e
    api_behavior(e)
  end

  DEFAULT_ACTIONS_FOR_VERBS = {
    post: :new,
    patch: :edit,
    put: :edit
  }

  def navigation_behavior(error)
    if get?
      raise_error
    elsif has_errors? && default_action
      rende4r :action => default_action
    else
      redirect_to navigation_location
    end
  end

  def api_behavior(error)
    raise error unless resourceful?
    if get?
      display resource
    elsif post?
      display resource, :status => :created, :location => api_location
    else
      head :no_content
    end
  end

  def resourceful?
    resource.respond_to?("to_#{format}")
  end

  def has_errors?
    resource.respond_to?(:errors) && !resource.errors.empty?
  end

  def resource_location
    options[:location] || resources
  end

  def display(resource, given_options = {})
    controller.render given_options.merge!(options).merge!(format => resource)
  end

  # 配置方式
  ApplicationController.responder = MyAppResponder
  class UsersController < ApplicationController
    self.responder = MyAppResponder
  end
  ```

  **如果想特殊化一个format的展现， 可以像respond_to 一样使用block**

  ```ruby
  def index
    @users = User.all
    respond_with(@users) do |format|
      format.json {render json: @users.to_json(some_specific_option: true)}
    end
  end
  ```

  ***最后章节， 关于generator的自定义，类似于rails guides，可以作为参考***

### Notification API
    1. instrument(), subscribe(),
      ```ruby
        ActiveSupport::Notification.instrument(event_name,
        payload: {format: :html, name: 'xxx'}) do
          process_action("index")
        end
        ActiveSupport::Notification.subscribe(event_name) do |*args|
          args
        end

        args: {
          name: 事件名字,
          started_at: 事件开始时间,
          ended_at: 事件完结时间,
          instrument_id: 事件唯一id,
          payload: 事件携带的信息,
        }
      ```
    2. Rails and rack
      > 任何一个响应call方法的ruby对象都是Rack应用，接受一个参数， environment， 然后返回 status, headers, body

      ```ruby
      class HelloRack
        def call(env)
          [200, {'Content-Type' => 'text/html'}, ['Hello Rack!']]
        end
      end
      run HelloRack.new
      ```
    3. Rails Router
      Rails自动将 Controller#action 转换成Rack application， 可以这样， PostsController.action(:index).responds_to?(:call)
    4. middleware stacks
      * 除了在 config 中配置middleware外，还可以在Conttoller中配置使用， class Userscontroller use MyMiddleware end;
      * Request ---> Web server -> middleware -> Rails Appplication -> middleware -> Router -> Controller -> middleware -> Action

### I18n （没看）


### 总结
  * 创建自己的render： 创建自己的pdf handler
  * 自定义自己的ActionModel： 讲的有点鸡肋， 没有进行深入的剖析, 譬如Active::Model中的callback实现， 自定义model的原因， 目的。FormBuilder的自定义也没有涉及到
  * websocket： 似乎并不是一个完整的实现， 还存在缺陷
  * Responders: 并不是解耦的很好方法，实际情况中，似乎并不需要
  * Notification: 主要讲的并不是Notification的实现机制，实现方法，主要讲的是Rails Engin， middleware

  > 书中讲解的并不算太多，大部分都是在Rails Engin 中进行， 进行了一个markdown的view handler 值得注意看，之外， 其他的都没有进行深入的讲解，譬如， 自定义ActiveModel 的作用，目的，实现方法， FormBuilder又是如何, Reponder 感觉并没有太多的进行简化，

