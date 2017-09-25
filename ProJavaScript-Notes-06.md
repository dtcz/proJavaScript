---
title: '重读 JS 高程（Chapter07）'
date: 2017-08-30 09:32:42
tags: javascript
category: javascript
---

## Chapter07. 函数表达式 

定义函数的方式有两种：一种是函数声明，另一种就是函数表达式。

``` javascript
//函数声明
function functionName(arg0, arg1, arg2) {
    //函数体
}
//函数表达式
var functionName = function(arg0, arg1, arg2) {
    //函数体
}
```

函数声明的一个重要特征就是**函数声明提升（function declaration hoisting)**，意思是在执行代码前会先读取函数声明。

### 递归

递归函数是在一个函数通过名字调用自身的情况下构成的。推荐做法是使用命名函数表达式，这种方式在严格模式和非严格模式下都行得通：

``` javascript
var factorial = (function f(num) {
    if(num <= 1) {
        return 1;
    } else {
        return num * f(num-1);
    }
});
```

以上代码创建了一个名为`f()`的命名函数表达式，然后将它赋值给`factorial`。即便把函数赋值给了另一个变量，函数的名字`f`仍然有效，所以递归调用照样能正确完成。

### 闭包

**闭包**是指**有权访问另一个函数作用域中的变量的函数**。创建闭包的常见方式，就是在一个函数内部创建另一个函数，例如：

``` javascript
function createComparisonFunction(propertyName) {
    return  function (object1, object2) {
        return object1[propertyName] - object2[propertyName];
    };
}
```

<!-- more -->

当某个函数被调用时，会创建一个执行环境（execution context）及相应的作用域链。然后，使用 arguments 和其他命名参数的值来初始化函数的活动对象（activation object）。但在作用域链中，外部函数的活动对象始终处于第二位，外部函数的外部函数的活动对象处于第三位，……直至作为作用域链终点的全局执行环境。 

在函数执行过程中，为读取和写入变量的值，就需要在作用域链中查找变量。如下例：

``` javascript
function compare(value1, value2) {
    return value1 - value2;
}
var result = compare(5, 10);
```

以上代码先定义了`compare()`函数，然后又在全局作用域中调用了它。当第一次调用`compare()`时，会创建一个包含`this`、`arguments`、`value1`和`value2`的活动对象。全局执行环境的变量对象（包含`this`、`result`和`compare`）在`compare()`执行环境的作用域链中则处于第二位。如下图：

![compare()作用域链](/images/compare.jpg)

后台的每个执行环境都有一个表示变量的对象——**变量对象**。全局环境的变量对象始终存在，而像`compare()`函数这样的局部环境的变量对象，则只在函数执行的过程中存在。在创建`compare()`函数时，会创建一个预先包含全局变量对象的作用域链，这个作用域链被保存在内部的`[[Scope]]`属性中。当调用`compare()`函数时，会为函数创建一个执行环境，然后通过复制函数的`[[Scope]]`属性中的对象构建起执行环境的作用域链。此后又有一个活动对象（在此作为变量对象使用）被创建并被推入执行环境作用域链的前端。对于这个例子中`compare()`函数的执行环境而言，其作用域链中包含两个变量对象：**本地活动对象**和**全局变量对象**。显然，作用域链本质上是一个指向变量对象的指针列表，它只引用但不实际包含变量对象。 

无论什么时候在函数中访问一个变量时，都会从作用域链中搜索具有相应名字的变量。**一般在函数执行完毕后，局部活动对象就会被销毁，内存中仅保存全局作用域（全局执行环境的变量对象），然而闭包的情况有所不同。** 

在另一个函数内部定义的函数将会包含函数（即外部函数）的活动对象添加到它的作用域链中。因此，在`createComparisonFunction()`函数内部定义的匿名函数的作用域链中，实际上将会包含外部函数`createComparisonFunction()`的活动对象。下图展示了当下列代码执行时，包含函数与内部匿名函数的作用域链。

``` javascript
var compare = createComparisonFunction("name");
var result = compare({ name: "Nicholas" }, { name: "Greg" });
```

在匿名函数从`createComparisFunction()`中返回后，它的作用域链被初始化为包含`createComparisonFunction()`函数的活动对象和全局变量对象。这样，匿名函数就可以访问在`createComparisonFunction()`中定义的所有变量。**更为重要的是，`createComparisonFunction()`函数在执行完毕后，其活动对象也不会被销毁，因为匿名函数的作用域链仍然在引用这个活动对象。**换句话说，当`createComparisonFunction()`函数返回后，其执行环境的作用域链会被销毁，但它的活动对象仍然会留在内存中；直到匿名函数被销毁后，`createComparisonFunction()`的活动对象才会被销毁，例如：

