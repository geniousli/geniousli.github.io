---
title: Rust Async
date: 2021-05-09
tags: rust, async, cocurrent
---

### 并发模型
* **OS threads**:  1) 不需要改变代码编码模式  2) 线程间 同步困难  3) 性能开销比较大， 线程池 可以一定程度上减少这些开销， 但是并不能够支撑起 庞大的IO 工作
* **事件驱动模型(Event-driven)**： 结合callback, 性能非常好，但是 导致 非线性的控制流， Data flow and error propagation is often hard to follow.
* **协程(coroutines)**： like thread， 不需要改变代码编码模式， 共容易使用，提供了包括async 的工具， 支持大量的 任务，但是，其对于 底层的细节抽象 导致 系统编程非常困难
* **Actor model**： 将所有的并行运算抽象为 actor ，其之间的沟通通过 message passing 非常像 分布式系统， 可以实现非常好的性能， 但是留下了许多实际问题 依然没有解决， 比如 flow control and retry logic
* 总结： async 编程 能够让 Rust 此类 **系统编程语言** 编写出高性能代码，同时具有 thread 与 协程(coroutines) 的优势;

### Rust Async 的现状
1. Rust Async 的特点： 
  *  Futures是rust 内置的
  *  Async是zero-cost 的，即： 并不需要 在堆上 进行内存分配 和动态分发，即高性能（you can use async without heap allocations and dynamic dispatch, which is great for performance!）
  *  没有内建的runtime， 由社区提供
  *  可以支持 单线程 或者 多线程 的runtime
2. Rust Async vs threads 的比较
  * threads： 适合少量的工作， 不用改变代码结构， 生成 线程 与线程之间的切换 是非常昂贵的， 线程池可以一定程度上减轻此类开销
  * Async： 可以显著的减少CPU 与 内存的开销， 尤其是在对于 大量IO任务时。 比thread 模式能够处理更多的任务，因为 该模型，使用 较少的 threads 处理大量的 task。 但是其二进制的文件会比传统的非异步编码的代码要大。

3. 异步编程的支持：
  * 标准库： 提供 Future trait 抽象
  * Rust Compiler： 提供对  async/await 语法的支持
  * futures crate： 提供 工具类型， macros， 以及 方法
  * async runtime： 提供对 async code, IO, task spawn 的运行。有 TOkio, async-std
4. 编译 与  debug：
  * 为了支持异步代码：  rust 需要使用异常一些更复杂的 语言特性， 比如lifetimes pinning。 你可能将经常遇到此类错误
  * runtime errors： 编译器 遇到 async function，将产生一个 状态机（state machine）， Stack traces 将包含状态机内部的详细信息， 对比 Rust 同步代码, runtime errors debug 要复杂不少

5. New failure modes： 异步Rust中可能会出现一些新颖的故障模式，例如，如果您从异步上下文中调用了阻止函数，或者您错误地实现了Future特性。 这样的错误可以无声地通过编译器，有时甚至可以通过单元测试。 本书旨在为您提供对底层概念的深刻理解，可以帮助您避免这些陷阱。


### Rust 异步编程 简单示例： 
  * async/.await 语法： rust compiler 为 async 声明的block转换成一个实现了 Future trait 的 state machine。 await 等待Future的完成， 但是将thread yield 出去以允许其他Future执行
  * block_on: 阻塞当前 thread，直到 future 完成， 不允许 thread 运行其他的 Future

