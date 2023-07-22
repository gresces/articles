---
title: "Runge-Kutta法解一阶微分方程组"
date: 2023-07-20T11:12:57+08:00
draft: false
tags: ["数学", "编程"]
description: Runge-Kutta法解一阶微分方程组
katex: true
summary: |
    一阶常微分方程初值问题的RT解法
---

## 理论基础

### Runge-Kutta法

{{<keepit>}}
$$
\left\{
\begin{aligned}
    y' = f(x,y) \\
    y(x_0) = \alpha
\end{aligned}
\right.
$$
{{</keepit>}}

Runge-Kutta方法是采用不同点上函数值的不同组合来提高函数精度的方法，同时又避免了函数偏导数的计算。 R-K方法的一般形式为 

{{<keepit>}}
$$
y_{n+1} = y_{n}	+ h\varphi(x_n,y_n;h)
$$
{{</keepit>}}

其中

{{<keepit>}}
$$
\textstyle\varphi(x_n,y_n;h) = \sum^s_{i=1}b_ik_i \\
\textstyle k_i = f(x_n+c_ih, y_n + h \sum^s_{j=1}a_{ij}k_j)
$$
{{</keepit>}}

经典四阶R-K方法(RK4)的具体计算格式为

{{<keepit>}}
$$
\left\{\begin{matrix}
y_{n+1} = y_n + \frac{1}{6}h(k_1 + 2K_2 + 2K_3 + k_4),\\
k_1 = f(x_n, y_n),\\
k_2 = f(x_n + \frac12h, y_n + \frac12hk_1),\\
k_3 = f(x_n + \frac{h}2h, y_n + \frac12hk_2),\\
k_4 = f(x_n + h, y_n + hk_3).
\end{matrix}\right.
$$
{{</keepit>}}

### 一阶常微分方程组初值问题的RT解法

考虑一阶常微分方程组

{{<keepit>}}
$$
\left\{\begin{matrix}
 \frac{\mathrm{d}y_1}{\mathrm{d}x} = f_1(x,y_1,y_2,\cdots,y_m),\\
 \frac{\mathrm{d}y_2}{\mathrm{d}x} = f_2(x,y_1,y_2,\cdots,y_m),\\
 \cdots \\
 \frac{\mathrm{d}y_m}{\mathrm{d}x} = f_m(x,y_1,y_2,\cdots,y_m)
\end{matrix}\right.
$$
{{</keepit>}}

$x\in (x_0, b]$，且具有如下初始条件

{{<keepit>}}
$$
y_i(x_0)= \alpha_i, i = 1,2,\cdots,m
$$
{{</keepit>}}

则可以写成向量形式

{{<keepit>}}
$$
\mathbf{f} = (y_1,y_2,\cdots,y_m)^T, \mathbf{f}(x,\mathbf{y}) = (y_1(x,\mathbf{y}),y_2(x,\mathbf{y}),\cdots,y_m(x,\mathbf{y}))^T
$$
{{</keepit>}}

初值问题可以写成

{{<keepit>}}
$$
\left\{\begin{matrix}
\frac{\mathrm{d}\mathbf{y}}{\mathrm{d}x} = \mathrm{f}(x,\mathbf{y}), x\in(x_0,b]\\
\mathbf{y}(x_0) = \mathbf{\alpha}
\end{matrix}\right.
$$
{{</keepit>}}

经典Runge-Kutta方法公式如下：

{{<keepit>}}
$$
\left\{\begin{aligned}
&\mathbf{y}_{n+1} = \mathbf{y}_n + \frac{1}{6}h(\mathbf{K}_1 + 2\mathbf{K}_2 + 2\mathbf{K}_3 + \mathbf{K}_4),\\
&\mathbf{K}_1 = \mathbf{f}(x_n, \mathbf{y}_n),\\
&\mathbf{K}_2 = \mathbf{f}(x_n + \frac12h, \mathbf{y}_n + \frac12h\mathbf{K}_1),\\
&\mathbf{K}_3 = \mathbf{f}(x_n + \frac{h}2h, \mathbf{y}_n + \frac12h\mathbf{K}_2),\\
&\mathbf{K}_4 = \mathbf{f}(x_n + h, \mathbf{y}_n + h\mathbf{K}_3).
\end{aligned}\right.
$$
{{</keepit>}}


## python 实践

首先安装三个python库

```bash
pip install numpy
pip install matplotlib
pip install pandas
```

代码
```python {linenos=table}
## Author: GuoYangfan
## Date: 2023.7.13
## Function: Calculate the value based on the input equation and output figures
## 所有的#注释是用于模拟物质与时间的关系的代码

import matplotlib.pyplot as plt
import numpy as np 
import re

## Function inputODEs
## Input 微分方程个数numOfODEs， 底物浓度s
## Output 问题表述的列表
## list：[方程式(list_string),
##        初值(list_int)，
##        反应时间开始时间(int or float)，
##        方程式个数(int)]
def inputODEs(numOfODEs = 1, s = 0):
    odes = ['string'
            , '-1*y[1]*y[2]+1.05*y[3]'
            , '-1*y[1]*y[2]+y[3]'
            , 'y[1]*y[2]-1.05*y[3]'
            , '0.05*y[3]']
    init = [0.0]
    # x = 1
    # while x <= numOfODEs:
    #     odes.append(input("dy" + str(x) + "/dt: "))
    #     x = x + 1
    # print(odes)

    # x = 1
    # while x <= numOfODEs:
    #     init.append(eval(input("y_" + str(x) + "0: ")))
    #     x = x + 1
    # print(init)
    # x0 = eval(input("x0: ")) S-P

    init = [0.0, 0.5, s, 0, 0]
    ivfodes = [odes, init, 0, numOfODEs]
    return ivfodes

## Function  rungeKutta4
## Input 问题表述的列表(list)
## Output 所有参数的值的列表(list)
def rungeKutta4(ivfodes, h, max):
    y0 = np.asarray(ivfodes[1]) # Y初值
    x0 = ivfodes[2] #x0初值
    k1 = np.empty(ivfodes[3]+1,dtype=float)
    k2 = np.empty(ivfodes[3]+1,dtype=float)
    k3 = np.empty(ivfodes[3]+1,dtype=float)
    k4 = np.empty(ivfodes[3]+1,dtype=float)
    npode = np.asarray(ivfodes[0])

    xPlot = list()
    yPlot = list()
    yPlot1 = list()
    yPlot2 = list()
    yPlot3 = list()
    yPlot4 = list()

    # visNum = eval(input("observed variable: "))

    while x0 < max:
        y, x = y0, x0
        k1 = k1 + 0.0
        i = 1
        while i <= ivfodes[3]:
            k1[i] = eval(str(npode[i]))
            i = i+1

        y, x = y+0.5*h*k1, x+0.5*h
        i = 1
        while i <= ivfodes[3]:
            k2[i] = eval(npode[i])
            i = i+1

        y, x = y0+0.5*h*k2, x0+0.5*h
        i = 1
        while i <= ivfodes[3]:
            k3[i] = eval(npode[i])
            i = i+1

        y, x = y0+h*k3, x0+h
        i = 1
        while i <= ivfodes[3]:
            k4[i] = eval(npode[i])
            i = i+1

        y0 = y0 + h*(k1+2*k2+2*k3+k4)/6.0
        # print("x: ", x0)
        # print("y: ", y0[visNum])
        xPlot.append(x0)
        # yPlot.append(y0[visNum])
        yPlot1.append(y0[1])
        yPlot2.append(y0[2])
        yPlot3.append(y0[3])
        yPlot4.append(y0[4])
        
        x0 = x0+h

## 以下用于绘制时间相关图像
    # plt.plot(xPlot
    # , yPlot1
    # , yPlot2
    # , yPlot3
    # , yPlot4)
    # plt.savefig(input("Save Fig: "))
    # plt.show()

    return [xPlot, yPlot1, yPlot2, yPlot3, yPlot4]

## Function spDiff 用于模拟米氏函数
## Input 浓度起始点
##       步长 
##       浓度终点
## Output 图片
def spDiff(stpoint, x, edpoint):
    rep = list()
    res = list()
    while stpoint <= edpoint:
        result = rungeKutta4(inputODEs(4, stpoint), 0.01, 0.05) 
        pv = np.gradient(result[4], result[0])
        rep.append(pv.mean(axis=0))
        res.append(stpoint)
        stpoint = stpoint + x
    plt.plot(res, rep)
    plt.savefig(input("Save Fig: "))


## main program
## 绘制时间相关图像
# rungeKutta4(inputODEs(eval(input("Virable Nums: "))),
#  eval(input("Step: ")), eval(input("Time:")))

## 反应速度与浓度的关系
spDiff(0.1, 0.1, 200)
```

***
Ref.

[zhihu.com一阶常微分方程（组）及二阶常微分方程边值问题数值解法](https://zhuanlan.zhihu.com/p/557967675)
[^1]: [zhihu.com一阶常微分方程（组）及二阶常微分方程边值问题数值解法](https://zhuanlan.zhihu.com/p/557967675)
