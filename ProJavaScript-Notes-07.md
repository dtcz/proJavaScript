---
title: '重读 JS 高程（Chapter08、Chapter09）'
date: 2017-08-31 18:18:21
tags: javascript
category: javascript
---

## Chapter08. BOM

在 Web 中使用 JavaScript ，BOM 是真正的核心。BOM 提供了很多对象，用于访问浏览器的功能。

### window对象

BOM 的核心对象是 window，它表示浏览器的一个实例。window 既是通过 JavaScript 访问浏览器窗口的一个接口，又是 ECMAScript 规定的 Global 对象。这意味着网页中定义的任何一个对象、变量和函数，都以 window 作为其 Global 对象。

#### 全局作用域 

所以在全局作用域中声明的变量、函数都会变成 window 对象的属性和方法。

定义全局变量与在 window 对象上直接定义属性有差别：全局变量不能通过 delete 操作符删除，而直接定义在 window 对象上的可以。例如：

``` javascript
var age = 29;
window.color = "red";

delete window.age;
delete window.error;   //returns true

alert(window.age);    //29
alert(window.color);  //undefined
```

尝试访问未声明的变量会抛出错误，但是通过查询 window 对象，可以知道某个可能未声明的变量是否存在。例如：

``` javascript
//抛出错误，因为 oldValue 未定义
var newValue = oldValue;
//不会抛出错误，因为这是属性查询，
var newValue = window.oldValue;   //undefined
```

#### 窗口关系及框架

``` html
<html>
  <head>
    <title>Frameset Example</title>
  </head>
  <frameset rows="160,*">
    <frame src="frame.htm" name="topFrame">
    <frameset cols="50%,50%">
      <frame src="anotherframe.htm" name="leftFrame">
      <frame src="yetanotherframe.htm" name="rightFrame">
    </frameset>
  </frameset>
</html>  
```

top 对象始终指向最高（最外层）的框架，也就是浏览器窗口。建议用`top.frames[0]`或者`top.frames["topFrame"]`来引用上方的框架。

与 top 相对的另一个 window 对象是 parent。顾名思义，parent 对象始终指向当前框架的直接上层框架。

与框架相关的最后一个对象是 self，它始终指向 window；实际上，self 和 window 可以互换使用。引入 self 只为与 top 和 parent 对象对应起来。

#### 窗口位置

确定和修改 window 对象位置的属性，表示青口相对于屏幕左边和上边的位置

- `screenLeft`，`screenTop`：IE，Safari，Opera 和 Chrome 支持
- `screenX`，`screenY`：Firefox，Safari，Opera 和 Chrome 支持

``` javascript
var leftPos = (typeof window.screenLeft == "number") ? window.screenLeft : window.screenX;
var topPos = (typeof window.screenTop == "number") ? window.screenTop : window.screenY;
```

IE, Opera 中，这两个属性保存的是从屏幕左边和上边由 window 对象表示的页面可见区的距离，也就是说如果存在工具栏，那么 window.screenTop 是工具栏像素高度。但是 Chrome，Firefox，Safari 中，保存的是整个浏览器窗口相对屏幕的坐标值。

还有两个方法可以将窗口移动到一个新位置（有可能被浏览器禁用）：

- `window.moveTo(x, y)`：接收新位置的 x 和 y 坐标值
- `window.moveBy(dx, dy)`：接收水平和垂直方向上移动的像素数。

<!-- more -->

#### 窗口大小

- `innerWidth`，`innerHeight`，`outerWidth`，`outerHeight`：Firefox，IE，Safari 中`outerWidth`和`outerHeight`返回浏览器窗口本身尺寸；`innerWidth`和`innerHeight`表示该容器页面视图区大小（减去边框宽度）。Chrome 中它们均返回视口（viewport）大小而非浏览器窗口大小。
- `document.documentElement.clientWidth`，`document.documentElement.clientHeight`，`document.body.clientWidth`，`document.body.clientWidth`：可以用来确定视口大小。

最终取得浏览器视口大小的代码：

``` javascript
var pageWidth = window.innerWidth,
    pageHeight = window.innerHeight;

if (typeof pageWidth != "number"){
    if (document.compatMode == "CSS1Compat"){
        pageWidth = document.documentElement.clientWidth;
        pageHeight = document.documentElement.clientHeight;
    } else {
        pageWidth = document.body.clientWidth;
        pageHeight = document.body.clientHeight;
    }
}
```

