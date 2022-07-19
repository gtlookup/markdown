# Future

https://www.jianshu.com/p/2151e0a13b27?utm_campaign=hugo

字面翻译为"未来、将来"，在计算机中的含义：

- 对异步的抽象，类似 promise 概念（为解决Callback）
- 用于在并发编程语言中同步程序执行的构造。由于某些任务未能马上完成，需要一个对象来代理这个未知结果
- 相当于 golang 里的协程

## 1. Future 异步编程思路

1）异步环境下当前调用就绪时执行，没就绪则不等待任务就绪，而是返回一个 Future，等待将来任务就绪时再调度执行

2）这里返回 Future 时，关键的是要声明事件什么时候就绪，就绪后怎么唤醒这个任务到调度器去调度执行

## 2. Rust 中的 Future

Rust 中的 Futures 类似于 javascript 里的 promise，是对 Rust 并发原语的强大抽象，也是 async / await 的基石。Futures 在标准库的 std 中。

具体来讲，Futures 是一系列异步计算所代表的值。Futures 允许你定义一个可以被异步运行的任务，比如一个网络调用或计算。可以在任务结果上链接函数，对其进行转换、错误处理、与其它 Futures 合并以及执行许多其它计算。这些函数只有当 Futures 执行 run 函数时才会被执行。

让 Futures 工作需要：一个 runner、futures trait 以及 poll 类型

1）一个 Future 可以理解为一段供将来调度执行的代码

2）当前 Rust 中使用的：

- futures = "0.3.14"，标准库中的 async / await 来源（尚未全部移入标准库）
- tokio = "1.5.0"，为 Future 提供平台支持，当然也可以用 async-std

## 3. 标准库中线程的一些问题

1）操作系统调试开销

2）写代码难度比较大（如收集多线程结果）

3）线程切换和跨线程共享数据上会产生很多额外开销

## 4. Future 下的包

1）Future：异步生成单个值

```rust
pub train Future {
    type Output; // Future 返回值的类型
    // poll 注册 future 进行 Executor，起轮循作用
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

```rust
pub enum Poll<T> { // Future::poll 的返回值
    Ready(T), // 表示就绪
    Pending,  // 表示等待
}
```

```rust
pub struct Context<'a> { // 对 Future 进行调度的功能
    waker: &'a Waker,
    _marker: PhantomData<fn(&'a ()) -> &'a ()>,
}

pub struct Waker { waker: RawWaker }

impl Waker {
    /// 唤醒绑定在 Waker 上的数据，通常是 Future
    pub fn wake(self) {}
    pub fn wake_by_ref(&self) {}
    pub fn will_wake(&self, other: &Waker) -> bool {}
    pub unsafe fn from_raw(waker: RawWaker) -> Waker {}
}

pub struct RawWaker {
    data: *const (),
    vtable: &'static RawWakerVTable,
}

pub struct RawWakerVTable { // RawWaker 行为的虚函数表
    clone: unsafe fn(*const ()) -> RawWaker,
    wake: unsafe fn(*const ()),
    wake_by_ref: unsafe fn(*const ()),
    drop: unsafe fn(*const ()),
}
```

2）Stream

3）Sink：异步写入。通常可以抽象网络连接的写入端（即消息队列中的提供者 `producer`），包含：Channels、Sockets、Pipes

```rust
pub trait Sink<Item> {
    type Error;

