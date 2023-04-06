---
layout: post
title: Javascript String Xor Operation
date: 2023-04-06 08:54:24.000000000 +08:00
tag: javascript
---

JavaScript中字符串是不可变的，因此不能直接进行异或运算。但是，我们可以将字符串转换为数字数组，然后对每个数字进行异或运算，最后将得到的结果转换回字符串。

下面是一个使用异或运算加密字符串的示例代码：

``` js
// 将字符串转换为数字数组
function stringToByte(str) {
  var arr = [];
  for (var i = 0; i < str.length; i++) {
    arr.push(str.charCodeAt(i));
  }
  return arr;
}

// 将数字数组转换回字符串
function byteToString(arr) {
  var str = "";
  for (var i = 0; i < arr.length; i++) {
    str += String.fromCharCode(arr[i]);
  }
  return str;
}

// 对每个字符进行异或运算
function xorEncrypt(str, key) {
  var bytes = stringToByte(str);
  var keyBytes = stringToByte(key);
  
  for (var i = 0; i < bytes.length; i++) {
    bytes[i] ^= keyBytes[i % keyBytes.length];
  }
  
  return byteToString(bytes);
}

// 测试代码
var plaintext = "Hello, world!";
var key = "secret";

console.log("Plaintext: ", plaintext);

var ciphertext = xorEncrypt(plaintext, key);

console.log("Ciphertext: ", ciphertext);

var decryptedText = xorEncrypt(ciphertext, key);

console.log("Decrypted text: ", decryptedText);
```

需要注意的是，因为JavaScript中的数字类型是双精度浮点数，因此在进行位运算时，需要将数字转换为32位整数。可以使用<< 0或者>>> 0操作符来实现这个功能。
