---
title: Java-CPP-Concurrence
key: 201811240
tags: Java CPP Threads
---


# Java-CPP-Concurrency
> 这里只讨论线程机制
> 即线程原语, 线程通信, 线程同步互斥
> 

### Java

#### 单一/任一线程有限状态机
- 继承`Thread`, `Override public void run()`创建对象, 将达到同如下的效果

<!--more-->

- `new Thread(task).start() => New born state / ready state`(处于就绪队列, 据说采用多级优先级队列反馈调度, 抢占, 动态优先级/静态优先级)
- `Ready <=> Running` 
   - `=>`通过`t.run()`这个过程通过调度器调用
   - `<=` 通过`类方法Thead.yield()`让出CPU, 或调度时`swap out`
- `Running => finish/dead`一去不复返
   - task over, 善于死循环是一种开发模式
- `Running => Blocking`
   - `t.join(timeout)`caller thread将会陷入阻塞状态, 等待t结束返回就绪队列(有checked exception`InterruptedException`)
   - `Thread.sleep(long ms)`caller thread陷入阻塞状态, 等待毫秒计数后, 返回就绪队列(有checked exception`InterruptedException`)
   - 被其他线程`interrupt() 后抛出异常, catch永远为false t.isInterrupted()`
   - `object.wait()`**前提是位于synchronized section获得对象的key**, caller thread陷入阻塞状态, 加入对象阻塞进程pool, 释放key, 等待signaled(object.notify()/.notifyAll()), 显然是有其他哥哥妹妹唤醒, 再次进入就绪队列等待调度
- `Blocking => Ready`如上所述
- deprecated

```Java
// ThreadDeath 捕获继续抛出, 线程才会真正终止
Thread.stop(); // task kill, resource? mutex? lock?
t.suspend();   // t/this被挂起不运行, JVM唤醒句柄, 只有此处唯一变量
x.resume(t);   // 唤醒挂起的t, by variable
```

#### 线程, 线程组的运行时刻信息
- `Thread.currentThread()返回句柄thread`类似System V POSIX `pthread_self()返回tid`
- `t.getName()`
- `t.getThreadGroup()`
- `t.getPriority()`
- `t.isAlive()`等价于`!t.isDead()`
- `t.isDaemon()`守护, 常驻进程, 系统服务进程
- `t.isInterrupted()`tg.isDaemon(), interrupt(), t.interrupt()

#### Executor/thread pool 机制
- `工厂Executors.new{Fixed, Cached}ThreadPool(1/INF Number) 子接口ExecutorService`
- `shutdown(Now), isShutdown(不可execute), isTerminated`
- why 类方法???
#### 同步
- shared前提 => 属性/变量
- `synchronized section/method(this)`
- `new ReentrantLock().newCondition()`
   - `await/signalAll/lock/unlock`
- `wait/notify`
- `new Semaphore() P() V() => acquire/release`
- `***BlockingQueue() for Consumer/Producer()`

#### JVM 与 Thread 与 杂项
- 根为系统线程组, 线程组才有父子, 单根概念, 资源过继概念, 儿子过继概念
- Main入口默认是一个线程组main, 一个main线程
- 默认在一个线程组内new Thread(), 将属于同一线程组
- 自己构造线程组, 而后, 可以在`new Thread(Group, task, name, stackSize)`


### CPP std::thread
#### 构造
- 拷贝控制成员
   - 无参构造 => 非线程的std::thread 对象
   - `thread(thread&& other) noexcept;`允许move constructor
   - `thread& operator=(thread&& other) noexcept;`允许move assignment
   - `thread(const thread& other) = delete noexcept;`
   - `thread& oparator=(const thread& other) = delete noexcept;`
   - `virtual ~thread();`
- 最常用构造函数(成员模板)

```C++
// std::function, 模板参数变长
// 内部实现可能std::forward转发
// 可能std::invoke(), 成员变量指针, 成员函数指针, 要手动传入this
// 非静态成员函数, std::bind(::*, obj *) <=> 捕获obj *
// 静态成员函数直接用
template <typename F, typename... T>
explicit thread(F&& f, T&&... args);
```

#### 线程操作
- `std::this_thread::sleep_for(std::chrono::seconds(s))`
- `std::this_thread::sleep_until()`
- `std::this_thread::yield()`
- `std::this_thread::get_id()`
- `t.detach()`
- `t.join()`
- `std::swap(t1, t2)`

- `合并属性, 分离属性`成员变量

   - `t.joinable()`默认构造, 执行完了, 不可合并.
- 其他运行时属性

   - `t.get_id()`返回std::thread::id 类型
- 得到线程返回值, 即异步I/O的需求, 对比POSIX, `pthread_join(id, res)`
   - 基础机制, `std::future<T>共享状态, 类型为T`
      - `.valid()`非阻塞, 推荐实时检查
      - `.wait(timeout)`部分阻塞
      - `.get()`阻塞
   - 第一级`std:promise<RET> 共享状态类型T`, `p.get_future()返回句柄`, 同时将此`promise<RET>`作为参数, 此时task无返回值`void task()`,  显然 =>线程构造函数多一个参数, task内部使用, `p.set_value/exception_at_thread_exit(value)`, 外部使用`future.get()`获取即可.
    > promise.set_exception(std::current_exception());
    > std::exception_ptr eptr = std::current_exception()
    > std::rethrow_exception(eptr);
    > pass by value is OK
    > 

   - 第二级`std::packaged_task<std::function>`, `pt.get_future()返回句柄`, 此时使用的task的函数有返回值`RET task()`, 内部正常返回, 外部使用`future.get()`获取即可
   - 第三级`std::future<std::result_of_t/std::invoke_result_t<std::decay_t<F>, Args...>> std::async(std::launch::async/deferred, F&&, Args&&... args)`简单的说, 将policy, 线程构造参数传入, 返回future句柄, 可获得异步I/O返回类型


#### 线程互斥同步
- 互斥体/互斥量/互斥锁std::mutex, 本身暴露为共享, 没毛病, 加锁临界区上下文
   - `.try_lock() => bool, 非阻塞`
   - `.lock()`
   - `.unlock()`
   - `std::lock_guard<std::Mutex>(M m, std::adopt_lock)互斥量封装` RAII风格的局部作用域的临界区管理, 无tag时阻塞
   - `std::unique_lock<std::Mutex>(M m, std::defer_lock不请求/try_to_lock/adopt_lock)互斥量封装` RAII风格的局部作用域的临界区管理, 更灵活, 构造式可选加锁, 可手动加锁, 释放, 无tag时阻塞`lock/unlock/owns_lock bool()`
   - `std::lock(), std::try_lock()`类似AND信号量, 避免死锁
- `std::timed_mutex`
   - `.try_lock_for( std::chrono::milliseconds d)`
   - `.try_lock_until(std::chrono::steady_clock::now() + d)`
- `recursive_*`
   - 类似嵌套synchronized section => trival可重入

- `std::condition_variable`, 必须配合`std::unique_lock<M>`使用, 默认tag即可, 注意手动解锁, 后发通知, 或退出作用域发通知
   - wait(lock, P = true); true就不用wait, while等待()逻辑比if()等待逻辑更安全
   - notify_on()/notify_all();
- `std::condition_variable`, 可配合`std::lock_gard<M>`

- 原子操作
   - `std::atomic<T> value`全局, 成员变量, 成员对象
   - 避免`Mutex, lock_gard`的使用

- 内存序, 指令序