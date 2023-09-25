---
title: 在终端中使用代理加速
date: 2023-09-25T21:41:01+08:00
draft: false
tags: ["计算机"]
description: 计算机使用技巧
katex: false
toc: false
summary: 设置终端中的文件，添加终端命令
---

## 前言

之前在`ArchLinux`上鼓捣好了代理，但是发现在终端中不走代理，即使设置了系统代理。

查了一些帖子，最后用了[这篇](https://weilining.github.io/294.html)，并修改一些错误。

## 使用方法

在`.bashrc`或`.zshrc`中加入如下片段。

``` bash
function proxy_on() {
    export http_proxy=http://127.0.0.1:1089
	export https_proxy=http://127.0.0.1:1089
    echo -e "Proxy OPEN"
}

function proxy_off(){
    unset http_proxy https_proxy
    echo -e "Proxy CLOSE"
}
```

其中1089是自己的代理端口。

参考的帖子一开始是不能用的，`export
https_proxy=\$http_proxy`这句应该改为上面片段中的语句，否则无法代理。

