---
title: 重读 JS 高程（Chapter10、Chapter11）
tags: javascript
category: javascript
date: 2017-09-02 10:30:01
---


## Chapter10. DOM

DOM 是针对 HTML 和 XML 文档的一个API。DOM 描绘了一个层次化的节点树，允许开发人员添加、移除和修改页面的某一部分。

### 节点层次

DOM 可以将任何 HTML 或 XML 文档描绘成一个由多层次节点构成的结构。

#### Node 类型

DOM1 级定义了一个 Node 接口，该接口将由 DOM 中的所有节点实现。JS 中所有节点类型都继承自 Node 类型，因此所有节点类型都共享着相同的基本属性和方法。

**`nodeType`**

每个节点都有一个`nodeType`属性，用于表明节点的类型，由 12 个数值常量表示。如下：

  - `Node.ELEMENT_NODE`(1)
  - `Node.ATTRIBUTE_NODE`(2)
  - `Node.TEXT_NODE`(3)
  - `Node.CDATA_SECTION_NODE`(4)
  - `Node.ENTITY_PEFERENCE`(5)
  - `Node.ENTITY_NODE`(6)
  - `Node.PROCESSING_INSTRUCTION_NODE`(7)
  - `Node.COMMENT_NODE`(8)
  - `Node.DOCUMENT_NODE`(9)
  - `Node.DOCUMENT_TYPE_NODE`(10)
  - `Node.DOCUMENT_FRAGMENT_NODE`(11)
  - `Node.NOTATION_NODE`(12)

因为 IE 没有公开 Node 类型构造函数，因此不能直接使用上述常量，需要用数值比较。

``` javascript
if(someNode.nodeType == 1){ //适用所有浏览器
}
```

<!-- more -->

**`nodeName`和`nodeValue`**

这两个属性的值完全取决于节点的类型。在使用这两个值以前，最好是像下面这样先检测一下节点类型。

``` javascript
if(someNode.nodeType == 1){
    value = someNode.nodeNmae;
}
```

**`childNodes`**

每个节点都有一个`childNodes`属性，其中保存着一个`NodeList`对象。`NodeList`是一种类数组对象，用于保存一组有序的节点。可以用方括号语法和`item()`方法。

``` javascript
var firstChild = someNode.childNodes[0];
var secondChild = someNode.childNodes.item(1);
var count = someNode.childNodes.length;
```

**其余属性**

- `parentNode`：指向文档树中的父节点。
- `previousSibling`,`nextSibling`：可以访问同一`childNodes`列表中的其他节点，列表中第一个节点的`previousSibling`属性值为`null`，列表中最后一个节点的`nextSibling`属性值为`null`。
- `firstChild`,`lastChild`：父节点的`firstChild`和`lastChild`指向`childNodes`列表第一个节点和最后一个节点。
- `ownerDocument`：指向表示整个文档的文档节点。

**操作的节点的方法**

- `appendChild(node)`：用于向`childNodes`列表末尾添加一个节点。如果传入的节点已经是文档的一部分，那结果就是将该节点从原来的位置转移到新位置。
- `insertBefore(newNode, node)`：把节点放在`childNodes`列表中的某个特定位置上，该方法接受两个参数：要插入的节点和作为参照的节点。
- `replaceChild(newNode, node)`：替换节点，该方法接受两个参数：要插入的节点和要替换的节点。要替换的节点将由这个方法返回并从文档树中移除，同时要插入的节点占据其位置。
- `removeChild(node)`：移除节点，被移除的节点成为返回值。
- `cloneChild(isDeepCopy)`：创建一个节点副本，该方法接受一个参数，表示是否执行深复制。
- `normalize()`：这个方法用来处理文档树的文本节点。这个方法会删除空文本节点，合并相邻的文本节点。

#### Document 类型

浏览器中，`document`对象是`HTMLDocument`的一个实例，表示整个 HTML 页面。Document 节点有下列特征：

- `nodeType`的值为 9。
- `nodeName`的值为 "#document"。
- `nodeValue`的值为`null`。

**文档（`document`）的子节点和属性**

