---
title: JS 事件循环
tags: 事件循环
categories: javascript
---

概述： JavaScript 有一个基于事件循环的并发模型，事件循环负责执行代码、收集和处理事件以及执行队列中的子任务

## 概念

1. 栈： 函数调用形成了一个由若干帧组成的栈(先进后出)

- 执行栈: 同步代码的执行，按照顺序添加到执行栈中

```javascript
function sampleInner(a) {
  let b = 10;
  return a + b;
}
function sampleOuter(m) {
  let n = 3;
  // step 2
  // 第二个帧被创建并被压入栈中, 放在第一个帧之上
  // 帧中包含 sampleInner 的参数 a 和局部变量 b
  return sampleInner(m + n);
  // step 3
  // sampleInner 执行完毕然后返回
  // 第二个帧就被弹出栈（剩下 sampleOuter 函数的调用帧 ）
}

// step 1
// 第一个帧被创建并压入栈中
// 帧中包含了 sampleOuter 的参数 m 和局部变量 n
sampleOuter(12);
// step 4
// 当 sampleOuter 也执行完毕然后返回时，第一个帧也被弹出，栈就被清空了
```

2. 堆： 对象被分配在堆中，堆是一个用来表示一大块（通常是非结构化的）内存区域的计算机术语。

3. 队列：

