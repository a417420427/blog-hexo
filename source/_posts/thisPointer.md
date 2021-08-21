---
title: bind, apply, call 的使用及区别
categories: javascript
tags: this指针
---

<!--
  name: bind, apply, call
  description: bind, apply, call的使用场景,区别及转换
-->

- bind，call，apply 的作用都是用来改变 this 指向的

1. 如果函数不带参数， 只是绑定 this 对象， 下面三种情况产生的效果是一样的

```js
//
function main() {
  const thisObj = { name: "this" };
  function log() {
    console.log(this.name);
    return "log";
  }
  // 执行函数, 返回 'log'
  log.apply(thisObj);
  log.bind(thisObj)();
  log.call(thisObj);
}
main();
```

2. 如果函数带参数,

- apply 第二个参数(必须是数组或者类数组)将展开作为函数的参数， bind, call 则是第二个参数以后的参数都作为函数的参数
- bind 会返回 this 指针改变后的函数， call 会直接执行

```js
function main() {
  const thisObj = { name: "this" };
  function logWidthArgs(...args) {
    console.log(...args, this.name);
    return "log";
  }

  // 打印 1, 2 "this"
  logWidthArgs.apply(thisObj, [1, 2], 3);
  const fn = logWidthArgs.bind(thisObj, [1, 2], 3);
  // 打印 [1,2], 3 "this"
  fn();
  // 打印 [1,2],3 "this"
  logWidthArgs.call(thisObj, [1, 2], 3);
}
```

3. 使用场景

- 如果不带参数或者只有一个, 且无需保存函数， 则三个函数都可以使用
- 如果需要保存函数， 则使用 bind

4. 转换(不同过三者直接生成的暂时没找到方法， js 改变 this 指针的方法主要只提供了这三个)

```js
function bindFn(thisObj, ...args) {
  const _this = this;
  return function () {
    _this.apply(thisObj, [...args]);
  };
}

function applyFn(thisObj, ...args) {
  return thus.apply(thisObj, [...args]);
}
Function.prototype.bindFn = bindFn;
Function.prototype.applyFn = applyFn;
```
