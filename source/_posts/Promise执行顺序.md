---
title: Promise执行顺序
date: 2020-09-23 19:43:35
tags:
- Promise
- ES6+
categories:
- JavaScript
---
Promise的执行顺序会设计到JavaScript的事件循环。

找到一张图，来理解：

![eventloop](https://gitee.com/GeoDaoyu/PicGo/raw/master/blog/eventloop-1600651261570.png)
<!-- more -->
1. 宏队列: 用来保存待执行的宏任务(回调), 比如: 定时器回调/DOM事件回调/ajax回调
2. 微队列: 用来保存待执行的微任务(回调), 比如: promise的回调/MutationObserver的回调
3. JS执行时会区别这2个队列
    JS引擎首先必须先执行所有的初始化同步任务代码
    每次准备取出第一个宏任务执行前, 都要将所有的微任务一个一个取出来执行

下面列出一道试题：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>Promise执行顺序</title>
</head>

<body>
  <script>
    /*
    同步 
    宏: []
    微: []
    */
    setTimeout(() => {
      console.log("0")
    }, 0)
    new Promise((resolve, reject) => {
      console.log("1")
      resolve()
    }).then(() => {
      console.log("2")
      new Promise((resolve, reject) => {
        console.log("3")
        resolve()
      }).then(() => {
        console.log("4")
      }).then(() => {
        console.log("5")
      })
    }).then(() => {
      console.log("6")
    })

    new Promise((resolve, reject) => {
      console.log("7")
      resolve()
    }).then(() => {
      console.log("8")
    })
  </script>
</body>

</html>
```

判断思路：

先判断同步的代码，然后把异步的任务push到数组中。

同步代码的打印顺序就是真实的打印顺序。

当同步代码执行完毕后，执行宏任务，执行宏任务时，要先处理所有的微任务（也就是要保证微任务的数组是空的）。

在执行宏任务或微任务时，如果遇到新的异步任务，则push到对应的数组中。在数组中的任务是还未执行的，到移动到同步的数组中时，才是真正执行的时候。

过程：

```javascript
// 1
// 同步: [1, 7]
// 宏: [0]
// 微: [2, 8]
// 2
// 同步: [1, 7, 2, 3]
// 宏: [0]
// 微: [8, 4, 6]
// 3
// 同步: [1, 7, 2, 3, 8]
// 宏: [0]
// 微: [4, 6]
// 4
// 同步: [1, 7, 2, 3, 8, 4]
// 宏: [0]
// 微: [6, 5]
// 5
// 同步: [1, 7, 2, 3, 8, 4, 6, 5]
// 宏: [0]
// 微: []
// 6
// 同步: [1, 7, 2, 3, 8, 4, 6, 5, 0]
// 宏: []
// 微: []
```