```rust
// Cargo.toml
[dependencies]
futures = "0.3"

// main.rs
use futures::executor::block_on;

async fn hello_world() {
  println!("hello, world!");
}

fn main() {
  let future = hello_world(); // Nothing is printed
  block_on(future); // `future` is run and "hello, world!" is printed
}


// await 示例： 

async fn learn_song() -> Song { /* ... */ }
async fn sing_song(song: Song) { /* ... */ }
async fn dance() { /* ... */ }


// 这里使用 block_on 导致  learn_song  aing_song dance 的 串行执行, 因为block_on 将阻塞thread，直到 Future 执行完成
fn main() {
  let song = block_on(learn_song());
  block_on(sing_song(song));
  block_on(dance());
}



// 这里使用await 可以将 thread 让出，以便  Future f2 执行。 join 能够同时执行 两个 future
async fn learn_and_sing() {
  // Wait until the song has been learned before singing it.
  // We use `.await` here rather than `block_on` to prevent blocking the // thread, which makes it possible to `dance` at the same time.
  let song = learn_song().await;
  sing_song(song).await;
}


async fn async_main() {
  let f1 = learn_and_sing(); 
  let f2 = dance();
  // `join!` is like `.await` but can wait for multiple futures concurrently. 
  // If we're temporarily blocked in the `learn_and_sing` future, the `dance`
  // future will take over the current thread. If `dance` becomes blocked, 
  // `learn_and_sing` can take back over. If both futures are blocked, then 
  // `async_main` is blocked and will yield to the executor. 
  futures::join!(f1, f2);
}



fn main() { 
  block_on(async_main());
}

```

#### Future Trait: rust 异步编程的 核心点， Future 即是一个可以产生 value 的异步计算抽象。简单的 Future 可以如下： 

```rust
trait SimpleFuture {
    type Output;
    fn poll(&mut self, wake: fn()) -> Poll<Self::Output>;
}
enum Poll<T> {
    Ready(T),
    Pending,
}
pub struct SocketRead<'a> {
    socket: &'a Socket,
}
impl SimpleFuture for SocketRead<'_> {
    type Output = Vec<u8>;
    fn poll(&mut self, wake: fn()) -> Poll<Self::Output> {
        if self.socket.has_data_to_read() {
            // The socket has data-- read it into a buffer and return it.
            Poll::Ready(self.socket.read_buf())
        } else {
            // The socket does not yet have data. //
            // Arrange for `wake` to be called once data is available.
            // When data becomes available, `wake` will be called, and the
            // user of this `Future` will know to call `poll` again and
            // receive data. self.socket.set_readable_callback(wake); Poll::Pending
        }
    }
}

```
* poll func 为Future 接口， 如果future 完成  则返回 Poll::Ready(result), 否则 返回 Poll::Pending, 并在当 Future 可以取得进一步进展时 调用 wake() 函数， 当wake 函数调用， Future 的 executor 将再次 在future 调用poll 以取得进展
* wake 的作用： 如果没有wake 则 executor 将没有任何知识 能够知道 一个特定的future 可以取得进展， 除非周期性的在 每个future 上进行poll。 wake的存在能够让 executor 知道 那个 future 需要被poll
* Timer Future 的简单实现：

