---
title: CPP-template-advanced-continuation
key: 201809050
tags: CPP template
---

# CPP-template-advanced-continuation

- T(args...) 构造 => 匿名对象是(非const)右值, 类似函数**对象返回** => 难以体现右值性(与对象相关的右值) 
- 什么是右值 => exp, 函数返回, 匿名构造, 运算符相关
- 什么是左值 => 类型声明, 实例化对象
- 右值/左值差别 => 语义(semantic)相关, 运算符(operator)相关, 小命长短相关
- 注意const右值 当且仅当对于对象才有意义
- const右值引用也存在, 可以绑定const/非const右值
>只是为了语法的完整性而存在, 无法触发真正的移动语义 => 因为无法进行steal operation
>如move constructor, move assignment, std::move()返回无意义啊
>将只会触发copy assignment, 即 const T& 范式
>


#### `std::move()`的左值实参与右值实参详解

```CPP
template <typename T>
typename remove_reference<T>::type&& move(T&& t)
{
    return static_cast<typename remove_reference<T>::type&&>(t);
}

string s1 = "hi!", s2;
s2 = std::move(string("bye!")); //1 构造返回右值 => string move assignment
s2 = std::move(s1)              //2 string move assignment
```

- 对于#1
>T = string
>函数模板实例 string&& move(string&& t); => 右值引用绑定右值
>trival cast
>

- 对于#2
>T = string&
>函数模板实例 string&& move(string& t); => 左值引用绑定左值
>static_cast<string&&>(t) 是合法的
>即使用C++强制类型转换允许将左值引用/绑定后的左值截断(clopper/expire lvalue)为右值
>这意味着左值的资源管理权限已经交出, 小命玩儿完了! 移后源必会析构, 值不确定
>特例 cast
>

- 显然当实参有const限定时, std::move()的const右值引用返回任何意义啊, 与之前讨论相同

- `注意/结论: 放心使用std::move(非const l/r value)即可, 放心移动语义, 放心截断左值, 此处的特例自己少用啊! 当然实参const限定时DSB啊!`

#### 转发