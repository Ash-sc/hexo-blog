---
title: 前端笔记（持续更新）
date: 2018-04-04 20:39:54
tags: [fe, note]
---

探索知识的过程总是让人兴奋的，尤其是当你发现你所了解的知识对于接受新的知识有帮助，并且他们之间是有关联的时候。

## GZIP

（这里仅仅针对web前端中的应用，不讨论其算法实现）

gzip在前端是很常见的一种压缩（编码）方式，同时也是效果比较显著的一种网页优化手段。

那么，服务端是怎么知道浏览器支持gzip压缩呢？浏览器是怎么知道服务端传输文件的是gzip压缩过的呢？

下面这张图是一个使用了gzip压缩传输的一个http请求：

![gzip-console](http://web-site-files.ashshen.cc/font-end-notes/gzip.png)

**图中的两个箭头标注了两个字段**

一个是请求头（`Request Headers`）中的`Accept-Encoding`字段

另一个是响应头（`Response Headers`）中的`Content-Encoding`字段

这两个字段分别表示的是浏览器能够支持的编码方式、以及服务器返回的内容编码的方式。通过这两个字段，客户端与服务端直接关于http请求的编码也就能够统一了。

### gzip的两种常用方式：

1. 服务端在收到客户端发起的http请求后，对（静态）资源进行压缩，然后传输给客户端。（当然，服务端会使用缓存来减少服务端压力）

2. 服务器直接存储压缩后的.gz文件，当服务端收到http请求（eg: index.html）时，返回对应的压缩文件（index.html.gz）。（这种场景通常使用于vue、react等spa框架构建的项目）

### gzip压缩级别：

gzip压缩是有级别的（1-9）。级别越高压缩强度越大，同时消耗的cpu资源也越高。

以下是nginx中开启gzip的部分配置：

``` bash
gzip on; # 打开gzip压缩
gzip_http_version 1.0; # 启用gzip的最低http版本
gzip_disable "MSIE [1-6]."; # 禁用gzip的条件，这里表示ie6以下不使用gzip（兼容性）
gzip_types text/plain application/javascript text/css text/javascript application/x-httpd-php; # 使用gzip压缩的静态资源类型 
gzip_min_length 1000; # 小于1000b的文件不使用gzip压缩
gzip_proxied expired no-cache no-store private auth; # 反向代理相关
```

### 不推荐使用gzip压缩图片：

像jpg、png之类格式的图片，本身就有针对文件体积做压缩处理，所以使用gzip的意义不大，反而浪费服务器资源。

*PS*：页面性能优化中，我们经常提到可以使用Base64来代替图片url，从而减少http请求的数量，达到优化的目的。但是，Base64也不是万能的，同一张图片的Base64的体积是要远大于jpg、png这样的格式的。但是嘛！！！如果是将Base64编码后的图片再进行gzip压缩的话，得到的图片大小与jpg、png的体积是没有多大差别的。

## Base64

Base64是一种基于64个可打印字符来表示二进制数据的表示方法（维基百科）。简单来说就是使用64个字符来表达0-63从而表示二进制数据（2^6 = 64）。

Base64不仅仅可以用来表示图片，它还可以用来字体文件（.ttf等），也有一些网页应用的数据请求返回也是用Base64格式。

### 优点：

从前端来讲，Base64最大的优点应该算是使用它来替代图片url或者字体文件从而减少http请求数量了。

### 缺点：

同样，Base64的缺点也十分明显：

1. 体积问题

2. 缓存问题

3. 性能问题

*体积问题：*

Base64表示的图片体积相较于jpg、png格式的同一图片的体积会要大很多。（当然，这个问题可以通过Gzip压缩来解决）

*缓存问题：*

减少http请求就意味着合并，合并也就意味着当一个部分发生变化时，整体资源都需要重新加载。打一个很简单的比方：CSS文件a中使用了base64格式的图片，那么，当a文件中的任意代码发生变化（哪怕只是一个字母）时，整个a文件都要重新从服务端获取。这也就意味着你无法单独缓存a中的图片以及字体这一类非频繁变化的资源。

*性能问题：*

首先我们要知道，当浏览器解析渲染一个网页时：图片资源（url）是不会阻塞浏览器解析渲染的，但是css则会。也就是说：*当你的css文件没有加载完成前，浏览器是无法构造渲染树的（因为无法生成CSSOM树）*。所以，如果你在css中使用了Base64的样式（background-image、@font-face等）的时候，这些资源将会影响你的页面加载。

PS：假设你的CSS文件中有使用媒体查询来根据不同的分辨率使用不同的图片作为background-image。那么，如果你使用的是url的图片资源，那么浏览器只会根据实际情况下载其中的一个（浏览器自身优化）；但是，如果你使用的是Base64格式的图片，浏览器则会**下载全部**。

举个栗子：
``` css
.section-body {
  background-image: url(https://www.xxx.com/.../image-normal.png); // 方案1： 使用url
  background-image: url('data:image/png;base64,...') // 方案2：使用base64
}
@media screen and (max-width: 800px) {
  .section-body {
    background-image: url(https://www.xxx.com/.../image-middle.png); // 方案1： 使用url
    background-image: url('data:image/png;base64,...') // 方案2：使用base64
  }
}
@media screen and (max-width: 600px) {
  .section-body {
    background-image: url(https://www.xxx.com/.../image-small.png); // 方案1： 使用url
    background-image: url('data:image/png;base64,...') // 方案2：使用base64
  }
}

#### 当屏幕宽度为500px时，方案1仅会下载image-small.png，而方案2无论宽度大小，都会下载3张图片的base64数据
```

总的来说，使用Base64来表示静态资源的时候，我们需要找到它的优缺点之间的平衡点（比如：小图片、图片验证码可以考虑使用Base64）。并没有什么方案是绝对最优的，因地制宜才是关键。




