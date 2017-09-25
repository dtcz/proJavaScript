---
title: 重读 JS 高程（Chapter13）
tags: javascript
category: javascript
date: 2017-09-13 10:01:05
---


## Chapter13. 事件

> 事件，就是文档或浏览器窗口中发生的一些特定的交互瞬间。

JavaScript 与 HTML 之间的交互是通过**事件**实现的。可以使用事件处理程序来预定事件，以便事件发生时执行相应的代码。

### 事件流

> **事件流**描述的是从页面中接收事件的顺序。

#### 事件冒泡

IE 的事件流叫做**事件冒泡**（event bubbling），即事件开始时由最具体的元素（文档中嵌套层次最深的那个节点）接收，然后逐级向上传播到较为不具体的节点（文档）。

所有现代浏览器支持事件冒泡，但具体实现上还是有一些区别。IE5.5 事件冒泡会跳过`<html>`元素。IE9+ 则将事件一直冒泡到`window`对象。

#### 事件捕获

Netscape 团队提出的另一种事件流叫做**事件捕获**（event capturing）。事件捕获与冒泡相反，首先由不具体的节点接收，最后是最具体的节点接收事件。事件捕获的用意是在事件到达预定目标之前捕获它。

由于老版本的浏览器不支持事件捕获，因此建议使用事件冒泡，有特殊需要的时候在使用事件捕获。

#### DOM 事件流

“DOM2 级事件”规定的事件流包括三个阶段：事件捕获阶段、处于目标阶段和事件冒泡阶段。首先发生事件捕获，然后是实际目标接收到事件，最后一个阶段是事件冒泡阶段，可以在这个阶段对事件做出响应。

IE9+ 支持 DOM 事件流，而且在捕获阶段也会出发事件对象上的事件，就是有两个机会在目标对象上操作事件，IE8 及更早版本不支持。

<!--more-->

### 事件处理程序

#### HTML 事件处理程序

通过 HTML 指定事件处理程序。

```html
<input type="button" value="click me" onclick="showMessage()" />
```

这样做有几个缺点：

1. 有时差，用户可能在 HTML 元素一出现在页面上就触发相应事件，但可能事件处理程序还不能执行。
2. 这种事件处理程序的作用域链在不同浏览器中会导致不同结果，不同 JavaScript引擎遵循的标识符解析规则略有差异，很可能会在访问非限定对象成员时出错。
3. HTML 与 JavaScript 代码紧密耦合，一旦改动，起码改动连个地方。不能满足结构、样式、行为分离。

#### DOM0 级事件处理程序

使用 JavaScript 指定事件处理程序的传统方式，就是将一个函数复制给一个事件处理程序属性。该方法简单，而且可以跨浏览器使用，知道现在依然应用广泛。

```javascript
var btn = document.getElementById("myBtn");
btn.onclick = function() {
    alert(this.id);
}

// 删除事件处理程序
btn.onclick = null;
```

#### DOM2 级事件处理程序

“DOM2 级时间”定义了两个方法来添加和删除事件处理程序，IE9+ 支持。

- `addEventListener() `
- `removeEventListener()`

它们都接受 3 个参数，要处理的事件类型，作为事件处理程序的函数和一个布尔值，`true`表示在捕获阶段调用事件处理函数，`false`表示在冒泡阶段调用。

使用 DOM2 级方法可以添加多个事件处理程序。通过`addEventListener()`添加的事件监听必须使用`removeEventListener()`来移除。

#### IE 事件处理程序

IE 实现了与 DOM 中类似两个方法：

- `attachEvent`
- `detachEvent` 

它们都接受两个参数：事件处理程序名称与事件处理程序函数。注意`attachEvent()`的第一个参数是"onclick"，而非 DOM 的`addEventListener()`的"click"。

通过`attachEvent()`添加的事件处理程序是在全局作用域下运行。所以`this == window`。此外，`attachEvent`添加的多个事件处理程序的触发顺序是相反的，即后添加的先触发。

