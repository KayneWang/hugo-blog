---
title: "DOM 遍历方式"
date: 2020-09-24T17:19:17+08:00
lastmod: 2020-09-24T17:19:17+08:00
draft: false
keywords: ["DFS", "BFS"]
description: "深度优先，广度优先遍历"
tags: []
categories: []
author: ""

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

DOM 的深度优先遍历和广度优先遍历

<!--more-->

## DOM 树

```html
<div class="root">
  <div class="container">
    <section class="sidebar">
      <ul class="menu"></ul>
    </section>
    <section class="main">
      <article class="post"></article>
      <p class="copyright"></p>
    </section>
  </div>
</div>
```

## 深度优先

结果：[root, container, sidebar, menu, main, post, copyright]

### 递归方式

```js
const arr = [];
function traversalDFSDOM (rootDom) {
  if (!rootDom) return;
  if (rootDom.children.length === 0) { // 叶子节点
    arr.push(rootDom);
    return;
  }
  arr.push(rootDom); // 非孩子节点，每次遍历孩子节点之前需要 push 到数组中
  for (let i = 0; i < rootDom.children.lenght; i++) {
    traversalDFSDOM(rootDom.children[i]);
  }
}
```

### 非递归

```js
const arr = [];
function traversalDFSDOM (rootDom) {
  if (!rootDom) return;
  const stack = [];
  let node = rootDom;
  while (node !== null) {
    arr.push(node);
    if (node.children.length >= 0) {
      for (let i = node.children.length - 1; i >= 0; i--) {
        stack.unshift(node.children[i]);
      }
      node = stack.shift();
    }
  }
}
```

## 广度优先

结果：[root, container, sidebar, main, menu, post, copyright]

### 递归方式

```js
const stack = [rootDom];
function traversalBFSDOM (count) {
  count = count | 0;
  if (stack[count]) {
    let children = stack[count].children;
    for (let i = 0; i < children.length; i++) {
      stack.push(children[i]);
    }
    traversalBFSDOM(++count);
  }
}
```

### 非递归

<b>先进先出的思想</b>

```js
const arr = [];
function traversalDFSDOM (rootDom) {
  if (!rootDom) return;
  arr.push(rootDom);
  const queue = [rootDom];
  while (queue.length) {
    let node = queue.shift();
    if (!node.children.length) {
      continue;
    }
    for (let i = 0; i < node.children.length; i++) {
      arr.push(node.children[i]);
      queue.push(node.children[i]);
    }
  }
}
```

