---
title: 手写常用的js方法(call apply bind new等)
date: 2020-05-11 10:55:02
tags:
- Javasript
- 前端
---

### 起因
在项目开发中，经常使用`apply`，`call`以及`bind`来改变`js`的`this`指针的指向，但是使用的时候只知道其用处，并不知道其原理，这样对于使我们编程时对`js`的理解并不够深入，今天我们来手写几个`js`常用的方法，有助我们深入理解`js`编程。


### apply
`apply`方法接收两个参数，第一个是`this`指针指向的对象，第二个是被调用函数的参数，参数是以`数组`的形式入参。方法的作用主要是改变`this`指针的指向。
```javascript
Function.prototype.apply2 = function() {
  let ctx = arguments[0];
  /* 判断第一个参数是否是为null或者undefined，如果是的话this指针指向windows */
  if (ctx === null || ctx === 'undefined') {
    ctx = window;
  }
  /* 获取参数 */
  const params = arguments[1];
  /* 改变this指向 */
  const symbol = Symbol();
  ctx.symbol = this;
  /* 显示绑定绑定注入的参数 */
  const res = ctx.symbol(...params);
  delete ctx.symbol;
  return res;
}
```

### call
`call`方法与`apply`方法的作用是一样的，不同的地方是`call`方法接收的不是数组，而是正常的参数。
```javascript
Function.prototype.call2 = function() {
	const arr = [...arguments];
  let ctx = arr.shift();
  /* 如果对象为null或者undefined时，则this指向window */
  if (ctx === null || ctx === undefined) {
    ctx = window;	
  }
  /* 获取参数 */
  const params = arr;
  /* 改变this指向 */
  const symbol = Symbol();
  ctx.symbol = this;
  /* 显示绑定绑定注入的参数 */
  const res = ctx.symbol(...params);
  delete ctx.symbol;
  return res;
}
```

### bind
`bind`稍微复杂一点，`bind`方法与`call`和`apply`不一样，`bind`方法是返回的是被绑定this指向的函数，而不是直接执行函数，入参与`call`方法一样。有点需要注意的是当绑定的函数使用`new`关键字调用时，此时的`this`指向自身，而不是绑定的对象。
```javascript
Function.prototype.bind2 = function() {
  const arr = [...arguments];
  let ctx = arr.shift();
  /* 如果对象为null或者undefined时，则this指向window */
  if (ctx === null || ctx === undefined) {
    ctx = window; 
  }
  let self = this;
  const bound = function() {
    /* 如果this指向自己，则认为使用new来执行，此时this绑定的是自身，因为new绑定大于显示绑定 */
    if (this instanceof bound) {
      ctx = this;
    }
    /* 拼接参数，因为两个地方都可以入参 */
    const params2 = arr.concat(...arguments);
    const res = self.call(ctx, ...params2);
    return res;
  };
  return bound;
}
```

### new
`new`关键字的用作是实例化一个对象。主要内部的步骤如下：
- 创建一个空的对象
- 改变空对象的指向
- 执行函数，如果函数有返回值就返回返回值，没有则返回创建的对象
```javascript
function new2(func) {
  const obj = Object.create(func.prototype);
  const res = func.call(obj);
  if (res) {
    return res;
  }
  return obj;
}
```

### instanceof
`instanceof`的作用主要是检查一个对象是否属于某个类，原理是判断对象的原型属性是否等于类的原型属性，不等就一直沿着对象的原型链然后一直往下找。
```javascript
function instanceof2(obj, Func) {
  let proto = obj.__proto__;
  while(true) {
    if (proto === undefined || proto === null) {
      return false;
    }
    if (proto === Func.prototype) {
      return true;
    }
    proto = proto.__proto__;
  }
}
```