#### 跨浏览器的事件处理程序

```javascript
var EventUtil = {
    addHandler: function(elem, type, handler) {
        if(elem.addEventListener) {
            elem.addEventListener(type, handler, false);
        } else if(elem.attachEvent) {
            elem.attachEvent('on' + type, handler);
        } else {
            elem['on' + type] = handler;
        }
    },
    removeHandler: function(elem, type, handler) {
        if(elem.removeEventListener) {
            elem.removeEventListener(type, handler, false);
        } else if(elem.detachEvent) {
            elem.detachEvent('on' + type, handler);
        } else {
            elem['on' + type] = null;
        }
    }
}
```

###  事件对象

在触发 DOM 上的某个时间时，会产生一个**事件对象**`event`，这个对象中包含着所有与事件有关的信息。

#### DOM 中的事件对象

无论用什么方法添加事件处理程序，都会传入`event`对象。`event`对象包含与创建它的特定事件有关的属性和方法。

| 属性/方法                      |      类型      | 读/写  | 说明                                       |
| -------------------------- | :----------: | :--: | ---------------------------------------- |
| bubbles                    |   Boolean    |  只读  | 表明事件是否冒泡                                 |
| cancelable                 |   Boolean    |  只读  | 表明是否可以取消事件的默认行为                          |
| currentTarget              |   Element    |  只读  | 表明事件处理程序当前正在处理事件的元素                      |
| defaultPrevented           |   Boolean    |  只读  | 为`true`表示已经调用了`preventDefault()`方法       |
| detail                     |   Integer    |  只读  | 与事件相关的细节信息                               |
| eventPhase                 |   Integer    |  只读  | 调用事件处理程序的阶段：1 表示捕获阶段，2 表示“处于目标”阶段，3 表示冒泡阶段 |
| preventDefault()           |   Function   |  只读  | 取消事件的默认行为                                |
| stopImmediatePropagation() |   Function   |  只读  | 取消事件的进一步捕获或冒泡，同时阻止任何事件处理程序被调用            |
| stopPropagation()          |   Function   |  只读  | 取消事件的进一步捕获或冒泡                            |
| target                     |   Element    |  只读  | 目标元素                                     |
| type                       |    String    |  只读  | 被触发的事件类型                                 |
| trusted                    |   Boolean    |  只读  | 为`true`表示事件是浏览器生成的，为`false`表示事件是由开发人员通过 JavaScript 创建的 |
| view                       | AbstractView |  只读  | 与事件相关联的抽象试图，等同于发生事件的`window`对象           |

在事件处理程序中，对象`this`始终等于`currentTarget`，而`target`只包含事件的实际目标。

只有在事件处理程序执行期间，`event`对象才存在。一旦时间处理程序执行完毕，`event`对象将会被立即销毁。

#### IE 中的事件对象

| 属性/方法        | 类型      | 读/写  | 说明                                       |
| ------------ | ------- | ---- | ---------------------------------------- |
| cancelBubble | Boolean | 读写   | 默认为`false`。用于取消事件冒泡，与 DOM 中的`stopPropagation()`相同 |
| returnValue  | Boolean | 读写   | 默认为`true`，用于取消默认行为，与 DOM 中的`preventDeafault()`相同 |
| srcElement   | Element | 只读   | 事件目标元素，与 DOM 中的`target`相同                |
| type         | String  | 只读   | 被触发的事件类型                                 |

#### 跨浏览器的事件对象

```javascript
var eventUtil = {
    addHandler: function(elem, type, handler) {
        // 省略代码
    },
    getEvent: function(event) {
        return event || window.event;
    },
    getTarget: function(event) {
        return event.target || event.srcElement;
    },
    preventDefault: function(event) {
        if(event.preventDefault) {
            event.preventDefault();
        } else { 
            event.returnValue = false;
        }
    },
    removeHandler: function(elem, type, handler) {
        // 省略代码
    },
    stopPropagation: function(event) {
        if(event.stopPropagation) {
            event.stopPropagation();
        } else {
            event.cancelBubble = true;
        }
    }
};
```

