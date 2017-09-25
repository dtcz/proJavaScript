---
title: 重读 JS 高程（Chapter16-Chapter19）
tags: javascript
category: javascript
date: 2017-09-22 10:15:46
---


## Chapter16. HTML5 脚本编程

### 跨文档消息传送

跨文档消息传送（cross-document messaging），简称 XDM，指的是在来自不同域的页面间传递消息。

XDM 的核心是`postMessage()`方法，除了 XDM 部分之外的其他部分也会提到这个方法名，但都是为了同一个目的：向另一个地方传递数据。对于 XDM 而言，另一个地方指的是包含在当前页面中的`<iframe>`元素，或者由当前页面弹出的窗口。

`postMessage()`方法接受两个参数：一条消息和一个表示消息接收方来自哪个域的字符串。第二个参数对保障安全通信非常重要，可以防止浏览器把消息发送到不安全的地方。如下：

``` javascript
var iframeWindow = document.getElementById("myFrame").contentWindow;  
iframeWindow.postMessage("A secret", "http://www.wrox.com");  //发送方代码  
```

最后一行代码尝试向内嵌框架中发送一条消息，并指定框架中的文档必须来源于`http://www.wrox.com`域。如果来源匹配，消息会传送到内嵌框架中；否则，`postMessage()`什么都不做。

接收到 XDM 消息后，会触发`window`对象的`message`事件，这个事件是以异步触发的。触发`message`事件后，传递给`onmessage`处理程序的事件对象包含以下三方面重要信息：

- data：作为`postMessage()`第一个参数传入的字符串数据。
- origin：发送消息的文档所在的域，例如`http://www.wrox.com` 。
- source：发送消息的文档的`window`对象的代理。这个代理主要用于在发送上一条消息的窗口中调用`postMessage()`方法。如果发送消息的窗口来自同一个域，那这个对象就是`window`。

``` javascript
window.addEventListener("message", function(){  
    if(event.origin == "http://www.wrox.com"){  
        processMessage(event.data); //处理接收的数据  
        event.source.postMessage("Received!", "http://p2p.wrox.com"); //可选：向来源窗口发送回执  
    }  
}, false);
```

最早`postMessage()`只能传入字符串，后来改成了允许传入任何数据结构，但并非所有浏览器都实现了这一变化，一般对传入的数据使用`JSON.stringify()`。XDM的兼容 IE8+ 和 其他主流浏览器，也支持相同域的页面间使用。

<!-- more -->

### 原生拖放

从 IE4 时代起，网页中的元素就可以被拖动了，HTML5 以 IE 的实例为基础制定了拖放规范。

#### 拖放事件

拖动元素过程中会依次触发下列事件：`dragstart`、`drag`、`dragend`。这三个事件的目标都是被拖动的元素。`drag`事件会在拖动过程中被持续地触发，与`mousemove`事件相似。

当元素被拖动到一个有效的放置目标上时会依次触发下列事件：`dragenter`、`dragover`、`dragleave`或者`drop`。当被拖放目标在放置目标的范围内移动时会持续地触发`dragover`事件。如果元素被放到了放置目标中，则会触发`drop`事件而不是`dropleave`事件。

#### 自定义放置目标

当元素在不允许放置的目标上会出现禁止图标，释放元素时不会触发`drop`事件。可以通过取消`dragenter`和`dragover`事件的默认行为将任何元素转变为允许放置的元素。在 Firefox 中，放置事件的默认行为是打开被拖放的目标的 URL，因此我们还需要取消`drop`事件的默认行为，阻止它打开 URL。

#### dataTransfer 对象

拖放事件的事件对象拥有`dataTransfer`属性，用于从被拖放元素向放置目标传递字符串数据。

`dataTransfer`对象有两个主要的方法：

- `setData()`：保存数据，这个方法接受两个参数，保存的数据类型和要保存的数据字符串。
- `getData()`：取得数据，这个方法接受一个参数，即保存的数据类型。

数据类型允许指定各种 MIME 类型，由于要向后兼容，HTML5 也支持"text"和"URL"，不过它们会被映射为"text/plain"和"text/uri-list"。`dataTransfer`对象可以为每种 MIME 都存储一个值，不过只能在`drop`事件处理程序中处理。

