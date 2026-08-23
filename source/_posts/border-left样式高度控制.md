---
title: border-left样式高度控制
date: 2020-08-13 20:49:41
tags:
- CSS
- border
categories:
- CSS
---
有时候加个border-left，可以有分割的效果。

但是直接设置，高度顶满之后，感觉不好看。

``` css
.navbar-right .navbar-temperature {
  border-left: 1px solid #FFF;
}
```
<!-- more -->
![image-20200813180118899](https://gitee.com/GeoDaoyu/PicGo/raw/master/blog/image-20200813180118899.png)

所以通过before来插入一个元素，就可以控制高度，更好看些。

```css
.navbar-right .navbar-temperature:before {
  content: ' ';
  border-left: 1px solid #FFF;
  display: inline-block;
  height: 12px;
  position: absolute;
  left: 0;
}
```

![image-20200813180155480](https://gitee.com/GeoDaoyu/PicGo/raw/master/blog/image-20200813180155480.png)