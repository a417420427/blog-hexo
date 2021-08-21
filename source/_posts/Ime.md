---
title: input method editors (IMEs) 输入法输入文字问题
cover: https://source.unsplash.com/random
---

使用输入法时(Input Method Editors/IME)
在 safari 浏览器上与在 chrome/firefox 浏览器上有不同的表现
导致 keydown 事件触发出问题

# 问题及复现步骤

<iframe height="265" style="width: 100%;" scrolling="no" title="ZEeVyKP" src="https://codepen.io/a417420427/embed/ZEeVyKP?height=265&theme-id=light&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/a417420427/pen/ZEeVyKP'>ZEeVyKP</a> by a417420427
  (<a href='https://codepen.io/a417420427'>@a417420427</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

分辩在不同浏览器上观察事件触发时间可以看出

- 在 chrome/firefox 上用户完成非英文输入按下 Enter 键确定输入结果时，先触发了 keydown 事件，然后才触发 compositionend 事件(相隔 0-1ms，应该是一个微任务)
- 在 safari 上进行同样的操作 会先触发了 compositionend 事件，然后才触发 keydown 事件(相隔 5ms 左右)

# 解决方案

- 监听与 keydown 事件相同容器的 compositionend 事件，记录下时间 compositionEndedAt
- 在 keydown 事件触发时，查看 isComposing 属性以及在 safari 上判断 keydown 与 compositionEndedAt 的时间差

```javascript
  class CompositionManager {
  private compositionEndedAt = -2e8

  isComposing(event: KeyboardEvent): boolean {
    /** isComposing 表示正在输入中 */
    if (event.isComposing) return true
    /** 只针对safari判断事件 */
    if (getBrowserName() === 'Safari') {
      const isComposing =
        Math.abs(event.timeStamp - this.compositionEndedAt) < 500
      this.compositionEndedAt = -2e8
      return isComposing
    }
    return false
  }

  constructor(container: HTMLElement) {
    container.addEventListener('compositionend', (e) => {
      this.compositionEndedAt = e.timeStamp
    })
  }
}

```

# 参考资料

- https://www.w3.org/TR/uievents/#keys-IME
- https://github.com/w3c/uievents/issues/202
- https://github.com/ProseMirror/prosemirror-view/blob/b5420e17f2486e1633a2c06c0a6b25a60b276c5f/src/input.js#L389
- https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/isComposing
