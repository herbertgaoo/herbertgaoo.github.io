---
layout: post
title: vue transition-group 添加缓动效果
date: 2023-04-11 18:54:24.000000000 +08:00
tag: svg
---

## 原理
vue transition-group缓动效果的原理添加`moveClass`， `moveClass`的特性核心代码在 `transition-group` 组件中的 update 钩子中（在修改数据时就会触发这个函数）：

1、获取 moveClass，如果没有在 transition-group 组件节点中定义 moveClass 属性，那么就在 name 属性值后面拼接 move；

2、如果没有子节点或者检测到子节点没有通过 transition 监听 transform 或者 all（通过 hasMove 函数来检测），那么就直接 return；

3、遍历 children 执行 callPendingCbs 函数，由于整个过渡是异步的，在上一个过渡结束之前，如果又触发了 updated 钩子，那么就立即执行上一个 moveCb 或者 enterCb 动画结束的回调函数，确保整个执行是没有问题的；

3、第二次遍历 children 执行 recordPosition 函数，保留当前每个子节点的位置（为平缓移动记录最终态）；

4、第三次遍历 children 执行 applyTranslation 函数，计算每个子节点的最初态和最终态的差值，如果差值不为零，那么通过 transfrom 立即将元素从最终态移动到最初态，并且将这个节点的 data.move 设置成 true，表示这个节点需要缓动；

5、通过 offsetHeight 强制执行重绘，那么节点在最初态就会呈现出来（同一个 tick 中只会呈现出最终计算的位置）；

6、再次遍历 children，给需要缓动的子节点添加 moveClass（使节点带有 transition class 属性，监听 transform 或者 all 的变化），移除 transform class 属性，添加 transitionEndEvent 事件，回调函数为 moveCb，清除 transitionEndEvent 事件、移除 moveClass、将 el._moveCb 设置为 null，表示回调函数已经被执行了。

因此想让 transition-group 组件的子节点有缓动效果有三种方式：1、给 transiiton-group 组件节点添加 moveClass 属性，自定义 class 名，然后在 style 中给该 class 添加 transiiton；2、直接在 style 中给 name + 'move' 拼接的 class 添加 transition；3、在每个子节点中添加一个自定义的 class 名，然后在 style 中给该 class 添加 transition。

然后要注意，子节点的 display 不能为 inline，如果默认是 inline 必须写成 inline-block，这样 FLIP 动画才会生效，这是 Vue 官网中特别提醒的。

## 代码示例
``` html
<template>
    <div>
        <button @click="sort">revers Array</button>
        <transition-group type="transition" name="flip-list">
          <div class="sort-item" v-for="m in messages">{{m}}</div>
        </transition-group>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Emit, Vue } from "vue-property-decorator"
@Component({
    name: "FlipList"
})
export default class FlipList extends Vue {
  public messages = [
    "vue.draggable",
    "draggable",
    "component",
    "for",
    "vue.js 2.0",
    "based",
    "on",
    "Sortablejs"
  ]
  
  public sort () {
    this.messages.reverse()
  }
}
</script>

<style lang="less" scoped>
.flip-list-move {
  transition: .5s;
}
</style>

```


