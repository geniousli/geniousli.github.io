---
title: Rails Application Refactor
date: 2017-08-02
tags: rails, ruby, refactor
---

重构现有Rails应用
--------

### 面临问题

  * 数据库层面单独建立项目，导致产生更新流程， 更改interface, 然后重新启动， Rails应用（这里面存在问题，是不是可以使用listener， 设定脚本，更改 其他项目时候，主动重启Rails应用呢）, 也是流程问题， 两个项目interface, rails 分开发行，分支不同
  * 项目趋于复杂， 不同模型，使用字段映射到相同的数据表，然而两个model却趋向于完全不同。导致使用统一model，的validation， callback，变得复杂，混乱。页面中使用同一个controller，出现了各种判断流程。

### 解决方法
  * << Growing Rails Application in Practice >> 中的方法是正确的，
  * 本文实现了一个类似的 class

  ```ruby

  class CmsModel

    include ActiveModel::Model
    extend ActiveModel::Callbacks

    # 如果更加简单的话，可以使用 method_missing 将全部的方法转发到 model中， 但是这样的话，存在如果保存失败的时候的情况，所以使用当前方法
    DEFAULT_PRO = Proc.new do |name, instance|
      if name != :model
        instance.instance_variable_get("@#{name.to_s}") ||
        (instance.instance_variable_get("@model").present? &&
        instance.instance_variable_get("@model").respond_to?(name) &&
        instance.instance_variable_get("@model").send(name)) ||
        nil
      else
        instance.instance_variable_get("@model")
      end
    end

    def self.cms_attr_reader(* names, &block)
      block ||= DEFAULT_PRO
      names.each do |item|
        define_method item do
          block.call(item, self)
        end
      end
    end

    # save_model 用来在callback中， before_save中， 创建model不能保存为 @model，会导致callback new? 为false
    attr_reader :save_model
    # ActiveModel::Attributes 中的方法, 使model伪装成 active record
    alias :set :assign_attributes

    # 生命callback，统一使用一个save的interface来更新，创建 model， 使用if: :persisted?进行update， create的区分
    define_model_callbacks :save
    define_model_callbacks :init

    def initialize(hash = {})
      _run_init_callbacks do
        set(hash)
      end
    end

    def save
      if valid?
        _run_save_callbacks
      else
        false
      end
    end

    # 供给页面调用，是否 持久化的判断
    def persisted?
      @model
    end

    def new?
      !persisted?
    end

    # 减少delegate 的声明
    def method_missing(method_name, *args)
      if @model && @model.respond_to?(method_name)
        @model.send(method_name, *args)
      else
        super
      end
    end

    # 工具类
    def params_present(*columns)
      (columns.flatten - [:model, :id]).inject({}) do |hash, item|
        infer = instance_variable_get("@#{item.to_s}")
        hash[item] = infer if infer.present?
        hash
      end
    end
  end


  # cms post model

  class CmsPost < CmsModel

    ATTRIBUTES = [:username, :userkey :tags, :model, :id]

    NOT_PARAMS_COLUMN = [:username, :model, :id]

    cms_attr_reader(* ATTRIBUTES)
    attr_writer(* ATTRIBUTES)

    before_save :create_model, if: :new?
    before_save :update_model, unless: :new?

    def initialize(hash = {})
      super
    end

    def model_name
      ActiveModel::Name.new(self, nil, "Post")
    end

    def params
      columns = ATTRIBUTES - NOT_PARAMS_COLUMN
      columns.map do |item|
        [item, instance_variable_get("@#{item.to_s}")]
      end.select {|_, value| value.present?}.to_h.merge(type_id: FORUM_TYPE[:community])
    end



  private

  # validator ------------------------------------
    def userkey_exist
      unless User[:userkey].present?
        errors.add(:userkey, "用户不存在")
      end
    end

  # callback ------------------------------------
  # 这其中不能保存为 model
    def create_model
      # do something
      @save_model = User.create()
    end

    def update_model
      # do something
    end

  end


  # controller

  class PostsController < ApplicationController
    before_action :set_post, only: [:show, :edit, :update, :destroy]

    def new
      @post = CmsPost.new()
    end

    def show
    end

    def edit
    end

    def create
      @post = CmsPost.new(post_params)
      respond_to do |format|
        if @post.save
          format.html { redirect_to @post, notice: '创建成功' }
        else
          format.html {render :new}
        end
      end
    end

    def update
      @post.set(post_params)
      respond_to do |format|
        if @post.save
          format.html { redirect_to @post, notice: 'Post was successfully updated.' }
        else
          format.html { render :edit}
        end
      end
    end

    private

    def set_post
      model = Post.first(id: params[:id])
      @post = CmsPost.new(model: model)
    end
  end
  ```

### 总结
  * 大量的使用rails中自带的现有功能，validatoin, callback, attributes, naming
  * 使用统一的save方法进行更新， 创建操作
  * 使用@model区分更新或者创建, persisted?的返回值
  * 使用 before callback 进行创建更新model， after callback进行关联model的区分
  * 可以通过继承提供大量的工具类型函数， 应用模板方法创建 流程
