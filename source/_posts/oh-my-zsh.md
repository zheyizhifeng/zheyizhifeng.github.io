---
title: oh my zsh!
date: 2020-10-11 21:39:31
tags: terminal
---
# 查看已有终端
```bash
cat /etc/shells

# /bin/bash
# /bin/csh
# /bin/dash
# /bin/ksh
# /bin/sh
# /bin/tcsh
# /bin/zsh
```

# 查看当前正在使用的终端
```bash
echo $SHELL

# /bin/bash
```

# 安装 zsh 终端
```bash
# MacOS
brew install zsh

# Ubuntu, Debian, Windows 10 WSL
apt install zsh

# CentOS
yum -y install zsh
```

# 切换终端为 zsh
```bash
chsh -s $(which zsh) $(whoami)
```

# 安装 oh-my-zsh

```zsh
# curl 方式安装
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# wget 方式安装
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

```

安装完成以后效果如下

![Installing Oh-My-Zsh on Debian-Based Systems](https://www.hildeberto.com/images/posts/oh-my-zsh.png)

# 修改主题以及配置

**oh-my-zsh** 的配置文件位于 `~/.zshrc`，可以进行自定义的修改配置。

![~/.zshrc配置文件](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20201011231204276.png)

## 主题配置

**ZSH_THEME** 字段为主题字段，可选的主题参考 `~/.oh-my-zsh/themes/` 目录。

## 插件配置

本文推荐 **3** 个较为常用的 `oh-my-zsh` 插件

### 插件简介

- **zsh-autosuggestions**

 输入命令时可提示自动补全

![提示自动补全](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20201011235101847.png)

- **zsh-syntax-highlighting** 

常用的命令会高亮显示，命令错误显示红色

![高亮提示](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20201011235321240.png)

- **git**

插件默认启用，支持各种 **git** 命令简写, 参考 `~/.oh-my-zsh/plugins/git/git.plugin.zsh`
  ![常见git命令简写](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20201011233927923.png)

### 安装插件

```zsh
# 安装 zsh-autosuggestions
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
# 安装 zsh-syntax-highlighting
git clone git://github.com/zsh-users/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting

# 安装后的插件位于~/.oh-my-zsh/custom/plugins/目录
```

安装插件以后还需要更新 `~/.zshrc` 配置文件

![image-20201011232535934](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/image-20201011232535934.png)



# 保存配置后进行更新

```zsh
source ~/.zshrc
```

下图为 **agnoster** 主题的效果

![agnoster-zsh-theme](https://cdn.jsdelivr.net/gh/zheyizhifeng/picture_repo/images/20201011222823.png)

 **注意**：在使用 **agnoster** 主题时，**部分符号在终端无法正常显示**，还需**安装** [Powerline fonts](https://github.com/powerline/fonts) **字体**

```zsh
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```
