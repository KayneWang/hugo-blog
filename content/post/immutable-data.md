---
title: "如何实现不可变数据"
date: 2020-09-11T14:48:20+08:00
lastmod: 2020-09-11T14:48:20+08:00
draft: true
keywords: [immutable]
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

丢掉幻想，自己实现一个轻量的<b>不可变数据</b>解决方案。

<!--more-->

# 不可变数据

React 文档中写到：建议把 State 当作不可变对象。实际开发过程中大家肯定也体验过不可变数据带来的好处：

* 数据更易追踪、推导
* 对 React diff 的检测机制更友好

## inmutable.js

facebook 工程师耗费3年倾力打造，相比较深克隆性能有极大的提升，但是带来的问题也不少：

1. 实现了所有的不可变结构，导致包很重
2. 全新的数据操作 api，无法从 js 的习惯无缝转换
3. 调试困难，很多情况下都需要使用 toJS() 来转换，使用起来体验不太好
4. 很容易发生滥用的行为，immutable.js 的本质是为了提高 react 渲染性能，但是在使用其数据的时候，大多通过 toJS() 进行转换，这样每次页面更新都会 render，从而导致性能降低

当然，纵使它有一些缺点，但是 immutable.js 也是目前最为完善的库，在大型前端团队并且对数据可靠度要求很高的时候还是首选。

## 自己实现

我们如何通过原生 js 来自己实现一个简单的 immutable 呢？首先可以想到 defineProperty 和 Proxy 来进行额外操作。

我们选择 Proxy 来实现（要用就用新的）

首先定义一个目标对象

```js
const target = {name: 'kayne', age: 27};
```

下面我们实现每访问一次，age 自动加 1 的操作

```js
const handler = {
  get: function(target, key, receiver) {
    console.log(`getting ${key}`);
    if (key === 'age') {
      const age = Reflect.get(target, key, receiver)
      Reflect.set(target, key, age + 1, receiver);
      return age + 1;
    }
    return Reflect.get(target, key, receiver);
  }
}

const a = new Proxy(target, handler);
console.log(a.age, a.age);
//getting age!
//getting age!
//30 31
```