```rust

use std::{
    future::Future,
    pin::Pin,
    sync::{Arc, Mutex},
    task::{Context, Poll, Waker},
    thread,
    time::Duration,
};


pub struct TimerFuture {
    shared_state: Arc<Mutex<SharedState>>, // 需要使用锁， 跨 thread 更改变量
}

/// Shared state between the future and the waiting thread
struct SharedState {
    /// Whether or not the sleep time has elapsed
    completed: bool,

    // The waker for the task that `TimerFuture` is running on.
    // The thread can use this after setting `completed = true` to tell
    // `TimerFuture`'s task to wake up, see that `completed = true`, and
    // move forward.
    waker: Option<Waker>,
}


impl Future for TimerFuture {
    type Output = ();
    // 需要注意 这里面 self参数类型为Pin<&mut Self> 以及cx 为Context
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        // Look at the shared state to see if the timer has already completed.
        let mut shared_state = self.shared_state.lock().unwrap();
        if shared_state.completed {
            Poll::Ready(())
        } else {
            // Set waker so that the thread can wake up the current task
            // when the timer has completed, ensuring that the future is polled
            // again and sees that `completed = true`.
            //
            // It's tempting to do this once rather than repeatedly cloning
            // the waker each time. However, the `TimerFuture` can move between
            // tasks on the executor, which could cause a stale waker pointing
            // to the wrong task, preventing `TimerFuture` from waking up
            // correctly.
            //
            // N.B. it's possible to check for this using the `Waker::will_wake`
            // function, but we omit that here to keep things simple.
            shared_state.waker = Some(cx.waker().clone());
            Poll::Pending
        }
    }
}


impl TimerFuture {
    // Create a new `TimerFuture` which will complete after the provided
    // timeout.
    pub fn new(duration: Duration) -> Self {
        let shared_state = Arc::new(Mutex::new(SharedState {
            completed: false,
            waker: None,
        }));

        // Spawn the new thread
        // 这个 Future 实现的 比较奇怪，直接 使用另一个 thread 中sleep 进行 timer 的实现, 所以 在上面 TimerFuture 中的 Sharedstate 需要进行 Mutex 进行保护
        // 这也是后面提到 Runtime  需要提供 Timer的 重要原因吧
        let thread_shared_state = shared_state.clone();
        thread::spawn(move || {
            thread::sleep(duration);
            let mut shared_state = thread_shared_state.lock().unwrap();
            // Signal that the timer has completed and wake up the last
            // task on which the future was polled, if one exists.
            shared_state.completed = true;
            if let Some(waker) = shared_state.waker.take() {
                waker.wake()
            }
        });

        TimerFuture { shared_state }
    }
}


// Executor 的实现：

/// Task executor that receives tasks off of a channel and runs them.
struct Executor {
    ready_queue: Receiver<Arc<Task>>,
}

/// `Spawner` spawns new futures onto the task channel.
#[derive(Clone)]
struct Spawner {
    task_sender: SyncSender<Arc<Task>>,
}

/// A future that can reschedule itself to be polled by an `Executor`.
struct Task {
    /// In-progress future that should be pushed to completion.
    ///
    /// The `Mutex` is not necessary for correctness, since we only have
    /// one thread executing tasks at once. However, Rust isn't smart
    /// enough to know that `future` is only mutated from one thread,
    /// so we need to use the `Mutex` to prove thread-safety. A production
    /// executor would not need this, and could use `UnsafeCell` instead.
    future: Mutex<Option<BoxFuture<'static, ()>>>,

    /// Handle to place the task itself back onto the task queue.
    task_sender: SyncSender<Arc<Task>>,
}

fn new_executor_and_spawner() -> (Executor, Spawner) {
    // Maximum number of tasks to allow queueing in the channel at once.
    // This is just to make `sync_channel` happy, and wouldn't be present in
    // a real executor.
    const MAX_QUEUED_TASKS: usize = 10_000;
    let (task_sender, ready_queue) = sync_channel(MAX_QUEUED_TASKS);
    (Executor { ready_queue }, Spawner { task_sender })
}


impl Spawner {
    fn spawn(&self, future: impl Future<Output = ()> + 'static + Send) {
        let future = future.boxed();
        let task = Arc::new(Task {
            future: Mutex::new(Some(future)),
            task_sender: self.task_sender.clone(),
        });
        self.task_sender.send(task).expect("too many tasks queued");
    }
}


impl ArcWake for Task {
    fn wake_by_ref(arc_self: &Arc<Self>) {
        // Implement `wake` by sending this task back onto the task channel
        // so that it will be polled again by the executor.
        let cloned = arc_self.clone();
        arc_self
            .task_sender
            .send(cloned)
            .expect("too many tasks queued");
    }
}

impl Executor {
    fn run(&self) {
        while let Ok(task) = self.ready_queue.recv() {
            // Take the future, and if it has not yet completed (is still Some),
            // poll it in an attempt to complete it.
            let mut future_slot = task.future.lock().unwrap();
            if let Some(mut future) = future_slot.take() {
                // Create a `LocalWaker` from the task itself
                let waker = waker_ref(&task);
                let context = &mut Context::from_waker(&*waker);
                // `BoxFuture<T>` is a type alias for
                // `Pin<Box<dyn Future<Output = T> + Send + 'static>>`.
                // We can get a `Pin<&mut dyn Future + Send + 'static>`
                // from it by calling the `Pin::as_mut` method.
                if let Poll::Pending = future.as_mut().poll(context) {
                    // We're not done processing the future, so put it
                    // back in its task to be run again in the future.
                    *future_slot = Some(future);
                }
            }
        }
    }
}



fn main() {
    let (executor, spawner) = new_executor_and_spawner();

    // Spawn a task to print before and after waiting on a timer.
    spawner.spawn(async {
        println!("howdy!");
        // Wait for our timer future to complete after two seconds.
        TimerFuture::new(Duration::new(2, 0)).await;
        println!("done!");
    });

    // Drop the spawner so that our executor knows it is finished and won't
    // receive more incoming tasks to run.
    drop(spawner);

    // Run the executor until the task queue is empty.
    // This will print "howdy!", pause, and then print "done!".
    executor.run();
}

