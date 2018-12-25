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


持续施工中......


















