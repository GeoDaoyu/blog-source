---
title: 手写Promise
date: 2020-09-18 23:08:04
tags:
- Promise
- ES6+
categories:
- JavaScript
---
Promise是一个异步编程的解决方案。

之前的方案是回调函数的方式。优点是指定回调函数的位置更灵活，和解决了回调地狱问题。

规范是Promise A+。

---

下面手写实现一遍Promise便于理解。
<!-- more -->
### 总体结构

```javascript
(function (window) {
  // 三个状态常量
  const PENDING = 'pending', // 等待
    FULFILLED = 'fulfilled', // 成功
    REJECTED = 'rejected';   // 失败
  /**
   * Promise 类
   * @param {Function} excutor 执行器
   */
  function Promise(excutor) {
    // 初始化状态
    this.state = PENDING;
    // 存储数据
    this.data = null;
    // 存储.then的回调函数 {onFulfilled, onRejected}
    // 可能多次.then，所以是一个数据
    this.callbacks = [];

    /**
     * resolve 成功函数
     * @param {*} value 成功的数据
     */
    function resolve(value) {}

    /**
     * reject 失败函数
     * @param {*} reason 失败的原因
     */
    function reject(reason) {}
    try {
      excutor(resolve, reject); // 立即执行 执行器函数
    } catch (error) {
      reject(error); // 异常就直接抛出
    }
  }

  /**
   * 原型对象上的then方法
   * @param {Function} onFulfilled 获取成功的回调函数
   * @param {Function} onRejected 获取失败的回调函数
   * @returns {Promise} 新的Promise对象
   */
  Promise.prototype.then = function (onFulfilled, onRejected) {}

  /**
   * 原型对象上的catch方法
   * @param {Function} onRejected 获取失败的回调函数
   * @returns {Promise} 新的Promise对象
   */
  Promise.prototype.catch = function (onRejected) {}

  /**
   * Promise函数对象的resolve方法
   * @param {*} value 
   */
  Promise.resolve = function (value) {}

  /**
   * Promise函数对象的reject方法
   * @param {*} reason 
   */
  Promise.reject = function (reason) {}

  /**
   * Promise函数对象的all方法
   * 所有成功才成功，只要一个失败则失败
   */
  Promise.all = function (promises) {}

  /**
   * Promise函数对象的race方法
   * 第一个结束的 成功则成功 失败则失败
   */
  Promise.race = function (promises) {};

  window.Promise = Promise;
})(window); // IIEF
```

### 执行器的resolve

``` javascript
/**
 * resolve 成功函数
 * @param {*} value 成功的数据
 */
resolve = value => {
  // 检查状态，保证状态只能改变一次
  if (this.state !== PENDING) {
    return;
  }
  // 改状态
  this.state = FULFILLED;
  // 存数据
  this.data = value;
  // 检查回调，如果有，立即执行回调函数
  if (this.callbacks.length > 0) {
    setTimeout(() => {
      this.callbacks.forEach(callbackObj => {
        callbackObj.onFulfilled(this.data);
      });
    });
  }
}
```

### 执行器的reject,

复制粘贴resolve，改状态、名字即可。

``` javascript
/**
 * reject 失败函数
 * @param {*} reason 失败的原因
 */
reject = reason => {
  // 查状态，保证状态只能改变一次
  if (this.state !== PENDING) {
    return;
  }
  // 改状态
  this.state = REJECTED;
  // 存数据
  this.data = reason;
  // 检查回调，如果有，立即执行回调函数
  if (this.callbacks.length > 0) {
    setTimeout(() => {
      this.callbacks.forEach(callbackObj => {
        callbackObj.onRejected(this.data);
      });
    });
  }
}
```

### 原型对象上的then 方法