- `documentElement`：文档中只包含一个子节点，即`<html>`元素。可以通过`document.documentElement`来访问这个元素。
- `body`：`document.body`直接指向`<body>`元素。
- `doctype`：可以通过`document.doctype`可以取得`<!DOCTYPE>`的引用。
- `title`：文档标题，`<title>`。
- `URL`：包含页面完整的 URL。 
- `domain`：包含页面的域名。
- `referrer`：保存着来源页面的 URL。

**查找元素**

- `getElementById(id)`：接受一个参数：要取得元素的 ID。如果找到相应的元素则返回该元素，如果不存在带有相应 ID 的元素，则返回`null`。
- `getElementsByTagName(tagName)`：接受一个参数：要取得元素的标签名。返回包含 0 个或多个元素的`NodeList`。在 HTML 文档中，这个方法会返回一个`HTMLColletion`对象，作为一个“动态”的集合，该对象与`NodeList`非常类似。可以使用`nameItem()`通过元素的`name`特性取得集合中的项。
- `getElementsByName(name)`：接受一个参数：要取得元素的`name`。返回带有给定`name`特性的所有元素。

**特殊集合**

- `document.anchors`：包含文档中所有带`name`特性的`<a>`元素。
- `document.forms`：包含文档中所有的`<form>`元素。
- `document.images`：包含文档总所有的`<img>`元素。
- `document.links`：包含文档红所有带`href`特性的`<a>`元素。

**DOM 一致性检测**

`document.implementation.hasFeature()`：这个方法接受两个参数：要检测的 DOM 功能的名称及版本号，如果支持给定的名称和版本的功能，则该方法返回`true`。可检测的功能：Core、XML、HTML、Views、StyleSheets、CSS、CSS2、Events、UIEvents、MouseEvents、MutationEvents、HTMLEvents、Range、Traversal、LS、LS-Async、Validation。

**文档写入**

- `document.write()`,`document.writeln()`：接受一个字符串参数，即要写入到输出流中的文本。
- `document.open()`,`document.close()`：分别用于打开和关闭网页的输出流。如果在页面加载期间使用`write()`和`writeln()`，则不需要用到这两个方法。

#### Element 类型

Element 类型用于表现 XML 或 HTML 元素，提供了对元素标签名、子节点及特性的访问。

- `nodeType`的值为 1。
- `nodeName`的值为元素的标签名。
- `nodeValue`的值为`null`。

可以用`nodeName`属性或`tagName`属性访问元素标签名，两个属性返回相同的值——大写的标签名。

**HTML 元素**

所有 HTML 元素都由 HTMLElement 类型表示，其属性如下：

- `id`：元素在文档中的唯一标识符。

- `title`：有关元素的附加说明信息，一般通过工具提示条显示出来。

- `lang`：元素内容的语言代码，很少使用。

- `dir`：语言的方向，值为"ltr"或"rtl"，很少使用。

- `className`：与元素的 class 特性对应，即为元素指定的 CSS 类。

**操作特性（Attribute）**

- `getAttribute()`：可以取得公认特性和自定义特性的值。任何元素的所有特性，都可以通过 DOM 元素本身的属性来访问。不过只有非自定义特性才会以属性的形式添加到 DOM 对象中。

``` html
<div id="myDiv" align="left" my_special_attribute="hello!"></div>
```

``` javascript
alert(div.id);                   // "myDiv"
alert(div.my_special_attribute); // undefined
alert(div.align);                // "left"
```

有两类特殊的特性，属性的值与通过`getAttribute()`返回的值并不相同。第一类是`style`特性，通过`getAttribute()`访问得返回 CSS 文本，通过属性访问会返回一个对象。第二类是`onclick`特性：通过`getAttribute()`访问返回代码字符串，通过属性访问返回一个函数。

一般通过属性取得特性，只有在取自定义特性值时，才用`getAttribute()`。

- `setAttribute()`：接受两个参数，要设置的特性名和值。如果特性已经存在，`setAttribute()`会以指定的值替换现有的值；如果特性不存在，`setAttribute()`则创建该属性并设置相应的值。该方法既可以操作HTML特性也可以操作自定义特性。（IE7 以前慎用）


