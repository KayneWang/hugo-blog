---
title: "Event 实现"
date: 2020-09-03T17:09:50+08:00
lastmod: 2020-09-03T17:09:50+08:00
draft: false
keywords: []
description: "订阅-发布设计模式"
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

<!--more-->

# Event bus

event bus既是node中各个模块的基石，又是前端组件通信的依赖手段之一，同时涉及了订阅-发布设计模式，是非常重要的基础。

```js
class EventEmitter {
  constructor() {
    this._events = this._events || new Map();
  }
}

EventEmitter.prototype.emit = function(type, ...args) {
  const handler = this._events.get(type);
  if (Array.isArray(handler)) {
    handler.forEach(h => {
      if (args.length > 0) {
        h.apply(this, args)
      } else {
        h.call(this)
      }
    })
  } else {
    if (args.length > 0) {
      handler.apply(this, args)
    } else {
      handler.call(this)
    }
  }
}

EventEmitter.prototype.addListener = function(type, fn) {
  const handler = this._events.get(type)
  if (!handler) {
    this._events.set(type, fn);
  } else if (handler && typeof handler === 'function') {
    this._events.set(type, [handler, fn])
  } else {
    handler.push(fn)
  }
}

EventEmitter.prototype.removeListener = function(type, fn) {
  const handler = this._events.get(type)
  if (handler && typeof handler === 'function') {
    this._events.delete(type)
  } else {
    let position
    handler.forEach((h, i) => {
      if (h === fn) {
        position = i
      } else {
        position = -1
      }
    })

    if (position !== -1) {
      handler.splice(position, 1)
      if (handler.length === 1) {
        this._events.set(type, handler[0])
      }
    } else {
      return this;
    }
  }
}
```