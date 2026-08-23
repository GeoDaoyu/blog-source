---
title: GeoJSONLayer不能被hitTest选中
date: 2025-02-10 10:25:34
tags:
  - GeoJSONLayer
  - hitTest
categories:
  - ArcGIS JS API
---

## 问题

在三维下，使用GeoJSONLayer加载的GeoJSON文件，不能被hitTest选中。

## 调试

写测试页面调试，发现是GeoJSON数据中带有z值, 且均为0。

## 解决方案

法一：数据上处理，把z值都过滤掉。

法二：设置GeoJSONLayer的`elevationInfo.mode`属性为`on-the-ground`。

法三：设置GeoJSONLayer的`hasZ`属性为`false`。
