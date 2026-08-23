---
title: CSS使用外部字体
date: 2020-06-28 20:05:58
tags:
- CSS
- font
categories:
- CSS
---

UI提供的字体，浏览器并没有，需要手动引入。

以普惠体为例，引入方式如下：
<!-- more -->
1. 下载`.ttf`文件，然后在全局的CSS中用路径引入。

      ```css
      @font-face {
        font-family: AlibabaPuHuiTi;
        src: url("./fonts/Alibaba-PuHuiTi.ttf");
      }
      ```

   > webpack打包部署，可能需要考虑路径指向。

2. 使用示例

      ```css
      html,body {
        font-family: AlibabaPuHuiTi;
      }
      ```