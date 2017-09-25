---
title: 重读 JS 高程（Chapter14）
tags: javascript
category: javascript
date: 2017-09-17 09:46:39
---


## Chapter14. 表单脚本

JavaScript 最早的应用就是分担服务器处理表单的责任，打破处处依赖服务器的局面。

### 表单基础知识

在 HTML 中，表单是由`<form>`元素来表示的，而在 JavaScript 中，表单对应的则是 HTMLFormElement 类型。HTMLFormElement 继承了 HTMLElement，因而与其他 HTML 元素具有相同的默认属性。不过， HTMLFormElement 也有它自己下列独有的属性和方法。 

- `acceptCharset`：服务器能够处理的字符集；等价于 HTML 中的`accept-charset`特性。 
- `action`：接受请求的 URL；等价于 HTML 中的`action`特性。 
- `elements`：表单中所有控件的集合（HTMLCollection）。 
- `enctype`：请求的编码类型；等价于 HTML 中的`enctype`特性。 
- `length`：表单中控件的数量。 
- `method`：要发送的 HTTP 请求类型，通常是"get"或"post"；等价于 HTML 的`method`特性。 
- `name`：表单的名称；等价于 HTML 的`name`特性。 
- `reset()`：将所有表单域重置为默认值。 
- `submit()`：提交表单。 
- `target`：用于发送请求和接收响应的窗口名称；等价于 HTML 的`target`特性。

除了通过`getElementById()`获取表单元素，也可以使用`document.forms`取得页面中所有表单。

#### 提交表单

用户单击提交按钮或图像按钮时，就会提交表单。使用`<input>`或`<button>`都可以定义提交按钮，只要将其`type`特性的值设置为"submit"即可，而图像按钮则是通过将`<input>`的`type`特性值设置为"image"来定义的。因此，只要我们单击以下代码生成的按钮，就可以提交表单。

```html
<!-- 通用提交按钮 -->
<input type="submit" value="Submit Form">
<!-- 自定义提交按钮 -->
<button type="submit">Submit Form</button>
<!-- 图像按钮 -->
<input type="image" src="graphic.gif" />
```

只要表单中存在上面列出的任何一种按钮，那么在相应表单控件拥有焦点的情况下，按回车键就可以提交该表单（textarea 是一个例外，在文本区中回车会换行）。如果表单里没有提交按钮，按回车键不会提交表单。

以这种方式提交表单时，浏览器会在将请求发送给服务器之前触发`submit`事件。这样，我们就有机会验证表单数据，并据以决定是够允许表单提交。阻止这个事件的默认行为就可以取消表单提交。

也可以使用编程方式调用`submit()`方法也可以提交表单。不过用该方法提交表单时不会触发`submit`事件，因此在调用该事件之前需要先验证表单数据。

<!-- more -->

#### 重置表单

在用户单击重置按钮时，表单会被重置。使用`type`特性值为"reset"的`<input>`或`<button>`都可以创建重置按钮，如下面的例子所示：

```html
<!-- 通用重置按钮 -->
<input type="reset" value="Reset Form">
<!-- 自定义重置按钮 -->
<button type="reset">Reset Form</button>
```

这两个按钮都可以用来重置表单。在重置表单时，所有表单字段都会恢复到页面刚加载完毕时的初始值。如果某个字段的初始值为空，就会恢复为空；而带有默认值的字段，也会恢复为默认值。

用户单击重置按钮重置表单时，会触发`reset`事件。利用这个机会，我们可以在必要时取消重置操作。

#### 表单字段

可以像访问页面中的其他元素一样，使用原生 DOM 方法访问表单元素。此外，每个表单都有`elements`属性，该属性是表单中所有表单元素（字段）的集合。这个`elements`集合是一个有序列表，其中包含着表单中的所有字段，例如`<input>`、 `<textarea>`、 `<button>`和`<fieldset>`。每个表单字段在`elements`集合中的顺序，与它们出现在标记中的顺序相同，可以按照位置和`name`特性来访问它们。下面来看一个例子。

```javascript
var form = document.getElementById("form1");
//取得表单中的第一个字段
var field1 = form.elements[0];
//取得名为"textbox1"的字段
var field2 = form.elements["textbox1"];
//取得表单中包含的字段的数量
var fieldCount = form.elements.length;
```

