---
title: Rust little book
date: 2021-02-20
tags: rust, cargo, rustup
---
## Rust little book

### rustup:  rust complier 的管理工具， 可以方便的切换 stable, beta, and nightly
### cargo 是rust的包管理工具， 用来 下载 rust的依赖， 编译， 以及 分发 到 crates.io
  * 在安装 rust 之后， cargo 也会被自动安装上
  * cargo 提供了一些有用的工具有:
    1. cargo new package # default --bin 生成 可执行 program, 可以传递 --lib 来产生库程序
    2. cargo build
    3. cargo 存在的意义：
       * 剥离 rustc 的复杂度， 类似 make 与 c 一样
       * cargo 最终调用rustc 来编译 项目， 当然可以 直接使用 rustc 来编译项目，但是 需要出入 复杂的参数 来 添加项目 依赖关系， 编译文件， 依赖关系 等，并精心安排顺序 来进行调用。
       * 所以使用cargo： make工具， cargo 的功能
          1. 使用两个文件 来 包含 package的信息
          2. 拉取，构建  package 依赖
          3. 使用正确的参数 来调用rustc 或者其他 tool， 来构建项目
          4. 引用约定，方便package 构建


### cargo 使用笔记：
* cargo new hello_world --bin

```shell
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

* 其中 Cargo.toml 被称为 manifest,包含package的元数据 

```shell
fn main() {
    println!("Hello, world!");
}


$ cargo build
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)

$ ./target/debug/hello_world
Hello, world!


$ cargo run
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
     Running `target/debug/hello_world`
Hello, world!



$ cargo build --release
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)

