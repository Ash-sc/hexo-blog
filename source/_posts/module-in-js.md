---
title: 前端模块加载浅析
date: 2018-11-26 21:03:52
tags: [module]
---

提到前端中的模块加载，大概脑海中能想到的就是：AMD、CMD、UMD、ES6 Module这样的字眼。

1. 那么，它们之间又有什么区别和关系呢？

2. Node.js中的模块又是如何加载的呢？

3. Webpack打包后的模块又是如何加载的呢？

本文将针对这几个问题做一些简单的梳理和介绍。

## AMD ( asynchronous module definition )

从字面的意思来理解，AMD的意思是异步模块加载规范，它是为浏览器设计的一套规范。

通过require.js来了解AMD规范是一个比较不错的方式。当然，这里你并不需要深入理解require.js，我们只是借用了一些它的API。

让我们来写一个简单的 Demo—— 在页面中调用一个add函数和一个isUrl函数。

目录结构如下：

``` bash
-- root folder
|-- index.html (调用 main.js，并执行 add 以及 isUrl 两个方法)
|-- main.js (依赖 calculate.js 以及 is-url.js)
|-- calculate.js (依赖 add.js )
|-- add.js
|-- is-url.js
```
index.html中主要代码如下：
``` html
<!DOCTYPE html>
<html>
 <head>
 <meta charset="utf-8">
 <title>AMD DEMO</title>
 </head>
 <body>
 <script data-main="main" src="./require.js"></script>
 <!-- <script src="./main.js"></script> -->
 </body>
</html>
```
data-main是require.js的写法，主要表示require.js的主模块为main.js文件，当然你也可以另写一个script标签来引用它。

然后是main.js：
``` js
require(['./is-url', './calculate'], function(isUrlModule, calculateModule) {
 console.log('main.js loaded !')
 console.log('add result :', calculateModule.add(1, 2, 3, 4, 5))
 console.log('is url \'https://google.com.ph\' :', isUrlModule.isUrl('https://google.com.ph'))
})
```
比较容易理解，require是require.js的API，第一个参数为一个数组，表示它依赖的模块；第二个参数为一个callback函数，函数的参数为依赖的模块的输出。

calculate.js代码如下：
``` js
define(['./add'], function(addModule) {
console.log('calculate.js loaded ! (required by main.js)')
 return {
  add: addModule.add
 }
})
```
我们可以发现，calculate.js与main.js只有一点细微的差别：require变成了define、多了一个return。

add.js代码：
``` js
define(function() {
 console.log('add.js loaded ! (required by calculate.js)')
 const add = function(a = 0, ...args) {
 let result = a
 if (args.length) result += add(...args)
  return result
 }
 return {
  add
 }
})
```
由于add.js没有依赖其他js文件了，所以它与calculate.js的差别在于它的第一个参数是空的了，在这里空的参数可以省略掉。

is-url.js：
``` js
define(function() {
 console.log('isUrl.js loaded ! (required by main.js)')
 return {
  isUrl: function(url = '') {
   return url ? /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)$/.test(url) : ''
  }
 }
})
```
is-url.js与add.js一致。

[github代码点这里](https://github.com/Ash-sc/js-module-definition/tree/master/amd-demo)。

上面例子中使用到的写法就是符合AMD规范的写法

（ 使用define定义模块；参数依次为(模块名称、)依赖的模块与回调；被依赖的模块提供return ）

在浏览器中打开index.html，控制台中会有如下输出：

![amd-module-console](//web-site-files.ashshen.cc/blog/module-in-js/amd-demo-console.png)

可以看到，使用AMD规范的模块加载顺序为：被依赖的模块先加载，主模块最后加载。

而main.js中实际上是依赖了两个模块—— is-url.js与calculate.js，加载的顺序与他们的层数有关（is-url.js只有一层而calculate.js还依赖了add.js），层数相同的模块依赖，顺序靠前的先加载。









