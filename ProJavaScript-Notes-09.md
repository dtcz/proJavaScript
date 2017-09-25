---
title: 重读 JS 高程（Chapter12）
tags: javascript
category: javascript
date: 2017-09-08 08:52:33
---


## Chapter12. DOM2 和 DOM3

### DOM 变化

DOM2 级和 3 级的目的在扩展 DOM API，以满足操作 XML 的所有需求，同时提供更好的错误处理及特性检测能力。

#### 针对XML命名空间的变化

**Node类型的变化**

DOM2 级中，Node类型包含下列特定于命名空间的属性。

- `localName`：不带命名空间前缀的节点名称。
- `namespaceURI`：命名空间 URI，未指定则为`null`。
- `prefix`：命名空间前缀，未指定则为`null`。

DOM3 级

- `isDefaultNamespace(namespaceURI)`：指定的 namespaceURI 在当前的默认命名空间的情况下返回true。
- `lookupNamespaceURI(prefix)`：返回给定 prefix 的命名空间。
- `lookupPrefix(namespaceURI)`：返回给定 namespaceURI 的前缀。

**Document 类型的变化**

DOM2 级中与命名空间相关的方法：

- `createElementNS(namespaceURI, tagName)`：使用给定的 tagName 创建一个属于命名空间 namespaceURI 的新元素。
- `createAttributeNS(namespaceURI, attributeName)`：使用给定的 attributeName 创建一个属于命名空间 namespaceURI 的新元素。
- `getElementBytagNameNS(namespaceURI, tagName)`：返回属于命名空间 namespaceURI 的 tagName 元素的 NodeList。

<!-- more -->

**Element 类型的变化**

“DOM2级核心中”有关 Element 的变化，主要涉及操作特性。新增方法如下：

- `getAttributeNS(namespaceURI, localName)`：取得属于命名空间 namespaceURI 的且名为 localName 的特性。
- `getAttributeNodeNS(namespaceURI, localName)`：取得属于命名空间 namespaceURI 的且名为 localName 的特性节点。
- `getElementsByTagNameNS(namespaceURI, localName)`：返回属性命名空间 namespaceURI 且名为 tagName 元素的 NodeList。
- `hasAttributeNS(namespaceURI, localName)`：确定当前元素是够有一个名为 localName 的特性，该特性的命名空间是 namespaceURI。
- `removeAttributeNS(namespaceURI, localName)`：删除属性命名空间 namespaceURI 且名为 localName 的特性。
- `setAttributeNS(namespaceURI, qualifiedName, value)`：设置属于命名空间 namespaceURI 的特性节点。
- `setAttributeNodeNS(attrNode)`：设置属于命名空间 namespaceURI 的特性节点。

**NamedNodeMap 类型的变化**

- `getNamedItemNS(namespaceURI, localName)`：取得属于命名空间 namespaceURI 且名为 localName 的项。
- `removeNamedItemNS(namespaceURI, localName)`：移除属于命名空间 namespaceURI 且名为 localName 的项。
- `setNamedItemNS(node)`：添加 node，这个节点已事先指定了命名空间信息。

#### 其他方面的变化

**DocumentType 类型的变化**

新增 3 个属性：`publicId`、`systemId` 和 `internalSubset`。

前两个表示文档类型声明中的两个信息段，最后一个属性`internalSubset`用于访问包含在文档类型声明中的额外定义。

**Document 类型的变化**

- `importNode()`：从一个文档中取得一个节点，然后将其导入到另一个文档，使其成为文档结构的一部分。每个节点都有一个`ownerDocument`属性，表示所属文档。`appendChild()`传入的节点如果属于不同文档，会导致错误。但是在调用`importNode()`时传入不同的文档节点则会返回一个新节点，这个新节点的所有权归当前文档所有。`importNode()`方法与 Element 的`cloneNode()`方法非常相似，接受两个参数：要复制的节点和一个表示是否复制子节点的布尔值，返回原节点的副本，但能够在当前文档中使用。
- `defaultView`：保存着一个指针，指向拥有给定文档的窗口（或框架）,IE不支持此属性，但有等价属性parentWindow。
- `createDocuemntType()`：创建一个新的 DocumentType 节点，接受三个参数：文档类型名称、publicId、systemId。
- `createDocument()`：创建新文档，接受三个参数：针对文档中元素的 namespaceURI、文档元素的标签名、新文档的文档类型。
- `document.implementation.createHTMLDocument()`：只接受 title 文本作为参数，返回新HTML文档（包括`<html>`、`<head>`、`<title>`和`<body>`）。