**共有的表单字段属性**

- `disabled`：布尔值，表示当前字段是否被禁用。 
- `form`：指向当前字段所属表单的指针，只读。 
- `name`：当前字段的名称。 
- `readOnly`：布尔值，表示当前字段是否只读。 
- `tabIndex`：表示当前字段的切换（tab）序号。 
- `type`：当前字段的类型，如"checkbox"、"radio"等等。 
- `value`：当前字段将被提交给服务器的值。对文件字段来说，这个属性是只读的，包含着文件在计算机中的路径。

**共有的表单字段方法**

- `focus()`：用于将浏览器的焦点设置到表单字段，即激活表单字段，使其可以相应键盘事件。
- `blur()`：从元素中移走焦点。一般用于设置`readyOnly`字段（现在已经不需要了）。

HTML5 中添加了一个`autofocus`属性，在支持这个属性的浏览器中，只要设置了这个属性，不用 JavaScript 就能自动把焦点移动到相应字段。

**共有的表单字段事件**

除了支持鼠标、键盘、更改和 HTML 事件之外，所有表单字段都支持下列 3 个事件。 

- `blur`：当前字段失去焦点时触发。 
- `change`：对于`<input>`和`<textarea>`元素，在它们失去焦点且`value`值改变时触发；对于`<select>`元素，在其选项改变时触发。 
- `focus`：当前字段获得焦点时触发。

通常，可以使用`focus`和`blur`事件来以某种方式改变用户界面，要么是向用户给出视觉提示，要么是向界面中添加额外的功能（例如，为文本框显示一个下拉选项菜单）。而`change`事件则经常用于验证用户在字段中输入的数据。例如，假设有一个文本框，我们只允许用户输入数值。此时，可以利用`focus`事件修改文本框的背景颜色，以便更清楚地表明这个字段获得了焦点。可以利用`blur`事件恢复文本框的背景颜色，利用`change`事件在用户输入了非数值字符时再次修改背景颜色。

```javascript
var textbox = document.forms[0].elements[0];
EventUtil.addHandler(textbox, "focus", function(event) {
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);
    if (target.style.backgroundColor != "red") {
        target.style.backgroundColor = "yellow";
    }
});
EventUtil.addHandler(textbox, "blur", function(event) {
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);
    if (/[^\d]/.test(target.value)) {
        target.style.backgroundColor = "red";
    } else {
        target.style.backgroundColor = "";
    }
});
EventUtil.addHandler(textbox, "change", function(event) {
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);
    if (/[^\d]/.test(target.value)) {
        target.style.backgroundColor = "red";
    } else {
        target.style.backgroundColor = "";
    }
});
```

### 文本框脚本

在 HTML 中，有两种方式来表现文本框：一种是使用`<input>`元素的单行文本框，另一种是使用`<textarea>`的多行文本框。

要表现文本框，必须将`<input>`元素的`type`特性设置为"text"。而通过设置`size`特性，可以指定文本框中能够显示的字符数。通过`value`特性，可以设置文本框的初始值，而`maxlength`特性则用于指定文本框可以接受的最大字符数。

`<textarea>`元素则始终会呈现为一个多行文本框。要指定文本框的大小，可以使用`rows`和`cols`特性。其中， `rows`特性指定的是文本框的字符行数，而`cols`特性指定的是文本框的字符列数（类似于`input`元素的`size`特性）。与`<input>`元素不同， `<textarea>`的初始值必须要放在`<textarea>`和`</textarea>`之间，如下面的例子所示：

```html
<textarea rows="25" cols="5">initial value</textarea>
```

#### 选择文本

上述两种文本框都支持`select()`方法，这个方法用于选择文本框中的所有文本。在调用`select()`方法时，大多数浏览器都会将焦点设置到文本框中。这个方法不接受参数，可以在任何时候被调用。 

``` javascript
var textbox = document.forms[0].elements["textbox1"];
```

在文本框获得焦点时选择其所有文本，这是一种非常常见的做法，特别是在文本框包含默认值的时候。因为这样做可以让用户不必一个一个地删除文本。下面展示了实现这一操作的代码。

```javascript
EventUtil.addHandler(textbox, "focus", function(event){
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);
    target.select();
});
```