### 事件类型

#### UI事件

| 事件     | 说明                                       |
| ------ | ---------------------------------------- |
| load   | 页面完全加载后触发                                |
| unload | 页面完全卸载或触发                                |
| abort  | 当用户停止下载过程时，如果嵌入的内容没有加载完，则在`<object>`元素上面触发 |
| error  | 当发生 JavaScript 错误时在`window`上触发，当无法加载图像时在`<img>`元素上面触发 |
| select | 用户选中文本框中的文本的时候触发                         |
| resize | 窗口或框架大小变化的时候触发                           |
| scroll | 滚动带滚动条的内容时触发                             |

#### 焦点事件

焦点事件会在页面获得或者失去焦点的时候触发。利用这些事件和`document.hasFocus()`方法及`document.activeElement`属性配合，可以知晓用户在页面上的行踪。

| 方法/属性       | 说明                                       |
| ----------- | ---------------------------------------- |
| blur        | 在元素失去焦点时触发，不会冒泡                          |
| DOMFocusIn  | 在元素获得焦点时触发，与`focus`等价，但是会冒泡，只有 Opera 支持，在 DOM3 中被废弃 |
| DOMFocusOut | 在元素失去焦点时触发，是`blur`的通用版本，只有 Opera 支持，在 DOM3 中被废弃 |
| focus       | 在元素获得焦点时触发，不会冒泡                          |
| focusin     | 在元素获得焦点时触发                               |
| focusout    | 在元素失去焦点时触发，会冒泡                           |

当焦点从页面的一个元素移动到另一个元素，会依次触发下列事件：

1. `focusout`在失去焦点的元素上触发；
2. `focusin`在获得焦点的元素上触发；
3. `blur`在失去焦点的元素上触发；
4. `DOMFocusOut`在失去焦点的元素上触发；
5. `focus`在获得焦点的元素上触发；
6. `DOMFocusIn`在获得焦点的元素上触发。

判断浏览器是否支持某些事件：

```javascript
var isSupported = document.implementation.hasFeature("FoucusEvent", "3.0");
```

#### 鼠标和滚轮事件

DOM3 中定义了以下 9 个鼠标事件，如下表：

| 事件             | 说明                                |
| -------------- | --------------------------------- |
| click          | 单击或回车时触发                          |
| dbclick        | 双击鼠标触发                            |
| mousedown      | 鼠标按下时触发                           |
| mouseenter     | 鼠标首次进入元素范围触发，这事件不冒泡，且在其后代元素上不会触发  |
| mouseleave     | 鼠标离开元素范围触发，这事件不冒泡，且在其后代元素上不会触发    |
| mousemove      | 鼠标在元素内部移动时，就会不断触发                 |
| mouseover      | 鼠标位于一个元素外部，然后用户将其首次移入另一个元素边界之内时触发 |
| mouseout       | 鼠标离开元素范围进入另一个元素范围时触发              |
| mouseup        | 在用户释放鼠标按钮时触发                      |
| mousewheel     | 鼠标滚轮滚动时触发，Firefox 不支持             |
| DOMMouseScroll | 鼠标滚轮滚动时触发，Firefox 支持              |

**坐标位置**


| 属性      | 说明                                       |
| ------- | ---------------------------------------- |
| clientX | 相对于浏览器视口中的水平坐标                           |
| clientY | 相对于浏览器视口中的垂直坐标                           |
| pageX   | 相对于页面的水平坐标（页面没有滚动时，等于 clientX），IE 下有 x 属性 |
| pageY   | 相对于页面的垂直坐标（页面没有滚动时，等于 clientY），IE 下有 y 属性 |
| screenX | 相对于屏幕坐标的水平坐标                             |
| screenY | 相对于屏幕坐标的垂直坐标                             |

**修改键**

虽然鼠标事件主要是在鼠标上触发的，但在按下鼠标时键盘上的某些键状态也可以影响到所要采取的操作，修改键如下：

