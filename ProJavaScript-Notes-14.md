---
title: 重读 JS 高程（Chapter20、Chapter21）
tags: javascript
category: javascript
date: 2017-09-23 01:03:30
---


## Chapter20. JSON

JSON 是一种数据格式，不是一种编程语言。虽然具有相同的语法形式，但 JSON 并不从属于 JavaScript。

### 语法

JSON 的语法可以表示一下三种类型的值。

- 简单值：使用与 JavaScript 相同的语法，可以在 JSON 表示字符串、数值、布尔值和`null`。但 JSON 不支持 JavaScript 中的特殊值`undefined`。
- 对象：对象作为一种复杂的数据类型，表示的是一组无序的键值对。每个键值对中的值可以是简单值，也可以是复杂的数据类型的值。
- 数组：数组也是一种复杂数据类型，表示一组有序的值的列表，可以通过数值索引来访问其中的值。数组的值也可以是任意类型。

JSON 不支持变量、函数或者对象实例，它就是一种表示结构化数据的格式。

#### 简单值

> 最简单的 JSON 数据形式就是简单值。

值得注意的是，JSON 字符串必须使用双引号，其他和 JavaScript 没有区别。

#### 对象

JSON 中的对象要求给属性加双引号，属性的值可以是复杂类型。如下：

``` 
{
    "name": "Nicholas",
    "age": 29,
    "school": {
        "name": "Merrimack College",
        "location": "North Andover, MA"
    }
}
```

#### 数组

JSON 的数组可以与对象结合起来，构成复杂的数据结构。

<!-- more -->

### 解析与序列化

JSON 的流行是因为可以把它直接转换成 JavaScript 对象，比用 DOM 方法解析 XML 要容易许多。

#### JSON 对象

早期的 JSON 解析器基本上就是用 JavaScript 的`eval()`函数。由于 JSON 是 JavaScript 语法的子集，因此`eval()`函数可以解析、解释并返回 `JavaScript`对象和数组。

ES5 对解析 JSON 的行为进行了规范，定义了全局对象 JSON。该对象有两个方法：`stringify()`和`parse()`。最简单的情况下，这两个方法分别用于把 JavaScript 对象序列化为 JSON 字符串和把 JSON 字符串解析为原生 JavaScript 值。

序列化 JavaScript 对象时，所有函数及原型成员都会被有意忽略，不会体现在结果，另外值为 undefined 的任何属性也都会被跳过。最终值都是有效的 JSON 数据类型。

#### 序列化选项

`JSON.stringify()`除了要序列化的 JavaScript 对象外，还可以接收另外两个参数，这两个参数用于指定以不同的方式序列化 JavaScript 对象。第一个参数是个过滤器，可以是一个数组，也可以是一个函数；第二个参数是一个选项，表示是否在 JSON 字符串中保留缩进。

**过滤结果**

如果过滤器参数是数组，那么`JSON.stringify()`的结果将只包含数组中列出的属性。如下：

``` javascript
var book = {
    title: "Professional JavaScript",
    authors: [
        "Nicholas C. Zakas"
    ],
    edition: 3,
    year: 2011
};

var jsonText = JSON.stringify(book, ["title", "edition"]);
alert(jsonText); // {"title":"Professional JavaScript","edition":3}
```

如果第二个参数是一个函数，传入的函数接收两个参数，属性名和属性值。根据属性名可以知道应该如何处理要序列化的对象中的属性，函数的返回值就是相应属性的值。如下：

``` javascript
var book = {
    title: "Professional JavaScript",
    authors: [
        "Nicholas C. Zakas"
    ],
    edition: 3,
    year: 2011
};

var jsonText = JSON.stringify(book, function(key, value){
    switch(key){
        case "authors":
            return value.join(",")

        case "year":
            return 5000;

        case "edition":
            return undefined;

        default:
            return value;
    }
});
```

**字符串缩进**

`JSON.stringify()`方法的第三个参数用于控制结果中的缩进和空白符。如果这个参数是一个数值，那它表示的是每个级别缩进的空格数。要在每个级别缩进 4 个空格，可以这样设置：

``` javascript
var book = {
    title: "Professional JavaScript",
    authors: [
        "Nicholas C. Zakas"
    ],
    edition: 3,
    year: 2011
};

var jsonText = JSON.stringify(book, null, 4);
```