**选择（select）事件**

与`select()`方法对应的，是一个`select`事件。在选择了文本框的文本时，就会触发`select`事件。

**取得选择的文本**

HTML5 添加了两个属性：`selectionStart`和`selectionEnd`。这连个属性中保存的是基于 0 的数值，表示所选择文本的范围（即文本选区开头和结尾的偏移量）。

``` javascript
function getSelectedText(textbox){
    return textbox.value.substring(textbox.selectionStart, textbox.selectionEnd);
}
```

IE8 及更早的版本里有一个`document.selection`对象，其中保存着用户在整个文档范围内选择的文本信息。要取得所选文本需要先创建一个范围：`document.selection.createRange()`，然后可取得所选内容。

``` javascript
function getSelectedText(obj) {
    if (document.selection) {
        return document.selection.createRange().text;
    } else {
        return obj.value.substring(obj.selectionStart, obj.selectionEnd);
    }
}
```

**选择部分文本**

Firefox 引入`setSelectionRange()`方法，所有文本框都都有该方法。该方法接受两个参数，即要选择的第一个字符的索引的和要选择的最后一个字符的索引。

除 IE 外的浏览器都支持该方法，IE 使用`createTextRange()`方法创建范围，放在恰当位置上，然后用`moveStart()`和`moveEnd()`这两个范围方法将范围移动到位。还需要在之前调用`collapse()`方法将范围文本折叠到文本框的开始位置。兼容代码：

``` javascript
function selectText(textbox, startIndex, stopIndex) {
    if (textbox.setSelectionRange) {
        textbox.setSelectionRange(startIndex, endIndex)
    } else {
        var range = textbox.createTextRange();
        range.collapse(true);
        range.moveStart("character", startIndex);
        range.moveEnd("character", stopIndex - startIndex);
        range.select();
    }
    textbox.focus();
}
```

#### 过滤输入

我们经常会要求用户在文本框中输入特定的数据，或者输入特定格式的数据。由于文本框在默认情况下没有提供多少验证数据的手段，因此必须使用JavaScript 来完成此类过滤输入的操作。

**屏蔽字符**

如果只想屏蔽特定的字符，则需要检测`keypress`事件对应的字符编码，然后再决定如何响应。例如，下列代码只允许用户输入数值。

```javascript
EventUtil.addHandler(textbox, "keypress", function(event) {
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);
    var charCode = EventUtil.getCharCode(event);
    if (!/\d/.test(String.fromCharCode(charCode)) && charCode > 9 && !event.ctrlKey) {
        EventUtil.preventDefault(event);
    }
});
```

上述代码中，由于 Firefox 和 Safari 会对向上键、向下键、退格键和删除键触发`keypress`事件；Firefox 中所有非字符键触发的`keypress`事件的字符编码是 0，Safari 3 以前版本中，对应的字符编码均为 8。除此之外还有一个问题需要考虑到买就是 Ctrl + C、Ctrl + V 以及其他 Ctrl 组合键。

**操作剪贴板**

IE 是第一个支持与剪贴板相关事件，以及通过 JavaScript 访问剪贴板数据的浏览器。IE 实现成为事实上的标准：

- `beforecopy`：在发生复制操作前触发
- `copy`：在发生复制操作时触发
- `beforecut`：在发生剪切操作前触发
- `cut`：在发生剪切操作时触发
- `beforepaste`：在发生粘贴操作前触发
- `paste`：在发生粘贴操作时触发

要访问剪贴版中的数据，可以使用 clipboardData 对象：IE 中这个对象是`window`对象的属性，其他浏览器中，是相应`event`对象的属性。最好只在发生剪贴板事件期间使用这个对象。

clipboardData 对象有三个方法：

- `getData()`：接受一个参数，要取得的数据的格式。IE 中，有两种数据格式："text"和"URL"。其他浏览器中，参数是一种 MIME 类型，不过，可以用"text"代表"text/plain"。
- `setData()`：第一个参数是数据类型，第二个参数是要放在剪贴板中的文本。
- `clearData()`：清除数据

