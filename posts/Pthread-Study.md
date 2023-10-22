---
title: "Pthread 笔记"
date: 2023-10-19T19:48:47+08:00
draft: false
tags: ["计算机"]
description: 使用Pthread编程，以pi的计算为例
katex: false
toc: false
summary: 使用Pthread并行编程
---

## 编写Pthread程序

### 前置知识

<!--{{{--> 

写这个部分是为了理解并行程序的编写的基本思路以及需要注意的点，所以中间显然会插入一些看起来非常基础的东西。

#### 线程

线程就是运行在进程上下文中的逻辑流，每个线程都拥有它自己的线程上下文，包括一个唯一的整数线程ID(Thread ID)、栈、栈指针、程序计数器、通用目的寄存器和条件码，一个进程中的所有线程共享进程虚拟地址空间中的所有内容，包括代码、数据、堆、共享库和打开的文件。[^1]

#### C中的变量

<!-- {{{ -->
##### static

在C语言中，静态变量是一种特殊类型的变量，其生命周期和作用域有一些特殊规则。静态变量有以下主要特点：

1. **生命周期**：静态变量在程序执行期间保持其值，不会随函数的调用而销毁。它们在程序启动时分配内存，并在程序终止时才被释放。

2. **作用域**：静态变量的作用域可以是局部的，也可以是全局的。这取决于静态变量的定义位置。

3. **初始值**：静态变量在第一次定义时初始化，如果没有明确初始化，则它们将自动初始化为零或与其类型相关的默认值。

4. **局部静态变量**：在函数内定义的静态变量是局部的，但它们在函数调用之间保持其值。这使得它们在存储状态信息方面非常有用。

5. **全局静态变量**：在函数外部定义的静态变量（通常在全局范围内）对整个程序可见，但它们的可访问性被限制在文件内，因此它们对外部文件是不可见的。

6. **访问控制**：静态变量的访问受到限制，只有定义它们的函数（局部静态变量）或文件（全局静态变量）可以直接访问它们。
<!--}}}-->

##### extern

外部变量可以被其他文件访问

#### PThread 的使用

##### 创建线程 pthread_create

<!--{{{-->
``` C
int pthread_create(
    pthread_t *thread,
    const pthread_attr_t *attr, 
    void *(*start_routine)(void *), 
    void *arg);
```

`pthread_t *thread` 用于缓存新线程的 `Thread ID`.

`const pthread_attr_t *attr` 制定新线程的 `attr` ，一般可以通过`pthread_attr_setdetachestate`来确定，如果为 `NULL`，那么将使用默认的 `JOINABLE`.

`start_routine` 是新线程的执行函数.

`arg` 向新线程传递的某些参数，一般封装为结构体传入[^2].

线程的中止可以通过以下方式：

- 调用 `pthread_exit(void *retval)`

- 直接 return

- 该线程被取消 (See pthread_cancel)

- 线程所属的进程调用了 `exit`, 或者该进程的 `main` 函数中执行了 `return`
<!--}}}-->

##### 停止线程 pthread_join

<!--{{{-->
```C
int pthread_join(pthread_t thread, void **retval);
```
`thread` 是某个线程的 `pid(Thread ID)`

`retval` 用于获取线程 `start_routine` 的返回值

在[这个网页](https://hpc-tutorials.llnl.gov/posix/joining_and_detaching/)中有关于这个函数的使用方式的说明.

如果尝试编程，就会发现这个函数中的二级指针是一个比较难以理解的变量，虽然在本例中并未使用这个参数，而是直接将其赋值为`NULL`. 

这里给出一个例子：

```C
typedef struct { double value; } result_t;
void *worker(void *arg)
{
    result_t *res = (result_t *)malloc(sizeof(result_t));
    /**
     * Program lines
     */
    return res;
}

int main ()
{
    /**
     * Program lines
     */
    
    result_t *res = NULL;
    pthread_join(tid, (void **)&res);
    
    /**
     * Program lines
     */
}
```

这是二级指针传递返回值的一种形式，如果必须通过传递参数与 `join` 获取返回值实现，使用二级指针是无法避免的.
<!--}}}-->

##### 互斥锁 pthread_mutex_t

<!--{{{-->
这里有一对函数：`pthread_mutex_lock` 使线程等待，直到没有其他线程进入临界区，调用`pthread_mutex_unlock` 则通知系统该线程已经完成了临界区中代码的执行. 

所以可以在临界区之前使用`lock`，之后使用`unlock`.

```C
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
// 初始化

int pthread_mutex_lock(pthread_mutex_t∗  mutex_p);
sum = sum + local_ret;
int pthread_mutex_unlock(pthread_mutex_t∗  mutex_p);
```
<!--}}}-->

<!--}}}-->

### 程序内容

<!--{{{-->

```C {linenos=table}
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <time.h>

int steps_num = 1000000000;
int thread_num;
double step;
double sum = 0.0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void* pi_thread(void * pArg) {
	long id = (long)pArg;
	double ret = 0.0;

	for (long i = (steps_num/thread_num) * id; i < (steps_num/thread_num) * (id + 1); i++) {
		double temp = 4.0/(1 + ((i + 0.5)*step)*((i + 0.5)*step));
		ret += temp;
	}
	printf("%.12f \n" , ret * step);
	pthread_mutex_lock(&mutex);
	sum += ret;
	pthread_mutex_unlock(&mutex);
	return NULL;
}

int main(int argc, char* argv[])
{

	clock_t start, finish;
	start = clock();
	thread_num = atoi(argv[1]);
	step = 1./(double)steps_num;
	pthread_t* thread_handles;

	thread_handles = (pthread_t*)malloc(thread_num * sizeof(pthread_t));
	long i;
	for (i = 0; i < thread_num; i++) {
		int cerr = pthread_create(&thread_handles[i], NULL, pi_thread, (void *)i);
		if (cerr != 0) {
			printf("ERROR: Creat thread");
			exit(1);
		}
	}

	for (i = 0; i < thread_num; i++) {
		int jerr = pthread_join(thread_handles[i], NULL);
		if (jerr != 0) {
			printf("ERROR: Join thread");
		}
	}

	finish = clock();
	printf("————>%15.12f\n in %.3f", sum * step, (double)(finish-start)/CLOCKS_PER_SEC);

	return 0;
}
```

<!--}}}-->

[^1]: 《深入理解计算机系统》 Randal E. Bryant
[^2]: [ pthread 多线程基础 by sinkinben](https://www.cnblogs.com/sinkinben/p/13987001.html)
