---
title: 重读 JS 高程（Chapter23）
tags: javascript
draft: true
category: javascript
date: 2017-09-24 13:18:42
---


## Chapter23. 离线应用与客户端存储

支持离线 Web 应用开发是 HTML5 的另一个重点。所谓离线 Web 应用，就是在设备不能上网的情况下仍然可以运行的应用。

### 离线检测

开发离线应用首先要检测设备的状态，根据设备在线或者离线进行接下来的操作。HTML5 为我们提供了两种判断设备在线状态的方式：

- `navigator.onLine`属性：属性值为`true`表示设备能上网，值为`false`表示设备离线。
- `online`/`offline`事件：当网络从离线变为在线或者从在线变为离线时，分别触发这两个事件。这两个事件在`window`对象上触发。

为了检测应用是否离线，我们可以通过navigator.onLine判断设备的初始状态，同时通过online/offline事件来确定网络的连接状态是否变化。online/offline事件首先在body上触发，随后会沿着document.body, document, window的顺序冒泡，所以我们可以对它们进行监听获得设备的状态变化。

```javascript
if(navigator.onLine){
    //正常工作
} else {
    //执行离线事件处理程序
}

window.addEventListener('online', function(){
  //进行离线缓存
});
window.addEventListener('offline', function(){
  //使用离线缓存
})
```

<!-- more -->

### 应用缓存

HTML5 的应用缓存（application cache），或者简称为 appcache，是专门为开发离线 Web 应用而设计的。Appcache 就是从浏览器的缓存中分出来的一块缓存区。要想在这个缓存中保存数据，可以使用一个描述文件（manifest file），列出要下载和缓存的资源。下面是一个简单的描述文件示例。

```
CACHE MANIFEST
#Comment

index.html
common.css

NETWORK:
*.html

CACHE:
login.html

FALLBACK:
*.html offline.html
/project /offline/project
```

描述文件的基本格式如下：

- CACHE MANIFEST书写在第一行，而且必不可少；
- \# 开头代表注释；
- CACHE 标识符：这是条目的默认部分。系统会在首次下载此标头下列出的文件（或紧跟在`CACHE MANIFEST` 后的文件）后显式缓存这些文件；
- NETWORK 标识符：代表“白名单”，它表示的是无论在线离线都需要联网访问的文件；
- FALLBACK 标识符：此部分是可选的，用于指定无法访问资源时的后备网页。其中第一个 URI 代表资源，第二个代表后备网页。两个 URI 必须相关，并且必须与清单文件同源。可使用通配符。

在声明好描述文件的后，需要将描述文件与页面关联起来。在文档的 `html` 标记中添加 manifest 属性：

```javascript
<html manifest='/offline.manifest'>
```

浏览器除了在第一次访问 Web 应用时缓存资源外，只有在以下三种情况才会对缓存进行更新：

1. 浏览器缓存被删除
2. `manifest`描述文件本身发生变化
3. 应用缓存通过编程方式进行更新

而 cache manifest 中的资源文件发生变化并不会触发更新，所以在对资源进行更改后，需要向描述文件中加入时间戳或版本号。

要以编程方式对缓存进行更新，需要用到缓存对象`window.applicationCache`。可以使用它的属性`window.applicationCache`来判断缓存状态，全部状态如下：

- 0：无缓存，即没有与页面相关的应用缓存。
- 1：闲置，应用缓存未更新。
- 2：检查中，正在下载描述文件并检查更新。
- 3：下载中，即应用换攒正在描述文件中的指定资源。
- 4：更新完成，即应用缓存已经更新了资源，而所有资源都已经下载完毕，可以应用`swapCache()`应用更新的缓存。
- 5：废弃，即描述文件不存在，不能获取应用缓存

与之相同的还有缓存事件：

- checking：在浏览器在为应用缓存查找更新时触发
- error：查找错误时触发
- noupdate：检查后发现描述文件无变化时触发
- downloading：开始下载应用缓存资源时触发
- progress：在下载应用缓存资源时不断的触发
- updateready：应用缓存已经更新完毕，此时可以手动触发`swapCache()`应用更新的缓存
- cached：在应用缓存完整可用时触发

一般来讲，这些事件会随着页面加载按上述顺序依次触发。不过，通过调用`update()`方法也可以手工干预，让应用缓存为检查更新而触发上述事件。在调用`update()`后，若触发`catched`事件，说明应用缓存已经准备就绪，不会发生其他操作；若触发`updateready`事件，就调用`swapCache()`实现对浏览器缓存的更新。

