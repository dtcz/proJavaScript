---
title: 重读 JS 高程（Chapter24）
tags: javascript
category: javascript
date: 2017-09-24 23:15:30
---


## Chapter24. 最佳实践

### 可维护性

#### 什么是可维护的代码

- 可理解性
- 直观性
- 可适应性
- 可扩展性
- 可调试性

#### 代码约定

一种让代码变得可维护的简单途径是形成一套 JavaScript 代码的书写约定。

**可读性**

- 缩进，常见为 4 个空格。
- 注释，需要进行注释的地方有：
  - 函数和方法
  - 大段代码
  - 复杂的算法
  - Hack

<!-- more -->

**变量和函数命名**

- 变量应为名词
- 函数名应以动词开始，返回布尔值类型的函数一般以 is 开头
- 变量和函数都应该使用合乎逻辑的名字，不要担心长度。

**变量类型透明**

- 变量初始化
- 匈牙利标记法，如 bFound(布尔值)、iCount(整数)等
- 用类型注释，如`var found /* Boolean */ = false;`

#### 松散耦合

- 解耦 HTML/JavaScript，尽量不使用内联`<script>`或者用 HTML 属性分配事件处理程序，尽量不用 JavaScript 直接创建大量 HTML 内容。
- 解耦 CSS/JavaScript，避免直接操作元素的`style`属性，最好使用修改`className`的方式。
- 解耦应用逻辑/事件处理程序。

#### 编程实践

**尊重对象所有权**

如果你不负责维护某个对象、它的对象或者它的方法，那么你就不能对他们进行修改。

- 不要为实例或原型添加属性；
- 不要为实例或原型添加方法；
- 不要重定义已存在的方法。

最佳的方法是永远不修改不是由你所有的对象，可以通过以下方式为对象创建新的功能：

- 创建包含所需功能的新对象，并用它与相关对象进行交互。
- 创建自定义类型，继承需要进行修改的类型。然后为自定义类添加额外功能。

**避免全局变量**

使用命名空间，如：

```javascript
//创建全局对象。
var Wrox = {};

//为Professional JavaScript创建命名空间
Wrox.ProJS = {};

//将书中用到的对象附加上去
Wrox.ProJS.EventUtil = {};
Wrox.ProJS.CookieUtil = {};
```

**避免与 null 进行比较**

- 如果值为一个引用类型，使用`instanceof`操作符检查其构造函数；
- 如果值为一个基本类型，使用`typeof`检查其类型
- 如果是希望对象包含某个特定的方法名，则使用`typeof`操作符确保指定名字的方法存在于对象上。
- 代码中的`null`比较少，就容易确定代码的目的，并消除不必要的错误。

**使用常量**

关键在于将数据和使用它的逻辑进行分离，要注意的值的类型如下：

- 重复值：多处用到的值。
- 用户界面字符串：任何用于显示给用户的字符串，都应该被抽取出来以方便国际化。
- URLs：在 Web 应用中，资源位置很容易变更，推荐使用一个公用的地方存放所有的 URL。
- 任意可能会更改的值。

### 性能

#### 注意作用域

随着作用域链中的作用域数量的增加，访问当前作用域意外的变量的时间也在增加。访问全局变量的速度比访问局部变量慢啊，因为要遍历作用域链。

**避免全局查找**

全局查找的开销比直接使用局部变量的开销要大，要在函数中多把全局变量存储为局部变量。

**避免 with 语句**

`with`语句会创建自己的作用域，会增加作用域链长度，会导致执行代码速度变慢。

#### 选择正确方法

**避免不必要的属性查找**

尽可能多地使用局部变量将属性查找替换为值查找。

**优化循环**

- 减值迭代，从最大值开始迭代。
- 简化终止条件，每次循环过程都会计算终止条件，所以必须保证它尽可能快。
- 简化循环体
- 使用后测试循环

**展开循环**

- 当循环的次数是确定的，消除循环并使用多次函数调用往往更快。
- 循环次数不一定，使用 Duff 装置处理循环。

``` javascript
var iterations = Math.floor(values.length/8);
var leftover = values.length%8;
var i = 0;

if(leftover > 0){
    do{
        process(values[i++]);
    }while(--leftover > 0);
}

do {
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
}while(--iterations > 0);
```

