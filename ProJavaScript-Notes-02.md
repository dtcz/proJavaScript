---
title: '重读 JS 高程（Chapter03）'
date: 2017-08-14 12:35:56
tags: javascript
category: javascript
---

## Chapter03. 基本概念

### 语法

ECMAScript 的语法大量借鉴了 C 及其他类 C 语言（如 Perl 和 Java）的语法。

#### 区分大小写

ECMAScript 中的一切（变量、函数名和操作符）都区分大小写。

#### 标识符

指变量、函数、属性的名字，或者函数的参数。标识符可以是按照下列格式规则组合起来的一个或多个字符：

- 第一个字符必须是一个字母、下划线（\_）或一个美元符号（$）；
- 其他字符可以是字母、下划线、美元符号或数字。

标识符中的字母也可以包含扩展的 ASCII 或 Unicode 字母字符，但不推荐这样做。

#### 注释

``` javascript
// 单行注释

/*
*  这是一个多行
*  （块级）注释
*/
```

<!-- more -->

#### 严格模式

ES5 引入了严格模式的概念，在严格模式下，ES3 中的一些不确定行为将得到处理，而且队某些不安全的操作也会抛出错误。要在整个脚本中启用严格模式，可以在顶部添加如下代码：

```javascript
"use strict";
```

这行代码看起来像是字符串，而且也没有赋值给任何变量，但其实它是一个编译指示（pragma），用于告诉支持的JavaScript引擎切换到严格模式。在函数内部的上方包含这条编译指示，也可以指定函数在严格模式下执行：

``` javascript
function doSomething(){
  "use strict";
  //函数体
}
```

严格模式下，JavaScript 的执行结果会有很大不同，支持严格模式的浏览器包括 IE10+、Firefox4+、Safari 5.1+、Opera 12+ 和 Chrome。

#### 语句

ECMAScript 中的语句以一个分号结尾；如果省略分号，则由解析器确定语句的结尾。建议任何情况下都不省略分号。

#### 变量

ECMAScript 的变量是松散类型的，即可以用来保存任何类型的数据。定义变量时要使用 var 操作符，后跟一个变量名（标识符），比如：

``` javascript
var message;      //未经过初始化的变量，会保存一个特殊的值——undefined
```

也可以在定义变量时初始化，同时修改变量的值也可以修改变量类型：

``` javascript
var message = "hi";
message = 100;      //有效
```

可以用一条语句定义多个变量：

```javascript
var message = "hi",
    found = false,
    age = 29;
```

**有一点必须注意，用 var 操作符定义的变量将成为定义该变量的作用域中的局部变量，如果省略 var  操作符，就会创建一个全局变量。**

``` javascript
function test(){
    var message = "hi"; // 局部变量
}
test();
alert(message); // Error!

function test(){
    message = "hi"; // 全局变量
}
test();
alert(message); // "hi"
```

注意，虽然省略 var 操作符可以定义全局变量，但这也不是我们推荐的做法。因为在局部作用域中定义的全局变量很难维护，而且如果有意地省略了 var 操作符，也会由于相应变量不会马上有定义而导致不必要的混乱。给未经声明的变量赋值在严格模式下会导致跑出 ReferenceError 错误。

### 数据类型

ES5中有 5 种**基本数据类型**：undefined、null、Boolean、Number、String。还有一种复杂数据类型 Object。

#### typeof 操作符

typeof 是用来检测给定变量的数据类型的操作符。对一个值使用 typeof 操作符可能返回下列某个字符串：

- "undefined"
- "boolean"
- "string"
- "number"
- "object"  // 如果这个值是对象或者null
- "function"

#### undefined

在使用 var 声明变量但未对其加以初始化时，这个变量的值就是 undefined。

值得注意的一点是，包含undefined值的变量与尚未定义的变量还是不一样。比如：

``` javascript
var message; //undefined
//下面这个变量没有声明
//var age;

alert(message);  // "undefined"
alert(age);      // 产生错误
```

然而，令人困惑的一点是：对未初始化的变量执行 typeof 操作符会返回 undefined 值，而对未声明的变量执行 typeof 操作符同样会返回 undefined 值。

``` javascript
var message; //undefined
//下面这个变量没有声明
//var age;
 
alert(typeof message);  // "undefined"
alert(typeof age);      // "undefined"
```

**因此，在声明变量时同时初始化变量是明智的选择。**

#### null

从逻辑角度来看，null 值表示一个空对象指针，而这也正是使用 typeof 操作符检测 null 值时会返回"object"的原因。

``` javascript
var car = null;
alert(typeof car); // "object"
```

如果定义的变量准备再将来用于保存对象，那么最好将该变量初始化为 null 而不是其他值。这样只要直接检查 null 值就可以知道相应的变量量是否已经保存了一个对象的引用

