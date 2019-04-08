---
title: 一道前端面试题
date: 2018-04-27 20:46:03
tags: [前端, 面试题]
categories: 前端
---

今天在公交车上的时候，某K给我发了一道前端面试题让我解答。

然后， 我看第一眼的时候，被绕了进去……(╯-_-)╯~╩╩

<!-- more -->

## 多了一个slice(0)

题目如下，问两段代码console的结果有什么区别：
``` js
// 第一段代码
var arr = [0,1,2,3,4,5,6,7,8,9];
var res = [0,0,0,0,0,0,0,0,0,0];
var t = 1000;
for(var i = 0; i < t; i++){
  var sorted = arr.slice(0).sort(() => Math.random() - 0.5);
  sorted.forEach((o,i) => res[i] += o);
}
res = res.map(o => o / t);
console.log(res);

// 第二段代码
var arr = [0,1,2,3,4,5,6,7,8,9];
var res = [0,0,0,0,0,0,0,0,0,0];
var t = 1000;
for(var i = 0; i < t; i++){
  var sorted = arr.sort(() => Math.random() - 0.5);
  sorted.forEach((o,i) => res[i] += o);
}
res = res.map(o => o / t);
console.log(res);
```
说实话，第一眼看到的时候，我觉得这两段代码的执行结果并不会有什么区别。

然而，真实的执行结果：

![console-output](http://web-site-files.ashshen.cc/blog/font-end-question/run-result.png)

两份代码唯一的区别在于第一份代码多了一个`slice(0)`。

首先我们需要知道：**数组的sort方法是会改变源数组的，并且会返回排序后的数组；而slice(0)则是返回一个浅拷贝的新数组**。

所以，第一份代码中：1000次循环，每一次得到的`sorted`都基于[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]；而第二份代码中：1000次循环，每一次得到的`sorted`都基于前一次循环后的`sorted`。

那么，为什么第一份代码得到的res数组元素会是依次递增，而第二份代码得到的res数组元素是相似的值呢？

## sort()和Math.random()

JS中，数组原生的sort方法采用的排序算法是*快速排序*，假设你写了这样一行代码：
``` js
[1, 2, 7, 4, 5].sort((a, b) => { console.log(a, b);return a > b; })
```
那么，你会在浏览器中看到如下执行结果：

![output](http://web-site-files.ashshen.cc/blog/font-end-question/arr-sort.png)

也就是说，sort方法执行时：从数组的第二个元素开始（假设它为m），依次把该元素（m）与它前面的元素（从arr[0]开始：n）进行比较，如果return的值小于0，则把该元素（m）插入到比较元素（n）前面，否则，继续与下一位元素比较（n的下一位）。

OK，回过头来我们看之前的代码。我们之前已经分析过：两份代码的区别在于每一次sort的源数组不同。

那么我们可以做一下这样的假设对于下面这样一段代码：
``` bash
[0, 1, 2, 3, 4, 5].sort(() => Math.random() - 0.5);
```

如果要得到形如[x, x, x, x, x, 0]这样的数组（也就是0在最后一位），需要**5次Math.random() - 0.5的值都大于0**，也就是说概率大概是：0.5^5 = 3.125%左右。而如果每次sort的数组都是上一次sort之后的数组的话，第n次得到[x, x, x, x, x, 0]的几率则会大大增加。

PS：这两份代码就好比给你1000个游戏币，第一份代码代表你每次死了只能从第一关重新开始，第二份代码则代表你可以从已经上次通过的关继续玩，很明显，通关的概率有很大的差别。

所以，当循环的次数越多，第二份代码得到数组混乱程度也就越大，当我们对其取平均值时，则会发现最后得到的res数组各个元素值会很平均。

以下为t = 10, t = 100, t = 1000, t = 10000, t =100000时两份代码执行结果：

![result](http://web-site-files.ashshen.cc/blog/font-end-question/final-result.png)

可以看到，t越大，这样的特征是越明显的。
