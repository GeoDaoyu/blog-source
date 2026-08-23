---
title: iframe滚动条
date: 2024-04-17 10:21:23
tags:
- CSS
- iframe
categories:
- CSS
---
系统中嵌入了iframe，设置了100%高度，但是iframe中还是出现了滚动条。

查询资料发现，iframe是内联元素，默认与baseline对齐。在iframe后有一个行内空白节点，空白节点始终占据着高度，导致外部容器被撑开，出现滚动条。

解决方法：设置iframe的vertical-align:top;
