
---
title: Vue slot
categories: Vue javascript
tags: Vue slot
---

### 普通插槽 默认名称为 default

```html
<!-- SlotChild -->
<slot>
  替换前内容1
</slot>

<slot-child>
  替换后内容
</slot-child>
```

### 具名插槽 Named Slots

```html
<slot name="content">
  替换前内容2
</slot>
<slot-child>
  这里不会显示
  <div slot="content">
    替换方法1
  </div>
  <template v-slot:content>
    替换方法2
  </template>
  <template #header>替换方法3</template>
  3种写法都可以 写了多个只会展示最后一个
</slot-child>
```

### 具名+作用域插槽 Named Scoped Slots

```html
<slot name="content" :variable1="{ name: 'variable1' }" varible2="varible2"></slot>
<slot-child>
  这里不会显示
  <template v-slot:content="content">
    {{ content }}
  </template>
  content = { variable1: {name: 'variable1'}, variable2: 'variable2' }
</slot-child>
```

### tips

1. slot 不要添加事件， 因为 slot 会在被替换内容渲染时被移除掉

2. 替换 slot 内容的元素必须在组件的第一层

```html
<slot-child>
  <div slot="sample">这里的替换会生效</div>
  <div>
    <div slot="sample">这里的替换不会生效</div>
  </div>
</slot-child>
```
3. 需要传递变量时建议使用template包裹
  - 不容易造成歧义
  - 避免可能的属性冲突

### 参考资料

1. https://vuejs.org/guide/components/slots.html#scoped-slots 官方文档
2. https://github.com/vuejs/vue/issues/4332#issuecomment-263444492 slot方法传递问题
3. https://stackoverflow.com/questions/43370275/vue-named-slots-do-not-work-when-wrapped slot内容元素
4. https://github.com/vuejs/rfcs/blob/master/active-rfcs/0001-new-slot-syntax.md 官方更新记录
5. https://github.com/vuejs/vue/issues/7740#issuecomment-371309357 相关issue