**避免双重解释**

当 JavaScript 代码想解析 JavaScript 的时候就会存在双重解释惩罚。这种情况要尽量避免出现。

**性能的其他注意事项**

- 原生方法较快
- switch语句较快
- 位运算符较快

#### 最小化语句数

完成多个操作的单个语句要比完成单个操作的多个语句快。

- 多个变量声明变成使用逗号分隔的一句变量声明。
- 插入迭代值
- 使用数组和对象字面量

#### 优化 DOM 交互

- 最小化现场更新，使用`document.createDocumentFragment()`更新节点。
- 使用`innerHTML`，使用它要比 DOM 方法快。
- 使用事件代理，可以避免为每个节点都绑定事件处理程序
- 注意 HTMLCollection，访问它都会在文档进行查询，开销很小，需要减少它的访问次数。发生以下情况会返回 HTMLCollection 对象：
  - `getElementsByTagName()`；
  - 获取元素的`childNodes`属性；
  - 获取元素的`attributes`属性；
  - 访问了特殊的集合，`document.forms`、`document.images`等。

### 部署

#### 构建过程

前端的代码不应该原封不动的放在浏览器中，理由如下：

- 知识产权问题
- 文件大小
- 代码组织

可以用 Ant 构建。

#### 验证

主要使用 JSLint，现在要使用 ESLint。

#### 压缩

- 文件压缩，一般会删除额外的空白和注释，缩短变量名。
- HTTP 压缩，一般由浏览器进行。




## Chapter25. 新兴的 API

### 动画

#### 早期动画循环

使用`setInterval()`方法控制所有动画，不过定时器并不能保证动画按时执行。

平滑动画的最佳循环间隔是 1000ms/60，即 17ms。

#### 循环间隔的问题

浏览器的计时精度并不是到毫秒级的，也会对动画实际效果有影响。

#### requestAnimationFrame()

使用如下方式保证`requestAnimationFrame()`的兼容性。这个方法用于告诉浏览器某些 JavaScript 代码要执行动画。解决了浏览器不知道 JavaScript 动画什么时候开始、不知道最佳循环间隔和不知道代码什么时候执行的问题。

``` javascript
window.requestAnimFrame = (function(){
  return  window.requestAnimationFrame     ||
        window.webkitRequestAnimationFrame ||
        window.mozRequestAnimationFrame    ||
        window.msRequestAnimationFrame     ||
        window.oRequestAnimationFrame      ||
        function( callback ){
            window.setTimeout(callback, 1000/60);
        };
})();
  
window.cancelAnimFrame = (function (){
    return window.cancelAnimationFrame     ||
        window.cancelRequestAnimationFrame ||
        window.webkitCancelAnimationFrame  ||  
        window.mozCancelAnimationFrame     ||  
        window.msCancelAnimationFrame      ||  
        window.oCancelAnimationFrame       ||  
        window.clearTimeout;
})();
```

### Page Visibility API

该 API 是为了让开发人员知道页面是否对用户可见而推出的。这个 API 本身非常简单，由以下几部分组成：

- `document.hidden`：表示页面是否隐藏，包括在后台标签页中或者浏览器最小化。
- `document.visibilityState`：表示下列 4 个可能的状态值。
  - 页面在后台标签页或浏览器最小化
  - 页面在前台标签页中。
  - 实际的页面已经隐藏，但用户可以看到页面的预览。
  - 页面在屏幕外执行预渲染处理。
- `visibilitychange`事件：当文档从可见变为不可见或从不可见变为可见时，触发该事件。

### Geolocation API

地理定位的实现是`navigator.geoloaction` 对象，这个对象包含 3 个方法。

第一个方法是`getCurrentPosition()`，调用这个方法就会触发请求用户共享地理定位信息的对话框。这个方法接收 3 个参数：成功回调函数、可选的失败回调函数和可选的选项对象。

其中，成功回调函数会接受到一个 Position 对象参数，有两个属性`coords`和`timestamp`。

`coords`可能包含如下信息：