最大缩进空格数为 10，所有大于 10 的值都会自动转换为 10。

如果缩进参数是一个字符串而不是数值，则这个字符串将在 JSON 字符串中被用作缩进字符。如下：

``` javascript
var book = {
   "title": "Professional JavaScript",
    "authors": [
        "Nicholas C. Zakas"
    ],
    edition: 3,
    year: 2011
};

var jsonText = JSON.stringify(book, null, "--");
```

**toJSON() 方法**

有些情况下，可以给对象定义`toJSON()`方法，返回其自身的 JSON 数据格式。原生 Date 对象有一个`toJSON()`方法，能够将 JavaScript 的 Date 对象自动转换成 ISO 8601 日期字符串。可以为任何对象添加`toJSON()`方法，如下：

``` javascript
var book = {
   "title": "Professional JavaScript",
    "authors": [
        "Nicholas C. Zakas"
    ],
    edition: 3,
    year: 2011,
    toJSON: function(){
        return this.title;
    }
};

var jsonText = JSON.stringify(book);
```

用`JSON.stringify()`序列化对象的顺序：

- 如果存在`toJSON()`方法而且能够通过它取得有效值，则调用该方法，否则返回对象本身。
- 如果提供了第二个参数，应用这个函数过滤器。传入函数过滤器的值是上一步返回的值。
- 进行序列化。
- 有第三个参数，执行格式化。

#### 解析选项

`JSON.parse()`方法也可以接收另一个参数，该参数是一个函数，将在每个键值对上调用。这个函数与`JSON.stringify()`的第二参数类似，不过被被称为还原函数。如下：

``` javascript
var book = {
   "title": "Professional JavaScript",
    "authors": [
        "Nicholas C. Zakas"
    ],
    edition: 3,
    year: 2011,
    releaseDate: new Date(2011, 11, 1)
};

var jsonText = JSON.stringify(book);
var bookCopy = JSON.parse(jsonText, function(key, value){
    if (key == "releaseDate"){
        return undefined;
    } else {
        return value;
    }
});

alert("releaseDate" in bookCopy); // fasle
```



 ## Chapter21. Ajax 与 Comet

Ajax 是 Asynchronous JavaScript and XML 的缩写，在 2005 年被发现后，成为了沿用至今的浏览器端与服务器端的交互方式。

### XMLHttpRequest 对象

```javascript
function createXHR(){
    if (typeof XMLHttpRequest != "undefined"){
        return new XMLHttpRequest();
    } else if (typeof ActiveXObject != "undefined"){
        if (typeof arguments.callee.activeXString != "string"){
            var versions = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0", "MSXML2.XMLHttp"];

            for (var i = 0, len = versions.length; i < len; i++){
                try {
                    new ActiveXObject(versions[i]);
                    arguments.callee.activeXString = versions[i];
                    break;
                } catch (ex){
                    //skip
                }
            }
        }

        return new ActiveXObject(arguments.callee.activeXString);
    } else {
        throw new Error("No XHR object available.");
    }
}

var xhr = createXHR();
```

#### XHR 的用法

使用 XHR 对象时，要调用的第一个方法是`open()`，它接受 3 个参数：要发送的请求类型（"get"、"post"等）、请求的 URL 和表示是否异步发送请求的布尔值。如下：

```javascript
xhr.open("get", "exeample.php", false);
```

注意：

- URL 是相对于执行代码的当前页面（当然也可以使用绝对路径）
- `open()`方法并不会真正发送请求，而只是启动一个请求以备发送。

要发送请求，必须调用`send()`方法，接受一个参数，即要作为请求主体发送的数据。如果不需要发送数据，则传入`null`。调用`send()`之后，请求就会被分派到服务器。

```javascript
xhr.open("get", "example.txt", false);
xhr.send(null);
```

这次请求是同步请求，JavaScript 代码会等到服务器响应后再继续执行。收到服务器响应后，相应的数据会自动填充 XHR 对象的属性，相关属性如下：

- `responseText`：作为响应主体被返回的文本。
- `responseXML`：如果响应的内容类型是"text/xml"或"application/xml"，这个属性中将保存包含着响应数据的 XML DOM 文档。
- `status`：响应的 HTTP 状态。
- `statusText`：HTTP 的说明。

