---
title: '重读 JS 高程（Chapter05）'
date: 2017-08-26 13:17:15
tags: javascript
category: javascript
---

## Chapter05. 引用类型

引用类型的值（对象）是**引用类型**的一个**实例**。新对象是使用 new 操作符后跟一个构造函数来创建的。构造函数本身就是一个函数，只不过该函数是出于创建新对象的目的而定义的。比如下面代码：

``` javascript
var person = new Object();
```

### Object 类型

到目前为止，我们看到的大多数引用类型值都是 Object 类型的实例；而且Object 也是 ECMAScript 中最多的一个类型。创建 Object 实例的方式有两种：

- 第一种是使用 new 操作符后跟 Object 构造函数

``` javascript
var person = new Object();
person.name = "Nicholas";
person.age = 29;
```

- 另一种方式是使用对象字面量表达法

``` javascript
var person = {
    name: "Nicholas",
    age: 29
};
```

在需要向函数传入大量可选参数的情况下，对象字面量语法最合适，命名参数虽然容易处理，但是不够灵活。最好的做法是对那些必需值使用命名参数，而使用对象字面量来封装多个可选参数。

访问对象属性可以使用点表达式和方括号表达式。方括号表达式的主要优点是可以通过变量来访问属性，而且如果属性名中包含会导致语法错误的字符，或者属性名使用的是关键字和保留字，也可以使用方括号表示法。

``` javascript
person.name;
person["name"];

var propertyName = "name";
person[propertyName]

person["first name"] = "Nicholas";
```

<!-- more -->

### Array 类型

ECMAScript 中的数组与其他多数语言中的数组有着很大的区别。虽然 ECMAScript 数组与其他语言中的数组都是数据的有序列表，但与其他语言不同的是，**ECMAScript 数组的每一项可以保存任何类型的数据**。

创建数组的基本方式有两种：

- 第一种是使用 Array 构造函数，如果预先知道数组要保存的项目数量，也可以给构造函数传递该数量，而该数量会自动变成 length 属性的值；给构造函数传递值也可以创建数组。如：

```javascript
var colors = new Array();                          //空数组
var colors = new Array(20);                        //创建一个包含3项的数组
var colors = new Array("red", "blue", "green");    //创建一个包含三个字符串值的数组
var names = Array("Greg");                         //创建一个包含一个字符串值的数组
```

- 第二种是使用数组字面量表示法，如：

``` javascript
var colors = ["red", "blue", "green"];          //创建一个包含3个字符串的数组
var names = [];                                 //创建一个空数组
var values = [1,2,];                            //禁止，会创建一个包含2或3项的数组
var options = [, , , , ,];                      //禁止，会创建一个包含5或6项的数组
```

数组 length 的属性不是只读的，通过设置这个属性，可以从数组的末尾移除项或向数组中添加新项。

``` javascript
var colors = ["red", "blue", "green"];
colors.length = 2;
alert(color[2]); // undefined

var colors = ["red", "blue", "green"];
colors.length = 4;
alert(color[3]); // undefined

var colors = ["red", "blue", "green"];
colors[colors.length] = "black";
colors[colors.length] = "brown";
alert(colors); //["red", "blue", "green", "black", "brown"];

var colors = ["red", "blue", "green"];
colors[99] = "black";
alert(colors.length); // 100
```

#### 检测数组

自从 ES3 作出规定后，就出现了确定某个对象是不是数组的经典问题。对于一个网页，或者一个全局作用域而言，使用 instanceof 操作符就能得到满意结果。

``` javascript
if (value instanceof Array) {
    //对数组执行某些操作
}
```

如果网页中包含多个框架，那实际上就存在两个以上不同的全局执行环境，从而存在两个以上不同版本的 Array 构造函数。如果从一个框架向另一个框架传入一个数组，那么传入的数组与第二个框架中原生创建的数组分别具有各自不同的构造函数。为了解决这个问题，ECMAScript5 新增了Array.isArray() 方法。

``` javascript
if (Array.isArray(value)) {
    //对数组执行某些操作
}
```

#### 转换方法

- toString()：返回由数组中的每个值的字符串拼接而成的一个以逗号分隔的字符串。
- valueOf() ：返回数组。
- toLocaleString()：返回由数组中的每个值调用 toLocaleString()  返回的字符串拼接而成的一个以逗号分隔的字符串。
- join()：只接收一个参数，即用作分隔符的字符串，返回包含所有数组项以给定分隔符分隔的的字符串。

#### 栈方法和队列方法

ECMAScript 提供了一种让数组的行为类似于其他数据结构的方法。

实现栈的方法（LIFO）：

- push()：接收任意数量的参数，把它们逐个添加到数组末尾，并返回修改后的数组长度。
- pop()：从数组末尾移除最后一项，减少数组的 length 值，然后返回移除的项。

实现队列的方法（FIFO）：

- push()：同上。
- shift()：移除数组中的一个项并返回该项，同时将数组长度减 1。
- unshift()：在数组前端添加任意个项，并返回新数组的长度。

#### 重排序方法

数组中已经存在两个可以直接用来重排序的方法：

- reverse()：反转数组项的顺序。
- sort()：默认情况下使用升序排列数组项（最小值位于最前面，最大值排在最后面），为了实现排序，sort() 方法会调用每个数组项的 toString()，然后比较得到的字符串，以确定如何排序。另外，sort() 方法可以接收一个比较函数作为参数。比较函数接受两个参数，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回0，如果第一个参数应该位于第二个之后则返回一个正数。实际操作可以根据需要自行定义。如最简单的一个例子：

``` javascript
function compare(value1, value2) {
    return value1 - value2;
}
```

只需将其作为参数传递给sort()方法即可。

``` javascript
var values = [0, 1, 5, 10, 15];
values.sort(compare);
alert(values);  //0,1,10,15,5
```

#### 操作方法

ECMAScript 为操作已经包含在数组中的项提供了很多内置方法。

