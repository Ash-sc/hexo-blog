---
title: 用vue撸了一发Evernote网页版
date: 2017-11-07 19:55:14
tags: [vue, notebook]
categories:
  - 前端
  - vue
---

在很久之前曾用过一段时间的Evernote，后来由于在手机上经常卡顿，也就弃坑了改用Wiznote。然鹅Wiznote也不是个什么好鸟，相较于Evernote对于免费用户只能同时绑定两个客户端而言，Wiznote居然对免费用户限制笔记数量。好吧，不能忍了，刚好最近这段时间也在看Vue，所以也就自己动手撸了个网页版的笔记，存放点BT网站url地址啥的（笑）。

功能还在陆续完善中，后续应该会有手机版的…… ￣▽￣   我的todo，我自己都不信。

<!-- more -->

项目地址：[https://github.com/Ash-sc/notebook](https://github.com/Ash-sc/notebook)

前端：Vue + vuex + axios （不得不说Vue比react上手容易太多）

后端：Node + Express + MongoDB 

项目结构：
``` bash
project
│   README.md
│   package.json
│   .eslintrc
│   ...
└───build  // webpack配置
│
└───server  // Node部分
│   │
│   │   node-app.js // 启动文件 
│   └───controller // 路由控制器
│   └───models // model层
│   └───middlewares // 中间件，负责过滤拦截
│   └───mongo // 数据库配置
│
└───src // source文件夹，主要代码
│   │   index.html // html入口
│   │   app.js // js入口 
│   └───assets // css、font、image等
│   └───components // 公共组件
│   └───filter // 过滤器
│   └───routers // 路由
│   └───services // 数据请求
│   └───store // vuex
│   └───validate // 校验器
│   └───views // 页面模块
│
└───static // 静态资源（moment、bootstrap等）
│
```
## 前端部分：Vue+vuex+axios

**注册/登录页面：**

暂时只做了注册和登录功能。

登录页面采用左右布局，左边暂时还没想好放什么，先用一个css3动画占位好了，右边是固定宽度的表单模块。

 // todo: 邮箱验证
![login-page](http://web-site-files.ashshen.cc/blog/notebook/login_page.png)

**笔记本页面：**

登录之后则跳转到笔记本页面，与Evernote布局一致，分为顶部功能栏和左侧菜单栏，顶部和左侧部分与其他页面公用，所以抽出成组件以便复用。

笔记本页面截图：

![main-page](http://web-site-files.ashshen.cc/blog/notebook/notebook_page_1.png)

新用户第一次进入页面是没有任何默认笔记本存在的，点击左上方的按钮可以新建一个笔记本

![new-notebook](http://web-site-files.ashshen.cc/blog/notebook/notebook_page_newnotebook.png)

笔记本有私有和公开两种分类，暂时来说都是私有的。

// todo: 公开的笔记将可以通过url或二维码分享给他人。

新建笔记本后，笔记本将以条状块形式展示，显示笔记本名称和该笔记本下笔记数量，鼠标悬浮笔记数量出现删除操作按钮，这里选用悬浮是为了不占用右键操作，避免修改网页用户习惯。

// todo: 需要添加笔记本名称修改功能。

![notebook-list](http://web-site-files.ashshen.cc/blog/notebook/notebook_page_list.png)

顶部功能栏设置有排序功能，可根据名称、笔记数量、更新时间 来对笔记本进行排列。

双击笔记本部分跳转到对应笔记本的笔记页面。

**笔记页面**

![note-page](http://web-site-files.ashshen.cc/blog/notebook/note_page_1.png)

同样，初始状态的笔记页面也是啥都没有滴，除了页面公共顶部和左侧之外，右侧的主体部分被我分成了两块：

左边是笔记预览部分，右边是笔记编辑部分。

OK，点击添加按钮添加一个笔记，添加笔记与添加笔记本不同，添加笔记是直接添加了一个标题和内容都为空的笔记。所以不使用弹框的形式，点击按钮后直接生成新笔记。

![edit-note](http://web-site-files.ashshen.cc/blog/notebook/note_page_newnote.png)

然后你会发现左侧菜单会有一些变化，多出来一个“最近笔记”部分；是的，最新更新的五条笔记会显示在左侧菜单的顶部。

然后我们可以直接编辑新建笔记的标题和内容，让界面看起来不那么丑……

编辑完毕，点击保存（不保存也没有关系，下一次刷新页面时会自动同步最新的笔记到服务端）。

![save-note](http://web-site-files.ashshen.cc/blog/notebook/note_page_edit.png)

Emmmm……回头看了一下床上，还好，这次的女朋友质量还是可以的。

同样，笔记缩略图的显示跟笔记本缩略图类似，只是添加了高亮并且右边没有了笔记数量，取而代之的是一个操作提示按钮，鼠标悬浮上去变为删除。

然后是登出功能，与Evernote一样，点击顶部栏的账号名称，会弹出账号操作的悬浮框，然后点击“Log Out”即可退出并清空所有本地缓存。

![user-info](http://web-site-files.ashshen.cc/blog/notebook/account_option.png)

OK，有这些基本功能已经可以确保项目的正常使用了，但是为了方便，再添加一个关键字搜索好了。

搜索的入口：顶部栏右侧，输入关键词并后，会从笔记标题、内容和笔记本名称中搜索出含有改关键词的结果，点击之后跳转到相应的页面。

![search](http://web-site-files.ashshen.cc/blog/notebook/search_function.png)

搜索功能也加好了，前端的工作到这里也就告一段落了。

*前端数据的处理：*

笔记的本地缓存：笔记本和笔记的数据都是缓存在本地的localStorage中的，但是笔记的标题和内容是加密过的。

*数据刷新问题：*

每次刷新页面，都会去后台取数据库中最新的笔记列表，但是只返回笔记的id和更新时间，并且以更新时间降序排列，然后与用户本地数据进行比较，如果二者长度和最新的一条不能匹配，那么则对二者全量进行比较，本地较新的上传到服务端，然后更新列表。

*登录验证：*

服务端接口通过session进行登录验证，session设置过期时间时间为30天。

## 服务端部分：Node.js+MongoDB

待续……

/************** 2017-11-13 ****************/

由于主要是为了熟悉Vue的api，顺便寻找一个存储笔记数据的应用；

想来用户也不会超过一掌之数，所以服务端的实现也就直接使用Node.js了

Node.js的强大之处在于它有丰富的package可以使用，比起前端还要自己撸组件，Node省事了不少。

MongoDB的操作与连接部分，我使用的是mongoose这个包，API使用方式也比较简单易懂。

### 服务端部分代码分层：

1. node-app.js ：  入口文件。

2. router/ ：          路由转发部分，权限控制可以在这边添加。

3. controllers/ ：  请求处理控制器。

4. modal/ ：         数据库modal层，只对外暴露特定接口。

由于是自用，暂时不考虑数据量比较大的情况，所以也就没有对数据库操作进行优化， ……然而实际情况是：ﾉ)ﾟДﾟ(  作为一个前端，并不擅长数据库性能优化。

### 数据加密：

服务端数据加密的方式采用了比较简单的对称加密算法——AES算法，加解密的密钥在用户注册时生成并存放在用户信息中，生成的规则也比较简单：使用md5加密用户的用户名和密码和一个随机值。