``` javascript
var EventUtil = {
    // 省略的代码
    getClipboardText: function(event) {
        var clipboardData = event.clipboardData || window.clipboardData;
        return clipboardData.getData("text");
    },
    setClipboardText: function(event) {
        if (event.clipboardData) {
            return event.clipboardData.setData("text/plain", value);
        } else if (window.clipboardData) {
            return window.clipboardData.setData("text", value);
        }
    },
    // 省略的代码
}
```

#### 自动切换焦点

使用 JavaScript 可以从多个方面增强表单字段的易用性。其中，最常见的一种方式就是在用户填写完当前字段时，自动将焦点切换到下一个字段。通常，在自动切换焦点之前，必须知道用户已经输入了既定长度的数据（例如电话号码）。 如下表单：

``` html
<input type="text" name="tel1" id="txtTel1" maxlength="3">
<input type="text" name="tel2" id="txtTel2" maxlength="3">
<input type="text" name="tel3" id="txtTel3" maxlength="4">
```

为增强易用性，同时加快数据输入，可以在前一个文本框中的字符达到最大数量后，自动将焦点切换到下一个文本框。换句话说，用户在第一个文本框中输入了 3 个数字之后，焦点就会切换到第二个文本框，再输入 3 个数字，焦点又会切换到第三个文本框。

```javascript
function tabForward(event) {
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);
    if (target.value.length == target.maxlength) {
        var form = target.form;
        for (var i = 0, len = form.elements.length; i < len; i++) {
            if (form.elements[i] == target) {
                if (form.elements[i + 1]) {
                    form.elements[i + 1].focus();
                }
                return;
            }
        }
    }
};
var textbox1 = document.getElementById("txtTel1");
var textbox2 = document.getElementById("txtTel2");
var textbox3 = document.getElementById("txtTel3");

EventUtil.addHandler(textbox1, "keyup", tabForward);
EventUtil.addHandler(textbox2, "keyup", tabForward);
EventUtil.addHandler(textbox3, "keyup", tabForward);
```

#### HTML5 约束验证 API

**必填字段**

```javascript
<input type="text" name="username" required>
```

任何标注有`required`的字段，在提交表单时都不能空着。JavaScript 中可以通过对应的`required`属性，可以检查某个表单字段是否为必填字段

``` javascript
var isUsernameRequired = doccument.forms[0].elements["username"].required;
```

**其他输入类型**

HTML5 为`<input>`元素添加了几个新的值，比较实用的就是`email`和`url`。

**数值范围**

还定义了另外几个输入元素，这几个元素都要求填写某种基于数字的值："number"、"range"、"datetime"、"datetime-local"、"date"、"month"、"week"和"time"。

对所有这些数值类型的输入元素，可以指定`min`属性，`max`属性和`step`属性：

``` html
<input type="number" min="0" max="100" step="5" name="count">
```

以上属性在 JavaScript 中都能通过对应的元素访问（或修改），还有两个方法：

- `stepUp()`：在当前值基础上加上数值。


- `stepDown()`：在当前值基础上减去数值。

**输入模式**

HTML5 为文本字段新增了`pattern`属性，这个属性是一个正则表达式，用于匹配文本框中的值。

**检测有效性**

使用`checkValidity()`方法可以检测表单的某个字段是否有效，`validaity`属性则会告诉你为什么字段有效或者无效。这个对象包含一系列属性，每个属性会返回一个布尔值。

- `customError`：如果设置了`setCustomValidity()`，则为`true`，否则返回`false`。
- `patternMismatch`：如果值与指定的`pattern`属性不匹配，返回`true`。
- `rangeOverflow`：如果值比`max`大，返回`true`。
- `rangeUnderflow`：如果值比`min`小，返回`true`。
- `stepMisMatch`：如果`min`和`max`之间的步长值不合理，返回`true`。
- `tooLong`：如果值的长度超过了`maxLength`属性指定的长度，返回`true`。
- `typeMismatch`：如果值不是"mail"或"url"要求的格式，返回`true`。
- `valid`：如果这里的其他属性都是`false`，返回`true`。
- `validMissing`：如果标注为`required`的字段中没有值，返回`true`。

**禁用验证**

设置`noValidate`属性，可以告诉表单不进行验证。

### 选择框脚本

选择框是通过`<select>`和`<option>`元素创建的。为了方便与这个控件交互，除了所有表单字段共有的属性和方法外， HTMLSelectElement 类型还提供了下列属性和方法：

