---
title: https初探
date: 2017-08-17 19:36:40
tags: [https]
categories: https
---

最近在琢磨着把博客升级成https，于是也就去看了一些关于https的资料，把自己学到的一些知识写成笔记，以供自己将来回顾，当然，也为有需要的人提供一份参考。

<!-- more -->

## 什么是http

http的全称为Hypertext Transfer Protocol，即超文本传输协议，是一种属于应用层的协议。

## http的缺点

    使用明文传输，内容有被窃取的风险

    不验证通信双方的身份，可能被与被伪装的对象交流

    无法验证报文的完整性，因此得到的报文可能已经被篡改

因此，http不适合传输一些敏感的信息，比如：银行卡卡号、支付密码等。

## https

https并不是一种新的协议，只是在http下加入了一层ssl层，所以https的基础是ssl，https的加密也是通过ssl来完成的。

ssl协议采用的是公钥加密算法：客户端向服务器索要公钥，然后使用服务器返回的公钥加密信息，服务器使用私钥来解密信息。

*http与https*

（http在TCP完成三次握手后即可开始报文的传输，而https在完成TCP三次握手之后，还需要进行ssl的四次握手，然后开始报文传输）

![http-vs-https](http://web-site-files.ashshen.cc/blog/http&https.png)

这样一来，ssl也有两个问题需要解决：一、保证公钥不被截取篡改（公钥的传输也是明文的）；二、加解密的计算开销过大。

对于第一个问题，ssl的解决办法是：**服务器不会直接将公钥传给客户端，而是将公钥放在证书中传输给客户端，只要保证证书的安全可信，公钥自然也就能够安全可信。**

至于加解密计算开销过大的问题，在每次对话（TCP连接）中，客户端和服务端会都生成一个对话密钥，然后用对话密钥来加密信息，由于对话密钥使用的是对称加密，所以运算速度很快，而客户端只需要在ssl四次握手的过程中将对话密钥使用服务器提供的公钥加密后传输给服务端就行（这句话可能会有点拗口，没关系，下面还有图解）。

**ssl四次握手过程：**

![https-hands-shake](http://web-site-files.ashshen.cc/blog/ssl-shake.png)

*第一步*，客户端向服务器发起请求，请求包括一个随机数、客户端支持的ssl协议版本、加密算法以及压缩算法。

当第一步完成后，此时：服务器和客户端都拥有了一个随机数。

*第二步*，服务器响应客户端的请求，并向客户端发送了第二个随机数、接下来将要使用的ssl协议版本、加密算法以及压缩算法，还有带有公钥的证书。

当第二步完成后，此时：服务器和客户端都拥有了两个随机数，并且ssl协议版本、加密算法、压缩算法都已经确定，而客户端也拿到了带有公钥的证书。

*第三步*，客户端响应服务器的请求，并向服务器发送了使用第二步获取到的公钥加密的第三个随机数，以及编码改变通知和握手结束通知。

当第三步完成后，此时：服务器和客户端都拥有了三个随机数，并且第三个随机数的传输过程是加密的，只有服务器拥有解密用的私钥，因此保证了安全性。

*第四步*，服务器响应客户端的请求，向客户端发送了编码改变通知和握手结束通知，表示之后的报文都使用双方约定的加密方式以及加密密钥（三个随机数组成）发送。ssl握手到此结束。

*PS*：之所以使用三个随机数组成加密密钥，只是为了保证加密密钥的唯一性。

*再PS*：一般一个站点只有一个公钥，也是就说一般而言，一个站点的TCP连接第二步中，服务器传回的证书都是同一个，只是第三步加密的随机数不一致。

## 总结

看到这里，相信你对https已经有了初步的了解了。

简而言之，https并不是独立于http之外的另一种协议，只是在TCP连接建立之后http通信之前，插入了ssl协议，然后使用加密密钥对http中传输的明文进行了加密传输。

当然；https并不是万能的，它也有一些缺点，比如：由于https在建立TCP连接之后还需要进行ssl四次握手，所以频繁的重建ssl，会对服务器造成巨大的压力；由于https使用密文传输，那么加密通信必然会对服务器造成更多的CPU资源和内存资源的消耗。


参考资料：

  阮一峰《SSL/TLS协议运行机制的概述》[http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)