**Node类型的变化**

- `isSupported()`：与 DOM1 级中`document.implementation`引入的`hasFeature()`方法类似。用于确定当前节点具有什么能力，两个参数：特性名和特性版本号。
- `isSameNode()`：传入节点，与引用节点为同一个节点则返回`true`。
- `isEqualNode()`：传入节点，与引用节点相等返回`true`。
- `setUserData()`：将数据指定给节点。3 个参数：要设置的键，实际的数据和处理函数。

处理函数接受 5 个参数：表操作类型的数值（1：复制；2：导入；3：删除；4：重命名）、数据键、数据值、源节点和目标节点。

``` javascript
var div = document.createElement("div");

div.setUserData("name", "Nicholas", fucntion(operation, key, value, src, dest){
    if(operation == 1){
        dest.setUserData(key, value, function(){});
    }
});

var newDiv = div.cloneNode(true);
alert(newDiv.getUserData("name")); //"Nicholas"
```

**框架的变化**

框架和内嵌框架分别用 HTMLFrameElement 和 HTMLIFrameElement 表示，它们在 DOM2 级中都有一个新属性`contentDocument`。此属性包含一个指针，指向表示框架内容的文档对象。

`contentDocument`属性是 Document 类型实例，因此可以像使用其他 HTML 文档一样使用它，包括所有属性和方法。IE8 之前不支持框架中的`contentDocument` `属性，但支持一个叫` `contentWindow` `属性，返回框架的window对象。访问内嵌框架文档对象兼容代码：

``` javascript
var iframe = document.getElementById("myIframe");
var iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
```

### 样式

#### 访问元素的样式

** `style`属性**

任何支持`style`特性的 HTML 元素在 JavaScript 中都有一个对应的`style`属性。这个`style`对象是 CSSStyleDeclaration 的实例，包含着通过`style`特性指定的所有样式信息，但不包含与外部样式表或嵌入式样式表经层叠而来的样式。

在`style`特性指定的任何 CSS 属性都将表现为这个`style`对象的相应属性。对于使用短划线的 CSS 属性名，必须转换成驼峰大小写形式才能通过 JavaScript 访问。**其中`float`属性不可直接使用。IE 用`styleFloat`访问，其他浏览器用`cssFloat`访问。**

如果没有为元素设置`style`特性，那`style`对象中可能会包含一些默认值，但这些值不反映元素正确的样式信息。

**DOM 样式属性和方法**

DOM2 级还为 style 对象定义了一些属性和方法。在提供特性值时也可修改样式。

- `cssText`：访问`style`特性中的 CSS 代码。
- `length`：应用给元素的 CSS 属性数量。
- `parentRule`：表示 CSS 信息的 CSSRule 对象。
- `getPropertyCSSValue(propertyName)`：返回包含给定属性值的 CSSValue 对象，含两个属性`cssText` 和 `cssValueType`。
- `getProperytPriority(propertyName)`：如果给定的属性使用了`!important`设置，则返回“important”，否则返回空字符串。
- `item(index)`：返回给定位置的 CSS 属性的名称。
- `removeProperty(propertyName)`：从样式中删除给定属性。
- `setProperty(propertyName, value, priority)`：将给定属性设置为相应的值，并加上优先权标志（“important”或者一个空字符串）。

设置`length`属性的目的，就是将其与`item()`方法配套使用，以便迭代元素中定义的 CSS 属性。设置`cssText`是为元素应用多项变化最快捷的方式，因此可以一次性地应用所有变化。

**计算的样式**

“DOM2 级样式”增强了`document.defaultView`，提供了`getComputedStyle()`方法，接受两个参数：要取得计算样式的元素和一个伪元素字符串（如`:after`）。如果不需要伪元素信息，第二个参数可以为`null`，返回一个 CSSStyleDeclaration 对象（与style属性类型相同），其中包含当前元素的所有计算的样式。

IE 不支持`getComputedStyle()`方法吗，但 IE 中有`style`特性的元素均有一个`currentStyle`属性（CSSStyleDeclaration的实例）包含当前元素全部计算后的样式。计算样式兼容代码：

``` javascript
var myDiv = document.getElementById("myDiv");
var computedStyle = document.defaultView.getComputedStyle(mydiv, null) || myDiv.currentStyle;

