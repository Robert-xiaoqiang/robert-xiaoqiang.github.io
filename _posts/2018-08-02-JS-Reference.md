---
title: JS-Reference
key: 201808020
tags: JavaScript Reference
---

# JS-Reference/Binding

- 函数声明
   - 有名 => 可以不绑定(符号提升), 也可以绑定(语法上整体用圆括号括起来更直观)
   - 无名 => 必须用变量绑定, 否则无意义语句!

<!--more-->

- 函数调用
   - f();
   - 声明后整体括起来(); 此时圆括号为必须;

```JavaScript
   (function f()
   {
       
   })();
   或
   (function()
   {
       
   })();   
```

   -  绑定后直接调用, 此时圆括号***不是必须***


```JavaScript
   var fo = function()
   {
       
   }();
   或
   var fo = function f()
   {
       
   }();   
```

   - 上述语法的本质是**JS中Binding/Reference后返回的是left operand**
   - **类似C/CPP, 不同于Python**

