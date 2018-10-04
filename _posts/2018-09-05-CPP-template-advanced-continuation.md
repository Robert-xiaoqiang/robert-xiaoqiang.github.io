---
title: CPP-template-advanced-continuation
key: 201809050
tags: CPP template
---

# CPP-template-advanced-continuation

- T(args...) 构造 => 匿名对象是(非const)右值, 类似函数**对象返回** => 难以体现右值性(与对象相关的右值) 
- 什么是右值 => 任何`exp(除了[], *解引用)`值(operator相关), 函数返回值, 匿名构造
- 什么是左值 => 类型声明, 实例化对象
- 右值/左值差别 => 语义(semantic)相关, 运算符(operator)相关, 小命长短相关
- 注意const右值 当且仅当对于对象才有意义
- const右值引用也存在, 可以绑定const/非const右值
>只是为了语法的完整性而存在, 无法触发真正的移动语义 => 因为无法进行steal operation => 进而没B用
>如move constructor, move assignment, std::move()返回无意义啊
>将只会触发copy assignment, 即 const T& 范式
>

<!--more-->

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

#### 拷贝控制成员的编译器自动生成(synthesized)简述
- trival
   - 构造/赋值操作是`memberwise`进行的, 包括(非静态)成员和直接/间接基类
   - 不能`构造/赋值操作`的成员(刺头成员) 或 直接间接虚基类引发定义删除
   - 程序员自定义`拷贝控制成员`引发删除, 但是可以**强制**
   - 不能析构, 导致移动语义定义`move constructor/assignment`删除
   - `=default`具有**显式**和**强制**双重意义
- 没有任何构造时 => 生成无参构造, 定义是trival进行的, 否则声明为`=delete`
- `copy/move assignment`自动生成trival, const成员引发删除, 只能自定义
- `copy/move constructor`自动生成trival, `const`成员变量, 无类内初始值, 引发删除
- 有`move constructor/assignment`声明时, `copy constructor/assignment`定义删除 => 原因是为了移动语义正确执行啊!

#### 转发或参数传递
>解决的问题是, 函数模板将参数再次传递时
>左值, 右值对象属性难以保证
>const限定符难以保证
>

- 先再次理解右值引用
   - 绑定右值后
   - 对外是移后源一定会析构, 值不确定
   - 对内, alias是**自由修改的左值**, 是左值, 事实正是如此
- 同理const右值引用
   - 不光无法实现对外的移后源保证
   - 而且alias是const左值 => read-only, 事实真是如此
   - 结果还是没B用
- 对应的左值引用, const左值引用trival => 很自然 => 即左值, const左值

```CPP
template<typename F, typename T1, typename T2>
void flip1(F f, T1 t1, T2 t2)
{
    f(t2, t1);
}

template<typename F, typename T1, typename T2>
void flip2(F f, T1&& t1, T2&& t2)
{
    f(t2, t1);
}

template<typename F, typename T1, typename T2>
void flip3(F f, T1&& t1, T2&& t2)
{
    f(std::forward<T1>(t2), std::forward<T2>(t1));
}

void f(int v1, int& v2);
int i;
flip1(f, i, 0);
/*
实例是 flip1(void(*)(int, int&), int, int);
i永远不会变
*/

flip2(f, i, 0);
/*
实例是 flip2(void(*)(int, int&), int&, int&&);
i会变, 显然, 而且会保持const左值/右值属性
i的变化过程是(左值 => 绑定到左值引用 => alias左值 => f调用)
但是0的变化过程是(右值 => 绑定到右值引用 => alias左值 => f调用)
问题是: 无法一直保持右值引用的特性
*/
void g(int&& v1, int& v2);
flip2(g, i, 0); // Error! 右值引用无法绑定左值lvalue

flip3(g, i, 0); // so good!
```

- `std::forward<T>()`常用来转发右值引用, 且显示模板参数为函数模板模板参数T, 与右值引用函数模板模板参数推导`T&&`配合使用 => 完美转发

#### 函数模板重载
- 函数模板之间, 函数之间, 函数与函数模板之间构成重载关系的条件相同
>即: 不看返回值, 只看参数类型, 个数, const, throw()声明符是否相同
>注意: 函数模板之间的重载可以使用相同的模板前缀, 也可使用不同的模板前缀
>使用相同的模板前缀时, 一般参数个数不同, 否则为函数模板特化(specialized)
>

- 重载决议匹配原则
   - 匹配的更好, 少执行转换(允许的转换中)
   - 更特例, 如`const T&` 与 `T *`面对`string *`或`const string *`
      - 面对前者, `T *`是更好的选择, 不需要const转换即可
      - 面对后者, `T *`是更好的选择, 更特化, 与模板特化同义
   - 多个候选版本中, 优先非模板版本
   - 详见`C++ Primer::P619`