同样存在两个方法可以调整浏览器窗口大小，不过可能被浏览器禁用

- `window.resizeTo(x, y)`：接收浏览器窗口新宽度和新高度。
- `window.resizeBy(dx, dy)`：接收新窗口与原窗口的宽度和高度之差。

#### 导航和打开窗口

使用`window.open()`方法既可以导航到一个特定的 URL，也可以打开一个新的浏览器窗口。这个方法可以接受 4 个参数：

- 要加载的 URL
- 窗口目标，可以是已存在的窗口或者框架，如"topFrame"、`_self`、`_parent`、`_top`、`_blank`
- 一个特性字符串，包括`height`,`left`,`location`,`menubar`,`resizable`,`scrollbars`,`status`,`toolbar`,`top`,`width`  


- 一个表示新页面是否取代浏览器历史记录中当前加载页面的布尔值

``` javascript
window.open("http://www/wrox.com/", "wroxWindow", "height=400, width=400, top=10, left=10, resizable=yes");
```

#### 间歇调用和超时调用

JavaScript 是单线程语言，但它允许通过设置超时值和间歇时间值来调度代码在特定的时刻执行。

- `setTimeout()`：超时调用函数，接受两个参数，要执行的代码和以毫秒表示的时间。
  - 第一个参数可以是一个包含 JavaScript 代码的字符串，也可以是一个函数
  - 第二个参数是一个表示等待多长时间的毫秒数，但经过该时间后指定的代码不一定会执行。因为 JavaScript 是一个单线程的解释器，因此一定时间只能执行一段代码。有一个任务队列可以控制要执行的代码，这些任务会按照将它们添加到队列的顺序执行。`setTimeout()`的第二个参数告诉 JavaScript 再过多长时间把当前任务添加到队列中。如果队列是空的，那么添加的代码会立即执行。
  - 返回值是一个数值 ID，它是计划执行代码的唯一标识符，可以通过它来取消超时调用。
- `clearTimeout()`：传入超时调用 ID，取消该调用。

``` javascript
var timeoutId = setTimeout(function (){
   alert("Hello World!"); 
}, 1000);

clearTimeout(timeoutId);
```

- `setInterval()`：间歇调用函数，与`setTimeout()`接收的参数相同，不过会按照指定的时间间隔重复执行代码，直至间歇调用被取消或者页面被卸载。
- `clearInterval()`：取消尚未执行的间歇调用。

``` javascript
var num = 0;
var max = 10;
var intervalId = null;

function incrementNumber() {
    num++;

    if (num == max) {
        clearInterval(intervalId);
        alert("Done");
    }
}

intervalId = setInterval(incrementNumber, 500);
```

以上代码也可以用超时调用实现：

``` javascript
var num = 0;
var max = 100;

function incrementNumber() {
    num++;
  
    if (num < max) {
        setTimeout(incrementNumber, 500);
    } else {
        alert("Done");
    }
}

setTimeout(incrementNumber, 500);
```

实际开发中很少使用间歇调用，因为最后一个间歇调用可能会在前一个间歇调用结束之前启动，使用超时调用模拟可以完全避免这一点。

#### 系统对话框

- `alert()`：接受一个字符串并将其显示给用户
- `confirm()`：与`alert()`基本相同，不过多了确认和取消按钮，会返回一个布尔值（OK/true，Cancel/false）；
- `prompt()`：除了显示确认和取消，还会显示一个文本输入域，接受两个参数，显示给用户的文本提示，和文本输入域的默认值，单击确认会返回文本输入域的值，单击取消会返回`null`
- `window.print()`：显示“打印”对话框
- `window.find()`：显示“查找”对话框

### location 对象

location 是最有用的 BOM 对象之一，它提供了与当前窗口中加载的文档有关的信息，还提供了一些导航功能。location 也是一个很特别的对象，它既是 window 对象的属性，也是 document 对象的属性。下标列出 location 对象的所有属性：