- concat()：该方法基于当前数组的所有项创建一个新数组。在没有参数时返回副本，接收参数会添加到数组末尾，如果接收的是数组，则数组每一项添加到末尾。如：

``` javascript
var colors = ["red", "green", "blue"];
var colors2 = colors.concat("yellow", ["black", "brown"];
                            
alert(colors);  //red, green, blue
alert(colors2);     //red, green, blue, yellow, black, brown
```

- slice()：该方法能够基于当前数组的一或多个项创建一个新数组。slice() 方法接收一个或两个参数，即返回项的起始和结束位置。在只有一个参数的情况下，该方法返回从该参数指定位置开始到当前数组末尾的所有项。如果有两个参数，该方法返回起始和结束位置之间的项——**但不包括结束位置的项**。如果 slice() 方法的参数中有一个负数，则用数组长度加上该数来确定相应的位置。**如果结束位置小于起始位置，则返回空数组**。

``` javascript
var colors = ["red", "green", "blue", "yellow", "purple"];
var colors2 = colors.slice(1);
var colors3 = colors.slice(1, 4);

alert(colors2);     //green, blue, yellow, purple
alert(colors3);     //green, blue, yellow
```

- splice()：主要用途是向数组的中部插入项，该方法始终都会返回一个数组，该数组中包含从原始数组中删除的项（如果没有删除任何项，则返回一个空数组），使用方式有3种：
  - 删除：可以删除任意数量的项， 只需指定两个参数：要删除的第一项的位置和要删除的项数。例如，splice(0, 2) 会删除数组中的前两项。
  - 插入：可以向指定位置插入任意数量的项，只需提供三个参数：起始位置、0（要删除的项数）和要插入的项。如果要插入多个项，可以再传入第四、第五，甚至任意多个项。例如 splice(2, 0, "red", "green") 会从当前数组的位置2开始插入字符串 ”red” 和 ”green”。
  - 替换：可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需提供三个参数：起始位置、要删除的项数和要插入的项。如果要插入多个项，可以再传入第四、第五，甚至任意多个项。例如 splice(2, 1, "red", "green") 会删除当前数组位置2的项，然后再从当前数组的位置2开始插入字符串 ”red” 和 ”green”。

``` javascript
var colors = ["red", "green", "blue"];
var removed = colors.splice(0, 1);      //删除第一项
alert(colors);       //green, blue
alert(removed);      //red, 返回的数组中只包含一项

removed = colors.splice(1, 0, "yellow", "orange"); //从位置1开始插入两项
alert(colors);       //green, yellow, orange, blue
alert(removed);      //返回的是一个空数组

removed = colors.splice(1, 1, "red", "purple");    //插入两项，删除一项
alert(colors);       //green, red, purple, orange, blue
alert(removed);      //yellow, 返回的数组中只包含一项
```


#### 位置方法

ES5 为数组实例添加了两个位置方法：indexOf() 和 lastIndexOf()。这两个方法都接受两个参数：要查找的项和（可选的）表示查找起点位置的索引。不同的是，indexOf() 方法从数组的开头向后查找，laseIndexOf() 方法则从数组的末尾开始向前查找。 

这两个方法都返回要查找的项在数组中的位置，没找到返回 -1。在比较第一个参数与数组中的每一项时，会使用全等操作符；也就是说，要查找的项必须严格相等。

``` javascript
var person = {name: "Nicholas"};
var people = [{name: "Nicholas"}];
var morePeople = [person];

alert(people.indexOf(person));          //-1
alert(morePeople.indexOf(person));      //0
```

#### 迭代方法

ES5 为数组定义了 5 个迭代方法。每个方法都接受两个参数：要在每一项上运行的函数和（可选的）运行该函数的作用域——影响 this 的值。传入这些方法中的函数会接收三个参数：数组项的值、该想在数组中的位置和数组对象本身。根据使用的方法不同，这个函数执行后的返回值可能会也可能不会影响方法的返回值。以下是这5个迭代方法的作用。

- every()：对数组中的每一项运行给定函数，如果该函数对每一项都返回true，则返回true。
- some()：对数组中的每一项运行给定函数，如果该函数对任一项返回true，则返回true。
- filter()：对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组。
- forEach()：对数组中的每一项运行给定函数。**这个方法没有返回值。**
- map()：对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。

**以上方法都不会修改数组中的包含的值。**

其中，every() 和 filter() 方法最相似。

``` javascript
var numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];

var everyResult = numbers.every(function(item, index, array) {
    return (item > 2);
});
alert(everyResult);     //false

var someResult = numbers.some(function(item, index, array) {
    return (item > 2);
});
alert(someResult);      //true
```

filter() 简称为滤波器，作用也有点类似滤波器。可以用来过滤出符合条件项。

``` javascript
var numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];

var filterResult = numbers.filter(function(item, index, array) {
    return (item > 2);
});
alert(filterResult);     //[3, 4, 5, 4, 3]
```

map() 可以用来创建包含的项与另一个数组一一对应的项。

``` javascript
var numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];

var mapResult = numbers.filter(function(item, index, array) {
    return item * 2;
});
alert(mapResult);        //[2, 4, 6, 8, 10, 8, 6, 4, 2]
```

#### 归并方法

ES5 还新增了两个归并数组的方法：reduce() 和 reduceRight()。这两个方法都会迭代数组的所有项，然后构建一个最终返回的值。这两个方法都接收两个参数：一个在每一项上调用的函数和（可选的）作为归并基础的初始值。传给 reduce() 和 reduceRight() 的函数接收4个参数：前一个值、当前值、项的索引和数组对象。这个函数返回的任何值都会作为第一个参数自动传给下一项。第一次迭代发生在数组的第二项上，因此第一个参数是数组的第一项，第二个参数就是数组的第二项。 使用reduce()方法可以执行数组中所有值求和操作：

``` javascript
var values = [1, 2, 3, 4, 5];

var sum = values.reduce(function(prev, cur, index, array) {
    return prev + cur;
});
alert(sum);     //15
```

### Date类型

