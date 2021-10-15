---
title: Chrome 控制台实用指南
date: 2021-10-15 11:22:34
categories: 
  - Tools
  - 浏览器
  - Chrome
tags: 
  - Tools
  - JavaScript
  - 浏览器
  - Chrome
  - 控制台
---

转载自掘金，原文链接 [点这里查看](https://juejin.cn/post/6844903443887308814)

---

## 前言

> Chrome浏览器我想是每一个前端er必用工具之一吧，一部分原因是它速度快，体积不大，支持的新特性也比其它浏览器多，还有一部分我想就是因为它的控制台功能强大了吧，说它是神器一点也不过分，很方便。但其实很多开发者并没有用出控制台的精髓，只是使用简单的console.log();其实控制台功能远远不止这么简单哦。

<!-- more -->

## console.clear
> console.clear();清空控制台，这个应该和console.log知名度一样高吧。

## console.log家族
> 先简单介绍一下chrome的控制台，打开chrome浏览器，按f12就可以轻松的打开控制台

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/16573bcd3e15fead9ffde70609380b06~tplv-t2oaga2asx-watermark.awebp)

如果你是一位开发者，我想console.log肯定是经常使用的了，我们主要看看console.log的几个兄弟：

```js
1.console.log ('普通信息')

2.console.info ('提示性信息')

3.console.error ('错误信息')

4.console.warn ('警示信息')
```

![控制台](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/82605c97399bbee425524c971d4685dc~tplv-t2oaga2asx-watermark.awebp)

**大家都会用log，但很少有人能够很好地利用console.error , console.warn 等将输出到控制台的信息进行分类整理。他们功能区别不大，意义在于将输出到控制台的信息进行归类，或者说让它们更语义化。**

> 如果再配合console.group 与console.groupEnd，可以将这种分类管理的思想发挥到极致。这适合于在开发一个规模很大模块很多很复杂的Web APP时，将各自的log信息分组到以各自命名空间为名称的组里面。

```js
console.group("app.bundle");
console.warn("来自bundle模块的警告信息1");console.warn("来自bundle模块的警告信息2");
console.groupEnd();

console.group("app.bundle");
console.log("来自bundle模块的信息1");console.log("来自bundle模块的信息2");
console.groupEnd();
```

![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/26e2ee45384a2fd5b34e08d61a3e27a0~tplv-t2oaga2asx-watermark.awebp)

**这样的控制台信息看上去就一目了然了，就不用再为了找这是属于那一行代码输出的再翻一遍源码了。**

> 另外，console.log家族还给我们提供了一个的API：第一个参数可以带一些格式化指令，比如%c,\n;看下面这个炫酷的效果：

```js
console.log('%chello world', 'background-image:-webkit-gradient( linear, left top, right top, color-stop(0, #f22), color-stop(0.15, #f2f), color-stop(0.3, #22f), color-stop(0.45, #2ff), color-stop(0.6, #2f2),color-stop(0.75, #2f2), color-stop(0.9, #ff2), color-stop(1, #f22) );color:transparent;-webkit-background-clip: text;font-size:5em;');
```

![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/e1635979936eaf5a2b8ff249e8921c95~tplv-t2oaga2asx-watermark.awebp)

**当然，图片也是可以的，读者可以自行尝试，修改上述代码即可。**

> 另外，console.log() 接收不定参数，参数间用逗号分隔，最终会输出会将它们以空白字符连接。
>
> ![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/e08f61646a8e55f4b1781894fdd226c6~tplv-t2oaga2asx-watermark.awebp)

## console.table

> 看着这种“黑魔法”是不是有种坑分的感觉呢，其实还不止哦！console.table可以让我们输出表格,示例：

```js
var data = {code:200,content:[{'品名': '杜雷斯', '数量': 4}, {'品名': '冈本', '数量': 3}]};
console.table(data.content);
```

![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/0ab99d1594fe63284c97bee06cc75a5e~tplv-t2oaga2asx-watermark.awebp)

**有的时候后端传回来一大串数据，是不是觉得直接console.log或是通过抓包工具查看都会让人晕头转向呢，这个时候正事console.table发挥作用的时候了，以表格的形式呈现数据，自然一目了然。**

## console.assert

```js
var isDebug=false;
console.assert(isDebug,'开发中的log信息。。。');
```

> 当你想代码满足某些条件时才输出信息到控制台，那么你大可不必写if或者三元表达式来达到目的，cosole.assert便是这样场景下一种很好的工具，它会先对传入的表达式进行断言，只有表达式为假时才输出相应信息到控制台。
>
> ![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/d7536829bced6078e897a52539fec6f8~tplv-t2oaga2asx-watermark.awebp)

## console.count

> 除了条件输出的场景，还有常见的场景是计数。 当你想统计某段代码执行了多少次时也大可不必自己去写相关逻辑，内置的console.count可以很地胜任这样的任务.
>
> ![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/d0b44ef61a042a1bcd2968c185870797~tplv-t2oaga2asx-watermark.awebp)

## console.dir

> 将DOM结点以JavaScript对象的形式输出到控制台 而console.log是直接将该DOM结点以DOM树的结构进行输出，与在元素审查时看到的结构是一致的。不同的展现形式，同样的优雅，各种体位任君选择反正就是方便与体贴。

```js
console.dir(document.body);
console.log(document.body);
```

> ![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/61cd5a4e62d45dc46c1daf48668956d1~tplv-t2oaga2asx-watermark.awebp)

## console.time & console.timeEnd

