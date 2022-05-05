---
title: vue3中的proxy
categories: javascript
---

之前看到 vue3 更新之后数据监听由 Object.defineProperty 改成了 Proxy, 这两者之前存在着哪些不同？

1. 首先这两者都是通过劫持对象来监听对象变化

```js
const data = { name: "data" };

Object.defineProperty(data, "name", {
  // 获取对象属性 name 的时候执行
  get() {
    return "get:" + data.name;
  },
  // 设置对象属性name 的时候执行
  set(val) {
    data.name = val;
  },
});

const proxyData = new Proxy(
  {},
  {
    get(target, key) {
      return "get:" + target[key];
    },
    set(target, key, val) {
      target[key] = val;
    },
  }
);
```

2. Object.defineProperty 存在几个缺陷

- 并不能检测对象属性的添加和删除
- 无法监控到数组下标的变化
- 嵌套层级比较深的情况下，存在性能问题

3. 利用 Proxy 实现简单的数据相应

```js
function changeAttribute(ele, key, props) {
  if (key === "children") {
    if (typeof props[key] === "string" || typeof props[key] === "number") {
      ele.innerHTML = props[key];
    } else {
      ele.appendChild(props[key]);
    }
  } else {
    ele.setAttribute(key, props[key]);
  }
}

function createElement(type, props) {
  const ele = document.createElement(type);
  Object.keys(props).forEach((key) => {
    changeAttribute(ele, key, props);
  });
  return ele;
}

function createProxyData(state) {
  return new Proxy(state.data, {
    get(target, key) {
      return target[key];
    },
    set(target, key, val) {
      target[key] = val;
      if (state.el) {
        changeAttribute(state.el, key, target);
      }
    },
  });
}
const Component = function () {
  const state = {
    el: null,
    data: {
      class: "nameEle",
      children: 0,
      style: "position: fixed; top: 100px; left: 100px;",
    },
  };
  const proxyNameState = createProxyData(state);
  state.el = createElement("div", proxyNameState);
  document.body.addEventListener("click", () => {
    proxyNameState.children = Number(proxyNameState.children) + 1;
  });
  document.body.addEventListener("dblclick", () => {
    proxyNameState.class += " addedClass";
  });
  return state.el;
};

document.body.appendChild(Component());
```