```


* cargo build  将会 构建 package
* cargo run 则 构建并运行它
* cargo build --release 将 构建 优化的代码
* cargo 默认的构建 代码优化级别 为 debug, 存在的目录为 target/debug, 构建 优化后的代码需要 显式传递 参数 --release 生成的文件目录为 target/release
* Dependencies：
  * crate.io 为 rust 中间的 package 机构， 用于发现 下载 更新package
  * 添加依赖关系： 在 Cargo.toml 中 的dependencies 下，添加项目

  ```shell
  [package]
  name = "hello_world"
  version = "0.1.0"
  authors = ["Your Name <you@example.com>"]
  edition = "2018"

  [dependencies]
  time = "0.1.12"
  regex = "0.1.41"
  ```

  * 之后的 cargo build 过程

  ```shell
  $ cargo build
        Updating crates.io index
     Downloading memchr v0.1.5
     Downloading libc v0.1.10
     Downloading regex-syntax v0.2.1
     Downloading memchr v0.1.5
     Downloading aho-corasick v0.3.0
     Downloading regex v0.1.41
       Compiling memchr v0.1.5
       Compiling libc v0.1.10
       Compiling regex-syntax v0.2.1
       Compiling memchr v0.1.5
       Compiling aho-corasick v0.3.0
       Compiling regex v0.1.41
       Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
  ```
  
#### package 构成：

```shell 
  .
  ├── Cargo.lock
  ├── Cargo.toml
  ├── src/
  │   ├── lib.rs
  │   ├── main.rs
  │   └── bin/
  │       ├── named-executable.rs
  │       ├── another-executable.rs
  │       └── multi-file-executable/
  │           ├── main.rs
  │           └── some_module.rs
  ├── benches/
  │   ├── large-input.rs
  │   └── multi-file-bench/
  │       ├── main.rs
  │       └── bench_module.rs
  ├── examples/
  │   ├── simple.rs
  │   └── multi-file-example/
  │       ├── main.rs
  │       └── ex_module.rs
  └── tests/
      ├── some-integration-tests.rs
      └── multi-file-test/
          ├── main.rs
          └── test_module.rs
  ```
  * Cargo.toml and Cargo.lock 在项目的根目录
  * src 下 为源代码
  * 默认的 library file 为 src/lib.rs
  * 默认的 executable file 是 src/main.rc, 其他的 放在 src/bin/
  * 基准测试 放在benches 目录下
  * 示例代码放在examples 目录下
  * 集成测试 放在 tests 目录下
  * 其他详细的需要参看 [[https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html]]


* Cargo.toml 与 Cargo.lock: 两种目的， 
  * cargo.toml 描述 大概的依赖关系 并不准确，是由 人来确定的
  * Cargo.lock 包含准确的依赖关系， 由cargo 来维护
  * [[https://doc.rust-lang.org/cargo/faq.html#why-do-binaries-have-cargolock-in-version-control-but-not-libraries]]
  * 示例:

  ```shell 
  Cargo.toml

  [package]
  name = "hello_world"
  version = "0.1.0"
  authors = ["Your Name <you@example.com>"]

  [dependencies]
  rand = { git = "https://github.com/rust-lang-nursery/rand.git", rev = "9f35b8e" }


  Cargo.lock


  [[package]]
  name = "hello_world"
  version = "0.1.0"
  dependencies = [
   "rand 0.1.0 (git+https://github.com/rust-lang-nursery/rand.git#9f35b8e439eeedd60b9414c58f389bdc6a3284f9)",
  ]

  [[package]]
  name = "rand"
  version = "0.1.0"
  source = "git+https://github.com/rust-lang-nursery/rand.git#9f35b8e439eeedd60b9414c58f389bdc6a3284f9"
  ```

 * Cargo.lock 中包含 依赖的确定的 version， 当其他人使用的时候， 他们将使用相同的 sha，即便我们并没有在Cargo.toml 中使用
  * cargo update 更新全部的依赖， cargo update -p rand 只更新依赖 rand

* Test: 
  * cargo test 执行 package中的所有test， test主要有两种： 1) 在每个 src 目录中的文件， 2） tests/ 目录下的所有文件。 1）中的为单元测试， 2）则为 集成测试，

  ```shell 
  $ cargo test
     Compiling rand v0.1.0 (https://github.com/rust-lang-nursery/rand.git#9f35b8e)
     Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
       Running target/test/hello_world-9c2b65bbb79eabce

  running 0 tests

  test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
  ```
  * cargo test foo  可以单独执行 名字为foo的测试，
  * cargo test 其实还会执行 额外的测试，包含在 src中的部分 文档中的测试（并不重要，为补充部分）
* Cargo Home： 当build package的时候， cargo 将下载的依赖package 存储到 Cargo home 下。当做cache 使用。
  * 可以通过 改变 环境变量 CARGO_HOME 来改变 cargo home的值， 默认 为 $HOME/.cargo/
  * Cargo Home 目录下的 数据：
     * bin 目录： 可执行crate， 包括cargo install 或者 rustup 安装的
     * git/db: crate 依赖git 项目时， cargo clone 项目到 该目录下
     * git/checkouts： git/db 项目中的检出到该文件， 比如 依赖于特定的commit
* registry： 项目依赖于 crate.io 中的crate 存放在该目录下
  * registry/index： crate 的原数据， 包括： version， dependencies 等
  * registry/cache： 下载的crate 储存到该目录下， 存储形式为 .crate 的gzip压缩文件
  * registry/src： cache 的解压形式 存放在 该文件中
    
#### 指定 Dependencies： 
* 依赖版本： 版本号各个位置数字的含义： [[https://semver.org/][SemVer compatible]] 与 link 中讲的同样， major.minor.patch
* Caret requirements： 指定 可以使用 一个 major 版本号 不变的更新， 但是 0 是 一个特殊的数字，标识不与 任何 数字兼容。即是： 0.0.1， 与 0.1.x 不兼容
* 下面为兼容样例： 

  ```shell 
  ^1.2.3  :=  >=1.2.3, <2.0.0
  ^1.2    :=  >=1.2.0, <2.0.0
  ^1      :=  >=1.0.0, <2.0.0
  ^0.2.3  :=  >=0.2.3, <0.3.0
  ^0.2    :=  >=0.2.0, <0.3.0
  ^0.0.3  :=  >=0.0.3, <0.0.4
  ^0.0    :=  >=0.0.0, <0.1.0
  ^0      :=  >=0.0.0, <1.0.0
  ```

* Tilde requirements:  分为以下情况:
  * 有 major.minor.patch 或者  major.minor 只有patch version 的升级是允许的
  * 有major情况下, minor patch verison的升级 是允许的
  * for example

  ```shell 
  ~1.2.3  := >=1.2.3, <1.3.0
  ~1.2    := >=1.2.0, <1.3.0
  ~1      := >=1.0.0, <2.0.0
  ```

* Wildcard requirements: 
    * 允许所在位置上的 任何版本
    * for example

  ```shell 
  \*     := >=0.0.0
  1.*   := >=1.0.0, <2.0.0
  1.2.* := >=1.2.0, <1.3.0
  ```

#### Workspaces: workspace 下的一系列package 共享同样的Cargo.lock， output dir 等配置（比如profile）， workspace下的packages 被称为 workspace members 
* 存在两种形式： 1） [package] 与 [workspace] 共存在 Cargo.toml 中 2） 只有 [workspace] 存在Cargo.toml 中， 被称作 Virtual manifest 
* 主要作用： 
  1. 共享Cargo.lock
  2. 共享output dir， Cargo.toml 中 [target] 
  3. [patch], [replace] and [profile.*] 等 在Cargo.toml 中的 段  只能识别在 workspace 中的 manifest，member package 中的被忽略
* workspace section 中的配置

  ```shell 
  [workspace]
  members = ["member1", "path/to/member2", "crates/*"]
  exclude = ["crates/foo", "path/to/other"]
  ```

* members 为 成员package 列表， exclude 排除 member
  * workspace 寻找： Cargo 自动的 向上目录 寻找 含有 [workspace] 的Cargo.toml， 在 member中可以指定 package.workspace 来 直接指定 workspace 的位置， 来防止自动查找， 这个对于 没有在 workspace 目录下的member package 非常有用
  * member package： cargo command -p package 可以指定 package 来 执行命令， 如果没有指定 package， 则 选择当前所在目录的package， default-members = ["path/to/member2", "path/to/member3/"] 可以指定默认的 操作的member package

###  rustup： 从官方下载rustc， 使能够随意的在 stable, beta, nightly 中切换。 让 cross-compiling 编译变的简单
* 工作原理：  rustup 通过 ~/.cargo/bin 下的工具 来实现其功能， 比如 安装在~/.cargo/bin   下的  rustc cargo 等 只是一个 到真正执行工具的 代理，
* rustup 提供了一个方便的机制来 控制 这些代理的行为， 比如 通过执行rustup default nightly 来切换 nightly 下的工具

#### 概念: 
* channel: rustc 按照 beta, night, stable 三个 channel 进行发布， channel 并没有什么用， 只是个概念而已。
* toolchain： rustc and cargo 等相关工具。 因为能够控制rustc (即是channel概念下的实际应用)
* target： rustc 可以为多个平台生成代码。 默认的 rustc 使用host(即本机) 作为target， 为了生成不同target的代码，我们需要 使用rustup target 来安装目标target
* component: 每个rust版本的发布，都会包含一些 组件， 包括rustc， clippy 等
* profile： 为了更好的与component 工作， profile 定义了 一组component，

> toolchain：  rustup 不仅可以 安装stable, beta, nightly 三个channel， 还可以安装 其他的 官方 历史版本

* channel 的命令规则:

```shell
<channel>[-<date>][-<host>]

<channel>       = stable|beta|nightly|<major.minor>|<major.minor.patch>
<date>          = YYYY-MM-DD
<host>          = <target-triple>

#+end_src
** 其他命令： 保持 rust 更新： 
#+begin_src 
$ rustup update
info: syncing channel updates for 'stable'
info: downloading component 'rustc'
info: downloading component 'rust-std'
info: downloading component 'rust-docs'
info: downloading component 'cargo'
info: installing component 'rustc'
info: installing component 'rust-std'
info: installing component 'rust-docs'
info: installing component 'cargo'
info: checking for self-updates
info: downloading self-updates

  stable updated: rustc 1.7.0 (a5d1e7a59 2016-02-29)
```
>  如上， rustup update 会更新 stable， component， 以及 rustup self， 可以使用 rustup self update 来手动更新rustup



### Rust 知识： 
#### std::fmt:
* format! 的使用 方法：  positional params：， named params： ， formating params： 
* 示例

  ```rust
  format!("Hello, {}!", "world");   // => "Hello, world!"
  format!("{1} {} {0} {}", 1, 2); // => "2 1 1 2"
  format!("{argument}", argument = "test");
  println!("Hello {:1$}!", "x", 5);
  ```

* formating trait: 实践显式 对于外部的Type  {}确实需要impl Display， 而{:?} 需要 impl Debug
   1. {} => Display
   2. \{:?\} => Debug
   3. \{:o\} => Octal
   4. \{:p\} => Pointer
* fmt::Display vs fmt::Debug
   1. Display: 断言 实现者 总是返回 UTF-8 的字符串， 并非所有的 都实现了 Display
   2. Debug：  应该为所有pub type 实现， 输出为 内部状态， 该Trait 的目的是为了方便Rust Debug， 可以 使用#[derive(Debug)] 来使用默认的内部实现
   
#### array & Slice
  * array的类型为： [T: len], let mut array: [i32; 3] = [0; 3]; 
  * Slice: [T]
  
#### structures： 有三种类型： 1） Tuple struct, 2) classic struct 3) Unit structs
  * struct Pair(i32, f32);
  * struct Person {  name: String,    age: u8}
  * struct Unit; unit Struct 没有任何的 field
  
#### Enums： 包含多个 变体 的 组合项， 任何一个变体 都是一个 正确的 enum 类型

  ``` rust
  enum WebEvent {
      // An `enum` may either be `unit-like`,
      PageLoad,
      PageUnload,
      // like tuple structs,
      KeyPress(char),
      Paste(String),
      // or c-like structures.
      Click { x: i64, y: i64 },
  }

  // A function which takes a `WebEvent` enum as an argument and
  // returns nothing.
  fn inspect(event: WebEvent) {
      match event {
          WebEvent::PageLoad => println!("page loaded"),
          WebEvent::PageUnload => println!("page unloaded"),
          // Destructure `c` from inside the `enum`.
          WebEvent::KeyPress(c) => println!("pressed '{}'.", c),
          WebEvent::Paste(s) => println!("pasted \"{}\".", s),
          // Destructure `Click` into `x` and `y`.
          WebEvent::Click { x, y } => {
              println!("clicked at x={}, y={}.", x, y);
          },
      }
  }
  ```
  
#### Type Alias：type 关键字能够 使用 type Name = ExistingType； 语法来 使用 Name 代替 ExistingType 使用。 Self 是一种Type Alias

#### const， static

#### Variable Bindings： 1）变量默认 是不可修改的， 使用mut 改变 2） 可以在内部的scope中 其同样的名字来shadow(即使不可见) 外部的变量 3）可以使用 先声明 后设定数值的形式 使用变量，但是rust 会检查 使用 未定义变量的错误， 来预防因此产生的问题

#### Types: 1）转换 as关键字  2） type alias： type NanoSecond = u64; 3） 数值的类型，可以添加到  后面最为后缀使用， 例如： 42i32

#### Conversion： rust 的struct 以及 enum 等自定义类型的 type转换

1.  From & Into: 
  * From 为一个类型定义， 如何create self 从 另一个type中转变
  * Into 则是From 的 调用者， From<T> for U 自动实现了 Into<U> for T ( blank implement)

2. TryFrom & TryInto: 类似于 From & Into 不同的是， 转换可能失败，返回Result
   *  ToString & FromStr:
   *  ToString： 单独为 String 类型 定义了一个 ToString Trait，但是并不需要直接实现 ToString，而是实现了 fmt::Display 之后 就自动了提供了 ToString 中的to_string 方法
   
```rust
    #[stable(feature = "rust1", since = "1.0.0")]
impl<T: fmt::Display + ?Sized> ToString for T {
    // A common guideline is to not inline generic functions. However,
    // removing `#[inline]` from this method causes non-negligible regressions.
    // See <https://github.com/rust-lang/rust/pull/74852>, the last attempt
    // to try to remove it.
    #[inline]
    default fn to_string(&self) -> String {
        use fmt::Write;
        let mut buf = String::new();
        buf.write_fmt(format_args!("{}", self))
            .expect("a Display implementation returned an error unexpectedly");
        buf
    }
}
```

#### FromStr & parse: 将String 转换为其他类型， 只需要实现了 FromStr for struct, 而String 中的parse 方法 只是 对FromStr::from_str(&string) 的调用

#### Expression： 程序是由一系列表达式组成的， 1） 赋值表达式 用; 结尾， 2） {} 也是表达式， 如果最后一个表达式 以; 结尾，则返回  (), 否则为最后一个表达式的 结果

#### Flow of control：
* if-else  也是表达式， 所有的分支必须返回同样的类型
* loop： loop break continue。 break 用来随时中断退出loop， continue 则用于 用于 跳过剩下的 代码，重新开始一个 循环
* loop 是可以嵌套的，并起名字, break ，以及continue 可以使用名字来进行 break， 或者continue

  ``` rust
  #![allow(unreachable_code)]

  fn main() {
      'outer: loop {
          println!("Entered the outer loop");

          'inner: loop {
              println!("Entered the inner loop");

              // This would break only the inner loop
              //break;

              // This breaks the outer loop
              break 'outer;
          }

          println!("This point will never be reached");
      }

      println!("Exited the outer loop");
  }

  fn main() {
      let mut counter = 0;

      let result = loop {
          counter += 1;

          if counter == 10 {
              break counter * 2;
          }
      };

      assert_eq!(result, 20);
  }

  ```

* loop 也是可以 返回数值的， 放到break 后面
* while
* for: for in 结构用来 遍历 所有实现了 IntoIterator 的对象， 比如简单 range形式： a..b, a..=b
  * for loop 会自动调用 into_iter 在参数上，我们可以主动产生下面几类实现IntoIterator 的 Iterator：
* iter: 产生 引用的 Iterator, 对 ownership不产生影响
  * into_iter: 将ownership 交给 Iterator， 调用过之后的对象，将不再可用。产生
  * iter_mut: 产生mut 引用的Iterator， 可以进行修改
* match: 
  * c like 方式： 即 match  number
  * 解构对象：
    * Tuples：  使用.. 来 忽略剩余所有的 tuple
    * Enums: 
    * Pointers: *  & ref ref mut 见下面示例
    * Structs: struct 同样可以被match
  * Guards: 在match 对象的arm中，同样可以 使用 if 条件判断 即是 guards
  * Bindings： match 在 arm中，除了解构对象的同时 可以将变量整体绑定 到一个变量上

    ``` rust
    fn main() {
        let triple = (0, -2, 3);
        // TODO ^ Try different values for `triple`

        println!("Tell me about {:?}", triple);
        // Match can be used to destructure a tuple
        match triple {
            // Destructure the second and third elements
            (0, y, z) => println!("First is `0`, `y` is {:?}, and `z` is {:?}", y, z),
            (1, ..)  => println!("First is `1` and the rest doesn't matter"),
            // `..` can be the used ignore the rest of the tuple
            _      => println!("It doesn't matter what they are"),
            // `_` means don't bind the value to a variable
        }
    }

    //point s 

    fn main() {
        // Assign a reference of type `i32`. The `&` signifies there
        // is a reference being assigned.
        let reference = &4;

        match reference {
            // If `reference` is pattern matched against `&val`, it results
            // in a comparison like:
            // `&i32`
            // `&val`
            // ^ We see that if the matching `&`s are dropped, then the `i32`
            // should be assigned to `val`.
            &val => println!("Got a value via destructuring: {:?}", val),
        }

        // To avoid the `&`, you dereference before matching.
        match *reference {
            val => println!("Got a value via dereferencing: {:?}", val),
        }

        // What if you don't start with a reference? `reference` was a `&`
        // because the right side was already a reference. This is not
        // a reference because the right side is not one.
        let _not_a_reference = 3;

        // Rust provides `ref` for exactly this purpose. It modifies the
        // assignment so that a reference is created for the element; this
        // reference is assigned.
        let ref _is_a_reference = 3;

        // Accordingly, by defining 2 values without references, references
        // can be retrieved via `ref` and `ref mut`.
        let value = 5;
        let mut mut_value = 6;

        // Use `ref` keyword to create a reference.
        match value {
            ref r => println!("Got a reference to a value: {:?}", r),
        }

        // Use `ref mut` similarly.
        match mut_value {
            ref mut m => {
                // Got a reference. Gotta dereference it before we can
                // add anything to it.
                *m += 10;
                println!("We added 10. `mut_value`: {:?}", m);
            },
        }
    }

    fn main() {
        struct Foo {
            x: (u32, u32),
            y: u32,
        }

        // Try changing the values in the struct to see what happens
        let foo = Foo { x: (1, 2), y: 3 };

        match foo {
            Foo { x: (1, b), y } => println!("First of x is 1, b = {},  y = {} ", b, y),

            // you can destructure structs and rename the variables,
            // the order is not important
            Foo { y: 2, x: i } => println!("y is 2, i = {:?}", i),

            // and you can also ignore some variables:
            Foo { y, .. } => println!("y = {}, we don't care about x", y),
            // this will give an error: pattern does not mention field `x`
            //Foo { y } => println!("y = {}", y),
        }
    }


    fn main() {
        let pair = (2, -2);
        // TODO ^ Try different values for `pair`

        println!("Tell me about {:?}", pair);
        match pair {
            (x, y) if x == y => println!("These are twins"),
            // The ^ `if condition` part is a guard
            (x, y) if x + y == 0 => println!("Antimatter, kaboom!"),
            (x, _) if x % 2 == 1 => println!("The first one is odd"),
            _ => println!("No correlation..."),
        }
    }


    // A function `age` which returns a `u32`.
    fn age() -> u32 {
        15
    }

    fn main() {
        println!("Tell me what type of person you are");

        match age() {
            0             => println!("I haven't celebrated my first birthday yet"),
            // Could `match` 1 ..= 12 directly but then what age
            // would the child be? Instead, bind to `n` for the
            // sequence of 1 ..= 12. Now the age can be reported.
            n @ 1  ..= 12 => println!("I'm a child of age {:?}", n),
            n @ 13 ..= 19 => println!("I'm a teen of age {:?}", n),
            // Nothing bound. Return the result.
            n             => println!("I'm an old person of age {:?}", n),
        }
    }

    ```

* if let: 在判断的同时 进行match
* while let

#### Functions: 

* methods: 依附于 对象的函数， 在methods 的block中，能够 通过self使用 对象的 数据
* closures:  \|val\| {val + x}
  * Capturing： 捕获， 其可以 捕获环境中的 变量， 可以是： &T, &mut T , T（by value） 
  * 作为参数： 分为三类： Fn, FnMut, FnOnce， 对应ownership  的 &T, &mut T, T. Fn 可以无限次执行， FnMut 则要求 capture 变量的 mut 引用， FnOnce 则 只能执行一次
  *  疑问：
     1. 如何 区分  Fn(i64, i64) -> i64 与Fn(&String) -> i64
     2. FnOnce 是如何确定的， 有些函数，即便将变量 move 到了block中， 然而依然可以调用多次， 这些优势如何判断的？ 根据内部函数调用的 Fn 属性吗？ 比如mem::drop(p) 则 包含其调用的函数 则为 FnOnce?
     3. 如何断定 Cloosure 为 Fn or FnMut ?
     
     ``` rust
       struct Point {
           x: f64,
           y: f64,
       }

       // Implementation block, all `Point` methods go in here
       impl Point {
           // This is a static method
           // Static methods don't need to be called by an instance
           // These methods are generally used as constructors
           fn origin() -> Point {
               Point { x: 0.0, y: 0.0 }
           }

           // Another static method, taking two arguments:
           fn new(x: f64, y: f64) -> Point {
               Point { x: x, y: y }
           }
       }
       ```
    
#### Modules:  
* mod visibility: mod 的可见性， mod 默认只能在本mod 可见， 需要使用 pub 来对外可见， 定义其中的fn， struct 使用同样的规则， 因为mod 可以nest， 所以 存在pub(self) (等同于private) pub(super) 即让super mod 可见,  而pub(crate)则让crate 可见
* struct visibility: struct 中包含fn 以及 field，默认都为 对 所在 定义的mod可见， pub 则开放为对外部的 mod可见
* mod vs struct: struct 的控制比较弱， mod的控制则相对复杂， struct 可能并不需要如此复杂的规则吧
* 关键字 use： 我们可以使用 use mod::struct as another_struct 来 减少路径的拼写， 使用as 更可以 启用别名
* mod 的 使用类似于 Unix下的 目录 安排， super 代表 .. self 则代表 本mod


#### Attributes: 可以用来作什么？ 1） 条件编译， 2）设定crate 属性， 3）关闭 warning 4)启用编译器特性(maros etc) 5） link to a foreign library 6) 设定unit test 
* 形式： 当应用到 整个crate： #![crate_attribute], 应用到module 或者 item ： #[item_attribute]
* 还可以接受参数： 1) #[attribute = "value"] 2) #[attribute(key = "value")] 3) #[attribute(value)]
* 示例：
  * #[allow(dead_code)]： 关闭rust 关于没有调用函数的提示
  * #![crate_name = "rary"]
  * cfg(Configuration):  1） #[cfg(...)] 条件编译  2） cfg!(...) 在运行阶段的条件判断， 返回bool值

 ```rust
   // This function only gets compiled if the target OS is linux
   #[cfg(target_os = "linux")]
   fn are_you_on_linux() {
       println!("You are running linux!");
   }

   // And this function only gets compiled if the target OS is *not* linux
   #[cfg(not(target_os = "linux"))]
   fn are_you_on_linux() {
       println!("You are *not* running linux!");
   }

   fn main() {
       are_you_on_linux();

       println!("Are you sure?");
       if cfg!(target_os = "linux") {
           println!("Yes. It's definitely linux!");
       } else {
           println!("Yes. It's definitely *not* linux!");
       }
   }
   ```


### Rust 工具 Trait：
#### 总览：

| Trait                | Desc                                                                                                           |
|----------------------|----------------------------------------------------------------------------------------------------------------|
| Drop                 | 析构函数，当value 被drop时候，自动调用                                                                         |
| Sized                | Marker Trait， 标记 在编译期间能够确定 size的类型（与之对应的 为 动态 sized 比如 slice）                              |
| Clone                | 支持clone方法的类型                                                                                            |
| Copy                 | Marker Trait， 标记 支持可以 通过简单的 memory byte-for-bytes 复制 来支持 clone的类型                          |
| Deref & DerefMut     | 为 smart 指针 支持的类型                                                                                       |
| Default              | 存在default 数值的类型                                                                                         |
| AsRef & AsMut        | Conversion traits for borrowing one type of reference from another.                                            |
| Borrow and BorrowMut | Conversion traits, like AsRef/AsMut, but additionally guaranteeing consistent hashing, ordering, and equality. |
| From and Into        | Conversion traits for transforming one type of value into another.                                             |
| ToOwned              | Conversion trait for converting a reference to an owned value.                                                 |


#### Drop:
1. 定义

    ```rust
    trait Drop {
      fn drop(&mut self);
    }


    //一个简单的 实现 示例：

    struct Appellation {
        name: String,
        nicknames: Vec<String>,
    }

    impl Drop for Appellation {
        fn drop(&mut self) {
            print!("Dropping {}", self.name);
            if !self.nicknames.is_empty() {
                print!(" (AKA {})", self.nicknames.join(", "));
            }
            println!("");
        }
    }

    ```

2. Drop trait中 drop函数调用的时机：
  * value drop时候调用
  * 在drop 内部的field之前 进行调用，所以 在Appellation 的drop实现中  内部的field 依然可用。
  * 在 drop调用之后 ，依次调用内部的field的 drop 函数，来释放field内存占用。

3. 何时需要： Drop 一般很少需要自己进行实现。 只有当 自己定义的类型 **拥有rust 并不知道如何清理的资源时**，才需要实现Drop Trait。比如下面：

   ```rust

   struct FileDesc {
       fd: c_int,
   }

   impl Drop for FileDesc {
       fn drop(&mut self) {
           let _ = unsafe { libc::close(self.fd) };
       }
   }
   ```
4. Drop组成的结构将是 一个 树状的 调用链。链中 链接关系为 field， 每个节点不是自己实现了 Drop，就是链接节点实现了Drop
5. Drop 与 Copy Trait 存在 互斥关系，即  实现了Drop 则不能实现  Copy。
6. 标准库中存在一个 drop函数， fn drop<T>(_x: T) {} 函数 拿到 T的ownership，但并不做任何事情。

#### Sized： 该类型的数值 在内存中 总是 固定的大小，即： 在编译期间 即能够确定空间大小。几乎rust中所有的type都是 Sized，包括 enum 类型，即便Vec<T> 在heap中存在一个变长的内存，但是Vec 本身 是一个指向 内存地址的指针，包含 address， capacity， length 所以 Vec<T>是一个 Size type。

* rust中少有的 unsized类型， str, [T],  reference of a trait object. 
  * str类型 实例 "name" "big" 在 内存中的大小并不相同。
  * trait Object 比如 std::io::Write 因为实现了 Trait Write 的类型不同，其对应的 size 也不相同。
* Rust 无法 存储 unsized数值的变量 以及 作为参数传递，只能 通过 Sized  pointer来进行包装， 比如  &str， Box<Write>
* slice‘s pointer 与 Trait pointer 同样是一个 fat pointer

* Sized Trait是一个marker trait， 即 rust 编译器自动实现 Sized 类型， 我们不能够自己为类型实现 Sized， 即 rust 使用 该trait实现一种 类型标记作用
* 在函数参数中可以 添加 T: Sized 来限制 参数
* 因为 unsized类型 非常少，而限制非常多，所以 默认情况下 rust 自动为我们的 T placeholder 添加 Sized限制，如果我们不需要 该约束 则 使用过 T： ?Sized 来取消约束，即： 类型 不需要是 Sized。 比如 Box<str>

* 示例： Box 与 RcBox 并不要求 模版参数 T 为Sized

   ```rust
   pub struct Box<
       T: ?Sized,
       #[unstable(feature = "allocator_api", issue = "32838")] A: Allocator = Global,
   >(Unique<T>, A);



   struct RcBox<T: ?Sized> {
     ref_count: usize,
     value: T,
   }
   ```

####  Clone： 复制一个  独立 的 self 并返回它， Self 不应该是unsized， clone一般是 比较耗费资源的操作，所以rust并没有为每个 类型实现它，而是 由我们自己来实现，但是Rc<T>  与 Arc<T>是一个例外，其clone只是简单的增加计数而已。
* 定义：
    ```rust
    trait Clone: Sized {
      fn clone(&self) -> Self;
      fn clone_from(&mut self, source: &Self) {
        *self = source.clone()
      }
    }
    ```
* Clone trait提供的 clone_from(&mut self, source: &Self) 函数， 该函数 通过 修改self， 复制source中的内容，来实现clone。应该总是在能够使用clone_from的时候 使用它，如下情况，将减少heap内存的分配和释放。

    ```rust
    let mut s = String::from("source");
    let mut target = String::from("target");
    // 第一种写法, 将造成 target 的 复制操作， 然后是 赋值操作， 将导致 s原先 heap的释放 与target的 heap复制
    s = target.clone();

    // 第二种写法， 该种写法， 在符合条件下，将不会在 heap重新分配内存， 也不需要s释放原先的heap内存，在s原地修改
    s.clone_from(&target)
    ```

* derive: rust为我们提供了一种简单的 clone 各个field 的方式。添加 #[derive(Clone)]  即可。
* 一些没有实现了Clone trait的type， std::sync::Mutex, std::fs::File

#### Copy: 在表达式 A = B中， 将 B赋值给A时 不是转移B的ownership给A，而是 直接 copy 一个 B出来 赋值给A，该种情况即是 实现了 Copy Trait， 比如基本的简单类型。 i32， i64
* 定义 以及 实现：
    ```rust
    trait Copy: Clone { }
    impl Copy for MyType { }
    ```

* Copy 同样是一个 Marker trait， 但 rust 允许我们为自己的类型实现 Copy， 因为 在Copy 过程中，我们需要Clone B， 所以 Clone Trait 是该 trait 的super trait。 同样rust 为我们实现了一个默认的实现 #[derive(Copy)] 通常情况下  Copy 存在的时候 Clone也需要存在 即 常见的形式为 #[derive(Copy, Clone)]

* 注意问题：
  * 与 Drop 的关系： 任何实现 Drop 特性的类型都不能是 Copy。 Rust 假定如果一个类型需要特殊的清理代码，它也必须需要特殊的复制代码，因此不能被复制。

  * 该Trait的实现 将 方便代码的编写，因为 clone() 隐式的实现， 但是需要注意应该谨慎的为 类型实现 copy， 导致大量clone 导致的 资源损耗。

####  Deref and DerefMut: rust 将自动尝试使用 两个trait提供的方法 将 类型转换到 需要的类型。即： 如果 deref 能够防止类型的不匹配，那么rust将自动帮我们 插入代码。
* 定义： 
  ```rust

  trait Deref {
  type Target: ?Sized;
    fn deref(&self) -> &Self::Target;
  }
  trait DerefMut: Deref {
    fn deref_mut(&mut self) -> &mut Self::Target;
  }

  ```

* std库中实例：
   * r: Rc<String> 我们可以 使用 r.find（‘？’） 而不是 (*r).find('?') 因为 Rc<T> 实现了 Deref<Target=T>, Rc<String> 可以转化为 String

   * r: String, 我们可以直接使用 str 的split_at 方法， 因为 String 实现了 Deref<Target=str>
   * r: Vec<T>, 同样可以使用 [T]的方法，因为 Vec<T> 实现了 Deref<Target=[T]>
* 注意情况：
  * rust 可能会插入 多个转换代码 如果需要的话，比如 &Rc<String> deref 到 &String, deref 到 &str， &str中拥有 split_at 方法。



  * rust为并不会为  模版方法 尝试 进行 类型的自动deref 转换。
* 使用场景： 
  * **该Trait 是为了实现smart pointer 而设计的**。比如 Box， Rc， Arc ， 一些 能够被视为 拥有 reference的类型（比如 String 与str 以及 Vec\<T\> 与 [T] ） 不应该为了 使用Target 的方法 而为类型实现 Deref\<Target\>

####  Default：为 有明显的理由 提供默认值的 type实现。
* 定义：
  ```rust
  trait Default {
    fn default() -> Self;
  }

  ```
* std库中的实例：
  * String 也提供了 Default 的实现。
  * 所有的rust collection 类型， Vec HashMap， BinaryHeap ... 都提供了 Default 的实现，default 函数提供空的 collection。 这对于 提供一个方法，有用户来指定 返回的collection 类型有很好的帮助，比如

     ```rust
     use std::collections::HashSet;
     let squares = [4, 9, 16, 25, 36, 49, 64];
     let (powers_of_two, impure): (HashSet<i32>, HashSet<i32>) = squares.iter().partition(|&n| n & (n-1) == 0);
     assert_eq!(powers_of_two.len(), 3);
     assert_eq!(impure.len(), 4);

     ```
* 使用场景： 1) 上述 std collection， 2） 为一个拥有大量field的 struct 提供默认数值。
* **derive**:  如果一个struct中各个字段实现了 Default， 则我们使用  #[derive(Default)] 来提供 默认的Default 实现。

#### AsRef and AsMut:  为类型转换提供了除了 Deref 之外的另外一种方法。 不同于 Deref 能够提供 rust 内部的代码deref()调用的插入， AsRef 仅仅是 提供了 as_ref的 trait
* 定义：
    ```rust
    trait AsRef<T: ?Sized> {
      fn as_ref(&self) -> &T;
    }
    trait AsMut<T: ?Sized> {
      fn as_mut(&mut self) -> &mut T;
    }

    ```
* std库中的实例：

    ```rust
    fn open<P: AsRef<Path>>(path: P) -> Result<File>

    //因为 String， 以及 str  实现了 AsRef<Path> 所以， 我们可以进行如下的 写法：

    let dot_emacs = std::fs::File::open("/home/jimb/.emacs")?;

    // 这里面需要注意，因为 实现了 AsRef<Path> 的是 str， 并不是 &str， 那么为什么上面的代码能够通过编译呢？ 这里面 rust 并不会进行类型转换，因为 rust并不提供 对于 变量约束 模版变量参数 的类型转换。
    // 因为 存在 如下 blanket implement: 即： any type T impl AsRef<U> then &T impl AsRef<U> also.

    impl<'a, T, U> AsRef<U> for &'a T
    where
        T: AsRef<U>,
        T: ?Sized,
        U: ?Sized,
    {
        fn as_ref(&self) -> &U {
            (*self).as_ref()
        }
    }

    ```
* 使用场景： 该Trait 显得简单，但是依然非常重要， 为 减少更具体类型 提供了帮助， 在 定义更具体的AsFoo trait之前，应该考虑是否可以 让现有类型实现 AsRef\<Foo\>



####  Borrow and BorrowMut: 与 AsRef 相似，如果type impl Borrow<T> 则 能够从type borrow() 出一个 &T, 区别在于 Borrow 添加了一些限制，要求 type 与 &T 能够hash 的数值一样 才行。（rust 并没有强制要求 ，而是 以文档方式要求） 这 为 Hash table 与 tree 的key 比较 提供了方便。

* 定义：

```rust
trait Borrow<Borrowed: ?Sized> {
  fn borrow(&self) -> &Borrowed;
}
```

* std 库中的实例： HashMap

     ```rust
     // 考虑我们有一个 HashMap<String, i32> 的collection，
     // 我们需要获取 'name' 对应的数值，
     // 考虑Hashmap的get 实现

     impl HashMap<K, V> where K: Eq + Hash {
        fn get(&self, key: K) -> Option<&V> { ... }
     }
     // 因为 get方法签名， key： K， 这要求我们 传递 一个String 类型的数值，来寻找 key对应的 value
     // 这里的问题在于 我们每次需要查询 都需要构建一个 String（进行内存分配）将是十分浪费的方法。

     impl HashMap<K, V> where K: Eq + Hash {
        fn get(&self, key: &K) -> Option<&V> { ... }
     }

     // 对比上面要好一点，方法要求我们传递 &String类型，但是 我们需要查找一个 constant string对应的数值时候，
     //我们需要这样写, 该种写法 同样造成 name 转换为string，即分配内存空间。

     hashtable.get(&"name".to_string());


     impl HashMap<K, V> where K: Eq + Hash {
       fn get<Q: ?Sized>(&self, key: &Q) -> Option<&V>
          where K: Borrow<Q>,
                Q:Eq+Hash
          {...}

     }

     // 最终的解法，我们要求 参数 Q 是一个能够 从 HashMap K中 borrow出来的 类型，并且 Q impl  Eq + Hash， 所以我们 将从HashMap 中的K 调用borrow 方法，来与 key: &Q 进行比较，来寻找到 对应的value
     // 因为 String impl  Borrow<str> Borrow<String>, 所以我们 可以 传递 &str, &String 给 get 方法
     ```

* 使用场景：
  * std 库中所有的collection 使用 Borrow 来进行 进行 lookup
  * std 库提供了 blanket implementation ， impl T for Borrow<T> ， impl &mut T for Borrow<T>  所以 在HashMap<K, V> 中，我们可以 .get(&K) 来进行查找。



#### From and Into:  消耗 type A 的ownership 返回 type B (对比 AsRef Borrow trait 他们并不使用 调用者 的ownership，只是返回一个 reference)

* 定义：

     ```rust
     trait Into<T>: Sized {
       fn into(self) -> T;
     }
     trait From<T>: Sized {
       fn from(T) -> Self;
     }

     ```

*  std 自动实现了 A impl From \<B\> 则  B impl Into\<A\>
*  使用场景： 两个Trait 虽然扮演了一个角色（类型转换）， 但用法不同，
   * Into： 一般用作 function 参数，来将  function 参数变得更加灵活。
   * From： 一般用作通用的构造函数， 从多种 类型中产生一个 type。
   如下是示例：

     ```rust
     use std::net::Ipv4Addr;
     fn ping<A>(address: A) -> std::io::Result<bool>
      where A: Into<Ipv4Addr>
      {
        let ipv4_address = address.into(); ...
      }


     let addr1 = Ipv4Addr::from([66, 146, 219, 98]);
     let addr2 = Ipv4Addr::from(0xd076eb94_u32);
     ```

* 限制； 1) 与 AsRef 的转换相比 From Into 可能会比较“重”， 即AsRef 的转换是相对廉价的。From Into的转换可能涉及到 内存分配，copy等操作 ， 比如 String impl From<&str> 需要 copy str 的内容到 string中。 2） From & Into 转换 不允许失败

#### ToOwned： 存在目的是： 为了解决 clone trait 的限制，即 A impl Clone， 则 &A.clone() 将返回 一个A， 但是 对于 &str .clone() 返回 str 则不能接受（因为str 为unsized 类型）， 所以 创建了 ToOwned trait：

* 定义： 即 任何实现了 A Borrow \<B\> 的，则能够 B.to_owned() A

     ```rust
     trait ToOwned {
        type Owned: Borrow<Self>;
        fn to_owned(&self) -> Self::Owned;
     }
     ```
* std库中的实例： str impl ToOwned<Owned=String> 则 str.to_owned() String


#### Cow: Borrow and ToOwned at Work

* 定义： Cow 是 clone on write的缩写。

     ```rust
     enum Cow<'a, B: ?Sized + 'a>
       where B: ToOwned
     {
       Borrowed(&'a B),
       Owned(<B as ToOwned>::Owned), //这里的语法 有些奇怪 
     }

     // cow存在两个 enum类型，Borrowed为 一个reference， Owned 则保存 一个 实际数值 ，该数值 应该是从 &'B 中 to_owned出来的，
     //下面为一个示例：

     use std::borrow::Cow;
     use std::path::PathBuf;
     fn describe(error: &Error) -> Cow<'static, str> {
         match *error {
             Error::OutOfMemory => "out of memory".into(),
             Error::StackOverflow => "stack overflow".into(),
             Error::MachineOnFire => "machine on fire".into(),
             Error::Unfathomable => "machine bewildered".into(),
             Error::FileNotFound(ref path) => format!("file not found: {}", path.display()).into(),
         }
     }

     println!("Disaster has struck: {}", describe(&error));

     let mut log: Vec<String> = Vec::new();
     log.push(describe(&error).into_owned());

     // Cow impl  Into 所以 在 describe match  arm 中可以使用 str.into() 方法返回 Cow
     // Cow的存在 可以让 println 中 继续 保持 str， 而在  Vec<String>.push中 则 into_owned()返回 String，即需要的时候才会分配内存

     ```


