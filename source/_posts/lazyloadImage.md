---
title: 使用队列进行图片并行懒加载
tags: lazyload
categories: javascript
---

## 多张图片并行加载

1. 需求： 之前碰到过这样的需求， 展示图片列表。 由于后端的请求有并行数量限制， 每次最多请求 5 张， 所有图片加载的时候邀请每次只能加载 5 张， 每次加载完一张之后才能继续加载下一张

2. 解决方式 这边使用的队列方法在[这里](http://localhost:4000/2021/08/21/queue/)
<iframe height="300" style="width: 100%;" scrolling="no" title="" src="https://codepen.io/a417420427/embed/KKqPgyG?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/a417420427/pen/KKqPgyG">
  </a> by a417420427 (<a href="https://codepen.io/a417420427">@a417420427</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

```js
import Queue from "https://cdn.skypack.dev/@a417420427/queue";

function render(ele) {
  const queue = new Queue({ concurrency: 5 });
  const links = getLinks();
  links.forEach((link) => {
    // 将图片加载的方法加入队列
    // 加载完成resolve
    queue.push(function () {
      return new Promise((resolve) => {
        const image = createImage(link);
        image.onload = function () {
          resolve();
          console.log("load image end");
        };
        image.onerror = function () {
          resolve();
          console.log("load image error");
        };
        console.log("load image start");
        ele.appendChild(image);
      });
    });
  });
  queue.start();
}
const ele = document.querySelector("#app");

function createImage(url) {
  const image = new Image(40, 40);
  image.src = url;
  return image;
}
// 获取图片地址列表
function getLinks() {
  const urls = new Array(10)
    .fill("")
    .map(
      (_, index) =>
        "https://blogs.zxueping.com/img/images" +
        (index >= 9 ? index + 1 : "0" + (index + 1)) +
        ".jpg"
    );
  return urls;
}

render(ele);
```

-
