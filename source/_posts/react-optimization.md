---
title: react项目构建优化笔记
date: 2017-04-06 17:33:43
tags: [react, webpack, 前端]
categories: 
  - react
---

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-19.jpg)

公司的项目架构采用的是react + redux，环境构建则采用的是webpack打包编译的方式。随着项目中模块的增加，bundle.js文件的大小也越来越大，导致网页首屏加载时间过长（3MB的文件需要加载20秒），严重影响了用户体验。于是，优化项目js文件也就成了必须和必要的了。

<!-- more -->

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-19.jpg)

## 工具

工欲善其事，必先利其器。首先，我们需要知道webpack打包后的bundle.js文件里面都包含了哪些模块依赖以及各个模块占用的比重。

这里推荐一个比较好用的图形化工具：[Webpack Chart](http://alexkuz.github.io/webpack-chart/)。

使用”webpack –profile –json > stats.json”在项目根目录下生成stats.json将文件上传到网页中解析就可以得到bundle.js中各模块占用的比重了。

效果图如下:

![webpack-chat-img](http://web-site-files.ashshen.cc/blog/react-online-reader/webpack-chat.png)

“~”部分代表第三方的依赖，”src”部分则代表项目自身编写的模块。

### 第一步，分离第三方模块

从Webpack Chart中可以看出，调用的第三方模块中，占比较大的有echarts、react、moment、zrender等。其中zrender是echarts的依赖。完全可以把这部分从bundle.js中拆分出来。然后用外联js的方式通过 script 标签在index.html中引入。

这里需要注意的是：分离的第三方依赖 script 标签需要写在bundle.js前面。

拆分方法：使用weback配置项中的externals，比如，我这边拆分出了moment.js、lodash、echarts：

``` js
output: {
  ...
},
externals: {
  "echarts": 'echarts',
  "moment": 'moment',
  "lodash": 'lodash',
},
```
拆分之前bundle.js的大小为3.1MB；拆分之后，减小到2.2MB。可以看到拆分后的效果还是比较显著的。

### 第二步，react-router按需加载

第三方模块拆分完成之后，剩下的就是按照路由来拆分chunk了，react-router本身带有按需加载的设置：使用”[getComponent](https://react-guide.github.io/react-router-cn/docs/guides/advanced/DynamicRouting.html)“配合webpack的”require.ensure”。

下面给出我的部分代码：

``` js
...
const App = (location, callback) => {
  require.ensure([], (require) => {
    callback(null, require('../components/_shared/App/App').default);
  }, 'app'); // 这里的"app"为chunk的名称，可自定义
};
...
```
``` html
<Route getComponent={App}>
  ...
</Route>
...
```
然后需要在webpack的配置项中增加：
``` js
output: {
  ...
  filename: 'bundle.js',
  chunkFilename: '[id].[hash:8].chunk.js', // "[id]"代表模块加载的id，"[hash:8]"表示使用8位hash值，还可以使用"[name]"获取到chunk的名称
  ...
},
```
写法上大多与这个类似。

这里比较值得注意的一点是：

**如果component使用的是es5标准（module.exports = …）的话，不需要以上代码中的”default”；如果使用的是es6标准（export default…）的话，需要加上”default”。**

将react-router改成按需加载后，发现bundle.js文件的大小已经从原本的2.2MB减小到了670KB，拆分的效果比之前的分离第三方模块还要显著。

### 第三步，模块中的配置项

OK，拆分了第三方的模块以及路由chunk之后，我们再来看看项目中还有哪些东西是可以拆分掉的。

使用”webpack –display-modules –sort-modules-by size”可以在打包是按照文件大小顺序显示所有的模块体积，把shell拉到最下面，发现项目中居然还有一个大小为231KB的json文件！！查看之后发现是一个省份城市地区的json，在之前的版本开发中直接把这部分的代码放在了前端。于是果断把它丢到后端去，改成通过fetch获取。

建议项目中的一些配置项尽量不要写死在前端，第一个原因是会增加前端js文件的体积；第二个原因则是不利于系统的扩展。

## 总结

到这里，本次对于bundle.js文件体积的优化也就告一段落了。

从结果来看：优化之前的文件体积为3.1MB，首屏加载时间大约为22秒；拆分优化之后的文件体积为510KB，首屏加载时间为5秒。

然后加上gzip压缩后，生产环境下js文件体积为220KB，首屏加载时间为2.2s。