| 属性名      | 例子                    | 说明                                       |
| -------- | --------------------- | ---------------------------------------- |
| hash     | "#contents"           | 返回URL中的hash（#号跟零或多个字符），无hash返回空字符串       |
| host     | "www.wrox.com8080"    | 返回服务器名称和端口号                              |
| hostname | "www.wrox.com"        | 返回不带端口号的服务器名称                            |
| href     | "http://www.wrox.com" | 返回当前加载页面的完整URL。location.toString()也返回这个值 |
| pathname | "/WileyCDA/"          | 返回URL中的目录和（或）文件名                         |
| port     | "8080"                | 返回URL中指定的端口号，无端口返回空字符串                   |
| protocol | "http:"               | 返回页面使用的协议。通常是http:或https:                |
| search   | "?q=javascript"       | 返回URL的查询字符串，以问号开头                        |

#### 查询字符串参数

``` javascript
function getQueryStringArgs(){    
    var qs = location.search.length > 0 ? location.search.substring(1) : "",
        args = {},
        items = qs.length ? qs.split("&") : [],
        item, name, value,
        i = 0,
        len = items.length;
  
    for (i = 0; i < len; i++){
        item = items[i].split("=");
        name = decodeURIComponent(item[0]);
        value = decodeURIComponent(item[1]);

        if (name.length){
            args[name] = value;
        }
    }
    return args;
}
//query：?q=javascript&num=10
var args = getQueryStringArgs();

alert(args["q"]);     //"javascript"
alert(args["num"]);   //"10"
```

#### 位置操作

- `location.assign()`：接受一个 URL 作为参数，可以立即打开新 URL 并在浏览器历史记录中生成一条记录。如果是将`location.href`或`window.location`设置为一个URL值，也会以该值调用`assign()`方法。
- `location.replace()`：接受一个 URL 作为参数，可以立即打开新 URL ，但是不会生成历史记录。
- `location.reload()`：重新加载页面，如果不传递参数，浏览器就会以最有效的方式重新加载，如果该页面自上次请求后没有改变过，页面就会从浏览器缓存中重新加载。如果要强制从服务器重新加载，则需要传递参数`true`。

### navigator对象

navigator 对象已经成为识别客服端浏览器的事实标准。下表列出一些 navigator 的属性和方法：

| 属性或方法                     | 说明                      |
| ------------------------- | ----------------------- |
| appCodeName               | 浏览器的名称，通常都是Mozilla      |
| appName                   | 完整的浏览器名称                |
| appVersion                | 浏览器版本                   |
| cookieEnabled             | 表示cookie是否启用            |
| javaEnabled()             | 表示当前浏览器中是否启用的Java       |
| language                  | 浏览器的主语言                 |
| mimeTypes                 | 在浏览器中注册的MIME类型数组        |
| onLIne                    | 表示浏览器是否链接到了因特网          |
| platform                  | 浏览器所在的系统平台              |
| plugins                   | 浏览器中安装的插件信息的数组          |
| userAgent                 | 浏览器的用户代理字符串             |
| vendor                    | 浏览器的品牌                  |
| product                   | 产品名称                    |
| registerContentHandler()  | 针对特定的MIME类型将一个站点注册为处理程序 |
| registerProtocolHandler() | 针对特定的协议将一个站点注册为处理程序     |

#### 检测插件

``` javascript
//plugin detection - doesn't work in IE
function hasPlugin(name){
    name = name.toLowerCase();
    for (var i=0; i < navigator.mimeTypes.length; i++){
        if (navigator.mimeTypes[i].name.toLowerCase().indexOf(name) > -1){
            return true;
        }
    }

    return false;
}
//detect flash
alert(hasPlugin("Flash"));
//detect quicktime
alert(hasPlugin("QuickTime"));     
//detect Java
alert(hasPlugin("Java"));

//plugin detection for IE
function hasIEPlugin(name){
    try {
        new ActiveXObject(name);
        return true;
    } catch (ex){
        return false;
    }
}

//detect flash
alert(hasIEPlugin("ShockwaveFlash.ShockwaveFlash"));
//detect quicktime
alert(hasIEPlugin("QuickTime.QuickTime"));

```

注：plugins 集合有一个名叫 refresh() 的方法，用于刷新 plugins 以反映最新安装的插件。这个方法接受一个参数，表示是否应该重新加载页面的一个布尔值。设置为 true，则会重新加载包含插件的所有页面；否则，只更新 plugins 集合，不重新加载页面。

#### 注册处理程序

`registerContentHandler()`，`registerProtocolHandler()`：这两个方法可以让一个站点指明它可以处理特定类型的消息，比如 RSS 和在线电子邮件程序。

- `registerContentHandler()`：接收三个参数：要处理的 MIME 类型、可以处理该 MIME 类型页面的 URL 以及应用程序名称。

