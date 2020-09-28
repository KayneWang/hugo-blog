---
title: "JavaScript 大整数"
date: 2020-09-28T10:15:17+08:00
lastmod: 2020-09-28T10:15:17+08:00
draft: true
keywords: ["bigNumber"]
description: ""
tags: ["js", "bigNUmber"]
categories: []
author: "kayne"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

JavaScript 大整数操作相关方法

<!--more-->

## 原理

js 整数最大不能超过 2^53 - 1，[参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER)

所以，为了避免丢失精度，我们需要用字符串来表示数据。

## 相加

[String.prototype.padStart()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/padStart) 使用另一个字符串填充当前字符串

```js
let a = "9007199254740991";
let b = "1234567899999999999";

function add(a, b) {
  // 取两个数字的最大长度
  const maxLength = Math.max(a.length, b.length);
  // 用 0 补齐长度
  a = a.padStart(maxLength, 0); // "0009007199254740991"
  b = b.padStart(maxLength, 0); // "1234567899999999999"
  // 定义变量
  let t = 0;
  let f = 0; // 进位
  let sum = "";
  for (let i = maxLength - 1; i >= 0; i--) {
    t = ~~a[i] + ~~b[i] + f; // ~~ 相当于 parseInt()
    f = Math.floor(t / 10); // 向下取整
    sum = t % 10 + sum;
  }
  if (f === 1) {
    sum = "1" + sum;
  }

  return sum;
}
```