- `add(newOption, relOption)`：向控件中插入新`<option>`元素，其位置在相关项（relOption）之前。 
- `multiple`：布尔值，表示是否允许多项选择；等价于 HTML 中的`multiple`特性。 
- `options`：控件中所有`<option>`元素的 HTMLCollection。 
- `remove(index)`：移除给定位置的选项。 
- `selectedIndex`：基于 0 的选中项的索引，如果没有选中项，则值为-1。对于支持多选的控件，只保存选中项中第一项的索引。 
- `size`：选择框中可见的行数；等价于 HTML 中的`size`特性。 

选择框的`type`属性不是"select-one"，就是"select-multiple"，这取决于 HTML 代码中有没有`multiple`特性。选择框的`value`属性由当前选中项决定，相应规则如下：

- 如果没有选中的项，则选择框的`value`属性保存空字符串。 
- 如果有一个选中项，而且该项的`value`特性已经在 HTML 中指定，则选择框的`value`属性等于选中项的`value`特性。即使`value`特性的值是空字符串，也同样遵循此条规则。 
- 如果有一个选中项，但该项的`value`特性在 HTML 中未指定，则选择框的`value`属性等于该项的文本。 
- 如果有多个选中项，则选择框的`value`属性将依据前两条规则取得第一个选中项的值。

在 DOM 中，每个`<option>`元素都有一个 HTMLOptionElement 对象表示。为便于访问数据，HTMLOptionElement 对象添加了下列属性： 

- `index`：当前选项在 options 集合中的索引。 
- `label`：当前选项的标签；等价于 HTML 中的`label`特性。 
- `selected`：布尔值，表示当前选项是否被选中。将这个属性设置为`true`可以选中当前选项。 
- `text`：选项的文本。 
- `value`：选项的值（等价于 HTML 中的`value`特性）

#### 选择选项

``` javascript
var selectedOption = selectbox.options[selectbox.selectedIndex];
```

对于多选项，可以通过`selected`属性获取被选中的项。

``` javascript
function getSelectedOptions(selectbox) {
    var result = new Array();
    var option = null;

    for (var i = 0, len = selectbox.options.length; i < len; i++) {
        option = selectbox.options[i];
        if (option.selected) {
            result.push(option);
        }
    }
    return result;
}
```

#### 添加选项

DOM 方法：

``` javascript
var newOption = document.createElement("option");
newOption.appendChild(docuemnt.createTextNode("Option Text"));
newOption.setAttribute("value", "Option value");
selectbox.appendChild(newOption);
```

用 Option 构造函数创建新选项：

``` javascript
var newOption = new Option("Option Text", "Option value");
selectbox.appendChild(newOption);
```

用`add()`方法：

``` javascript
var newOption = new Option("Option Text", "Option value");
selectbox.add(newOption, undefined);
```

#### 移除选项

可以使用`removeChild()`，也可以使用`remove()`方法，接受一个参数，要删除选项的索引。

#### 移动和重排选项

使用`appendChild()`和`insertBefore()`。

### 表单序列化

由于 Ajax 的出现，表单序列化已经成为常见需求，在序列化表单之前，必须了解在表单提交期间，浏览器是怎样将数据发送给服务器的。

- 对表单字段的名称和值进行 URL 编码，使用和 & 分隔。
- 不发送禁用的表单字段。
- 只发送勾选的复选框和单选框按钮。
- 不发送`type`为"reset"和"button"的按钮。
- 多选选择框中的每个选中的值单独一个条目。
- 在单击提交按钮提交表单的情况下，也会发送提交按钮；否则，不发送提交按钮。也包括 type 为"image"的`<input>`元素。
- `<select>`元素的值，就是选中的`<option>`元素的`value`特性值。如果`<option>`元素没有`value`特性，则是`<option>`元素的文本值。

