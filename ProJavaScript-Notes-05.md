---
title: '重读 JS 高程（Chapter06）'
date: 2017-08-28 22:07:38
tags: javascript
category: javascript
---

## Chapter06. 面向对象的程序设计

面向对象（Object-Oriented, OO）的标志就是有类的概念，通过类可以创建任意个具有相同属性和方法的对象。然而 ECMAScript 中没有类的概念，因此它的对象也与基于类的语言中的对象有所不同。ECMA-262 把对象定义为：

> 无序属性的集合，其属性可以包含基本值、对象或者函数。

ECMAScript 的对象可被看做散列表：即一组名值对，其中值可以是数据或函数。每个对象都是基于一个引用类型创建的，这个引用类型可以是原生类型，也可以是自定义类型。

### 理解对象

**ES5 在定义只有内部才用的特性时（attribute）时，描述了属性（property）的各种特征**。这些特性是为了实现 JavaScript 引擎用的，不能直接访问它们。为了表示特性是内部值，该规范把它们放在了两对方括号中，例如`[[Enumerable]]`。ECMAScript 中有两种属性：数据属性和访问器属性。

#### 数据属性

**数据属性包含一个数据值的位置**。在这个位置可以读取和写入值。数据属性有 4 个描述其行为的特性：

- `[[Configurable]]`：表示能否通过`delete`删除属性从而重新定义属性，以及除 writable 特性外的其他特性是否可以被修改，或者能否把属性修改为访问器属性。
- `[[Enumerable]]`：表示能否通过`for-in`循环返回属性。
- `[[Writable]]`：表示能否修改属性的值。
- `[[Value]]`：包含这个属性的数据值。读取属性值的时候，从这个位置读；写入属性值的时候，把新值保存在这个位置。

直接在对象上定义的属性，它们的`[[Configurable]]`、`[[Enumerable]]`和`[[Writable]]`特性都被设置为`true`，而`[[Value]]`特性被设置为特定的值。

要修改属性默认的特性，必须使用`Object.defineProperty(obj, prop, descriptor)`方法。该方法接收三个参数：

- `obj`：需要被操作的目标对象。
- `prop`：目标对象需要定义或修改的属性的名称。
- `descriptor`：将被定义或修改的属性的描述符。可选值：configurable、enumerable、writable 和 value。

``` javascript
var person = {};
Object.defineProperty(person, "name", {
    configurable: false;
    value: "Nicholas"
});
alert(person.name);      //"Nicholas"
delete person.name;
alert(person.name);      //"Nicholas"
```

注意： 一旦把 configurable 定义为不可配置的，就不能将它再变回可配置的，修改除 writable 之外的特性，都会导致错误。

使用`Object.defineProperty()`**创建新属性**时，若不指定参数，`configurable`、`enumerable`和`writable`特性的默认值都是`false`。 

<!-- more -->

#### 访问器属性

**访问器属性不包含数据值**，它们包含一对·getter 和 setter 函数（可选）。在读取访问器属性时，会调用 getter 函数，这个函数负责返回有效的值；在写入访问器属性时，会调用 setter 函数并传入新值，这个函数负责决定如何处理数据。访问器属性有如下 4 个特性：

- `[[Configurable]]`：表示能否通过`delete`删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为数据属性。默认值 true。
- `[[Enumerable]]`：表示能否通过`for-in`循环返回属性。默认值 true。
- `[[Get]]`：在读取属性时调用的函数。默认值 undefined。
- `[[Set]]`：在写入属性时调用的函数。默认值 undefined。

访问器属性不能直接定义，必须使用`Object.defineProperty()`来定义。例如：

``` javascript
var book = {
    _year: 2004,
    edition: 1
};

Object.defineProperty(book, "year", {
    get: function() {
        return this._year;
    },
    set: function(newValue) {
        if (newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
        }
    }
});

book.year = 2005;
alert(book.edition);        //2
```

#### 定义多个属性与读取属性

- `Object.defineProperties(obj, props)`：可以通过描述符一次定义多个属性。它接收两个对象参数：
  - `obj`：要添加和修改其属性的对象。
  - `props`：要添加或修改的属性。
- `Object.getOwnPropertyDescriptor()`：可以取得给定属性的描述符。方法接受两个参数：**属性所在的对象**和**要读取其描述符的属性名称**，**返回值是一个对象**。注：在JavaScript中，可以针对任何对象（包括 DOM 和 BOM 对象），使用`Object.getOwnPropertyDescriptor()`方法。