```CPP
// 二者为函数模板之间的重载, 都可以匹配所有类型
// 但是
// 左值, const左值, const右值
template <typename T> void f(const T&); 
// 右值
template <typename T> void f(T&&);
```

#### (函数)模板参数变长 <=> 可变参数函数模板
> 函数参数变长
> 使用的简单方法之一是: 初始化列表, 即`Aggragate类型`的花括号
> 实现为`initilizer_list<T>`模板的元素 + 继承
> 此外`<cstdarg>::var_arg` 参数变长必有一个非变长参数, 即`...`之前有还有参数
> 

- 模板使用递归, 递归的base case终止条件是一个模板参数不变长的, 其余使用参数包`sizeof...(Args)函数/模板参数包`作为递归阶段
- 模板前缀一般不同, 多个参数包啊, 参数一般不同, 多个参数包啊
- 本质是函数模板之间的重载, 终止条件 => 匹配最好的版本, 更特化的版本
- `注意: 递归终止条件一定声明在前`, 否则无限递归啊, 参数包为0 => GG

```CPP
template <typename T>
ostream& print(ostream& out, const T& t)
{
    out << t;
}

// 在前缀和函数形参中的 ... 前后允许空白符
template <typename T, typename... Args>
ostream& print(ostream& out, const T& t, const Args&... rest)
{
    out << t << ", ";
    return print(out, rest...);
}
```

- 语法`...`的语义为`包扩展`, 语法格式为`参数包模式 ...`
- 意义为: `Python/JS::map()`或`std::transform()`,  使用返回的线性表作为又一个参数包(即声明的rest又是一个参数包)
- 如`类型参数包 -> 名字对象参数包`
- 参数包模式可以很复杂
   - 模板参数包 => 对于第一次扩展 => 是类型上的声明 => 可以CV限定, 引用
   - 函数参数包 => 之后的扩展 => 是name/标识符/对象/变量 => 可以是任何表达式 
   - 对于模板参数包 => `typename ... Args`表示类型参数变长, `type ... Args`表示`类型type`的非类型参数变长 => 包括F/C模板
   - 参数包可以为空包(empty)
- 转发参数包
   - 不过是参数包模式比较特殊

#### 模板特化(specilized)
> 永远等价于接管编译器的操作
> 

- 函数模板
   - 特例化, 当然也够不成函数模板与不同函数之间的重载 => 等价于构成一个具体实例, 可以直接参与全局符号连接, 可以友元 => `inline使用`
   > 不会影响重载匹配决议(更好, 更特化具体, 非模板)
   > 特化声明要与原始F/C模板声明在同一命名空间, 一般使用同一文件
   > 特化声明要完整, 要有实际实现代码, 否则找不到实例
   > 禅 法门 公理 => 不过是接管编译器而已啊
   > 

   - 语法, 模板前缀为`template<>`告诉编译器我要特化, 可以有无花括号声明, 模板参数`定死一个实例`, 要符合原始函数模板的`模板参数T`声明, 而后在函数形参中体现
   - 偏特化不允许

```CPP
template <typename T>
int compare(const T& left, const T& right)
{
   return left - right;
}

template <>
int compare(const char* const& left, const char* const& right);

// for string literal
// const char[size] ITC const char *
template <>
int compare(const char* const& left, const char* const& right)
{
   cout << "specilized version" << endl;
   return strcmp(left, right);
}

int compare(const char (&left)[4], const char (&right)[4])
{
	cout << "non-template version" << endl;
	return strcmp(left, right);
}

int main()
{
   cout << (compare("abc", "abb") > 0) << endl; // 1
   return 0;
}
```

- 类模板
   - 特例化 => 等价于构成一个具体实例, 可以直接参与全局符号连接, 可以友元 => 有时`inline`使用
      - 语法, 模板前缀`template<>`, 可以花括号内外声明分开, 模板参数`定死一个实例`, 要符合原始类模板的`模板参数T`声明(模板之后, 花括号之前)
      - `template <> struct<int, double> { };`
   - 偏特化(部分参数定死实例, 或定死参数部分特性&, &&, 只能**类型参数**构成特化)
      - 语法, 模板前缀不变, 在模板名字后指定特性`template <typename T> struct remove_reference<T&> { };`
      - 语法, 模板前缀定死那部分不写, 模板名字后指定哪些实例`tempalte <typename T> struct psgt<T, int> { };`
      - 语法, 当然也可以将模板前缀变得更长(如使用模板进行其中一个参数的特化`std::tuple`), 但模板之后, 花括号之前同样要符合一一对应原始模板的模板参数声明
      - 特例化语法是极端情况 => ***本质是模板前缀不需要定死实例啊***
      - 最终效果是, 模板前缀变短/不变短, 模板名字后`或增加特性或定死部分实例`(所有参数与原始模板的模板参数按位置对应)
   - 仅仅成员特例化, 属于花括号外部声明
      - 语法, 模板前缀`template<>`, 模板名字后定死实例`template <> struct<int, double>::psf(){ }`
   - 一种奇异的做法是, 通用版本只做模板声明, 通通使用特化版本, 充分利用编译期???
   - 类模板的多个偏特化可能构成匹配时的**更好的匹配原则**

