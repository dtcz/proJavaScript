---
title: 重读 JS 高程（Chapter22）
tags: javascript
category: javascript
date: 2017-09-24 08:43:23
---


## Chapter22. 高级技巧

### 高级函数

#### 安全的类型检测

使用`Object.prorotpye.toString.call(value)`，会返回一个`[object NativeConstructorName]`格式的字符串。每个类在内部都有一个`[[Class]]`属性，这个属性中就指定了上述字符串中的构造函数名。例如：

``` javascript
Object.prorotpye.toString.call([]); // "[object Array]"
```

#### 作用域安全的构造函数

作用域安全的构造函数在进行任何更改前，首先确认`this`对象是正确类型的实例。如果不是，那么会创建新的实例并返回。如下：

``` javascript
function Person(name, age, job){
    if (this instanceof Person){
        this.name = name;
        this.age = age;
        this.job = job;
    } else {
        return new Person(name, age, job);
    }
}

var person1 = Person("Nicholas", 29, "Software Engineer");
alert(window.name);   //""
alert(person1.name);  //"Nicholas"

var person2 = new Person("Shelby", 34, "Ergonomist");
alert(person2.name);  //"Shelby"
```

<!-- more -->

#### 惰性载入函数

惰性载入函数表示函数执行的分支仅会发生一次。有两种实现惰性载入的方式：

第一种就是在函数被调用时再处理函数。在第一次调用的过程中，该函数会被覆盖为另一个按合适方式执行的函数，这样任何对原函数的调用都不用再经过执行分支了。如下：

``` javascript
function createXHR(){
    if (typeof XMLHttpRequest != "undefined"){
        createXHR = function(){
            return new XMLHttpRequest();
        };
    } else if (typeof ActiveXObject != "undefined"){
        createXHR = function(){                    
            if (typeof arguments.callee.activeXString != "string"){
                var versions = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0",
                                "MSXML2.XMLHttp"];

                for (var i = 0; i < 3; i++){
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
        };
    } else {
        createXHR = function(){
            throw new Error("No XHR object available.");
        };
    }

    return createXHR();
}
```

第二种实现惰性载入的方式是在声明函数时就指定适当的函数。这样在第一次调用函数时就不会损失性能了，而在代码首次加载时会损失一点性能。

``` javascript
var createXHR = (function(){
    if (typeof XMLHttpRequest != "undefined"){
        return function(){
            return new XMLHttpRequest();
        };
    } else if (typeof ActiveXObject != "undefined"){
        return function(){                    
            if (typeof arguments.callee.activeXString != "string"){
                var versions = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0",
                                "MSXML2.XMLHttp"];

                for (var i = 0; i < 3; i++){
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
        };
    } else {
        return function(){
            throw new Error("No XHR object available.");
        };
    }
})();
```

惰性载入函数的优点是只在执行分支代码时牺牲一点性能。这两种方式都能避免执行不必要的代码。

#### 函数绑定

函数绑定要创建一个函数，可以在特定的`this`环境中以指定参数调用另一个函数。解决方案是创建`bind()`函数。如下：

``` javascript
function bind(fn, context){
    return function (){
        return fn.apply(context, arguments);
    }
}
```

该函数创建了一个闭包，闭包使用`apply()`调用传入的函数，并给`apply()`传入`context`和参数。如下：

``` javascript
var handler = {
    message: "Event handled",

    handleClick: function(event){
        alert(this.message + ":" + event.type);
    }
};

var btn = document.getElementById("my-btn");
EventUtil.addHandler(btn, "click", bind(handler.handleClick, handler));
```

ES5 为所有函数定义了一个原生的`bind()`方法，进一步简化了操作。如下：

``` javascript
EventUtil.addHandler(btn, "click", handler.handleClick.bind(handler));
```

#### 函数柯里化

与函数绑定密切相关的主题是函数柯里化（function currying）,它用于创建已经设置好了一个或多个参数的函数。函数柯里化的基本方法和函数绑定是一样的：使用一个闭包返回一个函数。两者的区别在于，当函数被调用时，返回的函数还需要设置一些传入的参数。如下：

``` javascript
function add(num1, num2){
    return num1 + num2;
}

function curriedAdd(num2){
    return add(5, num2);
}

add(2, 3);      //5
curriedAdd(3);  //8
```

柯里化函数通常由一下步骤动态创建：调用另一个函数并它传入要柯里化的函数和必要参数。下面是创建柯里化函数的通用方式：