### 数据存储

#### Cookie

HTTP Cookie，通常叫做 cookie，最初是在客户端用于存储会话信息的。该标准要求服务器对任意 HTTP 请求发送 Set-Cookie HTTP 头作为响应的一部分，其中包含会话信息。这种服务器响应的头可能如下：

``` http
HTTP/1.1 200 OK
Content-type: text/html
Set-Cookie: name=value
Other-header: other-header-value
```

这个 HTTP 响应设置以`name`为名称、以`value`为值的一个 cookie，名称和值在传送时都必须是 URL 编码的。浏览器会存储这样的会话信息，并在之后通过未每个请求头添加 Cookie HTTP 头将信息发送回服务器。如下所示：

```http
GET /index.html HTTP/1.1
Cookie: name=value
Other-header: other-header-value
```

发送回服务器的额外信息可以用于唯一验证客服来自于发送的哪个请求。

**限制**

cookie 性质上是绑定在特定的域名下的。当设定了一个 cookie 后，再给创建它的域名发送请求时，都会包含这个 cookie。这个限制确保了储存在 cookie 中的信息只能让批准的接受者访问，而无法被其他域访问。

每个域的 cookie 总数是有限的，cookie 长度也有限制，一般为 4KB。

**cookie 的构成**

cookie 由浏览器保存的一下几块信息构成：

- 名称：一个唯一确定 cookie 的名称。必须经过 URL 编码，不区分大小写。
- 值：储存在 cookie 中的字符串，值必须被 URL 编码。
- 域：cookie 对于哪个域是有效的。所有向该域发送的请求中都会包含这个 cookie 信息。
- 路径：指定域中的路径才能向服务器发送 cookie。
- 失效时间：表示该 cookie 何时被删除的时间戳。默认情况下，浏览器会在回话关闭时将所有 cookie 删除，不过也可以自定义删除时间。
- 安全标志：指定后，cookie 只有在使用 SSL 连接的时候才发送到服务器。

```http
HTTP/1.1 200 OK
Content-type: text/html
Set-Cookie: name=value; expires=Mon, 22-Jan-0707:10:24 GMT; domain=.wrox.com; path=/; secure
Other-header: other-header-value
```

注意，域、路径、失效时间和 secure 标志都是服务器给浏览器的提示，以指定何时应该发送 cookie。这些参数并不会作为发送到服务器的 cookie 信息的一部分，只有名值对才会被发送。

**JavaScript 中的 cookie**

我们可以使用 BOM 提供的接口`document.cookie`来设置 Cookie，上述参数中只有 name/value 是必选项，只要`name`没有重复，cookie 赋值是不会覆盖已有信息的。如下：

```javascript
document.cookie = encodeURIComponent('name') + '=' + encodeURIComponent('Nicholas');
```

下面封装实现了对 Cookie 值的增删和查询，可以把这种常用操作加入到自己的库里：

```javascript
var CookieUtil = {
    get: function (name){
        var cookieName = encodeURIComponent(name) + "=",
            cookieStart = document.cookie.indexOf(cookieName),
            cookieValue = null,
            cookieEnd;
      
        if (cookieStart > -1){
            cookieEnd = document.cookie.indexOf(";", cookieStart);
            if (cookieEnd == -1){
                cookieEnd = document.cookie.length;
            }
            cookieValue = decodeURIComponent(document.cookie.substring(cookieStart + cookieName.length, cookieEnd));
        } 

        return cookieValue;
    },
    set: function (name, value, expires, path, domain, secure) {
        var cookieText = encodeURIComponent(name) + "=" + encodeURIComponent(value);
    
        if (expires instanceof Date) {
            cookieText += "; expires=" + expires.toGMTString();
        }
    
        if (path) {
            cookieText += "; path=" + path;
        }
    
        if (domain) {
            cookieText += "; domain=" + domain;
        }
    
        if (secure) {
            cookieText += "; secure";
        }
    
        document.cookie = cookieText;
    },
    unset: function (name, path, domain, secure){
        this.set(name, "", new Date(0), path, domain, secure);
    }
};
```

**子 cookie**

为了绕开浏览器单域名下的 cookie 数限制，可以使用子 cookie 的概念。格式如下：

``` javascript
name=name1=value1&name2=value2&name3=value3&name4=value4&name5=value5
```

子 cookie 一般也以查询字符串的格式进行格式化。下面是封装好的 SubCookieUtil 库：

