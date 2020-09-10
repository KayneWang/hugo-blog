---
title: "Html"
date: 2019-09-13
lastmod: 2020-09-02T10:18:57+08:00
draft: false
keywords: []
description: "HTML 面试题"
tags: ["html"]
categories: ["html"]
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

HTML 基础面试题总结

<!--more-->

## Doctype 有什么作用

DOCTYPE 是 html5 标准网页声明，用来告知浏览器使用什么文档类型来解析，不同的渲染模式会影响对 CSS 和 JS 的解析。

文档解析类型：

* 怪异模式，如果没有声明文档类型浏览器会使用自己的怪异模式来解析渲染页面
* 标准模式，满足 W3C 标准的解析渲染模式
* IE8 还有一个近乎标准模式，但是已经被淘汰了

### 这三种模式的区别有什么

* 标准模式：按照 CSS 和 HTML 定义来渲染页面
* 怪异模式：会模拟更旧的浏览器行为
* 近乎标准模式：会实施一种单元格尺寸行为，与 IE7 的单元格布局类似。其它方面与标准模式相同

## HTML、XHTML、XML有什么区别

* HTML（超文本标记语言）：4.0 之前先有实现，再有标准，导致 HTML 非常混乱和松散
* XML（可扩展标记语言）：可用于存储数据和结构，与现在常用的 JSON 很类型，但 JSON 更轻量和高效
* XHTML（可扩展超文本标记语言）：基于上述两种标准而来，W3C 为了解决 HTML 混乱问题而制定，由此诞生了 html5

XHTML 中的 DTD 是类似于 \<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"\> 的形式,有严格版、过渡版、框架版等几个版本

## 什么是 data- 属性

html 的数据属性，可以在标准 html 元素中插入额外需要存储的信息并通过 js 来获取

```html
<article
  id="electriccars"
  data-columns="3"
  data-index-number="12314"
  data-parent="cars">
...
</article>
```

目前各大框架盛行，这种形式的使用方式已经不流行了。

## 对HTML语义化的理解

语义化是指使用恰当语义的html标签，让页面具有良好的结构与含义，比如 \<p\> 代表段落，\<article\> 代表文章等。

语义化的好处：

* 开发者友好：增强了代码可读性，对整体布局结构清晰，便于团队的开发和维护
* 机器友好：带有语义的文字表现力丰富，更适合搜索引擎的爬虫爬取有效信息，语义类还可以支持读屏软件，根据文章可以自动生成目录

不足：

* 功能性的 web 软件重要性大打折扣，比如一个按钮之类的，不需要语义化，也不需要 SEO

## HTML5 与 HTML4 的不同之处

* 新的文本声明方式
* 新的解析顺序：不再基于 SGML
* 新的标签：section, video, progress, nav, meter, time, aside, canvas, command, datalist, details, embed, figcaption, figure, footer, header, hgroup, keygen, mark, output, rp, rt, ruby, source, summary, wbr
* input 元素的新类型：date, email, url等等
* 新的属性：ping（用于a与area）, charset（用于meta）, async（用于script）
* 全域属性：id, tabindex, repeat
* 新的全域属性：contenteditable, contextmenu, draggable, dropzone, hidden, spellcheck
* 移除元素：acronym, applet, basefont, big, center, dir, font, frame, frameset, isindex, noframes, strike, tt

## 有哪些常用的 meta 标签

meta标签由name和content两个属性来定义，来描述一个HTML网页文档的元信息，例如作者、日期和时间、网页描述、关键词、页面刷新等，除了一些http标准规定了一些name作为大家使用的共识，开发者也可以自定义name。

* charset，用于描述HTML文档的编码形式

```html
<meta charset="UTF-8" >
```

* http-equiv，顾名思义，相当于http的文件头作用,比如下面的代码就可以设置http的缓存过期日期

```html
＜meta http-equiv="expires" content="Wed, 20 Jun 2019 22:33:00 GMT"＞
```

* viewport，移动前端最熟悉不过，Web开发人员可以控制视口的大小和比例

```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
```

