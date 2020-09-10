---
title: "Promise 源码实现"
date: 2020-08-31T19:52:40+08:00
lastmod: 2020-08-31T19:52:40+08:00
draft: false
keywords: ["Promise", "源码实现"]
description: "Promise/A+ 规范源码实现"
tags: ["promise"]
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

<b>理解很重要！理解很重要！理解很重要！</b>

<!--more-->

## 规范

1. new Promise时，需要传递一个 executor 执行器，执行器立刻执行
2. executor 接受两个参数，分别是 resolve 和 reject
3. promise 只能从 pending 到 rejected, 或者从 pending 到 fulfilled
4. promise 的状态一旦确认，就不会再改变
5. promise 都有 then 方法，then 接收两个参数，分别是 promise 成功的回调 onFulfilled, 
    和 promise 失败的回调 onRejected
6. 如果调用 then 时，promise已经成功，则执行 onFulfilled，并将promise的值作为参数传递进去。
    如果promise已经失败，那么执行 onRejected, 并将 promise 失败的原因作为参数传递进去。
    如果promise的状态是pending，需要将onFulfilled和onRejected函数存放起来，等待状态确定后，再依次将对应的函数执行(发布订阅)
7. then 的参数 onFulfilled 和 onRejected 可以缺省
8. promise 可以then多次，promise 的then 方法返回一个 promise
9. 如果 then 返回的是一个结果，那么就会把这个结果作为参数，传递给下一个then的成功的回调(onFulfilled)
10. 如果 then 中抛出了异常，那么就会把这个异常作为参数，传递给下一个then的失败的回调(onRejected)
11. 如果 then 返回的是一个promise，那么会等这个promise执行完，promise如果成功，
    就走下一个then的成功，如果失败，就走下一个then的失败

## Promise 主函数

```js
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

function Promise(executor) {
  let self = this;
  self.status = PENDING;
  self.onFulfilled = []; // success callbacks
  self.onRejected = []; // rejected callbacks

  function resolve(value) {
    if (self.status === PENDING) {
      self.status = FULLFILLED;
      self.value = value;
      self.onFulfilled.forEach(fn => fn());
    }
  }

  function reject(reason) {
    if (self.status === PENDING) {
      self.status = REJECTED;
      self.reason = reason;
      self.onRejected.forEach(fn => fn());
    }
  }

  try {
    executor(resolve, reject);
  } catch (e) {
    reject(e);
  }
}
```

## Then 方法

```js
Promise.prototype.then = function(onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
  onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason };
  let self = this;

  let promise2 = new Promise((resolve, reject) => {
    if (self.status === FULFILLED) {
      setTimeout(() => {
        try {
          let x = onFulfilled(self.value);
          resolvePromise(promise2, x, resolve, reject);
        } catch(e) {
          reject(e);
        }
      });
    } else if (self.status === REJECTED) {
      setTimeout(() => {
        try {
          let x = onRejected(self.reason);
          resolvePromise(promise2, x, resolve, reject);
        } catch(e) {
          reject(e)
        }
      });
    } else if (self.status === PENDING) {
      self.onFulfilled.push(() => {
        setTimeout(() => {
          try {
            let x = onFulfilled(self.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch(e) {
            reject(e);
          }
        });
      });
      self.onRejected.push(() => {
        setTimeout(() => {
          try {
            let x = onRejected(self.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch(e) {
            reject(e);
          }
        });
      });
    }
  });
  return promise2;
}
```

## resolvePromise 方法

```js
function resolvePromise(promise2, x, resolve, reject) {
  if (promise2 === x) {
    reject(new TypeError('Chaining cycle'));
  }
  if (x && typeof x === 'object' || typeof x === 'function') {
    let used; // PromiseA+ 2.3.3.3 只能调用一次
    try {
      let then = x.then;
      if (typeof then === 'function') {
        then.call(x, (y) => {
          if (used) return;
          used = true;
          resolvePromise(promise2, y, resolve, reject);
        }, (r) => {
          if (used) return;
          used = true;
          reject(r);
        });
      } else {
        if (used) return;
        used = true;
        resolve(x);
      }
    } catch(e) {
      if (used) return;
      used = true;
      reject(e);
    }
  } else {
    resolve(x);
  }
}
```

## 测试

```js
Promise.defer = Promise.deferred = function() {
  let dfd = {};
  dfd.promise = new Promise((resolve, reject) => {
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd;
}
```

```shell
$ npm install -g promises-aplus-tests
```

## 简要说明

1. 规范中要求 onFulfilled 和 onRejected 必须异步调用，所以 setTimeout 只是用来模拟异步，原生的 Promise 并非这么实现。
2. resolvePromise 的 used 开关，也是因为规范中明确表示：当同时 resolvePromise 和 rejectPromise 被调用时，或者对同一参数进行多次调用，
则第一次调用优先，后续忽略任何一步的调用。
3. self.onFulfilled 和 self.onRejected 存储了成功和失败的回调，根据规范 2.6 显示，当状态从 pending 改变时，需要按照顺序去指定 then 对应的回调。