```

### Executors and System IO:
#### Executor： 谁来调用 Future 的poll 方法？ 答案是 Future Executor。 executor 调用一大堆 Futures 的poll方法 以便让Future 取得进展， 当 Future 能够 取得进一步进展时，通过调用wake 方法， 以便 executor 再次执行 Future。
#### System IO: 在上面的 SimpleFuture 代码中， 谁来执行 wake 方法呢？ self.socket.set_readable_callback(wake) 又是如何触发的呢？ 答案是 epoll 的IO多路复用，可以让我们 使用thread 对 socket文件进行 监听，循环检测 IO 事件。 
#### Executors: 单线程 与 多线程
> 多线程执行程序可同时在多个任务上取得进展。 对于具有许多任务的工作负载，它可以极大地加快执行速度，但是在任务之间同步数据通常会更加昂贵。 在单线程和多线程运行时之间进行选择时，建议测量应用程序的性能。

任务可以在创建任务的线程上运行，也可以在单独的线程上运行。
异步运行时通常提供将任务生成到单独线程上的功能。 即使任务在单独的线程上执行，它们也应该是非阻塞的。 为了在多线程执行器上安排任务，它们也必须是Send。 一些运行时提供了 生成并发送任务 的功能，以确保每个任务都在生成它的线程上执行。 它们还可以提供用于将阻塞任务生成到专用线程上的功能，这对于从其他库运行阻塞同步代码很有用。

* async Lifetimes， async move， 因为异步的存在， 导致 async { /../ } 可以 传递给变量 并进行 .await， 导致  block {} 中 的包含的变量，以及 引用 需要与 Future 存在的周期相同。 async move 允许 like normal block 一样， 允许将 block中变量 移入到 block中， 并跟随 Future 一样的生命周期
* 当使用 多线程的 executor时， Future 可能在 threads 中进行移动，所以 在async block中的 any variables 必须同样能够在 threads中进行移动， 意味着  任何没有实现 [Send trait](https://doc.rust-lang.org/std/marker/trait.Send.html)、reference type 没有 实现 [Sync trait](https://doc.rust-lang.org/std/marker/trait.Sync.html) 的都不能够 在async block中使用。
* 锁：不能使用 传统的 non-futures-aware lock, 因为Future 可能在threads 中进行移动 从而导致死锁，应该使用  futures::lock 中的Mutex

### 工具以及 trait:

#### Pin: 
* Pin： maker， 保证 对象 implement !Unpin 永远不会被移动。因为比较难以理解， 下面为英文：

> The Pin type wraps pointer types, guaranteeing that the values behind the pointer won't be moved. For example, Pin<&mut T> , Pin<&T> , Pin<Box<T>> all guarantee that T won't be moved if T: !Unpin .
> Most types don't have a problem being moved. These types implement a trait called Unpin . Pointers to Unpin types can be freely placed into or taken out of Pin . For example, u8 is
> Unpin , so Pin<&mut u8> behaves just like a normal &mut u8 .
> However, types that can't be moved after they're pinned have a marker called !Unpin .
> Futures created by async/await is an example of this.

* Pinning: 大概的以为是 是一个编译器 标记（marker） 用来保证 impl  !Unpin 的对象 在内存中 不被移动，即pin: 
* Pin Summary: 
    1. If T: Unpin (which is the default), then Pin<'a, T> is entirely equivalent to &'a mut T. in other words: Unpin means it's OK for this type to be moved even when pinned, so Pin will have no effect on such a type.
    2. Getting a &mut T to a pinned T requires unsafe if T: !Unpin.
    3. Most standard library types implement Unpin. The same goes for most "normal" types you encounter in Rust. A Future generated by async/await is an exception to this rule.
    4. You can add a !Unpin bound on a type on nightly with a feature flag, or by adding std::marker::PhantomPinned to your type on stable.
    5. You can either pin data to the stack or to the heap.
    6. Pinning a !Unpin object to the stack requires unsafe
    7. Pinning a !Unpin object to the heap does not require unsafe. There is a shortcut for doing this using Box::pin.
    8. For pinned data where T: !Unpin you have to maintain the invariant that its memory will not get invalidated or repurposed from the moment it gets pinned until when drop is called. This is an important part of the pin contract.

####  Stream Trait: like Future 但是能够 在完成之前 传递多个数值 like Iterator，即： 返回对象为   Poll<Option<Self::Item> >。 stream 可以实现并行，函数有 for_each_concurrent， try_for_each_concurrent
#### 多个Future 同时执行: 
* 工具方法: 
   1. join! : waits for futures to all complete  
   2. select! : waits for one of several futures to complete 
   3. Spawning: creates a top-level task which ambiently runs a future to completion 
   4. FuturesUnordered : a group of futures which yields the result of each subfuture

* join! 示例代码: try_join!  在其中一个 Future 返回错误的时候，立即返回

```rust
// book 与 music 串行执行， music 等待book 执行完 才能执行
async fn get_book_and_music() -> (Book, Music) {
    let book = get_book().await;
    let music = get_music().await;
    (book, music)
}
// 错误的尝试，将 book 与 music 并行执行
async fn get_book_and_music() -> (Book, Music) {
    let book_future = get_book();
    let music_future = get_music();
    (book_future.await, music_future.await)
}
// 真正的并行执行
async fn get_book_and_music() -> (Book, Music) {
    let book_fut = get_book();
    let music_fut = get_music();
    join!(book_fut, music_fut)
}