| 属性       | 说明              |
| -------- | --------------- |
| shiftkey | 检测是否按下了 shift 键 |
| ctrlkey  | 检测是否按下了 ctrl 键  |
| metakey  | 检测是否按下了 meta 键  |
| altkey   | 检测是否按下了 alt 键   |

当鼠标事件发生时，通过检测这几个属性就可以确定用户是否同时按下了其中的键。

**相关元素** 

对`mouseover`事件而言，事件的主目标是获得光标的元素，而相关元素就是那个失去光标的元素，`mouseout`则正好相反。

```javascript
var eventUtil = {
    // 省略其他代码
    getRelatedTarget: function (event) {
        if(event.relatedTarget) {
            return event.relatedTarget; 
        }　else if(event.toElement) {
            // for ie mouseout event
            return event.toElement;
        } else if(event.fromElement) {
            // for ie mouseover event
            return event.fromElement;
        } else {
            return null;
        }
    }
    // 省略其他代码
}
```

**鼠标按钮** 

DOM 下`button`属性有三个值：0 表示主鼠标按钮，1  表示滚轮按钮，2 表示次鼠标按钮。 IE8 以下，`button`属性的值有 8 个：

- 0：没有按下按钮
- 1：按下主鼠标
- 2：按下次鼠标
- 3：表示同时按下了主、次鼠标按钮
- 4：表示按下了中间的鼠标按钮
- 5：表示同时按下了主鼠标按钮和中间鼠标按钮
- 6：表示同时按下了次鼠标按钮和中间鼠标按钮
- 7：表示同时按下了是哪个鼠标按钮

```javascript
var eventUtil = {
    // 省略已有代码
    getButton: function(event) {
        if(document.implementation.hasFeature("MouseEvents", "2.0")) {
            return event.button;
        } else {
            // ie
            switch (event.button) {
                case 0:
                case 1:
                case 3:
                case 5:
                case 7:
                    return 0;
                case 2:
                case 6:
                    return 2;
                case 4:
                    return 1;

            }
        }
    }
}
```

**更多事件信息**

“DOM2 级事件”规范在`event`对象中提供了`detail` 属性，表示在给定位置上发生了多少次单击。

IE 也通过下列属性为鼠标事件提供了更多信息：

- `altLeft`：表示是否按下了 Alt 键
- `ctrlLeft`：表示是否按下了 Ctrl 键
- `offsetX`：光标相对于目标元素边界的 x 坐标
- `offsetY`：光标相对于目标元素边界的 y 坐标
- `shiftLeft`：表示是否按下了 Shift 键。如果 shiftleft 的值为`true`，则 shiftKey 的值也为`true`

**鼠标滚轮事件**

IE下：`mousewheel`事件，`wheelDelta`属性表示鼠标滚动方向，是 120 的倍数。向上表示 +120，向下表示 -120。 

FF下：`DOMMouseScroll`事件，`detail`属性表示鼠标滚动方向，是 3 的倍数。向上表示 -3，向下表示 +3。

```javascript
var eventUtil = {
    getWheelDelta: function(event) {
        if(event.mousewheel){
            return (client.engine.opera && client.engine.opera < 9.5) ? -wheelDelta : wheelDelta; 
            //在opera9.5之前，wheelDelta方向与现在相反，需要做一些处理
        } else {
            // for ff
            return -event.detail*40;
        }
    }
}
```

#### 键盘与文本事件

有三个键盘事件：

- `keydown`：当用户按下键盘上的任意键时触发，如果按住不放，会重复触发事件。
- `keypress`：当用户按下键盘上的字符键时触发，如果按住不放，会重复触发事件。
- `keyup`：当用户释放键盘上的键触发。

**键码和字符编码**

在发生`keydown`和`keyup`事件时，`event`对象的`keyCode`属性会包含一个代码，与键盘上一个特定的键对应。发生`keypress`事件时，`event`对象会包含一个`charCode`属性，会包含按下键所代表字符的编码。IE8 以下不支持`charCode`属性，但是会在`keyCode`中保存字符的 ASCII 编码。