alert(computedStyle.width);
alert(computedStyle.height);
```

#### 操作样式表

**CSSStyleSheet 类型**

CSSStyleSheet 继承自 StyleSheet，后者可以作为一个基础接口来定义非 CSS 样式表。从 StyleSheet 接口继承而来的属性如下：

- `disabled`：表示样式表是否被禁用的布尔值。可读/写，设为`true`则禁用。
- `href`：如果样式表是通过`<link>`包含的，则是样式表的 URL，否则是`null`。
- `media`：当前样式表支持的所有媒体类型的集合。与所有 DOM 集合一样，有`length`属性和`item()`方法。
- `ownerNode`：指向拥有当前样式表的节点的指针。若通过`@import`导入则属性为`null`，IE不支持此属性。
- `parentStyleSheet`：当前样式表是通过`@import`导入的情况下，这个属性指向导入它的样式表的指针。
- `title`：`ownerNode`中`title`属性的值。
- `type`：表示样式表类型的字符串。对 CSS 样式表而言，这个字符串是"type/css"。

以上属性除`disabled`外，全部只读。

CSSStyleSheet 类型还支持以下属性和方法：

- `cssRules`：样式表中包含的样式规则的集合。IE 不支持，但有类似的`rules`属性。
- `ownerRule`：如果样式表是通过`@import`导入的，这个属性就是一个指针，指向表示导入的规则；否则为`null`。IE不支持这个属性。
- `deleteRule(index)`：删除 cssRules 集合中指定位置的规则。IE 不支持，但支持`removeRule()`方法。
- `insertRule(rule, index)`：向 cssRules 集合中指定的位置插入`rule`字符串。IE 不支持，但支持类似的`addRule()`方法。

应用于文档的所有样式表是通过`document.styleSheets`集合来表示的。通过这个集合的`length`属性可以获取文档中样式表的数量，而通过方括号语法或`item()`方法可以访问每一个样式表。

``` javascript
var sheet = null;

for(var i = 0, len = document.styleSheets.length; i < len; i++){
    sheet = document.styleSheets[i];
    alert(sheet.href);
}
```

可以直接通过`<link>`或`<style>`元素取得 CSSStyleSheet 对象。DOM 规定了一个包含 CSSStyleSheet 对象的属性，名叫`sheet`；IE 不支持，但 IE 支持类似的`styleSheet`属性。

``` javascript
function getStyleSheet(element){
    return element.sheet || element.styleSheet;
}
//取得第一个<link>元素引入的样式表
var link = document.getElementsByTagName("link")[0];
var sheet = getStylesheet(link);
```

**CSS 规则**

CSSRule 对象表示样式表中的每一条规则，是一个供其他多种类型继承的基类型，其中最常见的就是 CSSStyleRule 类型，表示样式信息。CSSStyleRule 对象包含下列属性：

- `cssText`：返回整条规则对应的文本，IE 不支持。
- `parentRule`：如果当前规则是导入的规则，这属性引用的就是导入规则，否则这个值为`null`。IE 不支持。
- `parentStyleSheet`：当前规则所属的样式表。IE不支持。
- `selectorText`：返回当前规则的选择符文本。
- `style`：一个 CSSStyleDeclaration 对象，可以通过它设置和取得规则中特定的样式值。
- `type`：表示规则类型的常量值。对于样式规则，这个值是 1。IE 不支持。

其中最常用的属性是`cssText`、`selectorText`和`style`。`cssText`属性与`style.cssText`属性类似但并不相同。前者包含选择符文本和围绕样式信息的花括号，后者只包含样式信息（类似于元素的`style.cssText`）。而`cssText`只读，`style.cssText`可读/写。

``` javascript
var sheet = document.styleSheets[0];
var rules = sheet.cssRules || sheet.rules;
var rule = rules[0];