- `removeAttribute()`：彻底删除元素的特性（IE6不支持）。

**`attributes`属性**

Element 类型是使用`attributes`属性的唯一一个 DOM 节点类型。此属性包含一个`NamedNodeMap`对象，也是“动态”集合。元素的每一个特性都由一个`Attr`节点表示，每个节点保存在`NamedNodeMap`对象中。

- `getNamedItem(name)`：返回返回`nodeName`属性等于`name`的节点
- `removeNamedItem(name)`：从列表中移除`nodeName`属性等于`name`的节点。
- `setNamedItem(node)`：向列表中添加节点，以节点的`nodeName`属性为索引。
- `item(pos)`：返回位于数字`pos`位置的节点。

**创建元素**

使用`document.createElement()`方法可以创建新元素。接受一个参数：要创建元素的标签名。在 IE 中`createElement()`可传入完整的元素标签，可以包含属性：

``` javascript
var div = document.createElement('<div id="myNewDiv" class="box"></div>');
```

这种方式有助于避开 IE7/IE6 中动态创建元素的某些问题：

- 不能设置动态创建的`<iframe>`元素的`name`特性。
- 不能通过表单的`reset()`方法重设动态创建的`<input>`元素。
- 动态创建的`type`特性值为"reset"的`<button>`元素重设不了表单。
- 动态创建的一批`name`相同的单选按钮彼此毫无关系。`name`值相同的一组单选按钮本来应该用于表示同一选项的不同值，但动态创建的一批这种单选按钮之间却没有这种关系。

``` javascript
if(client.browser.ie && client.brower.ie <= 7){
    var iframe = document.createElement('<iframe name = "myframe"></iframe>');
    var input = document.createElement('<input type="checkbox">');
    var button = document.createElement('<button type="reset"></button>');
    var radio1 = document.createElement('<input type="radio" name="choice" value="1"');
    var radio2 = document.createElement('<input type="radio" name="choice" value="2"');
}
```

**元素的子节点**

元素可以有任意数目的子节点和后代节点，元素的`childNodes`属性中包含了它的所有子节点：元素、文本节点、注释或处理指令。不同浏览器在看待这些节点方面存在显著的不同，以下面代码为例。

``` html
<ul id="myList">
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
</ul>
```

IE 会忽略空白符节点，所以`<ul>`元素会有 3 个`<li>`子节点。其他浏览器中，`<ul>`元素会有 7 个元素，包括 3 个`<li>`元素和 4 个文本节点。如果像下面这样把元素间的空白符删除，那么所有浏览器都会返回相同数目的子节点。

``` html
<ul id="myList"><li>Item 1</li><li>Item 2</li><li>Item 3</li></ul>
```

因此在遍历`childNodes`时，需要检查`nodeType`属性。

``` javascript
for(var i = 0, len = element.childNodes.length; i < len; i++){
	if(element.childNodes[i].nodeType == 1){
        //执行某些操作
	}
}
```

元素节点也支持`getElementsByTagName()`方法。

#### Text 类型

文本节点由 Text 类型表示，包含的是纯文本内容，不含HTML代码。

- `nodeType`的值为 3。
- `nodeName`的值为"#text"。
- `nodeValue`的值为节点所包含的文本。
- 不支持（没有）子节点。

**操作文本节点**

可以通过过`nodeValue`或`data`属性访问 Text 节点中包含的文本，这两个属性包含的值相同。

- `appendData(text)`：将 text 添加到节点的末尾。

- `deleteData(offset, count)`：从 offset 位置开始删除 count 个字符。

- `insertData(offset, text)`：在 offset 位置插入 text；

- `replaceData(offset, count, text)`：用 text 替换从 offset 指定的位置开始到 offset + count 为止处的字符串。

- `splitText(offset)`：从 offset 指定位置将当前文本节点分成两个文本节点。

- `substringData(offset, count)`：提取 offset 指定的位置开始到 offset + count 为止处的字符串。

