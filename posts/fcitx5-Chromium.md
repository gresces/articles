---
title: "Fcitx5在Chromium中无法呼出"
date: 2023-07-31T22:55:53+08:00
draft: false
tags: ["Linux", "计算机"]
description: 解决Chromium内核浏览器中文输入问题
katex: false
toc: false
summary: |
    在.xinitrc文件中加入'dbus-launch --sh-syntax --exit-with-session > /dev/null'即可
---

在配置Archlinux的过程中，下载了以Chromium为内核的Vivaldi浏览器，但是在其中无法输入中文，具体问题为无法使用`Ctrl+Space`改变fcitx5的输入法，并且无法呼出输入选词框。

这里的环境为X11，且显示管理器为[ly](https://github.com/nullgemm/ly).

在fcitx的[Wiki页面](https://fcitx-im.org/wiki/FAQ/zh-hans#Chromium.E6.88.96.E8.80.85.E4.BB.BB.E4.BD.95.E5.9F.BA.E4.BA.8Echromium.E7.9A.84.E6.B5.8F.E8.A7.88.E5.99.A8.EF.BC.88.E4.BE.8B.E5.A6.82.EF.BC.8CMicrosoft_Edge.EF.BC.89)中有关于这个问题的解释。

解决办法参照这个[页面](https://fcitx-im.org/wiki/Configure_%28Other%29#Use_Slim_.28.7E.2F.xinitrc.29.2Fstartx)

在配置文件`~/.xinitrc`中加入如下一行

```shell 
dbus-launch --sh-syntax --exit-with-session > /dev/null
```

之后重启(reboot)即可。