```javascript
var eventUtil = {
    getCharCode: function(event) {
        if(typeof event.charCode == 'number') {
            return event.charCode;
        } else {
            // IE8及以前，早期opera在keyCode中保存键码
            return event.keyCode;
        }
    }
}
```

**DOM3 级变化**

DOM3 中不再包含`charCode`属性，而是包含两个新属性：`key`和`char`。`key`的值是一个字符串，按下某个字符键时，`key`的值就是相应的文本字符；按下非字符键，`key`的值是相应的键名（如“Shift”）。而`char`属性在按下字符键的行为与`key`相同，在按下非字符键时值为`null`。

**textInput 事件**

在 DOM3 级事件中引入，与`keypress`不同，只有在可编辑区域内输入字符的时候触发。出发该事件时，它的`event`对象中还包含一个`data`属性，这是属性就是用户输入的字符。`event`对象上还有一个属性，叫`inputMethod`，表示把文本输入到文本框中的方式。

- 0，表示浏览器不确定是怎么输入的
- 1，键盘输入
- 2，粘贴
- 3，拖放
- 4，IME 输入
- 5，在表单中选择某一项输入
- 6，手写输入
- 7，语音输入
- 8，几种方法组合输入
- 9，通过脚本输入

#### 复合事件

用于处理 IME（Input Method Editor，输入法编辑器）的输入序列，可以让用户输入在物理键盘上找不到的字符。

- `compositionstart`：在 IME 的文本复合系统打开时触发，表示要开始输入。
- `compositionupdate`：在向输入字段中插入新字符时触发。
- `compositionend`：在 IME 的文本复合系统关闭时触发，表示返回正常键盘输入状态。

触发该事件时，目标是接收文本的输入字段，比文本事件的时间对象多一个属性`data`，其中包含以下几个值中的一个：

- 如果`compositionstart`事件发生时访问，包含正在编辑的文本；
- 如果`compositionupdate`事件发生时访问，包含正插入的新字符；
- 如果`compositionend`事件发生时访问，包含此次输入会话中插入的所有字符。



#### 变动事件

DOM2 级的变动事件能在 DOM 中某一部分发生改变时给出提示。

| 事件                          | 说明                                       |
| --------------------------- | ---------------------------------------- |
| DOMSubtreeModified          | 在 DOM 结构中发生任何变化时触发                       |
| DOMNodeInserted             | 一个节点作为子节点被插入到另一个节点中时触发                   |
| DOMNodeRemoved              | 节点被移除的时候触发                               |
| DOMNodeInsertedIntoDocument | 节点被直接插入文档，或通过子树间接插入文档的时候触发。在DOMNodeInserted之后触发 |
| DOMNodeRemovedFromDocument  | 节点被直接从文档中移除，或通过子树被移除的时候触发，在DOMNodeRemoved之后被触发 |
| DOMAttrModified             | 在特性被修改之后触发                               |
| DOMCharacterDataModified    | 在文本节点的值发生变化的时候触发                         |

`event.relatedNode`保存着相关节点。可以类比`relatedTarget`。

#### HTML5事件

**contextmenu 事件**

用以表示何时应该显示上下文菜单，以便开发人员取消默认的上下文菜单而提供自定义菜单。

**beforeunload 事件**

为了让开发人员有可能在页面卸载前阻止这一操作，这个事件会在浏览器卸载页面之前触发。但是不能彻底取消这个事件，因为那就相当于让用户无法离开当前页面。这个事件的意图是将控制权交给用户。为了显示这个弹出对话框，必须将`event.returnValue`的值设置为要显示给用户的字符串，同时作为函数的值返回。

**DOMContentLoaded 事件**

