存储层次从上到下：寄存器 -> 缓存 -> 内存 -> SSD -> 磁盘 -> 光盘 -> 磁带

- 前三个为易失性存储，断电就没
- 存储量越来越大、存储速度越来越慢

> 名词解释

异步 == 随机，UI == user interface，CLI == command line interface，GUI == graphic user interface

> <h4>操作系统提供的服务</h4>

面向普通用户接口：GUI(图形化用户接口)、batch(批处理)、command line(命令行)

面向应用程序(程序员{通过API间接进行系统调用}、或叫系统调用)接口：

- 程序执行
- 对IO设备的操作
- 文件系统
- 进程间通信
- 资源分配
- 记账：记录某一资源用了多久、存储量多少
- 错误系统
- 防止恶意代码滥用系统资源

> 一段程序中的 user mode 和 kernel mode

```c
int main() {
    int a = 1;
    printf("%d", a); // 在调用系统api之前，都是 user mode，一旦调用api，
                     // 就是切换到 kernel mode，系统调用完成后再切回 user mode
}
```

每个系统调用都有个唯一的 `系统调用号`，OS 根据这个号找到该系统调用代码在内核中的位置



# 进程

当一个程序（可执行文件）被加载到内存，它就变成了**进程**

**程序计数器**：是CPU中的一个寄存器，里面存放下一条要执行指令的内存地址，在x86中叫它**指令指针**、**指令地址寄存器**、**指令计数器**

## 内存中的进程

PCB：进程控制块（看下面）

stack：栈。用于存入局部变量、函数返回地址

heap：堆。用于程序运行时的动态内存分配

data：全局或静态变量数据

text：二进制机器码

```c
// 用一段代码来说明
int global = 100;

void f(int x, int y) { int* p = malloc(100); }
void g(int a) { f(a, a + 1); }
int main() {
    static int i = 10;
    g(i);
}
```

```bash
# 上面代码在内存中的表现形式

# stack：
# | void f()   |
# | int x,y,*p |
# | void g()   |
# | int a      |
# | main()     |

# heap：
# | int* p = 100 | 因为是malloc的，所以需要手动释放

# data：
# | int global = 100  |
# | static int i = 10 |

# test：
# | 所有代码编译成的二进制机器码 |
```

## 进程状态

进程在执行期间，其自身状态会发生三种变化：

- 运行态（Running）：此时代码在CPU上运行
- 就绪态（Ready）：进程具备运行条件，等待分配CPU
- 等待态（Waiting）：进程不具备运行条件，在等待某些事件发生（如IO操作结束）

## 进程何时放弃CPU

内部事件：进程主动放弃（yield）CPU，进入等待/终止状态（如使用IO设备，非正常结束）

外部事件：进程被剥夺CPU使用权，进入就绪状态，这个动作叫**抢占**(preempt)。如时间片到达，高优先权进程到达

## 进程切换

并发进程切换：并发进程中，一个正执行的进程可能被另一个进程抢占CPU，这个过程叫**进程切换**

## 中断源

外中断：来自处理器之外的硬件中断信号

- 如时钟中断、键盘中断、外围设备中断
- 外部中断均为异步中断

内中断（异常 Exception）：来自于处理器内部，指令执行过程中发生的中断，属同步中断

- 硬件异常：掉电、奇偶校验错误
- 程序异常：非法操作、地址越界、断点、除数为0
- 系统调用

## 特权/非特权指令

特权：只能运行在 `kernel mode` 下的指定

- IO指令或停止指令
- 关闭所有中断（意味着不再响应任何事件）
- 时钟
- 进程切换
  - 切换时机
    - 进程需要进入等待状态
    - 进程被抢占CPU而进入就绪状态
  - 切换过程
    - 保存被中断进程的**上下文信息**（Context）
    - 修改被中断进程的**控制信息**（如状态等）
    - 将被中断的进程加入相应的**状态队列**
    - 调度一个新进程并恢复它的上下文信息

非特权：只能运行在 `user mode` 下的指令

## 模式切换

中断是用户态（user mode）向内核态（kernel mode）转换的唯一途径，系统调用实质上也是一种中断

OS提供 Load PSW 指令装载用户进程返回状态

## 进程控制块

每个进程都有一个 PCB（process control block），其中包含：

- state（进程状态）：就绪、运行还是等待
- number（进程号）：唯一编号，即进程号PID
- counter：保存下一条进程要执行指令的内存地址
- register：在CPU上运行时，用了哪些寄存器，以及这些寄存器的值
- memory limits：保存一些和内存相关的信息
- files（打开的文件）

## 进程调度

