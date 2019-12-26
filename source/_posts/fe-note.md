---
title: 前端笔记
date: 2019-12-25 16:27:54
tags: [fe, note]
categories: 前端
---

## gzip

gzip在前端是很常见的一种压缩（编码）方式，同时也是效果比较显著的一种网页优化手段。

那么，服务端是怎么知道浏览器支持gzip压缩？浏览器是怎么知道服务端传输文件的是gzip压缩过的？

<!-- more -->

下面这张图是一个使用了gzip压缩传输的一个http请求：

![gzip-console](http://web-site-files.ashshen.cc/font-end-notes/gzip.png)

**图中的两个箭头标注了两个字段**

一个是请求头（`Request Headers`）中的`Accept-Encoding`字段

另一个是响应头（`Response Headers`）中的`Content-Encoding`字段

这两个字段分别表示的是浏览器能够支持的编码方式、以及服务器返回的内容编码的方式。通过这两个字段，客户端与服务端直接关于http请求的编码也就能够统一了。

### gzip的两种常用方式

1. 服务端在收到客户端发起的http请求后，对（静态）资源进行压缩，然后传输给客户端。（当然，服务端会使用缓存来减少服务端压力）

2. 服务器直接存储压缩后的.gz文件，当服务端收到http请求（eg: index.html）时，返回对应的压缩文件（index.html.gz）。（这种场景通常使用于vue、react等spa框架构建的项目）

### gzip压缩级别

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

### 不推荐使用gzip压缩图片

像jpg、png之类格式的图片，本身就有针对文件体积做压缩处理，所以使用gzip的意义不大，反而浪费服务器资源。

*PS*：页面性能优化中，我们经常提到可以使用Base64来代替图片url，从而减少http请求的数量，达到优化的目的。但是，Base64也不是万能的，同一张图片的Base64的体积是要远大于jpg、png这样的格式的。但是嘛！！！如果是将Base64编码后的图片再进行gzip压缩的话，得到的图片大小与jpg、png的体积是没有多大差别的。

### gzip的算法原理

gzip对于要压缩的文件，先使用LZ77算法变种进行压缩，然后使用Huffman编码进行压缩，下面简要说明其原理。

LZ77算法的压缩原理：如果文件中有两块内容相同，那么我们只需要知道前一块的大小和位置，就可以用前后相同内容两者之间的距离和内容长度来替换后一块的内容，从而达到压缩的目的。

而Huffman算法则是把文件中一定长度的值看做是符号，然后根据这些符号在文件中出现的频率对其重新编码，对于次数出现较多的，用较少的位数来表示，反之则用较多的位数来表示，这样一来文件的一部分位数变小，另一部分位数变大，由于变少的部分比变大的部分多，所以整个文件的大小还是会变小，达到压缩的目的。

当然，这只是两种算法的基本原理，真正的实现方案还有很多部分，比如如何确定两个内容块相同，如何构建一个huffman树来确定字符出现频率等。

**小结：**

**gzip是一种常用的网页优化手段，具有1-9不同的压缩级别，图片类不适用，基于LZ77变种+Huffman算法实现。**

## http传输压缩方式

http请求报文压缩也是比较常见的一种性能优化手段。常见的压缩方式有：gzip、deflate、brotli等。

浏览器在发送http请求时，会在请求头部Accept-Encoding中携带浏览器支持的压缩格式，而服务器在返回数据时，同样会在返回头部content-encoding中表明实际使用的压缩格式。

gzip上面有提到，也是目前普及性比较高（应该是最高）的一种http压缩方式了。常用的表现方式为：
  
  1. 服务端将文件打包成.gz，浏览器http请求时直接将.gz文件返回。 （webpack插件中有集成）
  2. 服务端在接收到http请求时，将文件gzip压缩后返回。（会利用缓存提高转换效率）

deflate这个名字容易让人产生误会，在http传输压缩方式中，deflate实际表示的是RFC1950描述的ZLIB编码格式。它比gzip会更加的高效，但是由于命名容易产生误解的原因，gzip得到了更加广泛的应用。

brotli最初发布于2015年，而后google在2015年9月发布了侧重于http压缩的无损数据压缩格式：brotli增强版。但是其支持性并不是太好，在浏览器accept-encoding中标示值为：br。开启brotli需要https支持。

其他的压缩方式：sdch。

sdch通过字典压缩算法对各页面相同内容进行压缩，主要分为3个部分：首次请求，下载字典，之后的请求。

**小结：**

**gzip最常用，deflate比gzip好但是有歧义导致应用不多，brotli比gzip也要好但是得上https，sdch几乎没人用。**

## 常见压缩文件格式

比较常见的压缩文件格式有：zip、rar、7z、tar、tar.gz、tar.bz等

那么，他们分别有什么特点呢？

首先是zip。zip是一种比较常见的压缩格式，其算法不定，一般为DEFLATE，兼容性比较好，不过可能会出现乱码的问题（因为文件名使用非Unicode编码）。

rar和7z比zip要新，算法压缩的效率也更高，但是兼容性没有zip好，rar的编码器有专利，而7z的文件和管理程序都是开源的。

在Linux中，tar一般和其他没有文件管理的压缩算法结合使用，如tar.gz、tar.bz。用tar打包整个文件目录结构成一个文件，再用gzip/bzip等压缩算法压缩一次，是Linux中最常见的压缩方式。

**小结：**

**zip是最常见的压缩方式，兼容性最好但是可能会有乱码问题；rar和7z兼容性没有zip好，其中rar的编码器有专利7z开源；tar只是文件管理，不具备压缩功能，经常和其他压缩算法一起使用，所以我们常常见到tar.gz、tar.bz这样的后缀.**

## Base64

Base64是一种基于64个可打印字符来表示二进制数据的表示方法（维基百科）。简单来说就是使用64个字符来表达0-63从而表示二进制数据（2^6 = 64）。

Base64不仅仅可以用来表示图片，它还可以用来字体文件（.ttf等），也有一些网页应用的数据请求返回也是用Base64格式。

### Base64优点

从前端来讲，Base64最大的优点应该算是使用它来替代图片url或者字体文件从而减少http请求数量了。

### Base64缺点

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

/* 当屏幕宽度为500px时，方案1仅会下载image-small.png，而方案2无论宽度大小，都会下载3张图片的base64数据 */
```

**小结：**

**Base64是一种二进制表示方法，所以表示图片时体积会变大（图片都有压缩优化算法的），小图片更加适合使用base64，因为可以减少一次http请求。**

## 缓存

缓存是前端知识体系中一个比较重要的点，也是面试中经常考察的一个点。

前端缓存主要分为：

  1. 强缓存 200(from cache)
  2. 协商缓存 304

在http中，缓存主要由请求的头部（headers）来控制（还有一部分根据不同浏览器表现不同）

### 强缓存

强缓存意味着浏览器不会发送http请求资源，直接使用本地的缓存。

强缓存控制手段：

  1. Expires。过期时间（http/1.0），为时间戳，表示该时间之前都不向服务器发送http请求。
  2. Cache-control。缓存控制（http/1.1）具有多种参数值，可以组合使用。

Expires的缺点在于，服务器时间和浏览器时间不一致，所以服务器返回的过期时间也是不准确的。

Cache-control是目前主要使用的强缓存手段（优先级高于Expires），其值可能有：

  1. max-age=xxx，单位为秒，表示在xxx秒之内都使用本地缓存。
  2. private，表示只有当前浏览器缓存，中间代理服务器不缓存。
  3. public，表示当前浏览器和中间代理服务器都可以缓存。
  4. no-cache，不使用强缓存，使用协商缓存。
  5. no-store，不使用任何缓存。
  6. s-maxage=xxx，类似max-age，针对代理服务器的过期时间。


### 协商缓存

协商缓存的意思是：浏览器向服务器询问资源是否过期；过期则返回新资源，否则继续使用缓存。

协商缓存主要由以下几个字段控制：
  1. Last-Modified，文件最后修改时间。
  2. Etag，文件唯一标识。

Last-Modified是一个时间戳，第一次请求时服务端返回头中携带。与Expires不同的是，不是由浏览器来验证是否过期，而是浏览器在发送http请求时，头部If-Modified-Since字段中带上此值，服务器验证是否修改过，从而决定返回304使用缓存还是200并返回新资源。

由于Last-Modified是文件创建时间戳，所以只能精确到秒，而且对于文件内容没有修改但是重新生成了的情况，会造成缓存失效。

而Etag，采用的是文件唯一标识（默认根据文件索引节点INode+大小Size+最后修改时间MTime进行hash得到），比Last-Modified更加精准。

Etag优先级高于Last-Modified。

### 缓存位置

在浏览器存储缓存时，大致会有以下几种存储方式：

  1. service worker
  2. memory cache
  3. disk cache
  4. push cache

Service worker是pwa的重要实现机制，属于离线缓存。

memory cache是存储在内存中的缓存，是最快的同时也是存活时间最短的。

disk cache是存储在磁盘的缓存，存活时间较长，但是效率不及memory cache，内存总是有限的而磁盘则更加宽裕。

push cache是http/2的内容。

**小结**

**HTTP强缓存有Expires时间戳和Cache-control实现，Expires属于http/1.0优先级低于http/1.1的Cache-control。**

**HTTP协商缓存有Last-Modified和Etag，Last-Modified也是时间戳，不靠谱，优先级低于基于hash的Etag。**

**所有的时间戳都不靠谱，因为浏览器/服务器时间不一致并且他们粒度只能到秒级别。**

**浏览器存储缓存的方式可以有server worker/memory/disk cache和http2的push cache。**











