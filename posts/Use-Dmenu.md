---
title: 如何使用dmenu
date: 2023-11-29T17:09:57+08:00
draft: false
tags: ["工具"]
description: 一些使用Dmenu的技巧
katex: false
toc: true
summary: Dmenu的安装及使用
---

`dmenu`常常用来作为一个应用菜单来使用[^1]，但是它不止于此。它的实际功能是一个附加了选择功能的图形化选择器，以一个字符串列表作为输入，之后将选择出的字符输出。

## 安装

Debian/Ubuntu

``` bash
# apt-get install suckless-tools
```

Archlinux

``` bash
# pacman -S dmenu
```

## 使用

### 自定义显示与执行

`dmenu`会读取`stdin`中一系列的使用换行分开的元组，可以使用管道符`(|)`或者`<`

例子：

``` bash
$ echo -e "aaa\nbbb\nccc" | dmenu
```

在显示出来的菜单（屏幕顶部）左边是一个搜索栏，输入字符可以从三个字符串中搜索。
选中的那个将会在终端中显示出来，如果将`dmenu`的输出使用管道符传入另一个命令就可以进行执行操作。

例如

``` bash
$ echo -e "ls\nps\n" | dmenu | ${SHELL:-"/bin/sh"}

$ echo -e "ls -al" | dmenu | ${SHELL:-"/bin/sh"}
```

### 显示位置

``` bash
$ ps -e | dmenu -l 10
```

### 显示字体

``` bash
$ ... | dmenu -fn "Noto Sans Mono"
```

### 显示颜色

可以使用`HTML`颜色码，`-[n,s][b,f]`分别为`[普通，选中][背景色，前景色]`.

``` bash
$ ... | dmenu -nb "#EBCB8B" -nf "#2E3440" -sb "#2E3440" -sf "#EBCB8B"
```

### 多显示器

显示器编号从`0`开始

``` bash
$ ... | dmenu -m 0
```

### 提示语

``` bash
$ ... | dmenu -p "Prompt"
```

# More...

### 补丁

作为`suckless.org`的软件，可以去[官网](https://tools.suckless.org/dmenu/patches/)找补丁

### 自定义

**程序绞肉机**

``` bash
$ ps -e -o comm | dmenu -l 20 | xargs kill -9
```

可以替代`top`系列的打开。

### 替代AppImageLauncher

``` bash
ls ~/AppImage | xargs -n 1 | awk '{print "~/AppImage/" $0}' >> ~/.cache/dmenu_run
```

假设`AppImage`目录位于`~/AppImage`，则可以将上面的指令写到开机命令执行列表中。

例如在`i3`中可以先执行`dmenu_path`，再执行上面的命令。

[^1]:严谨地说是`dmenu_run`，这是一个脚本，将`dmenu_path`传出的数据送给`dmenu`显示，之后将选出来的软件送给`sh`执行.
