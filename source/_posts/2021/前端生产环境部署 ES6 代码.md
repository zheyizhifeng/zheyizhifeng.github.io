---
title: 前端生产环境部署 ES 6 代码
date: 2021-10-14 15:34:38
categories: 
  - Programming
  - JavaScript
  - ES 6
tags:
  - Programming
  - JavaScript
  - ES 6
---

## 背景

我们常说的 ES6，也就是 2015 年 6 月正式发布的 ECMAScript 2015。它是 ES 规范的第六个版本，发布至今已有两年多。在这个快节奏的时代，两年时间感觉是很长，可以发生很多事情了。比如，ES6 发布时你最多买到 iPhone 6，而现在不少人都用上 iPhone X 了。

我们不妨把时间再往前拨一些。ECMAScript 3（ES3）发布于 1999 年 12 月，ECMAScript 5（ES5）发布于 2009 年 12 月。这样一看，就发现两年其实还是比较短的。有人可能会问，这么快把 ES6 推向生产环境，靠谱吗？别急，我再说一个，你恐怕就会觉得我疯了。2016 年 4 月，淘宝和天猫正式停止对 IE6 浏览器的支持。IE6 可是 2002 年面世的，它甚至完全不支持 ES5！这么看来，生产环境部署 ES6 应该是一次非常冒险的尝试。那到底要多大的诱惑才能让人甘愿跳坑？

这个问题先按下不表。
<!-- more -->
前端圈这些年涌现出很多工具。我 2011 年接触的 NodeJS，2012 年知道的 Grunt，到现在日常使用的 Webpack、Rollup、Babel、ESLint，其间还经历过 Gulp、Browserify，最近又新出了 Parcel。框架类库这边，从 ExtJS、jQuery 到 Backbone、AngularJS，再到现在的 Vue、React、Angular，更新的速度之快也是令人目不暇接。有人对此很悲观，“前端好难，要学这么多。”也有人调侃，“前端好惨，要学这么多。”我倒觉得这很正常，这是前端圈的「工业革命」。正是这些工具的前仆后继，才使得我们在开发中可以轻松地使用 ES6，甚至 ES7、ES8 的语法和特性，部署到生产环境时再转译成 ES5。

## 现状

大部分前端开发人员热衷于使用新的 JavaScript 语言特性来书写 JS 代码，例如 async 、 await 、 classes 、 arrow functions 等。然而，尽管目前所有的前沿浏览器都能运行 ES2015+ 代码（ES2015及俗称的ES6），但是为了兼容占有小比例的低版本浏览器用户，大部分的开发者仍然使用 polyfills 将代码编译成 ES5 语法。

当下标准的做法是：写 ES6 代码 → 将所有代码编译成 ES5 的（比如通过 Babel）→ 再将编译后的代码加载到浏览器执行。

这可能已经不再是**最有效率**的方式了。因为用这种方式，我们强制最新的浏览器运行旧代码，实际上它们完全可以运行最新的代码。它们支持 ES6，我们难道不能直接给它们 ES6 代码吗？