```
* select! 示例代码： select macro 的使用风格比较奇怪， 需要在深入理解一下

```rust

#![allow(unused)]
fn main() {
use futures::{
    future::FutureExt, // for `.fuse()`
    pin_mut,
    select,
};

async fn task_one() { /* ... */ }
async fn task_two() { /* ... */ }

async fn race_tasks() {
    let t1 = task_one().fuse();
    let t2 = task_two().fuse();

    pin_mut!(t1, t2);

    select! {
        () = t1 => println!("task one completed first"),
        () = t2 => println!("task two completed first"),
    }
}
}
```

* Unpin ： Unpin 是必须的， 因为在 select 中 数值是使用mut reference，而非take ownership 的， 没有完成的future 依然可以在后面的select 中使用。
* FusedFuture: 同样是必须的， 因为 select 中 必须 not poll 已经完成的Future， FusedFuture 实现了 跟踪 Future 是否已经完成， 
* 这两个 trait 能够让 select 在 loop block 中使用。 如下代码示例：

```rust
#![allow(unused)]
fn main() {
use futures::{future, select};

async fn count() {
    let mut a_fut = future::ready(4);
    let mut b_fut = future::ready(6);
    let mut total = 0;

    loop {
        select! {
            a = a_fut => total += a,
            b = b_fut => total += b,
            complete => break,
            default => unreachable!(), // never runs (futures are ready, then complete)
        };
    }
    assert_eq!(total, 10);
}
}

```

### Async Blocks 编码存在的一些问题: 
* ？: 在 async block 中  使用 ? 需要  额外的 声明 Error type 来帮助 编译器确定 错误类型， 如下：

```rust
// 产生 编译错误
let fut = async {
    foo().await?;
    bar().await?;
    Ok(())
};

// 正确的解决方法
#![allow(unused)]
fn main() {
struct MyError;
async fn foo() -> Result<(), MyError> { Ok(()) }
async fn bar() -> Result<(), MyError> { Ok(()) }
let fut = async {
    foo().await?;
    bar().await?;
    Ok::<(), MyError>(()) // <- note the explicit type annotation here
};
}
```
* Future 是否 能够 send： 有些Future state machines 可以安全地被send, 而有些则不能。 是否Future 可以被 send 取决于是否在.await点上保留 non-send type。 当可能在.await点上保留值时，编译器会尽力进行近似，但是编译器的分析在许多地方都过于保守。 如下面代码示例：


```rust
// 通过编译
use std::rc::Rc;
#[derive(Default)]
struct NotSend(Rc<()>);
async fn bar() {}
async fn foo() {
    NotSend::default();
    bar().await;
}