```javascript
/**
 * 原型对象上的then方法
 * @param {Function} onFulfilled 获取成功的回调函数
 * @param {Function} onRejected 获取失败的回调函数
 * @returns {Promise} 新的Promise对象
 */
Promise.prototype.then = function (onFulfilled, onRejected) {
  // 保存当前Promise的this
  const that = this;
  // 返回一个新的Promise
  return new Promise((resolve, reject) => {
    if (that.state = FULFILLED) {
      // 立即异步执行成功的回调函数
      setTimeout(() => {
        /**
        * 返回Promise的结果由onFulfilled或onRejected执行结果决定
        * 情况1.抛出异常 返回Promise的结果为异常，reason为异常
        * 情况2.返回的是Promise，返回Promise的结果就是这个Promise的结果
        * 情况3.返回的不是Promise，返回的Promise为成功，value就是返回值
        */
        try {
          const result = onFulfilled(that.data);
          if (result instanceof Promise) {
            // 情况2
            result.then(resolve, reject);
          } else {
            // 情况3
            resolve(result);
          }
        } catch (error) {
          // 情况1
          reject(error);
        }
      });
    } else if (that.state = REJECTED) {
      // 立即异步执行失败的回调函数
      setTimeout(() => {
        /**
        * 返回Promise的结果由onFulfilled或onRejected执行结果决定
        * 情况1.抛出异常 返回Promise的结果为异常，reason为异常
        * 情况2.返回的是Promise，返回Promise的结果就是这个Promise的结果
        * 情况3.返回的不是Promise，返回的Promise为成功，value就是返回值
        */
        try {
          const result = onRejected(that.data);
          if (result instanceof Promise) {
            // 情况2
            result.then(resolve, reject);
          } else {
            // 情况3
            resolve(result);
          }
        } catch (error) {
          // 情况1
          reject(error);
        }
      });
    } else { // that.state = PENDING
      // 保存回调函数到callbacks中去
      // that.callbacks.push({
      //   onFulfilled, onRejected
      // }); // 直接赋值onFulfilled, onRejected
      // 没有机会改变Promise的状态，所以要像成功失败那样再包一层函数
      that.callbacks.push({
        onFulfilled: handle(onFulfilled),
        onRejected: handle(onRejected)
      });
    }
  });
}
```

### 封装handle

在.then方式中成功和失败的处理很类似，所以封装出来

``` javascript
const handle = callback => {
  /**
   * 返回Promise的结果由onFulfilled或onRejected执行结果决定
   * 情况1.抛出异常 返回Promise的结果为异常，reason为异常
   * 情况2.返回的是Promise，返回Promise的结果就是这个Promise的结果
   * 情况3.返回的不是Promise，返回的Promise为成功，value就是返回值
   */
  try {
    const result = callback(that.data);
    if (result instanceof Promise) {
      // 情况2
      result.then(resolve, reject);
    } else {
      // 情况3
      resolve(result);
    }
  } catch (error) {
    // 情况1
    reject(error);
  }
}
```

### 优化后的then方法

```javascript
/**
 * 原型对象上的then方法
 * @param {Function} onFulfilled 获取成功的回调函数
 * @param {Function} onRejected 获取失败的回调函数
 * @returns {Promise} 新的Promise对象
 */
Promise.prototype.then = function (onFulfilled, onRejected) {
  // 设置回调函数默认值（必须是函数），Promise继续传递的保障
  onResolved = typeof onResolved === 'function' ? onResolved : value => value;
  onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason; };

  // 保存当前Promise的this
  const that = this;
  // 返回一个新的Promise
  return new Promise((resolve, reject) => {
    const handle = callback => {
      /**
       * 返回Promise的结果由onFulfilled或onRejected执行结果决定
       * 情况1.抛出异常 返回Promise的结果为异常，reason为异常
       * 情况2.返回的是Promise，返回Promise的结果就是这个Promise的结果
       * 情况3.返回的不是Promise，返回的Promise为成功，value就是返回值
       */
      try {
        const result = callback(that.data);
        if (result instanceof Promise) {
          // 情况2
          result.then(resolve, reject);
        } else {
          // 情况3
          resolve(result);
        }
      } catch (error) {
        // 情况1
        reject(error);
      }
    }
    if (that.state = FULFILLED) {
      // 立即异步执行成功的回调函数
      setTimeout(() => {
        handle(onFulfilled);
      });
    } else if (that.state = REJECTED) {
      // 立即异步执行失败的回调函数
      setTimeout(() => {
        handle(onRejected);
      });
    } else { // that.state = PENDING
      // 保存回调函数到callbacks中去
      that.callbacks.push({
        onFulfilled: handle(onFulfilled),
        onRejected: handle(onRejected)
      });
    }
  });
}
```