- 一个 JavaScript 运行时包含了一个待处理消息的消息队列。每一个消息都关联着一个用以处理这个消息的回调函数。
- 在 [**事件循环**](#js-事件循环) 期间的某个时刻，运行时会从最先进入队列的消息开始处理队列中的消息。被处理的消息会被移出队列，并作为输入参数来调用与之关联的函数。正如前面所提到的，调用一个函数总是会为其创造一个新的栈帧。 (先进先出)
- 函数的处理会一直进行到执行栈再次为空为止；然后事件循环将会处理队列中的下一个消息（如果还有的话）。

## 特性/执行过程

- 检查调用栈是否为空，以及确定把哪个 task 加入调用栈的这个过程就是事件循环
- 浏览器至少有一个事件循环，一个事件循环至少有一个任务队列（macrotask），每个外任务都有自己的分组，浏览器会为不同的任务组设置优先级

```javascript
// 同步等待
while (isReady()) {
  // 完成后处理， 然后循环到isReady
  execute();
}
```

### 特征 1 **执行至完成**

- 既当一个函数执行时，它不会被抢占，只有在它运行完毕之后才会去运行任何其他的代码。 如果消息需处理时间过长，可能造成阻塞，

### 特征 2 **添加消息**

- 在浏览器里，每当一个事件发生并且有一个事件监听器绑定在该事件上时，一个消息就会被添加进消息队列。如果没有事件监听器，这个事件将会丢失。所以当一个带有点击事件处理器的元素被点击时，就会像其他事件一样产生一个类似的消息。
- setTimeout 的第二个参数表示最少延迟时间， 在队列之前的消息未处理完成时，setTimeout 并不会在计时器到期之后直接执行

```javascript
function getSeconds() {
  return new Date().getSeconds();
}

function consoleWidthDelay(time, originSeconds) {
  setTimeout(() => {
    // step3
    // setTimeout 消息必须等待其它消息处理完。因此第二个参数仅仅表示最少延迟时间
    console.log("等待了" + (getSeconds() - originSeconds) + "s");
  }, time);
}

function addMessageSample() {
  const originSeconds = getSeconds();
  const LOOP_TIME = 2;
  // step1
  consoleWidthDelay(500, originSeconds);
  // step2
  while (true) {
    if (getSeconds() - originSeconds >= LOOP_TIME) {
      console.log("循环" + LOOP_TIME.toString() + "s");
      break;
    }
  }
}

addMessageSample();
```

### 特征 3 **零延迟**

零延迟并不意味着回调会立即执行， 其等待的时间取决于队列里**待处理消息的处理时间**

## 宏任务和微任务

- 页面渲染事件，各种 IO 的完成事件等随时被添加到任务队列中，一直会保持先进先出的原则执行，我们不能准确地控制这些事件被添加到任务队列中的位置。但是这个时候突然有高优先级的任务需要尽快执行，那么一种类型的任务就不合适了，所以引入了微任务队列。

- 宏任务：

1. script(整体代码)
2. setTimeout()
3. setInterval()
4. postMessage
5. I/O
6. UI 交互事件
7. setImmediate(nodejs)
8. ...

- 微任务：

1. new Promise().then(回调)
2. MutationObserver(html5 新特性)
3. process.nextTick(nodejs)
4. Object.observe
5. ...

- 机制

```javascript
function microTask(count) {
  return (
    Promise.resolve()
      // 微任务
      .then(function () {
        console.log("microTask" + count);
      })
      // 微任务
      .then(function () {
        console.log("microTask-next" + count);
      })
  );
}

function macroTask() {
  // 宏任务
  setTimeout(() => {
    console.log("macroTask-执行");
  });
}

function mainTask() {
  console.log("开始");
  microTask(1);
  macroTask();
  microTask(2);
  macroTask();
  console.log("结束");
}
// 输出
// 开始
// 结束
// microTask1
// microTask2
// microTask-next1
// microTask-next2
// macroTask
```

```js
// 接上面， 修改macrotask
function macroTask(count) {
  setTimeout(() => {
    console.log("macroTask-" + count + "-执行");
    microTask(count);
  });
}

function mainTask() {
  macroTask(1);
  macroTask(2);
}

mainTask();
// 输出
// macroTask-1-执行
// microTask1
// microTask-next1
// macroTask-2-执行
// microTask2
// microTask-next2
```

- 一个完整的事件循环

1. **检查** macrotask 队列是否为空，非空则到 2，为空则到 3
2. **执行** macrotask 中的一个任务
3. **(继续)检查** microtask 队列是否为空，非空则到 4，否则到 5
4. 取出 microtask 中的任务**执行**，执行完成返回到步骤 3
5. 执行视图更新

### 视图渲染(update rendering)时间

- 视图渲染发生在本轮事件循环的 microtask 队列被执行完之后，也就是说执行任务的耗时会影响视图渲染的时机。通常浏览器以每秒 60 帧（60fps）的速率刷新页面，据说这个帧率最适合人眼交互，大概 16.7ms 渲染一帧，所以如果要让用户觉得顺畅，单个 [macrotask](#宏任务和微任务) 及它相关的所有 [microtask](#宏任务和微任务) 最好能在 16.7ms 内完成。
- 浏览器同时有自己的优化策略，例如把几次的视图更新累积到一起重绘，重绘之前会通知 requestAnimationFrame 执行回调函数，也就是说 requestAnimationFrame 回调的执行时机是在一次或多次事件循环的 UI render 阶段。

```javascript
function microTask(count) {
  return new Promise(function (resolve) {
    console.log("microTask" + count + " top");
    resolve();
    console.log("microTask" + count + " middle");
  }).then(function () {
    console.log("microTask" + count + " bottom");
  });
}

function animationFrame() {
  requestAnimationFrame(function () {
    console.log("requestAnimationFrame");
  });
}
function macroTask(count) {
  setTimeout(function () {
    console.log("macroTask" + count);
  }, 0);
}

function mainTask() {
  macroTask(1);
  animationFrame();
  microTask(1);
  macroTask(2);
  document.body.setAttribute("style", "color: #eee");
  console.log("task-end");
}
```

### 总结

1. 事件循环是 js 实现异步的核心
2. 每轮事件循环分为 3 个步骤：
   a) 执行 macrotask 队列的一个任务
   b) 执行完当前 microtask 队列的所有任务
   c) UI render

3. 浏览器只保证 requestAnimationFrame 的回调在重绘之前执行，没有确定的时间，何时重绘由浏览器决定