`window`的`load`事件会在页面中的一切都加载完毕时触发，但这个过程可能会因为要加载的外部资源过多而颇费周折。而`DOMContentLoaded`事件则在形成完整的 DOM 树之后就会触发，不理会图像、JavaScript 文件、CSS 文件或其他资源是否已经下载完毕。

**readystatechange 事件**

IE 为 DOM 文档中的某些部分提供了`readystatechange`事件。这个事件的目的是提供与文档或元素的加载状态有关的信息，支持该时间的每个对象都有一个`readyState`属性，可能包含下列 5 个值中的一个。

- uninitialized（未初始化）：对象存在但尚未初始化。
- loading（正在加载）：对象正在加载数据。
- loaded（加载完毕）：对象加载数据完成。
- interactive（交互）：可以操作对象了，但还没有完全加载。
- complete（完成）：对象已经加载完毕。

**pageshow 和 pagehide 事件**

Firefox 和 Opera 有一个特性，名叫“往返缓存（back--forward cache，或 bfcache），可以在用户使用浏览器的“后退”和“前进”按钮时加快页面的转换速度。

`pageshow`在页面显示时触发，无论是否来自 bfcache。在重新加载的页面中，`pageshow`会在`load`事件触发后触发；而对于 bfcache 中的页面，`pageshow`会在页面状态完全恢复的那一刻触发。`pageshow`事件的`event`对象还包含一个名为`persisted`的布尔值属性。如果页面被保存在 bfcache 中，则为`true`；否则，为`false`。`pagehide`事件触发时根据`persisted`的值采取不同的操作。

**hashchange 事件**

便于在 URL 的参数列表发生变化时通知开发人员。之所以新增这个事件，是因为在 Ajax 应用中，开发人员经常要利用 URL 参数列表来保存状态或导航信息。

必须要把该事件处理程序添加给`window`对象，然后 URL 参数列表只要变化就会调用它。此时的`event`对象应该额外包含两个属性：`oldURL`和`newURL`。这两个属性分别保存着参数列表前后的完整 URL。

#### 设备事件

**orientationchange 事件**

移动 Safari 中的事件，`window.orientation`属性可能包含 3 个值：0 表示肖像模式，90 表示向左旋转的横向模式，-90 表示向右旋转的横向模式。

**MozOrientation 事件**

与 iOS 中的事件不同，该事件只能提供一个平面的方向变化。此时`event`对象包含三个属性：x，y 和 z。这几个属性都介于 1 到 -1 之间，表示不同坐标轴上的方向。在静止状态下，x 值为 0，y 值为 0，z 值为 1。如果设备右倾，x 值会减小，向左倾，x 值会增大。如果设备向远离用户的方向倾斜，y 值会减小，向接近用户的方向倾斜 y 值会增大。z 轴检测垂直加速度，1 表示静止不动，在设备移动时值会减小。失去重量状态下值为 0。

**deviceorientation 事件**

触发该事件时，事件对象包含着每个轴相对于设备静止状态下发生变化的信息。事件对象包含以下 5 个属性：

- `alpha`：在围绕 z 轴旋转时（左右旋转），y 轴的度数差；是一个介于 0 到 360 之间的浮点数。
- `beta`：在围绕 x 轴旋转时（前后旋转），z 轴的度数差；是一个介于 -180 到 180 之间的浮点数。
- `gamma`：在围绕 y 轴旋转时（扭转设备），z 轴的度数差；是一个介于 -90 到 90 之间的浮点数。
- `absolute`：布尔值，表示设备是否返回一个绝对值。
- `compassCalibrated`：布尔值，表示设备的指南针是否校准过。

**devicemotion 事件**

触发 devicemotion 事件时，事件对象包含以下属性：

- `acceleration`：一个包含 x、y 和 z 属性的对象，在不考虑重力的情况下，告诉你在每个方向上的加速度。
- `accelerationIncludingGravity`：一个包含 x、y 和 z 属性对象，在考虑 z 轴自然重力加速度的情况下，考诉你在每个方向上的加速度。
- `interval`：以毫秒表示的时间值，必须在另一个 devicemotion 事件触发前传入。这个值在每个时间中应该是一个常量。
- `rotationRate`：一个包含表示方向的`alpha`、`beta`和`gamma`属性的对象。