```javascript
if (car != null) {
    //对 car 对象执行某些操作
}
```

**实际上，undefined 值是派生自 null 值的，因此 ECMA-262 规定它们的相等性测试要返回 true。**

``` javascript
alert(null == undefined); // true
```

不过需要注意的是这个操作符（==）会转换其操作数，使它们变成同一类型。

```javascript
alert(null === undefined); // false
```

#### Boolean 类型

该类型只有两个字面值：true 和 false。这两个值与数字值不是一回事，因此 true 不一定等于 1，而 false 也不一定等于 0。

Boolean 值和其他类型的转换规则:

|   数据类型    |   转换为true的值    | 转换为false的值 |
| :-------: | :------------: | :--------: |
|  Boolean  |      true      |   false    |
|  String   |    任何非空字符串     |  ""（空字符串）  |
|  Number   | 任何非零数字值（包括无穷大） |  0 和 NaN   |
|  Object   |      任何对象      |    null    |
| Undefined |      N/A       | undefined  |

#### Number 类型

这种类型使用 IEEE754 格式来表示整数和浮点数值。

```
var intNum = 55; // 十进制整数

var octalNum1 = 070; // 八进制的56
var octalNum1 = 079; // 无效的八进制数值——解析为79

var octalNum1 = 08; // 无效的八进制数值——解析为8
var hexNum1 = 0xA;  // 十六进制的10
var hexNum2 = 0x1F; // 十六进制的31
```

**注意，八进制字面量在严格模式下是无效的，会导致抛出错误。**

- 数值范围

ECMAScript 能够表示的最小数值保存在 Number.MIN_VALUE 中——在多数浏览器中，这个值是 5e-324；能够 Number.MAX_VALUE 中——在大多数浏览器中，这个值是1.7976931348623157e+308。如果某次计算的结果得到了一个超过 JavaScript 数值范围的值，那么这个数值将会自动转换为 Infinity 值，如果这个数值是负数，则会转换成 -Infinity（负无穷），如果这个数值是正数，则会转换成 Infinity（正无穷）。要确定一个数值是不是有穷的，可以使用 isFinite() 函数。

- NaN

NaN，即非数值（Not a Number）是一个特殊的数值，这个数值用于表示一个本来要返回数值的操作数未返回数值的情况。在 ECMAScript 中，任何数值除以 0 会返回 NaN ，因此不会导致错误，影响其他代码运行。

任何涉及NaN的操作都会返回NaN，这个特点在多步计算中可能导致问题；NaN 与任何值都不相等，包括 NaN 本身。

``` javascript
alert(NaN == NaN); //false
```

针对 NaN 的这两个特点，ECMAScript 定义了 isNaN() 函数。

``` javascript
alert(isNaN(NaN));            //true
alert(isNaN(10));             //false
alert(isNaN("10"));           //false
alert(isNaN("blue"));         //true
alert(isNaN(true));           //false
```

- 数值转换

有 3 个函数可以把非数值转换为数值：Number()、parseInt() 和 parseFloat()。第一个函数可以用于任何数据类型，后两个函数则专门用于把字符串转换成数值。值得注意的是，Number() 对空字符串返回0，parseInt() 转换空字符串会返回 NaN。

处理整数最常用的还是 parseInt() ，它会忽略字符前面的空格，直到找到第一个非空格字符。如果第一个字符不是数字字符或者负号，parseInt() 就会返回 NaN；也就是说，用 parseInt() 转换空字符串会返回 NaN 。如果第一个字符是数字字符，parseInt() 会继续解析第二个字符，直到解析完所有后续字符或者遇到了一个非数字字符。如果字符以“0x”开头且后面跟数字字符，会被解析为 16 进制整数；以“0”开头且后面跟数字字符，会被解析为 8 进制整数。下面给出一些例子：

``` javascript
var num1 = parseInt("1234blue");         // 1234
var num2 = parseInt("");                 // NaN
var num3 = parseInt("0xA");              // 10(十六进制)
var num4 = parseInt(22.5);               // 22
var num5 = parseInt("70");               // 70
var num6 = parseInt("0xf");              // 15(十六进制)
```

parseInt() 还有第二个参数：转换时使用的基数（即多少进制）。在使用这个函数时第二个参数是必需要有的。

```javascript
var num1 = parseInt(10, 2);              // 2
var num2 = parseInt(10, 8);              // 8
var num3 = parseInt(10, 10);             // 10
var num4 = parseInt(10, 16);             // 16
```

parseFloat() 解析字符串的规则与 parseInt() 类似，只不过 parseFloat() 只解析十进制值，没有第二个参数。