文本节点有一个`length`属性，保存节点中字符的数目。而`nodeValue.length`和`data.length`中也保存着同样的值。修改文本节点值时字符串经HTML编码，特殊符号会被转义：

``` javascript
div.firstChild.nodeValue = "Some <strong>other</strong> message"; 
//输出Some &lt;strong&gt;other&lt;/strong&gt; message
```

**创建文本节点**

使用`document.createTextNode()`创建新文本节点。参数：要插入节点的文本（自动按 HTML 或 XML 格式编码）

**规范化文本节点**

DOM 操作中可向同一父元素插入多个文本节点，不过可能会造成混乱。可以通过 Node 类型继承的`normalize()`方法合并同一父元素上的所有文本节点。

**分割文本节点**

Text 类型方法提供了一个作用与`normalize()`相反的方法`splitText()`：传入一个位置参数，按揭指定的位置拆分。原来的文本节点包含前半部分，新文本节点将包含剩下的文本。分割文本节点是从文本节点中提取数据的一种常用 DOM 解析技术。

#### Comment 类型

- `nodeType`的值为 8。
- `nodeName`的值为"#comment"。
- `nodeValue`的值是注释的内容。
- 不支持（没有）子节点。

Comment 类型与 Text 类型继承自相同的基类，因此它拥有除`splitText()`之外的所有字符串操作方法。与 Text 类型相似，也可以通过`nodeValue`或`data`属性来取得注释的内容。可使用`document.createComment()`创建注释节点。

#### CDATASection 类型

- `nodeType`的值为 4。
- `nodeName`的值为"#cdata-section"。
- `nodeValue`的值是 CDATA 区域中的内容。
- 不支持（没有）子节点

CDATASection 类型只针对基于 XML 的文档，表示的是 CDATA 区域。CDATASection 类型继承自 Text 类型，拥有除`splitText()`之外所有方法。各大浏览器均无法正确解析这种类型。

#### DocumentType 类型

- `nodeType`的值为 10。
- `nodeName`的值为`doctype`的名称。
- `nodeValue`的值`null`。
- 不支持（没有）子节点。

#### DocumentFragment 类型

DOM 规定文档片段（document fragment）是一种“轻量级”的文档，可以包含和控制节点，但不会像完整的文档那样占用额外资源。

- `nodeType`的值为 11。
- `nodeName`的值为"#document-fragment"。
- `nodeValue`的值为`null`。

文档片段不能直接添加到文档中，但可以保存将来可能会添加到文档的节点。要创建文档片段，可以使用`document.createDocumentFragment()`方法，如下所示：

``` javascript
var fragment = document.createDocumentFragment();
var ul = document.getElementById("myList");
var li = null;

for(var i = 0; i < 3; i++){
    li = document.createElement("li");
    li.appendChild(document.createTextNode("Item" + (i + 1)));
    fragment.appendChild(li);
}

ul.appendChild(fragment);
```

#### Attr 类型

- `nodeType`的值为 2。
- `nodeName`的值是特性的名称。
- `nodeValue`的值是特性的值。

Attr 对象 3 个属性：

- `name`：特性名称（如 nodeName）。
- `value`：特性的值（如 nodeValue）。
- `specified`：布尔值，用以区别特性是在代码中指定的还是默认的。

不建议直接访问特性节点。

### DOM 操作技术

#### 动态脚本

**外部文件**

``` javascript
function loadScript(url){
    var script = document.createElement("script");
    script.type = "text/javascript";
    script.src = url;
    document.body.appendChild(script);
}

loadScript('client.js');
```

**行内方式**

IE 将`<script>`视为一个特殊的节点，不允许访问其子节点。使用`script`的`text`属性可以解决这个问题。

``` javascript
function loadScriptString(code){
    var script = document.createElement("script");
    script.type = "text/javascript";
    try {
        script.appendChild(document.createTextNode(code));
    } catch(ex) {
        script.text = code; //兼容IE
    }
    document.body.appendChild(script);
}

loadScriptString('function sayHi(){alert("Hi");}');
```

#### 动态样式

**外部链接**