在拖动文本框中的文本时，浏览器会调用`setData()`方法，将拖动的文本以"text"格式保存在`dataTransfer`对象中。在拖放图片或者链接时，也会调用`setData()`方法将链接以"URL"格式保存在`dataTransfer`对象中。

#### dropEffect 和 effectAllowed

利用`dataTransfer`对象不只能传输数据，还能通过它来确定被拖动的元素以及作为放置目标的元素能够接收什么操作。需要访问`dataTransfer`对象的两个属性`dropEffect`和`effectAllowed`。

通过`dropEffect`属性可以知道被拖动的元素能够执行哪种放置行为。这个属性有下列的 4 个可能值。

- "none"：不能把拖动的元素放在这里。这是除文本框外所有元素的默认值。
- "move"：应该把拖动的元素移动到放置目标。
- "copy"：应该把拖动的元素复制到放置目标。
- "link"：表示放置目标会打开拖动的元素（拖动的元素必须是一个链接，有 URL）。

这个属性只能改变光标样式，光标所指示的动作需要通过编程实现，这个属性必须在 `dropenter`事件处理程序中针对放置目标设置。

`dropEffect`属性只有搭配`effectAllowed`属性才有用。它表示允许拖动元素的哪种`dropEffect`，可能值如下：

- "uninitialized"：没有给被拖动元素设置任何放置行为
- "none"：被拖动元素不能有任何行为。
- "copy"：只允许值为"copy"的`dropEffect`。 
- "link"：只允许值为"link"的`dropEffect`。 
- "move"：只允许值为"move"的`dropEffect`。 
- "copyLink"：只允许值为"copy"和"link"的`dropEffect`。 
- "copyMove"：只允许值为"copy"和"move"的`dropEffect`。 
- "linkMove"：只允许值为"link"和"move"的`dropEffect`。 
- "all"：允许任意`dropEffect`。

必须在`dragstart`事件中设置`effectAllowed`属性。

#### 可拖动

默认情况下图像、链接和文本是可以拖动的，其他的元素可以通过`draggable`属性设置为可拖动。IE10+ 支持。

```javascript
<div draggable="true">...</div>
```

#### 其他成员

`dataTransfer`对象还包含下列方法和属性。

- `addElement(element)`：为拖动操作添加一个元素。添加这个元素只影响数据（即增加作为拖动源而相应回调的对象），不会影响拖动操作时页面元素的外观。
- `clearData(format)`：清除以特定格式保存的数据。
- `setDragImage(element, x, y)`：指定一幅图像，当拖动发生时，显示在光标下方。该方法接收的三个参数分别是要显示的 HTML 元素和光标在图像中的 x、y 坐标。
- `types`：当前保存的数据类型，是一个类似数组的集合。

### 媒体元素

HTML5 新增了两个与媒体相关的标签，让开发人员不必依赖任何插件就能再网页中嵌入跨浏览器的音频和视频内容。这两个标签就是`<video>`和`<audio>`。使用这两个元素时，至少要在标签中包含`src`属性，指向要加载的媒体文件。还可以设置`width`和`height`属性指定视频播放器大小，而为`poster`属性指定图像 URI 可以在加载视频内容期间显示一副图像。如果标签中有`controls`属性，则意味着浏览器应该显示 UI 控件。

由于并非所有浏览器都支持所有媒体格式，所以可以指定多个不同的媒体来源。为此，不用在标签中指定`src`属性，而是像下面这样显示一个或多个`<source>`元素。 

``` html
<!--  嵌入视频-->
<video id="myVideo">
    <source scr="1.webm" type="video/webm; codecs='vp8, vorbis'">
    <source scr="2.ogv" type="video/ogg; codecs='theora, vorbis'">
    <source scr="3.mpg">
    Video player not available.
</video>

<!--  嵌入音频 -->
<audio id="myAudio">
    <source scr="song1.ogg" type="audio.ogg">
    <source scr="song2.mp3" type="audio.mpeg">
    Audio player not available.
</audio>
```

#### 属性