#### String 类型

ECMAScript 中用双引号表示的字符串和用单引号表示的字符串完全相同。但是，以双引号开头的字符串也必须以双引号结尾，而以单引号开头的字符串必须以单引号结尾。

**ECMAScript 中的字符串是不可变的，也就是说，字符串一旦创建，它们的值就不能改变。要改变某个变量保存的字符串，首先要销毁原来的字符串，然后再用另一个包含新值的字符串填充该变量。**

要把一个值转换为字符串有两种方式，第一种是使用几乎每个值都有的 toString() 方法。多数情况下，调用 toString() 方法不必传递参数，但在调用数值的 toString() 方法时，可以传递一个参数：**输出数值的基数**。默认情况下，toString() 方法以十进制格式返回数值的字符串表示。而通过传递基数，toString() 可以输出二进制、八进制、十六进制等。

``` javascript
var num = 10;
alert(num.toString());                   // "10"
alert(num.toString(2));                  // "1010"
alert(num.toString(8));                  // "12"
alert(num.toString(10));                 // "10"
alert(num.toString(16));                 // "A"
```



由于 null 和 undefined 没有 toString() 方法，所以需要使用转型函数 String() ，该函数遵循如下规则：

- 如果值有 toSting() 方法，则调用该方法并返回结果；
- 如果值是 null，则返回“null"；
- 如果值是 undefined，则返回“undefined"。

#### Object 类型

ECMAScript 中的对象其实就是一组数据和功能的集合。对象可以通过执行 new 操作符后跟要创建的对象类型的名称来创建。而创建 Object 类型的实例并为其添加属性和（或）方法，就可以创建自定义对象，如下所示：

``` javascript
var o = new Object();
```

Object 的每个实例具有下列属性和方法：

| 属性                                  | 方法                                       |
| ----------------------------------- | ---------------------------------------- |
| constructor                         | 保存着用于创建当前对象的函数。对于前面的例子而言，构造函数（constructor）就是 Object () |
| hasOwnProperty(propertyName)        | 用于检查给定的属性再当前对象实例中是否存在。其中，作为参数的属性名（propertyName）必须以字符串形式指定（例如：o.hasOwnProperty("name") ） |
| isPrototypeOf(object)               | 用于检查传入的对是否是传入对象的原型                       |
| propertyIsEnumberable(propertyName) | 用于检查给定的属性是否能够使用 for-in 语句来枚举             |
| toLocaleString()                    | 返回对象的字符串表示，该字符串与执行环境的地区对应                |
| toString()                          | 返回对象的字符串表示                               |
| valueOf()                           | 返回对象的字符串、数值或布尔值表示。通常与 toString() 方法的返回值相同 |

### 操作符

#### 一元操作符

- 在对非数值应用一元加操作符时，该操作符会像 Number() 转型函数一样对这个值执行转换。


- 一元减操作符遵循与一元加操作符相同的规则，但是会将得到的数值转换为负数。

#### 位操作符

ECMAScript 中的所有数值都以 IEEE-754 64 位格式存储，但位操作符并不直接操作 64 位的值。而是先将 64 位的值转换成 32 位 的整数，然后执行操作，最后再将结果转换回 64 位。

如果对非数值应用位操作符，会先使用 Number() 函数将该值转换为一个数值（自动完成），然后再应用位操作。得到的结果将是一个数值。

- 按位非 (~) 操作的本质：操作数的负值减 1。例如：~25 = -25 - 1。由于按位非是在数值表示的最底层执行操作，因此速度更快。
- 按位与 (&)、或 (|) 、异或 (^) 操作的本质：将两个数值的每一位对齐，进行与、或、异或操作。
- 左移 (<<) 操作符会将数值的所有位向左移动指定的位数，左移不会影响操作数的符号位，如果将 -2 向左移动 5 位，结果将是 -64，而非64。
- 有符号的右移操作符 (>>) 会将数值向右移动，但保留符号位；无符号右移操作符  (>>>) 会将数值的所有 32 位都向右移动。对正数来说，无符号右移的结果与有符号右移相同。但是对负数来说，情况就不一样了。首先，无符号右移是以 0 来填充空位，而不是像有符号右移那样以符号位的值来填充空位。其次，无符号右移操作符会把负数的二进制码当成正数的二进制码。

#### 布尔操作符

- 逻辑非 (!) 操作符可以用于将一个值转换为与其对应的布尔值。使用两个逻辑非操作符，实际上相当于模拟 Boolean() 转型函数的行为。其中，第一个逻辑非操作会基于无论什么操作数返回一个布尔值，而第二个逻辑非操作则对该布尔值求反，于是就得到了这个值真正对应的布尔值。

