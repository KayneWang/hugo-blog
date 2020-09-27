---
title: "JS 中的 ~(按位非) 运算符"
date: 2020-09-25T17:57:03+08:00
lastmod: 2020-09-25T17:57:03+08:00
draft: false
keywords: []
description: ""
tags: []
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

JavaScript 中 ~ 运算符的巧妙运用

<!--more-->

## 基础

某个数值的按位非操作，可以理解为 <b>该值取负值然后减 1</b>

```js
~5 = -5 - 1 = -6
~-5 = 5 - 1 = 4
~4 = -4 - 1 = -5
```

## 原理

4 取反操作：

1. 二进制表示（补码）为：0 100
2. 按位取反：1 011
3. 输出：-5

这时候可能会纳闷为啥输出的是 -5，二进制计算的话不应该是 -3 么？

这是由于计算机所有二进制输出都是补码形式，因此 1011 会转换：

1. 最高位 1 表示负数，因此符号位不变，反码为：1 100 负数的补码等于 **反码 + 1**，即：1 101
2. 抛开符号位，运算得 5，加上符号位，即：-5

同理，我们对 -4 取反：

1. 二进制表示（补码）为：1 100
2. 按位取反：0 011
3. 输出：3

为什么是 3 呢？

很简单，因为 0 011 最高位是 0，正数的补码等于 **源码**，所以直接输出 3

## 应用场景

### indexOf()

判断数组或者字符中元素是否存在：

```js
if (str.indexOf(query) !== -1) {}
if (str.indexOf(query) >= 0) {}
```

可以用按位非操作替换：

```js
if (~str.indexOf(query)) {}
```

当然，这种写法不仅仅是B格高，对于一次运算来说相差无几，只是循环次数过大，比如超过了 10000000 次，效率会有差距。

### ~~value 使用

对于浮点数，使用 ~~value 可以代替 parseInt(value)，位操作效率更高些

```js
parseInt(-2.99) // -2
~~(-2.99) // -2
```