``` javascript
function curry(fn){
    var args = Array.prototype.slice.call(arguments, 1);
    return function (){
        var innerArgs = Array.prototype.slice.call(arguments);
        var finalArgs = args.concat(innerArgs);
        return fn.apply(null, finalArgs);
    };
}
```

`curry()`函数的主要工作就是将被返回函数的参数进行排序。第一个参数是要进行柯里化的函数，其他参数是要传入的值。可以按以下方式应用：

``` javascript
function add(num1, num2){
    return num1 + num2;
}

var curriedAdd = curry(add, 5, 12);
curriedAdd();  //17
```

函数柯里化还常常作为函数绑定的一部分包含在其中，构造出更为复杂的`bind()`函数。例如：

``` javascript
function bind(fn, context){
    var args = Array.prototype.slice.call(arguments, 2);
    return function () {
       var innerArgs = Array.prototype.slice.call(arguments);
       var finalArgs = args.concat(innerArgs);
      
       return fn.apply(context, finalArgs);
    }
}
```

ES5 的`bind()`函数也实现了函数柯里化。

``` javascript
var handler = {
    message: "Event handled",

    handleClick: function(name, event){
        alert(this.message + ":" + name + ":" + event.type);
    }
};

var btn = document.getElementById("my-btn");
EventUtil.addHandler(btn, "click", bind(handler.handleClick, handler, "my-btn"));
EventUtil.addHandler(btn, "click", handler.handleClick.bind(handler, "my-btn"));
```

### 防篡改对象

JavaScript 中任何对象都可以被在同一环境中运行的代码修改，开发人员可能意外地修改别人的代码，甚至可能用不兼容的功能重写原生对象。ES5 可以让开发人员定义防篡改对象（tamper-proof object），一旦把对象定义为防篡改，就无法撤销了。

#### 不可扩展对象

使用`Object.preventExtensions()`方法可以禁止给对象添加属性和方法。

``` javascript
var person = {name: "Nicholas"};
Object.preventExtensions(person);

person.age = 29;
alert(person.age); // undefined
```

虽然不能给对象添加新成员，但已有的成员则丝毫不受影响，依然可以修改和删除已有成员。使用`Object.isExtensible()`方法还可以确定对象是否可以扩展。

``` javascript
var person = {name: "Nicholas"};
alert(Object.isExtensible(person)); // true

Object.preventExtensions(person);
alert(Object.isExtensible(person)); // false
```

#### 密封的对象

密封对象不可扩展，而且已有成员的`[[Configurable]]`特性被设置为`false`。这就意味着不能删除属性和方法，因为不能使用`Object.defineProperty()`把数据属性修改为访问器属性。属性值是可以修改的。

``` javascript
var person = {name: "Nicholas"};
Object.seal(person);

person.age = 29;
alert(person.age); // undefined

delete person.name;
alert(person.name); // "Nicholas"
```

使用`Object.isSealed()`方法可以确定对线是否被密封了。

``` javascript
var person = {name: "Nicholas"};
alert(Object.isSealed(person)); // false

Object.seal(person);
alert(Object.isSealed(person)); // true
```

#### 冻结的对象

最严格的防篡改级别是冻结对象。冻结的对象既不可扩展，又是密封的，而且对象的数据属性的`[[Writable]]`特性会被设置为`false`。如果定义`[[Set]]`函数，访问器属性仍然是可写的。使用`Object.freeze()`可以冻结对象。

``` javascript
var person = { name: "Nicholas" };
Object.freeze(person);

person.age = 29;
alert(person.age);      //undefined

delete person.name;
alert(person.name);     //"Nicholas"

person.name = "Greg";
alert(person.name);     //"Nicholas"
```

`Object.isFrozen()`方法用于检测冻结对象。

``` javascript
var person = { name: "Nicholas" };
alert(Object.isExtensible(person));   //true
alert(Object.isSealed(person));       //false
alert(Object.isFrozen(person));       //false

Object.freeze(person);
alert(Object.isExtensible(person));   //false
alert(Object.isSealed(person));       //true
alert(Object.isFrozen(person));       //true
```

### 高级定时器

JavaScript 运行于单线程的环境中，而定时器仅仅只是计划代码在未来的某个时间执行，执行时机不能保证。因为在页面的整个生存周期中，不同时间可能有其他代码控制着 JavaScript 进程。事实上，浏览器负责进行排序，指派某段代码在某个时间点运行的优先级。

除了主 JavaScript 执行进程外，还有一个需要在进程下一次空闲时执行的代码队列。随着页面在其生命周期中的推移，代码会按顺序加入队列。在 JavaScript 中没有任何代码是立刻执行的，但一旦进程空闲则尽快执行。

