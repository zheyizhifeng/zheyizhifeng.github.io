---
title: 撸一个 VSCode Extension
date: 2021-10-18 15:25:03
categories: 
  - Tools
  - VSCode
tags: 
  - Tools
  - VSCode
  - 插件
---

> 工欲善其事，必先利其器

# 背景

Visual Studio Code (以下简称 VSCode) 是一个轻量且强大的跨平台开源代码编辑器。VSCode 采用了 Electron 技术，使用的代码编辑器名为 Monaco，Monaco 也是 Visual Studio Team Service（Visual Studio Online）使用的代码编辑器，在开发语言上，VSCode 使用了自家的 TypeScript 语言进行开发。

VSCode提供了强大的插件拓展机制，并提供 [插件市场](https://marketplace.visualstudio.com/) 供开发者发布、下载插件。VSCode 提供了丰富的扩展能力模型，例如基础的语法高亮、API 提示、引用跳转（转到定义）、文件搜索、主题定制、高级的 Debug 协议等等，但不允许插件直接访问底层 UI DOM (即很难定制 VSCode 外观)。
<!-- more -->
当你想要开发一款专用 IDE 时，不想从零开始撸，而是站在巨人的肩膀上做二次开发的话，VSCode 将是你不二的选择，像 [Weex Studio](http://emas.weex.io/zh/tools/ide.html)、[白鹭Egret Wing](https://www.egret.com/products/wing.html)、[快应用IDE](https://www.quickapp.cn/docCenter/IDEPublicity) 等，都是基于 VSCode 扩展增强。

## Weex Studio

![Weex Studio](https://codeteenager.github.io/vscode-analysis/assets/img/weex.ce49efd6.png)

## Egret Wing

![Egret Wing](https://codeteenager.github.io/vscode-analysis/assets/img/egret.c4b0766a.png)

## 快应用 IDE

![快应用IDE](https://codeteenager.github.io/vscode-analysis/assets/img/quick.93816670.png)

# 技术构成

## Electron

众所周知，VSCode 是一款桌面编辑器应用，但是前端单纯用 JS 是做不了桌面应用的，所以采用 [Electron](https://electronjs.org/) 来构建。Electron 是基于 Chromium 和 Node.JS，使用 JavaScript、 HTML 和 CSS 构建跨平台的桌面应用，它兼容 Mac、Windows 和 Linux，可以构建出三个平台的应用程序。 ![electron-architecture](https://codeteenager.github.io/vscode-analysis/assets/img/electron-architecture.876dcbba.png) 从实现上来看，`Electron = Node.js + Chromium + Native API`

也就是说 Electron 拥有 Node 运行环境，赋予了用户与系统底层进行交互的能力，依靠 Chromium 提供基于 Web 技术（HTML、CSS、JS）的界面交互支持，另外还具有一些平台特性，比如桌面通知等。

![electron-process](https://codeteenager.github.io/vscode-analysis/assets/img/electron-process.13fda2d1.jpeg)

从 API 设计上来看，Electron App 一般都有 1 个主进程（Main Process）和多个渲染进程（Renderer Process）

- main process：主进程环境下可以访问 Node 及 Native API
- renderer process：渲染器进程环境下可以访问 Browser API 和 Node API 及一部分 Native API

### 主进程和渲染进程

在 Electron 应用中，通过执行 package.json 中的 main 字段所指向的文件，可以开启 Electron 的主进程（main process）。在主进程中使用 `BrowserWindow` 实例创建 Web 页面，而且一个 Electron 应用有且只能有一个主进程。 主进程一般用于：
- 多窗体管理（创建/切换）
- 应用生命周期管理
- 作为进程通信基站（IPC Server）
- 工具条菜单栏注册

由于 Electron 使用 Chromium 来展示 Web 页面， Chromium 多进程架构也会被用到。每一张 Web 页面都运行在它自己的进程里，该进程称为渲染进程（renderer process）。渲染进程一般负责界面交互相关的，具体的业务功能。

> 在 Web 页面里，调用系统底层的 API 是不被允许的，这是因为在 Web 页面上处理底层 GUI 资源是**非常危险**的，很容易导致资源泄漏。如果你想要在 Web 页面上执行 GUI 操作，相应 Web 页面的渲染进程必须与主进程进行通信，向主进程发起请求去执行那些操作。在 Electron 中，有几种主进程与渲染进程通信的方法，比如用 `ipcRenderer` 和 `ipcMain` 模块来发送信息，还有 RPC 风格的远程通信模块。

## Monaco Editor

微软之前有个项目叫做 Monaco Workbench，后来这个项目变成了 VSCode，而 [Monaco Editor](https://microsoft.github.io/monaco-editor/)（下文简称 Monaco）就是从这个项目中成长出来的一个 Web 编辑器，他们很大一部分的代码（monaco-editor-core）都是共用的，所以 Monaco 和 VSCode 在代码编辑、交互以及 UI 上几乎是一摸一样的，有点不同的是，两者的平台不一样，Monaco 基于浏览器，而 VSCode 基于Electron，所以功能上 VSCode 更加健全，并且性能比较强大。

## TypeScript

[TypeScript](https://www.tslang.cn/) 是 JavaScript 类型的超集，它可以编译为纯 JavaScript。TypeScript 可以在任何浏览器、任何计算机和任何操作系统中运行，并且是开源的。TypeScript具有以下特点：

- 类型批注和编译时的类型检查
- 强类型语言
- 面向对象
- 类 class
- 接口
- lambda 函数
- 泛型

# VSCode 架构
![vscode-architecture](https://codeteenager.github.io/vscode-analysis/assets/img/vscode-architecture.246227f5.jpeg)

VSCode 中包含主进程、渲染进程，同时因为 VSCode 提供了插件的扩展能力，又出于安全稳定性的考虑，又多了提供了一个独立的进程 Extension Host，用于运行我们的插件代码。并且同渲染进程一样，彼此都是独立互不影响的。Extension Host Process 暴露了一些 VSCode 的 API 供插件开发者去使用。 

## VSCode的进程结构

VSCode采用多进程架构，启动后主要由下面几个进程：

- 后台进程
- 编辑器窗口 - 由后台进程启动，也是多进程架构
  - HTML 编写的 UI
    - ActivityBar
    - SideBar
    - Panel
    - Editor
    - StatusBar
  - Node.JS 异步 IO
    - FileService
    - ConfigurationService
  - 插件宿主进程
    - 插件实例
      - 插件子进程 - 如 TS 语言服务
    - 插件实例
    - 插件实例
  - Debug 进程
  - Search 进程

![vscode-process](https://codeteenager.github.io/vscode-analysis/assets/img/vscode-process.e445e292.png)

### 后台进程

后台进程是 VSCode 的入口，主要负责管理编辑器生命周期，进程间通信，自动更新，菜单管理等。

我们启动 VSCode 的时候，后台进程会首先启动，读取各种配置信息和历史记录，然后将这些信息和主窗口 UI 的 HTML 主文件路径整合成一个 URL，启动一个浏览器窗口来显示编辑器的 UI。后台进程会一直关注 UI 进程的状态，当所有 UI 进程被关闭的时候，整个编辑器退出。

此外后台进程还会开启一个本地的 Socket，当有新的 VSCode 进程启动的时候，会尝试连接这个 Socket，并将启动的参数信息传递给它，由已经存在的 VSCode 来执行相关的动作，这样能够保证 VSCode 的唯一性，避免出现多开文件夹带来的问题。

### 编辑器窗口

编辑器窗口进程负责整个 UI 的展示。也就是我们所见的部分。UI 全部用 HTML 编写没有太多需要介绍的部分。

### Nodejs异步IO

项目文件的读取和保存由主进程的 NodeJS API 完成，因为全部是异步操作，即便有比较大的文件，也不会对 UI 造成阻塞。IO 跟 UI 在一个进程，并采用异步操作，在保证 IO 性能的基础上也保证了 UI 的响应速度。

### 插件进程

每一个 UI 窗口会启动一个 NodeJS 子进程作为插件的宿主进程。所有的插件会共同运行在这个进程中。这样设计最主要的目的就是避免复杂的插件系统阻塞 UI 的响应。但是将插件放在一个单独进程也有很明显的缺点，因为是一个单独的进程，而不是 UI 进程，所以没有办法直接访问 DOM 树，想要实时高效的改变 UI 变得很难，在 VSCode 的扩展体系中几乎没有对 UI 进行扩展的 API。

### Debug进程

Debugger 插件跟普通的插件有一点区别，它不运行在插件进程中，而是在每次 debug 的时候由UI单独新开一个进程。

### 搜索进程

搜索是一个十分耗时的任务，VSCode 也使用的单独的进程来实现这个功能，保证主窗口的效率。将耗时的任务分到多个进程中，有效的保证了主进程的响应速度。

# VSCode Extension

VSCode 插件其实是一个类似于 npm package 后缀名为 `.vsix` 的文件，其可以实现的功能大概可以分为以下几类：

- 自定义命令
- 快捷键
- 自定义菜单项
- 自定义跳转
- 自动补全
- 悬浮提示
- 新增语言支持
- 语法检查
- 语法高亮
- ###### 代码格式化
  
  ····

# [快速上手 Extension](https://code.visualstudio.com/api/get-started/your-first-extension) 

微软官方提供了脚手架，可以快速搭建一个项目

## 项目初始化

```sh
npm install -g yo generator-code
yo code hello-world
cd hello-world
code-insiders .
# 如果没有 code-insiders 命令，手动用编辑器打开项目即可
```

![image-20211018171039777](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211018171039777.png)

> 脚手架工具支持创建插件（New Extension）、主题（New Color Theme）、新语言支持（New Language Support）、代码片段（New Code Snippets）、键盘映射（New Keymap）、插件包（New Extension Pack）。以上不同类型的脚手架模板只是侧重的预设功能不同，其本质还是 VSCode 插件。

![image-20211018171819449](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211018171819449.png)

![image-20211018172759719](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211018172759719.png)

## 运行调试

借由VSCode自带的调试功能，脚手架已经自动为我们配置好了，也就是`.vscode`目录下的内容

![image-20211018174442100](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211018174442100.png)

1）选择 VSCode 的调试菜单 (`Cmd/Ctrl+shift+D`)，点击运行 `Run Extension` 或者按下 `F5` 后，会自动打开一个新的 VSCode 界面，此时这个新打开的 VSCode，会自动安装我们开发的插件，VSCode 头部会展示 `[Extension Development Host]`，此时我们打开命令面板 `Cmd+Shift+P` 或者 `Ctrl+Shift+P` 搜索 `Hello World` 并且执行。

![image-20211018174635537](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211018174635537.png)

2）此时我们在这个界面调试插件功能，如果需要测试代码，我们可以在此 VSCode 新建一个本地项目或者打开已有项目测试

3）我们写的 `console` 相关日志，可以在 `DEBUG CONSOLE` 面板中查看，也可以在运行过程中增删断点

![image-20211018175139649](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211018175139649.png)

4）如果修改了插件内容，`Webpack` 会自动监听并编译

5）打开的 `[Extension Development Host]` 的 VSCode 调试页面如何更新修改呢？

- 方法一：断开并重新执行
- 方法二：在此界面按住 `Command+R`，则会自动重启并应用，推荐。

## 发布更新

1、注册账号 [https://aka.ms/SignupAzureDevOps](https://links.jianshu.com/go?to=https%3A%2F%2Faka.ms%2FSignupAzureDevOps)

2、生成并保存 access token

![image-20211019170036842](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211019170036842.png)

![image-20211019170012287](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211019170012287.png)

3、安装 vsce 打包工具

```sh
npm install -g vsce
```

4、创建发布账号

> ~~vsce create-publisher your-publisher-name~~ # 已废弃
![image-20211019165300566](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211019165300566.png)

原有的通过`vsce create publisher`命令创建发布者的方式已经被废弃，现在请访问 https://aka.ms/vscode-create-publisher 手动创建发布者信息。

![image-20211019170714638](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211019170714638.png)

5、登录账号

```sh
vsce login your-publisher-name
```

6、发布

```sh
vsce publish
```

# VSCode Extension 剖析

以下是一个 VSCode Extension 项目的目录结构。

```sh
# hello-world 插件目录结构
├── .eslintrc.json # eslint配置
├── .vscode # vscode调试配置
|  ├── extensions.json
|  ├── launch.json
|  ├── settings.json
|  └── tasks.json
├── .vscodeignore # 发布忽略内容
├── CHANGELOG.md # 修改日志
├── README.md # 插件发布后，插件主页内容
├── package-lock.json
├── package.json # vscode从这里识别插件贡献点
├── test # 测试文件夹
├── extension.js # 入口文件
├── jsconfig.json # js配置文件
├── vsc-extension-quickstart.md # 新手指引
└── webpack.config.js # wepack配置
```

其中最核心的两个文件是 `package.json` 和 `extension.js`，`package.json` 是整个插件工程的配置文件，`extension.js` 则是工程的入口文件。下面将对这两个文件进行详细的介绍。

![img](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/webp)

`Hello World` 插件包含了3个部分：
- 注册 `onCommand`：
`onCommand:extension.helloWorld`，用户输入 `Hello World` 命令后激活插件。
- 使用 `contributes.commands`，绑定一个命令 `extension.helloWorld`，然后 `Hello World` 命令就可以在命令面板中使用了。
- 使用 `commands.registerCommand`: 将一个函数实现绑定到注册的命令 `extension.helloWorld` 上。

大体上，插件就是通过组合内容配置和 VS Code API 扩展 VS Code 的功能。

## package.json

![image-20211018204136327](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211018204136327.png)

每个 VSCode 插件都必须包含一个 `package.json`，它是插件的声明文件。这里列出部分基本的字段：

| 字段名                                                       | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| name                                                         | 插件名（要求全小写，无空格）                                 |
| version                                                      | 插件版本（使用这个控制插件版本，类似 npm ）                    |
| publisher                                                    | 发布者                                                       |
| engines                                                      | 对 vscode 或其他环境依赖版本的要求                             |
| main                                                         | 插件入口文件                                                 |
| [contributes](https://link.juejin.cn?target=https%3A%2F%2Fcode.visualstudio.com%2Fapi%2Freferences%2Fcontribution-points) | 插件贡献点，基本上我们增加的功能点，都需要在这里显示声明，如注册的命令、注册的菜单、注册的快捷键 |
| [activationEvents](https://link.juejin.cn?target=https%3A%2F%2Fcode.visualstudio.com%2Fapi%2Freferences%2Factivation-events) | 插件触发动作，如在打开某种语言的文件时、当某个命令被触发时   |

### activationEvents

用于定义插件何时被加载/激活。例子中用到的是 onCommand，在 Hello World 命令被调用时，插件才会被激活。目前支持 9 种激活事件：

- `onLanguage:${language}` 打开特定语言文件时
- `onCommand:${command}` 调用某个 VSCode 命令时
- `onDebug` Debug 时
- `workspaceContains:${toplevelfilename}` 当打开包含某个命名规则的文件夹时
- `onFileSystem:${scheme}` 以某个协议（ftp/sftp/ssh 等）打开文件或文件夹时
- `onView:${viewId}` 指定 id 的视图展开时
- `onUri` 插件的系统级 URI 打开时
- `onWebviewPanel` Webview 触发时
- `*` VSCode 启动时。不建议使用，性能上会受到一定影响。

> 出于性能考虑，插件都是懒加载的，只有特定场景下才会加载/激活，才会消耗内存等资源。

### contributes

`contributes` 配置项是整个插件的贡献点，也就是说这个插件有哪些功能。`contributes` 字段可以设置的 `key` 也基本显示了 `VSCode` 插件可以做什么。

-  configuration：通过这个配置项我们可以设置一个属性，这个属性可以在 `VSCode` 的 `settings.json` 中设置，然后在插件工程中可以读取用户设置的这个值，进行相应的逻辑。
-  commands：命令，通过 `cmd`+`shift`+`p` 进行输入来实现的。
-  menus：通过这个选项我们可以设置右键的菜单
-  keybindings：可以设置快捷键
-  languages：设置语言特点，包括语言的后缀等
-  grammars：可以在这个配置项里设置描述语言的语法文件的路径，VSCode 可以根据这个语法文件来自动实现语法高亮功能
-  snippets：设置语法片段相关的路径......

## extension.js/ts

插件的执行入口文件，通常包括激活（activate）和禁用（deactivate）2 个方法。注册的**激活事件**被触发之时执行 `activate`，`deactivate` 则提供了插件关闭前执行清理工作的机会。命令必须先使用`vscode.commands.registerCommand` 进行注册，然后将返回的实例添加至 context.subscriptions 中。当命令被激活时，会执行相应的回调方法。

[`vscode `](https://www.npmjs.com/package/vscode) 模块包含了一个位于 `node ./node_modules/vscode/bin/install` 的脚本，这个脚本会拉取 `package.json` 中 `engines.vscode`字段定义的 VS Code API。这个脚本执行过后，你就得到了智能代码提示，定义跳转等 TS 特性了。

```typescript
// 'vscode'模块包含了VS Code extensibility API
// 按下述方式导入这个模块
import * as vscode from 'vscode';

// 一旦你的插件激活，vscode会立刻调用下述方法
export function activate(context: vscode.ExtensionContext) {

    // 用console输出诊断信息(console.log)和错误(console.error)
    // 下面的代码只会在你的插件激活时执行一次
    console.log('Congratulations, your extension "my-first-extension" is now active!');

    // 入口命令已经在package.json文件中定义好了，现在调用registerCommand方法
    // registerCommand中的参数必须与package.json中的command保持一致
    let disposable = vscode.commands.registerCommand('extension.sayHello', () => {
        // 把你的代码写在这里，每次命令执行时都会调用这里的代码
        // ...
        // 给用户显示一个消息提示
        vscode.window.showInformationMessage('Hello World!');
    });

    context.subscriptions.push(disposable);
}
```

# Snippets 代码片段自动补全 Extension

> 代码片段自动补全也是日常编写代码时使用频率最高、最能帮助提高编码速度的功能。这里以一个简单的代码片段自动补全插件为例，再次熟悉VSCode的插件开发流程。

## 添加 Snippets 配置项

```json
// package.json
"contributes": {
		"snippets": [
			{
				"language": "javascript",
				"path": "./snippets/javascript.json"
			},
			{
				"language": "typescript",
				"path": "./snippets/javascript.json"
			}
		]
	}
```

在 package.json 的 contributes 下添加自定义的 Snippets。language 表示在某种特定语言下，对应的代码片段才会被加载生效。path 表示代码片段文件的存放路径。上面配置即表示 javascript 或 typescript 语言环境下，将加载 `./snippets/javascript.json` 文件中的代码片段。

## 编写 Snippets 代码片段

```json
// ./snippets/javascript.json
{
  "forEach": {
    "prefix": "fe",
    "body": [
      "${1:array}.forEach(function(item) {",
      "\t${2:// body}",
      "});"
    ],
    "description": "Code snippet for 'forEach'"
  },
  "setTimeout": {
    "prefix": "st",
    "body": [
      "setTimeout(function() {",
      "\t${0:// body}",
      "}, ${1:1000});"
    ],
    "description": "Code snippet for 'setTimeout'"
  }
}
```

上述例子中：

- **forEach**、**setTimeout** 是代码片段的名称。
- **prefix** 中定义一个或多个（设置数组时可以指定多个）触发词（trigger words），当用户输入内容是触发词时编辑器会弹出自动补全提示。
- **body** 中定义的就是填充的代码段内容。body 中可以使用占位符（placeholders），如上面的 `${1:array}`、 `{2:// body}`，使用占位符方便在引用代码段的时候可以通过 tab 键快速切换跳转到对应位置编辑。冒号前面的序号表示切换的顺序，冒号后面的内容则是占位显示的默认文本。
- **description** 顾名思义就是代码段的描述说明，编辑器弹出补全提示的时候会展示该描述，如果没有设置 description 字段，那么会直接展示代码段名称。

## 运行结果

![image-20211018210823127](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20211018210823127.png)

# 推荐 Extension

- Tabnine
- Code Runner
- Remote SSH
- Live Server
- Path Intelligence
- Bracket Pair Colorizer
- Prettier
