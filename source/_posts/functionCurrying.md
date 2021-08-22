---
title: 函数柯里化
tags: 函数 柯里化
categories: javascript
---

## 定义

在数学和计算机科学中，柯里化是一种将使用多个参数的一个函数转换成一系列使用一个参数的函数的技术。

```js
function sample(arg1, arg2, arg3) {
  return arg1 + arg2 + arg3;
}

const currySample = curry(sample);
// 返回结果同 sample(arg1, arg2, arg3)
currySample(arg1)(arg2)(arg3);
```

## 实现

1. 基础版本 这个是比较常见的实现版本

```js
function curry() {
  let args = [].slice.call(arguments, 1);
  const fn = args.shift();
  return function () {
    const newArgs = args.concat([].slice.call(arguments));
    return fn.apply(this, newArgs);
  };
}
```

2. 灵活传参版本

```js
function curry() {
  let args = Array.from(arguments);
  const fn = args.shift();
  const length = fn.length;
  if (canbeExcute()) {
    return execute();
  }

  return curryFn;

  function curryFn(...argsInner) {
    args = args.concat(argsInner);
    if (canbeExcute()) {
      return execute();
    }
    return curryFn;
  }
  function execute() {
    return fn.apply(this, args);
  }
  function canbeExcute() {
    return args.length >= length;
  }
}
```

## 用途

1. 参数复用， 避免重复传参

```js
// 需要在容器内添加元素时可以直接使用appendToContainer
function append(container, el) {
  container.appendChild(el);
}
const appendToContainer = curry(append, container);
```

2. 提前返回，延时执行， 避免重复判断

```js
// 如果只有在邮箱出错的情况下才验证错误， 可以提前将邮箱验证前置。
function validate(name, age, mail) {
  if (!validateName(name)) {
    // handle
  }
  if (validateAge(age)) {
    // handle
  }
  if (validateMail(email)) {
    return false;
  }
}

function validatePriorityEmail(email, callback) {
  // 将邮箱验证单独提出来
  return validateMail(mail) && callback();
}

const validateWithoutEmail = curry(validatePriorityEmail)(email);

validateWithoutEmail(validate(name, age));
validateWithoutEmail(validate(name, age));
```

## 其他

这里涉及到部分偏函数的功能， 偏函数和柯里化的区别其实可以理解成部分和全部的区别
柯里化是将一个 n 元函数转换成 n 个一元函数
偏函数则是将一个 n 元函数转换成一个 n - x 元
柯里化可以理解成自动版的偏函数，
这里的灵活传参版更像是一个偏函数，这里就不做过多论述