| 属性                  | 描述                                   |
| ------------------- | ------------------------------------ |
| audioTracks         | 返回表示可用音轨的 AudioTrackList 对象          |
| autoplay            | 设置或返回是否在加载完成后随即播放音频/视频               |
| buffered            | 返回表示音频/视频已缓冲部分的 TimeRanges 对象        |
| controller          | 返回表示音频/视频当前媒体控制器的 MediaController 对象 |
| controls            | 设置或返回音频/视频是否显示控件（比如播放/暂停等）           |
| crossOrigin         | 设置或返回音频/视频的 CORS 设置                  |
| currentSrc          | 返回当前音频/视频的 URL                       |
| currentTime         | 设置或返回音频/视频中的当前播放位置（以秒计）              |
| defaultMuted        | 设置或返回音频/视频默认是否静音                     |
| defaultPlaybackRate | 设置或返回音频/视频的默认播放速度                    |
| duration            | 返回当前音频/视频的长度（以秒计）                    |
| ended               | 返回音频/视频的播放是否已结束                      |
| error               | 返回表示音频/视频错误状态的 MediaError 对象         |
| loop                | 设置或返回音频/视频是否应在结束时重新播放                |
| mediaGroup          | 设置或返回音频/视频所属的组合（用于连接多个音频/视频元素）       |
| muted               | 设置或返回音频/视频是否静音                       |
| networkState        | 返回音频/视频的当前网络状态                       |
| paused              | 设置或返回音频/视频是否暂停                       |
| playbackRate        | 设置或返回音频/视频播放的速度                      |
| played              | 返回表示音频/视频已播放部分的 TimeRanges 对象        |
| preload             | 设置或返回音频/视频是否应该在页面加载后进行加载             |
| readyState          | 返回音频/视频当前的就绪状态                       |
| seekable            | 返回表示音频/视频可寻址部分的 TimeRanges 对象        |
| seeking             | 返回用户是否正在音频/视频中进行查找                   |
| src                 | 设置或返回音频/视频元素的当前来源                    |
| startDate           | 返回表示当前时间偏移的 Date 对象                  |
| textTracks          | 返回表示可用文本轨道的 TextTrackList 对象         |
| videoTracks         | 返回表示可用视频轨道的 VideoTrackList 对象        |
| volume              | 设置或返回音频/视频的音量                        |

#### 事件

| 事件名称              | 描述                                       |
| ----------------- | ---------------------------------------- |
| abort             | 在播放被终止时触发，当播放中的视频重新开始播放时会触发这个事件          |
| canplay           | 在媒体数据已经有足够的数据（至少播放数帧）可供播放时触发。`readyState`为 2 |
| canplaythrough    | 在媒体的`readyState`为 3 时触发，表明媒体可以在保持当前的下载速度的情况下不被中断地播放完毕。注意：手动设置`currentTime`会使得 Firefox 触发一次`canplaythrough`事件，其他浏览器或许不会如此 |
| durationchange    | `duration`属性发生了改变。例如，在媒体已被加载足够的长度从而得知总长度时会触发这个事件 |
| emptied           | 媒体被清空（初始化）时触发                            |
| ended             | 播放结束时触发                                  |
| error             | 在发生错误时触发                                 |
| loadeddata        | 媒体的第一帧已经加载完毕                             |
| loadedmetadata    | 媒体的元数据已经加载完毕，现在所有的属性包含了它们应有的有效信息         |
| loadstart         | 在媒体开始加载时触发                               |
| mozaudioavailable | 当音频数据缓存并交给音频层处理时                         |
| pause             | 播放暂停时触发                                  |
| play              | 在媒体回放被暂停后再次开始时触发                         |
| playing           | 在媒体开始播放时触发（不论是初次播放、在暂停后恢复、或是在结束后重新开始）    |
| progress          | 告知媒体相关部分的下载进度时周期性地触发，有关媒体当前已下载总计的信息可以在元素的`buffered`属性中获取到 |
| ratechange        | 在播放速率变化时触发                               |
| seeked            | 在跳跃完成时触发                                 |
| seeking           | 在跳跃开始时触发                                 |
| stalled           | 在尝试获取媒体数据，但数据不可用时触发                      |
| suspend           | 在媒体资源加载终止时触发，这可能是因为下载已完成或因为其他原因暂停        |
| timeupdate        | 元素的`currentTime`属性表示的时间已经改变              |
| volumechange      | 在音频音量改变时触发（既可以是`volume`属性改变，也可以是`muted`属性改变） |
| waiting           | 在一个待执行的操作（如播放）因等待另一个操作（如跳跃或下载）被延迟时触发     |

