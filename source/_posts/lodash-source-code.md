---
title: lodash源码解读
date: 2019-12-20 14:41:59
tags: [js, lodash, source code]
categories:
  - 前端
  - js
  - lodash
  - source code
---

# 开始之前

写这篇的初衷是想吸收借鉴一下框架库中的一些比较优秀的编程思维。

我选择的lodash版本是4.17.15，（[https://registry.npm.taobao.org/lodash/download/lodash-4.17.15.tgz](https://registry.npm.taobao.org/lodash/download/lodash-4.17.15.tgz)）。

lodash库的使用方式大致如下（以require形式举例）：

``` js
// 方式1
const _ = require('lodash')  // 指向 lodash/index.js

// 方式2
const _ = require('lodash/core') // 指向 lodash/core.js

// 方式3
const array = require('lodash/array') // 指向 lodash/array

/* End */
```

以方式1引用的`lodash/index.js`中包含lodash库的所有代码，而以方式3引用的`lodash/xx`中则只包含单个方法的实现。

所以我们可以通过两种方式来阅读lodash的源码：

  1. 直接从`lodash/index.js`中阅读 (`lodash/core.js`一样，但只包含一些基本方法)。

  2. 从`lodash/xxx`文件中挨个阅读单个方法的实现。

在开始阅读源码前，我们先看一下`lodash`库的文件分布：

  1. 文件平铺，`index.js`作为默认入口，指向`lodash.js`。

  2. 文件命名分为两种，`_`开头的表示公共函数封装，其他文件为供用户调用的方法。

  3. `fp/`与普通模块分开，`fp/`中模块具有：“纯函数”、“参数顺序调整”、“iteratee参数限制”、“完全柯里化”、“没有可选参数”五个特征。

`fp/`文件夹我们稍后再提。

# 从公共方法开始

<!-- more -->

我们先来看带有`_`的文件，从`_apply.js`开始：

## _apply.js

``` js
/**
 * A faster alternative to `Function#apply`, this function invokes `func`
 * with the `this` binding of `thisArg` and the arguments of `args`.
 *
 * @private
 * @param {Function} func The function to invoke.
 * @param {*} thisArg The `this` binding of `func`.
 * @param {Array} args The arguments to invoke `func` with.
 * @returns {*} Returns the result of `func`.
 */
function apply(func, thisArg, args) {
  switch (args.length) {
    case 0: return func.call(thisArg);
    case 1: return func.call(thisArg, args[0]);
    case 2: return func.call(thisArg, args[0], args[1]);
    case 3: return func.call(thisArg, args[0], args[1], args[2]);
  }
  return func.apply(thisArg, args);
}

module.exports = apply;

```

可以看到lodash在自定义的apply方法中，**对于参数多余3个的时候，才使用apply方法，否则使用call方法**。

这其实是一个很常见的封装操作：因为在于call的速度是要比apply更快的。（大概快4%左右）

`Function.prototype.call(thisArg [ , arg1 [ , arg2, … ] ] )`的调用步骤为:

  1. 若 Function 不可以被调用，则抛出一个 TypeError 异常。
  2. 定义 argList 为一个空列表。
  3. 如果使用超过一个参数调用此方法，则以从arg1开始的从左到右的顺序将每个参数附加为 argList 的最后一个元素。
  4. 返回调用func的[[Call]]内部方法的结果，提供 thisArg 作为该值，argList 作为参数列表

`Function.prototype.apply(thisArg, argArray)`的调用步骤为:

  1. 即 Function 不可以被调用，则抛出一个 TypeError 异常。
  2. 如果 argArray 为 null 或未定义，则返回调用func的[[Call]]内部方法的结果，提供thisArg 和一个空数组作为参数。
  3. 如果 Type（argArray）不是 Object，则抛出 TypeError 异常。
  4. 获取 argArray 的长度。调用 argArray 的 [[Get]] 内部方法，找到属性 length。 赋值给 len。
  5. 定义 n 为 ToUint32（len）。
  6. 初始化 argList 为一个空列表。
  7. 初始化 index 为 0。
  8. 循环迭代取出 argArray。重复循环 while（index < n）
    1. 将下标转换成String类型。初始化 indexName 为 ToString(index).
    2. 定义 nextArg 为 使用 indexName 作为参数调用argArray的[[Get]]内部方法的结果。
    3. 将 nextArg 添加到 argList 中，作为最后一个元素。
    4. 设置 index ＝ index＋1
  9. 返回调用func的[[Call]]内部方法的结果，提供 thisArg 作为该值，argList 作为参数列表。


**当然，你可能会说：“用ES6中的结构符`...`不就行了么？何必这么麻烦”。**

**请注意：**
  
  1. 作为函数库，lodash是要考虑兼容性的，所以短时间内不可能使用ES6+标准的代码。
  2. 作为项目代码，如果你的项目最终构建时没有使用babel等库进行编译，那么使用`...`+ call 代替apply是可行的；但是，如果你使用了babel，那么`a.call(null, ...[1, 2, 3])`将会被编译成`var _a;(_a = a).call.apply(_a, [null].concat([1, 2, 3]));`。


## _arrayEach.js

``` js
/**
 * A specialized version of `_.forEach` for arrays without support for
 * iteratee shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @returns {Array} Returns `array`.
 */
function arrayEach(array, iteratee) {
  var index = -1,
      length = array == null ? 0 : array.length;

  while (++index < length) {
    if (iteratee(array[index], index, array) === false) {
      break;
    }
  }
  return array;
}

module.exports = arrayEach;
```

一个`forEach`函数的封装，与`Array.prototype.forEach`的区别在于它是可以通过在`iteratee`函数中`return false`来终止循环的。

TIPS：在平时的编码过程中，我们也可以尝试着用while代替for循环；循环判断条件中，将数组length赋值给一个变量会具有更高的可读性和代码执行效率。

## _arrayFilter.js

``` js
function arrayFilter(array, predicate) {
  var index = -1,
      length = array == null ? 0 : array.length,
      resIndex = 0,
      result = [];

  while (++index < length) {
    var value = array[index];
    if (predicate(value, index, array)) {
      result[resIndex++] = value;
    }
  }
  return result;
}
```

一个过滤数组的方法，实现了数组的的filter方法。

可以从这个方法中学到的点是：

  这里定义了一个空数组`result`和一个变量`resIndex`，遍历数组将符合条件的元素通过`result[resIndex++] = value;`直接赋值到数组中。

  在我们平时的编码过程中，我们一般会习惯不定义`resIndex`而使用`result.push(value)`，但实际上**数组直接赋值的效率是要比push高很多的**。


