ECMAScript 中的 Date 类型是在早期 Java 中的 java.util.Date 类基础上构建的。为此，Date 类型使用自UTC（Coordinated Universal Time，国际协调时间）1970 年 1 月 1 日午夜（零时）开始经过的毫秒数来保存日期。

创建日期对象，使用 new 操作符和 Date 构造函数即可。

``` javascript
var now = new Date();  // 无参数情况下，获取系统当前日期和时间
```

如果想根据特定日期时间创建日期对象，必须传入该日期毫秒数。为简化计算过程，ECMPScript 提供两个方法：Date.parse() 和 Date.UTC()。 

- Date.parse() 接收一个表示日期的字符串。ECMA-262 没有定义 Date.parse() 应该支持哪种日期格式，因此这个方法的行为因实现而异，而且通过时因地区而异。将地区设置为美国的浏览器通常都接受下列日期格式：

  - “月/日/年“，如 6/13/2004；
  - “英文月名 日，年”，如 January 12,2004；
  - “英文星期几 英文月名 日 年 时:分:秒 时区”，如 Tue May 25 2004 00:00:00 GMT-0700。
  - ISO 8601 扩展格式 YYYY-MM-DDTHH:mm:ss.sssZ（例如 2004-05-25T00:00:00）。只有兼容 ES5 的实现支持这种格式。

``` javascript
var someDate = new Date(Date.parse("May 25, 2004"));
```

如果传入 Date.parse() 方法的字符串不能表示日期，那么它会返回 NaN。实际上，如果直接将字符串传递给Date构造函数，也会在后台调用 Date.parse()。

- Date.UTC() 方法同样也返回表示日期的毫秒数。Date.UTC() 的参数分别是年份、基于 0 的月份（0 - 11）、月中的哪一天、小时数（0 - 23）、分钟、秒以及毫秒数。前两个参数是必须的，其他默认为 0。

``` javascript
//GMT 时间 2000 年 1 月 1 日午夜零时
var y2k = new Date(Date.UTC(2000, 0));

//GMT 时间 2005 年 5 月 5 日下午 5:55:55
var allFives = new Date(Date.UTC(2005, 4, 5, 17, 55, 55));
```

- Date.now() 是 ES5 添加的方法，返回表示调用这个方法时的日期和时间的毫秒数。这个方法简化了使用 Date 对象分析代码的工作。例如：

``` javascript
//取得开始时间
var start = Date.now();  // Date.now() === +new Date()
//调用函数
doSomething();
//取得停止时间
var stop = Date.now(),
    result = stop -start;
```

#### 继承的方法

与其他引用类型一样，日期类型继承了 toLocaleString()、toString() 和 valueOf() 方法。Date 类型的 toLocaleString() 方法会按照与浏览器设置的地区相适应的格式返回日期和时间。valueOf() 不返回字符串，而是返回日期的毫秒表示，目的是方便日期值的比较。

#### 日期格式化方法

Date类型还有一些专门用于将日期格式化为字符串的方法，如下：

- toDateString()：以特定于实现的格式显示星期几、月、日和年；
- toTimeString()：以特定于时间的格式显示时、分、秒和时区；
- toLocaleDateString()：以特定于地区的格式显示星期几、月、日和年；
- toLocaleTimeString()：以特定于实现的格式显示时、分、秒；
- toUTCString()：以特定于实现的格式显示完整的UTC日期。

其他相关日期/时间组件方法，可参考w3cschool的JavaScript Date对象

### RegExp对象

ECMAScript 通过 RegExp 类型来支持正则表达式，使用下面类似 Perl 的语法，就可以创建一个正则表达式。

``` javascript
var expression = /pattern/flags ;
```

其中的模式（pattern）部分可以是任何简单或复杂的正则表达式，可以包含字符类、限定符、分组、向前查找以及反向引用。每个正则表达式都可带有一或多个标志（flags），用以表明正则表达式的行为。正则表达式匹配模式支持下列 3 个标志。

- g：表示全局模式（global），即模式将被用于所有字符串，而不是在发现第一个匹配项后立即停止；
- i：表示不区分大小写（case-insensitive）模式，即在确定匹配项是忽略模式与字符串的大小写；
- m：表示多行（multiline）模式，即在到达一行文本末尾时还会继续查找下一行中是否存在与模式匹配的项。

与其他语言中的正则表达式类似，模式中使用的所有元字符都必须**转义**。正则表达式中的元字符包括：

``` javascript
\ ^ $ | ? * + . () [] {}
```

下面给出几个例子：

```javascript
var pattern1 = /[bc]at/i;       // 匹配第一个"bat"或"cat"，不区分大小写
var pattern2 = /\[bc\]at/i;     // 匹配第一个"[bc]at"，不区分大小写
var pattern3 = /.at/gi;         // 匹配所有以"at"结尾的3个字符的组合，不区分大小写
var pattern4 = /\.at/gi;        // 匹配所有".at"，不区分大小写
```

除了用字面量形式来定义正则表达式，另一种创建正则表达式的方式是使用 RegExp 构造函数，它接收两个参数：一个是要匹配的字符串模式（pattern），另一个是可选的标志字符串（flag）。由于 RegExp 构造函数的模式参数是字符串，所以在某些情况下要对字符进行**双重转义**，**所有元字符都必须双重转义**。如下：

``` javascript
var pattern1 = /[bc]at/i;       // 匹配第一个"bat"或"cat"，不区分大小写
var pattern2 = new RegExp("[bc]at", "i");     // 与 pattern1 相同 
var pattern3 = /\[bc\]at/i;     // 匹配第一个"[bc]at"，不区分大小写
var pattern4 = new RegExp("\\[bc\\]at", "i"); // 与 pattern3 相同，不过元字符要进行双重转义
```

#### RegExp 实例属性

- global：布尔值，表示是否设置了 g 标志。
- ignoreCase：布尔值，表示是否设置了 i 标志。
- lastIndex：整数，表示开始搜索下一个匹配项的字符位置，从 0 算起。
- multiline：布尔值，表示是否设置了 m 标志。
- source：正则表达式的字符串表示，按照字面量形式而非传入构造函数中的字符串模式返回。

