---
title: ramda-curd
date: 2020-09-14 20:55:48
tags:
- Ramda
categories:
- Ramda
---
## 对象属性的增删改查
<!-- more -->
| 方法                                           | 说明             |
| ---------------------------------------------- | ---------------- |
| **assoc / assocPath**                          | 添加或者修改属性 |
| **dissoc / dissocPath / omit**                 | 删除属性         |
| **evolve**                                     | 修改属性         |
| **merge**                                      | 合并对象         |
| **prop / pick / has / path / propOr / pathOr** | 读取属性         |
| **keys / values**                              | 读取属性         |

## 数组的增删改查

| 方法                       | 说明                     |
| -------------------------- | ------------------------ |
| **nth / slice / contains** | 读取元素                 |
| **insert / update**        | 添加或者修改元素         |
| **append / prepend**       | 在数组头部或尾部添加元素 |
| **concat**                 | 合并数组                 |
| **remove**                 | 删除元素                 |
| **adjust**                 | 修改元素                 |

## 透镜

创建透镜：

- lensProp：创建关注对象某一属性的透镜。
- lensPath: 创建关注对象某一嵌套属性的透镜。
- lensIndex: 创建关注数组某一索引的透镜。

使用透镜：

- view：读取透镜的值。
- set：更新透镜的值。
- over：将变换函数作用于透镜。