#### 触摸与手势事件

**触摸事件**

触摸事件会在用户手指放在屏幕上面时、在屏幕上滑动时或从屏幕上移开时触发。具体来说，有以下几个触摸事件：

- `touchstart`：当手指触摸屏幕时触发，即使有一个手指放在屏幕上也会触发。
- `toucehmove`：当手指在屏幕上滑动时连续地触发。
- `touchend`：当手指从屏幕移开时触发。
- `touchcancel`：当系统停止跟踪触摸时触发。

除了常见的 DOM 属性外，触摸事件还包含下列三个用于跟踪触摸的属性：

- `touches`：表示当前跟踪的触摸操作的 Touch 对象的数组。
- `targetTouches`：特定于事件目标的 Touch 对象的数组。
- `changeTouches`：表示自上次触摸依赖发生了什么改变的 Touch 对象的数组。

每个 Touch 对象包含下列属性：

- `clientX`：触摸目标在视口中的 x 坐标。
- `clientY`：触摸目标在视口中的 y 坐标。
- `pageX`：触摸目标在页面中的 x 坐标。
- `pageY`：触摸目标在页面中的 y 坐标。
- `screenX`：触摸目标在屏幕中的 x 坐标。
- `screenY`：触摸目标在屏幕中的 y 坐标。
- `identifier`：标识触摸的唯一 ID。
- `target`：触摸 DOM 节点目标。

这些事件会在文档的所有元素上面触发，因而可以分别操作页面的不同部分。在触摸屏幕上的元素时，这些事件发生的顺序如下：

1. `touchstart`
2. `mouseover`
3. `mousemove`
4. `mousedown`
5. `mouseup`
6. `click`
7. `touchend`

**手势事件**

两个手指触摸屏幕时就会产生手势，手势通常会改变显示项的大小，或者旋转显示项。有三个手势事件：

- `gesturestart`：当一个手指已经按在屏幕上而另一手指又触摸屏幕时触发。
- `gesturechange`：当触摸屏幕的任何一个手指的位置发生变化时触发。
- `gestureend`：当任何一个手指从屏幕上移开时触发。

该事件包含两个额外属性

- `rotation`：手指变化引起的旋转角度，负值表示逆时针旋转，正值表示顺时针旋转。
- `scale`：表示两个手指间距离的变化情况；这个值从 1 开始，并随距离拉大而增长，随距离缩短而减小。

### 内存和性能

#### 事件委托

只需在 DOM 树中尽量最高的层次上添加一个事件处理程序：

``` html
<ul id="myLinks">
    <li id="goSomewhere">Go somewhere</li>
    <li id="doSomething">Do something</li>
    <li id="sayHi">Say hi</li>
</ul>
```
``` javascript
var list = document.getElementById('myLinks');
EventUtil.addHandler(list, 'click', function(event) {
    var target = EventUtil.getTarget(event);
    target.style.backgroundColor = 'red';
});
```

#### 移除事件处理程序 

在 Web 应用程序中，过时不用的事件处理程序是影响性能的一个重要原因。

当直接移除带有事件处理函数的元素的时候，造成事件处理函数保持引用，无法被回收。 在进行元素移除的时候，首先对事件处理程序进行手工移除。 

当进行页面卸载的时候，如果在事件处理程序没有被清理干净之前卸载完成，事件处理程序就会滞留在内存中，每次加载完在卸载页面的时候，滞留的对象就会增加。 应该在卸载前通过 onunload 事件移除事件处理程序。

### 模拟事件

#### DOM中的事件模拟

 通过`document.createEvent()`创建`event`对象。在DOM2级中，所有事件使用英文复数，DOM3级中，使用单数。

