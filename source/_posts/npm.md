---
title: 前端秃头指南之npm
date: 2019-09-09 17:49:20
tags: [node, npm]
categories:
  - node
---

### TL;DR

本文主要介绍：npm install可以安装来自哪些途径的包、一些npm不常用的命令（repo、docs、ci、shrinkwrap等）、package.json中的各种dependency（dependencies、dev、peer、bundle、optional）。

<!-- more -->

### 一、npm命令

npm命令中，我们最常用的就是`npm install/init/start/run`之类。其实除此之外，npm cli还偷偷地支持一些比较有意思的npm命令。

#### 0. npm install

我们可能每天都会遇到这样的情况，当我们`npm install`安装了一个不存在的包时，npm会抛出这样一个404错误：

![npm install error](http://fe-share.ashshen.cc/npm/npm%20install%20error.png)

但是，你有没有注意到过Error info中的notice：

> Note that you can also install from a tarball, folder, http url, or git url.

其实，`npm install`其实不仅仅可以从`npm registry`安装包，还可以通过其他方式安装package。

比如：

##### 0-0. from tarball

我们先用`npm view react`获取到react最新版的tarball地址。

![npm view info](http://fe-share.ashshen.cc/npm/npm%20view%20info.png)

把dist描述下的tarball下载到本地后，我们使用`npm install ./react-16.9.0.tgz`来进行安装。

![npm install with local tarball](http://fe-share.ashshen.cc/npm/npm%20install%20local%20tarball.png)

安装完成后，在package.json中可以看到react值为file协议地址。

![packagejson with local tarball](http://fe-share.ashshen.cc/npm/packagejson-with-local-tarball.png)

`npm install tarball`在不想发包到npm源或者需要离线安装时比较有用，但是同时需要额外的对包进行维护，不然就是版本锁定了。

##### 0-1. from folder

我们也可以从一个本地文件夹安装package，比如：我们在~目录下新建了一个`demo-pkg`的文件夹，并且使用`npm init --yes`初始化它。

然后，我们就可以使用`npm install ~/demo-pkg`来安装刚刚新建的`demo-pkg`啦。

![npm install folder](http://fe-share.ashshen.cc/npm/npm%20install%20folder.png)

安装完成之后在package.json中的表现为一个相对路径的file协议链接。

![package json with file](http://fe-share.ashshen.cc/npm/package-json-with-file.png)

`npm install tarball`可用于在本地调试未发布的npm包。

##### 0-2. from http(s) url

在安装tarball示例中，我们使用`npm view react`获取到了react最新版本的tarball url地址。我们也可以直接用`npm install https://registry.npmjs.org/react/-/react-16.9.0.tgz`安装react。

![npm install tarball](http://fe-share.ashshen.cc/npm/npm%20install%20tarball.png)

react安装成功后，package.json中表现为：

![package.json](http://fe-share.ashshen.cc/npm/packagejson.png)

可以看到react的值指向的是install的url地址。

`npm install url`可用于：不想将包发布到npm，但是又想让团队或其他成员能够安装此pkg。

##### 0-3. from git url

`npm install`还可以从git来安装包。以moment为例，我们可以通过`npm install https://github.com/moment/moment.git`来进行安装。

![npm install git url](http://fe-share.ashshen.cc/npm/npm%20install%20url.png)

此时在package.json中的表现为：

![packagejson with git url](http://fe-share.ashshen.cc/npm/packagejson-with-url.png)

以上几种方式都可以用来安装package，可根据具体使用场景自行选择；当然，在平时正常的业务开发过程中我们还是尽量使用registry来进行package安装。


下面我们来看一些npm支持的一些不常用的命令：

#### 1. npm repo/bugs

当我们遇到了一个关于某个npm的使用问题时，我们的通常做法可能是看一下这个包的github上是否有类似的issue。

于是我们可能会这么做：打开浏览器 → google关键字包名 github → 访问github页面。

其实，这时候我们可以在terminal中直接输入：`npm repo <pkg name>`，这时浏览器就会自动打开该npm包在`package.json`中的设置的github repo页面啦。当然，也可以在terminal中输入`npm bugs <pkg name>`直接跳转到github repo页面issue模块。

怎么样，是不是很方便。

#### 2. npm docs（npm home）

作为一个前端程序猿，如果你嗦你在开发中已经不需要`lodash`、`momentjs`的文档了，我还是比较相信的；但是你如果说你已经不需要`echarts`、`webpack`的文档的话...，请留下您的简历和联系方式。

`npm docs <pkg name>`正是为了这样的场景而存在的。例如：在terminal中输入`npm docs echarts`即可调用浏览器打开echarts的官方文档首页了。

PS：这里需要注意的是：`npm repo`和`npm docs`能够调用浏览器打开页面的前提是该npm包的作者在package.json指定了包的主页（homepage属性）。

#### 3. npm ci

`npm ci`的作用类似于`npm install`，不过它不能携带包名。

`npm ci`一般用于测试平台，持续集成和部署一类的自动化环境，通过跳过某些面向用户的功能，它可以比常规的npm安装快很多，同时，它也会更加的严格，可以捕获由于本地增量安装的引起的错误和不一致并抛出异常。

我们都知道npm5以后，增加了`package-lock.json`文件，而`npm ci`则要求项目必须有`package-lock.json`文件或`npm-shrinkwrap.json`文件。

如果`package-lock.json`中的依赖项与`package.json`不匹配，`npm ci`会退出并显示错误。

已经存在的`node_modules`文件夹会在运行`npm ci`时自动删除。

这样也就保证了使用`npm ci`在各个环境下安装的pkg版本都是一致的。

#### 4. npm shrinkwrap

上面ci命令中提到了`npm-shrinkwrap.json`文件，`npm-shrinkwrap.json`文件实际上也是npm为了依赖包锁定而设计的。
并且`npm shrinkwrap`其实很早以前npm就支持了，只不过我们大多对`package-lock.json`的熟悉多于`npm-shrinkwrap.json`。

`package-lock.json`与`npm-shrinkwrap.json`的区别在于：

1. npm强制`package-lock.json`不会被发布，而`npm-shrinkwrap.json`不受限制。

2. 在存在`package-lock.json`的项目中运行`npm shrinkwrap`时，只会把`package-lock.json`重命名为`npm-shrinkwrap.json`。

3. 两个文件同时存在时，以`npm-shrinkwrap.json`为准。

`package-lock.json`是npm5的新特性，并且不具备向下兼容，而`npm-shrinkwrap.json`在npm4或以下可用。这也就能解释为什么`npm-shrinkwrap.json`优先级会更高了。

npm还支持一些其他的命令，比如：

`npm edit <pkg name>`：可以编辑当前项目node_modules中安装的包，编辑时使用的编辑器通过在npm-config中editor字段配置（默认vi）。在前端普遍使用工程化构建的当下，除非修改验证一些第三方库的bug外，还是极不推荐使用这种方式直接修改项目安装的npm包内容的。

`npm view`：可以查看npm包的信息。比如`npm view moment@2.24.0`可以查看moment2.24.0的信息；`npm view lodash devDependencies`可以查看lodash最新版本的开发依赖。

`npm ls`：查看当前项目`node_modules`中的依赖树。

说了这么多npm命令，我们现在来说说package.json。

### 二、dependency

#### 1. dependencies和devDependencies

当我们的项目只是一个基于工程化构建最后输出静态文件的项目时，package依赖是放在`dependencies`或`devDependencies`中并没有什么区别。

当你的项目是一个前后端一体的项目时（比如server/中存放后端Node脚本，其他文件夹为前端工程），那么你可以把前端工程的项目放在`devDependencies`中，Node服务中用到的服务放在`dependencies`里，这样在部署生产环境时可以使用`npm install --production`只安装Node运行的依赖，减少体积。

当你的项目A是作为一个npm pkg需要被别人安装使用时：位于pkg A的`devDependencies`中的依赖在A被`npm install`时是不会被安装的。比如：moment的`devDependencies`中有grunt，当你在项目中使用`npm install moment`时，node_modules中是不会有grunt的。

#### 2. peerDependencies

`dependencies`与`devDependencies`在开发中经常可见，那么`peerDependencies`又是个什么鬼？

`peerDependencies`的作用类似于：如果你装了我，那么你**最好**也~~装一下B~~装一下pkg B。

当pkg A的package.json中有如下`peerDependencies`时：

```json
{
    "peerDependencies": {
        "B": "1.0.0"
    }
}
```
我们执行`npm install A`后，`node_modules`中的目录结构大致如下：
```json
your-project
├─┬ node_modules
│ ├── A
│ ├── B@1.0.0
```
如果pkg A的`peerDependencies`指定了pkg B版本为`1.0.0`，同时在your-project项目package.json又独立安装了pkg B的`1.1.0`版本，那么执行`npm install A`后`node_modules`会是这样：
```json
your-project
├─┬ node_modules
│ ├── A
│ ├── B@1.1.0
```
而由于A说**最好**安装B@1.0.0但是没有对应版本被安装，所以在terminal中会有一行warning的提示：

![peerDependencies notice](http://fe-share.ashshen.cc/npm/peer-notice.png)

（npm3及以上`peerDependencies`才不会被强制安装；如果是npm2的话，`peerDependencies`是会随着`npm install`被强制安装的。）

#### 3. bundleDependencies

也叫*bundledDependencies*它的值是一个数组。

当我们在test-npm这个测试包的package.json中添加moment和react为`bundledDependencies`然后`npm publish`发布。

![bundleDependencies](http://fe-share.ashshen.cc/npm/bundle-dependencies.png)

然后我们分别下载添加`bundleDependencies`前后的tarball，解压。

![bundle diff](http://fe-share.ashshen.cc/npm/bundle-diff.png)

可以看到添加了`bundleDependencies`的版本（1.0.0）中，会多出一个node_modules文件夹，文件夹中有react和moment以及他们的`dependencies`依赖，而没有添加`bundleDependencies`的版本（1.0.0）文件夹中则只有一个package.json。

在项目中安装有`bundleDependencies`的test-npm版本时，react和moment并不会被展开到项目自身的node_modules中去，而是依然存在于test-npm/node_modules/中。

*PS*： 这里推荐一个轻量的私有npm仓库工具：[verdaccio](https://github.com/verdaccio/verdaccio)，只需要`npm install -g verdaccio && verdaccio`就可以用一键搭建一个私有npm仓库，对于平时的一些开发测试还是比较有用的。

#### 4. optionalDependencies

看到这个名字应该就能猜到`optionalDependencies`表示的是可选的依赖了。

比如：你在项目中依赖了一个pkg C，但是你又希望无论C有没有被安装你的项目都可以正常运行，那么你可以把C放在`optionalDependencies`中。

`optionalDependencies`中的依赖包，即使没有被安装成功（比如不存在对应包名或版本），`npm install`时也不会抛出错误提示。

### 小结

关于npm的知识点还有很多很多，比如scripts、比如npx、比如npm各版本的差异点以及解决的问题和导致的问题，都是很有意思的东西。

不论是按部就班的啃文档还是懵懵懂懂的看issue，又或者逛论坛、看博客、搜google，不管如何，我们都是在秃头的路上一去不返了。










