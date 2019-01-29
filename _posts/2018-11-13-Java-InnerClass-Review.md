---
title: Java-InnerClass-Review
key: 201811130
tags: Java InnerClass
---

# Java-InnerClass-Review

### 局部(作用域)内部类, 可以final, abstract, 不能PPP, 不能static, 这点同局部变量
> final属性的初始化, **不可以**使用成员普适初始化
> 当内部类, 外部类之间属性/方法冲突时 => `O.this.* VS. *`
> 当内部类, 外部类 类属性冲突 => `O.* trival`
> 

- 局部作用域对局部内部类
   - 访问权限, 类属性/方法, 属性/方法使然
- 局部内部类对外
   - 取决于位置, 是否可以拿到`this`,  将拿到所有成员,  一定可以拿到静态成员
   - `this`相当于`final`, `CPP => T *const`
   - 对于其他变量, 局部作用域, 隐式要求`final => why => copy JVM`
- 局部内部类本身
   - 属性
   - 类属性, 必须final, 且`contexpr`
   - 方法
   - 类方法, 禁止(只能在top-level 或 static-level)
   - 静态初始化块, 禁止
   - 内部interface / enum, 禁止(只能在top-level 或 static-level)
   - 可以继续类内部类只能final/abstract, 不能static

<!--more-->

### 匿名内部类, 当然同上, 同局部内部类, 无静态/非静态之分
> 特点无构造函数, 实现接口, 继承基类
> 方法的访问权限设置public @Override
> 其他访问权限设置, 极其容易成为SB, SB, SB
> 当内部类, 外部类之间属性/方法冲突时 => `O.this.* VS. *`
> 当内部类, 外部类 类属性冲突 => `O.* trival`
>

- 外对内
   - 同局部内部类

- 内对外
   - 同局部内部类

- 内部类本身
   - 同局部内部类

### 类内部类, 非static, 可以PPP, 可以final
> 特点在于内部类对象一定关联一个外部类对象
> 类外类型表示 O.I
> 对象构造 O.I i = O.new O.I()
> 非静态方法内new()
> 

---
> 当内部类, 外部类之间属性/方法冲突时 => `O.this.* VS. *`
> 当内部类, 外部类 类属性冲突 => `O.* trival`
> 

- 外对内
   - 访问权限, 类属性/方法, 属性/方法使然

- 内对外
   - 可以拿到`this`, 可以访问所有外部类成员(属性+方法, 静态+非静态)

- 内部类本身
   - 属性
   - 类属性, 必须final, 且`contexpr`
   - 方法
   - 类方法, 禁止(只能在top-level 或 static-level)
   - 静态初始化块, 禁止
   - 内部interface / enum, 禁止(只能在top-level 或 static-level)
   - 可以继续类内部类只能final/abstract, 不能static


### 类内部类, static, 可以PPP, 可以final
> 特点在于内部类对象与外部类对象无关
> 类外类型表示 O.I
> 对象构造 O.I i = new O.I()

---
> 当内部类, 外部类 类属性冲突 => `O.* trival`
> 当内部类, 外部类 类方法冲突 => `O.* trival`
> 

- 外对内
   - 访问权限, 类属性/方法, 属性/方法使然

- 内对外
   - 类属性
   - 类方法

- 内部类本身
   - 属性
   - 类属性,  无限制
   - 方法
   - 类方法,  无限制
   - 静态初始化块, 允许
   - 内部interface / enum, 允许, 且修饰PPP
   - 可以继续**类内部类**- 可以继续类内部类只能final/abstract, **也能static**



### 内部类继承杂项
- CPP override 3item
- Java @Override 2item, 为什么Java的Annotation是大写的, 为什么, 为什么, 为什么, 太丑陋了, 太丑陋了, 太丑陋了
- 内部类覆盖是不存在的
   - 可能分别外部与外部之间, 内部与内部之间覆盖
- 非静态内部类之间在内部类之间继承
   - so far, so good
- 非静态内部类暴露除去用于继承`class Foo extends O.I{ }`, 构造函数第一参为外部类对象, 第一句为`oo.super()` => `无法委托, 无法super()` => `这无疑是傻逼之举`
- 对比静态内部类 => 无任何区别 => `top level == static context(成员限定符)`
   - 类方法, 无限制类属性
   - static初始化块
   - 内部接口, 内部枚举
   - static(类)内部类
   - 可能有更优雅的概括, 可我没发现
- 接口内部可以有接口, 可以枚举(极其有用), 可以有static内部类, 也可以有非静态内部类,  可以类方法且必须给定义, 类属性, default 方法(不可static) =>大都隐式public, 可选final/abstract, strictfp
   - **public abstract, public final static** 足矣, 特例在于接口的**类方法**可以private, 不能protected
   - Java只有public继承
- 接口不可以静态初始化块, 初始化块, why???
- **接口位于top level只能public, friend(trival), 接口内的成员只能public, 接口作为成员PPP完全可以, 注意: 当与上述不冲突时**

