---
title: rust 编码模式
date: 2021-06-30
tags: rust, design
---

### 设计模式
* Design Patterns: 是在编写软件时解决常见问题的方法。
* Anti-patterns: 反模式是解决这些相同常见问题的方法。 然而，虽然设计模式给我们带来了好处，但反模式却带来了更多的问题。
* idioms(惯用方法): 是编码时要遵循的准则。 它们是社区的社会规范。 你可以打破它们，但如果你这样做了，你应该有一个很好的理由。
#### 示例：
* 使用 borrowed type 作为 参数（为参数提供灵活）， 例如： &str 替换 &String, &[T] 替换 &Vec[T] 
* 使用format! 拼接字符串： 该方法是最清晰、可读的 组合string的方法， 缺点是 并不是最高效的。
* 提供 static new 方法 作为 构造方法
* Default trait 实现： 提供默认的构造方法， new 则提供具体参数的构造方法
* Collection 是 smart pointers： 一般的collect 实现了 Deref trait 来 提供 smart pointers 
  * 优势： 提供更多的灵活性
  * 劣势： 边界检查时不考虑仅通过解引用可用的方法和特征，因此使用这种模式的数据结构的泛型编程可能会变得复杂（参见 Borrow 和 AsRef 特征等）。 
* rust 没有提供 finally 代码块： 函数可能存在多个返回点， 导致 finally块（退出时执行）变得困难，对于 ？ macro， panick 来说更是如此， 所以rust 并没有提供 finally 块， 相反的 对象的 Drop trait中的drop方法 总是调用，无论怎样退出。（但是在drop 中发生panic 会导致 线程终止，从而不能够运行所有的 析构drop函数， 所以 drop 析构函数 可能并不能够得到保障，所以 需要在drop中格外小心，不能panic）
* mem::take, mem::replace 来 替换已有的enum中的数值， 示例： 我们使用 mem::take 来 将数值 使用 default 替换 并返回前值。 replace 则需要自己提供 数值。

  ```rust
  use std::mem;

  enum MyEnum {
      A { name: String, x: u8 },
      B { name: String }
  }

  fn a_to_b(e: &mut MyEnum) {
      if let MyEnum::A { name, x: 0 } = e {
          // this takes out our `name` and put in an empty String instead
          // (note that empty strings don't allocate).
          // Then, construct the new enum variant (which will
          // be assigned to `*e`).
          *e = MyEnum::B { name: mem::take(name) }
      }
  }
  ```
* 在 stack 上 进行动态分发（dynamic dispatch）：  rust 只保证 每个使用到的 变量 是 初始化过的。所以 可以存在未使用的变量 是未初始化的。

```rust
use std::io;
use std::fs;


// These must live longer than `readable`, and thus are declared first:
let (mut stdin_read, mut file_read);

// We need to ascribe the type to get dynamic dispatch.
let readable: &mut dyn io::Read = if arg == "-" {
    stdin_read = io::stdin();
    &mut stdin_read
} else {
    file_read = fs::File::open(arg)?;
    &mut file_read
};

// Read from `readable` here.

```
* Option 可以被 视为 Interator， 一个 包装 None 或者 element 的iterator， 可以如下使用：

```rust
let turing = Some("Turing");
let mut logicians = vec!["Curry", "Kleene", "Markov"];

logicians.extend(turing);

// equivalent to
if let Some(turing_inner) = turing {
    logicians.push(turing_inner);
}



let turing = Some("Turing");
let logicians = vec!["Curry", "Kleene", "Markov"];

for logician in logicians.iter().chain(turing.iter()) {
    println!("{} is a logician", logician);
}
```
* private in struct:  struct 中private field  可以 使用如下模式 来进行修改：

```rust

mod a {
    // Public struct.
    pub struct S {
        pub foo: i32,
        // Private field.
        bar: i32,
    }
}

fn main(s: a::S) {
    // Because S::bar is private, it cannot be named here and we must use `..`
    // in the pattern.
    let a::S { foo: _, ..} = s;
}

```
*  temporary mutable 暂时性的 mut： 

```rust
// 使用 内嵌的 {} 嵌套代码
let data = {
    let mut data = get_vec();
    data.sort();
    data
};

// 重新绑定
let mut data = get_vec();
data.sort();
let data = data;
```