``` javascript
function loadStyles(url){
    var link = document.createElement("link");
    link.rel = "stylesheet";
    link.type = "text/css"; 
    link.href = url;
    var head = document.getElementsByTagName("head")[0];
    head.appendChild(link);
}

loadStyles('styles.css');
```

**使用`<style>`元素包含嵌入式 CSS**

IE 将`<style>`视为一个特殊的、与`<script>`类似的节点，不允许访问其子节点。解决 IE 这个问题的办法就是使用`styleSheet.cssText`属性。

``` javascript
fucnction loadStyleString(css){
    var style = document.createElement("style");
    style.type = "text/css";
    try {
        style.appendChild(document.createTextNode(css));
    } catch(ex) {
        style.styleSheet.cssText = css;
    }
    var head = document.getElementsByTagName("head")[0];
    head.appendChild(style);
}

loadStyleString("body{background-color: red;}");
```

#### 操作表格

**`<table>`元素的属性和方法**

- `caption`：保存着对`<caption>`元素（如果有）的指针；
- `tBodies`：是一个`<tobdy>`元素的 HTMLCollection;
- `tFoot`：保存着对`<tfoot>`元素（如果有）的指针；
- `tHead`：保存着对`<thead>`元素（如果有）的指针；
- `rows`：是一个表格中所有行的 HTMLCollection；
- `createTHead()`：创建`<thead>`元素，将其放到表格中，返回引用。
- `createTFoot()`：创建`<tfoot>`元素，将其放到表格中，返回引用。
- `createCaption()`：创建`<caption>`元素，将其放到表格中，返回引用。
- `deleteTHead()`：删除`<thead>`元素。
- `deleteTFoot()`：删除`<tfoot>`元素。
- `deleteCaption()`：删除`<caption>`元素。
- `deleteRow(pos)`：删除指定位置的行。
- `insertRow(pos)`：向 rows 集合中的指定位置插入一行。

**`<tbody>`元素的属性和方法**

- `rows`：保存着`<tbody>`元素中行的 HTMLCollection。
- `deleteRow(pos)`：删除指定位置的行；
- `insertRow(pos)`：向 rows 集合中的指定位置插入一行，返回插入行的引用。

**`<tr>`元素的属性和方法**

- `cells`：保存着`<tr>`元素中单元格的 HTMLCollection；
- `deleteCell(pos)`：删除指定位置的单元格。
- `insertCell(pos)`：向 cells 集合中指定位置插入一单元格，返回插入的单元格引用。

#### 使用 NodeList

NodeList，NamedNodeMap（Element 类型中的`attribute`属性）和 HTMLCollection，这三个集合都是“动态的”，每当文档结构发生变化时，它们都会得到更新。

尽量减少访问 NodeList 的次数。因每次访问 NodeList，都会运行一次基于文档的查询。可考虑将从 NodeList 取得的值缓存起来。

## Chapter11. DOM扩展

###  选择符 API

可以通过 Document 和 Element 类型的实例调用它们。前两个方法兼容 IE8+。

- `querySelector()`：接受一个 CSS 选择符，返回与该模式匹配的第一个元素，如果没有，返回`null`。


- `querySelectorAll()`：接受一个 CSS 选择符，返回与该模式匹配的所有元素，是一个 NodeList 实例。


- `matchesSelector()`：接受一个 CSS 选择符，如果调用元素与该选择符匹配，返回`true`；否则返回`false`。

### 元素遍历

对于元素间的空格，IE9 及之前版本不会返回文本节点，而其他浏览器都会返回文本节点。为了弥补差异，Element Traversal 规范新添加了五个属性：

- `childElementCount`：返回子元素（不包括文本和注释节点）的个数。
- `firstElementChild`：指向第一个子元素。
- `lastElementChild`：指向最后一个子元素。
- `previousElementSibling`：指向前一个同辈元素。
- `nextElementSibling`：指向后一个同辈元素。

### HTML5

#### 与类相关

**`getElementsByClassName()`**

可以通过`document`对象及所有的 HTML 元素调用该方法，该方法接受一个参数，即一个包含一或多个类名的字符串，返回带有指定类的所有元素的 NodeList。传入多个类名时，类名先后顺序不重要。IE9+ 支持。

