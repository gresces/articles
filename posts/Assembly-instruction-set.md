---
title: "一些汇编指令"
date: 2023-04-15T22:28:46+08:00
draft: false
tags: ["计算机", "汇编"]
description: 计算机系统拆炸弹的一些汇编指令
katex: true
toc: true
summary: jump, shl(r), and命令
---

## BOOM指令集
### Test的使用

```text-plain
test %eax %eax
```

test 一个相同的操作数，判断内容是否为0，test的实质为 \\(S\_1 \\& S\_2\\)。

### 跳转指令

|     |     |
| --- | --- |
| 指令  | 条件  |
| jg  | 大于  |
| jne | 不相等 |
| ja  | 超过  |
| js  | 为负数 |

### 逻辑位移 [算术右移与逻辑右移](#root/53h7aI8mjR1z/OVuLxIweXKWt/rgcdmsOt7a40)

|     |     |
| --- | --- |
| 指令  | 效果  |
| shl %al 2 | 逻辑左移 |
| shr %al 2 | 逻辑右移 |

### 算术位移

|     |     |
| --- | --- |
| 指令  | 效果  |
| sal %al 2 | 逻辑左移 |
| sar %al 2 | 逻辑右移 |

### 逻辑运算

|     |     |
| --- | --- |
| 指令  | 效果  |
| and | 按位与运算 |
|     |     |