```
var book = {};

Object.defineProperties(book, {
    _year: {
        value: 2004
    },
    edition: {
        value: 1
    },
    year: {
        get: function() {
            return this._year;
        },
        set: function(newValue) {
            if (newValue > 2004) {
                this._year = newValue;
                this.edition += newValue - 2004;
            }
        }
    }
});

var descriptor = Object.getOwnPropertyDescriptor(book, "_year");
alert(descriptor.value);            //2004
alert(descriptor.configurable);     //false
alert(typeof descriptor.get);       //"undefined"

var descriptor = Object.getOwnPropertyDescriptor(book, "year");
alert(descriptor.value);        //undefined
alert(descriptor.enumerable);   //false
alert(typeof descriptor.get);   //"function"
```

### 创建对象

虽然 Object 构造函数或对象字面量都可以用来创建单个对象，但这些方法有个明显的缺点：**使用同一个接口创建很多对象，会产生大量重复代码。**

#### 工厂模式

``` javascript
function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        alert(this.name);
    };
    return o;
}

var person1 = createPerson("Nicholas", 29, "Software Engineer");
var person2 = createPerson("Greg", 27, "Doctor");
```

工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题（即怎样知道一个对象的类型）。

#### 构造函数模式

像 Object 和 Array 这样的原生构造函数，在运行时会自动出现在执行环境中。此外，**也可以创建自定义的构造函数，从而定义自定义的对象类型的属性和方法**。例如：

``` javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function() {
        alert(this.name);
    };
)
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

Person() 中的代码除了与 createPerson() 中相同的部分外，还存在以下不同之处：

- 没有显式地创建对象
- 直接将属性和方法赋给了this对象
- 没有 return 语句

**按照惯例，构造函数始终都应该以一个大写字母开头，而非构造函数则应该以一个小写字母开头。**主要是为了区别 ECMAScript 中的其它函数；因为构造函数本身也是函数，只不过可以用来创建对象而已。 

**要创建 Person 的新实例，必须使用 new 操作符**。以这种方式调用构造函数实际上经历以下 4 个步骤：

1. 创建一个新对象
2. 将构造函数的作用域赋给新对象（因此 this 就指向了这个新对象）
3. 执行构造函数中的代码（为这个新对象添加属性）
4. 返回新对象

在前面例子的最后，person1 和 person2 分别保存着 Person 的一个不同实例。这两个对象都有一个 constructor（构造函数）属性，该属性指向 Person。

``` javascript
alert(person1.constrcutor == Person); //true
alert(person2.constrcutor == Person); //true

alert(person1 instanceof Object); //true
alert(person1 instanceof Person); //true
alert(person2 instanceof Object); //true
alert(person2 instanceof Person); //true
```

创建自定义的构造函数意味着将来可以将它的实例标志为一种特定的类型；而这正是构造函数模式胜过工厂模式的地方。

##### 将构造函数当做函数

构造函数与其他函数的唯一区别，就在于调用它们的方式不同。不过，构造函数毕竟也是函数，不存在定义构造函数的特殊语法。任何函数，只要通过 new 操作符来调用，那它就可以作为构造函数；而任何函数，如果不通过 new 操作符来调用，那它跟普通函数也不会有什么两样。如下：

``` javascript
//当作构造函数使用
var person = new Person("Nicholas", 29, "Software Engineer");
person.sayName();   //"Nicholas"

//作为普通函数调用
Person("Greg", 27, "Doctor");   //添加到window
window.sayName();   //"Greg"

//在另一个对象的作用域中调用
var o = new Object();
Person.call(o, "Kristen", 25, "Nurse");
o.sayName();       //"Kristen"
```

##### 构造函数的问题

构造函数的主要问题，就是每个方法都要在每个实例上重新创建一遍。在前面的例子中，person1 和person2 都有一个名为 sayName() 的方法，但那两个方法不是同一个 Function 的实例。在 ECMAScript 中的函数是对象，每定义一个函数，也就是实例化了一个对象。**<u>因此，不同实例上的同名函数是不相等的</u>** ：

``` javascript
alert(person1.sayName == person2.sayName);  //false
```

创建两个完成同样任务的 Function 实例的确没有必要；况且有 this 对象在，根本不用在执行代码前就把函数绑定到特定对象上面。因此，可以把函数定义转移到构造函数外部来解决这个问题。

``` javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = sayName;
}