rule.style.backgroundColor = "red";
alert(rule.style.backgroundColor);
```

**创建规则**

- `insertRule()`：接受两个参数：规则文本和表达在哪里插入规则的索引。//IE不支持

``` javascript
sheet.insertRule("body{background-color:silver}",0); //DOM方法。
```

- `addRule()`：IE中支持。两个必选参数：选择符文本和CSS样式信息。一个可选参数：插入规则的位置。//仅IE支持

``` javascript
sheet.addRule("body","background-color:silver",0);
```

跨浏览器兼容：

``` javascript
function insertRule(sheet, selectorText, cssText, position){
    if(sheet.insertRule){
        sheet.insertRule(selectorText + "{" + cssText + "}", position);
    }else if(sheet.addRule){
        sheet.addRule(selectorText, cssText, position);
    }
}
insertRule(document.styleSheets[0], "body", "background-color:red", 0);
```

当需要添加的规则多时，操作繁琐。建议采用动态加载样式表。

**删除规则**

``` javascript
sheet.deleteRule(0); //DOM方法，IE不支持
sheet.removeRule(0); //IE方法，两者都是传入要删除的规则的位置。
```

跨浏览器兼容：

``` javascript
function deleteRule(sheet, index){
    if(sheet.deleteRule){
        sheet.deleteRule(index);
    }else if(sheet.removeRule){
        sheet.removeRule(index);
    }
}
```

这种做法不是实际web开发中常见做法，删除规则会影响 CSS 层叠效果，慎用。

#### 元素大小

**偏移量（offset dimension）**

> 包括元素在屏幕上占用的所有可见的空间。元素的可见大小由其高度、宽度决定，包括所有内边距、滚动条和边框大小（注意，不包括外边距）

- `offsetHeight`：元素在垂直方向上占用的空间大小，以像素计。包括元素的高度、水平滚动条高度、上下边框高度。
- `offsetWidth`：元素在水平方向上占用的空间大小，以像素计。包括元素的宽度、垂直滚动条宽度、左右边框宽度。
- `offsetLeft`：元素的左外边框至包含元素的左内边框之间的像素距离。
- `offsetTop`：元素的上外边框至包含元素的上内边框之间的像素距离。

其中，`offsetLeft`和`offsetTop`属性与包含元素有关，包含元素的引用保存在`offsetParent`属性中。`offsetParent`属性不一定与`parentNode`的值相等。

这些偏移量属性是只读的，每次访问需要重新计算。若重复使用，应保存在局部变量中。

``` javascript
function getElementLeft(element){
    var actualLeft = element.offsetLeft;
    var current = element.offsetParent;
    
    while(current !== null){
        actualLeft += current.offsetLeft;
        current = current.offsetParent;
    }
  
    return actualLeft;
}