``` javascript
var SubCookieUtil = {  
    get: function (name, subName){
        var subCookies = this.getAll(name);
        if (subCookies){
            return subCookies[subName];
        } else {
            return null;
        }
    },
    getAll: function(name){
        var cookieName = encodeURIComponent(name) + "=",
            cookieStart = document.cookie.indexOf(cookieName),
            cookieValue = null,
            cookieEnd,
            subCookies,
            i,len,
            parts,
            result = {};
            
        if (cookieStart > -1){
            cookieEnd = document.cookie.indexOf(";", cookieStart)
            if (cookieEnd == -1){
                cookieEnd = document.cookie.length;
            }
            cookieValue = document.cookie.substring(cookieStart + cookieName.length, cookieEnd);
            
            if (cookieValue.length > 0){
                subCookies = cookieValue.split("&");
                
                for (i=0, len=subCookies.length; i < len; i++){
                    parts = subCookies[i].split("=");
                    result[decodeURIComponent(parts[0])] = decodeURIComponent(parts[1]);
                }
    
                return result;
            }  
        } 

        return null;
    },   
    set: function (name, subName, value, expires, path, domain, secure) {    
        var subcookies = this.getAll(name) || {};
        subcookies[subName] = value;
        this.setAll(name, subcookies, expires, path, domain, secure);
    },    
    setAll: function(name, subcookies, expires, path, domain, secure){
        var cookieText = encodeURIComponent(name) + "=",
            subcookieParts = new Array(),
            subName;
        
        for (subName in subcookies){
            if (subName.length > 0 && subcookies.hasOwnProperty(subName)){
                subcookieParts.push(encodeURIComponent(subName) + "=" + encodeURIComponent(subcookies[subName]));
            }
        }
        
        if (subcookieParts.length > 0){
            cookieText += subcookieParts.join("&");
                
            if (expires instanceof Date) {
                cookieText += "; expires=" + expires.toGMTString();
            }
        
            if (path) {
                cookieText += "; path=" + path;
            }
        
            if (domain) {
                cookieText += "; domain=" + domain;
            }
        
            if (secure) {
                cookieText += "; secure";
            }
        } else {
            cookieText += "; expires=" + (new Date(0)).toGMTString();
        }
    
        document.cookie = cookieText;        
    
    },    
    unset: function (name, subName, path, domain, secure){
        var subcookies = this.getAll(name);
        if (subcookies){
            delete subcookies[subName];
            this.setAll(name, subcookies, null, path, domain, secure);
        }
    },    
    unsetAll: function(name, path, domain, secure){
        this.setAll(name, null, new Date(0), path, domain, secure);
    }
};
```

**关于 cookie 的思考**

还有一类 cookie 被称为“HTTPOnly cookie”,它只能从服务器端读取，不能通过 JavaScript 获取。

由于所有的 cookie 都会由浏览器作为请求头发送，所以在 cookie 中存储大量信息会影响到特定域的请求性能。 cookie 信息越大，完成对服务器请求的时间也就越长。尽管浏览器对 cookie 进行了大小限制，不过最好还是尽可能在 cookie 中少存储信息，以避免影响性能。

#### IE 用户数据

IE 中可以采用如下方式使用持久化数据：

``` html
<div style="behavior:url(#default#userData)" id="dataStore"></div>
```

一旦该元素使用了 userData 行为，就可以使用`setAttribute()`和`removeAttribute()`增删数据。用`save()`方法指定数据空间。下一次页面载入时，可以使用`load()`指定同样的数据空间名称来加载数据。

 #### Web 存储机制

主要为了提供在 cookie 之外存储会话数据的途径和存储

**Storage 类型**

Storage 类型提供最大的存储空间来存储名值对。Storage 实例有如下方法：

- `clear()`：删除所有值。
- `getItem(name)`：根据指定的名字`name`获取对应的值。
- `key(index)`：获得`index`位置处的值的名字。
- `removeItem(name)`：删除由`name`指定的名值对。
- `setItem(name, value)`：为指定的`name`设置一个对应的值。

**sessionStorage 对象**

该对象存储特定于某个会话的数据，也就是该数据只保持到浏览器关闭。这个对象就像会话 cookie，也在浏览器关闭后消失。存储在 sessionStorage 中的数据可以跨越页面刷新而存在，同时如果浏览器支持，浏览器崩溃后依然可用。

**globalStorage 对象**

Firefox2 实现的对象，已经被`localStorage`替代了。

**localStorage 对象**

该对象可以跨越会话存储数据，但是不能跨域。数据保留到通过 JavaScript 删除或者是用户清除浏览器缓存。

**storage 事件**