进程的整个生命周期，会通过调度器的调度，在各个调度队列间迁移



# 线程

在进程内存中，每个线程都有各自的 `thread id`、`register`、`counter` 、`stack`，而 `data`、`text`、`files` 是共享的。

现代操作系统的系统调度是以线程为单位

## 多核编程

在多处理器系统中，多核编程让应用程序可以将自身多个任务分散到不同处理上运行，以实现**并行**计算

## 多线程模型

用户线程ULT（user level thread）：在用户态（user mode）下运行，不需要内核管理

内核线程KLT（kernel level thread）：在内核态（kernel mode）下运行，由OS支持与管理

M:1模型：多个用户线程对应一个内核线程

1:1模型：一个用户线程对应一个内核线程

M:M模型：多个用户线程对应多个内核线程

## 线程库

POSIX Pthreads：用户线程库和内核线程库

Windows Threads：内核线程库

Java Threads：依赖其运行的操作系统

# CPU调度

非抢占式调度：一直执行到结束才让出CPU

抢占式调度：未执行完便让出CPU

## 调度性能衡量

CPU利用率：CPU的忙碌程度

响应时间：从提交任务到第一次响应时间

等待时间：进程累积在就绪队列中等待的时间

周转时间：从提交到完成的时间

吞吐率：每个时钟单位处理的任务数

公平性：以合理的方式让各个进程共享CPU

## 调度算法

先来先服务（FCFS）：非抢占式调度

- 优点：实现简单
- 缺点：当短任务排在了长任务后面，会导致周转时间变长

时间片轮转（ROUND ROBIN）：抢占式调度，每个进程都可以得到相同的CPU时间，当时间片到达，进程将被剥夺CPU并加入就绪队列尾部



# Api

所有的api可通过`man`查看用法及包含的头文件

## 进程

### getpid

返回当前进程号。rust：`unsafe { libc::getpid() }`

### fork

创建一个当前进程的子进程，成功返回 pid，失败则返回-1，被创建的子进程返回 0。rust：`unsafe { libc::fork() }`

fork 后的父子进程是异步执行的

windows下调该方法报错，因为没有

```rust
unsafe {
    println!("before fork pid: {}", libc::getpid());
    let cid: libc::c_int = libc::fork();
    println!("cid: {}", cid);
    println!("after fork pid: {}", libc::getpid());
    libc::wait(std::ptr::null_mut()); // 等待子进程结束，c里的 NULL == std::ptr::null_mut()
} // 结果：
// before fork pid: 563 父进程的结果
// cid: 584             父进程的结果，正常返回pid
// after fork pid: 563  父进程的结果
// cid: 0               子进程的结果，返回 0
// after fork pid: 584  子进程的结果
```

#### 孤儿进程

当父进程 fork 了个子进程后就结束了，但子进程还没结束，此时子进程就成了**孤儿进程**。

孤儿进程会托管给系统进程（pid=1）

### wait

等待fork的子进程结束，`wait(NULL)` == `libc::wait(std::ptr::null_mut())`

### sleep

`sleep(3)` 睡3秒

## 线程

### pthread_create

创建一个线程，创建成功后x86_64默认分配2M的栈大小

==注意==：由于 `pthread` 不是 linux 系统默认库，连接时需要静态库 `libpthread.a`，所以编译时要加 ==`-lpthread`或`-pthread(推荐)`==，如：

==`gcc thr.c -o thr -pthread`==

```c
// 返回值：成功0，失败返回错误码
int pthread_create(
    pthread_t *thread,               // 成功后返回thread id
    const pthread_attr_t *attr,      // 线程属性，可以给NULL
    void *(*start_routine) (void *), // 执行线程的函数指针
    void *arg                        // 传给线程的参数，可以给NULL
);
```

### pthread_join

等待线程结束

```c
// 成功返回0，失败返回错误号
int pthread_join(pthread_t thread, void **retval); // arg1：要等待的线程id，arg2：可以NULL
```

### pthread_exit

`pthread_exit(0);` 正常结束返回一个线程

### 例子

> 创建、等待、传参

```c
#include <stdio.h>
#include <pthread.h>

void* fn(void* arg) {
    printf("thread run fn, paramter: %d\n", *((int*)arg));
}

int main() {
    int p = 7;
    pthread_t tid;
    pthread_create(&tid, NULL, fn, (void*)&p);
    pthread_join(tid, NULL); // 结果：thread run fn, paramter: 7
}
```





https://www.bilibili.com/video/BV1bf4y147PZ?p=10&vd_source=e611dc7ed99505a0ef548dd66f0a11dd  1:02:22