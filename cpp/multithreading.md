# MultiThreading

## 概念

* 并发（concurrency）：多个活动在同一时间发生。
* 任务并行（parallelism）：在多核处理器或多处理器上，任务并行执行，实现并发。
* 任务切换（task switching）：在单核处理器上，任务进行快速切换，产生并发的错觉。

## 多线程

多线程在多处理器上运行时，优点是更安全的并发代码（操作系统提供处理器隔离保护）、不阻塞；缺点是通信成本、维护成本。\
多线程在单处理器上运行时，则相反。

### 原因

* 关注点分离
* 性能提升

但也可能带来复杂性，亦或者使程序变慢，根据场景考虑。

### 线程通信

* 信号
* sockets
* 文件
* 管道

## thread.h

程序启动时的初始线程，执行`main`函数。创建新的线程需要提供初始执行函数（函数、函数对象、lambda函数），直到该函数最终返回，线程才结束。

创建线程后，未调用`join`或`detach`\
线程对象销毁时，会调用`std::terminate`结束进程。解决方案有scoped_thread。

创建线程后，调用`join`\
阻塞并等待线程结束。

创建线程后，调用`detach`\
新建线程独立运行。

### 参数

拷贝值或者引用（不推荐）

### 共享数据

* 数据竞争：多个线程同时访问同一数据，至少一个线程为写操作。
* 关键区域：同一时刻只允许一个线程进入的代码块。

mutex\
互斥量，只允许一个线程拥有它，其他线程需等待解锁。

```cpp
std::mutex m;
m.lock();
doSomething();
m.unlock();
```

lock_guard\
构造时自动上锁，解构时自动解锁。

```cpp
{
  std::mutex m,
  std::lock_guard<std::mutex> lockGuard(m);
  doSomething();
}
```

unique_lock\
死锁产生的原因是多个互斥量以不同的顺序被上锁；关键区域中需要锁。

```cpp
std::unique_lock<std::mutex> guard1(a.mut, std::defer_lock);
std::unique_lock<std::mutex> guard2(b.mut, std::defer_lock);
std::lock(guard1, guard2);
```

shared_lock\
读写锁，允许多个线程同时读，只允许一个线程写。

```cpp
std::shared_lock<std::shared_timed_mutex> readerLock(m);

std::lock_guard<std::shared_timed_mutex> writerLock(m);
```