#### RegExp 实例方法

- exec()：该方法专门为捕获组设计。接受一个参数，即**要应用模式的字符串**，然后返回**包含第一个匹配项信息的数组**，或者**在没有匹配项的情况下**返回 **null**。返回的数组虽然是 Array 实例，但包含两个额外的属性：index 和 input。其中，index 表示匹配项在字符串中的位置，而 input 表示应用正则表达式的字符串。在数组中，**第一项是与整个模式匹配的字符串**，其他项是与模式中的捕获组匹配的字符串（若没有捕获组，则数组只包含一项）。
- test()：接受一个**字符串**参数。在模式与该参数匹配的情况下返回 **true**；否则，返回 **false**。
- toLocaleString(), toString(), valueOf()：toLocaleString() 和 toString() 都会返回正则表达式的字面量；valueOf() 方法返回正则表达式本身。

#### RegExp 构造函数属性

|     长属性名     | 短属性名 |             说明             |
| :----------: | :--: | :------------------------: |
|    input     |  $_  |        最近一次要匹配的字符串         |
|  lastMatch   |  $&  |          最近一次的匹配项          |
|  lastParen   |  $+  |         最近一次匹配的捕获组         |
| leftContext  |  $`  | input 字符串中 lastMatch 之前的文本 |
| rightContext |  $'  | input 字符串中 lastMatch 之后的文本 |

例子：

``` javascript
var text = "this has been a short summer";
var pattern = /(.)hort/g;  
if (pattern.test(text)){
    alert(RegExp.input);               //this has been a short summer
    alert(RegExp.leftContext);         //this has been a            
    alert(RegExp.rightContext);        //summer
    alert(RegExp.lastMatch);           //short
    alert(RegExp.lastParen);           //s
}

var text = "this has been a short summer";
var pattern = /(.)hort/g;       
if (pattern.test(text)){
    alert(RegExp.$_);               //this has been a short summer
    alert(RegExp["$`"]);            //this has been a            
    alert(RegExp["$'"]);            // summer
    alert(RegExp["$&"]);            //short
    alert(RegExp["$+"]);            //s
}
```

除了上面介绍的几个属性之外，还有多达 9 个用于存储捕获组的构造函数属性。访问这些属性的语法是`RegExp.$1`-`RegExp.$9`。

``` javascript
var text = "this has been a short summer";
var pattern = /(..)or(.)/g;

if (pattern.test(text)){
    alert(RegExp.$1);       //sh
    alert(RegExp.$2);       //t
}
```

### Function类型

**函数实际上是对象**，**每个函数都是Function类型的实例**，而且和其他引用类型一样具有属性和方法。由于函数是对象，因此**函数名实际上是一个指向函数对象的指针，不会与某个函数绑定**。换句话说，一个数可能会有多个名字，如下面例子所示：

``` javascript
function sum(num1, num2){
    return num1 + num2;
}        
alert(sum(10,10));    //20

var anotherSum = sum;        
alert(anotherSum(10,10));  //20

sum = null;        
alert(anotherSum(10,10));  //20
```

#### 没有重载

由于函数是对象，函数名是一个指向函数对象的指针，不会与某个函数绑定，这有助于理解 ECMAScript 中为何没有函数重载的概念

``` javascript
//函数声明
function addSomeNumber(num) {
    return num + 100;
}

function addSomeNumber(num) {
    return num + 200;
}

var result = addSomeNumber(100);    //300
```

该例声明两个同名函数，结果是后面的函数覆盖了前面的。该例与下面的代码几乎没有区别：

``` javascript
var addSomeNumber = function (num) {
    return num + 100;
};

addSomeNumber = function (num) {
    return num + 200;
};

var result = addSomeNumber(100);        //300
```

#### 函数声明与函数表达式

解析器在向执行环境中加载数据时，对函数声明和函数表达式并非一视同仁。<u>解析器会率先读取函数声明，并使其执行代码之前可用（可以访问）；至于函数表达式，则必须等到解析器执行到它所在的代码行，才会真正被解析执行。</u>例子：

``` javascript
alert(sum(10, 10));
function sum(num1, num2) {
    return num1 + num2;
}
```

上例能够正常运行，因为在代码执行前，解析器就已经通过一个名为**函数声明提升**（function declaration hoisting）的过程，读取并将函数声明添加到执行环境中。对代码求值时，JavaScript 引擎在第一遍会声明函数并将他们放到源代码树的顶部。所以，即使声明函数的代码在调用它的带码后面，JavaScript 引擎也能把函数声明提升到顶部。而下例则不能运行，因为函数位于一个初始化语句中，而不是一个函数声明：

``` javascript
alert(sum(10, 10));
var sum = function(num1, num2) {
    return num1 + num2;
}
```

除了什么时候可以通过变量访问函数这一点区别外，函数声明与函数表达式的语法其实是等价的。

#### 作为值的函数 

因为 ECMAScript 中的函数名本身就是变量，所以函数也可以作为值来使用。

- 可以像传递参数一样把一个函数传递给另一个函数。

``` javascript
function callSomeFunction (someFunction, someArgument) {
    return somFunction (someArgument);
}

function add10 (num) {
    return num + 10;
}

var result1 = callSomeFunction (add10, 10);
alert(result1);     //20

function getGreeting (name) {
    return "Hello, " + name;
}

var result2 = callSomeFunction (getGreeting, "Nicholas");
alert(result2);     //"Hello, Nicholas"
```

- 可以从一个函数返回另一个函数

``` javascript
function createComparisonFunction (propertyName) {
    return function(object1, object2) {
        return object1[propertyName] - object2[propertyName];
    };
}
```

该函数接收一个属性名，然后根据这个属性名来创建一个比较函数。使用如下：

``` javascript
var data = [{name: "Zachary", age: 28}, {name: "Nicholas", age: 29}];