- UIEvents：一般化的 UI 事件。鼠标事件和键盘事件都继承自 UI 事件。DOM3 级中是 UIEvent。
- MouseEvents：一般化的鼠标事件。DOM3 级中是 MouseEvent。
- MutationEvents：一般化的 DOM 变动事件。DOM3 级中是 MutationEvent。
- HTMLEvents：一般化的 HTML 事件。没有对应的 DOM3 级事件。


模拟事件的最后一步就是触发事件，这一步需要使用`dispatchEvent()`，调用该方法要传入一个参数，即表示要触发的事件的`event`对象。

**模拟鼠标事件**

模拟鼠标事件的做法是向`createEvent()`传入字符串"MouseEvent"。返回的对象有一个名为`initMouseEvent()`方法，用于指定与该鼠标事件有关的信息。该方法接受 15 个参数：type、bubbles、cancelable、view（基本设置为`document.defaultView`）、detail、screenX、screenY、clientX、clientY、ctrlKey、altKey、shiftKey、metaKey、button、relatedTarget。

```javascript
var btn = document.getElementById('myBtn');
var event = document.createEvent('MouseEvents');

event.initMouseEvent('click', true, true, document.defaultView, 0, 0, 0, 0, 0, false, false, false, false, 0, null);

btn.dispatchEvent(event);
```

**模拟键盘事件**

与鼠标事件类似，传入"KeyboardEvent"就可以创建一个键盘事件，返回对象包含`initKeyEvent()`方法。该方法接受如下参数：type、bubbles、cancelable、view、key、location、modifiers、repeat。Firefox 中传入"KeyEvents"可以创建一个键盘事件，参数如下：type、bubbles、cancelable、view、ctrlKey、altKey、shiftKey、metaKey、keyCode、charCode。

```javascript
var textbox = document.getElementById('myTextbox');
if(document.implementation.hasFeature('KeyboardEvents', '3.0')) {
    var event = document.createEvent('KeyboardEvents');
    event.initKeyboardEvent('keydown', true, true, document.defaultView, 'a', 0, 'Shift', 0);
    textbox.dispatchEvent(event);
}
// 模拟按下A同时按下shift键。
```

以上代码虽然能模拟键盘事件的触发，却无法向文本域输入任何字符，这是由于无法精确模拟键盘事件造成的。

**模拟其他事件**

除了鼠标和键盘事件，有时同样需要模拟变动事件和 HTML 事件。要模拟变动事件，可以使用`createEvent('MutationEvents')`，该方法接受参数包括：type、bubbles、cancelable、relatedNode、preValue、newValue、attrName、attrChange。

``` javascript
var event = document.createEvent('MutationEvents');
event.initMutaionEvent('DOMNodeInserted', true, false, someNode, '', '', '', 0);
target.dispatchEvent(event);
```

模拟 HTML 事件也一样，通过`createEvent('HTMLEvents')`，然后再使用这个对象的`initEvent()`方法来初始化它。

``` javascript
var event = document.createEvent('HTMLEvents');
event.initMutaionEvent('focus', true, false);
target.dispatchEvent(event);
```

**自定义 DOM 事件**

DOM3 级定义了“自定义事件”。自定义事件不是由原生 DOM 触发的，它的目的是让开发人员创建自己的事件。同样，要调用`createEvent('CustomEvent')`，返回的对象有一个名为`initCustomEvent()`的方法，接受如下 4 个参数：type、bubbles，cancelable，detail。

#### IE 中的事件模拟 

思路类似，不过创建`event`对象使用的是`document.createEventObject()`，与 DOM 方式不同，这方法不接受参数，结果会返回一个通用的`event`对象。然后，必须手工添加必要信息。最后一步就是在目标上调用`fireEvent()`方法。

```javascript
var event = document.createEventObject(); // 不接收参数

// 初始化事件对象，必须手工添加属性
event.altKey = false;
event.ctrlKey = false;
event.keyCode = 65;
// 对添加的属性没有任何限制，即使添加的属性不支持也不所谓。

// 触发事件
textbox.fireEvent('onkeypress', event)
```