- `latitude`：以十进制数表示的纬度。
- `longtitude`：以十进制数表示的经度。
- `accuracy`：经纬度坐标精度，以米为单位。
- `altitude`：以米为单位的还不高度，如果没有相关数据则为`null`。
- `alitudeAccracy`：海拔高度的精度，以米为单位。
- `heading`：指南针的方向，0 度表示正北，值为 NaN 表示没有检测到数据。
- `speed`：速度，即每秒移动多少米，没有则为`null`。

第二个参数即失败回调函数，在被调用的时候也会接收到一个参数，这个参数包含两个属性：`message`和`code`。

第三个参数是一个配置选项对象，用于设定信息的类型。可以设置的选项有三个：

- `enableHighAccuracy`：表示必须尽可能使用最准确的位置信息。
- `timeout`：表示等待位置欣欣的最长时间。ma
- `maximumAge`表示上一次取得的坐标信息的有效时间。

``` javascript
navigator.geolocation.getCurrentPosition(function (position){
    drawMapCenteredAt(position.coords.latitude, position.coords.longtitude);
}, function (error){
    console.log("Error code: " + error.code);
    console.log("Error message: " + error.message);
}, {
    enableHighAccuracy: true,
    timeout: 5000,
    maximumAge: 25000
});
```

如果希望跟踪用户的位置，可以用`watchPosition()`方法，参数与`getCurrentPostion()`相同。调用该方法会返回一个数值标识符，可以用于传入`clearWatch()`方法取消监控操作。

``` javascript
var watchId - navigator.geolocation.watchPosition(function (position){
    drawMapCenteredAt(position.coords.latitude, position.coords.longtitude);
}, function (error){
    console.log("Error code: " + error.code);
    console.log("Error message: " + error.message);
});

clearWatch(watchId);
```

### File API

File API 在表单中的文件传入字段的基础上，又添加了一些直接访问文件信息的接口。HTML5 在 DOM 中为文件输入元素添加了一个`files`集合。每个  File 对象都有下列属性：

- `name`：本地文件系统中的文件名
- `size`：文件的字节大小。
- `type`：字符串，文件的 MIME 类型。
- `lastModifiedData`：字符串，文件上一次被修改的时间。

``` javascript
var filesList = document.getElementById("files-list");

EventUtil.addHandler(filesList, "change", function(event){
    var info = "",
        output = document.getElementById("output"),
        files = EventUtil.getTarget(event).files,
        i = 0,
        len = files.length;

    while (i < len){
        info += files[i].name + " (" + files[i].type + ", " + files[i].size + " bytes)<br>";
        i++;
    }

    output.innerHTML = info;
});
```

#### FileReader 类型

通过 FileReader 类型可以读取文件中数据，是一种异步文件读取机制，可以想象成 XMLHttpRequest。提供以下几个方法：

- `readAsText(file, encoding)`：以纯文本形式读取文件，将读取到的文本保存在`result`属性中。第二个参数用于指定编码类型（可选的）。
- `readAsDataURL(file)`：读取文件并将文件以数据 URI 的形式保存在`result`属性中。
- `readAsBinaryString(file)`：读取文件并将一个字符串保存在`result`属性中，字符串中的每个字符表示一字节。
- `readAsArrayBuffer(file)`：读取文件并将一个包含文件内容的 ArrayBuffer 保存在`result`属性中。

由于读取过程是异步的，因此 FileReader 也提供了几个事件，比较实用的是`progress`、`error`、`load`这三个事件。下面是使用上述三个事件的例子：

``` javascript
var filesList = document.getElementById("files-list");
EventUtil.addHandler(filesList, "change", function(event){
    var info = "",
        output = document.getElementById("output"),
        progress = document.getElementById("progress"),
        files = EventUtil.getTarget(event).files,
        type = "default",
        reader = new FileReader();

    if (/image/.test(files[0].type)){
        reader.readAsDataURL(files[0]);
        type = "image";
    } else {
        reader.readAsText(files[0]);
        type = "text";
    }
    reader.onerror = function(){
        output.innerHTML = "Could not read file, error code is " + reader.error.code;
    };
    reader.onprogress = function(event){
        if (event.lengthComputable){
            progress.innerHTML = event.loaded + "/" + event.total;
        }
    };
    reader.onload = function(){
        var html = "";

        switch(type){
            case "image":
                html = "<img src=\"" + reader.result + "\">";
                break;
            case "text":
                html = reader.result;
                break;

        }
        output.innerHTML = html;
    };
});
```