定时器对队列的工作方式是，当特定时间过去后将代码插入。注意，给队列添加代码并不意味着对它立即执行，而是能表示它会尽快执行。设定一个 150ms 后执行的定时器不代表到 150ms 代码就立刻执行，它表示代码会在 150ms 后被加入到队列中。如果，在这个时间点上，队列中没有其他东西，那么这段代码就会被执行。

#### 重复的定时器

`setInterval()`定时器确保定时器代码规则地插入队列中，这个方式的问题在于，定时器代码可能在代码再次被添加到队列之前还没有完成执行。JavaScript 引擎会在仅当没有该定时器的任何代码实例时，才将定时器代码添加到队列。这确保了定时器代码加入到队列中的最小时间间隔为指定间隔。 

不过这样会出现两个问题：

1. 某些间隔会被跳过；
2. 多个定时器的代码执行之间的间隔可能会比预期的小。

这种情况一般是定时器代码执行的时间过于长，超过了指定的时间间隔。

解决上述问题的办法就是链式调用`setTimeout()`。

```javascript
setTimeout(function(){
    // 处理中
    setTimeout(arguments.callee, interval)
}, interval);
```

这样在前一个定时器代码执行完之前，不会向队列插入新的定时器代码，确保不会有任何缺失的间隔。而且，可以保证在下一次定时器代码执行之前，至少等待指定的间隔，避免了连续的运行。

#### Yielding Processes

运行在浏览器中的 JavaScript 都被分配了一个确定数量的资源，脚本的长时间运行在 Web 中是不被允许的。 造成该脚本长时间运行的原因一般是过长、过深嵌套的函数调用或者是进行大量处理的循环。后者比较容易解决，长时间运行的循环通常遵循如下模式：

``` javascript
for(var i = 0, len = data.length; i < len; i++){
    process(data[i]);
}
```

此时需要考虑两个问题：

1. 该处理是否必须同步完成？
2. 数据是否必须按顺序完成？

如果都是“否”，则需要考虑数组分块（array chunking）技术。如下：

```javascript
function chunk(array, process, context) {
    setTimeout(function(){
        var item = array.shift();
        process.call(context, item);
        if(array.length > 0) {
            setTimeout(arguments.callee, 100);
        }
    }, 100);
}

var data = [12, 123, 1234, 453, 436, 23, 23, 5, 4123, 45, 346, 5634, 2234, 345, 342];
function printValue(item){
    var div = document.getElementById("myDiv");
    div.innerHTML += item + "<br>";        
}

chunk(data, printValue); // 函数在全局中，所以无需传入执行环境
console.log(data);       // []
```

数组分块的重要性在于它可以将多个项目的处理在执行队列上分开，在每个项目处理之后，给予其他的浏览器处理机会运行，这样就可以避免长时间运行脚本的错误。

#### 函数节流

在浏览器中，DOM 操作需要更多的内存和 CPU 时间。连续尝试进行过多的 DOM 相关操作可能会导致浏览器挂起，甚至崩溃，尤其当使用 onresize 事件处理程序时容易发生。可以用定时器给函数节流，函数节流的基本思想是某些代码不可以在没有间断的情况连续重复执行。 目的是只有在执行函数的请求停止了一段时间之后才能再次执行。

```javascript
function throttle(method, context) {
    clearTimeout(method.tId);
    method.tId = setTimeout(function (){
        method.call(context);
    }, 100);
}

function resizeDiv(){
    var div = document.getElementById("myDiv");
    div.style.height = div.offsetWidth + "px";
}

// 节流在resize事件中最常用
window.onresize = function(){
    throttle(resizeDiv);
};
```

### 自定义事件

事件是一种叫做观察者的设计模式，是一种创建松散耦合代码的技术。 对象可以发布事件，用来表示在该对象生命周期中某个有趣的时刻到了。然后其他对象可以观察该对象，等待这些有趣的时刻到来并通过运行代码来响应。

观察者模式由两类对象组成：主体和观察者。主体负责发布事件，同时观察者订阅这些事件来观察该主体。 关键点是主体并不知道观察者的任何事情，也就是说他们独自存在并正常运作，即使观察者不在。