接收到响应，第一步是检查`status`属性，一般来说 HTTP 状态代码为 200 时响应成功，304 表示资源未修改，可以直接使用缓存，也意味着响应有效。

```javascript
// 为确保接收到适当的响应 200:成功，304:资源未被修改
if((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
    console.log(xhr.responseText);
}
```

注意：

- 有的浏览器会错误的报告成功状态码为 204 
- 无论内容类型是什么，响应主体的内容都会保存到`responseText`属性中；而对于 XML 数据而言，`responseXML`同时也将被赋值，否则其值为`null`。

对于异步请求，可以检测 XHR 对象的`readyState`属性，该属性表示请求/响应过程的当前活动阶段：

- 0：未初始化。尚未调用`open()`方法。
- 1：启动。已经调用`open()`方法，但尚未调用`send()`方法。
- 2：发送。已经调用`send()`方法，但尚未接收到响应。
- 3：接收。已经接收到部分响应数据。
- 4：完成。已经接收全部响应数据，而且已经可以在客户端使用了。

`readyState`值发生变化，会触发`readystatechange`事件。可以利用这个事件来检测每次状态变化后`readyState`的值。不过，必须在调用`open()`之前指定`onreadystatechange`事件处理程序才能确保跨浏览器兼容性。

```javascript
var xhr = createXHR();        
xhr.onreadystatechange = function(){
    if (xhr.readyState == 4){
        if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
            console.log(xhr.responseText);
        } else {
            console.log("Request was unsuccessful: " + xhr.status);
        }
    }
};
xhr.open("get", "example.txt", true);
xhr.send(null);
```

在接收到响应数据之前可以调用`abort()`方法来取消异步请求：

```javascript
xhr.abort();
xhr = null; // 解除引用，释放内存
```

#### HTTP 头部信息

默认情况下，在发送 XHR 请求的同时，还会发送下列头部信息。

- Accept：浏览器能处理的内容类型。
- Accept-Charset：浏览器能显示的字符集。
- Accept-Encoding：浏览器能处理的压缩编码。
- Accept-Language：浏览器当前设置的语言。
- Connection：浏览器与服务器之间连接的类型。
- Cookie：当前页面设置的任何 Cookie。
- Host：发送请求的页面所在的域。
- Referer：发出请求的页面的 URL。
- User-Agent：浏览器的用户代理字符串。

使用`setRequestHeader()`可以设置自定义的请求头信息。这个方法接受两个参数：头部字段的名称和头部字段的值。要成功发送请求头部信息，必须在调用`open()`方法之后且调用`send()`方法之前调用`setRequestHeader()`。 

调用`getResponseHeader()` 方法并传入头部字段名称，可以取得相应的响应头部信息。而调用`getAllResponseHeaders()`可以取得一个包含所有头部信息的长字符串。

#### GET 请求

GET 是最常见的请求类型，最常用语向服务器查询某些信息。必要时可以将查询字符串添加到 URL 末尾，以便将信息发送给服务器。对于 XHR 而言，位于传入`open()`方法的 URL 末尾查询字符串必须经过正确的编码才行。查询字符串的每个参数的名称和值都必须使用`encodeURIComponent()`进行编码。如下：

```javascript
function addURLParam(url, name, value) {
    url += (url.indexOf("?") == -1 ? "?" : "&");
    url += encodeURIComponent(name) + "=" + encodeURIComponent(value);
    return url;
}

var url = "http://test.com";
url = addURLParam(url, "uid" , 5);
url = addURLParam(url, "siteid", 123);  // "http://test.com?uid=5&siteid=123"
xhr.open("get", url, true);
xhr.send(null);
```

#### POST 请求

POST 请求通常用于向服务器发送应该被保存的数据，POST 请求应该把数据作为请求的主体提交。模拟表单提交时，需要要 Content-Type 头部信息设置为 application/x-www-form-urlencoded。

```javascript
/* 发送请求 */
function submitData(){
    var xhr = createXHR();
    xhr.onreadystatechange = function(){
        if (xhr.readyState == 4){
            if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
                alert(xhr.responseText);
            } else {
                alert("Request was unsuccessful: " + xhr.status);
            }
        }
    };

    xhr.open("post", "postexample.php", true);
    // 表单提交的内容类型
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    var form = document.getElementById("user-info");
    // 请求主体为数据
    xhr.send(serialize(form));
}
```