### 原型对象上的catch方法

```javascript
/**
 * 原型对象上的catch方法
 * @param {Function} onRejected 获取失败的回调函数
 * @returns {Promise} 新的Promise对象
 */
Promise.prototype.catch = function (onFulfilled, onRejected) {
  return this.then(undefined, onRejected);
}
```

### 函数对象上的resolve和reject方法

```javascript
/**
 * Promise函数对象的resolve方法
 * 如果收到的是个Promise对象，那么传递下去
 * 如果是一个值，就直接返回值
 * @param {*} value
 * @returns Promise
 */
Promise.resolve = function (value) {
  return new Promise((resolve, reject) => {
    if (value instanceof Promise) {
      value.then(resolve, reject);
    } else {
      resolve(value);
    }
  })
}

/**
 * Promise函数对象的reject方法
 * @param {*} reason 
 * @returns Promise
 */
Promise.reject = function (reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  });
}
```

### 函数对象上的all和race方法

```javascript
/**
 * Promise函数对象的all方法
 * 所有成功才成功，只要一个失败则失败
 * @param {Array<Promise>} promises promise的数组
 */
Promise.all = function (promises) {
  // 定义返回结果数组的长度，因为返回的数组要按照传入的数组
  const values = new Array(promises.length);
  // 计算器变量，保证 所有成功才成功
  let resolvedCount = 0;
  return new Promise((resolve, reject) => {
    promises.forEach((promise, index) => {
      P.resolve(promise).then(
        value => {
          // 计数器增加
          resolvedCount++;
          // 按数组顺序 塞进返回结果数组
          values[index] = value;
          if (resolvedCount === promises.length) {
            resolve(values);
          }
        },
        reason => {
          reject(reason);
        }
      );
    });
  });
}

/**
 * Promise函数对象的race方法
 * 第一个结束的 成功则成功 失败则失败
 * @param {Array<Promise>} promises promise的数组
 */
Promise.race = function (promises) {
  return new Promise((resolve, reject) => {
    promises.forEach(promise => {
      Promise.resolve(promise).then(
        value => {
          resolve(value);
        },
        reason => {
          reject(reason);
        }
      );
    });
  });
};
```

### 完整的Promise

