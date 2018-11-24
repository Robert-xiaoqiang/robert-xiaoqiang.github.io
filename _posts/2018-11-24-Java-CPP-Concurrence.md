---
title: Java-CPP-Concurrence
key: 201811240
tags: Java CPP Threads
---


# Java-CPP-Concurrence
> 这里只讨论线程机制
> 即线程原语, 线程通信, 线程同步互斥
> 

### Java

#### 单一/任一线程有限状态机
- 继承`Thread`, `Override public void run()`创建对象, 将达到同如下的效果
- `new Thread(task).start() => New born state / ready state`(处于就绪队列, 据说采用多级优先级队列反馈调度, 抢占, 动态优先级/静态优先级)
- `Ready <=> Running` 
   - `=>`通过`t.run()`这个过程通过调度器调用
   - `<=` 通过`类方法Thead.yield()`让出CPU, 或调度时`swap out`
- `Running => finish/dead`一去不复返
   - task over, 善于死循环是一种开发模式
- `Running => Blocking`
   - `t.join(timeout)`caller thread将会陷入阻塞状态, 等待t结束返回就绪队列(有checked exception`InterruptedException`)
   - `Thread.sleep(long ms)`caller thread陷入阻塞状态, 等待毫秒计数后, 返回就绪队列(有checked exception`InterruptedException`)
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

#### Executor/thread pool 机制

#### JVM 与 Thread 与 杂项

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
template <typename F, typename... T>
explicit thread(F&& f, T&&... args);
```
#### 线程操作
- `std::this_thread::sleep_for(std::chrono::seconds(ms));`
- `t.detach()`
- `t.join()`
- `std::swap(t1, t2)`

- `合并属性, 分离属性`成员变量
   - `t.joinable()`默认构造, 执行完了, 不可合并.
- 其他运行时属性
   - `t.get_id()`返回std::thread::id 类型

#### 线程互斥
- 互斥体/互斥量/互斥锁std::mutex, 本身暴露为共享, 没毛病
   - `.try_lock() => bool`
   - `.lock()`
   - `.unlock()`
   - `std::<M>(M m)互斥量封装` RAII风格的局部作用域的临界区管理
- `std::timed_mutex`
   - `.try_lock_for( std::chrono::milliseconds d)`
   - `.try_lock_until(std::chrono::steady_clock::now() + d)`
- `recursive_*`
   - 类似嵌套synchronized section => trival