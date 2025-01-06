---
title: Thor option parse 的代替者
date: 2017-07-21
tags: ruby, gem
---
 thor
--------

> option parse 的代替者，可以在shell中调用脚本，更方便的传递参数，转换参数类型，　设定默认值，进行必要参数校验等.


### 简单的示例：

```ruby
class Test < Thor
  desc "example FILE", "an example task"
  method_option :delete, :aliases => "-d", :desc => "Delete the file after parsing it"
  def example(file)
    puts "You supplied the file: #{file}"
    delete_file = options[:delete]
    if delete_file
      puts "You specified that you would like to delete #{file}"
    else
      puts "You do not want to delete #{file}"
    end
  end
end

thor test:example 'test.rb' -d

```

#### method_options
    aliases, type, desc, 描述参数的类型，简写形式
    method_options :value, aliases: 'v', default: 1, type: :numeric

#### invocations

```ruby
  class Counter < Thor
    desc "one", "Prints 1 2 3"
    def one
      puts 1
      invoke :two
      invoke :three
    end

    desc "two", "Prints 2 3"
    def two
      puts 2
      invoke :three
    end

    desc "three", "Prints 3"
    def three
      puts 3
    end
  end
```
#### Executable

```ruby
  #!/usr/bin/env ruby
  require "rubygems" # ruby1.9 doesn't "require" it though
  require "thor"

  class MyThorCommand < Thor
    desc "foo", "Prints foo"
    def foo
      puts "foo"
    end
  end

  MyThorCommand.start
```


> 最终要的是rails 中的generator使用thor创建，拷贝模板，　参数解析等

### 源码导读

1. MyThorCommand < Thor < Thor::Shell < Thor::Invocation < Thor::Base, 真正的执行 __send__(name, args)
2. Command 负责收集 desc， method_options的信息，形成一个command 实例
3. Base 中， 有回调函数 method_add， 继承 Thor 之后的方法定义（def method） 都会通过 method_add，然后创建命令， create_command, create_command 函数中，会将 desc， method_options 调用设定的 类变量 进行收集，创建一个command实例， 并关联到 @commands 中，等待调用。
4. start(ARGV), 调用dispatch， dispatch做了主要的工作内容， 寻找到command，找到其中的 options来创建 MyThorCommand 实例（主要设定其中的 options， args， parent_options 变量内容，动作操作在 Invocation, Shell等的initialize 初始化函数中)， 通过instance.invoke_command, command.run 来执行，对应的函数调用。
5. 其中主要工作的内容为Thor::Option.parse, 将命令行内容解析成对应的options, 因为函数调用需要对应的参数，所以，task的 参数个数是固定的，options则是非常灵活的，所以对应的应该多使用options
6. options 的参数类型， 涉及到 numeric, string, array, hash等，还有requirement 检查
