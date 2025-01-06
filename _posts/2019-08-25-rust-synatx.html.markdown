---
title: rust synatx
date: 2019-08-25
tags: rust
---

### Ownership Rules

  * Each value in Rust has a variable that’s called its owner.
  * There can only be one owner at a time.
  * When the owner goes out of scope, the value will be dropped.

  Scope : { //scope }
  内存申请 let s = String::from("xx");
  内存释放： s 超出scop，变为不可用。rust自动添加drop调动代码， 归还内存
  所有权规则：

  堆、栈 中变量的其他 赋值方式：
  * stack-only: copy
  
    ```rust
    let x = 5;
    let y = x; // x, y  都可用， 因为x 为栈上分配， 对于内存方式为 copy， 不影响所有权
    ```
  * Heap: clone、所有权转移
  
  ```rust
  let s1 = String::from("hello");
  let s2 = s1.clone();// s2 copy s1的内存，s1 依然可用
  ```
  ```rust
  let s1 = String::from("xx");
  let s2 = s1; // s1 不在可用， s2为 字符串的 owner, 这里只是转移 指向 string的指针，而非 copy string
  ```
  
### Function 调用：
  * Function 调用参数： 跟 赋值 一样的所有权  转移一样
  * Function 返回参数： 堆上的内存，作为返回值的时候， 两种情况： 1. 转移所有权 到 函数调用者 2. drop掉 （因为超过 函数中的作用域 scope）

### 引用： Function 调用： 每次都需要转移所有权，在将所有权转回到调用者。非常麻烦， 所以设计了 引用 
**指向变量的指针， 并不具有 ownership， 所以drop并不会，释放内存，使得引用的变量不可用。在Function 中使用 非常合适，因为不需要ownership传递回去， 因为根本没有ownership的转移**

  *  可变 mut 引用: 可以 改变 引用指向的内容。
  *  不可变引用
  *  引用 规则:
    1. 任何时候，只有一个 可更改引用，或者 同时多个不可变应用
    2. 引用需要总是有效的。即： 引用的scope应该小于变量的scope

  ```rust
  fn no_dangle() -> &String {
    let s = String::from("hello");
    &s
  }
  // 函数返回， s 会drop掉， 所以会造成空指针 null reference
  ```
  错误的函数，s在 函数中分配内存，但是只返回引用，**引用变量作用于大于 指向的变量作用域**
  *slice: 同refrence， 引用一个连续的collection，但是没有ownership。 这里用来防止，在同样的作用域使用 mut refrence， 或者 mut 调用 (因为不能同时存在  mut 引用，和 非mut引用。所以自动的添加一份检查)*

###  Generic define & syntax
  定义 函数参数签名（告诉编译器 参数类型）, 形式如下:
  
  ```rust
  fn largest<T>(list:&[T]) -> T {
  }

  struct Point<T> {
      x: T,
      y: T,
  }
  // x, y 是同一类型， 也可以写成不一样的类型
  struct Point<T, U> {
      x: T,
      y: U,
  }

  impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
  } 
  // 这里为什么显得如此怪异的原因， 在于 我们可以写出 impl Point<String> 来定定制 T=String 时候特有的方法定义。所以我们需要写成如此 impl<T> 来区分于 impl Point<String> , 声明 T 代表是一个place holder
  ```
**这里为什么显得如此怪异的原因， 在于 我们可以写出 impl Point<String> 来定定制 T=String 时候特有的方法定义。所以我们需要写成如此 impl<T> 来区分于 impl Point<String> , 声明 T 代表是一个place holder**
并不会牺牲性能，没有runtime的耗时， 在编译阶段， rust 会填充 placeholder, 来完成， 不同类型的定义。

### Trait: Defining Shared Behavior
 * impl 定义 及其 实现
 
  ```rust
    pub trait Summary {
        fn summarize(&self)-> String;
        fn speak(&self) -> {
            self.summarize();
            println!("speak");
        }
    }

    pub struct NewsArticle {
    }

    impl Summary for NewsArticle {
        fn summarize(&self) -> String {
            println!("---------");
        }
    }
  ```
  * Trait 类似于Interface， 共享 行为（函数） 定义，还可以 实现 类似 模板调用的方法。
  * Trait 当 Function 参数

  ```rust
    pub fn notify(item: impl Summary) {
        println!("---------");
    }
    // Trait Bound syntax
    pub fn notify<T: Summary>(item: T) {
          println!("---------");
    }
    // multiple Trait Bound
    pub fn notify<T: Summary + Display>(item: T) {
          println!("---------");
    }
    // use where
    pub fn some_function<T, U>(one: T, two: U) {
        where T: Display + Clone,
              U: Clone + Debug
        println!("---------");
    }
  ```

  * Trait 当做Function 的return type, 但是 存在一些限制： 主要有， 返回值不能是不同类型， 而只能是一个确定的类型 impl trait（例如{} 中 通过if else 返回一个完全不同类型，却实现了同样的Trait 的类型）
  * Trait with Generic 可以 约束 impl Generic 的 类型为实现了 Trait 的类型。

  ```rust
  impl<T: Display + PartialOrd> Point<T> {
      fn only_some(&self) {
      }// 只有实现了 Display & PartialOrd trait的 Point<_> 类型，才会有 only_some 方法
  }
  ```
 * Advanced Traits: Traits with placeholder
 
  ```rust
    pub trait Iterator {
      type Item;

       fn next(&mut self) -> Option<Self::Item>;
    }
    impl Iterator for Counter {
        type Item = u32;

        fn next(&mut self) -> Option<Self::Item> {
        }
    }
  ```
  为什么不适用这样的实现呢？

  ```rust
  pub trait Iterator<T> {
      fn next(&mut self) -> Option<T>;
  }
  ```
  原因在于 如果采用第二种实现， 我们需要 写成这样

  ```rust
  impl Iterator<String> for Counter {

  }
  impl Iterator<i32> for Counter {

  }
  ```
 这里面存在多种实现方式。 更重要的是，我们在调用next时候，需要显式的指定 .next::<Iterator<String>> 来 指导 rust使用哪个Iterator<T> for Counter 的代码实现。所以第一种更可取
但是确实存在 Generic 与 Trait 结合的例子：

```rust

trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}

// 常用的声明可以如下:
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}
// 然而我们依然可以这样，指定 RHS Generic参数， 来实现， Millimeters + Meters 的函数调用实现, 只不过，placeholder 并没有作为返回值，所有，可以直接+ 而不需要显示的，指定 + 之后的返回数值类型
struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

多个Trait 出现同样函数名字的情况：

```rust

trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

//两个 Trait 拥有同样的函数名称，不同的函数实现。那如何在调用时候，决策函数调用实体呢？
struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
// 下面，person.fly 默认调用 Human 自己的实现， 如果需要 显式的调用 Pilot::fly 则需要， 如下格式
fn main() {
    let person = Human;
    person.fly();
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
// 因为其为实例方法， 存在&self， 如果不存在呢？ 下面示例:

trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());// Ok
    println!("A baby dog is called a {}", Animal::baby_name()); // Error, 下面是正解
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}