function sayName() {
    alert(this.name);
}

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

这样做虽然解决了两个函数做同一件事的问题，但是带来了新问题：**在全局作用域中定义的函数实际上只能被某个对象调用，这让全局作用域有点名不副实**。更让人无法接受的是：**如果对象需要定义很多方法，那么就要定义很多个全局函数，于是我们这个自定义的引用类型就丝毫没有封装性可言了**。好在，这些问题可通过原型模式解决。

#### 原型模式

我们创建的每个函数都有一个 prototype（原型）属性，这个属性是一个指针，指向一个对象，而这个对象的用途是**包含可以由特定类型的所有实例共享的属性和方法**。按照字面意思来理解，prototype 就是通过调用构造函数而创建的那个对象实例的原型对象。**使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法**。如下：

``` javascript
function Person() {
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
    alert(this.name);
};

var person1 = new Person();
person1.sayName();  //"Nicholas" 
var person2 = new Person();
person2.sayName();  //"Nicholas"
alert(person1.sayName == person2.sayName);  //true
```

##### 理解原型对象

无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个 prototype 属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个 constructor（构造函数）属性，这个属性包含一个指向 prototype 属性所在函数的指针。

可以推断，Person.prototype.constructor 指向 Person。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

创建了自定义的构造函数之后，其原型对象默认只会取得 constructor 属性；至于其他方法，则都是从 Object 继承而来的。当调用构造函数创建一个新实例后，该实例的内部将包含一个指针（内部属性），指向构造函数的原型对象。ES5 管这个指针叫`[[Protoype]]`。虽然脚本中没有标准的方式访问`[[Protoype]]`，但是浏览器不约而同的支持`__proto__`属性，等价于`[[Protoype]]`，这个连接存在于实例与构造函数的原型对象之间。

- `isPrototypeOf()`：确定对象原型方法。

``` javascript
alert(Person.prototype.isPrototypeOf(person1)); //true
```

- `Object.getPrototypeOf()`：ES5 新增方法，取得对象的原型对象。

``` javascript
alert(Object.getPrototypeOf(person1) == Person.prototype);  //true
alert(Object.getPrototypeOf(person1).name); //"Nicholas"
```

- `hasOwnProperty()`：检测一个属性是存在于实例中，还是存在于原型中。

``` javascript
alert(person1.hasOwnProperty("name"));  //false
person1.name = "Greg";
alert(person1.name);                    //"Greg"
alert(person1.hasOwnProperty("name"));  //true
```

每当代码读取某个对象的某个属性时，都会执行一次搜索，目标是具有给定名字的属性。第一次搜索对象实例本身，找到了就返回该属性值，如果未找到，继续搜索原型对象。所以，当为对象实例添加一个属性时，这个属性就会**屏蔽**原型对象中保存的同名属性。这正是多个对象实例共享原型所保存的属性和方法的基本原理。

##### 原型与 in 操作符

有两种方式使用`in`操作符：单独使用何在`for-in`循环中使用。单独使用时，`in`操作符会在对象能够访问给定属性时返回`true`，无论该属性存在于实例中还是原型中。而同时使用`hasOwnProperty()`方法和`in`操作符，就可以确定该属性到底是存在于对象中，还是存在于原型中。

``` javascript
function hasPrototypeProperty(object, name) {
    return !object.hasOwnProperty(name) && (name in object);
}
```

在使用`for-in`循环时，返回的是所有能够通过对象访问的、可枚举的（enumerated）属性，其中既包括存在于实例中的属性，也包括存在于原型中的属性。

`Oject.keys()`接收一个对象作为参数，**返回一个包含所有可枚举属性的字符串数组**。例如：

``` javascript
function Person() {
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
    alert(this.name);
};
var keys = Object.keys(Person.prototype);
alert(keys);        //"name, age, jog, sayName"

var p1 = new Person();
p1.name = "Rob";
p1.age = 31;
var p1keys = Object.keys(p1);
alert(p1keys);      //"name, age"
```

如果想得到所有实例属性，**无论它是否可枚举**，可以使用`Object.getOwnPropertyNames()`方法：

``` javascript
var keys = Object.getOwnPropertyNames(Person.prototype);
alert(keys);        //"constructor, name, age, job, sayName"
```

