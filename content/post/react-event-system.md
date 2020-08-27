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