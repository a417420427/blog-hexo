---
title: 蒙特卡洛方法 Monte Carlo method
categories: 算法
---

## 蒙特卡洛方法基本思想

- 常蒙特卡罗方法可以粗略地分成两类：

1. 所求解的问题本身具有内在的随机性，借助计算机的运算能力可以直接模拟这种随机的过程。
2. 所求解问题可以转化为某种随机分布的特征数，比如随机事件出现的概率，或者随机变量的期望值。通过随机抽样的方法，以随机事件出现的频率估计其概率，或者以抽样的数字特征估算随机变量的数字特征，并将其作为问题的解。

### 一个经典的用蒙特卡洛方法求 π 值

- 基本思路

1. 正方形内部有一个相切的圆，它们的面积之比是 π/4
2. 正方形内部，随机产生 n 个点，计算它们与中心点的距离，并且判断是否落在圆的内部
3. 若这些点均匀分布，则圆周率 pi=4 \* num/n (num 表示落到圆内投点数，n 表示总的投点数）

```js
// 根据正方形
function getPi(count) {
  let num = 0;
  for (let i = 0; i < count; i++) {
    // 在一个边长为2的正方形内随机投放点
    let x = randomLocation(-1, 1);
    let y = randomLocation(-1, 1);
    // x,y 到中心的距离小于1 则在圆内
    if (Math.pow(x) + Math.pow(y) < 1) {
      num++;
    }
  }
  return (num * 4.0) / count;
}
// 范围内随机投放某个点
// 所有投放的点概率相同
function randomLocation(min, max) {
  return min + Math.random() * (max - min);
}
```
