---
title: Basic-Template
key: 201903200
tags: CPP Template
---

# Basic-Template

- 类模板的{友元类模板, 友元函数模板}
  - 模板参数不共享, 全部实例
  - 共享, 用已有参数进行1`totally实例化`, 友元参数多, `部分序列实例`友元, 友元参数少, `不限制其他参数各种实例`也将是友元
- 模板嵌套定义
  - 共享参数, 是无参数的模板
  - 非共享, 两套模板前缀
- 函数与函数模板, trival, 同理



<!--more-->



```C++
#include <iostream>
#include <typeinfo>
#include <string>
#include <algorithm>
#include <functional>
#include <string>
#include <regex>
#include <vector>
#include <iterator>
#include <fstream>
#include <iomanip>
#include <sstream>
#include <map>
#include <ctime>
#include <cmath>
#include <memory>
#include <type_traits>
#include <tuple>
#include <initializer_list>
#include <thread>
#include <cstdint>
#include <limits>
#include <stdexcept>
#include <future>
#include <chrono>
#include <mutex>
#include <random>
#include <condition_variable>
using namespace std;

template <typename T, typename S>
void gft1(T&& t, S&& s);

template <typename T, typename S, typename K>
void gft2(T&& t, S&& s, K&& k);

template <typename T>
void gft3(T&& t);


template <typename T, typename S>
class CT1 {
public:
	void f();
};

template <typename T, typename S, typename K>
class CT2 {
public:
	void f();
};

template <typename T>
class CT3 {
public:
	void f();
};

template <typename T, typename S>
struct Foo {
private:
	void f();
public:
	struct Too {
		void f();
	};
	template <typename IT, typename IS>
	struct Soo {
		void f();
	};
	template <typename FT, typename FS>
	void ft(FT&& ft, FS&& fs);

	// friene funtion template with different template parameter
	template <typename FRT, typename FRS>
	friend void gft1(FRT&& frt, FRS&& frs);
	// frient funtion template with same/more/less template parameter
	friend void gft2<>(T&&, S&&, T&&);
	friend void gft3<>(T&&);

	// friend class template with different template parameter
	template <typename CRT, typename CRS>
	friend class CT1;

	// friend class template with same/more/less template parameter
	friend class CT2<T, S, T>;
	friend class CT3<T>;
};

template <typename T, typename S>
void Foo<T, S>::f()
{

}

template <typename T, typename S>
template <typename FT, typename FS>
void Foo<T, S>::ft(FT&& ft, FS&& fs)
{

}

template <typename T, typename S>
void Foo<T, S>::Too::f()
{

}

template <typename T, typename S>
template <typename IT, typename IS>
void Foo<T, S>::Soo<IT, IS>::f()
{

}


//---------------------------------------

template <typename T, typename S>
void gft1(T&& t, S&& s)
{
	Foo<long, long long> f;
	f.f();
}

template <typename T, typename S, typename K>
void gft2(T&& t, S&& s, K&& k)
{
	Foo<T, S> f;
	f.f();
}

template <typename T>
void gft3(T&& t)
{
	Foo<T, int> f;
	f.f();
	Foo<T, bool> f2;
	f2.f();
	Foo<T, char> f3;
	f3.f();
}

//--------------------------------------------
template <typename T, typename S>
void CT1<T, S>::f()
{
	Foo<long, long long> f;
	f.f();
}

template <typename T, typename S, typename K>
void CT2<T, S, K>::f()
{
	Foo<T, S> f;
	f.f();
}

template <typename T>
void CT3<T>::f()
{
	Foo<T, bool> f;
	f.f();
	Foo<T, char> f2;
	f2.f();
}
//-------------------------
int main()
{
	Foo<int, short> f;
	f.ft(-1.0f, -1.0);
	Foo<int, short>::Too t;
	t.f();
	Foo<int, short>::Soo<double, float> s;
	s.f();

	gft1(-1.0f, -1.0);
	gft2(-1.0f, -1.0, -1.0f); // must be T, S, T; Tm S K is not friend
	gft3(-1.0f);

	CT1<int, short> c1;
	c1.f();
	CT2<int, short, int> c2; // must be T, S, T; T, S, K is not friend
	c2.f();
	CT3<int> c3;
	c3.f();
	return 0;
}

```