- 更恰当理解, 无论特例化还是偏特化, 对模板前缀本质是没有限制的
   - 如果使用的是非模板进行特化, 前缀的变化符合上述规律
   - 如果再次使用模板进行其中一个参数的特化, 显然可以出现任何种类的模板前缀
   - 最正确的理解是**不应该看模板前缀的变化, 而是要看模板名之后, 花括号之前的`<>`内的情况**(顺序对应, 声明符合原模板要求, 定死实例, 增加特性)

```C++
// std::tuple_element<size_t, std::tuple<>>::type 类模板的可能声明实现
template< std::size_t I, class T >
struct tuple_element;
 
// recursive case
template< std::size_t I, class Head, class... Tail >
struct tuple_element<I, std::tuple<Head, Tail...>>
    : std::tuple_element<I-1, std::tuple<Tail...>> { 

};
 
// base case
template< class Head, class... Tail >
struct tuple_element<0, std::tuple<Head, Tail...>> {
   typedef Head type;
};
```


#### 成员对象/变量指针, 成员函数指针简述
>成员是一种类型, 成员指针不同于普通指针
>成员类型, 成员
>

- 成员对象指针
   - 类型声明`const 成员类型(底层) className::*name/标识符;`
   - 初始化或指向成员`&className::成员对象`, 一定非静态, `&`不可省略, 要求`public`, 否则使用如下
   - 或者使用`静态成员函数`返回成员指针 => 进行初始化或指向成员 => 正确的数据封装姿势
   - 使用 => 成员对象指针, 只有`成员`和`指针`属性, 要确定实例即`object.*mp`或`pToObject->*mp`
   - auto/decltype() 类型推导是常用姿势
   - 静态成员对象本质是非成员啊, 注意

- 成员函数指针
   - 类型声明`返回类型 (className::*name/标识符)(className::类型成员, ...)const;`
   - 显然类型成员是`public`的
   - 初始化或指向成员`&className::成员对象`, 一定非静态, `&`不可省略, 要求`public`, 否则`非public`使用如下
   - 或者使用`静态成员函数`返回成员指针(此时要求不能直接`&name`,也要`&className::成员对象`, 因为`成员`) => 进行初始化或指向成员
   - 如果此处使用`非静态成员函数`返回指针(`&className::成员对象`) => 也行
   > 此时很奇怪的就是成员指针本质是实例无关的, 强行成员相关一下
   > 使用2个实例如以下代码中的f, f2
   > 无意义啊
   > 

   - 使用`(object.*mp)(...)`或`(pToObject->*mp)(...)` => 注意优先级
   - 静态成员函数本质是非成员啊, 注意

```CPP
class Foo {
public:
	const long long ago = -1;
	long long Foo::*getpagop()
	{
        return &Foo::pago;
	}
	Foo(int i):
		pago(i)
	{

	}
private:
	long long pago;
};

int main()
{
	// 以下成员对象指针使用方法等价
	auto mp0 = &Foo::ago;
	const long long Foo::*mp1 = &Foo::ago;
	typedef const long long Foo::*MP1;
	using MP2 = const long long Foo::*;
	MP1 mp2 = &Foo::ago;
	MP2 mp3 = &Foo::ago;

	Foo f(0), *pf = &f;
	cout << (f.*mp0 == pf->*mp1) << endl; // 1
	cout << (f.*mp2 == pf->*mp3) << endl; // 1

	auto mp4 = &Foo::getpagop;
	long long Foo::*(Foo::*mp5)();
	mp5 = &Foo::getpagop;
	typedef long long Foo::*(Foo::*MP3)();
	using MP4 = long long Foo::*(Foo::*)();
	MP3 mp6 = &Foo::getpagop;
	MP4 mp7 = &Foo::getpagop;

	Foo f2(1);

	cout << f2.*(f.*mp4)()  << endl;  // 1

	return 0;
}
```

> 一点理解
> 成员, 成员, 成员, 本质是实例无关的, 唯一的唯一的表达方式是className::name/成员对象className::name/成员函数, 可以在静态/非静态成员函数中使用, 仍然要满足PPP原则
> 返回的本质是类的原型部分的内容, 属于实例无关 :: 完备性
> 

- 