```javascript
/* 管理事件的对象 */
function EventTarget(){
    this.handlers = {};    
}

EventTarget.prototype = {
    constructor: EventTarget,
    /**
     * 添加事件
     * @param type 事件类型
     * @param handler 事件处理程序
     */
    addHandler: function(type, handler){
        if (typeof this.handlers[type] == "undefined"){
            this.handlers[type] = [];
        }

        this.handlers[type].push(handler);
    },
    /**
     * 触发事件
     * @param event 事件对象
     */
    fire: function(event){
        if (!event.target){
            event.target = this;
        }
        if (this.handlers[event.type] instanceof Array){
            var handlers = this.handlers[event.type];
            for (var i = 0, len = handlers.length; i < len; i++){
                handlers[i](event);
            }
        }            
    },
    /**
     * 移除事件
     * @param type 事件类型
     * @param handler 事件处理程序
     */
    removeHandler: function(type, handler){
        if (this.handlers[type] instanceof Array){
            var handlers = this.handlers[type];
            for (var i = 0, len = handlers.length; i < len; i++){
                if (handlers[i] === handler){
                    break;
                }
            }
            handlers.splice(i, 1);
        }            
    }
};
```

```javascript
// 事件处理程序
function handleMessage(event){
    alert("Message received: " + event.message);
}

var target = new EventTarget();
// 添加监听
target.addHandler("message", handleMessage);
// 触发事件
target.fire({ type: "message", message: "Hello world!"});
// 移除监听
target.removeHandler("message", handleMessage);
// 触发事件（无效）
target.fire({ type: "message", message: "Hello world!"}); 
```

```javascript
function object(o){
    function F(){}
    F.prototype = o;
    return new F();
}
// 继承    
function inheritPrototype(subType, superType){
    var prototype = object(superType.prototype);   //create object
    prototype.constructor = subType;               //augment object
    subType.prototype = prototype;                 //assign object
}

function Person(name, age){
    EventTarget.call(this);
    this.name = name;
    this.age = age;
}

inheritPrototype(Person, EventTarget);

Person.prototype.say = function(message){
    this.fire({type: "message", message: message});
};
// 事件处理程序
function handleMessage(event){
    alert(event.target.name + " says: " + event.message);
}

var person = new Person("Nicholas", 29);
// 监听事件
person.addHandler("message", handleMessage);
// 发布事件
person.say("Hi there.");
```

### 拖放 

点击某个对象，并按住鼠标按钮不放，将鼠标移动到另一个区域，然后释放鼠标按钮将对象“放”在这里。 拖放的实现方式：创建一个绝对定位的元素，使其可以用鼠标移动。

```javascript
var DragDrop = function(){
    // EventTarget为上述实例中对象
    // 继承EventTarget，具有事件功能
    var dragdrop = new EventTarget(),
        dragging = null,
        diffX = 0,
        diffY = 0;

    function handleEvent(event){
        //get event and target
        var target = event.target;
        //determine the type of event
        switch(event.type){
            case "mousedown":
                if (target.className.indexOf("draggable") > -1){
                    dragging = target;
                    // 保存x、y坐标上的差值
                    diffX = event.clientX - target.offsetLeft;
                    diffY = event.clientY - target.offsetTop;
                    // 发布自定义事件
                    dragdrop.fire({type:"dragstart", target: dragging, x: event.clientX, y: event.clientY});
                }
                break;
            case "mousemove":
                if (dragging !== null){

                    //assign location
                    dragging.style.left = (event.clientX - diffX) + "px";
                    dragging.style.top = (event.clientY - diffY) + "px";   

                    // 发布自定义事件
                    dragdrop.fire({type:"drag", target: dragging, x: event.clientX, y: event.clientY});
                }
                break;
            case "mouseup":
                // 发布自定义事件
                dragdrop.fire({type:"dragend", target: dragging, x: event.clientX, y: event.clientY});
                dragging = null;
                break;
        }
    };

    //全局接口
    // 可以拖放
    dragdrop.enable = function(){
            document.addEventListener("mousedown", handleEvent);
            document.addEventListener("mousemove", handleEvent);
            document.addEventListener("mouseup", handleEvent);
    };
    // 禁止拖放
    dragdrop.disable = function(){
            document.removeEventListener("mousedown", handleEvent);
            document.removeEventListener("mousemove", handleEvent);
            document.removeEventListener("mouseup", handleEvent);
    };

    return dragdrop;
}();

DragDrop.enable();

// 监听（订阅）相关自定义事件                
DragDrop.addHandler("dragstart", function(event){
    var status = document.getElementById("status");
    status.innerHTML = "Started dragging " + event.target.id;
});

DragDrop.addHandler("drag", function(event){
    var status = document.getElementById("status");
    status.innerHTML += "<br>Dragged " + event.target.id + " to (" + event.x + "," + event.y + ")";
});

DragDrop.addHandler("dragend", function(event){
    var status = document.getElementById("status");
    status.innerHTML += "<br>Dropped " + event.target.id + " at (" + event.x + "," + event.y + ")";
});
```















