---
title: XML基础
date: 2020-07-02 20:00:32
tags:
- XML
- HTML
categories:
- XML
---
在项目中，有时需要处理XML的数据，如WFS服务。

XML、HTML、XHTML，这三者长得太像了，所以按照一样的操作处理即可。

> XML，Extensible Markup Language，可扩展标记语言
<!-- more -->
## 解析

``` javascript
const parser = new DOMParser();
const xml = parser.parseFromString(response, 'text/xml');
```

## 获取节点

```javascript
const htmlCollection = xml.getElementsByTagName('title');
```

通过标签名，获取到指定标签名的所有元素的节点列表。

结果是一个htmlCollection对象，长的和数组一样，但是没有数组的许多方法。

## 遍历

for循环，遍历htmlCollection对象，通过下标拿到节点。

通过`nodeValue`或`innerHTML`来获取其中的值。

```javascript
for (let i = 0, len = htmlCollection.length; i < len; i++) {
  console.log(htmlCollection[i].innerHTML);
}
```

## 结语

解析服务数据，上面的操作就够用了。当做HTML去操作就行了，没有特别深入的去了解。