data.sort(createComparisonFunction("name"));
alert(data[0].name);        //Nicholas

data.sort(createComparisonFunction("age"));
alert(data[0].name);        //Zachary
```

#### 函数内部属性 

在函数内部，有两个特殊的对象：arguments 和 this。

- arguments的主要用途是保存函数参数，这个对象还有一个名叫 callee 的属性，该属性是一个指针，指向拥有这个arguments对象的函数。它可以消除函数与函数名的紧耦合现象，如下：

``` javascript
function factorial(num) {
    if (num <= 1) {
        return 1;
    } else {
        return num * factorial(num - 1)
    }
}

//使用arguments.callee替代函数名，消除耦合
function factorial(num) {
    if (num <= 1) {
        return 1;
    } else {
        return num * arguments.callee(num - 1)
    }
}
```

这样，无论引用函数时使用的是什么名字，都可以保证正常完成递归调用。如下：

``` javascript
var trueFactorial = factorial;

factorial = function () {
    return 0;
};

alert(trueFactiorial(5));           //120
alert(factorial(5));                //0
```

- 另一个特殊对象是 this，this 引用的是**函数据以执行的环境对象**——或者也可以说是 this 值（当在网页的全局作用域中调用函数时，this对象引用的就是window）。例如：

``` javascript
window.color = "red";
var o = { color: "blue"};

function sayColor() {
    alert(this.color);
}

sayColor();             //"red"

o.sayColor = sayColor;
o.sayColor();           //"blue"
```

函数 sayColor() 在全局作用域中定义，在调用该函数之前，this 值并不确定。当在全局作用域中调用时，this 引用的是全局对象 window；当把这个函数赋给对象 o 并调用 o.sayColor() 时，this 引用的是对象 o。

- ES5 也规范化了另一个**函数对象的属性**：caller。这个属性保存着调用当前函数的函数的引用，如果是在全局域中调用当前函数，它的值为null。例如：


``` javascript
function outer(){
    inner();
}

function inner(){
    alert(inner.caller);
}

outer();              // 显示 outer 源代码
```

#### 函数属性和方法

ECMAScript 中函数是对象，因此也有属性和方法。

每个函数都包含两个属性：length 和 prototype。

- length：表示函数希望接收的命名参数的个数。
- prototype：对于 ECMAScript 中的引用类型来说，prototype 是保存他们所有实例方法的真正所在。 诸如 toString() 和 valueOf() 等方法实际上都保存在 prototype 名下，只不过是通过各自对象的实例访问它们罢了。**在创建自定义引用类型以及实现继承时，prototype 属性的作用极为重要。**在 ES5 中，prototype 属性是不可枚举的，因此使用 for-in 无法发现。

每个函数都包含两个非继承而来的方法：apply()和call()。这两个方法的用途都是在特定的作用域中调用函数，实际上等于**设置函数体内 this 对象的值**。 

- apply()方法接收两个参数，一个是在其中运行函数的作用域，另一个是参数数组。其中，第二个参数可以是 Array 的实例，也可以是 arguments 对象。例如：

``` javascript
function sum(num1, num2) {
    return num1 + num2;
}
function callSum1(num1, num2) {
    return sum.apply(this, arguments);          //传入arguments对象
}
function callSum2(num1, num2) {
    return sum.apply(this, [num1, num2]);       //传入数组
}

alert(callSum1(10, 10));        //20
alert(callSum2(10, 10));        //20
```

- call() 方法与 apply() 方法的作用相同，他们的区别仅在于接收参数的方式不同。对于 call() 方法而言，第一个参数是 this 值没有变化，变化的是其余参数都直接传递给函数。所以在使用 call() 方法时，传递给函数的参数必须逐个列举出来。例如：

``` javascript
function sum(num1, num2) {
    return num1 + num2;
}
function callSum(num1, num2) {
    return sum.call(this, num1, num2);
}

alert(callSum(10, 10));     //20
```

事实上，传递参数并非 apply() 和 call() 真正用武之地，它们真正强大的地方是能够扩充函数赖以运行的作用域。例如：
``` javascript
window.color = "red";
var o = { color: "blue" };

function sayColor() {
    alert(this.color);
}

sayColor();                     //red
sayColor.call(this);            //red
sayColor.call(window);          //red
sayColor.call(o);               //blue
```

ECMAScript 5还定义了一个方法：bind()。这个方法会创建一个函数的实例，其this值会被绑定到传给bind() 函数的值。例如：

``` javascript
window.color = "red";
var o = { color: "blue" };

function sayColor() {
    alert(this.color);
}
var objectSayColor = sayColor.bind(o);
objectSayColor();           //blue
```

### 基本包装类型

为了便于操作基本类型值，ECMAScript 也提供了 3 个特殊的引用类型：Boolean、Number 和 String。<u>每当读取一个基本类型值的时候，后台就会创建一个对应的基本包装类型的对象，从而让我们能够调用一些方法来操作这些数据。</u>下面以 String 类型为例：

``` javascript
var s1 = "some text";
var s2 = s1.substring(2);
```

在第二行代码访问 s1 时，访问过程处于一种**读取模式**，在**读取模式**中访问字符串时，后台会自动完成下列处理：

1. 创建 String 类型的一个实例；
2. 在实例上调用指定的方法；
3. 销毁这个实例。

以上三个步骤可以等效成下列代码：

``` javascript
var tmp = new String("some text");
var s2 = s1.substring(2);
tmp = null;
```

经过此番树立，基本的字符串值就变得跟对象一样了。以上三个步骤同样适用于 Boolean 和 Number 类型对应的布尔值和数字值。 

**引用类型与基本包装类型主要区别就是对象的生存期。**使用 new 操作符创建的引用类型的实例，在执行流离开当前引用域之前一直都保存在内存中。而自动创建的基本包装类型的对象，则只存在一行代码的执行瞬间，然后立即销毁。**这意味着不能在运行时为基本类型值添加属性和方法。**

当然，可以调用 Boolean、Number 和 String 来创建基本包装类型的对象。但是，对基本包装类型的实例调用 typeof 会返回 "object"，而且所有基本包装类型的对象都会被转换为布尔值 true。Object 构造函数也会像工厂方法一样，根据传入值的类型返回相应基本包装类型的实例。例如：

``` javascript
var obj = new Object("some text");
alert(obj instanceof String);            //true;    
```

要注意的是，使用 new 调用基本包装类型的构造函数，与直接调用同名的转型函数是不一样的。例如：

``` javascript
var value = "25";
var number = Number(value);  //转型函数
alert(typeof number);        //"number"

