---
title: Chrome Extension With Manifest V3
date: 2022-02-27 17:21:37
categories: 
  - Chrome
tags:
  - Chrome
  - Extension
  - JavaScript
---

# Chrome Extension With Manifest V3

## What Extension Is

>  Extensions are software programs, built on web technologies (such as HTML, CSS, and JavaScript) that enable users to customize the Chrome browsing experience.
>
>  扩展程序是基于 Web 技术（例如 HTML、CSS 和 JavaScript）构建的软件程序，可让用户自定义 Chrome 浏览体验。

以上是对chrome extension的官方介绍，个人理解就是：

> Chrome Extension 是能通过『当前选项卡』『插件弹出页』『全局js脚本』『devtools信息』等合作通信去实现特定功能的后台程序。

## What Manifest Is
### About manifest.json
> Each extender needs to have a **configuration checklist `manifest.json` document **, which provides basic information about the extension, such as required permissions, names, versions, and so on.
>
> 每一个扩展程序都需要有一个**配置清单 `manifest.json` 文档**，它提供了关于扩展程序的基本信息，例如所需的权限、名称、版本等。

```json
  {
  // Required - 通俗易懂
  "manifest_version": 3,
  "name": "My Extension",
  "version": "versionString",

   // 『重点』action配置项主要用于点击图标弹出框，对于弹出框接受的是html文件
  "action": {
     "default_title": "Click to view a popup",
   	 "default_popup": "popup.html"
   }
    
  // 通俗易懂
  "default_locale": "en",
  "description": "A plain text description",
  "icons": {...},
  "author": ...,

  // 『重点』下面将出现的background.js 配置service work
  "background": {
    // Required
    "service_worker": "service-worker.js",
  },

    // 『重点』下面将出现content_script.js 应用于所有页面上下文的js
  "content_scripts": [
     {
       "matches": ["https://*.google.com/*"],
       "css": ["my-styles.css"],
       "js": ["content-script.js"]
     }
   ],

    // 使用/添加devtools中的功能
  "devtools_page": "devtools.html",


    /**
    * 三个permission
    * host_permissions - 允许使用扩展的域名
    * permissions - 包含已知字符串列表中的项目 【只需一次弹框要求允许】
    * optional_permissions - 与常规类似permissions，但由扩展的用户在运行时授予，而不是提前授予【安全】
    * 列出常见选项
    * {
    *		activeTab: 当扩展卡选项被改变需要重新获取新的权限
    *		tabs: 操作选项卡api（改变位置等）
    *		downloads: 访问chrome.downloads API 的权限 便于下载但还是会受到跨域影响
    *		history: history api权限
    *		storage: 访问localstorage/sessionStorage权限
    * }
    */
  "host_permissions": ["http://*/*", "https://*/*"],
  "permissions": ["tabs"],
  "optional_permissions": ["downloads"],

    // 内部弹出可选页面 - 见fehelper操作页
  "options_page": "options.html",
  "options_ui": {
    "chrome_style": true,
    "page": "options.html"
  },
}
```

### Why Not Manifest V2

**当前配置清单类型最新的版本是 Manifest V3，MV3**，遵循该配置清单版本的扩展程序，会更注重安全和用户的隐私保护，在性能方面也会得到提升，同时开发简易性和功能实现也会更佳。

![image-20220217175042667](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220217175042667.png)

