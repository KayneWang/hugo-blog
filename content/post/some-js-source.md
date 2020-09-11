---
title: "JS 常用方法源码实现"
date: 2020-09-03T17:32:21+08:00
lastmod: 2020-09-03T17:32:21+08:00
draft: false
keywords: []
description: ""
tags: []
categories: []
author: "Kayne"

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

JS 常用方法的源码实现

<!--more-->

## 字符串简单模板引擎

```js
let template = '我是{{name}}，年龄{{age}}，性别{{sex}}';
let data = {
  name: '姓名',
  age: 18
}
render(template, data); // 我是姓名，年龄18，性别undefined
```

## 千分位分隔符

```js
parseMoney(1234.56); // 1,234.56

function parseMoney(num) {
  const [integer, decimal] = String.prototype.split.call(num, '.');
  integer = integer.replace(/\d(?=(\d{3})+$)/g, '$&,');
  return integer + '.' + (decimal ? decimal : '');
}
```

```js
function render(template, data) {
  const reg = /\{\{(\w+)\}\}/; // 模板字符串正则
  if (reg.test(template)) { // 判断模板里是否有模板字符串
    const name = reg.exec(template)[1]; // 查找当前模板里第一个模板字符串的字段
    template = template.replace(reg, data[name]); // 将第一个模板字符串渲染
    return render(template, data); // 递归的渲染并返回渲染后的结构
  }
  return template; // 如果模板没有模板字符串直接返回
}
```

## 实现防抖函数

防抖函数原理：在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。

```js
const debounce = (fn, delay) => {
  let timer = null;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
};
```

## 实现节流函数

节流函数原理：规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。

```js
const throttle = (fn, delay = 500) => {
  let flag = true;
  return (...args) => {
    if (!flag) return;
    flag = false;
    setTimeout(() => {
      fn.apply(this, args);
      flag = true;
    }, delay);
  };
};
```

## instanceOf

```js
// 模拟 instanceof
function instance_of(L, R) {
  //L 表示左表达式，R 表示右表达式
  var O = R.prototype; // 取 R 的显示原型
  L = L.__proto__; // 取 L 的隐式原型
  while (true) {
    if (L === null) return false;
    if (O === L)
      // 这里重点：当 O 严格等于 L 时，返回 true
      return true;
    L = L.__proto__;
  }
}
```

## 模拟new

new操作符做了这些事：

* 它创建了一个全新的对象
* 它会被执行[[Prototype]]（也就是__proto__）链接
* 它使this指向新创建的对象
* 通过new创建的每个对象将最终被[[Prototype]]链接到这个函数的prototype对象上
* 如果函数没有返回对象类型Object(包含Functoin, Array, Date, RegExg, Error)，那么new表达式中的函数调用将返回该对象引用

```js
function objectFactory() {
  const obj = new Object();
  const Constructor = [].shift.call(arguments);

  obj.__proto__ = Constructor.prototype;

  const ret = Constructor.apply(obj, arguments);

  return typeof ret === "object" ? ret : obj;
}
```

## 实现 new

```js
function New(f) {
  return function () {
    var o = {
        "__proto__": f.prototype
    };
    f.apply(o, arguments);
    return o;
  }
}
```

## 实现一个call

```js
Function.prototype.myCall = function(context, ...args) {
  context.fn = this;
  context.fn(...args);
  let result = context.fn(...args);
  delete context.fn;
  return result;
};
```

## 实现apply

```js
Function.prototype.myApply = function(context, arr) {
  context.fn = this;
  let result;
  if (!arr) {
      result = context.fn();
  } else {
      result = context.fn(...arr);
  }
  delete context.fn;
  return result;
}
```

## 实现bind

```js
Function.prototype.es6Bind = function(context, ...rest) {
  if (typeof this !== 'function') throw new TypeError('invalid invoked!');
  var self = this;
  return function F(...args) {
    if (this instanceof F) {
      return new self(...rest, ...args)
    }
    return self.apply(context, rest.concat(args))
  }
}
```