fn require_send(_: impl Send) {}

fn main() {
    require_send(foo());
}

// 产生编译错误
use std::rc::Rc;
#[derive(Default)]
struct NotSend(Rc<()>);
async fn bar() {}
async fn foo() {
    let x = NotSend::default();
    bar().await;
}
fn require_send(_: impl Send) {}
fn main() {
   require_send(foo());
}


// 正确的解决方法
use std::rc::Rc;
#[derive(Default)]
struct NotSend(Rc<()>);
async fn bar() {}
async fn foo() {
    {
        let x = NotSend::default();
    }
    bar().await;
}
fn require_send(_: impl Send) {}
fn main() {
   require_send(foo());
}

```

* 递归问题： 因async fn 内部的状态机 实现导致 递归的使用 需要 额外 解决方案. 如下：

```rust
// This function:
async fn foo() {
    step_one().await;
    step_two().await;
}
// generates a type like this:
enum Foo {
    First(StepOne),
    Second(StepTwo),
}

// So this function:
async fn recursive() {
    recursive().await;
    recursive().await;
}

// generates a type like this:  因为类型对象存在递归，导致 无法通过编译，需要通过 Box 进行封装 来进行规避
enum Recursive {
    First(Recursive),
    Second(Recursive),
}

// --------------------
// In order to allow this, we have to introduce an indirection using Box. Unfortunately, compiler limitations mean that just wrapping the calls to recursive() in Box::pin isn't enough. To make this work, we have to make recursive into a non-async function which returns a .boxed() async block:

use futures::future::{BoxFuture, FutureExt};

fn recursive() -> BoxFuture<'static, ()> {
    async move {
        recursive().await;
        recursive().await;
    }.boxed()
}

```
### Async 生态: 

* Async Runtimes = reactor + one or more executors
* Reactors： 提供对于 外部事件 比如: 异步IO 进程内通讯 timer 等 简单的订阅方式 (provide subscription mechanisms for external events, like async I/O, interprocess communication, and timers. In an async runtime, subscribers are typically futures representing low-level I/O operations)
* Executors: Future 具体的执行者， 因为只有 thread ，process 才能够执行代码，所以Future等 抽象事物，依然需要具体的executor  (handle the scheduling and execution of tasks. They keep track of running and suspended tasks, poll futures to completion, and wake tasks when they can make progress)
* Futures Crate： 包含 async code 有用的trait 与 函数， 包含trait有： Stream, Sink, AsyncRead, AsyncWrite  , 工具有： join! select! etc （这些可能将成为 标准库 的一部分） future 实现了自己的 executor， 但是并没有 reactor。 所以一个常见的组合是： Future 的工具 + 其他crate 的executor
* 常见的 Async runtime:
  * Tokio: A popular async ecosystem with HTTP, gRPC, and tracing frameworks. async-std: A crate that provides asynchronous counterparts to standard library components.
  * smol: A small, simplified async runtime. Provides the Async trait that can be used to wrap structs like*  UnixStream or TcpListener
  * fuchsia-async: An executor for use in the Fuchsia OS.

### 相关资料：
* Event-driven programming（事件驱动 模型， rust future 就是此类模型 ） https://en.wikipedia.org/wiki/Event-driven_programming
* event driven  同样存在多种形式， refactor  pattern & https://en.wikipedia.org/wiki/Proactor_pattern
* 相关资料： 有： http://www.alan-g.me.uk/l2p/tutevent.htm http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf  https://altushost-swe.dl.sourceforge.net/project/eventdrivenpgm/event_driven_programming.pdf
* 在rust 的 async program 中： 主要有  rust async 的语法支持， 社区提供的 future 模型， 以及 对应 runtime executor 实现，则有： async-std, tokio， smol， fuchsia-async。 future 的抽象只有一种，而对应的 runtime executor 因为 与 操作系统 生态有关，则有多种的实现
* Coroutines （协程， 即是语言在 OS上 对于 轻量线程的抽象， 在 《现代操作系统》 中 有详细介绍过此类模型，但是因为其 一些根源性问题， 对于系统信号、调度等并不友好）
* The actor model: 将单元划分为 actor， 使用消息进行 通讯， Erlang 是典型的 actor并发代表
