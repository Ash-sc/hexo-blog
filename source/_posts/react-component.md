---
title: 封装一个react组件并发布到npmjs.com
date: 2017-09-05 19:48:52
tags: [react, component, npm]
---

如果说2015是属于Angular1.x的年代，那么2017已经是React和Vue两分天下了；虽然Angular4.x版本已经发布，但是无论是从目前的社区活跃还是使用比例来看，比之R&V还是有一定差距的。

而前端的组件化思想也从Angular1.x的Directive到R&V的Component，逐渐发展成熟。

对于组件化的好处（减少代码量、易于维护…）这里就不在赘述了，今天要说的是如何从零开始封装一个react组件并发布到npmjs.com。

## 开始之前

在开始封装组件之前，我们需要确定是用ES5的语法来编写还是ES6的语法来编写，如果是ES6的话，那么我们需要使用babel把代码转换成ES5。

## 组件模板

在我们开始封装组件之前，我们可以新建一个简单的组件模板，确定项目结构以及编译方式并且验证无误之后，再开始编写组件的逻辑部分。

我设定的目录结构如下
``` bash
project/
  |———— example/
  |———— lib/
  |———— src/
  |
  |—— .babelrc
  |—— gulpfile.js
  |—— index.js
  |—— package.json
  |—— webpack.config.js
```
src文件夹中存放源代码。

lib文件夹中存放编译后的代码。

example文件夹中存放示例信息，同时也可以作为开发时调试用。

index.js为组件模块的入口文件。

.babelrc为babel配置文件

gulpfile.js为gulp的配置文件，由于我习惯使用sass来写css，所以在build组件时，会先用gulp将scss文件转化成css文件。

package.json。

webpack.config.js为webpack配置文件，开发时用到。


### 开发步骤：

1. 把ES6代码都放在src文件夹下；

2. 然后使用gulp将sass编译成css；

3. 然后将src下ES6写的js代码都使用babel编译成ES5并输出到lib文件夹下，非js文件直接复制到lib文件夹下；

4. 然后将项目根目录下的index.js导向lib文件夹，一个组件就完成了。

5. 发布组件。

## 样例代码

以下是项目中的部分样例代码：全部代码可查看我的gitHub：[https://github.com/Ash-sc/react-component-temp](https://github.com/Ash-sc/react-component-temp)

.babelrc
``` js
{
    "presets": [
        "es2015",
        "stage-0",
        "react"
    ]
}
```
package.json：（PS： json文件本身是没有注释的）
``` js
{
  "name":..., // npm包名
  "version": xxx, // 版本
  "description": ..., // 描述
  "main": "index.js", // 组件模块入口
  "scripts": {
    "dev": "node webpack.config.js",
    "build": "./node_modules/.bin/rimraf lib && gulp sass && ./node_modules/.bin/babel src --copy-files --source-maps --extensions .es6,.es,.jsx --out-dir lib"
  },
  ...
}
```
在package.json里面，我定义了两条script指令：”dev”和”build”。

npm run dev：用来启动一个Node.js服务，开发时用到

npm run build：用来构建组件。

其中，npm run build又包含了三条指令：

1. ./node_modules/.bin/rimraf lib：清空lib文件夹，在编译构建之前，养成清空lib文件夹的良好习惯可以避免一些不必要问题。

2. gulp sass：编译src/下的scss文件为css。

3. ./node_modules/.bin/babel src –copy-files –source-maps –extensions .es6,.es,.jsx –out-dir lib：将以.es6、.es、.jsx结尾的文件使用babel编译然后输出到lib文件夹下。

index.js:
``` js
module.exports = require('./lib/js/index');
exports.default = require('./lib/js/index');
```

## 编写组件逻辑

组件模板构建完成之后，我们就可以着手开始在src/目录下index.jsx中编写组件的逻辑了，相关的样式写在src/style.scss中。

## 发布至npmjs

组件逻辑编写完成并确认无误之后，使用npm run build编译组件的源代码到lib文件夹中，然后我们就可以使用npm来发布我们的组件了。

使用npm发布组件：

    1. 首先我们要在npmjs.com注册一个账号。

    2. 然后使用npm adduser添加注册的账号。（添加账号完成之后，可以使用npm whoami查看当前使用的用户）

    3. 然后使用npm publish发布组件。

PS：每一次npm发布的组件version(package.json中的version)不能相同，否则会发布失败。

如果想要撤回发布的组件，可以使用”npm unpublish 组件名称”来撤销发布，但是只能撤销发布24小时内发布的组件版本。

 

关于更多npm命令的文档，[这里](https://docs.npmjs.com/)。

下面是我个人发布的react组件：一个日期选择组件和一个日期区间选择组件。

date-time-react：[https://www.npmjs.com/package/date-time-react](https://www.npmjs.com/package/date-time-react)

date-range-for-react：[https://www.npmjs.com/package/date-range-for-react](https://www.npmjs.com/package/date-range-for-react)