```

SuperTriat： 超级 Trait， 依赖于一个Trait的实现， 示例如下：
```rust

use std::fmt;
// 声明语法如下： trait SuperTrait: Trait {}
trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        println!("output is --- {}", output);
    }
}
// 我们只需要为point 实现 Display， 即可 拥有 outline_print 方法
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
impl OutlinePrint for Point {}
```
NewType: 因为 impl Trait for Type, 中的type需要在本地的crate，而不是 引用库 中的Type。 所以 可以通过Newtype类来包装 Type，实现一些 Trait. 
这里面包含另外一些需要东西： 如何让 NewType， 伪装成Type？
实现 Deref Trait。

  ```rust
  use std::fmt;

  struct Wrapper(Vec<String>); // 新的type 类似于下边的
  struct Color(i32, i32, i32);
  struct Point(i32, i32, i32);

  impl fmt::Display for Wrapper {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          write!(f, "[{}]", self.0.join(", "))
      }
  }

  fn main() {
      let w = Wrapper(vec![String::from("hello"), String::from("world")]);
      println!("w = {}", w);
  }
  ```

### Generic 生命周期 syntax： 用于区分函数中 参数生命周期， 对比 参数、返回值 等 生命周期之间的关系。 确保 参数传递生命周期符合 函数声明. 生命周期 需要关联 参数与返回值，才会有效果，只有参数的生命周期没有用处

  ```rust
  fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
  }
  // error, rust并不能知道 函数返回值， 是x还是y， 无法检查生命周期

  fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
      if x.len() > y.len() {
          x
      } else {
          y
      }
  }
  // 要求 函数返回值，应该小于等于 参数， x y的生命周期, 所以下面的函数调用是可以pass的
  fn main() {
      let string1 = String::from("long string is long");

      {
          let string2 = String::from("xyz");
          let result = longest(string1.as_str(), string2.as_str());
          println!("The longest string is {}", result);
      }
  }

  // 而这个， 则是编译失败的，因为 返回值的生命周期 大于其中参数 y 的生命周期， 会导致 dangling refrence, 比如， result指向 y, 而y 在 内部的scope中已经销毁了
  fn main() {
      let string1 = String::from("long string is long");
      let result;
      {
          let string2 = String::from("xyz");
          result = longest(string1.as_str(), string2.as_str());
      }
      println!("The longest string is {}", result);
  }
  // 还可以这样， 总是 返回其中的一个值
  fn longest<'a>(x: &'a str, y: &str) -> &'a str {
      x
  }
  ```
  * Struct 的生命周期： struct 保持的refrence 的生命周期 与struct 生命周期关联。struct 不应该 长于 内部变量的refrence。
  ```rust
  struct ImportantExcerpt<'a> {
    part: &'a str,
  }

    fn main() {
        let novel = String::from("Call me Ishmael. Some years ago...");
        let first_sentence = novel.split('.')
            .next()
            .expect("Could not find a '.'");
        let i = ImportantExcerpt { part: first_sentence };
  }// 其中 i 的生命从周期不应该长于 novel
  // impl Struct Generic 方法时候的 声明语法， 同 impl Generic。 其中声明方法时候，需要不要 生命周期 声明，需要看，是否与struct field 、 返回值 相关
 impl<'a> ImportantExcerpt<'a> {
     fn announce_and_return_part(&self, announcement: &str) -> &str {
         println!("Attention please: {}", announcement);
         self.part
     }
 }
  ```
  Static lifetime， 静态生命周期，表明变量， 将贯穿于整个program, 将直接保存于， 代码的二进制中。



### Closures:
 *  /|x/| {}
 *  FnOnce: 获取参数的ownership
 *  FnMut： 获取参数的 mut 引用
 *  Fn： 获取参数的 非mut引用
 *  以上三种 为 Trait， 可以声明类型为 FN(i32) -> i32
 *  function as paramas： Function Pointer(fn a Type diff with Fn) ， fn 类型，实现了， Fn, FnMut, FnOnce 的实现， 即 impl Fn, FnMut, FnOnce for fn {....} 所以，可以传递 fn 到 接受 closures的 函数中。 还可以声明接受fn类型的 函数

 ```rust
  fn add_one(x: i32) -> i32 {
      x + 1
  }

  fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
      f(arg) + f(arg)
  }

  fn main() {
      let answer = do_twice(add_one, 5);

      println!("The answer is: {}", answer);
  }

  // 可以 传递参数fn 类型
  let list_of_numbers = vec![1, 2, 3];
  let list_of_strings: Vec<String> = list_of_numbers
      .iter()
      .map(ToString::to_string)
      .collect();
 ```

## macros:
### macro_rules:
#### rust tokens 分类：
```text
  Identifiers: foo, Bambous, self, we_can_dance, LaCaravane, …
  Integers: 42, 72u32, 0_______0, …
  Keywords: _, fn, self, match, yield, macro, …
  Lifetimes: 'a, 'b, 'a_rare_long_lifetime_name, …
  Strings: "", "Leicester", r##"venezuelan beaver"##, …
  Symbols: [, :, ::, ->, @, <-, …
```
* c语言 需要特殊的 “macro层面” 即 预处理，而rust  macro处理时机 在 编译器 将 token 转换为  AST 之后进行（Abstract Syntax Tree）
* 编译过程 文本 -> token tree  -> AST  基本所有的 token都是叶子节点 ，只有 （...） [...] {...} 包含一组节点，是 树节点。

```text
表达式： a + b + (c + d[0]) + e
token tree
«a» «+» «b» «+» «(   )» «+» «e»
          ╭────────┴──────────╮
           «c» «+» «d» «[   ]»
                        ╭─┴─╮
                        «0»

AST
              ┌─────────┐
              │ BinOp   │
              │ op: Add │
            ┌╴│ lhs: ◌  │
┌─────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
│ Var     │╶┘ └─────────┘ └╴│ BinOp   │
│ name: a │                 │ op: Add │
└─────────┘               ┌╴│ lhs: ◌  │
              ┌─────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
              │ Var     │╶┘ └─────────┘ └╴│ BinOp   │
              │ name: b │                 │ op: Add │
              └─────────┘               ┌╴│ lhs: ◌  │
                            ┌─────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
                            │ BinOp   │╶┘ └─────────┘ └╴│ Var     │
                            │ op: Add │                 │ name: e │
                          ┌╴│ lhs: ◌  │                 └─────────┘
              ┌─────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
              │ Var     │╶┘ └─────────┘ └╴│ Index   │
              │ name: c │               ┌╴│ arr: ◌  │
              └─────────┘   ┌─────────┐ │ │ ind: ◌  │╶┐ ┌─────────┐
                            │ Var     │╶┘ └─────────┘ └╴│ LitInt  │
                            │ name: d │                 │ val: 0  │
                            └─────────┘                 └─────────┘


```


#### macro rule 的语法规则: https://danielkeep.github.io/tlborm/book/mbe-macro-rules.html

```text
匹配 语法如下：

macro_rules! four {
  (pattern) => {

  };
  (pattern1) => {

  };
}



捕获： $name:kind

kind类型如下：

  item: an item, like a function, struct, module, etc. 
  block: a block (i.e. a block of statements and/or an expression, surrounded by braces)
  stmt: a statement
  pat: a pattern
  expr: an expression
  ty: a type
  ident: an identifier
  path: a path (e.g. foo, ::std::mem::replace, transmute::<_, int>, …)
  meta: a meta item; the things that go inside #[...] and #![...] attributes
  tt: a single token tree

示例如下：

macro_rules! times_five {
    ($e:expr) => {5 * $e};
}


重复：语法 $(...) sep rep.

$: $ 符号
(...)： 被重复内容： 可以包含 token tree， captures（捕获） 或者 其他重复（递归）
seq 为 可选的split token，一般可选 , . ;
rep 为 重复控制，可选有 + *

示例如下：
macro_rules! vec_strs {
    (
        // Start a repetition:
        $(
            // Each repeat must contain an expression...
            $element:expr
        )
        // ...separated by commas...
        ,
        // ...zero or more times.
        *
    ) => {
        // Enclose the expansion in a block so that we can use
        // multiple statements.
        {
            let mut v = Vec::new();

            // Start a repetition:
            $(
                // Each repeat will contain the following statement, with
                // $element replaced with the corresponding expression.
                v.push(format!("{}", $element));
            )*

            v
        }
    };
}
```


#### 捕获与扩展 细节：

1. 匹配项 应该从 最具体 到最广泛的匹配规则， 因为 一旦 捕获表达式 开始消费 tokens，将不能后退 或者在匹配其他的 选项
2. macro 之间的内容传递: 即 第一个macro接受的 为token，如果 第一个macro将捕获传递给其他的macro 则为 AST形式。示例如下：

    ```text

    macro_rules! capture_expr_then_stringify {
        ($e:expr) => {
            stringify!($e)
        };
    }

    fn main() {
        println!("{:?}", stringify!(dummy(2 * (1 + (3)))));
        println!("{:?}", capture_expr_then_stringify!(dummy(2 * (1 + (3)))));
    }

    output like this:

    "dummy ( 2 * ( 1 + ( 3 ) ) )"
    "dummy(2 * (1 + (3)))"

    第一个 macro stringify 接受的形式 如下： tokens

    «dummy» «(   )»
       ╭───────┴───────╮
        «2» «*» «(   )»
           ╭───────┴───────╮
            «1» «+» «(   )»
                     ╭─┴─╮
                      «3»

    第二个 macro stringify 接受的形式如下： AST

    « »
     │ ┌─────────────┐
     └╴│ Call        │
       │ fn: dummy   │   ┌─────────┐
       │ args: ◌     │╶─╴│ BinOp   │
       └─────────────┘   │ op: Mul │
                       ┌╴│ lhs: ◌  │
            ┌────────┐ │ │ rhs: ◌  │╶┐ ┌─────────┐
            │ LitInt │╶┘ └─────────┘ └╴│ BinOp   │
            │ val: 2 │                 │ op: Add │
            └────────┘               ┌╴│ lhs: ◌  │
                          ┌────────┐ │ │ rhs: ◌  │╶┐ ┌────────┐
                          │ LitInt │╶┘ └─────────┘ └╴│ LitInt │
                          │ val: 1 │                 │ val: 3 │
                          └────────┘                 └────────┘

    ```

3. macro 将  input 从tokens转到 AST，将导致 input再也不能被 macro 表达式匹配，如下：

     ```rust
     macro_rules! capture_then_what_is {
         (#[$m:meta]) => {what_is!(#[$m])};
     }

     macro_rules! what_is {
         (#[no_mangle]) => {"no_mangle attribute"};
         (#[inline]) => {"inline attribute"};
         ($($tts:tt)*) => {concat!("something else (", stringify!($($tts)*), ")")};
     }

     fn main() {
         println!(
             "{}\n{}\n{}\n{}",
             what_is!(#[no_mangle]),
             what_is!(#[inline]),
             capture_then_what_is!(#[no_mangle]),
             capture_then_what_is!(#[inline]),
         );
     }

     // The output is:  即  macro capture_then_what_is 将input转化 传递给 macro what_is 之后 ，再也 不能被 what_is 中的 pattern 匹配

     no_mangle attribute
     inline attribute
     something else (# [ no_mangle ])
     something else (# [ inline ])

     ```
4. 避免此类情况的唯一方法: 使用 tt 与 ident 进行匹配，使用任何其他的匹配 获得的捕获 不能够传递给其他的 macro
5. 卫生： 默认情况下 macro都是卫生macro，除非我们需要：如下：

    ```rust
    // 卫生宏
    macro_rules! using_a {
        ($e:expr) => {
            {
                let a = 42;
                $e
            }
        }
    }
    // 宏调用是错误的，将导致编译错误，即a没有定义
    let four = using_a!(a / 10);

    // 非卫生宏，捕获 环境中的a
    macro_rules! using_a {
        ($a:ident, $e:expr) => {
            {
                let $a = 42;
                $e
            }
        }
    }

    let four = using_a!(a, a / 10);
    ```


6. self: 标识符或关键字。self在代码中是一个关键字，但是macro中可以成为一个标识符，使用macro在struct中定义方法：

    ```rust
    //依然需要借助  非卫生宏 来污染 空间，捕获 self
    macro_rules! double_method {
        ($self_:ident, $body:expr) => {
            fn double(mut $self_) -> Dummy {
                $body
            }
        };
    }

    struct Dummy(i32);

    impl Dummy {
        double_method! {self, {
            self.0 *= 2;
            self
        }}
    }

    ```

7. 一个巧妙的 macro之间传递内容的方法： 通过 ident捕获callback名称，通过添加 ! 来完成，对另一个macro的调用

    ```rust
    macro_rules! call_with_ident {
        ($c:ident($i:ident)) => {$c!($i)};
    }
    ```

6. macro的 作用域：
    * macro 在定义之后 的代码中 以及 sub-module 中可见。
    * macro 需要 使用 #[macro_use] attribute 才能被 export 出来。 
    * macro不同于 函数调用， 导致了一些 macro之间相互依赖的关系 与 可见性 关联起来 导致的复杂情景。  但基本上 可以按照 将macro层层 展开来，每层 macro依然 符合 前面两条规则 即： macro使用在 macro定义之后， macro 可见。
  
#### macro 中的模式：
1. callback: 因为macro之间传递参数的 限制，导致的一种间接调用形式。即：使用tt对callback 以及其参数 进行匹配，然后 拼接成  macro调用形式。

    ```rust
    //因为 macro 接受参数问题 导致的问题 示例：

    acro_rules! call_with_larch {
        ($callback:ident) => { $callback!(larch) };
    }

    macro_rules! expand_to_larch {
        () => { larch };
    }

    macro_rules! recognise_tree {
        (larch) => { println!("#1, the Larch.") };
        (redwood) => { println!("#2, the Mighty Redwood.") };
        (fir) => { println!("#3, the Fir.") };
        (chestnut) => { println!("#4, the Horse Chestnut.") };
        (pine) => { println!("#5, the Scots Pine.") };
        ($($other:tt)*) => { println!("I don't know; some kind of birch maybe?") };
    }

    fn main() {
        recognise_tree!(expand_to_larch!());
        call_with_larch!(recognise_tree);
    }

    // 展开形式 与 输出

    recognise_tree! { expand_to_larch ! (  ) }
    println! { "I don't know; some kind of birch maybe?" }
    // ...

    call_with_larch! { recognise_tree }
    recognise_tree! { larch }
    println! { "#1, the Larch." }

    //callback形式 解决：

    macro_rules! callback {
        ($callback:ident($($args:tt)*)) => { //注意这里 使用tt 不仅匹配了callback name 还匹配了参数 ，保留了参数token的形式 
            $callback!($($args)*)
        };
    }

    fn main() {
        callback!(callback(println("Yes, this *was* unnecessary.")));
    }

    ```

2. tt 递归匹配器，该模式 每次处理一个递归项目，然后调用自身 macro继续 处理后续 递归项目。需要注意macro的 递归次数限制。macro recursion limit 示例：

    ```rust
    macro_rules! mixed_rules {
        () => {};
        (trace $name:ident; $($tail:tt)*) => {
            {
                println!(concat!(stringify!($name), " = {:?}"), $name);
                mixed_rules!($($tail)*);
            }
        };
        (trace $name:ident = $init:expr; $($tail:tt)*) => {
            {
                let $name = $init;
                println!(concat!(stringify!($name), " = {:?}"), $name);
                mixed_rules!($($tail)*);
            }
        };
    }
    ```

3. 为了隐藏 内部的macro，因为macro 可见性问题 导致 简单的macro_use 可能与其他的crate 产生名称冲突。所以：  使用一个 包含 所有 pattern规则 的module 进行封装：

    ```rust
    #[macro_export]
    macro_rules! foo {
        (@as_expr $e:expr) => {$e};

        ($($tts:tt)*) => {
            foo!(@as_expr $($tts)*)
        };
    }
    //这样将 将as_expr 包装在 mod  foo 内部

    // macro 封装的通用形式
    macro_rules! crate_name_util {
        (@as_expr $e:expr) => {$e};
        (@as_item $i:item) => {$i};
        (@count_tts) => {0usize};
        // ...
    }
    ```

4. tt 模式常用方式，push down计算：

    > 如何实现 接受 形式 如 let strings: [String; 3] = init_array![String::from("hi!"); 3]; 的macro 实现。
    

    ```rust
    //一个明显的、直接的 但实现错误的 递归实现 方法。

    macro_rules! init_array {
        (@accum 0, $_e:expr) => {/* empty */};
        (@accum 1, $e:expr) => {$e};
        (@accum 2, $e:expr) => {$e, init_array!(@accum 1, $e)};
        (@accum 3, $e:expr) => {$e, init_array!(@accum 2, $e)};
        [$e:expr; $n:tt] => {
            {
                let e = $e;
                [init_array!(@accum $n, e)]
            }
        };
    }


    //该模式 编译不通过的原因 在于， 在 每一步骤中 构建了一个不完全的 {String::from("hi"), init_array!(@accum 1, $e)} 的表达式。因为 init_array! 并非一个完整的 表达式。所以 该种方法 并不正确。并导致 编译不通过。


    //下面是 push-down accumulation  方式 来实现：

    macro_rules! init_array {
        (@accum (0, $_e:expr) -> ($($body:tt)*))
            => {init_array!(@as_expr [$($body)*])};
        (@accum (1, $e:expr) -> ($($body:tt)*))
            => {init_array!(@accum (0, $e) -> ($($body)* $e,))};
        (@accum (2, $e:expr) -> ($($body:tt)*))
            => {init_array!(@accum (1, $e) -> ($($body)* $e,))};
        (@accum (3, $e:expr) -> ($($body:tt)*))
            => {init_array!(@accum (2, $e) -> ($($body)* $e,))};
        (@as_expr $e:expr) => {$e};
        [$e:expr; $n:tt] => {
            {
                let e = $e;
                init_array!(@accum ($n, e.clone()) -> ())
            }
        };
    }

    let strings: [String; 3] = init_array![String::from("hi!"); 3];

    //其展开形式为

    init_array! { String:: from ( "hi!" ) ; 3 }
    init_array! { @ accum ( 3 , e . clone (  ) ) -> (  ) }
    init_array! { @ accum ( 2 , e.clone() ) -> ( e.clone() , ) }
    init_array! { @ accum ( 1 , e.clone() ) -> ( e.clone() , e.clone() , ) }
    init_array! { @ accum ( 0 , e.clone() ) -> ( e.clone() , e.clone() , e.clone() , ) }
    init_array! { @ as_expr [ e.clone() , e.clone() , e.clone() , ] }

    //我们通过  在 一个新的匹配 部分 -> (($(body):tt)*) 的重复匹配项，来匹配 递归过程中的积累部分 。通过 ->($($body)* $e,)  来 进行 积累 以便 通过编译。

    ```


5. 分隔符号： 通过该种形式，我们可以实现 表达式尾部 匹配 任意多的 ，

     ```rust
     macro_rules! match_exprs {
         ($($exprs:expr),* $(,)*) => {...};
     }

     ```

    一般情况下，我们使用表达式  ($($exprs:expr),* 或者  $($exprs:expr,)*)  来写  macro的匹配项，这两种模式 只能匹配 存在 ， 或者 不存在 ， 而 使用上面的匹配方式 可以匹配任意多的，符号。


6. tt bundle 技巧： 在一个复杂的 macro中， 往往设定 许多中间的转发层 来携带 表达式或标识符 以传递给 最终处理层，在这种情况下， 中间层 可以将所有的参数 绑定到 一个tt中， 而不需要 写准确的 匹配规则，只是简单的将tt转发给 下一次层面 即可。

    ```rust
    // 下面的示例中，macro通过,  一个 ($ab:tt, $_skip:tt $($tail:tt)*) 的pattern 来进行 匹配，并在随后的 模式中 传递 $ab tt
    macro_rules! call_a_or_b_on_tail {
        ((a: $a:expr, b: $b:expr), call a: $($tail:tt)*) => {
            $a(stringify!($($tail)*))
        };

        ((a: $a:expr, b: $b:expr), call b: $($tail:tt)*) => {
            $b(stringify!($($tail)*))
        };

        ($ab:tt, $_skip:tt $($tail:tt)*) => {
            call_a_or_b_on_tail!($ab, $($tail)*)
        };
    }

    fn compute_len(s: &str) -> Option<usize> {
        Some(s.len())
    }

    fn show_tail(s: &str) -> Option<usize> {
        println!("tail: {:?}", s);
        None
    }

    fn main() {
        assert_eq!(
            call_a_or_b_on_tail!(
                (a: compute_len, b: show_tail),
                the recursive part that skips over all these
                tokens doesn't much care whether we will call a
                or call b: only the terminal rules care.
            ),
            None
        );
        assert_eq!(
            call_a_or_b_on_tail!(
                (a: compute_len, b: show_tail),
                and now, to justify the existence of two paths
                we will also call a: its input should somehow
                be self-referential, so let's make it return
                some ninety one!
            ),
            Some(91)
        );
    }

    ```

7. 因为rust对于 权限控制 可见性问题 上缺少 matcher，导致 macro处理起来非常困难。

    ```rust
    // 下面示例，使用单独的 pattern 来匹配， 不同的 权限的 （pub 或者无pub） 的 结构声明
    macro_rules! newtype_new {
        (struct $name:ident($t:ty);) => { newtype_new! { () struct $name($t); } };
        (pub struct $name:ident($t:ty);) => { newtype_new! { (pub) struct $name($t); } };

        (($($vis:tt)*) struct $name:ident($t:ty);) => {
            as_item! {
                impl $name {
                    $($vis)* fn new(value: $t) -> Self {
                        $name(value)
                    }
                }
            }
        };
    }

    macro_rules! as_item { ($i:item) => {$i} }

    ```

8. AST 强制： 解决rust 编译器 对于tt的一些问题，（当解析器期待一个特定的语法结构，而是找到一堆替换的 tt 标记时，就会出现问题。 它通常不会尝试解析它们，而是会放弃） 该种情况下，需要使用 AST强制， 这些强制通常与下推累积宏一起使用，以便让解析器将最终的 tt 序列视为一种特殊的语法结构。有以下几种 AST强制。

    ```rust
    macro_rules! as_expr { ($e:expr) => {$e} }
    macro_rules! as_item { ($i:item) => {$i} }
    macro_rules! as_pat  { ($p:pat) =>  {$p} }
    macro_rules! as_stmt { ($s:stmt) => {$s} }
    ```

9. 常见的 counting macro 的几种解法：

    ```rust

    // 重复方法
    macro_rules! replace_expr {
        ($_t:tt $sub:expr) => {$sub};
    }

    macro_rules! count_tts {
        ($($tts:tt)*) => {0usize $(+ replace_expr!($tts 1usize))*};
    }


    // 对于较小的数字，这是一种很好的方法，但可能会使编译器在输入大约 500 个左右的标记时崩溃。 考虑到输出将如下所示：
    //0usize + 1usize + /* ~500 `+ 1usize`s */ + 1usize

    macro_rules! count_tts {
        () => {0usize};
        ($_head:tt $($tail:tt)*) => {1usize + count_tts!($($tail)*)};
    }

    //递归调用， 存在与重复方法同样的 递归限制 问题，可以使用一次匹配 多个项目 来减少 递归深度。如下： 将能够匹配的项目 提高到 ~1,200 

    macro_rules! count_tts {
        ($_a:tt $_b:tt $_c:tt $_d:tt $_e:tt
         $_f:tt $_g:tt $_h:tt $_i:tt $_j:tt
         $_k:tt $_l:tt $_m:tt $_n:tt $_o:tt
         $_p:tt $_q:tt $_r:tt $_s:tt $_t:tt
         $($tail:tt)*)
            => {20usize + count_tts!($($tail)*)};
        ($_a:tt $_b:tt $_c:tt $_d:tt $_e:tt
         $_f:tt $_g:tt $_h:tt $_i:tt $_j:tt
         $($tail:tt)*)
            => {10usize + count_tts!($($tail)*)};
        ($_a:tt $_b:tt $_c:tt $_d:tt $_e:tt
         $($tail:tt)*)
            => {5usize + count_tts!($($tail)*)};
        ($_a:tt
         $($tail:tt)*)
            => {1usize + count_tts!($($tail)*)};
        () => {0usize};
    }
    ```


##### 对macro进行debug：
* 1）trace_macros 使用示例： 同样可以传递  -Z trace-macros 给rustc 命令调用上。

    ```rust
    #![feature(trace_macros)]

    macro_rules! each_tt {
        () => {};
        ($_tt:tt $($rest:tt)*) => {each_tt!($($rest)*);};
    }

    each_tt!(foo bar baz quux);
    trace_macros!(true); // 打开 debug
    each_tt!(spim wak plee whum); // 将 macro的调用进行展开
    trace_macros!(false); // 关闭debug
    each_tt!(trom qlip winp xod);


    The output is:

    each_tt! { spim wak plee whum }
    each_tt! { wak plee whum }
    each_tt! { plee whum }
    each_tt! { whum }
    each_tt! {  }
    ```

*  2) macro log_syntax  可以将 编译器 传递给macro的 内容全部输出出来，可以这样。

```rust
#![feature(log_syntax)]

macro_rules! sing {
    () => {};
    ($tt:tt $($rest:tt)*) => {log_syntax!($tt); sing!($($rest)*);};
}

```

* 3) 可以使用 编译参数 rustc -Z unstable-options --pretty=expanded hello.rs 来输出macro展开之后的 形式内容

```rust
// Shorthand for initialising a `String`.
macro_rules! S {
    ($e:expr) => {String::from($e)};
}

fn main() {
    let world = S!("World");
    println!("Hello, {}!", world);
    }
}

//使用 --pretty  之后输出如下， 这里面将 println同样输出出来了。
#![feature(no_std, prelude_import)]
#![no_std]
#[prelude_import]
use std::prelude::v1::*;
#[macro_use]
extern crate std as std;
// Shorthand for initialising a `String`.
fn main() {
    let world = String::from("World");
    ::std::io::_print(::std::fmt::Arguments::new_v1(
        {
            static __STATIC_FMTSTR: &'static [&'static str]
                = &["Hello, ", "!\n"];
            __STATIC_FMTSTR
        },
        &match (&world,) {
             (__arg0,) => [
                ::std::fmt::ArgumentV1::new(__arg0, ::std::fmt::Display::fmt)
            ],
        }
    ));
}

```
### 3种 procedural macros: 1) 在 struce & enum 上使用：#[derive] 2) 在任何 地方可以使用的： Attribute-like macros 3) Function-like macros
#### procedural macro 的 整体概览：
##### 1. 使用 rust func 对 macro接受的 TokenStream 进行处理， 并返回  TokenStream 
```rust
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    ...
}
```
##### 2. 依赖 crate:  proc-macro syn quote. 其中 proc-macro 存在于 rust lib 库中。
* proc-macro: 提供 将 #[derive()], #[some_type], println!() macro 声明方式，关联到 macro 定义的 rust func 中
* syn: 将TokenStream ---> syn::DeriveInput
* quote: rust code ---> TokenStream
* macro func: 将 syn::DeriveInput ---> TokenStream （需要quote 的帮助, 为 macro定义的主要逻辑）

##### 3. #[derive] example:
```rust
  // hello_macro
   pub trait HelloMacro {
       fn hello_macro();
   }

  // hello_macro_derive
  // Cargo.toml
  [lib]
  proc-macro = true

  [dependencies]
  syn = "1.0"
  quote = "1.0"

  // src/lib.rs
  extern crate proc_macro;

  use proc_macro::TokenStream;
  use quote::quote;
  use syn;

  #[proc_macro_derive(HelloMacro)]
  pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
      // Construct a representation of Rust code as a syntax tree
      // that we can manipulate
      let ast = syn::parse(input).unwrap();

      // Build the trait implementation
      impl_hello_macro(&ast)
  }

  fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
      let name = &ast.ident;
      let gen = quote! {
          impl HelloMacro for #name {
              fn hello_macro() {
                  println!("Hello, Macro! My name is {}!", stringify!(#name));
              }
          }
      };
      gen.into()
  }

  //useage

  use hello_macro::HelloMacro;
  use hello_macro_derive::HelloMacro;

  #[derive(HelloMacro)]
  struct Pancakes;

  fn main() {
      Pancakes::hello_macro();
  }


```

##### 4. Attribute-like macros: 跟 #[derive] 类似， 但是这里没有明确的示例， 即 不知道 item: TokenStream 的内容

```rust

#[route(GET, "/")]
fn index() {}

#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

##### 5. Function-like macros: 类似于 macro_rules，但是远比  macro_rule 要灵活，因为 可以使用rust code 对  TokenStream 进行处理。 所以可以sql 这种复杂的语句

```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);

#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
}

```
