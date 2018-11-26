---
title: React+Node.js在线小说阅读器
date: 2017-07-04 18:03:29
tags: [react, nodejs]
---

一直有追小说的习惯。前段时间连载的小说都追完了，书荒之下就下了一个“书旗小说”来看一些已经完结的小说。好处就是可以不用像追连载小说那样花钱，不过书旗App现在也是各种广告了，所以看得很是蛋疼。通过网页看的话，广告也无法避免，于是就想着可不可以用React和Node.js自己撸一个在线的小说阅读器出来。

[GitHub地址](https://github.com/Ash-sc/online-reader/tree/master)。

## 准备工作

首先需要构思一下，这个阅读器应该怎么实现，用什么步骤来实现。

1. 小说的来源可以从别的网站上获取，用定向爬虫就行。

2. 部署起来一定要简单，不能太过复杂。

3. 页面的话，搜索功能和章节列表功能是必须的。

4. 最好阅读界面的背景色字色可以自己设置，因为使用环境不一定，要是能扫描个二维码就能在手机上看就更好了。

5. …暂时没想到。

OK，思路有了，开始。

服务端接口，粗略估计有以下几个：

1. 搜索接口

2. 获取章节信息接口

3. 获取章节内容接口

前端模块，对应的有以下几个：

1. 搜索模块，包含搜索框以及搜索结果展示

2. 章节信息模块，包含章节列表展示

3. 章节内容展示模块，包含章节内容展示以及一些小工具

## 资源获取

先从资源的获取开始，因为想要代码量尽可能的少，而我自己本人也是对Node.js的熟悉程度超过了Python，所以最终还是决定用Node来作为服务端爬取资源。(使用superagent和cheerio这两个包)

衡量了一下小说中的错字和排版，最终我选择了[http://www.snwx8.com/](http://www.snwx8.com/)这个网站来作为资源获取来源。

先从搜索开始，使用了这个网页的搜索功能搜索了一本小说之后，发现网页跳转到了另外一个名为”search.php”的子页面，搜索的参数也直接挂在了url后面。恩，这样就好办了，直接用url爬”search.php”这个页面就行了，搜索参数的话，用urlencode方法转义一下就行。

（这里在实现的过程遇到了一个坑：superagent本身是只能爬取utf-8编码的页面的，但是这个网站的编码是gbk，所以还需要引入另外一个包superagent-charset）。

获取到了页面的代码，接下来要做的事情就是从页面代码中过滤出我们需要的内容：小说名称、链接、作者、更新时间、最新章节、封面等。so，右键页面 → 审查元素，然后发现搜索结果在id为”newscontent”的div下面的ul的li中，除了封面链接。

![www-image](https://ashshen.cc/wp-content/uploads/2017/07/search-result.png)

使用cheerio获取到ul中所有的搜索结果，并放入数组中，返回给前端页面，代码如下：

``` js
router.get('/searchArticle', (req, res) => {
  request.get(`http://www.snwx8.com/modules/article/search.php?searchkey=${urlencode(req.query.searchKey, 'gbk')}`)
    .charset('gbk')
    .end((error, resp) => {
      if (error) {
        log.error('search error : ', error);
        res.status(200);
        res.send({
          result: 1,
          error,
        });
      } else {
        const $ = cheerio.load(resp.text);
        const item = [];
        $('#newscontent .l ul li').each((idx, element) => {
          const $element = $(element);
          item.push({
            classify: $element.find('.s1').text().replace(/小说|\[|]/g, ''),
            articleName: $element.find('.s2 a').text(),
            articleLink: $element.find('.s2 a').attr('href'),
            latestCharterName: $element.find('.s3 a').text(),
            latestCharterLink: $element.find('.s3 a').attr('href'),
            authorName: $element.find('.s4 a').text(),
            authorLink: $element.find('.s4 a').attr('href'),
            updateTime: $element.find('.s5').text(),
          });
        });
        res.status(200);
        res.send({
          result: 0,
          content: item,
        });
      }
    });
});
```
好了，获取完搜索的结果后，我们的搜索接口服务端部分也就完成了。接下来要做的是：当点击了小说结果之后，获取小说章节列表接口以及获取章节内容接口，实现的步骤与获取搜索结果一致。

[具体代码](https://github.com/Ash-sc/online-reader/blob/master/server/article/article.js)。

## 前端界面

接口写好了，接下来就是前端界面的开发了，相比于使用原生js写界面，使用框架的好处在于：不用花大量的功夫去写dom操作，你只需要在合适的时候去调用接口，然后框架会根据你返回的数据渲染页面。

关于React这里就不介绍了。由于只是一个简单的小说阅读器，并不需要使用太过复杂的项目结构：像`react-router`、`redux`这类的组件其实都是没有必要的。不过我在项目中还是使用了我习惯使用的项目结构，所以会比较复杂，实际上最主要的部分都在`src/js/components/Article`文件夹下，样式文件为`src/assets/sass/pages/_article.scss`。

具体的React代码，在这里就不一一解释了，组件拆分得很细，函数命名也都是根据作用语义化命名的，没有什么好说的。

说一说我在前端开发中遇到的比较有意思的事情吧。

### 关于动画

一个项目中，恰到好处的动画是必不可少的，可以提升场景过渡过程中的用户体验，由于是单页面的应用，功能也很比较单一，所以我在页面中用到的动画模式也比较简单，绝大部分都是平移效果加上宽度变化以及渐显渐隐效果，通过css中的transition实现。

宽度的变化，最初的选择的是直接用width来变化，后来发现：当dom节点比较庞大时，对width做transition会造成界面卡顿，十分影响体验。

原因在于：**节点的width是通过CPU来计算的，变化的dom节点数越多，计算需要的资源越多，也就越容易造成卡顿了，CPU资源是非常宝贵的，尽量不要在动画上使用它**。

解决办法：不使用width变化来实现动画，转而使用transform:translate()。区别在于: **transform并没有改变元素的宽高等计算属性，所以并不会占用CPU资源，而是使用GPU资源来完成**。

背景颜色变化和字体颜色变化属于比较好实现但是非常实用的功能。

页面生成二维码功能的话，我这边选择的是通过`qrcode.react`组件来生成url二维码，具体的手机端页面暂时还没有实现，留一个TODO吧。

项目实现效果图：

![preview-image](http://web-site-files.ashshen.cc/gitHub/online-reader-preview.gif)

