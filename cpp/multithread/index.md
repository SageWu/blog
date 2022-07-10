# 多线程

## 概念

* 并发（concurrency）：多个活动在同一时间发生，需要等待共享资源。
* 并行（parallelism）：多个活动在同一瞬间发生。

## 线程通信

* 信号
* sockets
* 文件
* 管道

## thread.h

程序启动时的主线程，执行`main`函数。\
创建新的线程需要提供执行函数（function、lambda、functor、member function、static function），直到该函数最终返回，线程才结束。

### 执行情况

创建线程后，未调用`join`或`detach`。线程对象销毁时，会调用`std::terminate`结束进程。解决方案有scoped_thread。\
创建线程后，调用`join`。阻塞并等待线程结束。\
创建线程后，调用`detach`。新建线程独立运行。

### 参数

拷贝值或者引用（不推荐）

### 共享数据

* race condition：多个线程同时访问同一数据，至少一个线程为写操作。
* critical section：同一时刻只允许一个线程进入的代码块。

mutex\
互斥量，只允许一个线程拥有它，其他线程需等待解锁。

```cpp
std::mutex m;
m.lock(); // 阻塞调用
doSomething();
m.unlock();

m.try_lock(); // 非阻塞调用

std::try_lock(m, ...);  // 获取多个锁
```

timed_mutex\
获取锁时，可以指定阻塞超时。

```cpp
std::timed_mutex m;
m.try_lock_for(time);
m.try_lock_until(time);
```

recursive_mutex\
同一线程可以多次获取锁，但也需要多次解锁。

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