``` javascript
function serialize(form) {
    var parts = [],
        field = null,
        i,
        len,
        j,
        optLen,
        option,
        optValue;

    for (i = 0, len = form.elements.length; i < len; i++) {
        field = form.elements[i];

        switch (field.type) {
            case "select-one":
            case "select-multiple":
                if (field.name.length) {
                    for (j = 0, optLen = field.options.length; j < optLen; j++) {
                        option = field.options[j];
                        if (option.selected) {
                            optValue = "";
                            if (option.hasAttribute) {
                                optValue = (option.hasAttribute("value") ? option.value : option.text);
                            } else {
                                optValue = (option.attributes["value"].specified ? option.value : option.text);
                            }
                            parts.push(encodeURIComponent(field.name) + "=" + encodeURIComponent(optValue));
                        }
                    }
                }
                break;

            case undefined: //fieldset
            case "file": //file input
            case "submit": //submit button
            case "reset": //reset button
            case "button": //custom button
                break;

            case "radio": //radio button
            case "checkbox": //checkbox
                if (!field.checked) {
                    break;
                }
                /* falls through */

            default:
                //don't include form fields without names
                if (field.name.length) {
                    parts.push(encodeURIComponent(field.name) + "=" + encodeURIComponent(field.value));
                }
        }
    }
    return parts.join("&");
}
```

### 富文本编辑

富文本编辑，又称为 WYSIWYG（What You See Is What You Get，所见即所得）。虽然也没有规范，但在 IE 最早引入的这一功能基础上，已经出现了事实标准。各大浏览器都已经支持这一功能。这一技术的本质，就是在页面中嵌入一个包含空 HTML 页面的`<iframe>`。通过设置`designMode`属性，这个空白的 HTML 页面可以被编辑，而编辑对象则是该页面`body`元素的 HTML 代码。`designMode`属性有两个可能的值：`off`（默认值）和`on`。可以给 iframe 指定一个非常简单的 HTML 页面作为其内容来源。例如：

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Blank Page for Rich Text Editing</title>
    </head>
    <body>
    </body>
</html>
```

这个页面在 iframe 中可以像其他页面一样被加载。要让它可以编辑，必须要将`designMode`设置为`on`，但只有在页面完全加载之后才能设置这个属性。因此，在包含页面中，需要使用 onload 事件处理程序来在恰当的时刻设置`designMode`，如下面的例子所示：

```html
<iframe name="richedit" style="height:100px;width:100px;" src="blank.htm"></iframe>
<script type="text/javascript">
    EventUtil.addHandler(window, "load", function(){
        frames["richedit"].document.designMode = "on";
    });
