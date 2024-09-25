---
layout: post
title: 可编辑div插入html
date: 2023-04-14 08:54:24.000000000 +08:00
tag: javascript
---

## 背景
业务中需要在输入框中输入文本和html，所以采用了可编辑DIV。在插入html元素之后需要设置需要设置`range`开始位置为插入元素之后，然后再插入一个`&ZeroWidthSpace;`以保证可以继续在插入html元素之后继续进行输入内容。

## 代码示例
``` html
<template>
    <div>
      <div class="editable-div" contenteditable="true" placeholder="hahahah" v-html="fillContent" @keyup="getlastRange" @mouseup="getlastRange"></div>
      <button type="text" @click="insertBlank">插入填空</button>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Emit, Vue } from "vue-property-decorator"
@Component({
    name: "EditableDiv"
})
export default class EditableDiv extends Vue {
  public fillContent = ""
  public lastRange:any = null
  
  public getlastRange () {
    this.lastRange = window.getSelection()?.getRangeAt(0)
  }
  
  public insertBlank () {
    let selection = getSelection()
    selection?.removeAllRanges()
    selection?.addRange(this.lastRange)
    let range = selection?.getRangeAt(0)

    let blankBox = document.createElement('div')
    blankBox.setAttribute("class", "blank-box");
    blankBox.setAttribute("contenteditable", "true");

    range?.insertNode(blankBox);
    range?.setStartAfter(blankBox)
    
    /*
     * insert &ZeroWidthSpace; after blankBox
     */
    range?.insertNode(document.createTextNode('\u200b'))
    range?.collapse(false);
  }
  
}
</script>

<style lang="less" scoped>
.editable-div {
  background: #ffffff;
  padding: 10px !important;
  border: 1px solid #00000000;
  border-style: dashed!important;
  
  &:empty:before{
    /** add placeholder use attribute */
    content: attr(placeholder);
    color:#ddd;
  }

  &:focus:before{
    content:none;
  }
  &:hover {   border: 1px solid #3f94ff; }
  &:focus{
    border: 1px solid #3f94ff;
    outline:none;
  }
}
</style>

```


