---
title: "React 事件系统"
date: 2020-08-20T20:45:14+08:00
tags: ["React"]
categories: ["React"]
keywords: ["React", "事件委托"]
draft: false
---

# Web 的内存和性能

直接添加到页面上的事件处理程序的数量会直接影响页面性能：

* 每个函数都是对象，都会占用内存
* 必须事先指定所有事件，从而导致DOM访问次数过多

如何解决？

## 事件委托

利用事件冒泡，只指定一个事件处理程序，从而管理某一类的所有事件。例如：

```html
<ul id="myLinks">
    <li id="goSomewhere">Go somewhere</li>
    <li id="doSomething">Do something</li>
    <li id="sayHi">Say hi</li>
</ul>
```

```js
var list = document.getElementById("myLinks");
list.addEventListener("click", function(event) {
    var target = event.target;

    switch(target.id) {
        case "goSomewhere":
            document.title = "I changed the document's title";
            break;
        case "doSomething":
            location.href = "www.baidu.com";
            break;
        case "sayHi":
            alert("hi");
            break;
    }
}, false);
```

例子只取了一个 DOM 元素，添加了一个事件处理程序。对用户来说体验是一样的，但这种做法占用内存更少。

最适合采用事件委托的事件包括：

* click
* mousedown、mouseup
* keydown、keyup、keypress

# React 事件系统

React 基于 VDOM 实现了一个 SyntheticEvent(合成事件)层，完全满足 W3C 标准，所以同样支持事件冒泡。

所有的事件都是自动绑定到最外层，如果需要访问原生事件对象，可以使用 nativeEvent。

其中的机制之一就包括上面所说的事件委托，另外一个是自动绑定。当然在实际开发过程中(使用 ES6 的 class 或者纯函数)还是需要手动进行方法绑定。

## React 合成事件对比 JavaScript 原生事件

从 4 个方面对比

### 事件传播与阻止事件传播

浏览器原生 DOM 事件分为三个阶段：事件捕获、目标对象本身程序调用和事件冒泡。但是事件捕获在低于 IE9 版本的浏览器是不支持的，并且实际开发中意义不大，所以 React 只实现了事件冒泡机制。

阻止原生事件传播使用 e.preventDefault()，低于 IE9 使用 e.cancelBubble = true 。React 中做了兼容，仅需使用第一种即可。

### 事件类型

是原生事件的子集

### 绑定方式

```html
/* DOM 中直接绑定 */
<button onclick="test()"></button>
```

```js
// js 绑定
el.onclick = e => {......}

// 使用监听函数
el.addEventlistener("click", () => {}, false)
el.attachEvent("onclick", () => {})

// react
<button onClick={() => {......}}>React</button>
```

### 事件对象

低版本 IE 浏览器有使用 window.event 来获取事件对象，React 的事件合成系统则不存在这种问题。

# 后续

React 17 版本中事件系统会有变化，可以在文档中[查看](https://zh-hans.reactjs.org/blog/2020/08/10/react-v17-rc.html#changes-to-event-delegation)。