function getElementTop(element){
    var actualTop = element.offsetTop;
    var current = element.offsetParent;
    
    while(current !== null){
        actualTop += current.offsetTop;
        current = current.offsetParent;
    }
  
    return actualTop;
}
```

**客户区大小（client dimension）**

> 元素内容及其内边距所占空间的大小，滚动条占用空间不算在内。

- `clientWidth`：元素内容区宽度加上左右内边距宽度。
- `clientHeight`：元素内容区高度加上上下内边距高度。

确定浏览器视口大小可用`document.documentElement`或`document.body`（IE7 以前）。

``` javascript
function getViewport() {
    if (document.compatMode == "BackCompat") {
        return {
            width: document.body.clientWidth,
            height: document.body.clientheight
        };
    } else {
        return {
            width: document.documentElement.clientWidth,
            height: document.documentElement.clientHeight
        };
    }
}
```

**滚动大小（scroll dimension）**

>  指包含滚动内容的元素的大小。

- `scrollHeight`：在没有滚动条的情况下，元素内容的总高度。
- `scrollWidth`：在没有滚动条的情况下，元素内容的总宽度。
- `scrollLeft`：被隐藏在内容区域左侧的像素数。通过设置这个属性可以改变元素的滚动位置。
- `scrollTop`：被隐藏在内容区域上方的像素数。通过设置这个属性可以改变元素的滚动位置。

在确定文档的总高度时（包括基于视口的最小高度时），必须取得`scrollWidth`/`clientWidth`和`scrollHeight`/`clientHeight`中的最大值，保证跨浏览器。

``` javascript
var docHeight = Max.max(document.documentElement.scrollHeight, document.documentElement.clientHeight);
var docWidth = Max.max(document.documentElement.scrollWidth, document.docuemntElement.clientWidth);
```

注：对于运行在混杂模式下的 IE，则需要用`document.body`代替`document.documentElement`。

通过`scrollLeft`和`scrollTop`属性既可以确定元素当前滚动的状态，也可以设置元素滚动位置。

``` javascript
function scrollToTop(element) {
    if (element.scrollTop != 0) {
        element.scrollTop = 0;
    }
}
```

**确定元素大小**

`getBoundingClientRect()`：这个方法返回一个矩形对象，含4个属性：`left`、`top`、`right`和`bottom`。这些属性给出了元素在页面中相对于视口的位置。但 IE8 认为左上角坐标为 (2,2)，其他浏览器认为是 (0,0)。

``` javascript
function getBoundingClientRect(element) {
    var scrollTop = document.documentElement.scrollTop;
    var scrollLeft = document.documentElement.scrollLeft;

    if (element.getBoundingClientRect) {
        If(typeof arguments.callee.offset != "number") {
            var temp = document.createElement("div");
            temp.style.cssText = "position:absolute;left:0;top:0;";
            document.body.appendChild(temp);
            arguments.callee.offset = -temp.getBoundingClientRect().top - scrollTop;
            document.body.removeChild(temp);
            temp = null;
        }

        var rect = element.getBoundingClientRect();
        var offset = arguments.callee.offset;
        return {
            left: rect.left + offset,
            right: rect.right + offset,
            top: rect.top + offset,
            bottom: rect.bottom + offset
        };

    } else {
        var actualLeft = getElementLeft(element);
        var actualTop = getElementTop(element);

        return {
            left: actualLeft - scrollLeft,
            right: actualLeft + element.offsetWidth - scrollLeft,
            top: actualTop - scrollTop,
            bottom: actualTop + element.offsetHeight - scrollTop
        }
    }
}
```

### 遍历

“DOM2 级遍历和范围”模块定义了两个用于辅助完成顺序遍历DOM结构的类型：`NodeIterator`和`TreeWalker`。这两个类型能够基于给定的起点对 DOM 结构执行深度优先（depth-first）的遍历操作。

在与 DOM 兼容版本中可访问这些对象，IE 不支持遍历。检查浏览器对 DOM2 级遍历能力的支持：

``` javascript
var supportsTraversals = document.implementation.hasFeature("Traversal", "2.0");
var supportsNodeIterator = (typeof document.createNodeIterator == "function");
var supportsTreeWalker = (typeof document.createTreeWalker == "function");
```

#### NodeIterator

可以使用`document.createNodeIterator()`方法创建它的新实例。

接受 4 个参数：

- `root`：想要作为搜索起点的树中的节点。
- `whatToShow`：表示要访问哪些节点的数字代码。
- `filter`：是一个`NodeFilter`对象，或者一个表示应该接受还是拒绝某种特定节点的函数。
- `entityReferenceExpansion`：布尔值，表示是否要扩展实体引用。此参数在 HTML 页面中没有用，因为其中的实体引用不能扩展。

`whatToShow`参数是一个位掩码，其值以常量形式在 NodeFilter 类型中定义。

- `NodeFilter.SHOW_ALL`：显示所有类型的节点。

- `NodeFilter.SHOW_ELEMENT`：显示元素节点。

- `NodeFilter.SHOW_ATTRIBUTE`：显示特性节点。由于 DOM 结构原因，实际上不能使用这个值。

- `NodeFilter.SHOW_TEXT`：显示文本节点。

- `NodeFilter.SHOW_CDATA_SECTION`：显示 CDATA 节点。

- `NodeFilter.SHOW_ENTITY_REFERENCE`：显示实体引用节点。

- `NodeFilter.SHOW_ENTITYPE`：显示实体节点。

- `NodeFilter.SHOW_PROCESSING_INSTRUCTION`：显示处理指令节点。

- `NodeFilter.SHOW_COMMENT`：显示注释节点。

- `NodeFilter.SHOW_DOCUMENT`：显示文档节点。

- `NodeFilter.SHOW_DOCUMENT_TYPE`：显示文档类型节点。

- `NodeFilter.SHOW_DOCUMENT_FRAGMENT`：显示文档片段节点。

- `NodeFilter.SHOW_NOTATION`：显示符号节点。

除了`NodeFilter_SHOW_ALL`外，可以使用按位或操作符来组合多个选项。如下：

``` javascript
var whatToShow = NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_TEXT;
```

可以通过`createNodeIterator()`用`filter`参数自定义 NodeFilter 对象或一个过滤器函数。

每个 NodeFilter 对象只有一个方法，即`acceptNode()`，访问则放回`NodeFilter.FILTER_ACCEPT`，不访问则返回 `NodeFilter.FILTER_SKIP`。

迭代器：

``` javascript
var filter = {
    acceptNode: function (node) {
        return node.tagName.toLowerCase() == "p" ? NodeFilter.FILTER_ACCEPT : NodeFilter.FILTER_SKIP;
    }
};

