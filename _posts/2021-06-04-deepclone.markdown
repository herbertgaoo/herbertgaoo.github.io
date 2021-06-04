---
layout: post
title: Javascript 深度拷贝对象
date: 2021-06-04 10:27:24.000000000 +08:00
tag: Javascript
---

&emsp;&emsp;js通过`JSON.parse`和`JSON.stringify`复制或转换对象时会丢失function，所以使用`deepClone`去处理，记录一下。

``` javascript
// 深拷贝对象
  export function deepClone(obj) {
    const _toString = Object.prototype.toString
  
    // null, undefined, non-object, function
    if (!obj || typeof obj !== 'object') {
      return obj
    }
  
    // DOM Node
    if (obj.nodeType && 'cloneNode' in obj) {
      return obj.cloneNode(true)
    }
  
    // Date
    if (_toString.call(obj) === '[object Date]') {
      return new Date(obj.getTime())
    }
  
    // RegExp
    if (_toString.call(obj) === '[object RegExp]') {
      const flags = []
      if (obj.global) { flags.push('g') }
      if (obj.multiline) { flags.push('m') }
      if (obj.ignoreCase) { flags.push('i') }
  
      return new RegExp(obj.source, flags.join(''))
    }
  
    const result = Array.isArray(obj) ? [] : obj.constructor ? new obj.constructor() : {}
  
    for (const key in obj) {
      result[key] = deepClone(obj[key])
    }
  
    return result
  }
```