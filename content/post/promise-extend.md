---
title: "Promise 扩展方法"
date: 2020-09-01T10:40:27+08:00
lastmod: 2020-09-01T10:40:27+08:00
draft: false
keywords: ["promise"]
description: "Promise 扩展"
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

<!--more-->

## Promise.resolve

Promise.resolve(value) 返回一个以给定值解析后的 Promise 对象。

1. 如果 value 是 thenable 对象，则采用最终状态
2. 如果 value 是 promise 对象，则不做修改直接返回
3. 其他情况，直接返回以该 value 值为成功状态的 promise 对象

```js
Promise.resolve = function(param) {
  if (param instanceof Promise) {
    return param;
  }

  return new Promise((resolve, reject) => {
    if (param && typeof param === 'object' && typeof param.then === 'function') {
      setTimeout(() => {
        param.then(resolve, reject);
      });
    } else {
      resolve(param);
    }
  })
}
```

thenable 对象增加 setTimeout 的原因是根据原生 Promise 对象的执行结果推断的。例如下面的测试代码，原生的执行结果为：
20、400、30，为了保证同样的顺序，所以引入 setTimeout

测试代码：

```js
let p = Promise.resolve(20);
p.then((data) => {
  console.log(data);
}):

let p2 = Promise.resolve({
  then: function(resolve, reject) {
    resolve(30);
  }
});
p2.then((data) => {
  console.log(data);
});

let p3 = Promise.resolve(new Promise((resolve, reject) => {
  resolve(400);
}));
p3.then((data) => {
  console.log(data);
});
```

## Promise.reject

```js
Promise.reject = function(reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  })
}
```

## Promie.prototype.catch

用于指定出错时回调，是特殊的 then 方法，catch 之后可以继续使用 then

```js
Promise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected);
}
```

## Promise.prototype.finally

无论失败与否，都会执行该函数，finally 之后可以继续使用 then。并且将值原封不动的传递给后面的 then

```js
Promise.prototype.finally = function(callback) {
  return this.then((value) => {
    return Promise.resolve(callback()).then(() => {
      return value;
    });
  }, (r) => {
    return Promise.resolve(callback()).then(() => {
      throw r;
    })
  })
}
```

## Promise.all

1. 如果传入的参数是一个空的可迭代对象，那么此 promise 对象回调完成，只有这种情况是同步执行的，其他都是异步返回的。
2. 如果传入的参数不包含任何 promise，则返回一个异步完成。
3. promise 中所有的 promise 都完成时，或者不包含 promise 对象时，完成回调。
4. 如果参数中任何一个 promise 失败，则返回失败
5. 任何情况下，Promise.all 返回的 promise 完成状态都是一个数组。

```js
Promise.all = function(promises) {
  promises = Array.from(promises);
  return new Promise((resolve, reject) => {
    let index = 0;
    let result = [];
    if (promises.length === 0) {
      resolve(result);
    } else {
      function processValue(i, data) {
        result[i] = data;
        if (++index === promises.length) {
          resolve(result);
        }
      }
      for (let i = 0; i < promises.length; i++) {
        // promises[i] 有可能是普通值
        Promise.resolve(promises[i]).then((data) => {
          processValue(i, data);
        }, (r) => {
          reject(r);
          return;
        })
      }
    }
  });
}
```

## Promise.race

1. 返回一个 Promise 对象，状态取决于第一个完成的状态
2. 如果传入空数组，则返回的 promise 永远等待
3. 如果迭代包含一个或多个非 promise 值，或 resolve 或 reject，则 race 找到迭代中的第一个值

```js
Promise.race = function(promises) {
  promises = Array.from(promises);
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      return;
    } else {
      for (let i = 0; i < promises.length; i++) {
        Promise.resolve(promises[i]).then((data) => {
          resolve(data);
          return;
        }, (r) => {
          reject(r);
          return;
        });
      }
    }
  });
}
```