### XMLHttpRequest 2 级

XMLHttpRequest 1 级只是把已有的 XHR 对象的实现细节描述了出来，而 XMLHttpRequest 2 级则进一步发展了 XHR。

#### FormData

FormData 为序列化表单以及创建与表单格式相同的数据提供便利。

```javascript
var data = new FormData();
data.append("name", "Nicholas");
```

`append()`方法接受两个参数：键和值，分别对应表单字段的名字和字段中包含的值。

创建了 FormData 的实例后，可以将它直接传给 XHR 的`send()`方法。如下所示：

```javascript
xhr.open("post", "example.php", true);
var form = document.getElementById("user-info");
xhr.send(new FormData(form));
```

使用 FormData 的方便之处在于不必明确地在 XHR 对象上设置请求头。

#### 超时设定

IE8 为 XHR 对象添加了一个`timeout`属性，表示请求在等待响应多少毫秒后就终止。

```javascript
xhr.open("get", "timeout.php", true);
xhr.timeout = 1000;
xhr.ontimeout = function(){
    alert("Request did not return in a second.");
};        
xhr.send(null);
```

对于其他浏览器的兼容做法

```javascript
xhr.open("get", "timeout.php", true);
xhr.onreadystatechange = function(event){
    if (xhr.readyState == 4){
        clearTimeout(timeout);
        if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
            console.log(xhr.responseText);
        } else {
            console.log("Request was unsuccessful: " + xhr.status);
        }
    }
};

var timeout = setTimeout(function() {
    xhr.abort();
    xhr = null;
}, 1000);
xhr.send(null);
```

#### overrideMimeType() 方法

该方法用于重写 XHR 响应的 MIME 类型。如果服务器返回的 MIME 类型是 text/plain，但数据中实际包含的是 XML。根据 MIME 类型，responseXML 属性中仍然是`null`。此时，通过`overrideMimeType()`方法，可以保证把响应当作 XML 而非纯文本来处理。调用该方法必须在`send()`方法之前。

```javascript
var xhr = createXHR();
xhr.open("get", "text.php", true);
xhr.overrideMimeType("text/xml");
xhr.send(null);
```

### 进度事件

- loadstart：在接收到响应数据的第一个字节时触发。
- progress：在接收响应期间持续不断地触发。
- error：在请求发生错误时触发。
- abort：在因为调用abort()方法而终止时触发。
- load：在接收到完整的响应数据时触发。
- loadend：在通信完成或者触发error、abort或load事件后触发。 

#### load 事件

可以代替`readystatechange`事件。其处理程序会接收到一个`event`对象，其`target`属性指向 XHR 对象实例，因而可以访问到 XHR 对象的所有方法和属性。然而，并非所有浏览器都实现了事件对象。

#### progress 事件

事件处理程序会接收一个`event`对象，其`target`属性指向 XHR 对象实例，而且包含 3 个额外属性：

- `lengthComputable`：是一个表示进度信息是否可用的布尔值。
- `position`：表示已经接收的字节数。
- `totalSize`：根据 Content-Length 响应头部而确定的预期字节数 。

必须在调用`open()`方法之前添加`onprogress`事件处理程序。

```javascript
var xhr = createXHR();        
xhr.onload = function(event){
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
        console.log(xhr.responseText);
    } else {
        console.log("Request was unsuccessful: " + xhr.status);
    }
};
xhr.onprogress = function(event){
    var divStatus = document.getElementById("status");
    if (event.lengthComputable){
        divStatus.innerHTML = "Received " + event.position + " of " + event.totalSize + " bytes";
    }
};
xhr.open("get", "altevents.php", true);
xhr.send(null);
```

### 跨源资源共享

CORS（Cross-Origin Resource Sharing）背后的基本思想，就是使用自定义的 HTTP 头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功还是失败。

在发送请求时，给其附加一个额外的 Origin 头部，其中包含请求页面的源信息（协议、域名和端口），以便服务器根据这个头部信息来决定是否给予响应。

```
Origin: http://www.nozonline.net
```

如果服务器认为这个请求可以接受，在 Access-Control-Allow-Origin 头部中回发相同的源信息（如果是公共资源，可以回发"**"）。如：

