---
title: 前端模块加载浅析
date: 2018-11-26 21:03:52
tags: [module, js]
categories:
  - 前端
  - 模块化
---

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-15.jpg)

提到前端中的模块加载，大概脑海中能想到的就是：AMD、CMD、UMD、ES6 Module这样的字眼。

1. 那么，它们之间又有什么区别和关系呢？

2. Node.js中的模块又是如何加载的呢？

3. Webpack打包后的模块又是如何加载的呢？

本文将针对这几个问题做一些简单的梳理和介绍。

<!-- more -->

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-15.jpg)

## AMD ( asynchronous module definition )

从字面的意思来理解，AMD的意思是异步模块加载规范，它是为浏览器设计的一套规范。

通过require.js来了解AMD规范是一个比较不错的方式。当然，这里你并不需要深入理解require.js，我们只是借用了一些它的API。

让我们来写一个简单的 Demo —— 在页面中调用一个add函数和一个isUrl函数。

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

[Github代码点这里](https://github.com/Ash-sc/js-module-definition/tree/master/amd-demo)。

上面例子中使用到的写法就是符合AMD规范的写法

  使用define定义模块；参数依次为模块名称(可省略)、依赖的模块与回调；被依赖的模块提供return

在浏览器中打开index.html，控制台中会有如下输出：

![amd-module-console](http://web-site-files.ashshen.cc/blog/module-in-js/amd-demo-output.png)

可以看到，使用AMD规范的模块加载顺序为：被依赖的模块先加载，主模块最后加载。


## CMD ( common module definition )

说到CMD的话，就不得不说一下seajs这个库了。

还是刚刚那个add和isUrl的例子。

我们先来看一下它在seajs下是怎么实现的：

index.html：

``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>CMD DEMO</title>
  </head>
  <body>
    <script src="./sea.js"></script>
    <script>
      seajs.use("./main")
    </script>
  </body>
</html>
```

main.js：
``` js
define(function(require, exports, module) {
  console.log('main.js loaded !')

  const calculate = require('./calculate')
  console.log('add result :', calculate.add(1, 2, 3, 4, 5))

  const isUrl = require('./is-url')
  console.log('is url \'https://google.com.ph\' :', isUrl.isUrl('https://google.com.ph'))
})
```

calculate.js：
``` js
define(function(require, exports, module) {
  console.log('calculate.js loaded ! (required by main.js)')

  const add = require('./add')

  module.exports = {
    add: add.add
  }
})
```

add.js：
``` js
define(function(require, exports, module) {
  console.log('add.js loaded ! (required by calculate.js)')

  const add = function(a = 0, ...args) {
    let result = a
    if (args.length) result += add(...args)
    return result
  }

  module.exports = {
    add
  }
})
```

isUrl.js与add.js一致，这里就不贴出来了。

我们可以发现：seajs与require.js的代码区别在于：

1. seajs的依赖是通过require()这一API来引入的。

2. seajs依赖的输出是通过module.exports这个API，而require.js是通过return。

然后我们再来看一下seajs在控制台的输出情况：

![cmd-demo-console](http://web-site-files.ashshen.cc/blog/module-in-js/cmd-console.png)

**可以看到，输出与之前require.js的输出顺序是不同的。 require.js遵循AMD规范，所以是被依赖的模块先执行，而seajs是遵循的CMD规范，模块的执行顺序按照代码中require()的顺序。**

## CommonJS规范

AMD规范和CMD规范是基于浏览器的模块规范，而CommonJS则是基于服务端（Node）的规范。

还是刚刚那个add和isUrl的例子，如果我们要在Node中实现：

main.js
``` js
console.log('main.js loaded !')

const calculate = require('./calculate')
console.log('add result :', calculate.add(1, 2, 3, 4, 5))

const isUrl = require('./is-url')
console.log('is url \'https://google.com.ph\' :', isUrl.isUrl('https://google.com.ph'))

```

calculate.js
``` js
console.log('calculate.js loaded ! (required by main.js)')

const add = require('./add')

module.exports.add = add.add
```

add.js
``` js
console.log('add.js loaded ! (required by calculate.js)')

const add = function(a = 0, ...args) {
  let result = a
  if (args.length) result += add(...args)
  return result
}

module.exports.add = add
```

jsUrl.js
``` js
console.log('isUrl.js loaded ! (required by main.js)')

module.exports.isUrl = function(url = '') {
  return url ? /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)$/.test(url) : ''
}
```

执行`node ./main.js`后，控制台中的输出：

![node-output](http://web-site-files.ashshen.cc/blog/module-in-js/node-output.png)

可以看到，不仅从写法上，CMD与CommonJS很像，模块的加载顺序也是一致的。

## ES6 module

ES6中，也规定了新的模块加载方案，与前面AMD/CMD模块规范不同的是：ES6中采用`import`和`export`来引入导出模块。

那么，之前的示例使用ES6 module又该怎么实现呢？代码如下：

index.html
``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>ES6 DEMO</title>
  </head>
  <body>
    <script src="./main.js" type="module"></script>
  </body>
</html>
```
index.html与之前的示例稍有区别：在script标签中添加了`type="module"`，告知浏览器需要按照ES6模块来解析。

main.js
``` js
console.log('main.js loaded !')

import { add } from './calculate.js'
console.log('add result :', add(1, 2, 3, 4, 5))

import { isUrl } from './is-url.js'
console.log('is url \'https://google.com.ph\' :', isUrl('https://google.com.ph'))
```

calculate,js
``` js
console.log('calculate.js loaded ! (imported by main.js)')

import { add } from './add.js'
export {
  add
}
```

add.js
``` js
console.log('add.js loaded ! (imported by main.js)')

const add = function(a = 0, ...args) {
  let result = a
  if (args.length) result += add(...args)
  return result
}

export {
  add
}
```

isUrl.js
``` js
console.log('isUrl.js loaded ! (imported by main.js)')

const isUrl = function(url = '') {
  return url ? /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)$/.test(url) : ''
}

export {
  isUrl
}
```

js文件使用import和export这两个API，一一对应，比较好理解。对于这两个API这里就不具体多说了。

此外，与AMD/CMD示例不同的是，AMD/CMD示例你可以使用file协议直接在浏览器中打开index.html查看效果。

但是ES6 module的示例需要起一个静态服务才能查看。这里推荐一个比较好用的静态服务npm包：[http-server](https://www.npmjs.com/package/http-server)。

控制台输出如下：

![es6-module-console](http://web-site-files.ashshen.cc/blog/module-in-js/es6-module.png)

可以看到：与require.js一致，也是先加载依赖，后执行代码。

## UMD ( universal module definition )

UMD可以理解为是一个兼容模式的模块规范，它既支持AMD规范，也支持CommonJS规范。

而它的实现方式也很简单：

1. 先判断是否支持Node.js模块格式（exports是否存在），存在则使用Node.js模块格式。

2. 再判断是否支持AMD（define是否存在），存在则使用AMD方式加载模块。

3. 前两个都不存在，则将模块公开到全局。

## Babel

虽然ES6规范甚至ES789都已经发布了，但是这并不意味着我们可以在任何浏览器中直接使用它们。

通常情况下，我们需要使用babel之类的代码转换工具来将代码转换成兼容性比较好的ES5标准代码。

那么，babel都对代码进行了怎么样的转换呢？

我们把calculate.js的代码在[这里](https://www.babeljs.cn/repl/)进行转换，得到的结果是这样的：
``` js
'use strict';

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.add = undefined;

var _add = require('./add.js');

console.log('calculate.js loaded ! (imported by main.js)');

exports.add = _add.add;
```

然后，我们发现：转换后的代码，似乎有点像上面示例中Node中运行的代码，也就是说CommonJS规范的代码。

当然，上面的代码是无法直接在浏览器中运行的，因为浏览器环境下中并没有require和export。

所以我们还需要一个打包工具来把这些代码进行打包，才能运行。

## Webpack 模块化构建

webpack是目前比较主流的一个模块化打包工具。

上面我们提到，babel将ES6规范的代码转换成了CommonJS规范的ES5代码，而CommonJS规范的代码由于浏览器环境缺乏require和export这样的API而无法在浏览器中直接运行。

所以，webpack打包所做的工作实际上仅仅是模拟了这些require和export等API。

OK，那么我们来看一下webpack是怎么模拟这些API的呢？

我们使用webpack打包了上面ES6 module的例子，得到的main.js代码如下：

``` js
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
/******/ 		}
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 		if(mode & 1) value = __webpack_require__(value);
/******/ 		if(mode & 8) return value;
/******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
/******/ 		var ns = Object.create(null);
/******/ 		__webpack_require__.r(ns);
/******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
/******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
/******/ 		return ns;
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = "./src/main.js");
/******/ })
/************************************************************************/
/******/ ({

/***/ "./src/add.js":
/*!********************!*\
  !*** ./src/add.js ***!
  \********************/
/*! exports provided: add */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"add\", function() { return add; });\nconsole.log('add.js loaded ! (imported by main.js)');\n$('order-ul').innerHTML += '<li>add.js loaded ! (imported by main.js)</li>';\n\nvar add = function add() {\n  var a = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : 0;\n\n  var result = a;\n\n  for (var _len = arguments.length, args = Array(_len > 1 ? _len - 1 : 0), _key = 1; _key < _len; _key++) {\n    args[_key - 1] = arguments[_key];\n  }\n\n  if (args.length) result += add.apply(undefined, args);\n  return result;\n};\n\n\n\n//# sourceURL=webpack:///./src/add.js?");

/***/ }),

/***/ "./src/calculate.js":
/*!**************************!*\
  !*** ./src/calculate.js ***!
  \**************************/
/*! exports provided: add */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _add_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./add.js */ \"./src/add.js\");\n/* harmony reexport (safe) */ __webpack_require__.d(__webpack_exports__, \"add\", function() { return _add_js__WEBPACK_IMPORTED_MODULE_0__[\"add\"]; });\n\nconsole.log('calculate.js loaded ! (imported by main.js)');\n$('order-ul').innerHTML += '<li>calculate.js loaded ! (imported by main.js)</li>';\n\n\n\n\n//# sourceURL=webpack:///./src/calculate.js?");

/***/ }),

/***/ "./src/is-url.js":
/*!***********************!*\
  !*** ./src/is-url.js ***!
  \***********************/
/*! exports provided: isUrl */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
__webpack_require__.d(__webpack_exports__,"isUrl", function() { return isUrl; });
console.log('isUrl.js loaded ! (imported by main.js)');
$('order-ul').innerHTML += '<li>isUrl.js loaded ! (imported by main.js)</li>';
var isUrl = function isUrl() {
  var url = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : '';
  return url ? /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)$/.test(url) : '';
};
//# sourceURL=webpack:///./src/is-url.js?

/***/ }),

/***/ "./src/main.js":
/*!*********************!*\
  !*** ./src/main.js ***!
  \*********************/
/*! no exports provided */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _calculate_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./calculate.js */ \"./src/calculate.js\");\n/* harmony import */ var _is_url_js__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(/*! ./is-url.js */ \"./src/is-url.js\");\nconsole.log('main.js loaded !');\n$('order-ul').innerHTML += '<li>main.js loaded !</li>';\n\n\n\n\nconsole.log('add result :', Object(_calculate_js__WEBPACK_IMPORTED_MODULE_0__[\"add\"])(1, 2, 3, 4, 5));\nconsole.log('is url \\'https://google.com.ph\\' :', Object(_is_url_js__WEBPACK_IMPORTED_MODULE_1__[\"isUrl\"])('https://google.com.ph'));\n\n//# sourceURL=webpack:///./src/main.js?");

/***/ })

/******/ });
```

代码看上去比较长，实际上就是一个自执行函数，我们把打包后的main.js拆开来看：

**其实就是一个IIFE**。

第一部分：从第一行到第85行。是函数的定义，定义了`module`、`exports`、`require`（这里为`__webpack_require__`）方法，并且直接执行了`__webpack_require__('./src/main.js')`

PS：这里webpack也做了一部分优化，比如加载过的模块不再重新加载了，之后的调用直接exports。

第二部分：第86行到最后，执行了前面85行所定义的函数，并且传入了参数。

我们可以看到，传入的参数是一个对象，对象的key是像`./src/main.js`这样以模块路径作为模块名，而value则是一个函数，同时也把module（所有模块对象）、exports（webpack模拟的）、require（webpack模拟的）传入了这个函数中。函数内部使用eval执行了模块自身的逻辑。我这里把`./src/is-url.js`模块的代码从eval转换成了可直接执行的代码。

webpack打包之后的代码可读性不强，可以看我手写的简易版的：
``` js
(function(modules) {
  // const installedModules = {}
  const require = function(moduleName) { // 定义require
    // if (installedModules[moduleName]) return installedModules[moduleName].exports
    // const module = installedModules[moduleName] = {
    const module = {
      exports: {}
    }
    modules[moduleName](module, module.exports, require)

    return module.exports // 返回被调用模块的module.exports
  }
  return require('./main.js') // 执行入口文件
})({ // 实参是一个对象
  './main.js': (function(module, exports, require) {
    console.log('main.js loaded !')
    const calculate = require('./calculate.js')
    console.log('add result :', calculate.add(1, 2, 3, 4, 5))
    const isUrl = require('./is-url.js')
    console.log('is https://www,google.com.hk url :', isUrl.isUrl('https://www,google.com.hk'))
  }),
  './calculate.js': (function(module, exports, require) {
    console.log('calculate.js loaded !')
    const add = require('./add.js')
    module.exports = { add: add.add }
  }),
  './add.js': (function(module, exports, require) {
    console.log('add.js loaded !')
    const add = function(a = 0, ...args) {
      let result = a
      if (args.length) result += add(...args)
      return result
    }
    module.exports = {
      add: add
    }
  }),
  './is-url.js': (function(module, exports, require) {
    console.log('is-url.js loaded !')
    // const add = require('./add.js')
    // console.log('use add in is-url.js :', add.add(1, 3, 5, 7))
    module.exports = {
      isUrl: function(url = '') {
        return url ? /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)$/.test(url) : ''
      }
    }
  })
})
```
直接在浏览器中运行就可以看到效果。

至于之前说到webpack针对模块已加载优化，可以取消上面代码中的注释进行查看。


## 对于模块按需加载

对于使用React/Vue/Angular等前端框架构建的项目，我们通常会采用一系列的首屏优化手段，比如路由按需加载等。那么，webpack打包的代码又是如何实现按需加载的呢？

我截取了一个Vue项目webpack打包之后的index.js文件部分代码：
``` js
// index.js
/******/ 	// This file contains only the entry chunk.
/******/ 	// The chunk loading function for additional chunks
/******/ 	__webpack_require__.e = function requireEnsure(chunkId) {
/******/ 		var promises = [];
/******/
/******/
/******/ 		// JSONP chunk loading for javascript
/******/
/******/ 		var installedChunkData = installedChunks[chunkId];
/******/ 		if(installedChunkData !== 0) { // 0 means "already installed".
/******/
/******/ 			// a Promise means "currently loading".
/******/ 			if(installedChunkData) {
/******/ 				promises.push(installedChunkData[2]);
/******/ 			} else {
/******/ 				// setup Promise in chunk cache
/******/ 				var promise = new Promise(function(resolve, reject) {
/******/ 					installedChunkData = installedChunks[chunkId] = [resolve, reject];
/******/ 				});
/******/ 				promises.push(installedChunkData[2] = promise);
/******/
/******/ 				// start chunk loading
/******/ 				var head = document.getElementsByTagName('head')[0];
/******/ 				var script = document.createElement('script');
/******/ 				var onScriptComplete;
/******/
/******/ 				script.charset = 'utf-8';
/******/ 				script.timeout = 120;
/******/ 				if (__webpack_require__.nc) {
/******/ 					script.setAttribute("nonce", __webpack_require__.nc);
/******/ 				}
/******/ 				script.src = jsonpScriptSrc(chunkId);
/******/
/******/ 				onScriptComplete = function (event) {
/******/ 					// avoid mem leaks in IE.
/******/ 					script.onerror = script.onload = null;
/******/ 					clearTimeout(timeout);
/******/ 					var chunk = installedChunks[chunkId];
/******/ 					if(chunk !== 0) {
/******/ 						if(chunk) {
/******/ 							var errorType = event && (event.type === 'load' ? 'missing' : event.type);
/******/ 							var realSrc = event && event.target && event.target.src;
/******/ 							var error = new Error('Loading chunk ' + chunkId + ' failed.\n(' + errorType + ': ' + realSrc + ')');
/******/ 							error.type = errorType;
/******/ 							error.request = realSrc;
/******/ 							chunk[1](error);
/******/ 						}
/******/ 						installedChunks[chunkId] = undefined;
/******/ 					}
/******/ 				};
/******/ 				var timeout = setTimeout(function(){
/******/ 					onScriptComplete({ type: 'timeout', target: script });
/******/ 				}, 120000);
/******/ 				script.onerror = script.onload = onScriptComplete;
/******/ 				head.appendChild(script);
/******/ 			}
/******/ 		}
/******/ 		return Promise.all(promises);
/******/ 	}
...

...
/* harmony default export */ var pages_account = ({
  login: account_login,
  modifyPwd: function modifyPwd() {
    return __webpack_require__.e(/* import() */ "chunk-2d0da513").then(__webpack_require__.bind(null, "6aa3"));
  }
})
...

...
// chunk-2d0da513.js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([["chunk-2d0da513"],{
/***/ "6aa3": (function() { ... })
}])
```

从上面的代码我们可以看出：webpack打包后的代码中，在`__webpack_require__`对象上定义了一个新的方法`__webpack_require__.e`，然后通过传入chunkId来获取对应的js文件（通过往header元素内添加script标签）。

在js文件下载完成后触发`.then(__webpack_require__.bind(null, "6aa3"))`，这里的`6aa3`就是“chunk-2d0da513”这个js文件中定义的模块名称。

所以，从login跳转到modifyPwd页面，按需加载的步骤大致如下：

1. vue-router代码监控到url发生变化

2. 执行modifyPwd定义的函数，使用webpack定义的`__webpack_require__.e`获取chunkjs文件。

3. chunkjs获取完成 (script标签) 后，自动执行，将代码push到`window["webpackJsonp"]`中，`__webpack_require__.e`返回promise。

4. then()触发，`__webpack_require__`将modifyPwd定义为已加载，并且执行chunkjs逻辑代码，然后逻辑代码的exports存入`installedModules`中。

5. 下次进入modifyPwd页面，直接返回`installedModules`中对应chunkId的exports。


**PS**： 有兴趣的同学可以在webpack打包的Vue或者React项目浏览器控制台输入`window["webpackJsonp"]`查看模块加载信息。比如：[vue-admin](https://panjiachen.github.io/vue-element-admin)
