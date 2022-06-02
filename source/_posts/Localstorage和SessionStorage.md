---
title: Localstorage 和 SessionStorage
tags: javascript
categories: broswer
---

## 定义

### Localstorage

1. 只读的 localStorage 属性允许你访问一个 Document 源（origin）的对象 Storage, 存储的数据将保存在浏览器会话中

   - localStorage 是基于 Storage
   - localStorage 受同源策略影响(a=>a 保留, a=>b 消失)
   - 同源页面 a,b, a 打开 b, a 更改 localStorage 后， b 跟着变化
   - 关闭对应浏览器标签或窗口，不会清除对应的 localStorage

2. 定义

- window.localStorage

### Sessionstorage

1. sessionStorage 属性允许你访问一个，对应当前源的 session Storage 对象。它与 localStorage 相似，不同之处在于 localStorage 里面存储的数据没有过期时间设置，而存储在 sessionStorage 里面的数据在页面会话结束时会被清除。

- sessionStorage 是基于 Storage
- sessionStorage 受同源策略影响(a=>a 保留, a=>b 消失)
- 同源页面 a,b, a 打开 b, a 更改 sessionStorage 后， b 不会跟着变化
- 关闭对应浏览器标签或窗口，会清除对应的 sessionStorage

### Storage 对象

1. 定义

```c++
[Exposed=Window]
interface Storage {
  readonly attribute unsigned long length;
  DOMString? key(unsigned long index);
  getter DOMString? getItem(DOMString key);
  setter undefined setItem(DOMString key, DOMString value);
  deleter undefined removeItem(DOMString key);
  undefined clear();
};
```
  - map 
    a. A storage proxy map is equivalent to a map, except that all operations are instead performed on its backing map.
    b. map是一个映射， 所有操作都在它支持的映射上执行
  - type  'local' or 'session'

2. 广播storage实现步骤 To broadcast(同源下浏览器通讯) a Storage object storage, given a key, oldValue, and newValue, run these steps:

- Let url be storage's relevant global object's associated Document's URL.  关联全局DOM的URL  document.URL

- 令 remoteStorages 为所有储存对象,排除以下
- Let remoteStorages be all Storage objects including storage whose:
 
  - 排除掉type为local 和 session
  - type is storage's type (local 或者 session)
  - 排除同源storage
  - relevant settings object's origin is same origin with storage's relevant settings object's origin.
  - 相关设置对象的浏览会话为存储相关设置对象的浏览会话(session与local的区别)
  - and, if type is "session", whose relevant settings object's browsing session is storage's relevant settings object's browsing session.

- 循环 remoteStorages 中的每一个 remoteStorage
- For each remoteStorage of remoteStorages: 
  - 在全局DOM任务队列中增加一个全局任务， 任务内容为在 remoteStorage 的全局对象上 触发一个 storage 事件
  - queue a global task on the DOM manipulation task source given remoteStorage's relevant global object to fire an event named storage at remoteStorage's relevant global object, 
  - using StorageEvent, with key initialized to key, oldValue initialized to oldValue, newValue initialized to newValue, url initialized to url, and storageArea initialized to remoteStorage.


3. storage赋值过程  setItem(key, value)
  - 定义 orderValue null
  - 定义 reorder(排序器) 为true
  - 如果 map[key] 存在
   * orderValue设置为map[key]
   * 如果orderValue 为传入的value return 
   * Set reorder to false
  - 如果值无法被保存(数据过大或其他错误) throw a "QuotaExceededError" DOMException exception
  - 设置 map[key] 为 value
  - 如果recorder为true 对 storage进行排序
  - Broadcast this with key, oldValue, and value. 

### 参考资料

1. https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage
2. https://html.spec.whatwg.org/multipage/webstorage.html#dom-localstorage
3. https://html.spec.whatwg.org/multipage/webstorage.html#storage-2