#### 自定义媒体播放器

使用`<audio>`和`<video>`元素的`play()`和`pause()`方法，可以手工控制媒体文件的播放。

``` html
<div class="mediaplayer">
    <div class="video">
        <video id="player" src="movie.mov" poster="mymovie.jpg" width="300" height="200">
            Video player not available.
        </video>
    </div>
    <div class="controls">
        <input type="button" value="Play" id="video-btn">
        <span id="curtime">0</span>/<span id="duration">0</span>
    </div>
</div>
```

``` javascript
var player = document.getElementById("player"),
    btn = document.getElementById("video-btn"),
    curtime = document.getElementById("curtime"),
    duration = document.getElementById("duration");

duration.innerHTML = player.duration;            

EventUtil.addHandler(btn, "click", function(event){
    if (player.paused){
        player.play();
        btn.value = "Pause";
    } else {
        player.pause();
        btn.value = "Play";
    }
});

setInterval(function(){
    curtime.innerHTML = player.currentTime;
}, 250);
```

#### 检测编解码器的支持情况

两个媒体元素都有一个`canPlayType()`方法，该方法接收一种格式/编解码字符串，返回"probably"、"maybe"或""。

#### Audio 类型

`<audio>`元素还有一个原生的 JavaScript 构造函数 Audio，可以在任何时候播放音频。

### 历史状态管理

通过状态管理 API，能够在不加载新页面的情况下改变浏览器的 URL。需要使用`history.pushState()`方法，该方法接受三个参数：状态对象、新状态的标题和可选的相对 URL。

``` javascript
history.pushState({name: "Nicholas"}, "Nicholas' page", "nicholas.html");
```

执行`pushState()`后，新的状态信息就会被加入历史状态栈，而浏览器地址栏也会变成新的相对 URL。但是浏览器不会真的向服务器发送请求，即使状态改变以后查询`location.href`，也会返回与地址栏相同地址。另外，第二参数目前还没有浏览器实现，因此完全可以只传入一个空字符串，或者一个短标题。而第一个参数则应该尽可能提供初始化页面状态所需的各种信息。

按下“后退”或“前进”按钮会触发`window`的`popstate`事件，该事件对象有一个`state`属性，这个属性包含着当初以第一个参数传递给`pushState()`的状态对象。

更新当前状态，可以调用`replaceState()`，传入参数与`pushState()`的前两个参数相同。调用这个方法不会再历史状态栈中创建新状态，只会重写当前状态。

## Chapter17. 错误处理与调试

### 错误处理

错误处理在程序设计中的重要性是毋庸置疑的，任何有影响力的web应用程序都需要一套完善的错误处理机制。

#### try-catch语句

try-catch 语句是 JavaScript 处理异常的一种标准方式，try 块中的任何代码发生了错误，就会立即退出代码执行过程，然后接着执行 catch 块，catch 块会接收到一个包含错误信息的对象，一般用该对象的`message`属性获取错误信息。如下：

``` javascript
try {
    // 可能会导致错误的代码
} catch (error){
    // 处理错误
    alert(erorr.message);
}
```

**finally 子句**

finally 子句一经使用，其代码无论如何都会被执行。

```javascript
try{
    return 2;
}catch(err){
    return 1;
}finally{
    // finally子句无论如何都会执行
    return 0;
}
```

**错误类型**

JavaScript 有 7 种错误类型，其中，Error 是基类型，其他错误都继承自该类型。

- Error
- EvalError
- RangeError
- ReferenceError
- SyntaxError
- TypeError
- URIError

#### 抛出错误

使用`throw`语句，可以抛出上述的错误，也可以抛出自定义的错误；在遇到`throw`操作符时，代码会立刻停止执行。

