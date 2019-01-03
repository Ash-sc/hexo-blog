---
title: 图片马赛克小工具
date: 2017-07-10 19:05:13
tags: [马赛克, js]
categories: 前端
---

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-13.jpg)

前段时间聊微信的时候，看到微信群里有个童鞋的头像变成了自己的自拍并且打上了马赛克（(╯-_-)╯~╩╩  绝对不是我撸多了视力下降了）。轻度模糊的整体马赛克还挺有意思的，也具有一定的辨识度，于是我就想着：马赛克这玩意儿，前端也可以做的嘛（canvas），所以就试着写了一个图片马赛克小工具。

由于只是简单的使用canvas给图片加上马赛克，所以并不需要后端的服务，整个流程可以在前端完成。

<!-- more -->

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-13.jpg)

demo地址：[http://web-site-files.ashshen.cc/blog/pages/index.html](http://web-site-files.ashshen.cc/blog/pages/index.html)

小工具效果展示：

<img src="http://web-site-files.ashshen.cc/blog/origin-image.jpg" style="width:49%;display: inline-block;" /><img src="http://web-site-files.ashshen.cc/blog/download.png" style="width:49%;display: inline-block;" />

实现步骤：

1. 上传图片

2. 图片转化成添加了马赛克的canvas

3. 从canvas提取出图片的dataurl

4. 展示添加马赛克后的图片并下载

去网上搜了一下前端给图片添加马赛克的方法，然后搜到了[close-pixelate](https://github.com/desandro/close-pixelate)这个插件，一个可以把图片转换成添加了马赛克的canvas的插件。

## 实现过程

由于只是一个简单的小工具，我也就没有使用想jQuery的库来实现了，全部使用原生的js，也好锻炼一下原生js。

*首先*，是上传图片的过程，使用input type=file，然后把input使用display:none隐藏，并使用一个botton的点击事件来模拟input的点击事件触发文件选择。

PS：这里在实现的时候遇到了一个问题：我的想法是限制使用者选择文件的类型，于是使用input的accept属性做了如下限制：“image/jpg,image/jpeg,,image/png,image/gif”；写法麻烦不说，还会导致由于浏览器每次在弹出文件选择框之前都要去判断文件夹下文件类型是不是accpet的文件类型，所以容易造成弹框的卡顿。

google了之后，解决办法为：修改accept为“image/*”，这样不仅写法简单，还可以让支持更多图片格式选择，文件选择的弹框也不会卡顿了。

*然后*，文件上传完毕之后就是一些错误判断了，比如：使用者没有选择文件，选择了其他类型的文件。是的，即使设置了accept，使用者也是可以点击文件选择下面的选项去选择其他文件的，只是选择完之后，相当于没有选择。不过这一部分的错误提示是不能少的。

错误提示之后，就是图片文件的提取了，这里需要用到FileReader()，将选择的图片转换成DataURL，并使用FileReader自带的API：onload处理提取到DataURL之后的相关逻辑（创建img节点，并赋值src属性，然后使用appendChild将节点添加到dom指定位置，然后使用close-pixelate将img转化成canvas）。

``` js
const reader = new FileReader();
reader.readAsDataURL(files[0]);
reader.onload = function() {
    ...
}
```

*第三步*，上传文件 → 提取图片DataURL → 添加img到dom中 → 转化成带马赛克的canvas，这几步都是在选择完文件之后需要顺序执行的，所以我全部把它写在了一个函数中，需要在使用者上传了图片之后调用。

另外，还需要添加一个输入框让使用者自行选择马赛克颗粒的大小，以及一个文件导出格式的单选按钮，并监听对应的触发事件。

*第四步*，处理完这些之后，剩下的就是图片导出下载的功能了。

  一，从canvas中提取出DataURL，这个可以直接用canvas自带的API中的toDataURL()，参数为图片的格式，eg:”image/png”。

  二，将获取到的DataURL赋值给页面中已经准备好的a元素（a元素设置download属性可以用于下载），然后使用js模拟a元素的点击事件实现下载功能。

PS：由于使用者上传的文件大小是不确定的，但是页面中展示图片效果的区域是固定大小的，而close-pixelate插件也是按照图片大小生成1：1比例的canvas的，所以在图片上传完成之后，我需要先获取图片的大小，并与展示区域的大小进行比较，并计算缩放的比例，然后使用css中的scale()对canvas进行缩放和平移。

图片马赛克小工具完成，gitHub地址：[https://github.com/Ash-sc/mosaic-image](https://github.com/Ash-sc/mosaic-image)。
