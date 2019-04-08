---
title: 一个简单的vue文件上传组件
date: 2017-11-28 20:02:09
tags: [vue, file-upload]
categories:
  - 前端
  - vue
---

文件上传组件算得上是vue组件中使用比较频繁的了，npmjs上的vue文件上也有很多，但是找到一款适合自己的应用场景的还真不是那么容易。

于是，秉持着自己动手丰衣足食（闲着也是闲着）的理念，花了半天时间撸了一个简单的文件上传组件。

初始版本核心代码包含注释和代码分割也才两百行左右。

<!-- more -->

项目地址：[https://github.com/Ash-sc/vue-simple-upload](https://github.com/Ash-sc/vue-simple-upload)

示例地址：[http://demo.2017017.xyz/vue-demo/#/vue-simple-file-upload](http://demo.2017017.xyz/vue-demo/#/vue-simple-file-upload)

## 需求

既然是一个简单的文件上传组件，那么只要保证必要的功能点都有就行了：

1. 单文件上传 （后续可考虑扩展为多文件，// 2017-11-30 已添加该功能）

2. 支持文件类型限制

3. 支持实时上传进度以及上传速度

4. 支持额外的formData数据

当然，还有像支持手动开始、支持上传取消、拖拽上传等功能。由于当前的使用场景中并没有要求此类功能，所以也就放在后续版本的todoList中了。

## 组件运行逻辑

写组件之前，先整理出组件的运行步骤和需要注意的点，这对之后的编码环节会有很大帮助，大致如下

![flow](http://web-site-files.ashshen.cc/blog/vue-upload-component/upload-component-logic.png)

ok，大致罗列出组件的运行步骤以及注意事项后，开始编写组件的代码。

## 代码框架

组件项目的框架的话，使用webpack-simple就可以生成一个，具体步骤的话，可以自行google。当然，也直接可以从我的Github项目中clone现成的代码，然后修改src/lib下的文件即可。

## 关键点实现

*首先*，对于步骤图中第一步的上传按钮，我直接写在了template中，而不是使用<slot>插槽，对于不需要按钮的用户而言，可以使用css隐藏然后再用js模拟点击事件即可。

对于第一步到第二步的*两个注意点*的处理：

1. 由于input标签的*accept属性*并不能完全限制用户的文件选择行为（用户仍然可以选择限制之外的文件，并且会录入input的值中），我在第二步到第三步之间，XHR上传之前，会针对选择的文件类型重新使用js校验一次，对于不符合的文件类型，不会触发XHR请求。

2. 为了防止由于浏览器的文件选择框具有响应时间而导致的*多个文件选择框*问题，我采用的办法是：在组件的data中设置一个标志位，文件选择完成(change)后，重置标志位，但是这样也会有问题：如果点击了”取消”，那么文件选择input的change事件是不会触发的，所以我在上传按钮上也添加了一个blur事件，当事件触发时，重置标志位。

选择完成文件后，input的change事件触发，然后通过event参数获取到已经选择的文件。

PS：这里还需要注意的一点就是，在选择完文件，并获取到已选择文件之后，需要清空input的value以保证下一次选择相同的文件仍然可以触发change事件（当文件上传失败时，用户可能需要重新选择相同的文件）。

然后我将选择的文件导入到data中定义的文件信息数组fileInfoList中（是的，数组。因为要考虑到之后的多文件扩展）。

fileInfoList中的元素存储的不仅仅是文件信息，还包括一个具有唯一性的id（时间戳加随机数，用来作为标志位，为之后的abort功能做准备），包括一个type（值为”waiting”、”uploading”、”success”、”fail”、”abort”(添加取消上传后新增)中的一个，为手动开始功能做准备），包括一个用来描述上传进度的progress、上传速度描述uploadSpeed，以及文件名属性name。

 
接下来是XHR上传的部分，直接使用js的XMLHttpRequest方法发送http请求。同样，也在组件的data中定义了一个xhrObj用来存放上传的XMLHttpRequest实例对象（为了方便abort功能的实现）。

遍历上一步中定义的fileInfoList，把选择的文件依次上传，并且把id作为key，XMLHttpRequest实例作为value存入xhrObj中。

*上传进度的实现方法：*

首先定义一个uploading方法用于更新fileInfoList元素中的progress属性

然后在XMLHttpRequest的onprogress方法中调用uploading方法，onprogress方法带有一个progress参数，其中就包含loaded，也就是已经上传的文件size，除以文件总size就得到了上传的进度。

*上传速度的实现：*

上传速度的实现稍微要麻烦一点；由于已经上传的文件size是可以获取到的，而在onprogress过程中，每两次触发uploading方法的时间间隔是不一定的，所以需要在data中定义一个对象uploadedSize，用于存放每一个文件在uploading方法触发时已经上传的size以及当前时间戳，于是在下一次触发uploading方法时，用上传的size之差除以时间戳之差得到的就是上传的进度（PS：同时也要更新uploadedSize）。

*数据传出到父组件：*

每当uploading方法触发从而改变fileInfoList时，使用vue组件自带的$emit方法将fileInfoList变动通知父组件。

*上传失败和上传完成：*

文件上传成功以及上传失败，是通过XMLHttpRequest的onreadystatechange方法来判断的，当文件上传成功或者失败后，将该文件上传请求的reponseString存入fileInfoList对应元素的response属性中，并修改其type属性为”success”或”fail”，然后通过$emit方法通知父组件。

## 组件使用方法

文件上传组件总共有两个props：options与progress-update。

options为配置项入口。

progress-update为组件上传进度出口。

具体使用方法见：[README.md](https://github.com/Ash-sc/vue-simple-upload/blob/master/README.md)

*PS： 2017-11-29  // todo: 多文件上传、上传取消、拖拽上传、手动开始。*

## 添加多文件上传：（2017-11-30）

由于之前的代码中已经预留了多文件上传的相关接口（文件信息使用数组存放、组件传出参数为数组），所以添加多文件上传几乎没有花费什么力气。

首先，在options中增加支持属性”multiple”，类型为布尔类型；同时设定pros中默认值为false（防止用户未定义改属性值报错）；然后再将该属性绑定到input上即可。

## 添加上传取消功能：（2017-12-01）

考虑到上传取消功能的扩展，我在初始版本中就已经将XHR实例缓存在了data中，扩展上传取消功能只需要调用对应XHR实例中的abort钩子即可。

关于上传取消功能的使用：

我最初的想法是用户将需要取消的项id(传入”all”则表示所有)作为props传给组件，组件内部感知props变化从而abort请求；不过这样一来很可能出现用户忘记重置props而导致的无法上传。

为了避免这样的bug，最终还是决定由用户通过ref去自行调用组件内部的abort方法。

PS：添加了abort后，fileInfoList中的元素同时也添加了一个type：”abort”用于区分手动取消与上传失败。

## 添加手动开始上传功能：（2017-12-28）

2017年12月28日，在版本v1.2.0中添加了用户可手动选择开始上传功能。当然，为了保持版本的向下兼容以及使用的简易性，该功能默认是关闭的。

使用手动开始上传的功能首先要在config中将”autoStart”设置为false。与上传取消功能一样，同样是通过ref去调用组件内部的startUpload方法，传入需要开始上传的文件id（不传则表示开始上传所有等待上传的文件）。