```
Access-Control-Allow-Origin: http://www.nozonline.net
```

**注意**：请求和响应都不包含 cookie 信息。

####  IE 对 CORS 的实现

IE8 中引入 XDR（XDomainRequest）类型，用来实现安全可靠的跨域通信。与 XHR 不同之处：

- cookie 不会随请求发送，也不会随响应返回。
- 只能设置请求头部信息中的 Content-Type 字段。
- 不能访问响应头部信息
- 只支持 GET 和 POST 请求

XDR 使用方法类似于 XHR，不过所有的 XDR 请求都是异步的，不能创建同步请求。XDR 提供`contentType` ，这个属性是通过 XDR 对象影响头部信息的唯一方式。

#### 其他浏览器对CORS的实现

通过 XMLHttpRequest 对象实现对 CORS 的原生支持，只需给`open()`方法传入**绝对 URL** 即可，支持同步请求。跨域XHR对象的安全限制： 

- 不能使用`setRequestHeader()`设置自定义头部。 
- 不能发送和接收 cookie。 
- 调用`getAllResponseHeaders()`方法总会返回空字符串。

**建议**：访问本地资源，最好使用相对URL；访问远程资源，使用绝对URL。

#### Preflighted Requests

CORS 通过一种叫做 Preflighted Requests 的透明服务器验证机制支持开发人员使用自定义头部、GET 或 POST 之外的方法，以及不同类型的主体内容。在使用下列高级选项来发送请求时，就会向服务器发送一个 Prefilght 请求。这种请求使用 OPTIONS 方法，发送以下头部。

- Origin：与简单的请求相同。
- Access-Control-Request-Method：请求自身使用方法。
- Access-Control-Request-Headers：（可选）自定义的头部信息，多个头部以逗号分隔。

以下是一个带有自定义头部 NCZ 的使用 POST 方法发送的请求。

``` 
Origin: http://www.nczonline.net
Access-Control-Request-Method: POST
Access-Control-Request-Headers: NCZ
```

发送这个请求后，服务器可以决定是否允许这种类型的请求。服务器通过在响应中发送如下头部与浏览器进行沟通。

- Access-Control-Allow-Origin：与简单的请求相同。
- Access-Control-Allow-Methods：允许的方法，多个方法以逗号分隔。
- Access-Control-Allow-Headers：允许的头部，多个头部以逗号分隔。
- Access-Control-Allow-Max-Age：应该将这个 Preflight 请求缓存多长时间（以秒表示）。

例如：

``` 
Access-Control-Allow-Origin: http://www.ncznoline.net
Access-Control-Allow-Methods: POST, GET
Access-Control-Allow-Headers: NCZ
Access-Control-Allow-Max-Age: 1728000
```

IE 11+ 支持

#### 带凭据的请求

默认情况下，跨域请求不提供凭据（cookie、HTTP认证及客户端 SSL 证明等）。通过将`withCrentials`属性设置为`true`，可以指定某个请求应该发送凭据。如果服务器接收带凭据的请求，会用下面的 HTTP 头部来响应。

``` 
Access-Control-Allow-Credentials: true
```

如果发送的是带凭据的请求，但服务器的响应中没有包含这个头部，那么浏览器就不会把响应交给 JavaScript。另外，服务器还可以在 Preflight 响应中发送这个 HTTP 头部，表示允许源发送带凭据的请求。

#### 跨浏览器的CORS

```javascript
function createCORSRequest(method, url){
    var xhr = new XMLHttpRequest();
    if ("withCredentials" in xhr){ // 检测 XHR 是否支持 CORS 的简单方式，就是检测是否存在 withCredentials 属性
        xhr.open(method, url, true);
    } else if (typeof XDomainRequest != "undefined"){    // IE XDR
        xhr = new XDomainRequest();
        xhr.open(method, url);
    } else {
        xhr = null;
    }
    return xhr;
}

var request = createCORSRequest("get", "http://www.somewhere-else.com/page/");
if (request){
    request.onload = function(){
        //do something with request.responseText
    };
    request.send();
}
```

### 其他跨域技术

利用 DOM 中能够执行跨域请求的功能，在不依赖 XHR 对象的情况下也能发送某种请求，不需要修改服务器端代码。

#### 图像Ping