##### 更简单的原型语法

常见的做法是用一个包含所有属性和方法的对象字面量来重写整个原型对象，如下：

``` javascript
function Person() {
}

Person.prototype = {
    constructor : Person,
    name : "Nicholoas",
    age : 29,
    job : "Software Engineer",
    sayName : function () {
        alert(this.name);
    }
};
```

注意：每创建一个函数，就会同时创建它的 prototype 对象，这个对象也会自动获得 constructor 。由于这里重写了 prototype 对象，因此 constructor 属性也就变成了新对象的 constructor 属性——Object，不再指向 Person 函数。所以需要显式设置 constructor 属性到适当的值。

这种方式重设 constructor 属性会导致它的`[[Enumerable]]`特性被设置为`true`。可以用`Object.defineProperty()`重设为 false。

##### 原型的动态性

由于在原型中查找值的过程是一次搜索，因此对原型对象所做的任何修改都能够立即从实例上反映出来——**即使先创建了实例后修改原型也照样如此**。但是如果重写了原型对象就会切断现有原型与之前已经存在的对象实例之间的联系，它们引用的仍然是最初的原型。

##### 原生对象的原型

原型模式的重要性不仅体现在创建自定义类型方面，就连所有原生的引用类型，都是采用这种模式创建的。所有原生引用类型（Object、Array、String等等）都在其构造函数的原型上定义了方法。

##### 原型对象的问题

首先，它省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下都将取得相同的属性值。最重要的，其共享的本性对于函数非常合适，对于包含基本值的属性也说得过去，通过在实例上添加一个同名属性，可以隐藏原型中的对应属性。但对于包含引用类型值的属性来说，问题就比较突出，比如数组：

``` javascript
function Person(name, age, job) {
}

Person.prototype = {
    constructor : Person,
    ...
    this.friends : ["Shelby", "Court"],
    sayName : function() {
        alert(this.name);
    }
}

var person1 = new Person();
var person2 = new Person();

person1.friends.push("Van");

alert(person1.friends);     //"Shelby,Count,Van"
alert(person2.friends);     //"Shelby,Count,Van"
alert(perosn1.friends == person2.friends);      //true
```

#### 组合使用构造函数模式和原型模式

创建自定义类型的最常见方式，就是组合使用构造函数模式与原型模式。**构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。**结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存。另外，这种混合模式还支持向构造函数传递参数，可谓是集两种模式之长。

``` javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ["Shelby", "Court"];
}

Person.prototype = {
    constructor : Person,
    sayName : function() {
        alert(this.name);
    }
}

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");

person1.friends.push("Van");
alert(person1.friends);     //"Shelby,Count,Van"
alert(person2.friends);     //"Shelby,Count"
alert(perosn1.friends == person2.friends);      //false
alert(person1.sayName == person2.sayName);      //true
```

实例属性都是在构造函数中定义，共享属性`constructor`和方法`sayName()`则是在原型中定义的。这种构造函数和原型混成的模式，是目前在 ECMAScript 中使用最广泛、认同度最高的一种创建自定义类型的方法。可以说，这是用来定义引用类型的一种默认模式。

#### 动态原型模式

动态原型模式把所有信息都封装在了构造函数中，而通过构造函数中初始化原型（仅在必要的情况下），又保持了同时使用构造函数和原型的优点。

``` javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;

    if (typeof this.sayName != "function" {
        Person.prototype.sayName = function() {
            alert(this.name);
        };
    }
}

var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();
```

#### 寄生构造函数模式

通常，在前几种模式都不适用的情况下，可以使用寄生（parasitic）构造函数模式。这种模式的基本思想是创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后再返回新创建的对象；但从表面上看，这个函数又很像是典型的构造函数。这个模式可以在特殊的情况下用来为对象创建构造函数。假设我们想创建一个具有额外方法的特殊数组。由于不能直接修改 Array 构造函数，可以使用这个模式。

``` javascript
function SpecialArray() {
    //创建数组
    var values = new Array();
    //添加值
    values.push.apply(values, arguments);
    //添加方法
    values.toPipedString = function() {
        return this.join("|");
    };
    //返回数组
    return values;
}

var colors = new SpecialArray("red", "blue", "green");
alert(colors.toPipedString());  //"red|blue|green"
```

