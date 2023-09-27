---
title: 在TMUX中开启TrueColor
date: 2023-09-27T15:30:00+08:00
draft: false
tags: ["计算机", "技术"]
description: 记录在Tmux中开启真彩显示
katex: false
toc: false
summary: Tmux真彩显示
---

## 测试脚本

``` bash
awk 'BEGIN{
    s="/\\/\\/\\/\\/\\"; s=s s s s s s s s;
    for (colnum = 0; colnum<77; colnum++) {
        r = 255-(colnum*255/76);
        g = (colnum*510/76);
        b = (colnum*255/76);
        if (g>255) g = 510-g;
        printf "\033[48;2;%d;%d;%dm", r,g,b;
        printf "\033[38;2;%d;%d;%dm", 255-r,255-g,255-b;
        printf "%s\033[0m", substr(s,colnum+1,1);
    }
    printf "\n";
}'
```

## 开启TrueColor

``` .tmux.conf
set -g default-terminal		"screen-256color"
set-option -ga terminal-overrides ",*256col*:Tc"
```

``` .zshrc / .bashrc
export TERM=screen-256color
```

关掉`tmux`所有的会话，重启即可。
