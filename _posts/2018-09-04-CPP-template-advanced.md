---
title: CPP-template-advanced
key: 201809040
tags: CPP template
---

# CPP-template-advanced

#### 成员模板
- 成员函数模板
   - 类成员 => 由函数模板参数确定函数实例
   - 模板成员 =>2套前缀 由类模板参数确定类实例, 函数模板参数确定函数实例
   > 1套前缀时, 类模板的成员一定是函数模板(即成员模板)
   > 

<!--more-->

#### 控制实例化, 显式实例化
> 解决的问题:
> 分离编译的文件使用模板时分别进行实例化
> 相同的模板实例存在于多个文件
> 

- 显式实例化定义
> 一句话, 要定死某个实例

```CPP
template class GT<args...>;     // 类模板
template float f(int a, int b); // 函数模板
```

- 显示实例化声明
> 一句话, extern声明
> 

```CPP
extern template class GT<args...>;  // 类模板
extern float f(int a, int b);       // 函数模板
```

- 注意
- `显示实例化定义后, 类模板的所有成员(成员模板)一定会都进行实例化, 而一般的模板实例化, 能保证有使用之处才进行成员实例化`

#### 函数模板类型转换
- 顶层`const`忽略 trival
- 允许`非const`初始化`const` trival
- 数组到元素指针左值, 函数到函数指针
- 数组绑定到数组引用无事, 绑定到元素引用GG+ERROR 注意

- 注意 `函数模板中的(不使用模板前缀的参数)应用正常的类型转换`
- `显式模板参数对应的位置`应用正常的类型转换
- 显式模板参数规则`从左向右, 最右侧可以省略, 应用编译器参数/类型推导`
- 参数推导过程, 即相同即相同, 不相同即不相同, 从左向右
> 某种程度上, 函数模板显式模板参数已经定死参数类型, 可以进行较为灵活的类型转换
> 

- 有函数指针进行函数模板参数推导时, 加之重载, 可能需要显式函数模板参数

```CPP
// 函数声明
void func(int (*)(const string&, const string&));
void func(int (*)(const int&, const int&)); // 重载

// 函数模板前置声明
template <typename T>
int compare(const T&, const T&);

// Error
func(compare);

// Yep
func(compare<int>);
```

#### auto 类型指示符(type specifier)
- 定义一定要初始化, 类型推导
- 逗号连续声明, 初始化, 推导必须相同
- 以下为推导结果的详解
- 去引用 => trival
- 去顶层const => 保留底层const
- 加引用时, 即auto& 时会保留顶层const

#### decltype(exp) 类型指示符(type specifier)
- 推导后, 不求值, 可以不进行对象定义, 可以进行尾值返回类型声明
- 作用于单个变量/对象 => 完全类型+可选CV+引用+l/rvalue=> 必要初始化动作
- 作用于表达式 => 表达式结果/值的类型 => `r` `r + 0` `*p`
- 特例: `decltype((i))` => ()是优先级 => 特殊EXP => 永远T&返回

#### 实际类型与模板形参T(前缀)本质不同 => T多了推导环节, T中可能含有CV
- 左值引用的函数模板参数的类型推导
- 禅 法门 公理
> 与一般意义的绑定规则一致, const(l/r value)引用永远是底层的(加在对象上的)
> 参数T可以推导出const限定符, 但本身咩有const限定
> 

```CPP
// 函数模板前置声明
template <typename T>
void f1(T&);

int i;
const int ci;

f1(i);  // T == int
f1(ci); // T == const int
f1(0);  // Error! 无法绑定右值
```

`注意理解: 此时表面上看好像左值引用可以绑定const左值, 而实际是参数T推导出了const qualifier => 即T& 既可以看做左值引用, 也可以看做const左值引用(当且仅当面临const实参进行推导的时候)`


- const左值引用的函数模板参数的类型推导
> 可以绑定任何东西, 本身不会推导出const
> 

```CPP
// 函数模板前置声明
template <typename T>
void f2(const T&);

int i;
const int ci;

f2(i);  // T == int
f2(ci); // T == int
f2(0);  // T == int
```

- 右值引用的函数模板参数的类型推导
> 理想情况只能绑定右值, 但一类比, 也能绑定const右值 => 原因是T推导出const
> const右值引用能绑定const/非const右值, 如下一篇blog详细讨论
> 但如下的特例规则是极其美妙的

- 类似的const右值引用函数模板模板参数的类型推导也存在, 只不过`T本身`不会推导出const, 其余同右值引用推导 => 基本没什么用, 如下一篇blog详细讨论

```CPP
template <typename T> void f3(T&&);
f3(0.0f); // T == float
```

- 特例规则1 => 好像右值引用绑定左值 => WOC => 666

```CPP
template <typename T> void f3(T&&);
f3(0.0f); // T == float

float fp = 0.0F;
const float cfp = 0.0f;
f3(fp);   // T == float& => float& && 折叠为左值引用
f3(cfp);  // T == const float& => 折叠后的const左值引用绑定const左值
```

- 特例规则2 => 引用折叠(引用的引用)形成后, 即可进行折叠为左值引用(除去一种特例折叠为右值引用)
> X& &, X& &&, X&& & => X&
> 

- **或**

> X&& &&                      => X&&
> X 有无const无所谓
> 

- 注意引用的引用(引用折叠)无法直接的用声明语法写出, 但可用于
- typedef/using alias声明
- F/C模板参数T的使用 + 推导
- **注意:** 真正使用的实例就是实例化后的结果

- `右值引用函数模板 模板参数推导带来的问题是实现的代码中难以兼顾值类型或引用类型, 此种特技常用于模板转发其实参 或 模板被重载 等待下文`

- `<utility> std::move()`
- 将const/非const左值绑定到右值 函数模板
- How?

`typename 前置 => 模板默认解析static数据成员, 显示类型成员(类型/类型alias)`
`template 特殊解析了解一下`

```CPP
// std::move 实现
template <typename T>
typename remove_reference<T>::type&& move(T&& t)
{
    return static_cast<typename remove_reference<T>::type&&>(t);
}

```
