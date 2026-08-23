---
title: 修改hr标签样式
date: 2020-06-28 20:08:57
tags:
- CSS
- HTML
categories:
- CSS
---

一段来自的w3school介绍：

> `<hr>`标签在 HTML 页面中创建一条水平线。
>
> 水平分隔线（horizontal rule）可以在*视觉上*将文档分隔成各个部分。

`<hr>`标签的表现，像一个内容为空的div。

那么通过设置border的样式，则可以修改hr的样式。
<!-- more -->
例如：

```css
hr {
  border: none;
  border-bottom: 1px solid orange;
}
```

