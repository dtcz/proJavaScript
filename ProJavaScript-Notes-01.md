---
title: '重读 JS 高程（Chapter01、Chapter02）'
date: 2017-07-28 14:42:02
tags: javascript
category: javascript
---

## Chapter01. JavaScript 简介

一个完整的 JavaScript 由下列三个不同的部分组成：

- 核心（ECMAScript）
- 文档对象模型（DOM）
- 浏览器对象模型（BOM）

### ECMAScript

ECMA-262 的第 5 版是 JS 的第一个稳定版本，得到了各浏览器厂商的支持。它规定了语言的下列组成部分：

- 语法
- 类型
- 语句
- 关键字
- 保留字
- 操作符
- 对象

<!--more-->

### DOM

文档对象模型是针对 XML 但经过扩展用于 HTML 的 API 。DOM 把整个页面映射为一个多层次节点结构。HTML 或 XML 页面中的每个组成部分都是某种类型的节点，这些节点又包含着不同类型的数据。

DOM级别：

- DOM Level 1

  - DOM Core
  - DOM HTML

- DOM Level 2

  - DOM Views
  - DOM Events
  - DOM Style
  - DOM Traversal and Range

- DOM Level 3

  - 扩展DOM Core，支持 XML 1.0 规范
  - DOM Load and Save
  - DOM Validation

- 其他 DOM 标准

  - SVG 1.0
  - MathML 1.0
  - SMIL（同步多媒体集成语言）

### BOM

原则上讲，BOM只处理浏览器窗口和框架，但下面一些针对浏览器的 JS 扩展也被看做是BOM的一部分：

- 弹出新浏览器窗口
- 移动、缩放和关闭浏览器窗口
- 提供浏览器详细信息的 navigator 对象
- 提供浏览器所加载页面的详细信息的 location 对象
- 提供用户显示器分辨率详细信息的 screen 对象
- 对 cookies 的支持
- 像 XMLHttpRequest 和 ActiveXObject 这样的自定义对象



## Chapter02. 在 HTML 中使用 JavaScript 

###  script 元素

使用`<script>`元素的方式有两种：直接在页面中嵌入 JavaScript 代码和包含外部 JavaScript 文件。`<script>`定义了下列6种属性：

- async：可选。 表示应该立即下载脚本，但不应妨碍页面中的其他操作，比如下载其他资源或等待加载其他脚本。只对外部脚本文件有效。
- charset：可选。表示通过 src 属性指定代码的字符集。由于大多数浏览器会忽略它的值，因此这个属性值很少有人用。
- defer：可选。表示脚本可以延迟到文档完全被解析和显示之后再执行。只对外部脚本文件有效。IE7 及更早版本对嵌入脚本也支持这个属性。
- src：可选。表示包含要执行代码的外部文件。
- type：可选。可以当作 language 的替代属性；表示编写代码使用的脚本语言的内容类型（也称 MIME 类型）。这个属性一般指定为 text / javascript ，不过这个属性不是必需的，如果没有指定这个属性，则默认值仍为text / javascript。

### 标签的位置

一般把全部 JavaScript 引用放在 `<body>` 元素中页面内容的后面。这样，在解析包含的 JavaScript 代码之前，页面的内容将完全呈现在浏览器中。

### 延迟脚本

在 `<script>`元素中设置 defer 属性，相当于告诉浏览器立即下载，但延迟执行。defer 属性只用于外部脚本文件。HTML5 规范要求脚本按照它们出现的先后顺序执行，而这两个脚本会先于 DOMContentLoaded 事件触发前执行。在现实中，延迟脚本不一定会按照顺序执行，也不一定会先于 DOMContentLoaded  事件触发前执行，因此最好只包含一个延迟脚本。

### 异步脚本

设置 async 属性与 defer 类似，只用于外部脚本文件，并告诉浏览器立即下载，但与 defer 不同的是，异步脚本不一定会按照顺序执行。可能会先于 DOMContentLoaded  事件触发前执行。

### 文档模式

IE5.5 引入了文档模式的概念，而这个概念是通过使用文档类型（doctype）切换实现的。 最初的两种文档模式是：混杂模式（quirks mode）和标准模式（standards mode）。混杂模式会让IE的行为与（包含非标准特性的）IE5相同，而标准模式则让IE的行为更接近标准行为。

### 标准模式

``` html
<!-- HTML 4.01 严格型 -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

<!-- XHTML 1.0 严格型 -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

<!-- HTML 5 -->
<!DOCTYPE html>
```

### 准标准模式

``` html
<!-- HTML 4.01 过渡型 -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">

<!-- HTML 4.01 框架集型 -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN"
"http://www.w3.org/TR/html4/frameset.dtd">

<!-- XHTML 1.0 过渡型 -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<!-- XHTML 1.0 框架集型 -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">
```
