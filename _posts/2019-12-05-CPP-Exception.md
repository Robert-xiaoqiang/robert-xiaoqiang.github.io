---
title: CPP-Exception
key: 201912050
tags: CPP Exception
---

# CPP-Exception

## some moral or ethics

```C++
// Example 2(b): Very Buggy Class
//
class X : Y {
  T* t_;
  Z* z_;
public:
  X()
  try
    : Y(1)
    , t_( new T( static_cast<Y*>(this) )
    , z_( new Z( static_cast<Y*>(this), t_ ) )
  {
    /*...*/
  }
  catch(...)
  // Y::Y or T::T or Z::Z or X::X's body has thrown
  {
    // Q: should I delete t_ or z_? (note: not legal C++)
  }
};
```



Therefore the status quo can be summarized as follows:

> **Moral #1:** Constructor function-try-block handlers have only one purpose -- to translate an exception. (And maybe to do logging or some other side effects.) They are not useful for any other purpose.
>
> **Moral #2:** Since destructors should never emit an exception, destructor function-try-blocks have no practical use at all.**[[6\]](http://www.gotw.ca/gotw/066.htm#6)** There should never be anything for them to detect, and even if there were something to detect because of evil code, the handler is not very useful for doing anything about it because it can not suppress the exception.
>
> **Moral #3:** Always perform unmanaged resource acquisition in the constructor body, never in initializer lists. In other words, either use "resource acquisition is initialization" (thereby avoiding unmanaged resources entirely) or else perform the resource acquisition in the constructor body.

For example, building on Example 2(b), say T was char and t_ was a plain old char* that was new[]'d in the initializer-list; then in the handler there would be no way to delete[] it. The fix would be to instead either wrap the dynamically allocated memory resource (e.g., change char* to string) or new[] it in the constructor body where it can be safely cleaned up using a local try-block or otherwise.

> **Moral #4:** Always clean up unmanaged resource acquisition in local try-block handlers within the constructor or destructor body, never in constructor or destructor function-try-block handlers.
>
> **Moral #5:** If a constructor has an exception specification, that exception specification must allow for the union of all possible exceptions that could be thrown by base and member subobjects. As Holmes might add, "It really must, you know." (Indeed, this is the way that the implicitly generated constructors are declared; see [GotW #69](http://www.gotw.ca/gotw/069.htm).)
>
> **Moral #6:** If a constructor of a member object can throw but you can get along without said member, hold it by pointer and use the pointer's nullness to remember whether you've got one or not, as usual. Use the Pimpl idiom to group such "optional" members so you only have to allocate once.

And finally, one last moral that overlaps with the above but is worth restating in its own right:

> **Moral #7:** Prefer using "resource acquisition is initialization" to manage resources. Really, really, really. It will save you more headaches than you can probably imagine.



<!--more-->



## In Chinese

- 异常与ctor, 异常与dtor, 本质是一回事, 都基于RAII(异常与操作即try/catch handler or sub-clause)

  - try 子句退出编译器会自动调用异构
  - function-try-block(即try-catch 包上构造析构的正确姿势), 对于成员变量/对象, 写不写进try-block, 已构造的对象, 编译器都会自动调用析构, 包括基类对象(与成员变量/对象基本没差别)
  - function-try-block 不会调用`*this`当前对象的析构, 因为
  > there's nothing to destroy. "It cannot die, for it never lived." Note that this makes the phrase "an object whose constructor threw an exception" really an oxymoron. Such a thing is even less than an ex-object... it never lived, never was, never breathed its first

   - 也因此, function-try-block只用于异常转发/转换/翻译, 构造发生异常(即成员或者基类发生异常), 构造函数只能`fail`, 只能`rethrow or throw new exception`

- (处理资源 分配与异常 的关系 的)最佳实践: **构造用**(in construction): function-try-block + local-try-block + 智能指针对象(也可使用对象, 使用类的dtor), **操作用**(in operation): local-try-block + 智能指针对象(也可使用对象, 使用类的dtor)

  - C++11之前使用两阶段策略

    - stack内存初始化, 不会有任何异常
    - 资源逐个初始化try-catch测试, true-false测试, 得到构造成功与否的flag/tag
    - 逐个按照**构造的完成度**进行析构, 顺理成章
    - 注意, 此时不可以使用function-try-block, 因为function-try-block只用于异常转发/转换/翻译, catch handler中不能引用任何非静态局部变量/对象
- 析构一定不可抛异常, 即使`evil code`, 抛 -> `std::terminate 必然事件`, 不抛 -> 异常stacktrace or stacklist `UB`

## reference
[GotW](http://www.gotw.ca/gotw/066.htm)

​    

​    