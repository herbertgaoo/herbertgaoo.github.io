---
layout: post
title: jQuery QRCode extend
date: 2018-3-21 09:56:24.000000000 +09:00
tag: jQuery js
---

## jquery qrcode

基于jquery.qrcode.min.js的生成二维码功能，在options中扩展了以下几个属性：

    * border 边框（也就是留白）默认值为5px 
    * imgSrc logo图片地址 默认值为空 “” 
    * imgWidth logo图片宽度 默认值为0，如果没有imgSrc则失效 
    * imgHeight logo图片高度 默认值为0，如果不写则默认与
    * imgWidth值相同

## 参数说明options

``` js
{
    render   : "canvas",//设置渲染方式 还有table方式
    width       : 256,     //设置宽度
    height      : 256,     //设置高度
    typeNumber  : -1,      //计算模式
    correctLevel    : QRErrorCorrectLevel.H,//纠错等级
    background      : "#ffffff",//背景颜色
    foreground      : "#000000" //前景颜色
    border: 10,
    imgSrc: "../1.png", //图片地址
    imgWidth: 40,   //图片宽度
    imgHeight: 40 //图片高度可以省略不写， 但是宽度imgWidth是必
}
```

## 下载地址

[jquery-qrcode](https://github.com/herbertgaoo/jquery-qrcode)

