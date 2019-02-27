---
title: Fold-Expression
key: 201902270
tags: CPP Template
---



# Fold-Expression

## 作用

- 避免模板参数变长的basic case, 单独做函数实现(函数模板重载, 更具体, 更特化)
- 适用于模板参数变长, 当且仅当只有一个变长参数
- **记住, 折叠展开后, 再求值, 展开过程不求值**

## 分类

- 一元右折叠`exp op ...`

- 一元左折叠`... op exp`
- 二元左折叠`exp op ... op init`
- 二元右折叠`init op ... op exp`
- 语法上高度的一致性
  - 一元折叠时, 参数包为空, `&& || ,`trival
  - 二元折叠时, 参数包为空, init为求值结果



```C++

#include <iostream>
using namespace std;

/**
 * 二元左折叠
 * 参数包为空, 如果没有其他更匹配的实现 => cout << endl;
 */
template<typename... Ts>
void printAll(Ts&&... mXs)
{
    (cout << ... << mXs) << endl;
}

/**
 * 一元右折叠, 基于逗号挺有趣
 * { a1, a2, a3, a4 }
 * (mFn(a1), (mFn(a2), (mFn(a3), (mFn(a4) ) ) ) );
 * 展开不求值, 展开后求值
 */
template<typename TF, typename... Ts>
void forArgs(TF&& mFn, Ts&&... mXs)
{
    (mFn(mXs), ...);
}
 
int main() 
{
    printAll(3, 4.0, "5"); // 345
    printAll(); // 空行
    forArgs([](auto a){cout << a;}, 3, 4.0, "5"); // 345
    forArgs([](auto a){cout << a;}); // 空操作
}

```