    fn poll_ready(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>>;
    fn start_send(self: Pin<&mut Self>, item: Item) -> Result<(), Self::Error>;
    fn poll_flush(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>>;
    fn poll_close(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>>;
}
```

4）Timer：延时执行一些操作

## 5. async / await

用**线程做计算密集**的事情，而用**协程做 IO 密集**的事情，这样系统可以达到最好的吞吐能力。

这是因为 **Future 的调度是协作式多任务，除非 Future 主动放弃 CPU，不然它就会一直被执行，直到运行结束**

- 把相应的函数变成 async 函数，这样函数的返回值会变成一个 `Future`。

- 在调用 async 函数的地方，添加 `.await` 来处理 async 的状态机。

- 在使用 `spawn` 的地方，使用 `tokio` 或者 `async_std` 对应的 `spawn`，来创建一个协程。

- 在入口函数，引入 `executor`，比如使用宏 `#[tokio::main]`

```rust
// 实际上是个生成器，可将代码
async fn gen(arg: Parameter) -> Result {}
// 看成是
fn gen() -> Impl Future<Output=Result> {}
```

```rust
// 正常的异步调用，看起来很麻烦，但可以简化
loop {
    match f::poll(cx) {
        Poll::Ready(x) => return x;
        Poll::Pending => {}
    }
}
// 简化上面一段，实际上是个语法糖
let x = f.await;
```



# Tokio 介绍

是 Rust 语言的一种异步==运行时==（基于上面说的 Future），可用来编写可靠的异步 Rust 应用，具有以下特点：

- 快速：零成本抽象，可以接近裸机的性能
- 可靠：基于 Rust 的生命周期、类型系统、并发模型来减少 Bug 和确保线程安全
- 可扩展：占用资源少，并能处理背压（backpressure）和取消（cancellation）操作

是一个事件驱动的非阻塞I/O平台，用于使用Rust编写异步应用. 在较高的层次上，它提供了几个主要的组件:

- 基于多线程与工作流窃取的 任务调度器 [scheduler](https://docs.rs/tokio/latest/tokio/runtime/index.html).
  - **tokio 的调度器（executor）会运行在多个线程上**，运行线程自己的 ready queue 上的任务（Future），如果没有，就去别的线程的调度器上“偷”一些过来运行
- 响应式的，基于操作系统的事件队列(比如，epoll, kqueue, IOCP, 等...).
- 异步的 [TCP and UDP](https://docs.rs/tokio/latest/tokio/net/index.html) socket.

这些组件提供了用来构建异步应用所需要的运行时组件.

## 1. 异步 main 函数

`#[tokio::main]` 将 `async fn main` 转换成一个运行时实例，且执行异步 main 函数的同步 `fn main`

```rust
#[tokio::main]
async fn main() {
    println!("hello");
}
// 转换成以下：
fn main() {
    let mut rt = tokio::runtime::Runtime::new().unwrap();
    rt.block_on(async {
        println!("hello");
    })
}
```

## 2. async / await

前面加 async 的函数，在被调用时不会被执行，而是返回了一个表示操作了的值，如果想让其执行，加上.await就行

`async fn` 返回一个实现了 Future trait 的异步类型

```rust
use std::thread;
use std::time::Duration;

async fn f1() { println!("f1"); }
#[tokio::main]
async fn main() {
    let op = f1();
    println!("main");
    thread::sleep(Duration::from_millis(1000));
    op.await
}
// 结果：main f1
```

```rust
f1();
println!("main");
thread::sleep(Duration::from_millis(1000)); // 结果：就只有 main，f1() 需要 await 才会执行
```

```rust
async fn f1() { println!("f1"); thread::sleep(Duration::from_millis(1000));}
#[tokio::main]
async fn main() {
    f1().await;
    println!("main");
} // 结果：f1 main
```

# Spawning

一般情况下 tokio 暴露了与 std 相同的 API，只是 tokio 使用了 `async fn`

```rust
use tokio::net::{TcpStream, TcpListener};
use mini_redis::{Connection, Frame};

#[tokio::main]
pub async fn main() {
    // 监听 6479
    let mut listener = TcpListener::bind("127.0.0.1:6379").await.unwrap();
    loop {
        // 返回的第二个参数包含请求端的ip和端口
        let (socket, _) = listener.accept().await.unwrap();
        process(socket).await;
    }
}

async fn process(socket: TcpStream) {
    // Connection 在 mini-redis 库里
    let mut cnn = Connection::new(socket);

    if let Some(frame) = cnn.read_frame().await.unwrap() {
        println!("Got：{:?}", frame);
        
        let res = Frame::Error("error".to_string()); // 创建一个错误
        cnn.write_frame(&res).await.unwrap(); // 将错误返回给客户端
    }
}
// 关掉 mini-redis 后运行这个，然后再运行上个例子，得到结果：
// 上个例子：Error: "error"
// 本例：Got：Array([Bulk(b"set"), Bulk(b"hello"), Bulk(b"world")])
// 这是一个简单的例子，只能接受一次连接和请求，处理完就退出了
```

## 1. 并发

并发与并行不同，在两个任务间交替执行叫并发；两个任务由两个人分别执行，叫并行。tokio 可以在单线程下同时处理许多并发任务

```rust
// 为了处理多个链接，可以稍微改一下
// process(socket).await; 将客户端连接成功后的处理放到 spawn 里
tokio::spawn(async move { process(socket).await; });
```

## 2. 任务

一个 tokio 任务是一个异步的绿色线程（进阶.md里有说过，Rust 除OS线程外也有绿色线程库）。通过 `tokio::spawn(async move {})` 来创建并返回 `JoinHandle`，调用者可根据 `JoinHandle`与任务进行交互（.await 获取任务返回值）

```rust
#[tokio::main]
async fn main() {
    let h = tokio::spawn(async { "return a value" });
    println!("{}", h.await.unwrap()); // 结果：return a value
}
```

## 3. 静态边界(`'static bound`)

通过 `tokio::spawn` 产生的任务必须是 `'static` 的，产生的数据不能借用任何数据

# sleep

```rust
use tokio::time::{Instant, sleep, sleep_until, Duration};
// 两个差不多
sleep_until(Instant::now() + Duration::from_millis(100)).await;
sleep(Duration::from_millis(100)).await;
```

# join vec< handle >

将多个 `tokio::spawn` 返回的任务添加到 `Vec` 里，然后等待全部执行完

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut vs = Vec::new();
    let h1 = tokio::spawn(async {
        tokio::time::sleep(tokio::time::Duration::from_secs(1)).await;
        println!("1")
    });
    let h2 = tokio::spawn(async {
        tokio::time::sleep(tokio::time::Duration::from_secs(2)).await;
        println!("2")
    });
    let h3 = tokio::spawn(async {
        println!("3")
    });
    
    vs.push(h1);
    vs.push(h2);
    vs.push(h3);
    
    tokio::try_join!(h1,h2,h3); // 只能分别写 
    // 需要：
    // [dependencies]
    // futures = "0.3.17"
    // 这种写法跑的很慢：https://github.com/tokio-rs/tokio/issues/2401
    futures::future::join_all(vs).await; // await 别忘了
	// 结果 3 1 2
    Ok(())
}
```

# 运行时间

```rust
use tokio::time::Instant;

let t = Instant::now();                         // 开始计时
futures::future::join_all(vh).await;
tracing::error!("{}", t.elapsed().as_millis()); // 完成计时
```

# Semaphore

```rust
use tokio::sync::Semaphore;

lazy_static! {
    pub static ref COUNT: usize = 3;
    // 必须是 static
    pub static ref MAX_REQ: Semaphore = Semaphore::new(COUNT);
}

for i in 0..7 {
    let sh = MAX_REQ.acquire().await.unwrap();
    tokio::spawn(async move {
        println!("{}", i);
        tokio::time::sleep(tokio::time::Duration::from_secs(1)).await;
        drop(sh);
    });
}
// 先打印前3个数，一秒后打印其它的
```

# 设置线程数

```rust
// 强制 tokio 在一个线程上运行，这样多任务时不会被其它线程偷走执行
#[tokio::main(worker_threads = 1)]
async fn main() {
    tokio::spawn(async move { 
        eprintln!("task 1");
    	loop {} // task1 会一直执行，这样task2就没机会执行
    }); 
    tokio::spawn(async move { eprintln!("task 2"); });
}
```

# File

```rust
// 文件追加内容
let f = std::fs::OpenOptions::new()
	.write(true).create(true).append(true).open("a.txt")?;
let mut f = tokio::fs::File::from_std(f);

f.write(b"haha\n").await?;
```

# 测试

```toml
tokio-test = "0.4.2"
```

```rust
#[cfg(test)]
mod tests {
    #[tokio::test]
    async fn test1() {
        xxfn().await;
    }
}
```

# 收藏

```bash
https://skyao.io/learning-tokio/docs/introduction.html # 1.0中文文档
https://tokio.rs/tokio/tutorial # 官方文档
```

