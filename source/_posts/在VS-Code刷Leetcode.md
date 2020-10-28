---
title: 在VS Code刷Leetcode
date: 2020-10-28 10:39:06
tags:
---
# 如何优雅的在VS Code刷Leetcode算法题库？

## Requirements / 要求

------

**VS Code和Node的版本要求**

- [*VS Code 1.30.1+*](https://code.visualstudio.com/)
- [*Node.js 8+*](https://nodejs.org/en/)

------

## Quick Start / 快速开始

------

首先，我们需要给VS Code安装一个`leetcode`插件

**安装方法：**

> 打开`VS Code` -> 左边操作栏找到`Extensions` -> 搜索插件名称：`leetcode` -> `install`

[*VS Code LeetCode在线下载地址*](https://marketplace.visualstudio.com/items?itemName=shengchen.vscode-leetcode)

------

## Features / 功能

------

### 登陆登出

- 点击`LeetCode Explorer`中的`Sign in to LeetCode` 即可登入。

- 也可以使用快捷键(

  ```
  Ctrl+shift+p
  ```

  )唤起下列命令登入或利用cookie登入或登出:

  - **LeetCode: Sign in**
  - **LeetCode: Sign in (by cookie)**
  - **LeetCode: Sign out**

### 选择题目

- 直接点击题目或者在 `LeetCode Explorer` 中右键题目并选择 `Preview Problem` 可查看题目描述
- 选择 `Show Problem` 可直接进行答题。

> 注意：可以通过更新配置项 `leetcode.workspaceFolder` 来指定保存题目文件所用的工作区路径。默认工作区路径为：**$HOME/.leetcode/**。
> 注意：可以通过更新配置项 `leetcode.showCommentDescription` 来指定是否要在注释中包含题目描述。
> 注意：可以通过 `LeetCode: Switch Default Language` 命令变更答题时默认使用编程语言。

### 编辑器快捷方式

- `Submit`: 提交你的答案至 `LeetCode`；

- `Test`: 用给定的测试用例测试你的答案；

- `Solution`: 显示该问题的高票解答；

- ```
  Description
  ```

  : 显示该问题的题目描述。

  > *注意：可以通过 `leetcode.editor.shortcuts` 配置项来定制需要激活的快捷方式。默认情况下只有 `Submit` 和 `Test` 会被激活。*

### 通过关键字搜索题目

- 点击 `LeetCode Explorer` 导航栏中的搜索按钮可按照关键字搜索题目。

------

## Problem / 问题

------

### 国内无法登陆海外版问题

**如何解决VS Code的`vscode-leetcode`插件无法登陆海外版的问题？**

> *注意：[登陆的endpoint(端点)：https://leetcode.com（并非中国leetcode-cn.com社区）](https://leetcode.com/)*

**几个解决方案：**

1、切换 LeetCode 版本为中国版，点击`Switch Endpoint`切换为`leetcode-cn.com`(中国社区版)
2、用`leetcode-cli`命令行工具进行登陆
3、正常输入用户名和密码无法登陆的情况下，通过获取[*leetcode.com*](https://leetcode.com/)的`Cookies`信息进行登陆。

**具体实现步骤：**

- 先保证退出账户
- 确保当前激活的节点为`leetcode.com`（而非`leetcode-cn.com`）
- 访问端点：[*https://leetcode.com*](https://leetcode.com/)并打开Chrome开发者工具，选择`Network` -> 选择`XHR`
- Web端登陆账号密码后，并点击problems按钮获取leetcode的session和csrf token信息
- 找到`all`这个`api`接口：`https://leetcode.com/api/problems/all/`，并copy完整的`Cookies`
- 打卡VS Code用快捷键：`ctrl + shift + p` 输入leetcode，选择`Sign In by Cookie`
- 输入邮箱 -> paste 刚刚从览器copy的`cookies`信息 -> ok,done. Start coding…

------

## 参考链接

- [*vscode-leetcode Github*](https://github.com/jdneo/vscode-leetcode)
- [*vscode-leetcode 中文使用教程*](https://github.com/jdneo/vscode-leetcode/blob/master/docs/README_zh-CN.md)
- [*github issues: Failed to log in with a leetcode.com account*](https://github.com/jdneo/vscode-leetcode/issues/478)
- [*浏览器获取cookies的方法*](https://github.com/jdneo/vscode-leetcode/issues/478)