var iterator = document.createNodeIterator(root, NodeFilter.SHOW_ELEMENT, filter, false);
```

`filter`参数也可以是一个与`acceptNode()`方法类似的函数。

``` javascript
var filter = function(node) {
    return node.tagName.toLowerCase() == "p" ? NodeFilter.FILTER_ACCEPT : NodeFilter.FILTER_SKIP;
};

var iterator = document.createNodeIterator(root, NodeFilter.SHOW_ELEMENT, filter, false);
```

`NodeIterator`类型主要两个方法：`nextNode()`和`previousNode()`。

``` javascript
var div = document.getElemetnById("div1");
var iterator = document.createNodeIterator(div, NodeFilter.SHOW_ELEMENT, null, false);
var node = iterator.nextNode();

while (node !== null) {
    alert(node.tagName);
    node = iterator.nextNode();
}
```

#### TreeWalker

`TreeWalker`是`NodeIterator`的一个更高级版本。除了包括`nextNode()`和`previousNode()`外，还有下列用于在不同方向上遍历 DOM 结构的方法：

- `parentNode()`：遍历到当前节点的父节点。
- `firstChild()`：遍历到当前节点的第一个子节点。
- `lastChild()`：便历到当前节点的最后一个子节点。
- `nextSibling()`：遍历到当前节点的下一个同辈节点。
- `previousSibling()`：遍历到当前节点的上一个同辈节点。

创建`TreeWalker`对象要使用`document.createTreeWalker()`方法，接受 4 个参数，与`document.createNodeIterator()`方法相同。

其中第 3 个参数`filter`返回值除了可以返回`NodeFilter.FITER_ACCEPT`和`NodeFilter.FILTER_SKIP`外，还可以使用`NodeFILTER_REJECT`（作用：跳过相应的节点及该节点的整个子树）。

`TreeWalker`强大之处在于能够在 DOM 结构中沿任何方向移动。遍历 DOM 树，即使不定义过滤器，也可以取得所有`<li>`元素。如下：

``` javascript
var div = document.getElementById("div1");
var walker = document.createTreeWalker(div, NodeFilter.SHOW_ELEMENT, null, false);

walker.firstChild();
walker.nextSibling();

var node = walker.firstChild();

while (node !== null) {
    alert(node.tagName);
    node = walker.nextSibling();
}
```

`TreeWalker`类型还有一个属性，名叫`currentNode`，表示任何遍历方法在上一次遍历中返回的节点。

``` javascript
var node = walker.nextNode();
alert(node === walker.currentNode);
walker.currentNode = document.body;
```

### 范围

> 通过范围（range）可以选择文档中的一个区域，而不必考虑节点的界限（选择在后台完成，对用户看不见）。在常规DOM操作中不能更有效地修改文档时，使用范围往往可以达到目的。

#### DOM中的范围

DOM2 级在 Document 类型中定义了`createRange()`方法。检测浏览器是否支持范围：

``` javascript
var supportsRange = document.implementation.hasFeature("Range", "2.0");
var alsoSupportsRange = (typeof document.createRange == "function");
```

创建DOM范围: 

``` javascript
var range = document.createRange();
```

每个返回有一个 Range 类型的实例表示，这个实例拥有很多属性和方法，下列属性提供了当前范围在文档中的位置信息：

- `startContainer`：包含范围起点的节点（即选区中第一个节点的父节点）
- `startOffset`：范围在`startContainer`中起点的偏移量。如`startContainer`是文本节点，则`startOffset`就是范围起点之前跳过的字符数量。否则，`startOffset`就是范围中第一个子节点的索引。
- `endContainer`：包含范围终点的节点（即选取最后一个节点的父节点）。
- `endOffset`：范围在`endContainer`中终点的偏移量。
- `commonAncestorContainer`：`startContainer`和`endContainer`共同的祖先节点在文档树中位置最深的那个。

**用 DOM 范围实现简单选择**

使用范围选择文档中的一部分，可以使用`selectNode()`和`selectNodeContents()`，这两个方法都接受一个参数，即一个 DOM 节点，然后用该节点中的信息来填充范围。其中，`selectNode()`方法选择整个节点，包括子节点；而`selectNodeContents()`方法则只选择节点的子节点。

``` html
<p id="p1"><b>Hello</b> world!</p>
```

``` javascript
var range1 = document.createRange(),
    range2 = document.createRange(),
    p1 = document.getElementById("p1");
