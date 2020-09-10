---
title: "JavaScript"
date: 2019-09-15
lastmod: 2020-09-03T14:45:06+08:00
draft: false
keywords: []
description: "js 面试题"
tags: ["js"]
categories: ["js"]
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

JS 基础面试题总结

<!--more-->

## 解释下变量提升

JavaScript引擎的工作方式是，先解析代码，获取所有被声明的变量，然后再一行一行地运行。这造成的结果，就是所有的变量的声明语句，都会被提升到代码的头部，这就叫做变量提升（hoisting）。

```js
console.log(a) // undefined

var a = 1

function b() {
    console.log(a)
}
b() // 1
```
上面的代码实际执行顺序是这样的:

第一步： 引擎将<b>var a = 1</b>拆解为<b>var a = undefined</b>和 <b>a = 1</b>，并将<b>var a = undefined</b>放到最顶端，a = 1还在原来的位置

这样一来代码就是这样:

```js
var a = undefined
console.log(a) // undefined

a = 1

function b() {
    console.log(a)
}
b() // 1
```

## 理解闭包吗

### 闭包是什么

MDN的解释：闭包是函数和声明该函数的词法环境的组合。

按照我的理解就是：闭包 =『函数』和『函数体内可访问的变量总和』

举个简单的例子:

```js
(function() {
    var a = 1;
    function add() {
        var b = 2

        var sum = b + a
        console.log(sum); // 3
    }
    add()
})()
```

add函数本身，以及其内部可访问的变量，即 a = 1，这两个组合在一起就被称为闭包，仅此而已。

### 闭包的作用

闭包最大的作用就是隐藏变量，闭包的一大特性就是<b>内部函数总是可以访问其所在的外部函数中声明的参数和变量，即使在其外部函数被返回（寿命终结）了之后</b>

基于此特性，JavaScript可以实现私有变量、特权变量、储存变量等

我们就以私有变量举例，私有变量的实现方法很多，有靠约定的（变量名前加_）,有靠Proxy代理的，也有靠Symbol这种新数据类型的。

但是真正广泛流行的其实是使用闭包。

```js
function Person(){
    var name = 'cxk';
    this.getName = function(){
        return name;
    }
    this.setName = function(value){
        name = value;
    }
}

const cxk = new Person()

console.log(cxk.getName()) //cxk
cxk.setName('jntm')
console.log(cxk.getName()) //jntm
console.log(name) //name is not defined
```

## JavaScript的作用域链

JavaScript属于静态作用域，即声明的作用域是根据程序正文在编译时就确定的，有时也称为词法作用域。

其本质是JavaScript在执行过程中会创造可执行上下文，可执行上下文中的词法环境中含有外部词法环境的引用，我们可以通过这个引用获取外部词法环境的变量、声明等，这些引用串联起来一直指向全局的词法环境，因此形成了作用域链。

![JS作用域](/img/js-context.png)

## ES6模块与CommonJS模块有什么区别

区别：

* CommonJS是对模块的浅拷贝，ES6 Module是对模块的引用,即ES6 Module只存只读，不能改变其值，具体点就是指针指向不能变，类似const
* import的接口是read-only（只读状态），不能修改其变量值。 即不能修改其变量的指针指向，但可以改变变量内部指针指向,可以对commonJS对重新赋值（改变指针指向），但是对ES6 Module赋值会编译报错。

相同点：

* CommonJS和ES6 Module都可以对引入的对象进行赋值，即对对象内部属性的值进行改变。

## js有哪些类型

JavaScript的类型分为两大类，一类是原始类型，一类是复杂(引用）类型。

原始类型:

* boolean
* null
* undefined
* number
* string
* symbol

复杂类型：

* object

还有一个没有正式发布但即将被加入标准的原始类型BigInt

## 为什么会有BigInt的提案

JavaScript中Number.MAX_SAFE_INTEGER表示最大安全数字,计算结果是9007199254740991，即在这个数范围内不会出现精度丢失（小数除外）。

但是一旦超过这个范围，js就会出现计算不准确的情况，这在大数计算的时候不得不依靠一些第三方库进行解决，因此官方提出了BigInt来解决此问题。

## null与undefined的区别是什么

null表示为空，代表此处不应该有值的存在，一个对象可以是null，代表是个空对象，而null本身也是对象。

undefined表示『不存在』，JavaScript是一门动态类型语言，成员除了表示存在的空值外，还有可能根本就不存在（因为存不存在只在运行期才知道），这就是undefined的意义所在。

## 隐式类型转换的原理是什么

1. 如果变量为字符串，直接返回.
2. 如果!IS_SPEC_OBJECT(x)，直接返回.
3. 如果IS_SYMBOL_WRAPPER(x)，则抛出异常.
4. 否则会根据传入的hint来调用DefaultNumber和DefaultString，比如如果为Date对象，会调用DefaultString.
5. DefaultNumber：首先x.valueOf，如果为primitive，则返回valueOf后的值，否则继续调用x.toString，如果为primitive，则返回toString后的值，否则抛出异常
6. DefaultString：和DefaultNumber正好相反，先调用toString，如果不是primitive再调用valueOf.

({}) + 1（将{}放在括号中是为了内核将其认为一个代码块）会输出啥？可能日常写代码并不会这样写，不过网上出过类似的面试题。

加操作只有左右运算符同时为String或Number时会执行对应的%_StringAdd或%NumberAdd，下面看下({}) + 1内部会经过哪些步骤：

{}和1首先会调用ToPrimitive {}会走到DefaultNumber，首先会调用valueOf，返回的是Object {}，不是primitive类型，从而继续走到toString，返回[object Object]，是String类型 最后加操作，结果为[object Object]1

## 谈谈你对原型链的理解

这个问题关键在于两个点，一个是原型对象是什么，另一个是原型链是如何形成的

### 原型对象

绝大部分的函数(少数内建函数除外)都有一个prototype属性,这个属性是原型对象用来创建新对象实例,而所有被创建的对象都会共享原型对象,因此这些对象便可以访问原型对象的属性。

例如hasOwnProperty()方法存在于Obejct原型对象中,它便可以被任何对象当做自己的方法使用

```js
 var person = {
    name: "Messi",
    age: 29,
    profession: "football player"
  };
console.log(person.hasOwnProperty("name")); //true
console.log(person.hasOwnProperty("hasOwnProperty")); //false
console.log(Object.prototype.hasOwnProperty("hasOwnProperty")); //true
```

由以上代码可知,hasOwnProperty()并不存在于person对象中,但是person依然可以拥有此方法.

所以person对象是如何找到Object对象中的方法的呢?靠的是原型链。

### 原型链

原因是每个对象都有 __proto__ 属性，此属性指向该对象的构造函数的原型。

对象可以通过 __proto__与上游的构造函数的原型对象连接起来，而上游的原型对象也有一个__proto__，这样就形成了原型链。

![JS原型链](/img/js-prototype.png)

## 聊一聊如何在JavaScript中实现不可变对象

1. 深克隆，但是深克隆的性能非常差，不适合大规模使用
2. Immutable.js，Immutable.js是自成一体的一套数据结构，性能良好，但是需要学习额外的API
3. immer，利用Proxy特性，无需学习额外的api，性能良好

## JavaScript的基本类型和复杂类型是储存在哪里的

基本类型储存在栈中，但是一旦被闭包引用则成为常住内存，会储存在内存堆中。

复杂类型会储存在内存堆中。