``` javascript
//创建函数
var compareNames = createComparisonFunction("name");
//调用函数
var result = compareNames({ name: "Nicholas" }, { name: "Greg" });
//解除对匿名函数的引用（以便释放内存）
compareNames = null;
```

首先，创建的比较函数被保存在变量`compareNames`中。而通过将`compareNames`设置为等于`null`解除该函数的引用，就等于通知垃圾回收例程将其清除。随着匿名函数的作用域链被销毁，其他作用域链（除了全局作用域）也都可以安全地销毁了。下图展示了调用`compareNames()`的过程中产生的作用域链之间的关系。 

![compareNames()作用域链](/images/compareName.jpg)

#### 闭包与变量

作用域链的这种配置机制引出了一个值得注意的副作用，即**闭包只能取得包含函数中任何变量的最后一个值**。闭包所保存的是整个变量对象，而不是某个特殊的值。如下：

``` javascript
function createFunctions() {
    var result = new Array();

    for(var i = 0; i<10; i++) {
        result[i] = function() {
            return i;
        };
    }

    return result;
}
```

这个函数会返回一个函数数组，而其中的每个函数都返回10。 我们可以通过创建另一个匿名函数强制让闭包的行为符合预期，如下：

``` javascript
function createFunctions() {
    var result = new Array();

    for(var i = 0; i < 10; i++) {
        //匿名函数直接赋值
        result[i] = function(num) {
            return function() {
                return num;
            };
        }(i);
    }

    return result;
}
```

#### 关于this对象

`this`对象是在运行时基于函数的执行环境绑定的：在全局函数中，`this`等于`window`，而当函数被作为某个对象的方法调用时，`this`等于那个对象。匿名函数的执行环境具有全局性，因此其`this`对象通常指向`window`。

``` javascript
var name = "The Window";
var object = {
    name: "My Object",
    getNameFunc: function() {
        return function() {
            return this.name;
        };
    }
};

alert(object.getNameFunc()());  //"The Window"（在非严格模式下）
```

`getNameFunc()`返回一个匿名函数，因此`object.getNameFunc()()`就会立即调用它返回的函数，结果就是返回一个字符串"The Window"。而如果访问`object`的属性，就需要把外部作用域中的`this`对象保存在一个闭包能够访问到的变量里。如下：

``` javascript
var name = "The Window";

var object = {
    name: "My Object",

    getNameFunc: function() {
        var that = this;
        return function() {
            return that.name;
        };
    }
};

alert(object.getNameFunc()());  //"My Object"
```

在定义匿名函数之前，我们把`this`对象赋值给了一个名叫`that`的变量。而在定义了闭包之后，闭包也可以访问这个变量，因为它是我们在包含函数中特意声明的一个变量。即使在函数返回之后，`that`也仍然引用着`object`，所以调用`object.getNameFunc()()`就返回了"My Object"。

每个函数在被调用时都会自动取得两个特殊变量：`this`和`arguments`。如果想访问作用域中的`arguments`对象，同样的，必须将该对象的引用保存到另一个闭包能够访问到的变量中。

几种特殊情况下，this 值可能会意外地改变。例如：

``` javascript
var name = "The Window";
var object = {
    name: "My Object",
    getName: function () {
        return this.name;
    }
};

object.getName();                      //"My Object"
(object.getName)();                    //"My Object"             
(object.getName = object.getName)();   //"The Window"
```

#### 内存泄漏

由于IE9之前的版本对 JavaScript 对象和 COM 对象使用不同的垃圾收集例程，因此闭包在 IE 的这些版本中会导致一些特殊的问题。具体来说，如果闭包的作用域链中保存着一个 HTML 元素，那么就意味着该元素将无法被销毁。例如：

``` javascript
function assignHandler() {
    var element = document.getElementById("someElement");
    element.onclick = function() {
        alert(element.id);
    };
}
```

以上代码创建了一个作为`element`元素事件处理程序的闭包，而这个闭包则又创建了一个循环引用。由于匿名函数保存了一个对`assingHandler()`的活动对象的引用，因此就会导致无法减少`element`的引用数。可以通过该写代码来解决，如下：

