---
title: HTML5.2发布——你要的模态框来了
date: 2018-01-24 20:23:33
tags: [html5, dialog]
---

2017年12月14日，W3C发布了HTML5.2更新版本。那么，在这个版本中，又有哪些修改与删除呢？一起来看一下。

## 新增对话框 Dialog 元素

没错，HTML终于有原生的模态框元素了。使用dialog元素可以创建宽度默认为其内容的宽度的模态框，模态框默认隐藏。

同时，`dialog`元素支持`show()`、`showModal()`和`close()`三个方法来显示和隐藏。

简单示例如下：

```
# html
<dialog id="test-dialog">
    content here!
</dialog>

# js
document.getElementById('test-dialog').show()
document.getElementById('test-dialog').showModal()
document.getElementById('test-dialog').close()
```

在页面中（chrome浏览器、使用show方法打开）的展示是这样子的：（仅仅是html还是有点丑的）

![dialog-simple](//web-site-files.ashshen.cc/blog/html5.2/dialog-show.png)

打开控制台可以看到dialog的默认样式是这样子的：

![dialog-style](//web-site-files.ashshen.cc/blog/html5.2/dialog-style.png)

dialog元素是通过`display`来实现显示/隐藏的；通过绝对定位实现左右居中的。

OK，我们再来看一下dialog的show、showModal和close三个方法：

show和showModal方法用于模态框的打开，close方法用于模态框的关闭

**在chrome浏览器下**：（filefox浏览器下show和showModal并没有区别）

`showModal`方法会创建一个遮罩层，所以使用showModal打开的dialog是屏幕上下左右都居中的。

`showModal`方法创建的模态框可以通过Esc按键关闭，而show方法创建的模态框只能通过close方法关闭。

`close`方法支持一个String参数（当然你也可以传入一个函数，但是默认调用函数的toString方法变成字符串），并且可以通过returnValue获取到该参数值，使用方法如下：

![dialog-function](//web-site-files.ashshen.cc/blog/html5.2/dialog-close.png)

因为我这边直接是在控制台中测试，所以`$0`代表的就是dialog元素。

可以看到：在`dialog`元素没有使用过`show`方法之前，调用`close`方法传参`returnValue`的值是不会被复写的。

至于`returnValue`的作用，暂时没有发现有什么奇特的用处。

### 给dialog添加样式和动画

dialog终究只是一个HTML元素，在我们的实际业务场景中，还需要用它配合CSS样式使用。

需要给dialog加入动画，可以从dialog的自带的*open属性*入手，从而省去`addClass`和`removeClass`操作。

大致如下：

``` css
.beauty-modal[open] {
 animation: beauty-dialog-show 1s both linear;
}
```
另外，关于使用`openModal`方法打开的模态框背景，可通过伪类`::backdrop`来修改样式。

### 浏览器兼容性

关于dialog浏览器的兼容性，可以从`caniuse`获取。

大致如下：（可以点击show all看到更详细的信息）

![caniuse](//web-site-files.ashshen.cc/blog/html5.2/dialog-broswer.png)

对于firefox浏览器，需要在about:config页面将dom.dialog_element.enabled值修改成true才能使用dialog元素。

同时，上面也提到：**firefox浏览器中，show和showModal方法是没有区别的，都相当于show方法**。

对于不兼容的浏览器，可以使用*polyfill*，有兴趣的可以点击查看。

判断浏览器是否支持dialog元素，可以用：`typeof document.createElement('dialog').show === 'function'`

dialog的demo地址：http://demo.2017017.xyz/html5-2/dialog.html

**PS**：如果不使用CSS给dialog定位处理，在chrome浏览器中使用`showModal`方法打开模态框并关闭后，再使用`show`方法打开模态框，其位置会与直接使用`show`方法打开模态框的位置有差异。