> 输出一些调试信息是控制台最常用的功能，当然，它的功能远不止于此。当做一些性能测试时，同样可以在这里很方便地进行。比如需要考量一段代码执行的耗时情况时，可以用console.time与 console.timeEnd来做此事。
>
> ```js
> console.time("Array耗时");
> var array= new Array(10000000);
> for (var i = array.length - 1; i >= 0; i--) {
>     array[i] = new Object();
> };
> console.timeEnd("Array耗时");
> ```

![这里写代码片](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/1c19ece2590dd6897182c811f0ca4c76~tplv-t2oaga2asx-watermark.awebp)

> 当想要查看CPU使用相关的信息时，可以使用console.profile配合 console.profileEnd来完成这个需求。 这一功能可以通过UI界面来完成，Chrome 开发者工具里面有个tab便是Profile。使用方法和console.time基本一样，其实time开发者工具里也有个tab就是timeline。关于console.prefile博主就不做多余的介绍了。想要做更多了解的读者可以看[这里](https://link.juejin.cn/?target=https%3A%2F%2Fdevelopers.google.com%2Fweb%2Ftools%2Fchrome-devtools%2Fdebug%2Fconsole%2Fconsole-reference%3Futm_source%3Ddcc%26utm_medium%3Dredirect%26utm_campaign%3D2016q3%23consolelogobject-object)。

## $

> 讲真，米国程序员们真的很喜欢money啊（谁又不是呢），看看PHP就知道了,满屏的$。而在Chrome的控制台里，$用处同样是蛮多且方便的。

```js
2+2//回车，再
$_+1//回车得5
```

> 上面的`$_`需要领悟其奥义才能使用得当，而`$0~$4`则代表了最近5个你选择过的DOM节点。 什么意思呢？在页面右击选择审查元素，然后在弹出来的DOM结点树上面随便点选，这些被点过的节点会被记录下来，而$0会返回最近一次点选的DOM结点，以此类推，$1返回的是上上次点选的DOM节点，最多保存了5个，如果不够5个，则返回undefined。

![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/9b7ad8fcfb2f50ab540d191ac0760b2e~tplv-t2oaga2asx-watermark.awebp)

> 另外值得一赞的是，Chrome 控制台中原生支持类jQuery的选择器，也就是说你可以用$加上熟悉的css选择器来选择DOM节点，多么滴熟悉。

```js
$('body');
$('div');
```

![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/63a411d9a2e3e6b2c0797fb16d94b3a9~tplv-t2oaga2asx-watermark.awebp)

> $(selector)返回的是满足选择条件的首个DOM元素。 剥去她伪善的外衣，其实$(selector)是原生JavaScript document.querySelector() 的封装。 同时另一个命令?(selector)返回的是所有满足选择条件的元素的一个集合，是对document.querySelectorAll() 的封装。
>
> ## $x(path)
>
> 将所匹配的节点放在一个数组里返回

```js
$x("//p");
$x("//p[a]");
```

![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/27e9411e03611ab618c8043a55bf703a~tplv-t2oaga2asx-watermark.awebp)

> $x("//p")匹配所有的p节点，$x("//p[a]");匹配所有子节点包含a的p节点
>
> ## copy

```js
copy(document.body)
```

> 然后你就可以Ctrl+v了。

**注意：他不依附于任何全局变量比如window，所以其实在JS代码里是访问不了这个copy方法的，所以从代码层面来调用复制功能也就无从谈起。但愿有天浏览器会提供相应的JS实现吧~这样我们就可以通过js代码进行复制操作而不用再依赖Flash插件了。**

## keys & values

> 这是一对基友。前者返回传入对象所有属性名组成的数据，后者返回所有属性值组成的数组。具体请看下面的例子：

```js
var tfboy={name:'wayou',gender:'unknown',hobby:'opposite to the gender'};
keys(tfboy);
values(tfboy);
```

![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/30/314acd9a1c3e45bd400cf1b6d1f66241~tplv-t2oaga2asx-watermark.awebp)

## monitor & unmonitor

> monitor(function)，它接收一个函数名作为参数，比如function a,每次a被执行了，都会在控制台输出一条信息，里面包含了函数的名称a及执行时所传入的参数。而unmonitor(function)便是用来停止这一监听。
>
> ```js
> function sayHello(name){
>     alert('hello,'+name);
> }
> monitor(sayHello);
> sayHello('damonare');
> sayHello('tjz');
> unmonitor(sayHello);
> ```
>
> ![这里写图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/1b60d32c2373ad60e027e45075a2e7af~tplv-t2oaga2asx-watermark.awebp)

## debug & undebug

> debug同样也是接收一个函数名作为参数。当该函数执行时自动断下来以供调试，类似于在该函数的入口处打了个断点，可以通过debugger来做到，同时也可以通过在Chrome开发者工具里找到相应源码然后手动打断点。而undebug 则是解除该断点。而其他还有好些命令则让人没有说的欲望，因为好些都可以通过Chrome开发者工具的UI界面来操作并且比用在控制台输入要方便。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2016/11/29/2e8585130071b038a8b8040714efca7e~tplv-t2oaga2asx-watermark.awebp)

**本博文依据[Console API文档](https://link.juejin.cn/?target=https%3A%2F%2Fdevelopers.google.com%2Fweb%2Ftools%2Fchrome-devtools%2Fdebug%2Fconsole%2Fconsole-reference%3Futm_source%3Ddcc%26utm_medium%3Dredirect%26utm_campaign%3D2016q3%23consolelogobject-object)和[Commond API](https://link.juejin.cn/?target=https%3A%2F%2Fdevelopers.google.com%2Fweb%2Ftools%2Fchrome-devtools%2Fdebug%2Fcommand-line%2Fcommand-line-reference%3Futm_source%3Ddcc%26utm_medium%3Dredirect%26utm_campaign%3D2016q3%23debugfunction)书写。**
