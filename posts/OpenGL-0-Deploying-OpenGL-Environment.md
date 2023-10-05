---
title: "Archlinux部署OpenGL环境"
date: 2023-10-04T19:22:53+08:00
draft: false
tags: ["计算机", "OpenGl"]
description: 学习OpenGL：0搭建环境
katex: false
toc: false
summary: 搭建OpenGL环境
---

## Archlinux 配置 OpenGL 开发环境

### GLFW

![](/images/OpenGL-0/1-Install-GLFW.png)

### GLAD

在网站 [glad.dav1d.de](https://glad.dav1d.de)中选择适合自己的选择，之后点击最下方的按钮 `GENERATE`.

![](/images/OpenGL-0/2-Select-GLAD.png)

之后会弹出一个下载页面，在这个页面中，点击 `glad.zip` 即可下载自动生成的文件。

解压之后的文件目录树如下：

``` bash
→ tree
.
├── include
│   ├── glad
│   │   └── glad.h
│   └── KHR
│       └── khrplatform.h
└── src
    └── glad.c

5 directories, 3 files
```

将`GLAD`文件放入工程文件夹之后进行整理，得到的目录树如下：

``` bash
→ tree
.
├── include
│   ├── glad
│   │   └── glad.h
│   └── KHR
│       └── khrplatform.h
├── Makefile
├── obj
└── src
    ├── glad.c
    └── main.c

6 directories, 5 files
```

并将 `src/glad.c`中的头文件引用修改为正确的目录

``` C 
// #include <glad/glad.h>
#include "../include/glad/glad.h"
```
基本的环境配置到这里就结束了，编辑器可以直接使用`vim`，不用进行额外配置。`Linux`的`OpenGL`配置比大多数教程要简单得多，主要是由于相关的库已经封装好并存在于镜像站点，使用时直接安装即可。

## 测试

下面复制一下测试文件进行测试，查看环境是否正确配置。

``` C
#include <stdio.h>
#include "include/glad/glad.h"
#include <GLFW/glfw3.h>
#include <GLFW/glfw3native.h>

// settings
const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

int main()
{
	glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

	GLFWwindow* window = glfwCreateWindow(SRC_WIDTH, SRC_HEIGHT, "LearnOpenGL", NULL, NULL);
	if (window == NULL)
	{
	    printf("Failed to create GLFW window");
	    glfwTerminate();
	    return -1;
	}
	glfwMakeContextCurrent(window);
	if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
	{
		printf("Failed to initialize GLAD");
	    return -1;
	}
	while (!glfwWindowShouldClose(window))
    {
        // render
        // ------
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // glfw: swap buffers and poll IO events (keys pressed/released, mouse moved etc.)
        // -------------------------------------------------------------------------------
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // glfw: terminate, clearing all previously allocated GLFW resources.
    // ------------------------------------------------------------------
    glfwTerminate();
	return 0;
}
```

输入编译指令

```bash
gcc -lglfw -o main main.c src/glad.c
```

可以得到一个可执行文件，运行可以看到窗口。

![](/images/OpenGL-0/4-Test-OpenGL.png)

至此，OpenGL的开发环境配置结束。

## 编译优化

为这个工程写一个简单的 `Makefile`：

 ```Makefile 
 TARGET 	= OpenGL_Learn_0
PROJDIR = .
IDIR 	= $(PROJDIR)/include \
		  $(PROJDIR)/include/glad \
		  $(PROJDIR)/include/KHR
SDIR 	= $(PROJDIR)/src
CC 		= gcc -g -lglfw
CFLAGS 	= $(foreach in,$(IDIR),-I$(in))

ODIR 	= ./obj

DEPS 	= $(foreach n ,$(wildcard $(IDIR)),$(n)/*.h)

_OBJS 	= $(patsubst %.c, %.o, $(wildcard $(SDIR)/*.c))
OBJS 	= $(patsubst $(SDIR)/%,$(ODIR)/%,$(_OBJS))

$(ODIR)/%.o: $(SDIR)/%.c
	$(CC) -c -o $@ $< $(CFLAGS)

$(TARGET): $(OBJS)
	$(CC) -o $@ $^ $(CFLAGS)

.PHONT: clean

clean:
	rm -f $(ODIR)/*.o $(PROJDIR)/$(TARGET)
 ```

## 参考

[^1]:[Linux 安装glfw3及glad的C语言库](https://poemdear.com/2019/12/03/linux-%E5%AE%89%E8%A3%85glfw3%E5%8F%8Aglad%E7%9A%84c%E8%AF%AD%E8%A8%80%E5%BA%93/)
[^2]:[OpenGL开发环境搭建](https://dionysen.github.io/2023/06/09/note/Programming/evn/OpenGL-env/)
[^3]:[LearnOpenGL CN 你好，窗口](https://learnopengl-cn.github.io/01%20Getting%20started/03%20Hello%20Window/)