注意：返回的对象与构造函数或者与构造函数的原型属性之间没有关系；也就是说，构造函数返回的对象与在构造函数外部创建的对象没有什么不同。因此，不能依赖 instanceof 操作符来确定对象类型。

#### 稳妥构造函数模式

Douglas Crockford 发明了 JavaScript 中的**稳妥对象**（durable objects）这个概念。所谓稳妥对象，指的是没有公共属性，而且其方法也不引用 this 的对象。稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用 this 和 new），或者在防止数据被其他应用程序改动时使用。稳妥构造函数遵循与寄生构造函数类似的模式，但有两点不同：一是新创建的对象的实例方法不引用 this；二是不适用 new 操作符调用构造函数。

``` javascript
function Person(name, age, job) {
    //创建要返回的对象
    var o = new Object();
    //可以在这里定义私有变量和函数

    //添加方法
    o.sayName function() {
        alert(name);
    };
    //返回对象
    return o;
}
```

除了使用`sayName()`方法外，没有别的方式可以访问`name`的值。

``` javascript
var friend = Person("Nicholas", 29, "Software Engineer");
friend.sayName();   //"Nicholas"
```

变量 friend 中保存的是一个稳妥对象，而除了调用 sayName() 方法外，没有别的方式可以访问其数据成员。即使有其他代码会给这个对象添加方法或数据成员，但也不可能有别的方法访问到构造函数中的原始数据。稳妥构造函数模式提供的这种安全性，使得它非常适合在某些安全环境下使用。

### 继承

许多 OO 语言都支持两种继承方式：接口继承和实现继承。接口继承只继承方法签名，而实现继承则继承实际的方法。由于函数没有签名，在 ECMAScript 中无法实现接口继承。ECMAScript 只支持实现继承，而且实现继承主要是依靠原型链来实现的。

#### 原型链

ECMAScript 中描述了**原型链**的概念，并将原型链作为实现继承的主要方法。其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。 实现原型链有一种基本模式，代码如下：

``` javascript
function SuperType() {
    this.property = true;
}

SuperType.prototype.getSuperValue = function() {
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

//继承了SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function() {
    return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue());    //true
```

上述代码定义了两个类型：SuperType 和 SubType。它们的主要区别是 SubType 继承了 SuperType，而继承是通过创建 SuperType 的实例，并将该实例赋给 SubType.prototype 实现的。实现的本质是重写原型对象，代之以一个新类型的实例。该例中的实例以及构造函数和原型之间的关系如图： 

![原型链](/images/prototype.jpg)

本例中，我们没有使用 SubType 默认提供的原型，而是给它换了一个新原型；这个新原型就是 SuperType 的实例。于是，新原型不仅具有作为一个 SuperType 的实例所拥有的全部属性和方法，而且其内部还有一个指针，指向了 SuperType 的原型。 

通过实现原型链，本质上扩展了本章前面介绍的原型搜索机制。以调用`instance.getSuperValue()`为例，会经历三个搜索步骤：

1. 搜索实例；
2. 搜索 SubType.prototype
3. 搜索 SuperType.prototype，最后一步才会找到该方法。

##### 默认的类型 

所有引用类型默认都继承了`Object`，而这个继承也是通过原型链实现的。所有函数的默认原型都是`Object`的实例，因此默认原型都会包含一个内部指针，指向`Object.prototype`。这也是所有自定义类型都会继承`toString()`、`valueOf()`等默认方法的根本原因。下图是上面例子的完整原型链： 

![完整原型链](/images/prototype-chain.jpg)

##### 谨慎定义方法 

- 子类型有时需要重写超类型中的某个方法，或者需要添加超类型中不存在的某个方法。但无论怎样，给远行添加方法的代码一定要放在替换原型的语句之后。

- 在通过原型链实现继承时， 不能使用对象字面量创建原型方法。因为这样做就会重写原型链。

##### 原型链的问题 

- 最主要的问题来自包含引用类型值的原型。包含引用类型值的原型属性会被所有实例共享；而这也正是为什么要在构造函数中，而不是在原型对象中定义属性的原因。

``` javascript
function SuperType() {
    this.colors = ["red", "blue", "green"];
}

function SubType() {
}

//继承了SuperType
SubType.prototype = new SuperType();

var instance1 = new SuperType();
instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"

var instance2 = new SubType();
alert(instance2.colors);    //"red, blue, green, black"
```