var obj = new Number(value); //构造函数
alert(typeof obj);           //"object"
```

#### Boolean类型

Boolean类型是与布尔值对应的引用类型，**建议永远不要使用Boolean对象**。

#### Number类型

除了继承的方法外，Number类型还提供了一些用于将数值格式化为字符串的方。 
- toFixed()：按指定小数位返回数值的字符串表示，会对数值进行四舍五入。
- toExponential()：返回以指数表示法表示的数值的字符串形式，可以传入一个参数，指定输出结果中的小数位数。 
- toPrecision()：返回固定大小格式，也可能返回指数格式，具体规则是看哪种格式比较合适。该方法接收一个参数指定表示数值的所有数字的位数。

#### String类型

String 类型的每个实例都有一个 length 属性，表示字符串中包含多少个字符。

String 类型提供了很多方法，用于辅助完成对 ECMAScript 中字符串的解析和操作。

##### 字符方法

两个用于访问字符串中特定字符的方法是：charAt() 和 charCodeAt()。两个方法都接收一个参数，即**基于 0 的字符位置**。charAt() 返回**指定位置的字符**，charCodeAt() 返回**指定位置的字符编码**。ES5加入了方括号（[]）语法，等同于 charAt()。

##### 字符串操作方法

- concat()：用于将一多个字符串拼接起来，**接受任意多个参数**，返回**拼接得到的<u>新</u>字符串**。不过实践中使用更多的是加号操作符（+）拼接字符串。
- slice()、substr() 和 substring()： 这三个方法基于字符串创建新的字符串。它们都会返回被操作字符串的一个子字符串，而且也都接受一或两个参数。第一个参数**指定子字符串的开始位置**；。slice() 和 substr() 的第二个参数**指定的是子字符串最后一个字符后面的位置**；而 substr() 的第二个参数**指定的是返回的字符数**。如果没有给这些方法传递第二个参数，**则将字符串的长度作为结束位置**。

``` javascript
var StringObject = "hello world";
alert(StringObject.slice(3));              //输出 "lo world"
alert(StringObject.substring(3));          //输出 "lo world"
alert(StringObject.substr(3));             //输出 "lo world"
alert(StringObject.slice(3, 7));           //输出 "lo w"
alert(StringObject.substring(3, 7));       //输出 "lo w"
alert(StringObject.substr(3, 7));          //输出 "lo worl"
```

当传递的参数是负值时，它们的行为就不尽相同了。其中，slice() 方法**会将传入的赋值与字符串长度相加**，substr() 方法**将负的第一个参数加上字符串的长度，而将负的第二个参数转换为 0**，substring() 方法**会将所有负值参数都转换为 0**。

``` javascript
var StringObject = "hello world";
alert(StringObject.slice(-3));             //输出 "rld"
alert(StringObject.substring(-3));         //输出 "hello world"
alert(StringObject.substr(-3));            //输出 "rld"
alert(StringObject.slice(3, -4));          //输出 "lo w"
alert(StringObject.substring(3, -4));      //输出 "hel"
alert(StringObject.substr(3, -4));         //输出 ""
```

注意，slice() 和 substr() 方法中第一个参数的值不能大于第二参数的值，否则会返回空字符串 "" ；而 substring() 会将两个参数中较小的数作为开始位置。

##### 字符串位置方法

有两个可以从字符串中查找子字符串的方法：indexOf() 和 lastIndexOf()。两个方法都是**从一个字符串中搜索给定的子字符串**，然后返回**子字符串的位置**。前者从前往后搜索，后者从后往前搜索。 两个方法都可接收可选的第二个参数，**表明从字符串哪个位置开始搜索**。 在使用第二个参数的情况下，可以通过循环调用 indexOf() 或 lastIndexOf() 来找到所有匹配的子字符串。如下：

``` javascript
var stringValue = "Lorem ipsum dolor sit amet, consectetur adipisicing elit";
var positions = [];
var pos = stringValue.indexOf("e");

while (pos > -1) {
    positions.push(pos);
    pos = stringValue.indexOf("e", pos + 1);
}

alert(positions);       //"3, 24, 32, 35, 52"
```

##### trim() 方法

该方法创建一个字符串的副本，删除前置及后缀的所有空格，然后返回结果（原始字符串不变）。

##### 字符串大小写转换方法

- toLowerCase()
- toLocaleLowerCase()（针对特定地区的语言）
- toUpperCase()
- toLocaleUpperCase()（针对特定地区的语言）

##### 字符串的模式匹配方法

- match()：在字符串上调用这个方法，本质上与调用 RegExp 的 exec() 方法相同。match() 只接受一个参数，正则表达式或者 RegExp 对象。和 exec() 方法一样，match() 方法会返回一个数组：数组的第一项是与整个模式匹配的字符串，之后的每一项（如果有）保存着与正则表达式中的捕获组匹配的字符串。

``` javascript
var text = "cat, bat, sat, fat";
var pattern = /.at/;

//与pattern.exec(text)相同
var matches = text.match(pattern);
alert(matches.index);           //0
alert(matches[0]);              //"cat"
alert(pattern.lastIndex);       //0
```

- search()：参数与 match() 方法相同，**返回字符串中第一个匹配项的索引**，**如果没有则返回 -1**。

``` javascript
var text  = "cat, bat, sat, fat";
var pos = text.search(/at/);
alert(pos);     // 1
```

- replace()： 接受两个参数，第一个参数可以是一个 RegExp 对象或者一个字符串（这个字符串不会被转换为正则表达式），第二个参数可以是一个字符串或者一个函数。如果第一个参数是字符串，那么只会替换第一个字符串，想替换所有字符串必须提供一个正则表达式，并且要指定全局（g）标志。

``` javascript
var text = "cat, bat, sat, fat";
var result = text.replace("at", "ond");
alert(result);   //"cond, bat, sat, fat"

