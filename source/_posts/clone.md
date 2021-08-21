---
title: 深拷贝和浅拷贝
tags: 拷贝
categories: javascript
---

1. 定义 - 浅拷贝只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存。但深拷贝会另外创造一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象。

2. 实现

- 浅拷贝实现

```js
const isArray = (props) =>
  Object.prototype.toString.call(props) === "[object Array]";

const isObject = (props) =>
  Object.prototype.toString.call(props) === "[object Object]";
let result = isArray(obj) ? [] : {};

function shallowCopy(obj) {
  function shallowCopyArray(arr) {
    // return arr.concat([])
    // return arr.slice()
    // return Array.from(arr)
    // return [...arr]
  }

  function shallowCopyObject(obj) {
    // return Object.assign({}, obj)
    // return {...obj}
  }
  return isArray(obj) ? shallowCopyArray(obj) : shallowCopyObject(obj);
}
```

- 深拷贝实现

```js
function deepClone(obj) {
  // 函数的情况不考虑
  function isObjValid(obj) {
    const isObject = typeof obj === "object";
    let isCircular = false;
    try {
      JSON.stringify(obj);
    } catch (e) {
      isCircular = false;
    }
    return isObject && !isCircular;
  }
  if (!isObjValid(obj)) {
    throw new Error("非对象或者对象不可用(有循环引用)");
  }
  for (let key in obj) {
    result[key] = cloneVal(obj[key]);
  }
  return result;

  function cloneVal(val) {
    if (isArray(val) || isObject(val)) {
      return deepClone(val);
    }
    return val;
  }
}
```

- 深拷贝可以通过 JSON.parse(JSON.stringify(obj)) , 但是这种情况循环引用会报错
