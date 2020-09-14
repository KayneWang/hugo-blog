---
title: "如何实现不可变数据"
date: 2020-09-11T14:48:20+08:00
lastmod: 2020-09-11T14:48:20+08:00
draft: false
keywords: [immutable]
description: "不可变数据原理介绍"
tags: []
categories: ['immutable']
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

1. 如何判断目标对象有没有修改过呢？记录一个标志位最简单：

```js
function createState(target) {
  this.modified = false; // 是否被修改
  this.target = target; // 目标对象
  this.copy = undefined; // 拷贝的对象
}
```

2. 这时候，我们就可以通过分析状态来执行不同的操作了：

```js
createState.prototype = {
  // 对于get操作,如果目标对象没有被修改直接返回原对象,否则返回拷贝对象
  get: function(key) {
    if (!this.modified) return this.target[key];
    return this.copy[key];
  },
  // 对于set操作,如果目标对象没被修改那么进行修改操作,否则修改拷贝对象
  set: function(key, value) {
    if (!this.modified) this.markChanged();
    return (this.copy[key] = value);
  },
  // 标记状态为已修改,并拷贝
  markChanged: function() {
    if (!this.modified) {
      this.modified = true;
      this.copy = shallowCopy(this.target);
    }
  },
};
function shallowCopy(value) {
  if (Array.isArray(value)) return value.slice();
  if (value.__proto__ === undefined) {
    return Object.assign(Object.create(null), value);
  }
  return Object.assign({}, value);
}
```

3. 最后我们就可以利用构造函数<b>createState</b>接受目标对象<b>state</b>生成对象<b>store</b>,然后我们就可以用<b>Proxy</b>代理<b>store</b>,<b>producer</b>是外部传进来的操作函数,当<b>producer</b>对代理对象进行操作的时候我们就可以通过事先设定好的<b>handler</b>进行代理操作了

```js
const PROXY_STATE = Symbol('proxy-state');
const handler = {
  get(target, key) {
    if (key === PROXY_STATE) return target;
    return target.get(key);
  },
  set(target, key, value) {
    return target.set(key, value);
  },
};
// 接受一个目标对象和一个操作目标对象的函数
function produce(state, producer) {
  const store = new createState(state);
  const proxy = new Proxy(store, handler);
  producer(proxy);
  const newState = proxy[PROXY_STATE];
  if (newState.modified) return newState.copy;
  return newState.target;
}
```

4. 验证一下，我们可以看到 produce 并没有影响到原来的目标函数

```js
const baseState = [
  {
    todo: 'Learn typescript',
    done: true,
  },
  {
    todo: 'Try immer',
    done: false,
  },
];
const nextState = produce(baseState, draftState => {
  draftState.push({todo: 'Tweet about it', done: false});
  draftState[1].done = true;
});
console.log(baseState, nextState);
/*
[ { todo: 'Learn typescript', done: true },
  { todo: 'Try immer', done: false } ] 
  [ { todo: 'Learn typescript', done: true ,
  { todo: 'Try immer', done: true },
  { todo: 'Tweet about it', done: false } ]
*/
```

## 总结

其实这个库是参考 [immer](https://github.com/immerjs/immer) 来实现的，当然我们的代码十分简陋，主要目的是让大家了解一下不可变数据，感兴趣的可以去看一下完整代码。

由于 immutable.js 的侵入型较强，非必要情况，最好还是不要轻易选型。