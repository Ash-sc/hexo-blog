---
title: JS中的Generator
date: 2019-02-18 16:34:09
tags: [js, generator]
categories:
  - 前端
  - js
  - generator
---

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-26.jpg)

在js中，Generator一直是比较奇特的存在；它拥有十八般武艺却因为使用复杂、难以调试而没有得到广泛使用。

本篇主要介绍Generator基本语法以及经典用法。

<!-- more -->

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-26.jpg)

## 开始之前

首先，你需要知道这么几个概念。


### 1. 迭代器和迭代器对象

**迭代器对象是什么？**

    一个对象
    一个拥有next方法的对象
    一个next方法会返回具有done和value属性的对象的对象。


**PS：** 迭代器是函数，迭代器对象是迭代器函数返回的对象。

简单迭代器（函数makeIterator）：

``` js
function makeIterator (arr) { // 我是迭代器
    let nextIndex = 0
    return {
        next() {
            return nextIndex < arr.length ?
              { value: arr[nextIndex ++ ], done: false } :
              { done: true }
        }
    }
}

const it = makeIterator(['a', 'b']) // 我是迭代器对象
console.log(it.next().value) // 'a'
console.log(it.next().value) // 'b'
console.log(it.next().done) // true
```

### 2. 可迭代对象

**可迭代对象是什么？**

一句话来讲：一个定义了迭代行为（`Symbol.iterator`属性）的对象。（杠精远离，这只是为了方便理解定义）

可迭代对象具有`Symbol.iterator`属性并且`Symbol.iterator`属性值是一个迭代器（函数）。

可迭代对象可以使用`for...of`来循环迭代器中的值。

**PS: ** `String`，`Array`，`TypedArray`，`Map` 和 `Set` 都具有默认迭代行为。

可迭代对象示例：

``` js
const obj = {}

function makeIterator (arr) {
    let nextIndex = 0
    return {
        next() {
            return nextIndex < arr.length ?
              { value: arr[nextIndex ++ ], done: false } :
              { done: true }
        }
    }
}

obj[Symbol.iterator] = makeIterator.bind(null, ['a', 'b'])

for (let value of obj) {
    console.log('value :', value)
}
// value: a
// value: b
```
    
### 3. 生成器

上面的迭代器函数`makeIterator`是自定义迭代器，但由于需要显式地维护其内部状态（`nextIndex`），因此需要谨慎地创建。

使用生成器`Generators`可以定义一个包含自有迭代算法的函数，同时它也可以自动维护自己的状态。（https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators）

什么意思呢？就是说：我们用`Generators`定义的迭代器，不需要维护`nextIndex`，而是采用`yield`语法表示每一次迭代的输出。每一次调用`next()`方法后，代码执行至一个`yield`（执行完yield xxx即停止）

**PS：** `yield a`本身不返回值，虽然可以通过`Generators.next()`获取`a`的值。

定义一个`Generators`的语法：`function* gen() { ... }`。

让我们用`Generators`来实现之前的`makeIterator`。

``` js
const obj = {}

obj[Symbol.iterator] = function* (arr) {
    for (item of arr) {
        yield item
    }
}.bind(null, ['a', 'b'])

for (let value of obj) {
    console.log('value :', value)
}

// value: a
// value: b
```

**生成器还有一个特点**：传递给`next()`的值将被视为暂停生成器的最后一个yield表达式的结果。（摘自MDN文档）

什么意思呢？可以理解为：使用`next(value)`时，设置生成器***上一次***`yield`表达式返回值为`value`。

我们可以来看看MDN文档中的例子：

``` js
function* fibonacci() {
  var fn1 = 0
  var fn2 = 1
  while (true) {  
    var current = fn1
    fn1 = fn2
    fn2 = current + fn1
    var reset = yield current
    console.log(reset) // 这里输出查看reset的值
    if (reset) {
        fn1 = 0
        fn2 = 1
    }
  }
}
var sequence = fibonacci()
console.log(sequence.next().value) // 0
console.log(sequence.next().value) // 1
console.log(sequence.next().value) // 1
console.log(sequence.next().value) // 2
console.log(sequence.next().value) // 3
console.log(sequence.next(true).value) // 0
```
一般情况下`yield current`是不会有返回值的，所以`reset`的值为undefined。

**但是**，可以给next()方法传入值来改变**上一次**yield表达式的返回值。

当我们运行`sequence.next(true)`时，会把**上一次**（console输出3的那次）`yield current`返回值由`undefined`变成了`true`。

所以`sequence.next(true)`调用时，代码执行顺序为：

``` js
// yield current 返回值置为 true
var reset = yield current// “yield current”在上一次next()执行完了，这一次执行的是左半部分（=操作右结合)
// reset = true
console.log(reset) // true
if (reset) { // 判断成立
    fn1 = 0
    fn2 = 1
}
while (true) { // 进入while
var current = fn1 // currnet = 0
fn1 = fn2
fn2 = current + fn1
var reset = yield current // 执行右半部分， next()方法返回{ done: false, value: 0 }
// 停止，var reset = yield current 的左半部分留给下一次 next() 调用时执行
```


## Generator可以做什么

OK，上面介绍了迭代器、可迭代对象、生成器的基本概念和用法。那么，生成器`Generator`可以用来做些什么呢？

对于`Generator`，每一次调用`next()`方法或者`for...of`循环体每执行一次，都会将`Generator`中的代码执行到下一个`yield`。

利用这个特点，我们可以使用`Generator`来做很多事情。

比如：

### 1. 无限长度的数列

这个应该是比较常见的用法了，比如MDN文档中的`斐波拉契数列`。

我们不需要去维护迭代器的状态就可以通过`next()`方法获取到数列的下一个值。


### 2. 同步表达异步操作

示例：

``` js
// 模拟请求，一秒延迟
const mockRequest = () => new Promise(resolve => setTimeout(_ => {
    const num = Math.random()
    console.log('num :', num)
    resolve({ num })
}, 1000))

const tasks = function* () {
    const a = yield mockRequest()
    const b = yield mockRequest()
    return a.num + b.num
}

const run = (gen) => {
    gen = gen()

    const next = ({ done, value }) => {
        return new Promise(resolve => {
            if (done) return resolve(value)
            else {
                value.then(data => {
                    next(gen.next(data))
                      .then(res => resolve(res))
                })
            }
        })
    }

    return next(gen.next())
}

run(tasks).then(data => {
    console.log('sum :', data)
})
```

未完待续...

