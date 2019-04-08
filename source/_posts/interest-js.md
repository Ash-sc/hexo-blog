---
title: JS里的那些骚操作
date: 2018-12-10 17:24:17
tags: [js]
categories: 前端
---

JS中总会有一些奇奇怪怪却又非常有趣的东西，比如`[] + [] === ''`、`![] + ![] === 0`等等，时常让我们咬牙切齿、哭笑不得却又欲罢不能。

当然，本文中出现的不仅仅是js搞怪玩法，更多的是一些“高逼格”但是写在业务中可能会被打死的玩法。

<!-- more -->

素材来源于平时写代码时遇到的或者论坛。

OK，话不多说，上码 ！~

### 一、IIFE

IIFE自执行函数大家应该都不陌生，`(function() {})()`或者`(() => {})()`这样的形式嘛。概念的话，在这里就不介绍了；那么，IIFE为什么可以执行呢？又为什么需要两个`()`呢？

首先，我们来看一下：如果我们这样写`function(a) {}()`，执行时会报这样的错：

```
Uncaught SyntaxError: Unexpected token ( // 这里的'('指的是'function'后面的'('
```

其实，(function() {})外层的括号是为了告诉js解析引擎：括号内的是一个函数表达式，而不是函数声明。如果不加括号js解析引擎则认为是声明了一个函数，自然也就解析出错了。

至于第二个括号，是函数的调用。

OK，那么我们只需要告诉js引擎第一部分是一个函数表达式，IIFE也就能正常执行了。比如，下面这样：

``` js
// 普通IIFE
(function (status) {
  console.log('status :', status)
})('success') // status : success

// 使用!
!function (status) {
  console.log('status :', status)
}('success') // status : success

// 使用+
+function (status) {
  console.log('status :', status)
}('success') // status : success

// 使用-
-function (status) {
  console.log('status :', status)
}('success') // status : success

// 使用~
~function (status) {
  console.log('status :', status)
}('success') // status : success
```

你可以使用任何一元运算符，包括 `void`。

当然，这些的前提是你对IIFE的返回值，如果你需要IIFE的返回值，那么还是推荐使用()()。

### 二、构造函数

第二个其实不算什么骚操作，应该算是一种比较简便的写法。

在我们使用`new`关键字配合构造函数来实例化一个对象时，有时候会选在不携带参数，比如：`const today = new Date()`。

这时候，我们可以省略掉`Date`后面的`()`，变成这样：`const today = new Date`。

当然，如果构造函数需要接收参数时，还是需要带上`()`的，如：`const arr = new Array(10)`。


--- 2019-01-22 ---

### 三、逗号操作符

在js中，逗号操作符比较少见，它一般用于在同一条语句中进行多个表达式的书写，并且返回最后一个表达式的值。

什么意思呢？打个比方说：有a、b、c三个变量，他们的值都是1，我们希望a和c加1，b减1，新赋值d等于c。

我们可能这么写：

``` js
let a = b = c = 1
a++
b--
const d = ++c
```

使用逗号运算符我们可以这么写：

``` js
let a = b = c = 1
const d = (a++, b--, ++c)
```

除此之外，逗号运算符还有一个用法。

我们都知道IIFE（上面介绍过）。那么，当我们有如下场景时：

``` js
var obj = {
  func: function () {
    console.log(this === window, this === obj)
  }
}
(obj.func)() // false true
```

由于`this`的指向原因，输出`false true`。那么，想要输出`true false`应该怎么写呢？

可以使用`bind`。

``` js
var obj = {
  func: function () {
    console.log(this === window, this === obj)
  }
}
(obj.func.bind(window))() // true false
```

其实，我们还可以使用逗号运算符。刚刚我们说到：逗号运算符会返回最后一个表达式的值。那么实际上，使用逗号运算符相当于做了一次赋值操作`let a = obj.func`。

``` js
var obj = {
  func: function () {
    console.log(this === window, this === obj)
  }
}
(0, obj.func)() // true false
```

所以，在一些编译工具或者库中，经常使用`(0, xx.func)(...args)`来保证`func`执行时，`this`始终指向`window`。

持续施工中......

















