---
title: 手写简单Promise
date: 2020-05-09 10:44:03
tags: 
- Promise
- Javasript
- ES6
- 前端
---

[参考](https://juejin.im/post/5b2f02cd5188252b937548ab#)

### 什么是Promise
`Promise`是`Javascript`异步编程的一种解决方案，主要是解决地狱回调问题。但Promise具体原理是怎样的？我们今天就来手写一个简单的Promise。

### 一、Promise一共有三种状态
初始状态是`pending`，成功是`fulfilled`，失败时`rejected`。状态只能从初始状态到成功或者失败，不能够逆转。
```javascript
class Promise2 {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    /* 当reslove函数被执行时，状态从pending到fulfilled */
    let resolve = (value) => { 
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
      }
    };
    /* 当reslove函数被执行时，状态从pending到rejected */
    let reject = (reason) => { 
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
      }
    };
    try {
      executor(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }
}
```

### 二、实现异步then功能
第一步实现的功能主要是`Promise`状态转变，但是并没有实现`Promise`最主要的`then`功能以及链式调用，下面我们来实现一个`then`函数功能。
```javascript
class Promise2 {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallback = [];
    this.onRejectedCallback = [];
    /* 当reslove函数被执行时，状态从pending到fulfilled */
    let resolve = (value) => { 
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.onResolvedCallback.map((v) => v());
      }
    };
    /* 当reslove函数被执行时，状态从pending到rejected */
    let reject = (reason) => { 
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this.onRejectedCallback.map((v) => v());
      }
    };
    try {
      executor(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }

  then(onFulFilled, onRejected) {
    /* 如果是异步的话先把函数存起来，待resolve/reject函数执行后再回调 */
    if (this.state === 'pending') {
      this.onResolvedCallback.push(() => {
        onFulFilled(this.value);
      });
      this.onRejectedCallback.push(() => {
        onRejected(this.reason);
      });
    }

    if (this.state === 'fulfilled') {
      onFulFilled(this.value);
    }

    if (this.state === 'rejected') {
    	onRejected(this.reason);
    }

  }
}
```

### 三、链式调用
到这里已经已经能够进行简单的`Promise`使用了，但是`Promise`最牛X的功能链式调用还没有实现，目前`then`方法执行后没有任何操作了，正常的情况应该是返回一个`Promise`，因此我们需要实现链式调用，要实现链式调用需要实现两个步骤
- `then`方法返回一个新的`Promise`
- 根据`then`方法返回的参数做处理
  * 判断res是不是`Promise`
  * 如果是则`取它的结果`作为新的promise2成功的结果
  * 如果是普通值，则直接作为promise2的返回值
```javascript
class Promise2 {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallback = [];
    this.onRejectedCallback = [];
    /* 当reslove函数被执行时，状态从pending到fulfilled */
    let resolve = (value) => { 
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.onResolvedCallback.forEach((v) => v());
      }
    };
    /* 当reslove函数被执行时，状态从pending到rejected */
    let reject = (reason) => { 
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this.onRejectedCallback.forEach((v) => v());
      }
    };
    try {
      executor(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }

  then(onFulFilled, onRejected) {
    /* 用于解决then方法返回promise的返回参数的函数 */
    const onResolvePromise = function(promise2, res, resolve, reject) {
      if (promise2 === res) {
        reject(new TypeError('不能重复使用promise'));
      }
      /** 如果不为null而且是对象的话，就往下走判断是否是Promise  **/
      if (res !== null && (typeof res === 'object' || typeof res === 'function')) {
        const then = res.then;
        if (typeof then === 'function') {
          then.call(res, (x) => {
            onResolvePromise(promise2, x, resolve, reject);
          }, (y) => {
            reject(y);
          });
        } else {
          resolve(res);
        }
      } else {
        resolve(res);
      }
    }

    let p2 = new Promise2((resolve, reject) => {
      /* 如果是异步的话先把函数存起来，待resolve/reject函数执行后再回调 */
      if (this.state === 'pending') {
        this.onResolvedCallback.push(() => {
          /* 不能被同步调用 */
          setTimeout(() => {
          try {
              const res = onFulFilled(this.value);
              onResolvePromise(p2, res, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0);
        });
        /* 不能被同步调用 */
        this.onRejectedCallback.push(() => {
          setTimeout(() => {
            try {
              const res = onRejected(this.reason);
              onResolvePromise(p2, res, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0)
        });
      }
      /* 不能被同步调用 */
      if (this.state === 'fulfilled') {
        setTimeout(() => {
          try {
            const res = onFulFilled(this.value);
            onResolvePromise(p2, res, resolve, reject);
          } catch (e) {
            reject(e);
          }
        }, 0);	

      }  
      /* 不能被同步调用 */
      if (this.state === 'rejected') {
        setTimeout(() => {
          try {
            const res = onRejected(this.reason);
    	      onResolvePromise(p2, res, resolve, reject);	
          } catch (e) {
            reject(e);
          }
        }, 0);
      }
    });
    return p2;
  }
}
```

### 总结
到这里已经完成了`Promise`的大致功能了，但还有一些方法未实现，比如异常处理比如`catch`方法，还有`all`方法等后续有时间一一实现。
- catch：主要是实现了链式捕获异常
- all：用数组形式运行多个`Promise`，只有数组里的状态都为`fulfilled`后，总的`Promise`状态才会变为`fulfilled`，否则会变成`rejected`。
- race：接收一组`Promise`，只要任意一个`Promise`状态改变，那总的`Promise`也会跟着改变，而且会接收该`Promise`的返回参数
- any：接收一组`Promise`，只要任意一个`Promise`状态变更`fulfilled`，那总的`Promise`也会变更`fulfilled`，如果数组里的`Promise`状态变成`rejected`的话，总的也会变成`rejected`
- allSettled：接收一组`Promise`，等所有的`Promise`状态都变了之后才会结束
- finally：最后执行该方法