**`classList`属性**

操作类型时需要通过`className`属性添加、删除和替换类名。由于`className`是个字符串，所以操作起来比较麻烦。因此，HTML5 新增了一种操作类名的方式`classList`，它是新集合类型 DOMTokenList 的实例。定义如下语法：

- `add(value)`：将给定字符串值添加到列表中。
- `contains(value)`：表示列表中是否存在给定的值。
- `remove(value)`：从列表中删除给定字符串。
- `toggle(value)`：如果列表中已经存在给定值，删除它；如果没有，添加它。

IE10+ 支持

#### 焦点管理

- `document.activeElement`：这个属性始终会引用 DOM 中当前获得了焦点的元素。默认情况下，文档刚加载完时，该属性值为`document.body`。
- `document.hasFocus()`：这个方法用于确定文档是否获得了焦点，可以知道用户是不是正在与页面交互。

#### HTMLDocument 的变化

**`readyState`正式加入标准**

`document`的`readyState`属性有两个可能的值：

- `loading`：正在加载文档
- `complete`：已经加载完文档

**兼容模式**

`document`对象的`compatMode`属性，标准模式的值为`CSS1Compat`，混杂模式的值为`BackCompact`。

**`head`**

使用`document.head`获取`<head>`元素。

#### 字符集属性

- `document.charset`：表示文档中实际使用的字符集，也可以用来指定新的字符集。
- `document.defaultCharset`：默认字符集。

#### 自定义数据属性

HTML5 规定可以为元素添加非标准的属性，但要添加`data-`前缀。

``` html
<div id = "myDiv" data-appId = "12345" data-myname="Nicholas"></div>
```

添加了自定义属性后，可以通过元素的`dataset`属性来访问自定义属性的值。`dataset`属性的值是 DOMStringMap 的一个实例，也就是一个名值对映射。实例代码如下：

``` javascript
var div = docuemnt.getElementById('myDiv');
var appId = div.dataset.appId;
var myName = div.dataset.myName;

div.dataset.appId = 23456;
div.dataset.myName = "Michael";
```

#### 插入标记

**`innerHTML`**

读模式下，返回调用元素所有子节点的 HTML 标记；写模式下，会按照指定的值创建新的 DOM 树，并替换调用元素的所有子节点。其中的所有标签都会按照浏览器处理 HTML 的标准方式转换为元素（会转义符号）。

`innerHTML`限制：插入`<script>`元素不会被执行，插入`<style>`也有类似问题。因为`<script>`和`<style>`是无作用域元素，浏览器会在字符串解析前删除掉它们。一般使用如下方式添加：

``` javascript
div.innerHTML = '<input type="hidden"><script defer>alert("hi!");<\/script>'
```

并不是所有元素都有`innerHTML`属性，不支持的元素有：`<col>`、`<colgroup>`、`<frameset>`、`<head>`、`<html>`、`<style>`、`<table>`、`<tbody>`、`<thead>`、`<tfoot>`、`<title>`和`<tr>`。

**`outerHTML`**

与`innerHTML`基本相同，区别在于写入信息时，会替换给定节点。

**`insertAdjaceHTML()`**

- `beforebegin`：在当前元素之前插入一个紧相邻的同辈元素；
- `afterbegin`：在当前元素之下插入一个新的子元素或者在第一个子元素之前再插入新的子元素；
- `beforeend`：在当前元素之下插入一个新的子元素或者在最后一个子元素之后再插入新的子元素；
- `afterend`：在当前元素之后插入一个紧相邻的同辈元素。

**内存与性能问题**

使用`innerHTML`和`outerHTML`替换子节点可能会导致浏览器的内存问题。被删除的元素设置的事件处理程序或具有值为 JS 对象的属性，依旧存在于内存中。插入大量新 HTML 时，使用`innerHTML`要比通过多次 DOM 操作有效率得多。

#### `scrollIntoView()`方法

