---
title: Java-Generic
key: 201810280
tags: Java Generic
---

# Java-Generic

### 泛型类, 接口
- 本质是引入`一个或多个T`

### 泛型方法
- 本质是引入`T`或者`wild crads`
- `bounded/unbounded T`
- `bounded/unbounded WC`

### 关于泛型与继承, 能否引用的问题(类型声明的变量 + 创建的对象)
- `B[] = new D[]{}` 正确, 带来了风险
> 此处`{ }`可用于数组初始化, Java容器初始化, 其他情况`initializer_list<T>类模板`???


- `B = new D()` 正确
- `GPB<E> = new GPD<E>()`正确
- `GPB<E1> = new GPD<E2>()`错误
   - 解决方式`GPB<?> = new GPD<E2>()`
   - 或者`GPB<? extends E2> = new GPD<E2>()` 看似很怪,但正确, 原因如下
- `GP<B> = new GP<D>()` 错误
   - 解决方式 `GP<?> = new GP<D>()`
   - 或者 `GP<? extends B> = new GP<D>()`
   - 或者`GP<? extends D> = new GP<D>()`看似很怪,但正确, 原因如下

### 关于WC, UWC, BWC
- 元素的`WC` <=> `Object类型`, 而`GP`应用之一是`容器元素` => trival
- 无法进行插入, 即`add(), put(), set()`
- 可以`remove(), get(), iterator<E>同理`
- 是声明, 是deduce, 是infer => 不能用于定义或`new`

### 关于泛型与非泛型
- 非泛型 -> 泛型, 可能略去很复杂的过程, 也可能加`T`或`Object`
- 泛型->非泛型, 或(非泛型基类), 爽!

### 关于泛型数组
- 定义泛型参数数组, 错误
- 定义泛型参数为`T`(非`bounded WC`), 错误
- 上述做变量/引用/参数声明, 正确
- 原因是, 数组向基类数组`cast后`, 可以实现[插入](#关于WC, UWC, BWC), 再次`向上cast`后, 是默认`T`, 还是默认`Object`的问题

### Other
- 枚举, 异常, 不可泛型
- 可继承泛型类, 实现泛型接口, 种种内部类可泛型, 内部接口可泛型
- `instanceof`不可作用泛型

### Supplement
- 内部接口
   类内部接口, 接口内部接口, `Top Level`, `static block`, 正确, 其余位置错误
- `接口`组织`多个枚举`