``` javascript
alert(!!"blue”);  //true
alert(!!NaN);     //false
```

- 逻辑与 (&&) 操作可以应用于任何类型的操作数，然而在有一个操作数不是布尔值的情况下，逻辑与操作就不一定返回布尔值。它遵循以下规则：

  - 如果第一个操作数是对象，则返回第二个操作数；
  - 如果第二个操作数是对象，则只有在第一个操作数的求值结果为 true 的情况下才会返回该对象。
  - **如果两个操作数都是对象，则返回第二个操作数；**
  - 如果有一个操作数是 null，则返回 null；
  - 如果有一个操作数是 NaN，则返回 NaN；
  - 如果有一个操作数是 undefined，则返回 undefined。

逻辑与操作属于**短路操作**，即如果第一个操作数能够决定结果，那么就不会再对第二个操作数求值。另外，**不能在逻辑与操作中使用未定义的值**。

- 逻辑或 (||) 操作符和逻辑与操作符相似，有一个操作数不是布尔值的情况下，逻辑与操作就不一定返回布尔值。它遵循以下规则：

  - 如果第一个操作数是对象，则返回第一个操作数；

  - 如果第一个操作数的求值结果为 false ，则返回第二个操作数。

  - **如果两个操作数都是对象，则返回第一个操作数；**

  - 如果有两个操作数是 null，则返回 null；

  - 如果有两个操作数是 NaN，则返回 NaN；

  - 如果有两个操作数是 undefined，则返回 undefined。

利用逻辑或的短路特性来避免为变量赋 null 或 undefined 值。例如：

```javascript
var myObject = preferredObject || backupObject;
```

变量 preferredObject 优先赋值给 myObject，变量 backupObject 负责在 preferredObject 中不包含有效值的情况下提供后备值。

#### 相等操作符

ECMAScript中的相等操作符由两个等于号（==）表示，不等操作符右叹号后跟等于号（!=）表示。 在转换不同的数据类型时，相等和不相等操作符遵循下列基本规则：

- 如果有一个操作数是布尔值，则在比较相等性之前先将其转换为数值；
- 如果一个操作数是字符串，另一个操作数是数值，在比较相等性之前先将字符串转为数值；
- 如果一个操作数是对象，另一个操作数不是，则调用对象的 valueOf() 方法，用得到的基本类型值按照前面的规则进行比较；

这两个操作符在进行比较时则要遵循下列规则。

- null 和 undefined 是相等的。
- 要比较相等性之前，不能将 null 和 undefined 转换成其他任何值。
- 如果有一个操作数是 NaN，则相等操作符返回 false，而不相等操作符返回 true。重要提示：即使两个操作数都是 NaN，结果不变。
- 如果两个操作数都是对象，则比较他们是不是同一个对象。如果两个操作数都指向同一个对象，则相等操作符返回 true；否则，返回 false。

除了相等操作符和不等操作符，还有全等操作符（===），不全等操作符（!==）。除了在比较之前不转换操作数之外，全等和不全等操作符与相等和不相等操作符没有什么区别。

由于相等和不相等操作符存在类型转换问题，而为了保持代码中数据类型的完整性，推荐使用全等和不全等操作符。

### 语句

| 语句               | 语法                                       |
| ---------------- | ---------------------------------------- |
| if语句             | if (condition) statement1 else statement2 |
| do-while语句       | do {statement} while (expression);       |
| while语句          | while(expression) statement              |
| for语句            | for (initialization; expression; post-loop-expression) statement |
| for-in语句         | for (property in expression) statement   |
| label语句          | label:statement                          |
| break和continue语句 | break语句会立即退出循环，强制继续执行循环后面的语句。而continue语句虽然也是立即退出循环，但退出循环后会从循环的顶部继续执行 |
| with语句           | with语句的作用是将代码的作用域设置到一个特定的对象中。with (expression) statement; |
| switch语句         | switch (expression) { case value: statement  break; case value: statement  break; case value: statement  break; case value: statement  break; default: statement} |

### 函数

ECMAScript 函数的基本语法：

``` javascript
function functionName(arg0, arg1,..., argN){
    statements
}
```

不管函数在定义的时候可以接收几个参数，传递参数时参数个数没有限制，参数的数据类型也没有限制。原因是 ECMAScript 函数中的参数是用一个内部数组表示的，可以通过 arguments 对象来访问函数的参数数组。

虽然可以使拥方括号语法访问 arguments 的每一个元素，但它本质上是一个对象，可以使用 length 获得实际的参数的个数。

ECMAScript 中没有函数重载的概念，只能通过检查传入函数参数的类型和数量并作出不同的反应来模拟重载。

**ECMAScript 中所有的参数传递的都是值，不可能通过 引用传递参数。**