``` javascript
navigator.registerContentHandler("application/rss+xml", "https://www.somereader.com?feed=%s","Some Reader");
```

- `registerProtocolHandler()`：接收三个参数：要处理的协议、可以处理该协议页面的 URL 以及应用程序名称。

``` javascript
navigator.registerProtocolHandler("mailto", "https://www.somemailclient.com?cmd=%s","Some Mail Client");
```

### screen 对象

screen 对象使用率不是特别高，比较常见的使用场景是移动端。下表列出一些常见属性：

| 属性          | 说明                    |
| ----------- | --------------------- |
| availHeight | 屏幕像素高度减系统部件高度之后的值（只读） |
| availLeft   | 未被系统部件占用的最左侧的像素值（只读）  |
| availTop    | 未被系统部件占用的最上方的像素值（只读）  |
| availWidth  | 屏幕像素宽度减系统部件宽度之后的值（只读） |
| colorDepth  | 用于表示颜色的位数（只读）         |
| height      | 屏幕像素高度                |
| width       | 屏幕像素宽度                |
| pixelDepth  | 屏幕的位深（只读）             |

### history 对象

history 对象保存着用户上网的历史记录，从窗口被打开的那一刻算起。

- `go()`：使用`go()`方法可以在用户的历史记录中任意跳转，可以向前也可以向后，只接受一个参数。传递整数，表示向后或者向前跳转的页面的一个整数值。负数表示后退，整数表示前进。也可以传递字符串，此时浏览器会跳转到历史记录中包含该字符串的第一个位置。

``` javascript
history.go(-1);   //后退一页
history.go(1);    //前进一页
history.go(2);    //前进两页
history.go("wrox.com") // 跳转到最近的wrox页面
```

- `back()`，`forward()`：后退和前进，用来代替`go()`。
- `length`：history 对象还有一个`length`属性，保存着历史记录的数量。对于加载到窗口、标签页或者框架中的第一个页面，history.length 等于 0。



## Chapter09. 客户端检测

由于浏览器间存在差别，通常要根据不同浏览器的能力分别编写不同的代码。

### 能力检测

最常用也最为人们广泛接受的客户端检测形式的能力检测（又称特性检测）。能力检测的目标不是识别特定的浏览器，而是识别浏览器的能力。采用这种方式只要确定浏览器支持特定的能力，就可以给出解决方案。能力检测的基本模式如下：

``` javascript
if (object.propertyInQuestion) {
    // 使用 object.propertyInQuestion
}
```

举例来说，IE5.0 之前的版本不支持`document.getElementById()`这个DOM方法，尽管可以使用非标准的`document.all`属性实现相同的目的。于是，出现了如下能力检测代码：

``` javascript
function getElement(id){
    if(document.getElementById){
        return document.getElementById(id);
    } else if (document.all) {
        return document.all[id];
    } else {
        throw new Error("No way to retrieve element!")
    }
}
```

理解能力检测要理解两个概念：

- 第一个概念就是先检测达成目的的最常用的特性。
- 第二个概念就是必须测试实际要用到的特性。一个特性存在，不意味另一个特性也存在。

检测某个或某几个特性并不能确定浏览器，事实上，根据浏览器不同将能力组合起来是更可取的方式。最好一次性检测所有相关特性，而不要分别检测。

### 怪癖检测

与能力检测类似，怪癖检测（quirks detection）的目标是识别浏览器的特殊行为。但与能力检测确认浏览器支持什么能力不同，怪癖检测是想要知道浏览器存在什么缺陷（“怪癖”也就是 bug）。这通常需要运行一小段代码，以确定某一特性不能正常工作。

一般来说，“怪癖”都是个别浏览器的个别版本独有的，一般被归为 bug。在新版本中可能会也可能不会被修复。由于检测“怪癖”涉及运行代码，因此我们建议仅检测那些对你有直接影响的“怪癖”。而且最好在脚本一开始就执行此类检测，以遍今早解决问题。

### 用户代理检测

第三种，也是争议最大的一种客户端检测技术叫做**用户代理检测**。用户代理检测通过检测用户代理字符串来确定实际使用的浏览器。可以通过`navigator.userAgent`属性访问用户代理字符串。在服务器端，这是一种广为接受的做法，但是在客户端，用户代理则被当作一种万不得已才用的做法，其优先级排在前两种方法之后。

