---
title: "Objdump"
date: 2023-04-26T22:24:01+08:00
draft: false
tags: ["编程", "计算机"]
description: objdump命令的简单使用
katex: false
toc: false
summary: 以ELF格式可执行文件test为例介绍objdump命令
---

`objdump` 命令是Linux下的反汇编目标文件或者可执行文件的命令，它还有其他作用，下面以ELF格式可执行文件test为例详细介绍：

```objdump -f test```

显示test的文件头信息

```objdump -d test```

反汇编test中的需要执行指令的那些section


```objdump -D test```

与-d类似，但反汇编test中的所有section


```objdump -h test```

显示test的Section Header信息


```objdump -x test```

显示test的全部Header信息


```objdump -s test```

除了显示test的全部Header信息，还显示他们对应的十六进制文件代码