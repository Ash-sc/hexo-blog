---
title: javaScript中call、apply与bind使用方法的区别
date: 2017-03-19 16:57:47
tags: [js, 前端]
categories: 前端
---

js中call、apply与bind都可以用来改变this的指向，本篇主要介绍的是三者在语法使用上的区别。

<!-- more -->

首先，我们让我们来看一段代码：

``` js
var me = {
  name: "ash",
  age: 22,
  introduce: function() {
    console.log("Name: " + this.name + ", Age: " + this.age);
  }
}
var shen = {
  name: "shen",
  age: 20
}
me.introduce();
```

在控制台执行这段代码，会有如下输出：

> Name: ash, Age: 22

那么，如果我要使用me的introduce方法来显示shen的数据呢？

call方法这么写: `me.introduce.call(shen)`

apply方法这么写: `me.introduce.apply(shen)`

bind方法这么写: `me.introduce.bind(shen)()`

如果直接写`me.introduce.bind(shen)`则在控制台中会输出：

> function() {
>     console.log(“Name: “ + this.name + “, Age: “ + this.age);
> }

**由此可见，bind与call和apply这两者不同之处在于，bind方法返回的是一个函数，而call和apply则是直接调用函数。**

那么，call与apply又有什么不同呢？我们把上面的例子中的introduce函数稍微改动一下，添加两个参数：

``` js
introduce: function(sex, weight) {
  console.log("Name: " + this.name + ", Age: " + this.age + ", Sex: " + sex + ", Weight: " + weight);
}
```

call方法传参时，需要这么写: `me.introduce.call(shen, ‘男’, ‘62kg’)`

而apply需要这么写: `me.introduce.apply(shen, [‘男’, ‘62kg’])`

所以，call与apply使用上的区别是：**apply传参使用的是一个数组**。

**PS:** 由于bind方法返回的是一个函数，所以bind方法传参可以这么写: `me.introduce.bind(shen)(‘男’, ‘62kg’)`

