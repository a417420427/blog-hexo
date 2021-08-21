---
title: JavaScript实现队列结构
subtitle: 队列的实现及延伸
tags: queue
categories: javascript
---

## 队列简介

队列是是一种受限的线性表，特点为先进先出（FIFO：first in first out）

- 受限之处在于它只允许在表的前端（front）进行删除操作；
- 在表的后端（rear）进行插入操作；

## 队列实现

1. 基础版本

```js
function Queue() {
  this.items = [];
}
// 向队列尾部添加一个（或多个）新的项
function enqueue(item) {
  this.items = this.items.concat(item);
}
// 移除队列的第一（即排在队列最前面的）项，并返回被移除的元素
function dequeue() {
  this.items.shift();
}
// 返回队列中的第一个元素——最先被添加，也将是最先被移除的元素。队列不做任何变动
function front() {
  return this.items[0];
}
// 查看队列是否为空
function isEmpty() {
  return this.items.length === 0;
}
// 返回队列包含的元素个数
function size() {
  return this.items.length;
}
Queue.prototype.enqueue = enqueue;
Queue.prototype.dequeue = dequeue;
Queue.prototype.isEmpty = isEmpty;
Queue.prototype.size = size;
Queue.prototype.front = front;
```

2. 函数队列

```js
// 开始运行

function isFunction(props) {
  return Object.prototype.toString.call(props) === "[object Function]";
}
function isArray(props) {
  return Object.prototype.toString.call(props) === "[object Array]";
}

function validateArgs(args) {
  if (isArray(args)) {
    return !args.find((arg) => !isFunction(arg));
  }
  return isFunction(args);
}
// 向队列尾部添加一个（或多个）新的项
function enqueue(item) {
  if (!validateArgs(item)) {
    throw new Error("参数必须为函数或函数数组");
  }
  this.items = this.items.concat(item);
}

function start() {
  if (!this.isEmpty()) {
    this.next();
  }
}
function next() {
  const fn = this.dequeue.apply(this);
  if (fn) {
    fn();
    this.next();
  }
}
Queue.prototype.start = start;
Queue.prototype.next = next;
```

3. 异步函数队列

```js
function isAsyncFunction(props) {
  return Object.prototype.toString.call(props) === "[object AsyncFunction]";
}

async function next() {
  const fn = this.dequeue.apply(this);
  if (fn) {
    await fn();
    await this.next();
  }
}
```

4. 异步函数队列-可设置并行执行函数数量

```js
function Queue(options) {
  this.items = [];
  this.concurrency = (options && options.concurrency) || 1;
}

function start() {
  if (!this.isEmpty()) {
    for (let i = 0; i < this.concurrency; i++) {
      this.next();
    }
  }
}
```

4. 使用场景, 见[图片懒加载](/2021/08/21/lazyloadImage/)