result = text.replace(/at/g, "ond");
alert(result);   //"cond, bond, sond, fond"
```

如果第二个参数是字符串，那么还可以使用一些特殊的字符序列，将正则表达式操作得到的值插入到结果字符串中。下表列出了 ECMAScript 提供的这些特殊的字符序列：

| 字符序列  | 替换文本                                     |
| ----- | ---------------------------------------- |
| `$$ ` | $                                        |
| `$&`  | 匹配整个模式的子字符串。与 RegExp.lastMatch 的值相同      |
| `$'`  | 匹配的子字符串之前的子字符串。与 RegExp.leftContext 的值相同 |
| $`    | 匹配的子字符串之后的子字符串。与 RegExp.rightContext 的值相同 |
| `$n`  | 匹配第 n 个捕获组的子字符组，其中 nn 等于0~9。例如，`$1` 是匹配第一个捕获组的子字符串。如果正则表达式中没有定义捕获组，则使用空字符串 |
| `$nn` | 匹配第 nn 个捕获组的子字符串，其中 nn 等于01~99。例如，`$01` 是匹配第一个捕获组的子字符串。如果正则表达式中没有定义捕获组，则使用空字符串 |

通过这些特殊的字符序列，可以使用最近一次匹配结果中的内容，如下：

``` javascript
var text = "cat, bat, sat, fat";
result = text.replace(/(.at)/g, "word ($1)");
alert(result);          //word (cat), word (bat), word(sat), word (fat)
```

**replace() 方法的第二个参数也可以是一个函数。**在只有一个匹配项（即与模式匹配的字符串）的情况下，会向这个函数传递 3 个参数：**模式的匹配项**、**模式匹配项在字符串中的位置**和**原始字符串**。在正则表达式定义了多个捕获组的情况下，传递给函数的参数一次是模式的匹配项、第一个捕获组的匹配项、第二个捕获组的匹配项……，**但最后两个参数仍然分别是模式的匹配项在字符串中的位置和原始字符串**。这个函数应该**返回一个字符串，表示应该被替换的匹配项**。使用函数作为 replace() 方法的第二个参数可以实现更加精细的替换操作，如下例：

``` javascript
function htmlEscape(text) {
    return text.replace(/[<>"&]/g, function(match, pos, originalText) {
        switch (match) {
            case "<":
                return "&lt;";
            case ">":
                return "&gt;";
            case "&":
                return "&amp;";
            case "\"":
                return "&quot;"
        }
    });
}
alert(htmlEscape("<p class=\"greeting\">Hello world!</p>"));
//&lt;p class=&quot;greeting&quot;&gt;Hello world!&lt;/p&gt;
```

- split()： 该方法可以**基于指定的分隔符将一个字符串分割成多个子字符串，并将结果放在一个数组中**。分隔符**可以是字符串，也可以是一个 RegExp 对象**（这个方法不会将字符串看成正则表达式）。split() 方法**可以接受可选的第二个参数，用于指定数组的大小，以便确保返回的数组不会超过既定大小**。

``` javascript
var colorText = "red, blue, green, yellow";
var colors1 = colorText.split(",");         //["red", "blue", "green", "yellow"]
var colors2 = colorText.split(",", 2);      //["red", "blue"]
var colors3 = colorText.split(/[^\,]+/);    //["", ",", ",", ",", ""]
```

##### localeCompare() 方法

该方法比较两个字符串，并返回下列值中的一个：

- 如果字符串在字母表中应该排在字符串参数之前，则返回一个负数（大多数情况下是 -1，具体的值要视实现而定）；
- 如果字符串等于字符串参数，则返回0；
- 如果字符串在字母表中应该排在字符串参数之后，则返回一个正数（大多数情况下是 1，具体的值同样要使实现而定）。

``` javascript
var stringValue = "yellow";
alert(stringValue.localeCompare("brick"));      //1
alert(stringValue.localeCompare("yellow"));     //0
alert(stringValue.localeCompare("zoo"));        //-1
```

##### fromCharCode() 方法

String 构造函数本身还有一个**静态方法**：fromCharCode()。这个方法的任务是**接收一或多个字符编码，然后将它们转换成一个字符串**。从本质上讲，这个方法与实例方法 charCodeAt() 执行的是相反的操作。

``` javascript
alert(String.fromCharCode(104, 101, 108, 108, 111); //"hello"
```

###  单体内置对象

ECMA-262 对内置对象的定义是：“由 ECMAScript 实现提供的、不依赖宿主环境的对象，这些对象在 ECMAScript 程序执行之前就已经存在了。”意思就是说，开发人员不必显式地实例化内置对象，因为它们已经实例化了。除了前面介绍的大多数内置对象，例如 Object、Array 和 String。ECMA-262 还定义了两个单体内置对象：Global 和 Math。

#### Global对象

Global（全局）对象可以说是 ECMAScript 中最特别的一个对象了，因为不管从什么角度看，这个对象都是不存在的。不属于任何其他对象的属性和方法，最终都是它的属性和方法。**事实上，没有全局变量或全局函数；所有在全局作用域中定义的属性和函数，都是Global对象的属性。**前面介绍过的isNaN()、isFinite()、parseInt() 以及 parseFloat()，实际上都是Global对象的方法。除此之外，Global对象还包含其他一些方法。

##### URI编码方法 

Global 对象的 encodeURI() 和 encodeURIComponent() 方法可以对 URI（Uniform Resource Indentifiers，通用资源标识符）进行编码，以便发送给浏览器。与这两个方法相对的两个方法分别是 decodeURI() 和 decodeURIComponent()。其中，decodeURI() 只能对使用 encodeURI() 替换的字符进行解码。如下：

``` javascript
var uri = "http://www.wrox.com/illegal value.htm#start";

//"http://www.wrox.com/illegal%20value.htm#start"
alert(encodeURI(uri));

//"http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.htm%23start"
alert(encodeURIComponent(uri));

var uri = "http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.htm%23start"

//"http%3A%2F%2Fwww.wrox.com%2Fillegal value.htm%23start"
alert(decodeURI(uri));

//"http://www.wrox.com/illegal value.htm#start"
alert(decodeURIComponent(uri));
```

通过上例可以看出 encodeURI() 和 encodeURIComponent() 的区别，encodeURI() 不会对本身属于 URI 的特殊字符进行编码，例如冒号、正斜杠、问号和井号；encodeURIComponent() 则会对它发现的任何非标准字符进行编码。

##### eval() 方法

**只接受一个参数，即要执行的字符串。**当解析器发现代码中调用 eval() 方法时，它会将传入的参数当作实际的 ECMAScript 语句来解析，然后把执行结果插入到原位置。通过 eval() 执行的代码被认为是包含该次调用的执行环境的一部分，因此被执行的代码具有与该执行环境相同的作用域链。这意味着通过 eval() 执行的代码可以引用在包含环境中定义的变量，例如：

``` javascript
var msg = "hello world";
eval("alert(msg)");     //"hello world"
```

在 eval() 中创建的任何变量或函数都不会被提升，因为在解析代码的时候，它们被包含在一个字符串中；它们旨在 eval() 执行的时候创建。 严格模式下，在外部访问不到 eval() 中创建的任何变量或函数。

##### Global对象的属性

| 属性        | 说明           | 属性             | 说明                 |
| --------- | ------------ | -------------- | ------------------ |
| undefined | 特殊值undefined | Date           | 构造函数Date           |
| NaN       | 特殊值NaN       | RegExp         | 构造函数RegExp         |
| Infinity  | 特殊值Infinity  | Error          | 构造函数Error          |
| Object    | 构造函数Object   | EvalError      | 构造函数EvalError      |
| Array     | 构造函数Array    | RangeError     | 构造函数RangeError     |
| Function  | 构造函数Function | ReferenceError | 构造函数ReferenceError |
| Boolean   | 构造函数Boolean  | SyntaxError    | 构造函数SyntaxError    |
| String    | 构造函数String   | TypeError      | 构造函数TypeError      |
| Number    | 构造函数Number   | URIError       | 构造函数URIError       |

ES5 明确禁止给 undefined、NaN 和 Infinity 赋值，这样做即使在非严格模式下也会导致错误。

##### window对象 

ECMAScript 虽然没有指出如何直接访问 Global 对象，但 Web 浏览器都是将这个全局对象作为 window 对象的一部分加以实现的。因此，在全局作用域中声明的所有变量和函数，就都成为了 window 对象的属性。

另一种取得Global对象的方法是使用以下代码：

``` javascript
var global = function() {
    return this;
}();
```

#### Math 对象

ECMAScript还为保存数学公式和信息提供了一个公共位置，即Math对象。

##### Math对象的属性

| 属性           | 说明                 |
| ------------ | ------------------ |
| Math.E       | 自然对数的底数，即常量e的值     |
| Math.LN10    | 10的自然对数            |
| Math.LN2     | 2的自然对数             |
| Math.LOG2E   | 以2为底e的对数           |
| Math.LOG10E  | 以10为底e的对数          |
| Math.PI      | π的值                |
| Math.SQRT1_2 | 1/2的平方根（即2的平方根的倒数） |
| Math.SQRT2   | 2的平方根              |

##### Math对象的方法 

- Math.min() 和 Math.max() 确定一组数值中的最小值和最大值，可接受任意多个数值参数。例如：

``` javascript
var max = Math.max(3, 54, 32, 16);
var min = Math.min(3, 54, 32, 16);
alert(max);     //54
alert(min);     //3

//接收数组参数，技巧在于把Math对象作为apply()的第一个参数，从而正确设置this值，然后将任何数组作为第二个参数。
var values = [1, 2, 3, 4, 5, 6, 7, 8];
var max = Math.max.apply(Math, values);
```

- Math.ceil()、Math.floor() 和 Math.round() 可以将小数值舍入为整数。
  - Math.ceil() 向上舍入为最接近整数
  - Math.floor() 向下舍入为最接近整数
  - Math.round() 四舍五入为最接近整数
- Math.random() 方法返回 [0, 1) 的一个随机数。利用公式——值 = Math.floor(Math.random() * 可能值的总数 + 第一个可能的值），就可以在某个整数范围内随机选择一个值。例如：

``` javascript
//选择一个介于2到10之间的值
var num = Math.floor(Math.random() * 9 + 2);

//以下函数可以直接指定随机范围（整数）：
function selectFrom(lowerValue, upperValue) {
    var choices = upperValue - lowerValue +1;
    return Math.floor(Math.random() * choices + lowerValue);
}

var num = selectFrom(2, 10);
alert(num);     //介于2和10之间（包括2和10）的一个数值
```
##### 其他方法

| 方法                   | 说明             | 方法               | 说明         |
| -------------------- | -------------- | ---------------- | ---------- |
| Math.abs(num)        | 返回num的绝对值      | Math.asin(x)     | 返回x的反正弦值   |
| Math.exp(num)        | 返回Math.E的num次幂 | Math.atan(x)     | 返回x的反正切值   |
| Math.log(num)        | 返回num的自然对数     | Math.atan2(y, x) | 返回y/x的反正切值 |
| Math.pow(num, power) | 返回num的power次幂  | Math.cos(x)      | 返回x的余弦值    |
| Math.sqrt(num)       | 返回num的平方根      | Math.sin(x)      | 返回x的正弦值    |
| Math.acos(x)         | 返回x的反余弦值       | Math.tan(x)      | 返回x的正切值    |