</script>
```

#### 使用 contenteditable 属性

```  html
<div class="editable contenteditable"></div>
```

``` javascript
var div = document.getElementById("richeidt");
div.contentEditable = "true";  //有三个可能值"true","fasle"和"inherit"
```

#### 操作富文本

使用`document.execCommand()`，该方法可以对文档执行预定义的命令，接受 3 个参数：要执行的命令名称、表示浏览器是否应该为当前命令提供用户界面的一个布尔值（始终为`false`，因为如果设置`true`，Firefox 报错）和执行命令必须的一个值（不需要值，传递`null`）。

| 命令                   | 值（第 3 个参数）         | 说明                                       |
| -------------------- | ------------------ | ---------------------------------------- |
| backcolor            | 颜色字符串              | 设置文档的背景色                                 |
| bold                 | null               | 将选择的文本转换为粗体                              |
| copy                 | null               | 将选择的文本复制到剪贴板                             |
| createlink           | URL 字符串            | 将选择的文本转换为链接，然后指向指定的 URL                  |
| cut                  | null               | 将选择的文本剪贴到剪贴板                             |
| delete               | null               | 删除选择的文本                                  |
| fontname             | 字体名称               | 将选择的文本改为指定的字体                            |
| fontsize             | 1 ~ 7              | 将选择的文本改为指定的字体大小                          |
| forecolor            | 颜色字符串              | 将选择的文本改为指定的颜色                            |
| formatblock          | 需要包围当前文本块的 HTML 标签 | 使用指定的 HTML 标签来包裹指定的文本块                   |
| indent               | null               | 缩进文本                                     |
| inserthorizontalrule | null               | 在光标处插入一个 `<hr>` 元素                       |
| insertimage          | 图像的 URL            | 在光标处插入一个图像                               |
| insertorderedlist    | null               | 在光标处插入一个`<ol>` 元素                        |
| insertunorderedlist  | null               | 在光标处插入一个`<ul>` 元素                        |
| insertparagraph      | null               | 在光标处插入一个`<p>` 元素                         |
| italic               | null               | 将选择的文本改为斜体                               |
| justifycenter        | null               | 在光标处的文本块居中对齐                             |
| justifyleft          | null               | 在光标处的文本块左对齐                              |
| outdent              | null               | 减少缩进                                     |
| paste                | null               | 将剪贴板中的内容粘贴到选择的文本                         |
| removeformat         | null               | 移除包围当前文本块的 HTML 标签，这是撤销 formatblock 命令的操作 |
| selectall            | null               | 选中所有文本                                   |
| underline            | null               | 为选中的文本添加下划线                              |
| unlink               | null               | 移除文本链接，这是撤销 createlink 命令的操作             |

虽然所有的浏览器都支持这些命令，但它们生成的 HTML 文本并不相同。比如执行 bold 命令后，IE 和 Opera 会使用 `<strong>` 标签包围文本，而 Safari 和 Chrome 使用的是 `<b>` 标签，在 Firefox 中使用的是 `<span>` 标签。

- `queryCommandEnabled()`方法是用来检测是否可以针对当前选择的文本，或者当前插入字符所在位置执行某个命令。它接受一个参数，即要检测的命令。

``` javascript
var result = frames["richedit"].document.queryCommandEnabled("bold");
```

这个命令其实并不可靠，比如在 Firefox 中默认禁用剪切操作，这个命令可能会返回`true`。

- `queryCommandState()` 方法用于确定某个命令是否已经应用到所选中的文本：

``` javascript
var result = frames["richedit"].document.queryCommandState("bold"); 
```

富文本编辑器就是根据这个方法返回的值来更新粗体、斜体的按钮状态的。

- `queryCommandValue()`可以取得执行命令时传入的值（就是`document.execCommand()`方法的第 3 个值）：

``` javascript
var fontSize = frames["richedit"].document.queryCommandValue("fontsize"); //如果之前命名传入的是7，就会返回7
```

通过这个方法可以确定某个命令是怎么样应用到文本的，可以据以确定再对其应用后续命令是否合适。

#### 富文本选区

使用 iframe 的`getSelection()`方法，可以确定实际选择的文本。，该方法是`window`对象和`document`对象的属性，调用它会返回一个表示当前选择文本的 Selection 对象。每个 Selection 对象都有下列属性。 

- `anchorNode`：选区起点所在的节点。 
- `anchorOffset`：在到达选区起点位置之前跳过的`anchorNode`中的字符数量。 
- `focusNode`：选区终点所在的节点。 
- `focusOffset`： `focusNode`中包含在选区之内的字符数量。 
- `isCollapsed`：布尔值，表示选区的起点和终点是否重合。 
- `rangeCount`：选区中包含的 DOM 范围的数量。 

Selection 对象的这些属性并没有包含多少有用的信息。好在，该对象的下列方法提供了更多信息：

- `addRange(range)`：将指定的 DOM 范围添加到选区中。 
- `collapse(node, offset)`：将选区折叠到指定节点中的相应的文本偏移位置。 
- `collapseToEnd()`：将选区折叠到终点位置。 
- `collapseToStart()`：将选区折叠到起点位置。 
- `containsNode(node)`：确定指定的节点是否包含在选区中。 
- `deleteFromDocument()`：从文档中删除选区中的文本，与`document.execCommand("delete",false, null)`命令的结果相同。 
- `extend(node, offset)`：通过将`focusNode`和`focusOffset`移动到指定的值来扩展选区。 
- `getRangeAt(index)`：返回索引对应的选区中的 DOM 范围。 
- `removeAllRanges()`：从选区中移除所有 DOM 范围。实际上，这样会移除选区，因为选区中至少要有一个范围。 
- `reomveRange(range)`：从选区中移除指定的 DOM 范围。 
- `selectAllChildren(node)`：清除选区并选择指定节点的所有子节点。
- `toString()`：返回选区所包含的文本内容。


IE8- 中可以使用`selection`支持选区，还可以用`htmlText`和`pasteHTML()`方法实现文本高亮效果：

``` javascript
var range = frames["richedit"].document.selection.createRange();
range.pasteHTML("<span style='background-color:yellow'" + range.htmlText + “</span>”);
```

#### 表单与富文本

通过`innerHTML`取得富文本编辑器中的值，然后放入一个隐藏的`<input>`中。