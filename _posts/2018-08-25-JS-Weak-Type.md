---
title: JS-Weak-Type
key: 201808240
tags: JavaScript
---

# JS-Implicit-Type-Conversion

#### T -> Number/算数类型
> Number() 显式构造
- String
   - 空 => 0
   - 合法字面值 => parse正确
   - 不具有parseint, parsefloat的灵活性
- Boolean
   - t/f => 1/0
- null => 0
- undefined => NaN
- Function, Date, RegExp, Array, Customized Type(等等种种引用类型)
   - 先toString(), 再遵循第一条

<!--more-->

#### T -> String/字符串
> toString()方法
> String()显式构造
- Number
   - 格式化输出
- 'true' 'false' 'null' 'undefined' 'NaN'
- Function, Date, RegExp => 包围字面值
- Array  => ','.join(Array)
- Customized Type => '[object Object]'

`注意+的overload: 算数类型运算引发Number转换, 而+字符串拼接引发String转换`

#### T -> Boolean/布尔类型
> 没有其他中间过程的隐式转换
> 任何引用类型(Object)均为true
- 5兄弟为false, 其余为true
- String中空字符串
- Number中0, 0.0
- Number中NaN
- undefiend
- null

`注意: ! !! if for while 触发Boolean转换, 一般先进行Evaluate`

#### == 隐式转换规则
> 与===的唯一差别是不同type时是否进行ITC
> 同type时/后无差别
> 与Python不同, JS*值类型*比较*值*, *引用类型*比较*地址*

- 引用类型与引用类型
> 此时是===/==同义, 即比较地址
> 一般binding 同一个ooob才相等
> 引用类型比较地址

- 引用类型与算数类型(NaN)
   - 引用转换为算数类型(详见以上)
   - 而后值类型比较(值类型比较值)
- 引用类型与字符串
   - 引用类型转换为字符串(详见以上)
   - 而后值类型比较(值类型比较值)
- 引用类型与布尔类型
   - 引用类型装换为算数类型(详见以上)
   - 布尔类型转换为算数类型(详见以上)
   - 而后值类型比较(值类型比较值)
> 注意***不是***引用类型转布尔!
> 啊呀! 远远不及Python Intuitively!

- 字符串与算数类型
   - 字符串转换为算数类型(详见以上)
   - 而后值类型比较

- 字符串与布尔类型
   - 都转换为算数类型(详见以上)
   - 而后值类型比较

- 布尔类型与算数类型
   - 布尔类型转换为算数类型(详见以上)
   - 而后值类型比较

- 总结
   - `==`左右type不同, 转换为算数类型(***值***类型算数类型 `等价于Number()作用`)
   - `null` `undefined`是特例记住规则
   > 既是类型, 也是单例
      - 不会转换为算数类型, 而是与其他任何都不相等(!==/!=)
      > !==是显然的
      - 二者之间`null == undefiend` `null !== undefiend`
   - **引用类型与字符串凑合到一起时特例?**

```javascript
[] == []    // false
[] == 0     // true
({}) == NaN // Number('[object Object]') == NaN false
// NaN 牛逼
// 
[] == ''            // true
({}) == ''          // false
[12] == '12'        // true 
[12, 13] == '12,13' // true 
[12] == '12px'      // false

[] == false         // true
![] == false        // true 优先级!引发布尔转换
[1] == true         // true
[3] == true         // false
[1, 1] == true      // false

```

- Hold on / Hold Fast To