**[ECMAScript 6 compatibility table]((https://kangax.github.io/compat-table/es6/))**

![image-20210419235736321](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210419235736321.png)

这个页面从 2010 年开始一直在更新，可以说完整地记录了现代浏览器对 ES6 特性支持的历史进程。中间最大这块绿色是桌面浏览器，最右边的是移动浏览器，中间有两列是 NodeJS 环境。是不是绿得很整齐？如果我们的用户恰好都在绿色这一块，代码是不是只需要转译到 ES6 就够了？

当然可以。但可以做不等于必须做，我们还需要更充分的理由。为了寻找这样的理由，谷歌工程师 Philip Walton 就 [用自己的博客做了实验](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/)。他在构建自己的博客程序时，把 JS 代码分别转译为 ES5 和 ES6，结果很有意思。ES6 版本的文件尺寸和执行时间都只有 ES5 版本的一半不到！

## ES5/6 构建文件尺寸和执行时间对比

| 版本     | 文件尺寸  | 执行时间 |                     |           |
| -------- | --------- | -------- | ------------------- | --------- |
| 编译压缩 | Gzip 压缩 | 测量值   | 平均值              |           |
| ES2015+  | 80K       | **21K**  | 184ms、164ms、166ms | **172ms** |
| ES5      | 175K      | **43K**  | 389ms、351ms、360ms | **367ms** |

![image-20210420224919914](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420224919914.png)

![image-20210420224928669](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420224928669.png)

这里面虽然有些背景交代得不清楚，可能存在夸大的成分，但这样的数字确实很吸引人。对于互联网行业来说，带宽和时间最终都是可以换算成钱的！

## 收益在哪

程序员在进行技术决策的时候，通常的目的是为了~~kpi~~，~~升值加薪~~，~~这个技术很流行~~，不，当然是为了对业务产生价值。

不难发现，直接部署es 6代码可以带来的好处包括：

- 代码体积减小
- 性能提升
- 缩短构建时间

### 代码体积减小

首先，es6带来了新的语法和特性，这可以让代码更简洁，使得代码量减少。其次，babel转译之后，会产生一些`helper`，引入一些`polyfill`，而且有一些语法特性无法在运行时进行处理，必须在转译过程中修改源代码，这也会导致代码量增加。当然，部署es6代码也需要babel转译一部分更新的语法，并不是说完全不使用babel。

Google 工程师 es6代码的bundle size比es5代码减少了**50%**，而我自己的实践的结果没有这么夸张，是**25%左右**。

| version | bundle size | Gzipped |
| :-----: | :---------: | :-----: |
|   es5   |    221kb    |  39kb   |
|   es6   |    170kb    |  29kb   |

之所以我和他的测试结果差异较大，是因为最终的结果跟每个人的代码风格和代码内容有很大的关系。我用来进行测试的代码是一个线上项目的代码，我个人的代码风格是大量使用es6/7/8语法，不过项目本身大量是在写业务逻辑，并且项目使用了Vue，其实很多js代码是模板编译之后生成的渲染函数。

所以，大概的结论是es6的代码会比es5的代码体积减少**15% ~ 50%**，具体能减少多少，取决于代码风格以及代码内容。代码风格越现代，代码中"生成的代码"越少，收益越高。

### 性能提升

es6代码相对于es5代码，性能更高的原因我认为有2个

- 代码量减少带来的解析时间减少，这个正比于代码体积的减少
- 运行性能的提高

对于第一点，浏览器解析代码时间的减少应该占比较小，通过chrome的`Performance`面板可以看到，解析一个200kb的js文件Compile Script耗时为**10ms**。

对于第二点，运行时性能。在es6刚刚发布的时候，很多新特性的性能是比较低的，因为js引擎没有足够的时间去进行优化，相比较而言，es5以及更老的代码经过了浏览器的长时间优化，在es6刚出来的时候，很对人也对es6的性能有很大的担忧。

Github上有一个[仓库](http://incaseofstairs.com/six-speed/)，专门对es6和es5进行了性能对比。

![image-20210421122126368](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210421122126368.png)

从对比结果可以看出来，在chrome 72版本上，es6代码的性能优于babel转译成的es5代码，和手写的es5代码比起来各有千秋，基本算是55开。

针对运行时的es6代码和es5代码的性能对比，我也用一个线上项目进行了实验。实验方式如下

1. 利用`performance API`在打包后的app.js文件开头记下时间戳，在js文件末尾减去开始的时间，得到一个时间，众所周知，打包之后的app.js是一个立即执行函数，所以这个时间包含了app.js文件中**部分代码**的运行时间
2. 利用chrome 的`Performance`面板，可以直接得到一段js的执行时间`Evaluate Script`

实验结果如下（20次的平均值）

| version | 记录的运行时间 | Performance面板的Evaluate Script |
| :-----: | :------------: | :------------------------------: |
|   es5   |    258.88ms    |             295.29ms             |
|   es6   |    206.28ms    |             241.59ms             |

测试条件下，es6的代码取得了**20%左右**的运行性能提升

## Android Webview 对 ES 6 支持情况

[检查 ES 6 兼容性](http://ruanyf.github.io/es-checker/)

最新的Mac Chrome

![image-20210421104026586](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210421104026586.png)

![image-20210420232910160](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420232910160.png)

> 注释：[什么是尾调用优化](https://www.ruanyifeng.com/blog/2015/04/tail-call.html)
>
> 

- Android 版本对ES 6 支持情况

**本地设备列表**【Genymotion 模拟】

![image-20210420231211354](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420231211354.png)

- Android 4.4

![image-20210420231614343](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420231614343.png)

- Android 5.0

![image-20210420231821967](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420231821967.png)

- Android 5.1

![image-20210420231748619](/Users/chenlei/Library/Application Support/typora-user-images/image-20210420231748619.png)



![image-20210420232729704](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420232729704.png)

- Android 6.0

![image-20210420232053842](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420232053842.png)

![image-20210420232600122](/Users/chenlei/Library/Application Support/typora-user-images/image-20210420232600122.png)

- Android 7.0

![image-20210420232020159](/Users/chenlei/Library/Application Support/typora-user-images/image-20210420232020159.png)

- Android 7.1

![image-20210420232253070](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420232253070.png)

![image-20210420233722214](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210420233722214.png)



- 修改前：

![image-20210421000418656](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210421000418656.png)

- 修改后：

![image-20210421000447491](/Users/chenlei/Library/Application Support/typora-user-images/image-20210421000447491.png)


话说回来，构建成 ES5 和 ES6，到底差别在哪里？执行变快还容易理解，为什么代码量也可以少这么多？下面的图是通过 Webpack 构建分析工具得到的，ES5 版本代码比 ES6 主要多了两大块。一个是 **core-js**，包含了很多 ES6 特性的 polyfill，这些代码在 Gzip 压缩以后仍然超过 10K。

![image-20210421000552744](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210421000552744.png)

另一个是 **regenerator-runtime**，这是 ES 6 生成器转译为 ES 5 所产生的，单单这一个东西，Gzip 压缩后就有 2.37K 之多，是 ES 6 转译为 ES 5 当之无愧的第一大 polyfill。开发的时候用 ES 8 特性 `async`/`await`，构建时 Babel 就会把它们先转译为 ES 6 的生成器，再转译为 ES5。

![image-20210421000619834](/Users/chenlei/Library/Application Support/typora-user-images/image-20210421000619834.png)

除此之外，还有一些语法特性是无法在运行时进行处理的，必须在转译过程中修改源代码。这些修改也会导致代码量增加。

![image-20210421000812430](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210421000812430.png)



## 降级方案

![image-20210421133454968](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210421133454968.png)

```html
<script type="module" src="es6.js"></script>  
  
<script nomodule src="es5.js"></script>
```

![image-20210421111842163](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210421111842163.png)

![image-20210421000922146](/Users/chenlei/Library/Application Support/typora-user-images/image-20210421000922146.png)

### 实现方式

如果已经使用了 webpack 或者 rollup 这类模块打包工具来生成 JS 文件，那么应该继续保持。

除了当前的代码包，还需要生成类似于第一份的另外一份代码包。（该代码包使用了 ES2015+ 语法），唯一的不同是你不需要将其编译成 ES5 语法的代码，并且不需要引入 polyfills 插件。

例如，假设你使用了 webpack 并且 JS 的入口文件是` ./path/to/main.js` ，你当前的 ES5 版本的配置应该如下所示

```js
module.exports = {
  entry: {
    'main-legacy': './path/to/main.js',
  },
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname, 'public'),
  },
  module: {
    rules: [{
      test: /\.js$/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: [
            ['env', {
              modules: false,
              useBuiltIns: true,
              targets: {
                browsers: [
                  '> 1%',
                  'last 2 versions',
                  'Firefox ESR',
                ],
              },
            }],
          ],
        },
      },
    }],
  },
};
```



```js
module.exports = {
  entry: {
    'main': './path/to/main.js',
  },
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname, 'public'),
  },
  module: {
    rules: [{
      test: /\.js$/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: [
            ['env', {
              modules: false,
              useBuiltIns: true,
              targets: {
                browsers: [
                  'Chrome >= 60',
                  'Safari >= 10.1',
                  'iOS >= 10.3',
                  'Firefox >= 54',
                  'Edge >= 15',
                ],
              },
            }],
          ],
        },
      },
    }],
  },
};
```



一旦运行，这两个配置文件就会输出两个 JS 文件：

- main.js （该文件支持 ES2015+ 语法）
- main-legacy.js （该文件支持 ES5 语法）



```html
<!-- Browsers with ES module support load this file. -->
<script type="module" src="main.js"></script>
 
<!-- Older browsers load this file (and module-supporting -->
<!-- browsers know *not* to load this file). -->
<script nomodule src="main-legacy.js"></script>
```

![image-20210421000934553](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210421000934553.png)



## Polyfill

JavaScript 语言在稳步发展。也会定期出现一些对语言的新提议，它们会被分析讨论，如果认为有价值，就会被加入到 https://tc39.github.io/ecma262/ 的列表中，然后被加到 [规范](http://www.ecma-international.org/publications/standards/Ecma-262.htm) 中。

JavaScript 引擎背后的团队关于首先要实现什么有着他们自己想法。他们可能会决定执行草案中的建议，并推迟已经在规范中的内容，因为它们不太有趣或者难以实现。

因此，一个 JavaScript 引擎只能实现标准中的一部分是很常见的情况。

查看语言特性的当前支持状态的一个很好的页面是 https://kangax.github.io/compat-table/es6/（它很大，我们现在还有很多东西要学）。

`Polyfill`或者`Polyfiller`，是英国Web开发者 [Remy Sharp](https://remysharp.com/) 在咖啡店蹲坑的时候拍脑袋造出来的。当时他想用一个词来形容"用JavaScript（或者Flash之类的什么鬼）来实现一些浏览器不支持的原生API"。`Shim`这个已经有的词汇第一时间出现在他的脑海里。但是他回头想了一下`Shim`一般有自己的API，而不是单纯实现原生不支持的API。苦思冥想一直想不到合适的单词，于是他一怒之下造了一个单词`Polyfill`。除了他自己用这个词以外，他还给其他开发者用。随着他在各种Web会议演讲和他写的书《Introducing HTML5》中频繁提到这个词，大家用了都觉得很好，就一起来用。

`Polyfill`的准确意思为：**用于实现浏览器并不支持的原生API的代码。**

如，`querySelectorAll`是很多现代浏览器都支持的原生Web API，但是有些古老的浏览器并不支持，那么假设有人写了库，只要用了这个库， 你就可以在古老的浏览器里面使用`document.querySelectorAll`，使用方法跟现代浏览器原生API无异。那么这个库就可以称为`Polyfill`或者`Polyfiller`。

好，那么问题就来了。jQuery是不是一个`Polyfill` ?答案是No。因为它并不是实现一些标准的原生API，而是封装了自己API。一个`Polyfill`是抹平新老浏览器 ***标准原生API*** 之间的差距的一种封装，而不是实现自己的API。

已有的一些`Polyfill`，如 [Polymer](https://www.polymer-project.org/0.5/docs/start/platform.html) 是让旧的浏览器也能用上 HTML5 Web Component 的一个`Polyfill`。[FlashCanvas](http://flashcanvas.net/)是用Flash实现的可以让不支持Canvas API的浏览器也能用上Canvas的`Polyfill`。

这里有一堆`Polyfills`，有兴趣可以把玩一下：[HTML5 Cross Browser Polyfills](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-browser-Polyfills)

## 过去

### shim + sham

如果你是一个 3 年陈 + 的前端，应该会有听说过 shim、sham、[es5-shim](https://github.com/es-shims/es5-shim) 和 [es6-shim](https://github.com/es-shims/es6-shim) 等等现在看起来很古老的补丁方式。

那么，shim 和 sham 是啥？又有什么区别？

- shim 是**能用**的补丁
- sham 顾名思义，是假的意思，所以 sham 是一些**假的方法**，只能使用保证不出错，但不能用。至于为啥会有 sham，因为有些方法的低端浏览器里根本实现不了

### babel-polyfill.js

在 shim 和 sham 之后，还有一种补丁方式是引入包含所有语言层补丁的 `babel-polyfill.js`。比如：

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-polyfill/7.2.5/polyfill.js"></script>
```

然后就 es6、es7 特性随便写了。

但缺点是，babel-polyfill 包含所有补丁，不管浏览器是否支持，也不管你的项目是否有用到，都全量引了，所以如果你的用户全都不差流量和带宽（比如内部应用），尽可以用这种方式。

## 现在

现在还没有银弹，各种方案百花齐放。

### @babel/preset-env + useBuiltins: entry + targets

babel-polyfill 包含所有补丁，那我只需要支持某些浏览器的某些版本，是否有办法只包含这些浏览器的补丁？这就是 `@babel/preset-env` + `useBuiltins: entry` + `targets` 配置的方案。

我们先在入口文件里引入 `@babel/polyfill`，

```js
import '@babel/polyfill';
```

然后配置 .babelrc，添加 preset `@babel/preset-env`，并设置 `useBuiltIns` 和 `targets`，

```json
{
  "presets": [
    ["@babel/env", {
      useBuiltIns: 'entry',
      targets: { chrome: 60 }
    }]
  ]
}
```

`useBuiltIns: entry` 的含义是找到入口文件里引入的 `@babel/polyfill`，并替换为 targets 浏览器/环境需要的补丁列表。

替换后的内容，比如：

```
import "core-js/modules/es7.string.pad-start";
import "core-js/modules/es7.string.pad-end";
...
```

这样就只会引入 chrome@62 及以上所需要的补丁，什么 Promise 之类的都不会再打包引入。

###  Ployfill.io 方案

动态 Polyfill 是根据不同浏览器的特性，载入需要的特性补丁。Polyfill.io 通过尝试使用 polyfill 重新创建缺少的功能，可以轻松地支持不同的浏览器，并且可以大幅度地减少构建体积。

[Financial Times](http://www.ft.com/) 在开发和维护这个项目，所以我们能确信这个项目可以持续更新下去.

***有一点需要明白：Polyfill.io 没有提供语法糖支持。比如 类、增强的对象字面量，以及箭头函数之类的特性。对那些代码，你仍然需要进行编译。***

动态 Polyfill 方案对比：

| 方案                           | 优点                                       | 缺点                                                         | 是否采用 |
| ------------------------------ | ------------------------------------------ | ------------------------------------------------------------ | -------- |
| babel-polyfill                 | React16 官方推荐                           | 1. 包体积 200K+，难以单独抽离 Map、Set 2. 项目里 React 是单独引用的 CDN，如果要用它，需要单独构建一份放在 React 前加载 | ❌        |
| babel-plugin-transform-runtime | 能只 polyfill 用到的类或方法，相对体积较小 | 不能 polyfill 原型上的方法，不适用于业务项目的复杂开发环境   | ❌        |
| 自己写 Map、Set 的 Polyfill    | 定制化高，体积小                           | 1. 重复造轮子，容易在日后年久失修成为坑 2. 即使体积小，依然所有用户都要加载 | ❌        |
| polyfill-service               | 只给用户返回需要的 polyfill，社区维护      | 部分国内奇葩浏览器 UA 可能无法识别（但可以降级返回所需全部 Polyfill） | ✅        |

### Polyfill Service原理

- 每次打开页面，浏览器都会向Polyfill Service发送请求，Polyfill Service识别 User Agent，下发不同的 Polyfill，做到按需加载Polyfill的效果。

  ![Polyfill Service原理](https://serverless-action.com/assets/img/28.e7e06200.png)

## 使用方法

直接引入代码即可使用默认配置的 Polyfill：

```html
<script crossOrigin="anonymous" src="https://polyfill.io/v3/polyfill.min.js"></script>
```

Polyfill.io 通过分析请求头信息中的 UserAgent 实现自动加载浏览器所需的 polyfill。

![image-20210421143435576](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20210421143435576.png)

## 高级用法

Polyfill.io 有一份默认捆绑列表，包括了最常见的 HTML5 中的 `document.querySelector`、`Element.classList`、ES5、ES6、ES7 中的 `Promise`、`fetch`、`Array.from` 等等。

你可以通过传递 `features` 参数来自定义功能列表：

```html
<!-- 加载 Promise&fetch -->
<script src="https://cdn.polyfill.io/v3/polyfill.min.js?features=Promise,fetch"></script>

<!-- 加载所有 ES5&ES6 新特性 -->
<script src="https://cdn.polyfill.io/v3/polyfill.min.js?features=es5,es6,es7"></script>
```

Polyfill.io 还提供了其他 API，具体请查阅官方文档：

```html
<!-- 异步加载 -->
<script src="https://cdn.polyfill.io/v3/polyfill.min.js?callback=main" async defer></script>
<!-- 无视 UA，始终加载 -->
<script src="https://cdn.polyfill.io/v3/polyfill.js?features=modernizr:es5array|always"></script>
```

## 未来

关于补丁方案的未来，我觉得**按需特性探测 + 在线补丁**才是终极方案。

按需特性探测保证特性的最小集；在线补丁做按需下载。

按需特性探测可以用 `@babel/preset-env` 配上 `targets` 以及试验阶段的 `useBuiltIns: usage`，保障特性集的最小化。之所以说是未来，因为 JavaScript 的动态性，语法探测不太可能探测出所有特性，但上了 TypeScript 之后可能会好一些。另外，要注意一个前提是 node_modules 也需要走 babel 编译，不然 node_modules 下用到的特性会探测不出来。

## 结语

>  **时间带走一切，es5也不例外。**
>
>  **编写 ES6 只是程序员的胜利**
>
>  **部署 ES6 才是与用户的双赢**