要中断读取过程，可以调用`abort()`方法，会触发`abort`事件。在触发`load`、`error`或`abort`事件后，会触发`loadend`事件。

#### 读取部分内容

File 对象还支持一个`slice()`方法，该方法接受两个参数：起始字节和要读取的字节数。会返回一个 Blob 实例，Blob 是 File 类型的父类型。下面是一个通用函数：

``` javascript
function blobSlice(blob, startByte, length){
    if (blob.slice){
        return blob.slice(startByte, length);
    } else if (blob.webkitSlice){
        return blob.webkitSlice(startByte, length);
    } else if (blob.mozSlice){
        return blob.mozSlice(startByte, length);
    } else {
        return null;
    }
}
```

Blob 类型有一个`size`属性和`type`属性。下例只读取文件的 32B 内容。

``` javascript
var filesList = document.getElementById("files-list");
EventUtil.addHandler(filesList, "change", function(event){
    var info = "",
        output = document.getElementById("output"),
        progress = document.getElementById("progress"),
        files = EventUtil.getTarget(event).files,
        reader = new FileReader(),
        blob = blobSlice(files[0], 0, 32);

    if (blob){
        reader.readAsText(blob);

        reader.onerror = function(){
            output.innerHTML = "Could not read file, error code is " + reader.error.code;
        };

        reader.onload = function(){
            output.innerHTML = reader.result;
        };
    } else {
        alert("Your browser doesn't support slice().");
    }
});
```

#### 对象 URL

也称为 blob URL，指的是引用保存在 File 或 Blob 数据中的 URL。使用它的好处是可以不必把文件内容读取到 JavaScript 中而直接使用文件内容。创建 URL 对象，可以使用`window.URL.createObjectURL()`方法，并传入 File 或 Blob 对象。通用函数：

``` javascript
function createObjectURL(blob){
    if (window.URL){
        return window.URL.createObjectURL(blob);
    } else if (window.webkitURL){
        return window.webkitURL.createObjectURL(blob);
    } else {
        return null;
    }
}
```

该函数的返回值是一个字符串，指向一块内存地址。因为这个字符串是 URL，所以在 DOM 中也能使用。如下，可以显示图像文件：

``` javascript
EventUtil.addHandler(filesList, "change", function(event){
    var info = "",
        output = document.getElementById("output"),
        progress = document.getElementById("progress"),
        files = EventUtil.getTarget(event).files,
        reader = new FileReader(),
        url = createObjectURL(files[0]);

    if (url){
        if (/image/.test(files[0].type)){
            output.innerHTML = "<img src=\"" + url + "\">";
        } else {
            output.innerHTML = "Not an image.";
        }
    } else {
        output.innerHTML = "Your browser doesn't support object URLs.";
    }
});
```

直接把对象 URL 放在`<img>`标签中，省去了在 JavaScript 中读数据的麻烦。`<img>`会找到相应内存地址，直接读取数据并将图像显示在页面中。

如果不需要相应数据，可以用`window.URL.revokeObjectURL()`释放内存。

#### 读取拖放的文件

通过`drop`的`event`对象的`event.dataTransfer.files`属性，可以读取被放置的文件。代码如下：

``` javascript
var droptarget = document.getElementById("droptarget");
            
function handleEvent(event){
    var info = "",
        output = document.getElementById("output"),
        files, i, len;            

    EventUtil.preventDefault(event);

    if (event.type == "drop"){
        files = event.dataTransfer.files;
        i = 0;
        len = files.length;

        while (i < len){
            info += files[i].name + " (" + files[i].type + ", " + files[i].size + " bytes)<br>";
            i++;
        }

        output.innerHTML = info;
    }
}

EventUtil.addHandler(droptarget, "dragenter", handleEvent);
EventUtil.addHandler(droptarget, "dragover", handleEvent);
EventUtil.addHandler(droptarget, "drop", handleEvent);
```

#### 使用 XHR 上传文件

先创建 FormData 对象，通过它调用`append()`方法并传入相应 File 对象作为参数，然后再把 FormData 对象传递给 XHR 的`send()`方法。