![img](https://static.careerengine.us/api/aov2/https%3A_%7C__%7C_mmbiz.qpic.cn_%7C_mmbiz_png_%7C_dkwuWwLoRK95qOVpY72zicpWwLAHwxKPr7SrWicJERd9iaQklkZ0DpCKG2GMHOzS3AhicmVIur4pbMrcH5mbgyQKtw_%7C_640%3Fwx_fmt%3Dpng)

## Build A MV3 Hello Extension

https://github.com/GoogleChrome/chrome-extensions-samples/tree/main/examples/hello-world

### Project Structure

```bash
--hello
  --hello.html // 插件界面
  --background.js // service_worker
  --manifest.json // 配置文件
```

```json
// manifest.json

{
  "name": "Hello, World!",
  "version": "1.0",
  "manifest_version": 3,
  "background": {
    "service_worker": "background.js"
  },
  "action": {}
}
```

```js
// background.js

chrome.runtime.onInstalled.addListener(async () => {
  let url = chrome.runtime.getURL('hello.html');
  let tab = await chrome.tabs.create({ url });
  console.log(`Created tab ${tab.id}`);
});
```

```html
<!-- hello.html -->

<!DOCTYPE html>
<html>
<head>
  <title>Hello, World!</title>
</head>
<body>
  <p>Hello, World!</p>
</body>
</html>
```

### Install / Run Hello Extension

![image-20220217183003834](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220217183003834.png)

![image-20220217183440530](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220217183440530.png)

> 新版 Chrome 不再支持 `.crx` 形式的扩展程序，可以安装，但无法启用。

![image-20220217184100318](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220217184100318.png)

![image-20220217184118220](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220217184118220.png)

### Debug Extension

![image-20220217202754281](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220217202754281.png)

## More Extension

### page-render

https://github.com/GoogleChrome/chrome-extensions-samples/tree/main/examples/page-redder

### bookmarks

https://github.com/GoogleChrome/chrome-extensions-samples/tree/main/examples/bookmarks

##  Chrome Extension Basic Composition

![Keep coding, Keep thinking](http://www.co-ding.com/assets/img/2019-10-18-chrome-1.png)

### service_worker(background.js)

> The **background script** is the extension's event handler; it contains listeners for browser events that are important to the extension. It lies dormant until an event is fired then performs the instructed logic. An effective background script is only loaded when it is needed and unloaded when it goes idle.
> 
> **background script**是扩展的事件处理程序; 它包含对扩展很重要的浏览器事件的侦听器。它处于休眠状态，直到触发事件，然后执行指示的逻辑。有效的后台脚本仅在需要时加载，并在空闲时卸载。

要关注两点：

- 不使用时终止，需要时重新启动（类似于事件页面）。
- 无权访问 DOM。（service worker 独立于页面）

```js
chrome.runtime.onMessage.addListener((message, callback) => {
  const tabId = getForegroundTabId();
  if (message.data === "setAlarm") {
    chrome.alarms.create({delayInMinutes: 5})
  } else if (message.data === "runLogic") {
    chrome.scripting.executeScript({file: 'logic.js', tabId});
  } else if (message.data === "changeColor") {
    chrome.scripting.executeScript(
        {func: () => document.body.style.backgroundColor="orange", tabId});
  };
});
```

### content_script

> Extensions that read or write to web pages utilize a **content_script**. The content script contains JavaScript that executes in the contexts of a page that has been loaded into the browser. Content scripts read and modify the DOM of web pages the browser visits.
>
> Content scripts can communicate with their parent extension by exchanging [messages](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.chrome.com%2Fdocs%2Fextensions%2Fmv3%2Fmessaging%2F) and storing values using the [storage](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.chrome.com%2Fdocs%2Fextensions%2Freference%2Fstorage%2F) API.
>
> 读取或写入网页的扩展程序使用 **content_script**。内容脚本包含在已加载到浏览器的页面上下文中执行的 JavaScript。内容脚本读取和修改浏览器访问的网页的 DOM。
>
`content_script`可以通过使用storage/message API来与扩展其他部分进行通信。

#### 注入方式

- 对于`manifest.json`来说

  - 可以配置静态声明去注入
- 可以通过编程方式注入 需要获取`activeTab`权限

```json
// manifest.json
{
 "name": "My extension",
 ...
 "content_scripts": [
   {
     // 满足matches匹配的域名
     "matches": ["https://*.google.com/*"],
     // 注入css
     "css": ["my-styles.css"],
     // 注入js
     "js": ["content-script.js"],
     "run_at": "document_idle" | "document_start" | "document_end"
   }
 ],
  "permissions": [
    "activeTab"
  ],
}
```

- 对于获取了权限的`content_script`通过代码执行注入

```js
chrome.action.onClicked.addListener((tab) => {
  chrome.scripting.executeScript({
    target: { tabId: tab.id },
    files: ['content-script.js']
  });
});
```

### popup

运行于弹窗的html文件和JS脚本

![image-20220217195734158](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220217195734158.png)

### option

> Just as extensions allow users to customize the Chrome browser, the [options page](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.chrome.com%2Fdocs%2Fextensions%2Fmv3%2Foptions%2F) enables customization of the extension. Options can be used to enable features and allow users to choose what functionality is relevant to their needs.
> 
> 正如扩展程序允许用户自定义 Chrome 浏览器一样，[选项页面](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.chrome.com%2Fdocs%2Fextensions%2Fmv3%2Foptions%2F)支持扩展程序的自定义。选项可用于启用功能并允许用户选择与其需求相关的功能。

![image-20220217200117586](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220217200117586.png)

### devtools

>  A DevTools extension is structured like any other extension: it can have a background page, content scripts, and other items. In addition, each DevTools extension has a DevTools page, which has access to the DevTools APIs.
> 
> DevTools 扩展的结构与任何其他扩展一样：它可以有一个背景页面、内容脚本和其他项目。此外，每个 DevTools 扩展都有一个 DevTools 页面，可以访问 DevTools API。

![Extending DevTools - Chrome Developers](https://wd.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/kcLMpTY6qtez03TVSqt4.png?auto=format)

每次打开 DevTools 窗口时，都会创建一个扩展的 DevTools 页面实例。DevTools 页面在 DevTools 窗口的整个生命周期内都存在。DevTools 页面可以访问 DevTools API 和一组有限的扩展 API。devtools 可以提供能力：

- 能自定义添加新的Devtools panel
- 能嵌入自定义的页面到自定义的panel中
- *devtools中的 elements & sources两个面板中*

![image-20220217200944834](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220217200944834.png)

![image-20211020151945359](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/826e09b59b2647679eef9f7b42be652a~tplv-k3u1fbpfcp-watermark.awebp)

![2.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1932d450523c4b19972a820a00d920f4~tplv-k3u1fbpfcp-watermark.awebp?)

```js
chrome.devtools.panels.create(
  '🔨 Beyla',
  'images/128x128.png',
  'html/devtools.html',
  (panel) => {
    // panel loaded
    panel.onShown.addListener(onPanelShown);
    panel.onHidden.addListener(onPanelHidden);
  }
);

// [devtools].js
chrome.devtools.panels.elements.createSidebarPane(
	"element pannel",
	(sidebar) => {
		sidebar.setExpression("(() => {return {a:1}})()");
	}
);

chrome.devtools.panels.sources.createSidebarPane(
	"sources pannel",
	(sidebar) => {
		sidebar.setPage("sources.html");
	}
);
```

## Basic Composition Communication

![组件间关系图](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f890ba9522574bbda5a0223e22b01c71~tplv-k3u1fbpfcp-watermark.awebp)

| JS种类          | 可访问的API                                    | DOM访问情况      | JS访问情况       | 直接跨域 |
| --------------- | ---------------------------------------------- | ---------------- | ---------------- | -------- |
| injected script | 和普通JS无任何差别，不能访问任何扩展API        | 可以访问         | 可以访问         | 不可以   |
| content script  | 只能访问 extension、runtime等部分API           | 可以访问         | 不可以           | 不可以   |
| popup js        | 可访问绝大部分API，除了devtools系列            | 不可直接访问     | 不可以           | 可以     |
| background js   | 可访问绝大部分API，除了devtools系列            | 不可直接访问     | 不可以           | 可以     |
| devtools js     | 只能访问 devtools、extension、runtime等部分API | 可以访问devtools | 可以访问devtools | 不可以   |

### content script && service worker/popup

和 `content script` 有关的通信

- 使用 `chrome.runtime.sendMessage` 发送信息
- 使用 `chrome.runtime.onMessage.addListener ` 接收监听信息

```js
// -----[content script].js-----
// 发送信息
chrome.runtime.sendMessage({ beylaDetected: true });

// -----[service worker].js-----
// 监听消息
chrome.runtime.onMessage.addListener((req, sender) => {
  if (sender.tab && req.beylaDetected) {
    chrome.action.setIcon({
      tabId: sender.tab.id,
      path: '../images/128x128.png',
    });
    chrome.action.setPopup({
      tabId: sender.tab.id,
      popup: 'html/beyla-enable.html',
    });
  }
});
```

### popup && service worker

> 当只有一对一关系时，可使用万能的`chrome.runtime.sendMessage` & `chrome.runtime.onMessage.addListener`

```js
// -----[popup].js-----
document.querySelector("#button").addEventListener("click", () => {
	const val1 = document.querySelector("#input1").value || "0";
	const val2 = document.querySelector("#input2").value || "0";
	chrome.runtime.sendMessage({val1, val2}, (response) => {
		document.querySelector("#ans").innerHTML = response.res;
	});
});

// -----[service worker].js-----
const dealwithBigNumber = (val1, val2) => BigInt(val1) * BigInt(val2) + "";
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
	const {val1, val2} = request;
	sendResponse({res: dealwithBigNumber(val1, val2)});
});
```

### popup && content script

> 本demo中引用了jquery，想看怎么操作的请看 [源码](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcaifeng123%2Fchrome-extension) ~~

```js
// -----[content script].js-----
// 接收popup数据并修改dom
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
	// document.querySelector("body").style.setProperty("background", request.color);
	$("body").css("background", request.color);
	sendResponse({name: 1});
});


// -----[popup].js-----
// 获取当前tab标签
const getCurrentTab = async () => {
	let queryOptions = {active: true, currentWindow: true};
	let [tab] = await chrome.tabs.query(queryOptions);
	return tab;
};

$("#background").paigusu({color: "#1926dc"}, async (event, obj) => {
	$("#info").innerHTML = "修改中";
	$("#show").css("background", "#" + obj.hex);
	const tab = await getCurrentTab();
	await chrome.tabs.sendMessage(tab.id, {color: "#" + obj.hex});
	$("#info").innerHTML = "修改成功";
});
```

### devtools && content script

> 同1 content script与service worker/popup的通信

​	1对多通信 使用chrome.tabs去找对应页面

### devtools && popup

> 同2  content script与service worker/popup的通信

​	1对1通信 使用chrome.runtime

## Beyla Chrome Extension

Repo 参考 https://gitlab.ushareit.me/web/fe-workflow/h5/beyla-chrome-extension.git

## Recommend Extension

![image-20220224214333026](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20220224214333026.png)