该代码结果是`SubType`的所有实例都会共享`colors`属性。

- 原型链的第二个问题是：在创建子类型的实例时，不能向超类型的构造函数中传递参数。实际上，应该说是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。有鉴于此，再加上前面刚刚讨论过的由于原型中包含引用类型值所带来的问题，实践中很少会单独使用原型链。

#### 借用构造函数

在解决原型中包含引用类型值所带来的问题的过程中，开发人员开始使用一种叫做**借用构造函数**（constructor stealing）的技术（有时候也叫伪造对象或经典继承）。这种技术的基本思想相当简单，即**在子类型构造函数的内部调用超类型构造函数。**函数只不过是在特定环境中执行代码的对象，因此通过使用`apply()`和`call()`方法也可以在（将来）新创建的对象上执行构造函数，如下：

``` javascript
function SuperType() {
    this.colors = ["red", "blue", "green"];
}

function SubType() {
    //继承了SuperType
    SuperType.call(this);
}

var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"

var instance2 = new SubType();
alert(instance2.colors);    //"red, blue, green"
```

**第 7 行**代码“借调”了超类型的构造函数。通过使用`call()`方法（或`apply()`方法也可以），我们实际上是在（未来将要）新创建的 SubType 实例的环境下调用了 SuperType 构造函数。这样一来，就会在新 SubType 对象上执行 SuperType 函数中定义的所有对象初始化代码。结果，SubType 的每个实例就都会具有自己的 colors 属性的副本了。

##### 传递参数 

相对于原型链而言，借用构造函数有一个很大的优势，即可以在子类型构造函数中向超类型构造函数传递参数。

``` javascript
function SuperType(name) {
    this.name = name;
}

function SubType() {
    //继承了SuperType,同时还传递了参数
    SuperType.call(this, "Nicholas");

    //实例属性
    this.age = 29;
}

var instance = new SubType();
alert(instance.name);   //"Nicholas";
alert(instance.age);    //29
```

以上代码中的 SuperType 只接受一个参数 name ，该参数会直接赋给一个属性。在 SubType 构造函数内部调用 SuperType 构造函数时，实际上是为 SuperType 的实例设置了 name 属性。为了确保 SuperType 构造函数不会重写子类型的属性，可以在调用超类型构造函数后，再添加应该在子类型中定义的属性。

##### 借用构造函数的问题 

如果仅仅是借用构造函数，那么也将无法避免构造函数模式存在的问题——方法都在构造函数中定义，因此函数复用就无从谈起了。而且，在超类型的原型中定义的方法，对子类型而言也是不可见的，结果所有类型都只能使用构造函数模式。因此，借用构造函数的技术也是很少单独使用的。

#### 组合继承

组合继承（combination inheritance），有时候也叫做伪经典继承，指的是将原型链和借用构造函数的技术组合到一块，从而发挥二者之长的一种继承模式。其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。如下：

``` javascript
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    //继承属性
    SuperType.call(this, name);
    this.age = age;
}

//继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SupType.prototype.sayAge = function() {
    alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"
instance1.sayName();        //"Nicolas"
instance1.sayAge();         //29

var instance2 = new SubType("Greg", 27);
alert(instance2.colors);    //"red, blue, green"
instance2.sayName();        //"Greg"
instance2.sayAge();         //27
```

此例中， SuperType 构造函数定义了两个属性：`name`和`colors`。SuperType 的原型定义了一个方法`sayName()`。SubType 构造函数在调用 SuperType 构造函数时传入了`name`参数，紧接着又定义了它自己的属性`age`。然后，将 SuperType 的实例赋值给 SubType 的原型，然后又在该新原型上定义了方法`sayAge()`。这样一来，就可以让两个不同的 SubType 的原型既分别拥有自己属性——包括`colors`属性，又可以使用相同的方法了。 

组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点，成为了 JavaScript 中最常用的继承模式。而且，`instanceof`和`isPrototypeOf()`也能够用于识别基于组合继承创建的对象。

#### 原型式继承

道格拉斯 · 克罗克福德在2006年介绍了一种实现继承的方法，这种方法并没有使用严格意义上的构造函数。他的想法是借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。为了达到这个目的，他给出了如下函数：

``` javascript
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}
```