``` javascript
var droptarget = document.getElementById("droptarget");
            
function handleEvent(event){
    var info = "",
        output = document.getElementById("output"),
        data, xhr,
        files, i, len;            

    EventUtil.preventDefault(event);

    if (event.type == "drop"){
        data = new FormData();
        files = event.dataTransfer.files;
        i = 0;
        len = files.length;

        while (i < len){
            data.append("file" + i, files[i]);
            i++;
        }

        xhr = new XMLHttpRequest();
        xhr.open("post", "FileAPIExample06Upload.php", true);
        xhr.onreadystatechange = function(){
            if (xhr.readyState == 4){
                alert(xhr.responseText);
            }
        };
        xhr.send(data);
    }
}

EventUtil.addHandler(droptarget, "dragenter", handleEvent);
EventUtil.addHandler(droptarget, "dragover", handleEvent);
EventUtil.addHandler(droptarget, "drop", handleEvent);
```

### Web 计时

核心是`window.performance`对象。页面的所有度量信息，都包含在这个对象里面。Web Timing 规范一开始就为`performance`对象定义了两个属性：

- `redirectCount`：页面加载前的重定向次数。
- `type`：数值常量，表示刚刚发生的导航类型。
  - performance.navigation.TYPE_NAVIGATE（0）：页面第一次加载。
  - performance.navigation.TYPE_RELOAD（1）：页面重载过。
  - performance.navigation.TYPE_BACK_FORWARD（2）：页面是通过“后退”或“前进”按钮打开的。

另外，`performance.timing`属性也是一个对象，但这个对象的属性都是时间戳，不同事件会产生不同的值。如下:

- navigationStart
- unloadEventStart
- unloadEventEnd
- redirectStart
- redirectEnd
- fetchStart
- domainLookupStart
- domainLookupEnd
- connectStart
- connectEnd
- secureConnectStart
- requestStart
- responseStart
- responseEnd
- domLoading
- domInteractive
- domContentLoadedEventStart
- domContentLoadedEventEnd
- domComplete
- loadEventStart
- loadEventEnd

### Web Workers

Web Workers 规范通过让 JavaScript 在后台运行解决了单线程的缺点，不会影响用户体验。

#### 使用 Worker 

实例化 Worker 对象并传入要执行的 JavaScript 文件名就可以创建一个新的 Web Worker。如：

``` javascript
var worker = new Worker("stufftodo.js");
```

这行代码会导致浏览器下载`stufftodo.js`，但只有 Worker 接收到消息才会实际执行文件中的代码。要给 Worker 传递消息，可以使用`postMessage()`方法。与 XDM 不同的是，在所有支持的浏览器中，`postMessage()`都能接收对象参数。

然后通过`message`和`error`事件与页面通信。来自 Worker 的数据保存在`event.data`中。任何时候，只要调用`terminate()`方法就可以停止 Worker 的工作。而且，Worker 中的代码会立即停止执行，后续过程都不会再发生。

#### Worker 全局作用域

Web Worker 中的全局对象是`worker`对象本身。也就是说，在这个特殊的全局作用域中，`this`和`self`引用的都是`worker`对象。为便于处理数据，Web Worker 本身也是一个最小化的运行环境。

- 最小化的`navigator`对象，包括`onLine`、`appName`、`appVersion`、`userAgent`和`platform`属性；
- 只读的`location `对象；
- 定时器方法；
- XMLHttpRequest 构造函数。

``` javascript
// Web Worker内部代码
self.onmessage = function(event){
    var data = event.data;
    data.sort(function(a, b){
        return a - b;
    });
    
    self.postMessage(data);
};
```

``` javascript
// 页面中
var data = [23,4,7,9,2,14,6,651,87,41,7798,24],
    worker = new Worker("WebWorkerExample01.js");

worker.onmessage = function(event){
    alert(event.data);
};

worker.postMessage(data);
```

在 Worker 内部，调用`close()`方法也可以停止工作。

#### 包含其他脚本

既然无法在 Worker 中动态创建新的`<script>`元素，但可以调用`importScripts()`方法。这个方法接受一个或多个指向 JavaScript 文件的 URL。每个加载过程都是异步进行的，因此所有脚本加载并执行之后，`importScripts()`才会执行。例如：

``` javascript
importScripts("file1.js", "file2.js")
```