``` javascript
//防止内存泄露
function assignHandler() {
    var element = document.getElementById("someElement");
    var id = element.id;

    element.onclick = function() {
        alert(id);
    };

    element = null;
}
```

上面的代码中，通过把`element.id`的一个副本保存在一个变量中，并且在闭包中引用该变量消除了循环引用。但仅仅做到这一步，还是不能解决内存泄漏的问题。必须要记住：闭包会引用包含函数的整个活动对象，而其中包含着`element`。即使闭包不直接引用`element`，包含函数的活动对象中也仍然会保存一个应用。因此，有必要把`element`变量设置为`null`。这样就能够解除对 DOM 对象的引用，顺利地减少其引用数，确保正常回收其占用的内存。

### 模仿块级作用域

JavaScript 中没有块级作用域的概念。这意味着在块语句中定义的变量，实际上是在包含函数中而非语句中创建的，如下：

``` javascript
function outputNumbers(count) {
    for(var i=0; i<count; i++) {
        alert(i);
    }
    alert(i);    //计数
}
```

这个函数中定义了一个 for 循环，而变量 i 的初始值被设置为 0。在 Java、C++ 等环境中，变量 i 只会在 for 循环的语句块中有定义，循环一旦结束，变量 i 就会被销毁。可是在 JavaScript 中，变量 i 是定义在`outputNumbers()`的活动对象中的，因此从它有定义开始，既可以在函数内部随处访问它。即使像下面这样错误的重新声明同一个变量，也不会改变它的值。

``` javascript
function outputNumers(count) {
    for(var i = 0; i < count; i++) {
        alert(i);
    }

    var i;        //重新声明变量
    alert(i);     //计数
}
```

JavaScript 从来不会告诉你是否多次声明了同一个变量；遇到这种情况，它只会对后续的声明视而不见（不过，他会执行后续声明中的变量初始化）。匿名函数可以用来模仿块级作用域并避免这个问题。 

用作块级作用域（通常称为**私用作用域**）的匿名函数的语法如下，代码定义并立即调用了一个匿名函数。

``` javascript
//立即调用函数表达式
(function()) {
    //这里是块级作用域
})();
```

无论在什么地方，只要临时需要一些变量，就可以使用私有作用域，例如：

``` javascript
function outputNumbers(count) {
    (function() {
        for(var i=0; i<count; i++) {
            alert(i);
        }
    })();

    alert(i);    //导致一个错误！
}
```

这种技术经常在全局作用域中被用在函数外部，从而限制向全局作用域中添加过多的变量和函数。一般来说，我们都应该尽量少向全局作用域中添加变量和函数。在一个有很多开发人员共同参与的大型应用程序中，过多的全局变量和函数很容易导致命名冲突。而通过创建私有作用域，每个开发人员既可以使用自己的变量，又不必担心搞乱全局作用域。

### 私有变量

严格来讲，JavaScript 中**没有私有成员的概念**，所有对象属性都是公有的。不过，倒是有**私有变量的概念**。任何在函数中定义的变量，都可以认为是私有变量。私有变量包括**函数的参数**、**局部变量**和**在函数内部定义的其它函数**。 

如果在函数内部创建闭包，那么闭包通过自己的作用域链也可以访问这些变量。利用这一点，可以创建用于访问私有变量的公有方法。 有权访问私有变量和私有函数的公有方法称为**特权方法**（privileged method）。有两种在对象上创建特权方法的方式。第一种是在构造函数中定义特权方法，基本模式如下：

``` javascript
function MyObject() {
    //私有变量和私有函数
    var privateVariable = 10;

    function privateFunction() {
        return false;
    }
    //特权方法
    this.publicMethod = function() {
        privateVariable++;
        return privateFunction();
    };
}
```

这个模式在构造函数内部定义了所有私有变量和函数，然后创建了能够访问这些私有成员的特权方法。能够在构造函数中定义特权方法，是因为特权方法作为闭包有权访问在构造函数中定义的所有变量和函数。 

利用私有和特权成员，可以隐藏那些不应该被直接修改的数据，例如：

``` javascript
function Person(name) {
    this.getName = function() {
        return name;
    };
    this.setName = function (value) {
        name = value;
    };
}

var person = new Person("Nihcholas");
alert(person.getName());    //"Nicholas"
person.setName("Greg");
alert(person.getName());    //"Greg"
```

