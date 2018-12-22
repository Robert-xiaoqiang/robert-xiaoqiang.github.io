---
title: JavaLambda
key: 201812190
tags: Lambda Java
---

# Lambda

- 匿名内部类语法糖 (sugar)+ 强大类型推导
- 妥协于裸奔函数, 不会生成内部类`class`
- `java.util.function.* => FP`
- `Function类方法`
- `UnaryOperator<T>, BinaryOperator<T>.apply()`
- `UnaryPredict<T>, BinaryPredict<T>`
- `Supplier<T>返回值限制接口`
- `Consumer<T>参数限制接口`

<!--more-->

```Java
// 类型名不必要
// 单条语句, 不用return <=> CPP lambda 尾值返回类型省略
// 多表达式, 必须return <=> CPP 不可省略
i -> exp;
(i) -> exp;
(i) -> { exp; };
(i1, i2) -> { s1; s2; };
() -> { s1; s2; };
```

- Lambda与override, functional interface
   - 当且仅当一个方法, 继承此类, 实现此接口
   - 通常是泛型接口, 泛型抽象参数

- 变量/对象捕获
   - 局部变量 => static确定, 何以保证运行时 => copy, 同内部类语法, `final隐式/显示`
   - 成员函数, 成员函数调用, dynamic

# 流式计算与Lambda

- @note 以下stream非IO, 非NIO
- 流水化式进行, 内部循环一次而已
- 本质是注册若干函数接口`OP, Predict`, 最后一次注册执行返回, 当然保证注册顺序
- type[]/ Arrays => JCF
   - `Arrays.asList(函数参数变长)`本质new ArrayList<>(T...), 不可操作
- JCF => Arrays
   - toArray()
- JCF => stream
   - `Collection<T>.stream()/parallelStream()`
   - `Arrays.stream(type[])`
   - `Stream.of(函数参数变长)`
   - `BufferedReader.lines()`文本, 字符IO
   - `Files.walk()`NIO
   - `{Int, Long, Float, Boolean}Stream`, `range()`, `mapTo*`
   - `java.util.Spliterator并行拆分`
   - `Random.inits(), BitSet.stream(), Pattern.splitAsStream(), JarFile.stream()`
- stream => JCF(必要的.box()封装)
   - `stream.collect(Collectors.toList())`
   - `stream.collect(Collectors.toSet())`
   - `stream.collect(Collectors.toMap(UnaryOP<T>, UnaryOP<T>))`
- 流处理, 不一定连续进行, eager, lazy, 参数均为函数式接口
   - `filter(UnaryPredict<T>)` <=> std::copy_if
   - `long count()` lazy
   - `Object get()`
   - `map(UnaryOP<T>)` <=> std::transform
   - `reduce(initial, BinaryOP<T>)` <=> std::accumulate
   - `flatMap(UnaryOP<T> 返回流)`
   - `min(Java比较函数)`

```Java
Intermediate：lazy
map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

Terminal：eager
forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

Short-circuiting：无限流, 非阻塞
anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit   
```

# 标准库, JavaSE Logger与Lambda

# Pitfall & Issues
- Primitive
- @FunctionalInterface
- 函数式接口中可以有类方法多个(p, s), 类属性(p, f, s)
- FI多继承接口, 返回类型满足替代(relax in CPP), 签名满足子签名(泛型, 非泛型)
- 一旦实现不再是接口, 很显然, 使用`->覆盖`over, 且派生接口的体一般为空, 不能再添加虚方法
- 多继承与异常表
   - 具体化/交集, 或空集的异常表作为最终函数式接口的异常表
- `类型交集与强制`, 两个类型看成**多继承接口**的函数接口问题
- default函数 in Functional Interface, 有定义, 实现, 可以多个, 可以override基接口
- `Optional.of, ofNullable()`

# 关于内置FI
- `void Predict<T>.test(T)`
- `void Consumer<T>.accept(T)`
- `R Function<R, T>.apply(T)`
- `R BiFunction<R, T1, T2>.apply(T1, T2)`
- `T Supplier<T>.get()`生产者
- `UnaryOperator extends Function`
- `BinaryOperator extends BiFunction`

# 关于super, extends泛型bounded
- `T/? extends B` => 读, 删, 不可写, 修改
- `? super B` => 写,删 => 不可读, 修改
- T类型转换???, 数组???, 见之前
