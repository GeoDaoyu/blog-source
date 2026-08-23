---
title: CSS处理文字溢出
date: 2020-07-09 19:28:12
tags:
- CSS
- ellipsis
categories:
- CSS
---
有时，文字很长，从展示的地方溢出，导致UI上看着很乱。

这时候，通过CSS控制文字的宽度，把溢出的文字处理为缩略号，保证了UI上的美观。

下面列两种CSS处理文字溢出的方式，分别针对单行文字和多行文字。

## 单行文字

```css
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```
<!-- more -->
参数说明：

+ `text-overflow`— 规定当文本溢出包含元素时发生的事情.
+ `text-overflow: ellipsis`— 显示省略符号来代表被修剪的文本.
+ `white-space`— 设置如何处理元素内的空白 .
+ `white-space: nowrap`文本不会换行，文本会在在同一行上继续，直到遇到 `<br>` 标签为止.

## 多行文字

```css
display: -webkit-box;
-webkit-line-clamp: 2;
-webkit-box-orient: vertical;
overflow: hidden;
```

参数说明：

+ `-webkit-line-clamp`— webkit的私有属性，用来限制在一个块元素显示的文本的行数.
+ `-webkit-box-orient`— 设置或检索伸缩盒对象的子元素的排列方式 .
+ `display: -webkit-box`— 将对象作为弹性伸缩盒子模型显示.