在构造函数定义特权方法的缺点是必须使用构造函数模式来达到这个目的。而构造函数模式的缺点是针对每个实例都会创建同样一组新方法，而使用静态私有变量来实现特权方法就可以避免这个问题。

#### 静态私有变量

通过在私有作用域中定义私有变量或函数，同样也可以创建特权方法，这个模式与在构造函数中定义特权方法的主要区别，就在于私有变量和函数是由实例共享的。由于特权方法是在原型上定义的，因此所有实例都使用同一个函数。而这个特权方法，作为一个闭包，总是保存着对包含作用域的引用。如下所示：

``` javascript
(function() {
    var name = "";

    Person = function(value) {
        name = value;
    };

    Person.prototye.getName = function() {
        return name;
    };

    Person.prototype.setName = function(value) {
        name = value;
    };
})();

var person1 = new Person("Nicholas");
alert(person1.getName());   //"Nicholas"
person1.setName("Greg");
alert(person1.getName());   //"Greg"

var person2 = new Person("Michael");
alert(person1.getName());   //"Michael"
alert(person2.getName());   //"Michael"
```

以这种方式创建静态私有变量会因为使用原型而增进代码复用，但每个实例都没有自己的私有变量。

#### 模块模式

前面的模式是用于为自定义类型创建私有变量和特权方法的。而道格拉斯所说的**模块模式（module pattern）**则是为单例创建私有变量和特权方法。所谓单例（singleton），指的就是只有一个实例的对象。

``` javascript
var singleton = {
    name : value,
    method : function() {
        //这里是方法的代码
    }
};
```

模块模式通过为单例添加私有变量和特权方法能够使其得到增强，其语法格式如下：

```javascript
var singleton = function() {
    //私有变量和私有函数
    var privateVariable = 10;

    function privateFunction() {
        return false;
    }

    //特权/公有方法和属性
    return {
        publicProperty: true,
        publicMethod: function() {
            privateVariable++;
            return privateFunction();
        }
    };
}();
```

这个模块模式使用了一个返回对象的匿名函数。在这个匿名函数内部，定义了私有变量和函数,，并将一个对象字面量作为返回值。返回的对象字面量中只包含可以公开的属性和方法。由于这个对象是在匿名函数内部定义的，因此它的公有方法有权访问私有变量和函数。从本质上来讲，这个对象字面量定义的是单例的公共接口。这种模式在需要对单例进行某些初始化，同时又需要维护其私有变量时是非常有用的，例如：

``` javascript
var application = function() {
    //私有变量和函数
    var components = new Array();
    //初始化
    components.push(new BaseComponent());
    //公共
    return {
        getComponentCount : function() {
            return components.length;
        },
        registerComponent : function(component) {
            if(typeof component == "object") {
                components.push(component);
            }
        }
    };
}();
```

**如果必须创建一个对象并以某些数据对其进行初始化，同时还要公开一些能够访问这些私有数据的方法，那么就可以使用模块模式。**以这种模式创建的每个单例都是`Object`的实例，因为最终要通过一个对象字面量来表示它。事实上这也没有什么关系，毕竟单例通常都是作为全局对象存在的，我们不会将它传递给一个函数。因此，也没什么必要使用`instanceof`操作符来检查其对象类型。

####增强的模块模式

有人进一步改进了模块模式，即在返回对象之前加入对其增强的代码。这种增强的模块模式适合那些单例必须是某种类型的实例，同时还必须添加某些属性和方法对其加以增强的情况。如下：

``` javascript
var singleton = function() {
    //私有变量和私有函数
    var privateVariable = 10;

    function privateFunction() {
        retur false;
    }
    //创建对象
    var object = new CustomType();
    //添加特权/公有属性和方法
    object.publicProperty = true;
    object.publicMethod = function() {
        privateVariable++;
        return privateFunction();
    };
    //返回这个对象
    return object;
}();
```

如果前面演示模块模式的例子中的`application`对象必须是`BaseComponent`的实例，那么就可以使用以下代码。

``` javascript
var application = function() {
    //私有变量和函数
    var components = new Array();
    //初始化
    components.push(new BaseComponent());
    //创建application的一个局部副本
    var app = new BaseComponent();
    //公共接口
    app.getComponentCount = function() {
        return components.length;
    };
    app.registerComponent = function(component) {
        if(typeof component == "object") {
            components.push(component);
        }
    };
    //返回这个副本
    return app;
}();
```

