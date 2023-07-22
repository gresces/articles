---
title: "用KaTeX渲染hugo数学公式"
date: 2023-07-21T11:32:03+08:00
draft: false 
tags: ["编程", "hugo"]
description: 本篇为解决数学公式，特别是多行结构的数学公式的展示问题的纪实文章。最终的解决方法为shortcodes与python程序相结合的方式。
katex: false
summary: 本篇为解决数学公式，特别是多行结构的数学公式的展示问题的纪实文章。最终的解决方法为shortcodes与python程序相结合的方式。
---

**NOTE!!!**
**在使用时请将本文中的所有shortcode花括号`{` `}`中的空格去除，包括正文和代码。**

本篇为解决数学公式，特别是多行结构的数学公式的展示问题的纪实文章。最终的解决方法为shortcodes与自己的程序相结合的方式。

这里要着重感谢两篇文章[^1][^2]

> hugo内置的goldmark渲染器并不能识别直接识别`$[行内公式]$`和`$$[公式块]$$`语法的数学公式，所以在遇到公式时会将其中`\\、\_`之类的符号仍然按照markdown的的转义语法渲染，导致`\\`在输出中变为`\`。当用户或者主题需要渲染公式而引入mathjax或者katex等库时，会因此无法正确识别换行等语法导致公式排版混乱，或者直接报错。
>
> [来自du33169.tech](https://du33169.tech/posts/notes/hugomath/)

对于文章[^2]提供的三种方法，权衡利弊之后选择使用shortcode，鉴于公式较多的时候使用shortcode进行逐个替换较为麻烦，收到文章一[^1]的启发，选择自己编一个小程序进行自动化转换。

## shortcodes的使用

官网上有~~较为详细~~的[用法说明](https://gohugo.io/templates/shortcode-templates/)，这里直接贴上我的操作：

1. 新建文件：`\themes\<THEME>\layouts\shortcodes\<shortcodename>.html`

2. 在这个html文件中写入内容：
   ```html
   <div>{{- .Inner | safeHTML -}}</div>
   ```
3. 使用shortcodes：
   ```html
   {{ <shortcodename> }}  {{ </shortcodename> }}
   ```

## 转换程序的编写

文章一[^1]的程序的思路是将所有的出问题的`\`转换为`\\`。我这里则简单一些，可以直接识别`$$`并在其之前和之后分别放入
```
{{ <shortcodename> }}
	content
{{ </shortcodename> }}
```
这样markdown解析器就不会改变`\`，从而实现原来的latex代码正常渲染

下面是代码：

```python
# -*- coding: utf-8 -*-
import sys

def read_file(file_path):
    with open(file_path, 'r', encoding="utf-8") as file:
        content = file.read()
    return content

def replace_characters(content):
    content_list = content.split('\n')
    count = 0
    for i in range(len(content_list)):
        if content_list[i] == '$$':
            if count == 0:
                content_list[i] = '{{ <keepit> }}\n$$'
                count = 1
            else:
                content_list[i] = '$$\n{{ </keepit> }}'
                count = 0
    content = '\n'.join(content_list)
    return content

def write_file(new_content, file_path):
    with open(file_path, 'w', encoding='utf-8') as file:
        file.write(new_content)
    
file_path = sys.argv[1]
write_file(replace_characters(read_file(file_path)), file_path)

```



[^1]: [让Hugo优雅的显示公式](https://www.lbqaq.top/p/latex-in-hugo/)
[^2]: [Hugo中优雅地使用数学公式](https://du33169.tech/posts/notes/hugomath/)