`<img>`标签，可以从任何网页中加载图像，无需关注是否跨域。这也是广告跟踪浏览量的主要方式。可以通过动态创建图像，使用`onload`和`onerror`事件处理程序缺人是否接受到了响应 。

动态创建图像经常用于图像 Ping。图像 Ping 是与服务器进行简单、单向的跨域通信的一种方式。请求的数据是通过查询字符串的形式发送的，响应可以是任意内容，但通常是像素图或 204 响应。浏览器得不到任何具体的数据，但通过监听 load 和 error 事件，可以知道响应是什么时间接收到的。如下：

```javascript
var img = new Image();
img.onload = img.error = function() {
    console.log("Done!");
};
img.src = "http://www.test.com/getImage?id=1";
```

缺点：

- 只能发送 GET 请求


- 无法访问服务器的响应文本

#### JSONP（JSON with padding）

JSONP 由两部分组成：回调函数和数据。 回调函数是当响应到来时应该在页面调用的函数。回到函数的名字一般是在请求中指定的。而数据是传入回调函数中的 JSON 数据。 JSONP 是通过动态`<script>`元素来使用的，使用时可以为`src`属性指定一个跨域 URL。

```javascript
function handleResponse(response){
    alert("You're at IP address " + response.ip + ", which is in " + response.city + ", " + response.region_name);
}

var script = document.createElement("script");
script.src = "http://freegeoip.net/json/?callback=handleResponse";
document.body.insertBefore(script, document.body.firstChild);
```

优点：能够直接访问响应文本，支持在浏览器与服务器之间双向通信。 

缺点： 

- JSONP 是从其他域中加载代码执行，其安全性无法确保。 
- 不能很容易的确定 JSONP 请求是否失败。

#### Comet

Comet 是一种更高级的 Ajax 技术，是一种服务器向页面推送数据的技术。 有两种实现 Comet 的方式：长轮询和流。 

- 长轮询：页面发起一个到服务器的请求，然后服务器一直保持连接打开，直到有数据可发送。发送完数据之后，浏览器关闭连接，随即又发起一个到服务器的新请求。这一过程在页面打开期间一直持续不断。轮询的优势是所有浏览器都支持。
- HTTP 流：流不同于轮询，它在页面的生命周期内只使用一个 HTTP 连接。具体来说就是浏览器向服务器发送一个请求，而服务器保持连接打开，然后周期性地向浏览器发送数据。所有服务器端语言都支持打印到输出缓存然后刷新（将输出缓存中的内容一次性全部发送到客服端）的功能，这正是实现 HTTP 流的关键所在。通过检测`readyState`是否为 3，就可以利用 XHR 对象实现 HTTP 流。 

```javascript
function createStreamingClient(url, progress, finished){        
    var xhr = new XMLHttpRequest(),
        received = 0;

    xhr.open("get", url, true);
    xhr.onreadystatechange = function(){
        var result;
        if (xhr.readyState == 3){
            //get only the new data and adjust counter
            result = xhr.responseText.substring(received);
            received += result.length;

            //call the progress callback
            progress(result);
        } else if (xhr.readyState == 4){
            finished(xhr.responseText);
        }
    };
    xhr.send(null);
    return xhr;
}

var client = createStreamingClient("streaming.php", function(data){
    alert("Received: " + data);
}, function(data){
    alert("Done!");
});
```

#### 服务器发送事件

SSE（Sever-Sent Events，服务器发送事件） 是围绕只读 Comet 交互推出的 API 或者模式。SSE  API 用于创建到服务器的单向连接，服务器通过这个连接可以发送任意数量的数据。服务器响应的 MIME 类型必须是 text/event-stream，而且是浏览器中的 JavaScript API 能解析格式输出。SSE 支持轮询和 HTTP 流，而且能在断开连接时自动确定何时重新连接。

**SSE API**

要预订新的事件流，首先要创建一个新的 EventSource 对象，并传进一个入口点：

``` javascript
var source = new EventSource("myevents.php");
```

传入的 URL 必须与创建对象的页面同源。EventSource 的实例有一个`readyState`属性，值为 0 表示正在连接到服务器，值为 1 表示打开了连接，值为 2 表示关闭了连接。另外，还有以下三个事件：

