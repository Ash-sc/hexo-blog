---
title: Vue+Node爬虫webapp（持续更新中…）
date: 2018-01-09 20:13:43
tags: [vue, spider]
---

### 事情起因：

由于想要看电影，但是又不想每次了解最新的电影信息都要去某宝之类的App上找那些藏得很深的小程序（当然，买票的时候还是得向大佬低头）。

同时又喜欢看看微博，逛逛知乎，掘金什么的；各种App切来切去很麻烦的。

然后呢？就有了这么一个想法：把这些的东西杂糅到一个webapp中。然后顺便熟悉一下webapp前端的相关知识。

Emmmm，东西有点多。工作之余慢慢写吧。


OK，按照惯例，先上Github地址：https://github.com/Ash-sc/node-spider-webapp

和项目Demo地址：http://2017017.xyz/ （建议手机端食用）。

## 技术选型

前端：Vue，没什么好说的。项目结构的话，主体继续沿用之前Notebook项目那一版，然后做了一些微调。

后端：Node，这次不用express了，试了一下koa，感觉还好，async和await用起来比较舒服。数据的话，继续用superagent写爬虫，后续加入账户体系后再考虑数据库。

## 电影模块

既然需求是由电影引起的，那么就从电影开始。

备选的数据源的话，有IMDB和豆瓣，不过鉴于某渣的英语水平以及实际需求是国内上映的电影，所以数据源最终也就决定是豆瓣了。

在用爬虫爬豆瓣的数据之前，我先google了一下豆瓣Api，然后发现豆瓣还真的有供开发者使用的Api，然鹅当进入到应用页面后，却出现了下面这个尴尬的情况（摔）。

![douban-api](//web-site-files.ashshen.cc/blog/spider-webapp/douban_api.png)


mmp…… 还是老老实实用爬虫吧…

后端数据爬取没什么难度，superagent直接爬取对应页面的数据，然后使用cheerio提取需要用到的部分（名称、链接、评分、图片等）。

然后，第一版的电影列表页面效果大概是这样子的：（2018-01-10）

<video src="//web-site-files.ashshen.cc/blog/spider-webapp/movie-version01.mp4" width="300" height="550" controls="controls"></video>

电影列表页用双列展示，同时左右两列垂直错开2rem（40px）的像素，使列表页看上去不是那么的平淡。正下方是一个fixed定位的tab，后续考虑把这一块稍微优化一下。

咳咳咳，双列布局不是仿照的淘宝商品页，而是来源于一款叫做“尤果圈”的App（新世界大门在此打开，手动滑稽）。

### 电影详情页

详情页面，倒是与一般的电影App详情页没太大的出入。目前省略掉了演员表部分，PS：在添加电影宣传片部分的时候，原计划是直接用video标签引用豆瓣的宣传片链接，但是豆瓣的服务器那边对视频资源做了限制，不能直接访问。所以宣传片这部分后续再考虑一下解决方案。

第一版页面效果如下：

<img src="//web-site-files.ashshen.cc/blog/spider-webapp/movie-detail-view.png" width="300" />

电影模块，第一版本需求大致差不多了。当然，还需要解决掉宣传片的问题，这个是一定不能少的。

### 添加影片宣传片预览（2018-01-12）

由于豆瓣服务器对宣传片的资源做了限制，不能直接使用它来作为video标签的src。所以，我在Node中做了一层缓存。

大致步骤为：

1. 获取影片详情时，同时通过request模块获取到宣传片资源，然后使用pipe()写入到服务器文件中，缓存起来。

2. 前端调用宣传片资源时，调用同一个接口，id作为参数，服务端将视频流分成多个部分返回给前端（206）。

3. 服务端有一个定时任务，定时清空过期无效的宣传片文件。

PS：在此之前我也尝试过直接request模块获取到全部的视频流，但是在safari浏览器和iphone上会导致视频无法播放。

实现后的电影详情页面大致如下：

<video src="//web-site-files.ashshen.cc/blog/spider-webapp/movie-detail-version01.mp4" width="300" height="550" controls="controls"></video>