``` javascript
var ua = navigator.userAgent;
```

#### Opera

``` javascript
if(window.opera){
    engine.ver = browser.ver = window.opera.version();
    engine.opera = browser.opera = parseFloat(engine.ver);
}
```

#### WebKit(Chrome, Safari)

``` javascript
if(/AppleWebKit\/(\S+)/.test(ua)){ //WebKit 的 ua 中包含“Gecko”和“KHTML”， “AppleWebKit” 独一无二
    engine.ver = RegExp["$1"];
    engine.webkit = parseFloat(engine.ver);
  
    //分辨Chrome与Safari
    if (/Chrome\/(\S+)/.test(ua)){
        browser.ver = RegExp["$1"];
        browser.chrome = parseFloat(browser.ver);
    } else if (/Version\/(\S+)/.test(ua)){
        browser.ver = RegExp["$1"];
        browser.safari = parseFloat(browser.ver);
    } else {
        //approximate version
        var safariVersion = 1;
        if (engine.webkit < 100){
            safariVersion = 1;
        } else if (engine.webkit < 312){
            safariVersion = 1.2;
        } else if (engine.webkit < 412){
            safariVersion = 1.3;
        } else {
            safariVersion = 2;
        }   

        browser.safari = browser.ver = safariVersion;        
    }
}
```

#### KHTML

``` javascript
if (/KHTML\/(\S+)/.test(ua) || /Konqueror\/([^;]+)/.test(ua)){
    engine.ver = browser.ver = RegExp["$1"];
    engine.khtml = browser.konq = parseFloat(engine.ver);
}
```

#### Gecko(Firefox)

``` javascript
if (/rv:([^\)]+)\) Gecko\/\d{8}/.test(ua)){    
      engine.ver = RegExp["$1"];
      engine.gecko = parseFloat(engine.ver);

      //检测 Firefox
      if (/Firefox\/(\S+)/.test(ua)){
          browser.ver = RegExp["$1"];
          browser.firefox = parseFloat(browser.ver);
      }
  }
```

####MSIE(IE)

``` javascript
if (/MSIE ([^;]+)/.test(ua)){    
    engine.ver = browser.ver = RegExp["$1"];
    engine.ie = browser.ie = parseFloat(engine.ver);
}
```

#### OS（操作系统）

``` javascript
var p = navigator.platform;
system.win = p.indexOf("Win") == 0;
system.mac = p.indexOf("Mac") == 0;
system.x11 = (p == "X11") || (p.indexOf("Linux") == 0);

//detect windows operating systems
if (system.win){
    if (/Win(?:dows )?([^do]{2})\s?(\d+\.\d+)?/.test(ua)){
        if (RegExp["$1"] == "NT"){
            switch(RegExp["$2"]){
                case "5.0":
                    system.win = "2000";
                    break;
                case "5.1":
                    system.win = "XP";
                    break;
                case "6.0":
                    system.win = "Vista";
                    break;
                case "6.1":
                    system.win = "7";
                    break;
                default:
                    system.win = "NT";
                    break;                
            }                            
        } else if (RegExp["$1"] == "9x"){
            system.win = "ME";
        } else {
            system.win = RegExp["$1"];
        }
    }
}   
```

#### Mobile

``` javascript
system.iphone = ua.indexOf("iPhone") > -1;
system.ipod = ua.indexOf("iPod") > -1;
system.ipad = ua.indexOf("iPad") > -1;
system.nokiaN = ua.indexOf("NokiaN") > -1;

//windows mobile
if (system.win == "CE"){
    system.winMobile = system.win;
} else if (system.win == "Ph"){
    if(/Windows Phone OS (\d+.\d+)/.test(ua)){;
        system.win = "Phone";
        system.winMobile = parseFloat(RegExp["$1"]);
    }
}


//determine iOS version
if (system.mac && ua.indexOf("Mobile") > -1){
    if (/CPU (?:iPhone )?OS (\d+_\d+)/.test(ua)){
        system.ios = parseFloat(RegExp.$1.replace("_", "."));
    } else {
        system.ios = 2;  //can't really detect - so guess
    }
}

//determine Android version
if (/Android (\d+\.\d+)/.test(ua)){
    system.android = parseFloat(RegExp.$1);
}
```

#### Game

``` javascript
//gaming systems
system.wii = ua.indexOf("Wii") > -1;
system.ps = /playstation/i.test(ua);
```