``` javascript
(function (window) {
  // 三个状态常量
  const PENDING = 'pending', // 等待
    FULFILLED = 'fulfilled', // 成功
    REJECTED = 'rejected';   // 失败
  /**
   * Promise 类
   * @param {Function} excutor 执行器
   */
  const Promise = (excutor) => {
    // 初始化状态
    this.state = PENDING;
    // 存储数据
    this.data = null;
    // 存储.then的回调函数 {onFulfilled, onRejected}
    // 可能多次.then，所以是一个数据
    this.callbacks = [];

    /**
     * resolve 成功函数
     * @param {*} value 成功的数据
     */
    resolve = value => {
      // 检查状态，保证状态只能改变一次
      if (this.state !== PENDING) {
        return;
      }
      // 改状态
      this.state = FULFILLED;
      // 存数据
      this.data = value;
      // 检查回调，如果有，立即执行回调函数
      if (this.callbacks.length > 0) {
        setTimeout(() => {
          this.callbacks.forEach(callbackObj => {
            callbackObj.onFulfilled(this.data);
          });
        });
      }
    }

    /**
     * reject 失败函数
     * @param {*} reason 失败的原因
     */
    reject = reason => {
      // 查状态，保证状态只能改变一次
      if (this.state !== PENDING) {
        return;
      }
      // 改状态
      this.state = REJECTED;
      // 存数据
      this.data = reason;
      // 检查回调，如果有，立即执行回调函数
      if (this.callbacks.length > 0) {
        setTimeout(() => {
          this.callbacks.forEach(callbackObj => {
            callbackObj.onRejected(this.data);
          });
        });
      }
    }
    try {
      excutor(resolve, reject); // 立即执行 执行器函数
    } catch (error) {
      reject(error); // 异常就直接抛出
    }
  }

  /**
   * 原型对象上的then方法
   * @param {Function} onFulfilled 获取成功的回调函数
   * @param {Function} onRejected 获取失败的回调函数
   * @returns {Promise} 新的Promise对象
   */
  Promise.prototype.then = function (onFulfilled, onRejected) {
    // 设置回调函数默认值（必须是函数），Promise继续传递的保障
    onResolved = typeof onResolved === 'function' ? onResolved : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason; };

    // 保存当前Promise的this
    const that = this;
    // 返回一个新的Promise
    return new Promise((resolve, reject) => {
      const handle = callback => {
        /**
         * 返回Promise的结果由onFulfilled或onRejected执行结果决定
         * 情况1.抛出异常 返回Promise的结果为异常，reason为异常
         * 情况2.返回的是Promise，返回Promise的结果就是这个Promise的结果
         * 情况3.返回的不是Promise，返回的Promise为成功，value就是返回值
         */
        try {
          const result = callback(that.data);
          if (result instanceof Promise) {
            // 情况2
            result.then(resolve, reject);
          } else {
            // 情况3
            resolve(result);
          }
        } catch (error) {
          // 情况1
          reject(error);
        }
      }
      if (that.state = FULFILLED) {
        // 立即异步执行成功的回调函数
        setTimeout(() => {
          handle(onFulfilled);
        });
      } else if (that.state = REJECTED) {
        // 立即异步执行失败的回调函数
        setTimeout(() => {
          handle(onRejected);
        });
      } else { // that.state = PENDING
        // 保存回调函数到callbacks中去
        that.callbacks.push({
          onFulfilled: handle(onFulfilled),
          onRejected: handle(onRejected)
        });
      }
    });
  }

  /**
   * 原型对象上的catch方法
   * @param {Function} onRejected 获取失败的回调函数
   * @returns {Promise} 新的Promise对象
   */
  Promise.prototype.catch = function (onFulfilled, onRejected) {
    return this.then(undefined, onRejected);
  }

  /**
   * Promise函数对象的resolve方法
   * 如果收到的是个Promise对象，那么传递下去
   * 如果是一个值，就直接返回值
   * @param {*} value
   * @returns Promise
   */
  Promise.resolve = function (value) {
    return new Promise((resolve, reject) => {
      if (value instanceof Promise) {
        value.then(resolve, reject);
      } else {
        resolve(value);
      }
    })
  }

  /**
   * Promise函数对象的reject方法
   * @param {*} reason 
   * @returns Promise
   */
  Promise.reject = function (reason) {
    return new Promise((resolve, reject) => {
      reject(reason);
    });
  }

  /**
   * Promise函数对象的all方法
   * 所有成功才成功，只要一个失败则失败
   * @param {Array<Promise>} promises promise的数组
   */
  Promise.all = function (promises) {
    // 定义返回结果数组的长度，因为返回的数组要按照传入的数组
    const values = new Array(promises.length);
    // 计算器变量，保证 所有成功才成功
    let resolvedCount = 0;
    return new Promise((resolve, reject) => {
      promises.forEach((promise, index) => {
        P.resolve(promise).then(
          value => {
            // 计数器增加
            resolvedCount++;
            // 按数组顺序 塞进返回结果数组
            values[index] = value;
            if (resolvedCount === promises.length) {
              resolve(values);
            }
          },
          reason => {
            reject(reason);
          }
        );
      });
    });
  }

  /**
   * Promise函数对象的race方法
   * 第一个结束的 成功则成功 失败则失败
   * @param {Array<Promise>} promises promise的数组
   */
  Promise.race = function (promises) {
    return new Promise((resolve, reject) => {
      promises.forEach(promise => {
        Promise.resolve(promise).then(
          value => {
            resolve(value);
          },
          reason => {
            reject(reason);
          }
        );
      });
    });
  };

  window.Promise = Promise;
})(window); // IIEF
```