在`object()`函数内部，先创建了一个临时的构造函数，然后将传入的对象作为这个构造函数的原型，最后返回了这个临时类型的一个新实例。从本质上讲，`object()`对传入其中的对象执行了一次浅复制。如下：

``` javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = object(person);
anotherPerson.name = "Greg";
anttherPerson.friends.push("Rob");

var yetAnotherPerosn = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

alert(person.friends);  //"Shelby, Court, Van, Rob, Barbie"
```

克罗克福德主张的这种原型式继承，要求你必须有一个对象可以作为另一个对象的基础。如果有这么一个对象的话，可以把它传递给`object()`函数，然后再根据具体需求对得到的对象加以修改即可。其中，`person.friends`不仅属于`person`所有，而且也会被`anotherPerson`以及`yetAnotherPerson`共享。 

ES5 通过新增`Object.create()`方法规范化了原型式继承。这个方法接收两个参数：一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象。在传入一个参数的情况下，`Object.create()`与`object()`方法的行为相同。

``` javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = Object.create(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerosn = Object.create(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

alert(person.friends);  //"Shelby, Court, Van, Rob, Barbie"
```

`object.create()`方法的第二个参数与`object.defineProperties()`方法的第二个参数格式相同：每个属性都是通过自己的描述符定义的。以这种方式指定的任何属性都会覆盖原型对象上的同名属性。

``` javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"] 
};

var anotherPerson = Object.create(person, {
    name: {
        value: "Greg"
    }
});

alert(anotherPerson.name);  //"Greg"
```

#### 寄生式继承

寄生式（parasitic）继承是与原型式继承紧密相关的一种思路，并且同样也是有克罗克福德推而广之的。寄生式继承的思路与寄生构造函数和工厂模式类似，即创建一个仅用于封装继承过程的函数，该函数内部以某种方式来增强对象，最后再像真的是它做了所有工作一样返回对象。如下：

``` javascript
function createAnother(original) {
    var clone = object(original);    //通过调用函数创建一个新对象
    clone.sayHi = function() {       //以某种方式来增强这个对象
        alert("hi");
    };
    return clone;                    //返回这个对象
}

var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = createAnother(person);
anotherPerson.sayHi();  //"hi"
```

注意：使用寄生式继承来为对象添加函数，会由于不能做到函数复用而降低效率；这一点与构造函数模式类似。

#### 寄生组合式继承

组合继承是JavaScript最常用的继承模式；不过，它也有自己的不足。**组合继承最大的问题就是无论什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。**子类型最终会包含超类型对象的全部实例属性，但我们不得不在调用子类型构造函数时重写这些属性。如下：

``` javascript
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    SuperType.call(this, name);             //第二次调用SuperType()
    this.age = age;
}

SubType.prototype = new SuperType();        //第一次调用SuperType()
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
    alert(this.age);
};
```

第一次调用 SuperType() 构造函数时，SubType.prototype 会得到两个属性：`name`和`colors`；它们都是SuperType 的实例属性，只不过现在位于 SubType 的原型中。当调用 SubType 构造函数时，又会调用一次 SuperType 构造函数，这一次又在新对象上创建了实例属性`name`和`colors`。于是，这两个属性就屏蔽了原型中的两个同名属性。下图展示了上述过程：

![组合继承](/images/extend.jpg)

而寄生组合式继承，通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。其背后的思路是：不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型原型的一个副本而已。本质上，就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型。基本模式如下：

``` javascript
function inheritPrototype(subType, superType) {
    var prototype = object(superType.prototype);    //创建对象
    prototype.constructor = subType;                //增强对象
    subType.prototype = prototype;                  //指定对象
}
```

该示例中`inheritPrototype()`函数实现了寄生组合式继承的最简单形式。这个函数接收两个参数：子类型构造函数和超类型构造函数。在函数内部，第一步是创建超类型原型的一个副本。第二步是为创建的副本添加`constructor`属性，从而弥补因重写圆形而失去的默认的`constructor`属性。最后一步，将新创建的对象（即副本）赋值给子类型的原型。这样，就可以调用`inheritPrototype()`函数的语句，去替换前面例子中为子类型原型赋值的语句了。如：

``` javascript
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    SuperType.call(this, name);
    this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function() {
    alert(this.age);
}
```

该例的高效率体现在它只调用了一次`SuperType`构造函数，并且因此避免了在`SubType.prototype`上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用`instanceof`和`isPrototypeOf()`。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。