---
title: css多背景拼接
date: 2023-09-25 17:26:40
tags:
- CSS
- background
categories:
- CSS
---

多背景，要求上面 400px 是渐变色，下面全部是纯色。

```css
body {
  background-image: linear-gradient(
      180deg,
      #8392bb 0%,
      rgba(232, 234, 241, 0) 100%
    ), linear-gradient(180deg, #e8eaf1 0%, #e8eaf1 100%);
  background-size: 100% 400px, 100%;
  background-repeat: no-repeat, repeat-y;
}
```
