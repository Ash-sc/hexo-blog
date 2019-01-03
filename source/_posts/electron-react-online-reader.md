---
title: 使用Electron构建React桌面应用
date: 2017-07-05 18:10:40
tags: [electron, react, nodejs]
categories: 
  - electron
  - Node
---

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-5.jpg)

虽然用前端语言写桌面应用已经不是什么稀奇的事情了，现有的工具也有[Electron](https://electron.atom.io/)、[heX](http://hex.youdao.com/zh-cn/index.html)、[NW.js](https://nwjs.io/)等，但是用HTML、CSS和JS来构建一个自己的桌面应用还是挺让人兴奋的。

于是我也用Electron来封装了一下我的在线小说阅读器，[GitHub地址](https://github.com/Ash-sc/online-reader)。

<!-- more -->

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-5.jpg)

## 关于Electron

Electron内置了chromium内核以及Node，这意味着我们可以直接使用在浏览器端运行良好的代码直接构建一个桌面应用。而且，让人开心的是：我们不需要像在浏览器中一样考虑兼容性了，只需要兼容chrome就行。所以，我们可以把electron打包出来的程序看作是一个”小型”的chrome内核浏览器+网页的代码。

像vscode、atom的客户端都是基于Electron开发的。

Electron的入门教程这边就不赘述了，比较简单，[electron传送门](https://electron.atom.io/docs/tutorial/quick-start/)。

## 使用Electron运行React应用

使用Electron来封装React应用的过程，我们可以按照入门教程的步骤一步一步来：

首先，在项目根目录下创建一个”main.js”文件，main.js的内容与教程中的内容一致，可以自行修改BrowserWindow方法中的参数，调节主窗口初始大小。

由于我的React项目代码打包后都在public文件夹中，所以我需要修改createWindow方法中pathname的值为path.join(__dirname, ‘public/index.html’)。

然后我们按照教程修改package.json中的main和script中的start，并使用npm安装electron。

（为什么要配置script的start：配置了script的start之后，我们可以直接用npm start或者npm run XXX来调用electron，否则的话，想要直接在shell中使用electron等命令的话，需要全局安装electron并配合环境变量，否则就只能用./node_modules/.bin/electron，这样太过麻烦）

安装完成后shell中执行”npm start”，发现程序弹框正常，但是页面的中的内容不见了，打开控制台发现：

![dev-error](http://web-site-files.ashshen.cc/blog/react-online-reader/electron-error.png)

由于我在React项目中，对路由进行了分块打包，优化页面加载速度 ( [具体看这里](/2017/04/06/react-optimization/) ) 。而在Electron应用中，会发生分块文件路径引用错误，导致页面不能正常加载。

打开Electron控制台中的Network，发现引用的路径变成了file:///1.app.chunk.js，因此推测是webpack打包时，bundle.js中引用分块的js文件使用的路径在Electron中解析会出现问题。

好在这个路由分块的优化是为了解决由于用户网络差异导致bundle.js过大的情况下页面加载需要很长时间，而使用Electron打包后的js文件都在本地，所以可以不从bundle.js中拆分出chunk。

于是，恢复react-router的component方法为直接import，然后重新打包，再次执行npm start。

![dev-success](http://web-site-files.ashshen.cc/blog/react-online-reader/dev-success.png)

现在界面已经可以正常显示了，但是这仅仅是静态页面。

显然，我们的目标不只是静态页面的展示，我们需要与服务器进行数据交互。试着在搜索框中输入一些搜索关键字然后搜索：

![search-error](http://web-site-files.ashshen.cc/blog/react-online-reader/electron-request-error.png)

从控制台中，我发现：还是之前js文件那个问题，相对路径的资源和请求在Electron下被引用成了”file://”，怎么解决呢？既然相对路径解析会出错，那么我们换成绝对路径呢？于是我试着将”/src/js/utils/api/api.js”中将后端请求的地址改成了绝对路径”http://localhost:4396″，然后重新编译、运行：

![search-success](http://web-site-files.ashshen.cc/blog/react-online-reader/electron-success.png)

发现请求是成功的，也没有发生在浏览器中会发生的跨域行为，看来是Electron本身对此做了处理。

OK，到这里，一个桌面React应用已经成功运行起来了，接下来要做的就是将React应用封装成可以直接在操作系统上运行的文件。

## 打包可运行的Electron应用

打包的话，我这边使用的是[electron-packager](https://github.com/electron-userland/electron-packager)这个npm包，安装了之后，只需要在shell中敲一行命令就行了。同样，为了避免每次都敲太过复杂的命令或者全局安装electron-packager，我们把electron-packager的命令配置到package.json的script中去。

``` js
"scripts": {
  "start": "electron .",
  "build": "webpack --config ./.webpack/config/production.js",
  "dev": "webpack ./webpack-dev-server.js",
  "pack": "electron-packager ./public ashshen --platfrom=all",
  "pack-mac": "electron-packager ./public ashshen --platfrom=darwin",
  "pack-win32": "electron-packager ./public ashshen --platfrom=win32",
  "pack-linux": "electron-packager ./public ashshen --platfrom=linux"
},
```

然后我们只需要通过：

``` bash
npm install
npm run build
npm run pack-mac
```

就可以生成一个可在mac上运行的软件了。其他系统的话，只需要改变第三条命令即可。

PS：（2017-12-13）如果打包后的安装包出现无法打开的情况，可以使用[electron-packager-interactive](https://github.com/Urucas/electron-packager-interactive)手动打包，打包资源路径指向public文件夹即可。
