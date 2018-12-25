---
title: 前端iframe跨域通信
date: 2017-12-06 20:08:15
tags: [iframe, webpage]
categories: 前端
---

关于前端iframe之间的跨域通信方式。

<!-- more -->

近日开发项目时遇到一个需求，大致就是：A页面里面用iframe嵌套了一个B页面，希望用户在B页面进行操作（B页面路由改变）后，刷新整个页面，iframe中显示B页面操作后的路由。

很明显，这是一个网页跨域通信的问题，由于浏览器同源策略，我并不能用一个timer去轮询获取iframe的contentWindow中的信息。

于是我花了一些时间去了解了一下网页跨域通信相关的方法。再然后，我看到了这篇文章[https://www.ibm.com/developerworks/library/wa-crossdomaincomm/](https://www.ibm.com/developerworks/library/wa-crossdomaincomm/)，并在其中找到了适合我的解决办法。

解决完我的问题之后，我把这篇文章重新看了一遍，并将其中提到的几种方法用简单的demo实践了一遍，于是便有了这篇博客。

Github代码：[https://github.com/Ash-sc/cross-domain-communication](https://github.com/Ash-sc/cross-domain-communication)

在线Demo：[http://eg1.2017017.xyz/cdc](http://eg1.2017017.xyz/cdc) （服务器在国外，可能会比较慢）

## 方法一、document.domain

此方法适用于需要通信的两个页面属于同一个根域名下（eg：eg1.aaa.cc与eg2.aaa.cc都属于aaa.cc的子域名）

使用方法：只需要分别在A、B两个页面**设置document.domain为公共的根域名即可**。

至于为什么只适用于相同根域名下的两个子域名之间的跨域通信：

原因很简单，使用document.domain设置的根域名只能是当前页面URL的根域名，也就是说：如果A页面的访问地址是eg1.example.com，那么document.domain只能设置为example.com。

demo：[http://eg1.2017017.xyz/cdc/1-document.domain/page1.html](http://eg1.2017017.xyz/cdc/1-document.domain/page1.html)

## 方法二、hash

方法二没有子域名的限制，适用性比方法一稍广。

方法二的原理在于：**当URL发生变化任何时，网页将会刷新。但是hash值的变化却不在此列（也就是URL中”#”及后面部分**），所以我们可以利用hash来做一些事情。

方法二的流程大致是这样的，当A页面需要与iframe中的B页面通信时：

1. A、B页面分别使用timer轮询监听自身URL中hash值变化

2. A页面js事件修改iframe的src，添加hash值

3. B页面监听到自身hash值变化，做出相应的响应

4. B页面使用”parent.location.href”修改A页面URL中的hash值

5. A页面监听到自身hash值变化，得到响应数据

demo：[http://eg1.2017017.xyz/cdc/2-hash/page1.html](http://eg1.2017017.xyz/cdc/2-hash/page1.html)

但是，目前市面上的一部分网页中的hash值已经用于其他用途（比如SPA项目用来作为前端路由）。当这些页面想要使用hash来进行跨域通信时，就比较复杂了。

## 方法三、window.name

window.name这一属性原本应该是用来描述浏览器窗口名称的（然鹅现在几乎用不到它了）。

由于window.name用于窗口，所以**无论窗口中的内容与地址如何变化，window.name都不会发生变化，除非手动赋值**。

利用window.name进行跨域通信的步骤大致如下：

1. 假设A页面(eg1.example.com/xxx)需要与B页面(eg2.example.com/xxx)进行跨域通信

2. 在A页面打开一个隐藏的iframe，设置iframe的src值为向B页面通信的请求(eg1.example.com/xxx?name=aa&age=11&…)

3. B页面响应该请求，并且将请求的响应数据保存在window.name中

4. 此时，A并不能通过iframe.contentWindow.name访问到iframe中的window.name；所以B页面需要在第3步保存完window.name后，将页面导向至A页面所在域名的某个目录

5. 此时A页面就可以通过iframe.contentWindow.name获取到保存在iframe的window.name中的数据了，因为此时的iframe与A页面属于同一域下

demo：[http://eg1.2017017.xyz/cdc/3-window.name/page1.html](http://eg1.2017017.xyz/cdc/3-window.name/page1.html)

## 方法四、postMessage

postMessage是Html5新增的特性，本身就用与安全跨域通信。所以，如果你的用户浏览器环境支持Html5，那么可以考虑这种方法。

使用postMessage方法进行跨域通信比较简单：

    1. A页面需要与B页面(iframe)进行跨域通信

    2. A页面window对象添加事件监听”window.addEventListener(‘message’, (e) => { console.log(e.data), false})”

    3. B页面需要向A页面发送数据时，使用”parent.window.postMessage(‘消息内容’, ‘*’)”即可。parent.window.postMessage第一个参数为需要传递的消息内容，第二个参数为需要传递的域名（’*’表示所有）。

demo：[http://eg1.2017017.xyz/cdc/4-postMessage/page1.html](http://eg1.2017017.xyz/cdc/4-postMessage/page1.html)

以上四种方法都可用于前端跨域通信，可根据实际业务场景灵活选用。