```javascript
// 抛出常规错误
throw new SyntaxError("I don't like your syntax");
throw new TypeError("What type of variable do you take me for?");
throw new RangeError("Sorry, you just dont have the range");
throw new EvalError("That't doesn't evaluate.");
throw new URIError("Uri, is that you?");
throw new ReferenceError("You didn‘t cite your references properly.");

// 抛出自定义错误
function CustomError(message){
    this.name = "CustomError";
    this.message = message;
}

CustomError.prototype = new Error();
throw new CustomError("My message");
```

#### 错误（error）事件

任何没有通过 try-catch 处理的错误都会触发`window`对象的`error`事件。任何 Web 浏览器中，`onerror`事件处理程序都不会创建`event`对象，但可以接受三个参数：错误消息、错误所在的 URL 和行号。

``` javascript
window.onerror = function(message, url, line){
    alert(message);
    return false; //阻止浏览器报告错误的默认行为
}
```

图像也支持`error`事件，只要图像的`src`特性的 URL 不能反悔可以被识别的图像格式，就会触发`error`事件。此时的`erorr`事件遵循 DOM 格式，会返回一个以图像为目标的`event`对象。

``` javascript
var image = new Image();

EventUtil.addHandler(image, "load", function(event){
    alert("Image loaded!");
});

EventUtil.addHandler(image, "error", function(event){
    alert("Image not loaded!");
});

image.src = "smilex.gif";
```

#### 常见的错误类型

一般来说，需要关注三种错误：

- 类型转换错误
- 数据类型错误
- 通信错误

**类型转换错误**

为避免类型转换错误，建议使用全等操作符：

```javascript
alert(5 == "5");//true
alert(5 === "5");//false
```

流控制语句也很容易出错。像 if 之类的语句，在确定下一步操作之前，会自动把任何值转化成布尔值：

```javascript
function concat(str1, str2, str3){
    var result = str1 + str2;
    if(str3){ //当str3是大于0的数字，不是string类型时，也会执行
        resutl += str3;
    }
    return result;

}

// 正确的写法
function concat(str1, str2, str3){
    var result = str1 + str2;
    if(typeof str3 == "string"){ 
        resutl += str3;
    }
    return result;
}
```

**数据类型错误**

在将变量传递个函数时，对变量进行类型进行检测，可以避免类型错误：

```javascript
// 不安全的函数，任何非字符串都会导致错误
function getQueryString(url) {
    var pos = url.indexOf("?");
    if (pos > -1) {
        return url.substring(pos + 1);
    }
    return "";
}

// 安全的函数
function getQueryString(url) {
    if (typeof url == "string") {//通过检查类型确保安全
        var pos = url.indexOf("?");
        if (pos > -1) {
            return url.substring(pos + 1);
        }
        return "";
    }
}
```

在确切知道应该传入什么类型的情况下，应该使用`instanceof`来检测类型：

```javascript
// 不安全的函数，任何非数组类型都会导致错误
function reverseSort(values){
    if(values){
        values.sort();
        valuse.reverse();
    }
}

// 不安全的函数，任何非数组类型都会导致错误
function reverseSort(values){
    if(values != null){
        values.sort();
        valuse.reverse();
    }
}

// 不安全的函数，不能只针对某个特性进行检测
function reverseSort(values){
    if(typeof values.sort == "function"){
        values.sort();
        valuse.reverse();
    }
}

// 安全，非数组将被忽略
function reverseSort(values){
    if(values instanceof Array){
        values.sort();
        valuse.reverse();
    }
}
```

**通信错误**

常见的数据通信错误是把数据发给服务器之前，没有使用`encodeURIComponent()`对数据进行编码，可以定义一个处理查询字符串的函数：

```javascript
function addQueryStringArg(url, name ,value){
    if(url.indexOf("?") == -1){
        url += "?";
    }else{
        url += "&";
    }
    url += encodeURIComponent(name) + "=" +encodeURIComponent(value);
    return url;
}

var url = "http://www.baidu.com";
var newUrl = addQueryStringArg(url, "redir", "http://wwww.someotherdomain.com?a=b&b=c");
alert(newUrl);
```

#### 区分致命错误与非致命错误

非致命错误（这类错误没有必要对用户给出提示）：