可以在所有 HTML 元素中调用，通过滚动浏览器窗口或某个容器元素，调用元素就可以出现在视口中。如果给这个方法传入`true`，或者不传参数，那么窗口滚动之后会让调用元素的顶部与视口顶部尽可能平齐。如果传入`false`，调用元素会尽可能全部出现在视口中，不过顶部不一定平齐。

### 专有扩展

#### 文档模式

IE8 引入的新概念，可以用`document.documentMode`访问到，它会访问使用的文档模式的版本号。

到 IE9 一共有 4 种文档模式：IE5、IE7、IE8、IE9。版本号：Edge、EmulateIE9、EmulateIE8、EmulateIE7、9、8、7、5。渲染页面设置：

``` html
<meta http-equiv="X-UA-Compatible" content="IE=Edge">
```

#### `children`属性 

children 属性只包含元素中同样还是元素的子节点，是一个 HTMLCollection 对象。支持 IE9+。

#### `contains()`方法

用于查找某个节点是不是另一个节点的后代。应该在搜索开始的节点上调用，传入要检测的后代节点作参数，返回`true`则传入节点作为当前节点后代。

``` javascript
alert(document.documentElement.contains(document.body)); // true
```

使用 DOM3 级的`compareDocumentPosition()`方法也能确定两个节点间关系，返回一个表示该关系的位掩码（bitmask）。下表列出了这个位掩码的值：

|  掩码  | 节点关系                      |
| :--: | :------------------------ |
|  1   | 无关（给定的节点不在当前文档中）          |
|  2   | 居前（给定的节点在 DOM 树中位于参考节点之前） |
|  4   | 居后（给定的节点在 DOM 树中位于参考节点之后） |
|  8   | 包含（给定节点是参考节点的祖先）          |
|  16  | 被包含（给定的节点始参考节点的后代）        |

``` javascript
var result = document.documentElement.compareDocumentPosition(document.body);
alert(!!result & 16); //true
```

兼容各浏览器代码：

``` javascript
function contains(refNode, otherNode) {
    //在webkit版本小于552的Safari浏览器中，contains()不能正常使用
    if (typeof refNode.contains == "function" && (!client.engine.webkit || client.engine.webkit >= 552)) {
        return refNode.contains(otherNode);
    } else if (tyepof refNode.compareDocumentPosition == "function") {
        return !!(refNode.compareDocumentPosition(otherNode) & 16);
    } else {
        var node = otherNode.parentNode;
        do {
            if (node === refNode) {
                return true;
            } else {
                node = node.parentNode;
            }
        } while (node !== null);
        return false;
    }
}
```

#### 插入文本

**`innerText`属性**

可读取节点文档树中由浅入深的所有文本。

通过`innerText`属性设置节点文本，会移除先前存在的所有子节点，完全改变了 DOM 树。

赋给此属性的文本将自动进行 HTML 编码。

Firefox 不支持`innerText`，但支持作用类似的`textContent`属性。

兼容代码：

``` javascript
function getInnerText(element){
    return (typeof element.textContent == "string") ? element.textContent : element.innerText;
}

function setInnerText(element, text){
    if(typeof element.textContent == "string"){
        element.textContent = text;
    } else {
        element.innerText = text;
    }
}
```

**`outerText`属性**

与`innerText`基本相同，区别在于写入信息时，新的文本节点会替换调用`outerText`的元素。

Firefox 不支持，由于会替换元素，所以不建议使用。

#### 滚动

滚动方法都是对 HTMLElement 类型的扩展，可以在所有元素节点上调用。

- `scrollIntoView(bool)`：滚动浏览器窗口或容器元素，以便在视口（viewport）中看到元素。
- `scrollIntoViewIfNeeded(bool)`：只在当前元素在视口中不可见的情况下，才滚动浏览器窗口或容器元素。建议传入`true`参数，表示尽量显示在视口中部（Safari 和 Chorme实现）。
- `scrollByLines(lineCount)`：将元素的内容滚动指定的行高，`lineCount`可为正、负数。（safari 和 Chrome可用）。
- `scrollByPages(pageCount)`：将元素的内容滚动指定的页面高度，具体高度由元素高度决定。（safari 和 Chrome可用）。