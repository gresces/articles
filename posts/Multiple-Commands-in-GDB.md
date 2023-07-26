---
title: "GDB如何同时运行多条命令"
date: 2023-04-02T22:09:00+08:00
draft: false
tags: ["编程", "计算机"]
description: 在GDB中自定义多行命令
katex: false
toc: false
summary: 使用GDB中的编辑自定义函数功能，以define唤起，以end结尾
---

使用GDB中的编辑自定义函数功能

以define唤起，以end结尾

define function_name

```gdb
(gdb)define xni
>x/10i $rip
>ni
>end
```
