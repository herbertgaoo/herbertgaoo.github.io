---
layout: post
title: vue-quill-editor 工具栏自定义字体大小选项
date: 2022-09-02 08:54:24.000000000 +08:00
tag: quill 富文本编辑器
---

## 背景
---
在项目中使用富文本编辑器`vue-quill-editor`, 但是组件默认可选的文字大小仅有四种并且不是设置的行内样式，在编辑的富文本内容脱离了框架之外则不能正常显示设置的样式。

Baidu了好多之后发现都是通过修改`node_modules`中`quill`编译之后的代码，但是这并不是我想要的。在打开正确上网方式网上冲浪之后找到最终的解决方案。

>BTW: 墙裂建议大家不要直接修改`node_modules`中的js、css

## 修改之后的效果
---
![a表](/assets/images/2022/20220902_001.png)


## 修改步骤
---
1. 注册并添加白名单

``` js
import { Quill } from 'vue-quill-editor'
const sizeStyle = Quill.import('attributors/style/size')
sizeStyle.whitelist = ['12px', '14px', '16px', '18px', '20px', '24px', '30px', '32px', '36px']
Quill.register(sizeStyle, true)

```
2. 修改编辑器选项中的`size`属性
``` js
modules: {
    toolbar: [
    ...
    [{ size: ['12px', '14px', '16px', '18px', '20px', '24px', '30px', '32px', '36px']}], // 字体大小
    ...
    ],
}
```
3. 添加对应的css到项目中的css文件中
``` css
.ql-picker-item[data-value='12px']::before, .ql-picker-label[data-value='12px']::before {
  content: '12px' !important;
}

.ql-picker-item[data-value='14px']::before, .ql-picker-label[data-value='14px']::before {
  content: '14px' !important;
}

.ql-picker-item[data-value='16px']::before, .ql-picker-label[data-value='16px']::before {
  content: '16px' !important;
}

.ql-picker-item[data-value='18px']::before, .ql-picker-label[data-value='18px']::before {
  content: '18px' !important;
}

.ql-picker-item[data-value='20px']::before, .ql-picker-label[data-value='20px']::before {
  content: '20px' !important;
}

.ql-picker-item[data-value='24px']::before, .ql-picker-label[data-value='24px']::before {
  content: '24px' !important;
}

.ql-picker-item[data-value='30px']::before, .ql-picker-label[data-value='30px']::before {
  content: '30px' !important;
}

.ql-picker-item[data-value='32px']::before, .ql-picker-label[data-value='32px']::before {
  content: '32px' !important;
}

.ql-picker-item[data-value='36px']::before, .ql-picker-label[data-value='36px']::before {
  content: '36px' !important;
}
```