range1.selectNode(p1);                 //<p id="p1"><b>Hello</b> world!</p>
range2.selectNodeContents(p1);         //<b>Hello</b>
```

调用`selectNode()`时，`startContainer`、`endContainer`和`commonAncestorContainer`都等于传入节点的父节点，也就是这个例子中的`document.body`。而`startOffset`属性等于给定节点在父节点的`childNodes`集合中的索引,`endOffset`等于`startOffset`加 1。

调用`selectNodeContents()`时，`startContainer`、`endContainer`和`commonAncestorContainer`都等于传入的节点，也就是这个例子中的`<p>`元素。而`startOffset`属性等于 0，`endOffset`等于子节点数量。

**更精细地控制范围**

- `setStartBefore(refNode)`：将范围的起点设置在`refNode`之前，因此`refNode`也就是范围选区中第一个子节点。`startContainer`设置为`refNode.parentNode`，将`startOffset`属性设置为`refNode`在其父节点的`childNodes`集合中的索引。
- `setStartAfter(refNode)`：将范围的起点设置在`refNode`之后，因此`refNode`也就不在范围内了，其下一个同辈节点才是范围选区中第一个子节点。`startContainer`设置为`refNode.parentNode`，将`startOffset`属性设置为`refNode`在其父节点的`childNodes`集合中的索引加 1。
- `setEndBefore(refNode)`：将范围的终点设置在`refNode`之前，因此`refNode`也就不在范围内了，其上一个同辈节点才是范围选区中最后一个子节点。`endContainer`设置为`refNode.parentNode`，将`endOffset`属性设置为`refNode`在其父节点的`childNodes`集合中的索引。
- `setEndAfter(refNode)`：将范围的终点设置在`refNode`之后，因此`refNode`就是范围选区中最后一个子节点。同时会将`endContainer`属性设置为`refNode.parentNode`，将`endOffset`属性设置为`refNode`在其父节点的`childNodes`集合中的索引加 1。

**用 DOM 范围实现复杂选择**

- `setStart()`：传入一个参照节点和一个偏移量值。参照节点是`startContainer`，而偏移量是`startOffset`。
- `setEnd()`：传入一参照节点和一个偏移量值。参照节点是成`endContainer`，而偏移量是`endOffset`。

可以用这两个方法模拟`selectNode()`和`selectNodeContents`。如下：

``` javascript
var range1 = document.createRange(),
    range2 = document.createRange(),
    p1 = document.getElementById("p1"),
    p1Index = -1,
    i, len;

for(i = 0, len = p1.parentNode.childNodes.length; i < len; i++){
    if(p1.parentNode.childNodes[i] == p1){
        p1Index = i;
        break;
    }
}

range1.setStart(p1.parentNode, p1Index);
range1.setEnd(p1.parentNode, p1Index + 1);
range1.setStart(p1, 0);
range1.setEnd(p1, p1.childNodes);
```

它们更胜一筹的地方在于能够选择节点的一部分，比如选择"Hello"的"o"到"world!"的"o"：

``` javascript
var p1 = document.getElementById("p1"),
    helloNode = p1.firstChild.firstChild,
    worldNode = p1.lastChild,
    range = document.createRange();

range.setStart(helloNode, 2);
range.setEnd(worldNode, 3);
```

**操作DOM范围中的内容**

- `deleteContents()`：从文档中删除范围包含的内容。
- `extractContents()`：从文档中删除范围包含的内容，并返回范围文档片段，可以把片段插入其他位置。
- `cloneContents()`：创建范围对象的一个副本，然后在文档的其他东方插入该副本。

**插入DOM范围中的内容**

- `insertNode()`：向范围选区的开始处插入一个节点。
- `surroundContents()`：环绕范围插入内容，接受一个参数，即环绕范围内容的节点。后台会执行如下步骤：
  - 提取出范围中的内容（类似执行`extractContent()`）
  - 将给定节点插入到文档中原来范围所在的位置上。
  - 将文档片段的内容添加到给定节点中。

``` javascript
var p1 = document.getElementById("p1"),
    helloNode = p1.firstChild.firstChild,
    worldNode = p1.lastChild,
    range = document.createRange();

range.selectNode(helloNode);

var span = document.createElement("span");
span.style.backgroundColor = "yellow";
range.surroundContents(span);