对 Storage 对象进行任何修改，都会触发该事件，其`event`对象有如下属性：

- `domain`：发生变化的存储空间的域名
- `key`：设置或者删除的键名。
- `newValue`：如果是设置值，则是新值；如果是删除键，则是`null`。
- `oldValue`：键被更改之前的值。

#### IndexedDB

IndexedDB 的思想是创建一套 API 方便保存和读取 JavaScript 对象，同时还支持查询及搜索。操作都是异步的，需要注册 onerror 或 onsuccess 事件处理程序，确保可以适当处理结果。使用方式如下：

```javascript
var indexedDB = window.indexedDB || window.msIndexedDB || window.mozIndexedDB || window.webkitIndexedDB;
```

**数据库**

IndexedDB 最大的特色是使用对象保存数据，而不是用表来保存数据。

把要打开的数据库名传给`indexDB.open()`，有该数据库就打开，没有就先创建再打开。该方法会返回一个 IDBRequest 对象，可以用`onerror`和`onsuccess`获取信息。如果成功，`event.target.result`中将有一个数据库实例对象；如果出错，`event.target.errorCode`将保存一个错误码。

可以调用`setVersion()`方法设置数据库版本号。

**对象存储空间**

使用如下代码可以创建对象存储空间：

``` javascript
var users = [
      {
          username: "007",
          firstName: "James",
          lastName: "Bond",
          password: "foo"
      },
      {
          username: "ace",
          firstName: "John",
          lastName: "Smith",
          password: "bar"
      }                
  ];
database.createObjectStore("users", { keyPath: "username" }); //keyPath的值用作存储空间的键使用
```

全部代码：

``` javascript
var indexedDB = window.indexedDB || window.msIndexedDB || window.mozIndexedDB || window.webkitIndexedDB,
    request,
    store,
    database,
    users = [
        {
            username: "007",
            firstName: "James",
            lastName: "Bond",
            password: "foo"
        },
        {
            username: "ace",
            firstName: "John",
            lastName: "Smith",
            password: "bar"
        }                
    ];

request = indexedDB.open("example");
request.onerror = function(event){
    alert("Something bad happened while trying to open: " + event.target.errorCode);
};
request.onsuccess = function(event){
    database = event.target.result;
    initializeDatabase();
};    

function initializeDatabase(){
    if (database.version != "1.0"){
        request = database.setVersion("1.0");
        request.onerror = function(event){
            alert("Something bad happened while trying to set version: " + event.target.errorCode);
        };
        request.onsuccess = function(event){
            store = database.createObjectStore("users", { keyPath: "username" });
            var i = 0,
                len = users.length;

            while(i < len){
                store.add(users[i++]);
            }

            alert("Database initialized for first time. Database name: " + database.name + ", Version: " + database.version);
        };
    } else {
        alert("Database already initialized. Database name: " + database.name + ", Version: " + database.version);
        request = database.transaction("users").objectStore("users").get("007");
        request.onsuccess = function(event){
            alert(event.target.result.firstName);
        };
    }
}
```

**事务**

在数据库上，调用`transaction()`方法可以创建事务。只要想读取或修改数据，都要通过事务来组织所有操作。

``` javascript
var transaction = db.transaction("users");
```

这样只加载 users 存储空间的数据，以便通过事务进行访问。这些事务默认都是以只读方式访问数据，要修改访问方式，必须在创建事务时传入第二个参数，这个参数表示访问模式，用 IDBTransaction 接口定义的如下常量表示：

- READ_ONLY(0)：表示只读
- READ_WRITE(1)： 表示读写
- VERSION_CHANGE(2)： 表示改变

``` javascript
var transaction = db.transaction("users", IDBTransaction.READ_WRITE);
```

取得了事务的索引以后，就可以使用`objectStore()`并传入存储空间名称，就可以访问特定存储空间。然后就可以使用`add()`、`put()`、`get()`和`delete()`方法，还可以用`clear()`清空对象。

一个事务可以完成任何多个请求，所以事务本身也有处理程序，`onerror`和`oncomplete`。

**游标查询**

在需要检测多个对象的情况下需要在事务内部创建游标。游标是指向结果集的指针，与传统数据库不同，游标不提前收集结果，会先指向结果中的第一项，接到查询下一项的指令时，才指向下一项。

在对象存储空间上调用`openCursor()`方法可以创建游标，返回请求对象，因此必须为该对象指定`onsuccess`和`onerror`事件处理程序。

