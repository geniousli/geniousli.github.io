---
title: Rails ActiveModel
date: 2017-04-25
tags: rails, activemodel, ruby
---

[AttributeAssignment](#attribute_assignment)
[Callbacks](#callbacks)
[Dirty](#dirty)
[Validations](#validations)
## rails activemodel

  ```ruby
    module ActiveModel
      extend ActiveSupport::Autoload

      autoload :AttributeAssignment
      autoload :AttributeMethods
      autoload :BlockValidator, 'active_model/validator'
      autoload :Callbacks
      autoload :Conversion
      autoload :Dirty
      autoload :EachValidator, 'active_model/validator'
      autoload :ForbiddenAttributesProtection
      autoload :Lint
      autoload :Model
      autoload :Name, 'active_model/naming'
      autoload :Naming
      autoload :SecurePassword
      autoload :Serialization
      autoload :TestCase
      autoload :Translation
      autoload :Validations
      autoload :Validator

      eager_autoload do
        autoload :Errors
        autoload :RangeError, 'active_model/errors'
        autoload :StrictValidationFailed, 'active_model/errors'
        autoload :UnknownAttributeError, 'active_model/errors'
      end
    end
  ```

## attribute_assignment
  >  让任何的ruby对象接受hash赋值，实现是动态的分发, 2. 防止 = ActionController::Parameters 没有调用permit函数

  ```ruby
    def assign_attributes(new_attributes)
      if !new_attributes.respond_to?(:stringify_keys)
        raise ArgumentError, "When assigning attributes, you must pass a hash as an argument."
      end
      return if new_attributes.nil? || new_attributes.empty?

      attributes = new_attributes.stringify_keys
      _assign_attributes(sanitize_for_mass_assignment(attributes))
    end

    private

    def _assign_attributes(attributes)
      attributes.each do |k, v|
        _assign_attribute(k, v)
      end
    end

    def _assign_attribute(k, v)
      if respond_to?("#{k}=")
        public_send("#{k}=", v)
      else
        raise UnknownAttributeError.new(self, k)
      end
    end
  ```

## attribute_methods
  1. 源码
    目的 在方法中添加前缀后缀等(好像没啥用)

      ```ruby
      module AttributeMethods
        extend ActiveSupport::Concern

        NAME_COMPILABLE_REGEXP = /\A[a-zA-Z_]\w*[!?=]?\z/
        CALL_COMPILABLE_REGEXP = /\A[a-zA-Z_]\w*[!?]?\z/

        included do
          class_attribute :attribute_aliases, :attribute_method_matchers, instance_writer: false
          self.attribute_aliases = {}
          self.attribute_method_matchers = [ClassMethods::AttributeMethodMatcher.new]
        end

        module ClassMethods

          def attribute_method_suffix(*suffixes)
            self.attribute_method_matchers += suffixes.map! { |suffix| AttributeMethodMatcher.new suffix: suffix }
          end

          def generated_attribute_methods #:nodoc:
            @generated_attribute_methods ||= Module.new {
              extend Mutex_m
            }.tap { |mod| include mod }
          end

          def define_attribute_method(attr_name)
            attribute_method_matchers.each do |matcher|
              method_name = matcher.method_name(attr_name)

              unless instance_method_already_implemented?(method_name)
                define_proxy_call true, generated_attribute_methods, method_name, matcher.method_missing_target, attr_name.to_s
              end
            end
          end

          def define_proxy_call(include_private, mod, name, send, *extra) #:nodoc:
            defn = if name =~ NAME_COMPILABLE_REGEXP
              "def #{name}(*args)"
            else
              "define_method(:'#{name}') do |*args|"
            end

            extra = (extra.map!(&:inspect) << "*args").join(", ".freeze)

            target = if send =~ CALL_COMPILABLE_REGEXP
              "#{"self." unless include_private}#{send}(#{extra})"
            else
              "send(:'#{send}', #{extra})"
            end

            mod.module_eval <<-RUBY, __FILE__, __LINE__ + 1
              #{defn}
                #{target}
              end
            RUBY
          end

          class AttributeMethodMatcher #:nodoc:
            attr_reader :prefix, :suffix, :method_missing_target

            AttributeMethodMatch = Struct.new(:target, :attr_name, :method_name)

            def initialize(options = {})
              @prefix, @suffix = options.fetch(:prefix, ''), options.fetch(:suffix, '')
              @regex = /^(?:#{Regexp.escape(@prefix)})(.*)(?:#{Regexp.escape(@suffix)})$/
              @method_missing_target = "#{@prefix}attribute#{@suffix}"
              @method_name = "#{prefix}%s#{suffix}"
            end

            def match(method_name)
              if @regex =~ method_name
                AttributeMethodMatch.new(method_missing_target, $1, method_name)
              end
            end

            def method_name(attr_name)
              @method_name % attr_name
            end

            def plain?
              prefix.empty? && suffix.empty?
            end
          end

        end
      ```

  2. 调用栈为:

      ```
       attribute_method_affix  prefix: 'reset_'
       define_attribute_methods :name

       |- attribute_method_affix 的调用产生 一个 AttributeMethodMatcher的数组，维系着 添加前缀后缀的标记
       |- define_attribute_methods
        |- attribute_method_matchers.each |matcher|
          |- define_proxy_call true, generated_attribute_methods, method_name, matcher.method_missing_target, attr_name.to_s

        define_attribute_methods 作用的symbol， 将前缀后缀的标记应用在参数上， 主动定义 prefix_name_sufix 的方法，并将方法的调用分发到 prefix_attribute_sufix函数中.
      ```

  3. 值的注意的地方

      ```ruby
        # @method_name = "prefix_%s", method_name(name) => prefix_name
        def method_name(attr_name)
          @method_name % attr_name
        end

        # Module.new 产生新的module 并在当前类中include， 之后 define_proxy_call等都是在这个 module 中进行操作， 包括去除方法等， 保证了，不影响类中的其他代码
        def generated_attribute_methods #:nodoc:
          @generated_attribute_methods ||= Module.new {
            extend Mutex_m
          }.tap { |mod| include mod }
        end
      ```

## callbacks
  1. 代码调用

      ```ruby
        #activesupport
        class Record
          include ActiveSupport::Callbacks
          define_callbacks :save
          set_callback :save, :before, :saving_message

          def saving_message
            puts "saving..."
          end

          set_callback :save, :after do |object|
            puts "saved"
          end

          def save
            run_callbacks :save do
              puts "- save"
            end
          end
        end

        person = PersonRecord.new
        person.save
      ```

  2. 调用栈分析

      ```
        # /activesupport/callbacks
        |- define_callbacks(names)
          |- class_attribute "_#{name}_callbacks", instance_writer: false
          |- set_callbacks name, CallbackChain.new(name, options)
        |- set_callback :save, :before, :saving_message
          |- callback_chain = get_callbacks(name)
          |- Callback.build(callback_chain, filter, type, options)
          |- set_callbacks name, callback_chain
        |- run_callbacks :save  
          |- __run_callbacks__(callbacks, &block)
            |- runner = callbacks.compile
              |- CallbackChain#compile
                |- final_sequence = CallbackSequence.new { |env| Filters::ENDING.call(env) }
                |- @chain.reverse.inject(final_sequence) do |callback_sequence, callback|
                |-  callback.apply callback_sequence
                |- end
            |- runner.call(Filters::Environment.new(self, false, nil block)).value
            |- e = Filters::Environment.new(self, false, nil, block)
            |- runner.call(e).value
      ```
  3. 代码
      ```ruby
        def define_callbacks(*names)
          options = names.extract_options!

          names.each do |name|
            class_attribute "_#{name}_callbacks", instance_writer: false
            set_callbacks name, CallbackChain.new(name, options)

            module_eval <<-RUBY, __FILE__, __LINE__ + 1
              def _run_#{name}_callbacks(&block)
                __run_callbacks__(_#{name}_callbacks, &block)
              end
            RUBY
          end
        end
        def set_callback(name, *filter_list, &block)
          type, filters, options = normalize_callback_params(filter_list, block)
          self_chain = get_callbacks name
          mapped = filters.map do |filter|
            Callback.build(self_chain, filter, type, options)
          end

          __update_callbacks(name) do |target, chain|
            options[:prepend] ? chain.prepend(*mapped) : chain.append(*mapped)
            target.set_callbacks name, chain
          end
        end


        def __run_callbacks__(callbacks, &block)
          if callbacks.empty?
            yield if block_given?
          else
            runner = callbacks.compile
            e = Filters::Environment.new(self, false, nil, block)
            runner.call(e).value
          end
        end

        class Callback #:nodoc:#
          def self.build(chain, filter, kind, options)
            new chain.name, filter, kind, options, chain.config
          end

          attr_accessor :kind, :name
          attr_reader :chain_config

          def initialize(name, filter, kind, options, chain_config)
            @chain_config  = chain_config
            @name    = name
            @kind    = kind
            @filter  = filter
            @key     = compute_identifier filter
            @if      = Array(options[:if])
            @unless  = Array(options[:unless])
          end

          def compile
            @callbacks || @mutex.synchronize do
              final_sequence = CallbackSequence.new { |env| Filters::ENDING.call(env) }
              @callbacks ||= @chain.reverse.inject(final_sequence) do |callback_sequence, callback|
                callback.apply callback_sequence
              end
            end
          end

          def apply(callback_sequence)
            user_conditions = conditions_lambdas
            user_callback = make_lambda @filter

            case kind
            when :before
              Filters::Before.build(callback_sequence, user_callback, user_conditions, chain_config, @filter)
            when :after
              Filters::After.build(callback_sequence, user_callback, user_conditions, chain_config)
            when :around
              Filters::Around.build(callback_sequence, user_callback, user_conditions, chain_config)
            end
          end

        end

        class CallbackSequence
          def initialize(&call)
            @call = call
            @before = []
            @after = []
          end

          def before(&before)
            @before.unshift(before)
            self
          end

          def after(&after)
            @after.push(after)
            self
          end

          def around(&around)
            CallbackSequence.new do |arg|
              around.call(arg) {
                self.call(arg)
              }
            end
          end

          def call(arg)
            @before.each { |b| b.call(arg) }
            value = @call.call(arg)
            @after.each { |a| a.call(arg) }
            value
          end
        end

          class Before
            def self.build(callback_sequence, user_callback, user_conditions, chain_config, filter)
              halted_lambda = chain_config[:terminator]

              if user_conditions.any?
                halting_and_conditional(callback_sequence, user_callback, user_conditions, halted_lambda, filter)
              else
                halting(callback_sequence, user_callback, halted_lambda, filter)
              end
            end

            def self.halting_and_conditional(callback_sequence, user_callback, user_conditions, halted_lambda, filter)
              callback_sequence.before do |env|
                target = env.target
                value  = env.value
                halted = env.halted

                if !halted && user_conditions.all? { |c| c.call(target, value) }
                  result_lambda = -> { user_callback.call target, value }
                  env.halted = halted_lambda.call(target, result_lambda)
                  if env.halted
                    target.send :halted_callback_hook, filter
                  end
                end

                env
              end
            end
            private_class_method :halting_and_conditional

            def self.halting(callback_sequence, user_callback, halted_lambda, filter)
              callback_sequence.before do |env|
                target = env.target
                value  = env.value
                halted = env.halted

                unless halted
                  result_lambda = -> { user_callback.call target, value }
                  env.halted = halted_lambda.call(target, result_lambda)

                  if env.halted
                    target.send :halted_callback_hook, filter
                  end
                end

                env
              end
            end
            private_class_method :halting
          end
      ```

       > 大概的调用 调用顺序为, define_callbacks :save 定义callback的名字，初始化CallbackChain, set_callback :save, :before, :save_messsage, 在CallbackChain中添加Callback 的实例, 在最后run_callback 的时候最为重要， 将前面两步做的操作都进行了兑现， 1. 调用CallBackChain#compile, compile 有意思的是初始化了一个CallbackSequence实例， 并对Callback数组进行inject Callback.apply 中最为重要， CallBack.apply 中将 Filters::Before, Filters::After中的其他通用部分， 执行条件(set_callback :save, :before, :do_something if: :xxx?), hook函数(do_something) 进行了lambda化，(条件为conditions_lambdas函数， hook为make_lambda), 并根据 kind(:before, :after, :around)进行了分发， 下降到Filters::Before 中， 则在 hook 的lambda的基础上进行了， 完善，完成了各自 Before, After 等的具体操作，这是最底层的具体到执行逻辑的部分。2. 执行， 上面的lambda化结果， 提供了一个Environment的环境， 包含:halt, :value, :target 等信息，

  4. active_model 中的 callback

      > active_model 中封装了， activesupport 中设定callback的繁琐步骤，而是通过 define_model_callbacks 来设定

      ```
        |- define_model_callbacks :save
          |- define_callbacks(callback, options)
          |- send("_define_#{type}_model_callback", self, callback)
            |-  klass.define_singleton_method("before_#{callback}") do |*args, &block|
            |-    set_callback(:"#{callback}", :before, *args, &block)
            |-  end

      ```

      ```ruby
      module ActiveModel
        module Callbacks
          def self.extended(base) #:nodoc:
            base.class_eval do
              include ActiveSupport::Callbacks
            end
          end

          def define_model_callbacks(*callbacks)
            options = callbacks.extract_options!
            options = {
              terminator: deprecated_false_terminator,
              skip_after_callbacks_if_terminated: true,
              scope: [:kind, :name],
              only: [:before, :around, :after]
            }.merge!(options)

            types = Array(options.delete(:only))

            callbacks.each do |callback|
              define_callbacks(callback, options)
              types.each do |type|
                send("_define_#{type}_model_callback", self, callback)
              end
            end
          end

          private

          def _define_before_model_callback(klass, callback) #:nodoc:
            klass.define_singleton_method("before_#{callback}") do |*args, &block|
              set_callback(:"#{callback}", :before, *args, &block)
            end
          end

          def _define_around_model_callback(klass, callback) #:nodoc:
            klass.define_singleton_method("around_#{callback}") do |*args, &block|
              set_callback(:"#{callback}", :around, *args, &block)
            end
          end

          def _define_after_model_callback(klass, callback) #:nodoc:
            klass.define_singleton_method("after_#{callback}") do |*args, &block|
              options = args.extract_options!
              options[:prepend] = true
              conditional = ActiveSupport::Callbacks::Conditionals::Value.new { |v|
                v != false
              }
              options[:if] = Array(options[:if]) << conditional
              set_callback(:"#{callback}", :after, *(args << options), &block)
            end
          end
        end
      end

      ```

  5. good-part

      > 代码中使用了，大量的lambda化最为震撼（其中包括 条件执行，hook函数）， 将符号配置， 执行条件配置都lambda化， 变成了一个可以随时执行的代码(compile 的含义很符合)， 存储在一个数组中，hook的执行 变成了执行数组的 each item.call, 其中个个callback如何确定环境（比如是否执行，是否已经callback中断了）通过Environment 对象传递

  6. 问题
    **问题为什么需要CallbackChain 到CallbackSequence的转换呢？**
      - CallbackChain中主要针对Callback的实例数组操作， delete, append, prepend, clear, compile 等, CallbackSequence 也是类似的，存储lambda之后的可执行数组

## dirty
  > dirty 根据文档的描述是，为了提供一个工具集来跟踪变量变化(之前以为是auto， 用了一些callback， 或者元编程之类的结果不是！！！， 而是提供了一个工具集， 在使用中， 需要手动调用方法)

  下面的可见一斑:

  1. 在使用的类中 include ActiveModel::Dirty
  2. 使用define_attribute_methods 设定跟踪的变量
  3. 在变量变化之前 调用 [attr_name_]will_change!, 例如name, 在name=()中调用name_will_change!
  4. 调用函数 changes_applied 来"持久化", 将现有的改动， 复制到之前（previous）中

  ```ruby
      include ActiveModel::AttributeMethods

      included do
        attribute_method_suffix '_changed?', '_change', '_will_change!', '_was'
        attribute_method_suffix '_previously_changed?', '_previous_change'
        attribute_method_affix prefix: 'restore_', suffix: '!'
      end

      def attribute_will_change!(attr)
        return if attribute_changed?(attr)

        begin
          value = __send__(attr)
          value = value.duplicable? ? value.clone : value
        rescue TypeError, NoMethodError
        end

        set_attribute_was(attr, value)
      end

      def changed_attributes
        @changed_attributes ||= ActiveSupport::HashWithIndifferentAccess.new
      end

      def previous_changes
        @previously_changed ||= ActiveSupport::HashWithIndifferentAccess.new
      end

      # This is necessary because `changed_attributes` might be overridden in
      # other implementations (e.g. in `ActiveRecord`)
      alias_method :attributes_changed_by_setter, :changed_attributes # :nodoc:


      def changes_applied # :doc:
        @previously_changed = changes
        @changed_attributes = ActiveSupport::HashWithIndifferentAccess.new
      end
  ```

## validations
  1. 示例代码

      ```ruby
        class Person
          include ActiveModel::Validations
          validates_with MyValidator
        end

        class MyValidator < ActiveModel::Validator
          def validate(record)
            if some_complex_logic
              record.errors.add(:base, "This record is invalid")
            end
          end

          private
            def some_complex_logic
              # ...
            end
        end
      ```

  2. 调用栈分析

      ```
        # validation 中的代码
          define_callbacks :validate, scope: :name
          class_attribute :_validators, instance_writer: false

        |- validates_with MyValidator
          |- options = args.extract_options!
          |- validator = klass.new(options, &block)
          |- validator.attributes.each {|attribute| _validators[attribute.to_sym] << validator }
          |- validate(validator, options)
            |- set_callback(:validate, *args, &block)

        # 一些内置的validator, 例如PresenceValidator
        |- validates_presence_of(*attr_names)
          |- validates_with PresenceValidator, _merge_attributes(attr_names)

        |- valid?(context = nil)
          |- run_validations!
            |- _run_validate_callbacks
            .......
            |- validate(record)
              |- validate_each(record, attribute, value)
      ```

      **一个主要的数据流走向为:**

        1. > 在include  Validator 时候， define_callbacks :validate, scope: :name。 设定callback， scope: :name 的作用为调用 一个instance callback时候调用的方法, 在 define_callbacks :save, MyValidator, :before, scope: [:kind, :name], 在执行callback时候调用 MyValidator#before_save, Callback#apply 中存在

        2. > validates_with Class, options, 构造实例， 调用validate， 调用set_callback设定 callback时候的lambda, 2. 存在一些内置的校验器, validates_presence_of :name， 通过统一的调用 validate_with Class, options

        3. > valid?, 执行callback 中的lambda校验， 其中有意思的是valid?(context = nil), valid?的调用存在一个 context参数， 可以传递 在 validate_with Class, on: :save，的参数， 来决定执行特定环境中的validate， 所以猜想贯穿到 ActiveRecord#.save 时候的调用，应该存在类似的 valid?(:save)代码逻辑, 执行个个validator实例中的validate函数， 通过 继承EachValidator 分发到validate_each


      ```ruby
      class EachValidator < Validator #:nodoc:
        attr_reader :attributes

        def initialize(options)
          @attributes = Array(options.delete(:attributes))
          raise ArgumentError, ":attributes cannot be blank" if @attributes.empty?
          super
        end
      end

      # active_model/validation.rb
      module ActiveModel
        module Validations
          extend ActiveSupport::Concern

          included do
            extend ActiveModel::Callbacks
            extend  HelperMethods
            include HelperMethods

            attr_accessor :validation_context
            private :validation_context=
            define_callbacks :validate, scope: :name

            class_attribute :_validators, instance_writer: false
            self._validators = Hash.new { |h,k| h[k] = [] }
          end

          module ClassMethods
            def validate(*args, &block)
              options = args.extract_options!
              if options.key?(:on)
                options = options.dup
                options[:if] = Array(options[:if])
                options[:if].unshift ->(o) {
                  !(Array(options[:on]) & Array(o.validation_context)).empty?
                }
              end

              args << options
              set_callback(:validate, *args, &block)
            end
          end
        end
      end

      # active_model/validations/with.rb
      module ActiveModel
        module Validations
          module ClassMethods
            def validates_with(*args, &block)
              options = args.extract_options!
              options[:class] = self

              args.each do |klass|
                validator = klass.new(options, &block)

                if validator.respond_to?(:attributes) && !validator.attributes.empty?
                  validator.attributes.each do |attribute|
                    _validators[attribute.to_sym] << validator
                  end
                else
                  _validators[nil] << validator
                end

                validate(validator, options)
              end
            end
          end
        end
      end

      module ActiveModel

        module Validations
          class PresenceValidator < EachValidator # :nodoc:
            def validate_each(record, attr_name, value)
              record.errors.add(attr_name, :blank, options) if value.blank?
            end
          end

          module HelperMethods
            def validates_presence_of(*attr_names)
              validates_with PresenceValidator, _merge_attributes(attr_names)
            end
          end
        end
      end



      class EachValidator < Validator #:nodoc:
        attr_reader :attributes

        def initialize(options)
          @attributes = Array(options.delete(:attributes))
          raise ArgumentError, ":attributes cannot be blank" if @attributes.empty?
          super
          check_validity!
        end

        def validate(record)
          attributes.each do |attribute|
            value = record.read_attribute_for_validation(attribute)
            next if (value.nil? && options[:allow_nil]) || (value.blank? && options[:allow_blank])
            validate_each(record, attribute, value)
          end
        end
      end

      ```