* apple-mobile-web-app-status-bar-style,开发过PWA应用的开发者应该很熟悉，为了自定义苹果工具栏的颜色。

```html
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
```

## src 和 href 的区别

* src是指向外部资源的位置，指向的内容会嵌入到文档中当前标签所在的位置，在请求src资源时会将其指向的资源下载并应用到文档内，如js脚本，img图片和frame等元素。当浏览器解析到该元素时，会暂停其他资源的下载和处理，知道将该资源加载、编译、执行完毕，所以一般js脚本会放在底部而不是头部
* href是指向网络资源所在位置（的超链接），用来建立和当前元素或文档之间的连接，当浏览器识别到它他指向的文件时，就会并行下载资源，不会停止对当前文档的处理

## 知道 img 的 srcset 的作用是什么

可以设计响应式图片，我们可以使用两个新的属性srcset 和 sizes来提供更多额外的资源图像和提示，帮助浏览器选择正确的一个资源。

srcset 定义了我们允许浏览器选择的图像集，以及每个图像的大小。

sizes 定义了一组媒体条件（例如屏幕宽度）并且指明当某些媒体条件为真时，什么样的图片尺寸是最佳选择。

所以，有了这些属性，浏览器会：

* 查看设备宽度
* 检查 sizes 列表中哪个媒体条件是第一个为真
* 查看给予该媒体查询的槽大小
* 加载 srcset 列表中引用的最接近所选的槽大小的图像

> srcset提供了根据屏幕条件选取图片的能力

```html
<img src="clock-demo-thumb-200.png"
     alt="Clock"
     srcset="clock-demo-thumb-200.png 200w,
             clock-demo-thumb-400.png 400w"
     sizes="(min-width: 600px) 200px, 50vw">
```

## 还有哪一个标签能起到跟srcset相似作用

\<picture\> 元素通过包含零或多个 \<source\> 元素和一个 \<img\> 元素来为不同的显示/设备场景提供图像版本。浏览器会选择最匹配的子 \<source\> 元素，如果没有匹配的，就选择 \<img\> 元素的 src 属性中的URL。然后，所选图像呈现在 \<img\> 元素占据的空间中

> picture同样可以通过不同设备来匹配不同的图像资源

```html
<picture>
    <source srcset="/media/examples/surfer-240-200.jpg"
            media="(min-width: 800px)">
    <img src="/media/examples/painted-hand-298-332.jpg" />
</picture>
```

## script标签中defer和async的区别

* defer：浏览器指示脚本在文档被解析后执行，script被异步加载后并不会立刻执行，而是等待文档被解析完毕后执行。
* async：同样是异步加载脚本，区别是脚本加载完毕后立即执行，这导致async属性下的脚本是乱序的，对于script有先后依赖关系的情况，并不适用。

## 有几种前端储存的方式

cookies、localstorage、sessionstorage、Web SQL、IndexedDB

### 这些方式的区别是什么

* cookies： 在HTML5标准前本地储存的主要方式，优点是兼容性好，请求头自带cookie方便，缺点是大小只有4k，自动请求头加入cookie浪费流量，每个domain限制20个cookie，使用起来麻烦需要自行封装
* localStorage：HTML5加入的以键值对(Key-Value)为标准的方式，优点是操作方便，永久性储存（除非手动删除），大小为5M，兼容IE8+
* sessionStorage：与localStorage基本类似，区别是sessionStorage当页面关闭后会被清理，而且与cookie、localStorage不同，他不能在所有同源窗口中共享，是会话级别的储存方式
* Web SQL：2010年被W3C废弃的本地数据库数据存储方案，但是主流浏览器（火狐除外）都已经有了相关的实现，web sql类似于SQLite，是真正意义上的关系型数据库，用sql进行操作，当我们用JavaScript时要进行转换，较为繁琐。
* IndexedDB： 是被正式纳入HTML5标准的数据库储存方案，它是NoSQL数据库，用键值对进行储存，可以进行快速读取操作，非常适合web场景，同时用JavaScript进行操作会非常方便。