在`onsuccess`中，可以通过`event.target.result`，取得存储空间中的下一个对象。在结果集中有下一项时，这个属性中保存一个 IDBCursor 实例，没有下一项时，这个属性值为`null`。`IDBCurosor`的实例有以下几个属性：

- `direction`：数值，表示游标移动的方向。默认值为 IDBCursor.NEXT(0)，表示下一项。还有 IDBCursor.NEXT_NO_DUPLICATE(1)表示下一个不重复的项，IDBCursor.PREV(2) 表示前一项，而 IDBCursor.PREV_NO_DUPLICATE 表示前一个不重复的项。
- `key`：对象的键。
- `value`：实际的对象。
- `primaryKey`：游标使用的键。

调用`update()`方法可以用指定的对象更新当前游标的`value`。调用`delete()`方法，就会删除相应记录。

默认情况下，每个游标只发起一次请求，要想发起另一次请求，必须调用下面的一个方法：

- `continue(key)`：移动到结果集中的下一项。
- `advance(count)`：向前移动`count`指定的项数。


**键范围**

由 IDBKeyRange 的实例表示。有四种定义键范围的方式：

- `only()`：传入想要取得的对象的键，`IDBKeyRange.only("007")`。
- `lowerBound()`：指定结果集的下界，表示游标开始的位置，`IDBKeyRange.lowerBound("007")`。不想包含"007"，可以传入第二个参数`true`。
- `upperBound()`：指定结果集上界，也就是游标值不能超越哪个键，`IDBKeyRange.upperBound("007")`。不想包含"007"，可以传入第二个参数`true`。
- `bound()`：同时指定上下界。接受 4 个参数：表示下界的键、表示上界的键、可选的表示是否跳过下界的布尔值和可选的表示是否跳过上界的布尔值。

定义范围后，把它传给`openCursor()`方法，就能得到一个符合相应约束条件的游标。

**设定游标方向**

`openCursor()`可接受两个参数。第一个参数就是刚刚看到的 IDBKeyRange 的实例，第二是表示方向的数值常量。可传入IDBCursor.NEXT，表示下一项。还有 IDBCursor.NEXT_NO_DUPLICATE表示下一个不重复的项，IDBCursor.PREV 表示前一项，而 IDBCursor.PREV_NO_DUPLICATE 表示前一个不重复的项。

**索引**

用于为一个对象的存储空间指定多个键。创建索引先要引用对象存储空间，然后调用`createIndex()`方法。如下：

``` javascript
var store = db.transaction("users").objectStore("users"),
    index = store.createIndex("username", "username", {unique: false});
```

`createIndex()`的第一个参数是索引名字，第二个参数是索引的属性的名字，第三个参数是一个包含`unique`属性的选项对象。返回值是 IDBIndex 的实例，可以通过`index()`方法取得已存在的实例。

索引和对象存储空间很相似，调用`openCursor()`方法也可以创建新的游标，不过会将索引键保存在`event.result.key`中。

另外，可以创建一个特殊的只返回每条记录主键的游标，即`openKeyCursor()`方法，这种情况下，`event.result.key`仍然保存着索引键，而`event.result.value`中保存的是主键，而不是整个对象。

通过索引键取得主键可以使用`getKey()`方法，也会创建一个新的请求。

任何时候通过 IDBIndex 对象的下列属性都可以取得有关索引的相关信息。

- `name`：索引的名字
- `keyPath`：传入`createIndex()`中的属性路径
- `objectStore`：索引的对象存储空间。
- `unique`：表示索引键是否唯一的布尔值。

删除索引可以使用`deleteIndex()`方法，传入索引名字即可。

**并发问题**

只有浏览器仅有一个标签页使用数据库的情况下，调用`setVersion()`才能完成操作。这样可以避免两个标签页同时操作一个数据库引发的并发问题。方式是使用`onversionchange`事件处理程序，在每次更新时，关闭数据库，保证更新完成。

``` javascript
var request, database;

request = indexedDB.open("admin");
request.onsuccess = function (event){
    database = event.target.result;
    database.onversionchange = function (){
        database.close();
    }
}
```

指定`onblocked`事件处理程序也很重要，在你想要更新数据库的版本但另一个标签页已经打开数据库的情况下，就会触发这个事件处理程序。此时，最好先通知用户关闭其他标签页，然后再重新调用`setVersion()`。

``` javascript
var request = database.setVersion("2.0");
request.onblocked = function (){
    alert("Please close all other tabs and try again");
}
request.onsuccess = function (){
    
}
```

**限制**

只能由同源页面操作，不能跨域共享信息；每个来源的数据库占用的磁盘空间有限制。