- 不影响用户的主要任务
- 只影响页面的一部分
- 可以恢复
- 重复相同操作可以消除错误

致命错误（这类错误应该立即给用户发送消息）：

- 应用程序根本无法运行
- 错误明显影响到了用户的主要操作
- 会导致其他连带错误

### 调试技术

#### 将消息记录到控制台

可以使用`console`对象向 JavaScript 控制台写入消息来调试，这个对象具有下列方法：

- `error(message)`：将错误消息记录到控制台
- `info(message)`：将信息性消息记录到控制台
- `log(message)`：将一般消息记录到控制台
- `warn(message)`：将警告消息记录到控制台

```javascript
function sum(num1, num2){
    console.log("Enter sum(), arguements are " + num1 + ", " +num2 );

    console.log("Before calculation");
    var result = num1 + num2;
    console.log("After calculaation");

    console.log("Exiting sum()");
    return result;
}
```

#### 将消息记录在当前页面

在页面开辟一小块信息，用于显示错误信息：

```javascript
function log(message){
    var console = document.getElementById("debugInfo");
    if(console == null){
        console = document.createElement("div");
        console.id = "debugInfo";
        console.style.background = "#dedede";
        console.style.border = "1px solid silver";
        console.style.padding = "5px";
        console.style.width = "400px";
        console.style.position = "absolute";
        console.style.right = "0px";
        console.style.top = "0px";
        document.body.appendChild(console);
    }
    console.innerHTML += "<p>" + message + "</p>";
}
```

#### 抛出错误

如果错误消息很具体，基本可以把它当做错误来源的依据，那么可以直接抛出错误：

```javascript
function divide(num1, num2){
    if(typeof num1 != "number" || typeof num2 != "number"){
        throw new Error("devide (): both arguments must be number.");
    }
    return num1/num2;
}
```

也可以自定义一个`assert`语句：

```javascript
//assert接受两个参数，一个是求值结果应该为true的条件，一个是条件为false时需要抛出的错误
function assert(condition , message){
    if(!condition){
        throw new Error(message);
    }
}
function divide(num1, num2){
    assert(typeof num1 == "number" && typeof num2 == "number", "devide (): both arguments must be number.");
    return num1/num2;
}
```

### 常见的 IE 的错误

IE 是最难调试 JavaScript 错误的浏览器，以下介绍一些错误：

#### 操作中止

对于 IE8 之前的版本，在修改尚未加载完的页面时，会发生操作中止错误。

#### 无效字符

无效字符是 JavaScript 中没有定义的字符，例如直接从 word 中赋值文本到编辑器里，然后又在 IE 中运行，则可能会遇到无效字符的错误。

#### 未找到成员

IE 中的所有 DOM 对象都是以 COM 对象实现的，这会导致一些与垃圾收集相关的非常奇怪的行为。IE 中的未找到的成员错误，就是由于垃圾收集例程配合错误所直接导致的。

具体来说，如果在对象被销毁之后，又给该对象赋值，就会导致未找到成员错误。如下：

``` javascript
document.onclick = function (){
    var event = window.event;
    setTimeout(function (){
        event.returnValuee = false;  // 未找到成员错误
    }, 1000);
}
```

#### 未知运行时错误

当使用`innerHTML`或者`outerHTML`以下列方式指定 HTML 时，就会发生未知运行时错误：

- 把块级元素插入到行内元素
- 访问表格的任意部分的任意属性

#### 语法错误

如果引用外部的 JavaScript 文件，而该文件最终没有返回 JavaScript 代码，IE 会抛出语法错误。

#### 系统无法找到指定资源

当 URL 的长度超过 IE 对 URL 的最长不能超过 2083 个字符的限制时，就会发生这个错误。如下：

```javascript
function createLongUrl(url){
    var s = "?";
    for(var i = 0; i < 2500; i++){
        s += "a";
    }
    return url + s;
}

var x = new XMLHttpRequest();
x.open("get", createLongUrl("http://www.baidu.com/"), true);
x.send(null);　
```

## Chapter18. JavaScript 与 XML

## Chapter19. E4X

这两章是 XML 相关，目前前端用这个的不多，基本都用JSON，先跳过。







