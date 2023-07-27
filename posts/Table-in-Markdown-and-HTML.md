---
title: "Table in Markdown and HTML"
date: 2023-07-27T17:01:04+08:00
draft: false
tags: ["计算机"]
description: 配置本站table样式
katex: false
toc: true
summary: 配置本站table样式
---

## Markdown中的表格[^1]

```markdown
| 左对齐 | 居中对齐 | 右对齐 |
| :-----| :----: | ----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |
```

展现为

| 左对齐 | 居中对齐 | 右对齐 |
| :-----| :----: | ----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |

## CSS中的表格设置

1. 设置表格总体样式

添加上下两条实线，设置表格宽度并居中
```css
table {
    width: 80%;
    margin: 10px auto;
    border-top: #303030 solid 3px;
    border-bottom: #303030 solid 3px;
}
```

2. 设置单元格格式

```css
td {
    padding: 6px;
}

th {
    padding: 6px;
    background-color: #f2f2f2;
    border-bottom: #303030 solid 1.5px !important;
}
```

3. 设置动态样式
```css
tr:hover {
    background-color: #e2e2e2;
}
```

[^1]:[Markdown 表格 RUNOOB.COM](https://www.runoob.com/markdown/md-table.html)