- open：在建立连接时触发。
- message：在从服务器接收到新事件时触发。
- error：在无法建立连接时触发。

服务器发回的数据以字符串形式保存在`event.data`中，默认情况下 EventSource 对象会保持与服务器的活动连接。如果连接断开还会重新连接。想强制立即断开连接并不再重新连接，可以调用`close()`方法。

**事件流**

所谓的服务器事件会通过一个持久的 HTTP 响应发送，这个响应的 MIME 类型为 text/event-stream。响应的格式是纯文本，最简单的情况是每个数据项都带有前缀的`data:`，例如：

```
data: foo

data: bar

data: foo
data: bar

```

每个值之间以一个换行符分隔，只有在包含`data:`的数据行后面有空行时，才会触发 message 事件。

通过`id:`前缀可以给特定事件指定一个关联的 ID ，这个 ID 行位于`data:`行前面或后面均可。

``` 
data: foo
id: 1
```

设置了 ID 以后，EventSource 对象会跟踪上一次触发的事件。如果断开连接，会向服务器发送一个包含名为 Last-Event-ID 的特殊 HTTP 头部的请求，以便服务器知道下一次该触发哪个事件。在多次连接的事件流中，这种机制可以确保浏览器以正确的顺序收到连接的数据段。

#### Web Sockets

Web Sockets 的目标是在一个单独的持久连接上提供全双工、双向通信。 在 JavaScript 创建了 Web Socket 之后，会有一个 HTTP 请求发送到浏览器以发起连接。在取得服务器响应后，建立的连接会从 HTTP 协议变为 Web Socket 协议。

由于 Web Sockets 使用了自定义协议，所以 URL 模式也略有不同。未加密的连接不再是 http://，而是 ws://；加密的连接也不是 https://，而是 wss://。

使用自定义协议而非 HTTP 协议的好处是，能够在客户端和服务器之间发送非常少量的数据，而不必担心 HTTP 那样字节级的开销。由于传递的数据包很小，因此 Web Sockets 非常适合移动应用。

**Web Sockets API**

要创建 Web Socket，先实例一个 WebSocket 对象并传入要连接的 URL：

``` javascript
var socket = new WebSocket("ws://www.example.com/server.php");
```

必须给 WebSocket 构造函数传入绝对 URL。同源策略对 Web Sockets 不适用，因此可以通过它打开到任何站点的连接。

WebSocket 也有一个表示当前状态的`readystate`属性。不过这个属性的值与 XHR 并不相同：

- WebSocket.OPENING(0)：正在建立连接
- WebSocket.OPEN(1)：已经建立连接
- WebSocket.CLOSING(2)：正在关闭连接
- WebSocket.CLOSE(3)：已经关闭连接

要关闭 Web Socket 连接，可以调用`close()`方法。

**发送和接收数据**

- `send()`：可以向服务器发送数据。接受一个参数，任意字符串。复杂数据可以先用`JSON.stringify()`序列化字符串再发送。
- 当服务器向客户端发来消息时，WebSocket 对象会触发 messge 事件。这个事件与其他传递消息的协议类似，也是把返回的数据保存在`event.data`属性中。

```javascript
// 必须给WebSocket构造函数传入绝对URL
var socket = new WebSocket("ws://www.example.com/server.php");
// 向服务器发送数据（只能发送纯文本，其他数据需要序列化）
socket.send("Hello");
// 接收服务器的响应数据
socket.onmessage = function(event) {
    var data = event.data;
};
```

**其他事件**

WebSocket 对象还有其他三个事件，在连接生命周期的不同阶段触发。

- open：在成功建立连接时触发。
- error：在发生错误时触发，连接不能持续。
- close：在连接关闭时触发。

WebSocket 对象不支持 DOM 2 级事件侦听器。如下：

``` javascript
var socket = new WebSocket("ws://www.example.com/server.php");

socket.onopen = function (){
    alert("Connection established");
}

socket.onerror = function (){
    alert("Connection error");
}

socket.onclose = function (){
    alert("Connection closed");
}
```

在这三个事件中，只有 close 事件的`event`对象有额外的信息。这个事件的事件对象有 3 个额外的属性：

- wasClean：是一个布尔值，表示连接是否已经明确地关闭；
- code：服务器返回的数值状态码；
- reason：是一个字符串，包含服务器发回的消息。