//<p><b style="background-color:yellow;">Hello</span></b> world!</p>
```

**折叠 DOM 范围**

所谓折叠范围，就是指范围中未选择文档的任何部分。`collapse()`方法：一个参数布尔值。`true`折叠到范围起点，`false`折叠到范围终点。可以用`collapsed`属性检查是否已经折叠。

``` javascript
range.collapse(true);  //折叠到起点
alert(range.collapsed);  //输出true
```

**比较 DOM 范围**

在有多个范围的情况下，可以使用`compareBoundaryPoints()`方法来确定这些范围是否有公共的边界（起点或终点）。两个参数：表示比较方式的常量和要比较的范围。比较方式常量值：

- `Range.START_TO_START(0)`：比较第一个范围和第二个范围的起点。
- `Range.START_TO_END(1)`：比较第一个范围的起点和第二个范围的终点。
- `Range.END_TO_END(2)`：比较第一个范围和第二个范围的终点。
- `Range.END_TO_STRAT(3)`：比较第一个范围的终点和第二个范围的起点

可能的返回值如下，如果第一个范围中的点位于第二范围中的点之前，返回 -1；如果两个点相等，返回 0；如果第一个范围中的点位于第二范围中的点之后，返回 1。

**复制 DOM 范围**

可以使用`cloneRange()`方法复制范围，这个方法会创建调用它的范围的一个副本。

``` javascript
var newRange = range.cloneRange();
```

**清理 DOM 范围**

调用`detach()`方法，从文档分离出范围。

``` javascript
range.detach();  //从文档中分离
range = null;    //解除引用
```

#### IE8 以及更早版本中的范围

IE8 及早期版本不支持 DOM 范围。支持类似的文本范围（text range）。文本范围处理的主要是文本（不一定是 DOM 节点），通过`<body>`、`<button>`、`<input>`和`<textarea>`等元素调用`createTextRange()`方法。

``` javascript
var range = document.body.createTextRange();
```

通过`document`创建的范围可以在页面中的任何地方使用。

**用 IE 范围实现简单选择**

- `findText()`方法：找到第一次出现的给定文本，并将范围移过来以环绕该文本，返回布尔值，表示是否找到文本。同样以下面 HTML 为例：

``` html
<p id="p1"><b>Hello</b> world!</p>
```

``` javascript
var range = document.body.createTextRange();
var found = range.findText("Hello");

alert(true);       //true
alert(range.text); //"Hello"
```

还可以为`findText()`传入另一个参数，即一个表示向哪个方向继续搜索的数值，负值表示应该即当前位置向后搜，正值则向前搜。

- `moveToElementText()`方法：类似 DOM 中`selectNode()`方法，接受一个 DOM 元素，并选择该元素的所有文本，包括 HTML 标签。范围可用`htmlText`属性取得范围所有文本，包括 HTML 标签。`parentElement()`方法与 DOM 的`commonAncestorContainer`属性类似。

**使用 IE 范围实现复杂选择**

在 IE 中创建复杂范围的方法，就是以特定的增量向四周移动范围，IE 提供了 4 个方法：`move()`、`moveStart()`、`moveEnd()`、`expand()`。都接受两个参数：移动单位和移动单位的数量。移动单位为以下一种字符串：

- "character"：逐个字符地移动。
- "word"：逐个单词（一系列非空格字符）地移动
- "sentence"：逐个句子（一系列句号、问好或感叹号结尾的字符）地移动
- "textedit"：移到当前范围选区的开始或结束位置。

这四种方法的作用：

- `moveStart()`：移动到范围起点；
- `moveEnd()`：移动到范围终点。
- `expand()`：将任何部分选择的文本全部选中。
- `move()`：首先会折叠当前范围（起点终点相等），然后将范围移动指定的单位数量。

**操作 IE 范围中的内容**

可通过`text`属性取得范围中的内容文本。要向范围中插入 HTML 代码，要用`pasteHTML()`方法。

**折叠 IE 范围**

`collapse()`方法：传入`true`折叠到起点，`false`折叠到终点。可以用`boundingWidth`属性是否等于 0，来检查折叠是否完毕。

``` javascript
range.collapse(true);
var isCollapse = (range.boundingWidth == 0);
```
**比较 IE 范围**

- `compareEndPoints()`方法：接受两个参数，比较的类型和要比较的范围。比较类型取值字符串："StartToStart"、"StartToEnd"、"EndToEnd"和"EndToStart"。如果第一个范围的边界位于第二个范围的边界前，返回 -1；相等返回 0；在后面返回 1。
- `isEqual()`：用于确定两个范围是否相等。
- `inRange()`：用于确定一个范围是否包含另一个范围。

**复制 IE 范围**

使用`duplicate()`方法，复制文本范围，返回原范围的一个副本，如下：

``` javascript
var newRange = range.duplicate();
```