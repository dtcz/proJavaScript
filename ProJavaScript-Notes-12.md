---
title: 重读 JS 高程（Chapter15）
tags: javascript
category: javascript
date: 2017-09-20 12:45:30
---


## Chapter15. 使用 Canvas 绘图

与浏览器环境中的其他组件类似，`<canvas>`由几组 API 组成，但并非所有浏览器都支持所有这些 API。除了具备基本绘图能力的 2D 上下文，`<canvas>`还建议了一个名为 WebGL 的 3D 上下文。兼容性：IE9+ 以及主流浏览器

### 基本用法

要使用元素，必须先设置`width`和`height`属性，指定可以绘图的区域大小。

```html
<canvas id="drawing" width="200" height="200">A drawing of something.</canvas>
```

要在 canvas 上画图，先要取得绘图上下文的引用，需要调用`getContext()`，传入上下文的名字。传入"2d"则取得 2D 上下文。

```javascript
var drawing = document.getElementById("drawing");
var context = drwaing.getContext("2d");
```

使用`toDataURL()`方法可以导出绘制的图像，接受一个参数：图像的 MIME 类型格式。比如取得画布中一幅 PNG 格式的图像：

```javascript
var drawing = document.getElementById("drawing");
var imgURI = drawing.toDataURL("image/png");
var image = document.createElement("img");
image.src = imgURI;
document.body.appendChild(image);
```

### 2D 上下文

2D 上下文的坐标始于元素的左上角，原点坐标是 (0,0)。

#### 填充和描边

2D 上下文的两种基本绘图操作是填充和描边。

> 填充，就是用指定的样式（颜色、渐变或图像）填充图形；描边，就是只在图形的边缘划线。

操作的结果取决于两个属性：`fillStyle`和`strokeStyle`。它们的值可以是字符串、渐变对象或模式对象，而且他们的默认值都是"#000000"；字符串可以是颜色值的任何格式，包括颜色名、十六进制码、rgb、rgba、hsl 或 hsla。

```javascript
var context = drawing.getContext("2d");
context.strokeStyle = "red";
context.fillStyle = "#0000ff";
```

<!--more-->

#### 绘制矩形

矩形是唯一一种可以直接在 2D 上下文中绘制的形状。有关的方法：

- `fillRect()`：填充矩形，填充的颜色由`fillStyle`属性指定。 
- `strokeRect()`：绘制的矩形用指定的颜色描边。颜色由`strokeStyle`属性指定。
  - 描边线条的宽度由`lineWidth`属性控制，可以是任意整数
  - `lineCap`属性可以控制线条末端的形状是平头，圆头，还是方头。("butt", "round", "square")
  - `lineJoin`属性可以控制线条相交的方式是圆交，斜交，斜接("round", "bevel", "miter") 
- `clearRect()`：清除画布上的矩形区域。

这三个方法都接受 4 个参数：矩形的 x 坐标，矩形的 y 坐标，矩形宽度和高度，单位都是 px。

```javascript
var context = drawing.getContext("2d");

context.fillStyle = "#ff0000";
context.fillRect(10,10,50,50);

context.fillStyle = "rgba(0,0,255,0.5)";
context.fillRect(30,30,50,50);
context.clearRect(40,40,10,10);

context.strokeStyle = "#ff0000";
context.strokeRect(20,20,50,50);

context.strokeStyle = "rgba(0,0,255,0.5)";
context.strokeRect(40,40,50,50);
```

#### 绘制路径

要绘制路径，首先调用`beginPath()`方法，然后再调用下列方法绘制路径：

- `arc(x, y, radius, startAngle, endAngle, counterclockwise)`：以 (x, y) 为圆心绘制一条弧线，半径为`radius`，`startAngle`和`endAngle`分别为起始和结束角度，最后一个参数表示是否按逆时针方向计算角度。 
- `arcTo(x1, y1, x2, y2, radius)`：从上一点到 (x2, y2) 画一条弧线，并且以给定的半径`radius`穿过 (x1, y1)。
- `bezierCurveTo(c1x, c1y, c2x, c2y, x, y)`：从上一点开始绘制一条曲线，到 (x,y) 为止，并且以 (c1x, c1y),(c2x, c2y) 为控制点。 
- `lineTo(x, y)`：从上一点绘制一条直线，到 (x,y) 为止。 
- `moveTo(x, y)`：将绘图游标移动到 (x,y)，不画线。 
- `quadraticCruveTo(cx, cy, x, y)`：从上一点绘制一条二次曲线，到 (x, y) 为止，以 (cx, cy) 为控制点。 
- `rect(x, y, width, height)`：从点 (x, y) 开始绘制一个矩形，这个方法是绘制矩形路径。

创建路径后，可调用以下方法： 

1. `closePath()`：绘制一条连接到路径起点的线条。 
2. `fill()`：填充 
3. `stroke()`：描边 
4. `clip()`：在路径上创建一个剪切区域。 

绘制一个时钟：

```javascript
context.beginPath();
//绘制外圆
context.arc(100,100,99,0,2*Math.PI,false);
//内圆
context.moveTo(194,100);
context.arc(100,100,94,0,2*Math.PI,false);
//分针
context.moveTo(100,100);
context.lineTo(100,15);
//时针
context.moveTo(100,100);
context.lineTo(35,100);
context.stroke();
```

由于路径的使用很频繁，所以就有一个名为`isPointInPath(x, y)`的方法，用来在路径被关闭之前确定(x, y) 在不在路径上。

#### 绘制文本

绘制文本主要由两个方法：`fillText()`和`strokeText()`。这两个方法都接受 4 个参数：要绘制的文本字符串，x 坐标，y 坐标和可选的最大像素高度。而且这两个方法都以 3 个属性为基础： 

- `font`：表示文本样式、大小及字体，用 CSS 中指定字体的格式来写，例如："bold 14px Arial"。
- `textAlign`：表示文本对齐方式。可能的值有"start"、"end"、"left"、"right"、"center"。建议使用"start"和"end"而不是"left"和"right"。
- `textBaseLine`：表示文本的基线，可能的值有"top"、"hanging"、"middle"、"alphabetic"、"ideographic"和"bottom"。

```javascript
context.font = "bold 14px Arial";
context.textAlign = "center";
context.textBaseline = "middle";
context.fillText("12", 100, 20);
```

由于绘制文本比较复杂，特定是需要把文本控制在某一区域中的时候，2D 上下文提供了辅助确定文本大小的方法`measureText()`。这个方法接受一个参数，即要绘制的文本。返回一个 TextMetrics 对象，目前只有一个 width 属性。

#### 变换

通过上下文的变换，可以把处理后的图像绘制到画布上。可以通过如下方法修改变换矩阵：

- `rotate(angle)`：围绕原点旋转图像`angle`弧度。
- `scale(scaleX, scaleY)`：缩放图像，在 x 方向乘以 scaleX，在 y 方向乘以 scaleY。
- `translate(x, y)`：将坐标原点移动到 (x, y)。执行后，坐标 (0, 0) 会变成之前由 (x, y) 表示的点。
- `transform(m1_1, m1_2, m2_1, m2_2, dx, dy)`：直接修改变换矩阵。
- `setTransform(m1_1, m1_2, m2_1, m2_2, dx, dy)`：将变换矩阵重置为默认状态，然后再调用`transform()`。

如果你知道将来还要返回某组属性与变换的组合，可以对上下文对象调用`save()`方法。调用这个方法后，当时的所有设置都会进入一个栈结构，得以妥善保管。然会对上下文进行其他修改。等想要回到之前保存的设置时，可以调用`restore()`方法，在保存设置的栈结构中向前返回一级，恢复之前的状态。`save()`方法保存的只是对绘图上下文的设置与变换，不会保存绘图上下文的内容。

#### 绘制图像

对上下文对象使用`drawImage()`方法，可以绘制图像，调用该方法时，可以使用三种不同参数组合。 

- 传入一个`<img>`元素，绘制图像起点的 x 和 y 坐标。如：

```javascript
var image = document.images[0];
context.drawImage(image, 10, 10);
```

- 如果你想要改变绘制后图像的大小，可以再多传入两个参数，分别表示目标宽度和目标高度。通过这种方法来缩放图像并不影响上下文的变换矩阵。

```javascript
context.drawImage(image, 10, 10, 20, 30);
```

- 再复杂一点，可以选择把图像的某个区域绘制到上下文。这种调用方法需要传入 9 个参数：要绘制的图像、源图像的 x 坐标、源图像的 y 坐标、源图像的宽度、源图像的高度、目标图像的 x 坐标、目标图像的 y 坐标、目标图像的宽度、目标图像的高度。

除了给`drawImage()`方法传入`<img>`元素外，还可以传入另一个`<canvas>`元素作为第一个参数。这样，就可以把另一个画布内容绘制到当前画布上。

#### 阴影

2D 上下文会根据以下几个属性的值，自动为形状或路径绘制出阴影：

- `shadowColor`：用 CSS 颜色格式表示的阴影颜色，默认为黑色。 
- `shadowOffsetX`：形状或路径 x 轴的阴影偏移量，默认为 0。
- `shadowOffsetY`：形状或路径 y 轴的阴影偏移量，默认为 0。
- `shadowBlur`：模糊的像素数，默认为 0，即不模糊。

```javascript
var context = drawing.getContext("2d");
context.shadowOffsetX = 5;
context.shadowOffsetY = 5;
context.shadowBlur    = 4;
context.shadowColor   = "rgba(0, 0, 0, 0.5)";            

context.fillStyle = "#ff0000";
context.fillRect(10, 10, 50, 50);
```

#### 渐变

渐变由 CanvasGradient 实例表示。通过 2D 上下文来创建和修改。

对上下文使用`createLinearGradient()`方法创建线性渐变对象。该方法接收 4 个参数：起点的 x 坐标、起点的 y 坐标、终点的 x 坐标、终点的 y 坐标。

创建渐变对象后，下一步就是使用`addColorStop()`来指定色标。这个方法接收两个参数：色标位置和 CSS 颜色值。色标位置是一个 0 到 1 之间的数字。

```javascript
var context = drawing.getContext("2d"),
    gradient = context.createLinearGradient(30, 30, 70, 70); 

gradient.addColorStop(0, "white");
gradient.addColorStop(1, "black");
context.fillStyle = gradient;
context.fillRect(30, 30, 50, 50);
```

也可以创建径向渐变：`createRadialGradient()`。有 6 个参数：对应着两个圆的圆心和半径。前三个是起点圆的圆心和半径，后三个是终点圆的。

```javascript
gradient = context.createRadialGradient(55, 55, 10, 55, 55, 30);
```

#### 模式

模式其实就是重复的图像，可以用来填充或描边图形。调用上下文对象的`createPattern()`方法并传入两个参数：一个HTML`<img>`元素和一个表示如何重复图像的字符串。其中，第 2 个参数的值与 CSS 的`background-repeat`属性值相同，包括 "repeat"、"repeat-x"、"repeat-y" 和 "no-repeat"。

```javascript
 var context = drawing.getContext("2d"),
image = document.images[0],
pattern = context.createPattern(image, "repeat");
context.fillStyle = pattern;
context.fillRect(10, 10, 150, 150);
```

`createPattern()`的第一个参数可以是一个`<video>`元素或另一个`<canvas>`元素。

#### 使用图像数据

可以使用`getImageData()`取得原始图像数据。接受 4 个参数，要取得其数据的画面区域的 x 和 y 坐标以及该区域的像素宽度和高度。 

``` javascript
var imageData = context.getImageData(10,5,50,50);
```

返回的对象是一个 ImageData 的实例。每个 ImageData 对象有 3 个属性：`width`，`height`和`data`（一个数组，保存着图像中每个像素的数据，每个像素用 4 个元素保存，分别是红，绿，蓝，透明度）

```javascript
var data = imageData.data,
    red = data[0],
    green = data[1],
    blue = data[2],
    alpha = data[3];
```

可以使用`putImageData(ImageData, x, y)`方法把图像数据绘制到画布上。

#### 合成

还有两个会应用到 2D 上下文中所有绘制操作的属性：`globalAlpha`和`globalCompositionOperation`。其中，`globalAlpha`是介于 0-1 的值，用于指定所有绘制的透明度。 

第二个属性`globalCompositionOperation`表示后绘制的图形怎样与先绘制的图形结合。可能的值如下： 

- `source-over`(默认值)：后绘制的图形位于先绘制图形的上方。 
- `source-in`：后绘制的图形与先绘制图形的重叠的部分可见，其他部分完全透明。 
- `source-out`：后绘制的图形与先绘制的图形不重叠的部分可见，先绘制的图形完全透明。 
- `source-atop`：后绘制的图形与先绘制的图形重叠的部分可见，先绘制的图形不受影响。 
- `destination-over`：后绘制的图形位于先绘制图形的下方，只有之前透明像素下的部分才可见。 
- `destination-in`：后绘制的图形位于先绘制图形的下方，两者不重叠的部分完全透明。 
- `destination-out`：后绘制的图形擦除与先绘制图形重叠的部分。 
- `destination-atop`：后绘制的图形位于先绘制图形的下方，在两者不重叠的地方，先绘制的图形变透明。 
- `lighter`：后绘制的图形与先绘制的图形重叠部分的值相加，使该部分变亮。 
- `copy`：后绘制的图形完全替代与之重叠的先绘制图形。 
- `xor`：后绘制的图形与先绘制的图形重叠部分执行“异或”操作。

### WebGL

WebGL 是针对 Canvas 的 3D 上下文。

#### 类型化数组

WebGL 涉及的复杂计算需要提前知道数值的精度，而标准的 JavaScript 数值无法满足需要，为此 WebGL 引入了一个概念，叫做**类型化数组**(typed arrays)。类型化数组也是数组，只不过其元素被设置为特定类型的值。

类型化数组的核心是一个名为 ArrayBuffer 的类型，每个 ArrayBuffer 对象表示的只是内存中指定的字节数，但不会指定这些字节用于保存什么类型的数据。通过 ArrayBuffer 能做的，就是预先分配一定数量的字节，给未来可能的使用做准备。例如下面这行代码会在内存中分配 20B。

创建了ArrayBuffer 对象后，能够通过这个对象获得的信息只有一个，那就是它所包含的字节数，通过访问这个对象的`bytelength`属性。

```javascript
var buffer = new ArrayBuffer(20);
var bytes = buffer.byteLength; //20
```

**视图**

使用 ArrayBuffer（数组缓冲器类型）的一种特别的方式就是用它来创建数组缓冲器视图。其中，最常见的视图是 DataView，通过它可以选择 ArrayBuffer 中的一小段字节。可以在创建 DataView 实例传入一个 ArrayBuffer、一个可选的字节偏移量（从该字节开始选择）和一个可选的要选择的字节数。例如：

``` javascript
//基于整个缓冲器创建一个新视图
var view = new DataView(buffer);
//创建一个开始于字节9的新视图
var view = new DataView(buffer, 9);
//创建一个字节9开始到字节18的新视图
var view = new DataView(buffer,9, 10);
```

实例化之后，DataView 对象会把字节偏移量以及字节长度信息保存在`byteOffset`和`byteLength`属性中。通过这两个属性可以在以后方便的了解视图的状态。另外，通过`buffer`属性也可以取得数组缓冲器。

|   数据类型   | getter                               | setter                                   |
| :------: | ------------------------------------ | ---------------------------------------- |
| 有符号8位整数  | getInt8(byteOffset)                  | setInt8(byteOffset, value)               |
| 无符号8位整数  | getUint8(byteOffset)                 | setUint8(byteOffset, value)              |
| 有符号16位整数 | getInt16(byteOffset, littleEndian)   | setInt16(byteOffset, value, littleEndian) |
| 无符号16位整数 | getUint16(byteOffset, littleEndian)  | setUint16(byteOffset, value, littleEndian) |
| 有符号32位整数 | getInt32(byteOffset, littleEndian)   | setInt32(byteOffset, value, littleEndian) |
| 无符号32位整数 | getUint32(byteOffset, littleEndian)  | setUint32(byteOffset, value, littleEndian) |
|  32位浮点数  | getFloat32(byteOffset, littleEndian) | setFloat32(byteOffset, value, littleEndian) |
|  64位浮点数  | getFloat64(byteOffset, littleEndian) | setFloat64(byteOffset, value, littleEndian) |

所有这些方法的第一个参数都是一个字节偏移量，表示要从哪个字节开始读取或者写入。有些数据类型的数据可能需要不只 1B，比如 32 位浮点数要用 4B。使用 DataView 就需要手动管理这些字节，即明确知道自己的数据需要多少个字节，并选择正确的读写方法。例如：

``` javascript
var buffer = new ArrayBuffer(20),
    view = new DataView(buffer),
    value;

view.setUint16(0, 25);
view.setUint16(2, 50);   //don't start at 1, 16-bit integers take two bytes
value = view.getUint16(0);
```

以上代码把两个无符号 16 位整数保存到了数组缓冲器中，因为每个 16 位整数要用 2B，所以保存第一个数的字节偏移量为 0，而保存第二个数的字节偏移量为 2。

用于读写 16 位或更大数值的方法都有一个可选的参数`littleEndian`。这个参数是一个布尔值，表示读写数值时是否采用小端字节序（即将数据的最低有效位保存在低内存地址中），而不是大端字节序（即将数据的最低有效位保存在高内存地址中）。

**类型化视图**

类型化视图一般也被称为类型化数。类型化视图也分几种，而且它们都继承了 DataView。

- Int8Array：表示 8 位二补整数。
- Uint8Array：表示 8 位无符号整数。
- Int16Array：表示 16 位二补整数。
- Uint16Array：表示 16 位无符号整数。
- Int32Array：表示 32 位二补整数。
- Uint32Array：表示 32 位无符号整数。
- Float32Array：表示 32 位 IEEE 浮点值。
- Float64Array：表示 64 位 IEEE 浮点值。

每种视图类型都以不同的方式表示数据，而同一数据视图选择的类型不同有可能占用一个或多个字节。

每个视图构造函数都有一个名为 BYTES_PER_ELEMENT 的属性，表示类型化数组的每个元素需要多少字节。因此， Uint8Array.BYTES_PER_ELEMENT 就是 1，而 Float32Array.BYTES_PER_ELEMENT 则为 4，可以利用这个属性来辅助初始化。

``` javascript
var int8s = new Int8Array(buffer, 0, 10*Int8Array.BYTES_PER_ELEMENT);
var uint16s = new Uint16Array(buffer, int8s.byteOffset + int8s.byteLength, 5*Uint16Array.BYTES_PER_ELEMENT);
```

上面的代码基于同一个数组缓冲器创建了两个视图，缓冲器的前 10B 用于保存 8 位整数，而其他字节用于保存无符号 16 位整数。在初始化 Uint16Array 的时候，使用了 Int8Array 的`byteOffset`和`byteLength`属性，以确保 uint16s 开始于 8 位数据之后。

类型化视图的目的在于简化对二进制数据的操作，创建类型化视图还可以不用首先创建 ArrayBuffer 对象。只要传入希望数组保存的元素数，相应的构造函数就可以自动创建一个包含足够字节数的 ArrayBuffer 对象。另外，也可以把常规数组转化为类型化视图，只要把常规数组传入类型化视图的构造函数即可。

类型化视图还有一个`subarray()`方法，使用该方法可以基于底层数组缓冲器的子集创建一个新视图。这个方法接收两个参数：开始元素的索引和可选的结束元素的索引。返回的类型与调用该方法的视图类型相同。例如：

``` javascript
var uint16s = new Uint16Array(10),
    sub = uint16s.subarray(2, 5);
```

以上代码中，sub 也是 Uint16Array 的一个实例，而且底层与 uint16s 都基于同一个 ArrayBuffer。通过大视图创建小视图的主要好处是，在操作大数组的一部分元素时，无需担心意外修改了其他元素。

#### WebGL 上下文

目前，在支持的浏览器中，WebGL 的名字叫做"experimental-webgl"，这是因为 WebGL 规范仍然没有制定完成。制定完成后，这个上下文的名字就会变成简单的"webgl"。如果浏览器不支持 WebGL，那么取得该上下文的时候会返回`null`。在使用 WebGL 上下文的时候，必须先检查一下返回值。

取得了 WebGL 上下文之后，就可以进行 3D 绘图了。通过给`getContext()`函数传递第二个参数，可以为WebGL上下文设置一些选项，这个参数本身是一个对象，可以包含下列属性：

- `alpha`：值为`true`，表示为上下文创建一个 Alpha 通道缓冲区；默认值`true`。
- `depth`：值为`true`，表示可以使用 16 位深缓冲区；默认值`true`。
- `stencil`：值为`true`，表示可以使用 8 位模板缓冲区；默认值`false`。
- `antialias`：值为`true`，表示将使用默认机制执行抗锯齿操作；默认值`true`。
- `premutipliedAlpha`：值为`true`，表示绘图缓冲区有预乘 Alpha 值；默认值`true`。
- `preserveDrawingBuffer`：值为`true`，表示在绘图完成后保留绘图缓冲区；默认值`false`。

大多数上下文选项都在高级技巧中使用，一般情况下，各个选项的默认值就可以满足我们的需求。

``` javascript
var drawing = document.getElementById("drawing");            
  //make sure <canvas> is completely supported
  if (drawing.getContext) {
      var gl;
      try {
          gl = drawing.getContext("experimental-webgl");
      } catch (ex) {
          //noop
      }

      if (gl){
          gl.clearColor(0, 0, 0, 1);
          gl.clear(gl.COLOR_BUFFER_BIT);
      } else {
          alert("WebGL context could not be created.");
      }
  }
}
```

**常量**

OpenGL 中的常量`GL_COLOR_BUFFER_BIT`在 WebGL 上下文中就是`gl.COLOR_BUFFER_BIT`。

**方法命名**

OpenGL 中的很多方法都视图通过名字传达有关数据类型的信息。方法名的后缀会包含参数个数，比如`gl.uniform4f()`意味接受 4 个浮点数，而`gl.uniform3i()`则表示要接受 3 个整数。如果接受数组参数而非一个单独参数，`gl.uniform3iv()`表示接受一个包含 3 个值的整数数组。

**准备绘图**

在实际操作 WebGL 上下文之前，一般都要使用某种实色清除`<canvas>`，为绘图做好准备。为此，首先必须使用`clearColor()`方法来指定要使用的颜色值，该方法接收的 4 个参数：红、绿、蓝和透明度。是每个参数必须是一个 0 到 1 之间的数值，表示每种分量在最终颜色中的强度。

``` javascript
gl.clearColor(0, 0, 0, 1);
gl.clear(gl.COLOR_BUFFER_BIT);
```

以上代码先把清理颜色缓冲区的值设置为黑色，然后调用了`clear()`方法，其中传入的参数`gl.COLOR_BUFFER_BIT`用来告诉 WebGL 用之前定义的颜色来填充相应区域。

**视口与坐标**

开始绘图之前，通常要先定义 WebGL 的视口(viewport)。默认情况下，视口可以使用整个`<canvas>`区域。要改变视口大小，可以调用`viewport()`方法并传入 4 个参数：分别代表视口相对于`<canvas>`元素的 x 坐标，y 坐标，宽度和高度。例如，下面就使用了`<canvas>`元素：

``` javascript
gl.viewport(0, 0, drawing.width, drawing.height);
```

视口坐标与我们通常使用的网页坐标不一样，视口坐标的原点 (0, 0) 在`<canvas>`元素的左下角，x 轴和 y 轴的正方向分别是向右和向上。

另外，视口内部的坐标系与定义视口的坐标系也不一样。在视口内部，坐标原点 (0, 0) 是视口的中心点，因此视口左下角的坐标为 (-1, -1)，而右上角的坐标为 (1, 1)。如果在视口内部绘图时使用视口外部的坐标，结果可能会被视口剪切。比如，要绘制的形状有一个顶点在 (1, 2)，那么该形状在视口右侧的部分会被剪切掉。

**缓冲区**

顶点信息保存在 JavaScript 的类型化数组中，使用之前必须转换到 WebGL 的缓冲区。要创建缓冲区，可以调用`gl.createBuffer()`，然后使用`gl.bindBuffer()`绑定到 WebGL 上下文，然后就可以使用数据来填充缓冲区了。

``` javascript
var buffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([0, 0.5, 1]), gl.STATIC_DRAW);
```

调用`gl.bindBuffer()`可以将`buffer`设置为上下文的当前缓冲区。此后，所有缓冲区操作都直接在`buffer`中执行。因此调用`gl.bufferData()`时不明确传入`buffer`也没有问题。最后一行代码使用 Float32Array中 的数据初始化了`buffer`(一般都是用 Float32Array 来保存顶点信息)。如果想使用`drawElements()`来输出缓冲区的内容，也可以使用`gl.ELEMENT_ARRAY_BUFFER`。

`gl.bufferData()`的最后一个参数用于指定使用缓冲区的方式，取值范围是如下几个常量：

- `gl.STATIC_DRAW`：数据只加载一次，在多次绘图中使用。
- `gl.STREAM_DRAW`：数据只加载一次，在几次绘图中使用。
- `gl.DYNAMIC_DRAW`：数据动态改变，在多次绘图中使用。

在包含缓冲区的页面重载前，缓冲区始终保留在内存中。可以调用`gl.deleteBuffer()`释放内存：

``` javascript
gl.deleteBuffer(buffer);
```

**错误**

WebGL 一般都不会抛出错误。为了知道是否有错误发生，必须在调用某个可能出错的方法后，手工调用`gl.getError()`方法。这个方法返回一个表示错误类型的常量，可能的错误常量如下：

| 常量                      | Description              |
| ----------------------- | ------------------------ |
| `gl.NO_ERROR`           | 上一次操作没有发生错误（值为 0）        |
| `gl.INVALID_ENUM`       | 应该给方法传入 WebGL 常量，但却传错了参数 |
| `gl.INVALID_VALUE`      | 需要无符号数的地方传入了负值           |
| `gl.INVALID_OPERATION`  | 在当前状态下不能完成操作             |
| `gl.OUT_OF_MEMORY`      | 没有足够内存                   |
| `gl.CONTEXT_LOST_WEBGL` | 由于外部事件干扰丢失了当前 WebGL 上下文  |


**着色器**

着色器（shader）是 OpenGL 中的另一个概念。WebGL 有两种着色器，顶点着色器和片段(或像素)着色器。顶点着色器用于将 3D 顶点转换为需要渲染的 2D 点，片段着色器用于准确计算要绘制的每个像素的颜色。WebGL 着色器的独特之处也是其难点在于，它们并不是用 JavaScript 写的，这些着色器是使用 GLSL（OpenGL Shading Language，OpenGL 着色语言）写的，这是一种与 C 和 JavaScript 完全不同的语言。

**编写着色器**

每个着色器都有一个`main()`方法，该方法在绘图期间会反复执行。为着色器传递数据的方式有两种：Attribute 和 Uniform。通过 Attribute 可以向顶点着色器中传入顶点信息，通过 Uniform 可以向任何着色器传入常量值。Attribute 和 Uniform 在`main()`方法外部定义。分别使用关键字`attribute`和`uniform`。在这两个值类型关键字之后，是数据类型和变量名。下面是一个简单的顶点着色器的例子：

``` javascript
attribute vec2 aVertexPosition;

void main() {
    gl_Position = vec4(aVertexPosition, 0.0, 1.0);
}
```

定义了一个名为 aVertexPosition 的 Attribute，这个 Attribute 是一个数组，包含两个元素（数据类型为 vec2），表示 x 和 y 坐标。即使只接收到两个坐标，顶点着色器也必须把一个包含四方面信息的顶点赋值给特殊变量`gl_Position`。这里的着色器创建了一个新的包含四个元素的数组（vec4），填补缺失的坐标，结果是把 2D 坐标转换成了 3D 坐标。

除了只能通过 Uniform 传入数据之外，片段着色器与顶点着色器类似。下面是片段着色器的例子：

``` javascript
uniform vec4 uColor;

void main() {
    gl_FragColor = uColor;
}
```

片段着色器必须返回一个值，赋给变量`gl_FlagColor`，表示绘图时使用的颜色。这个着色器定义了一个包含四方面信息（vec4）的统一颜色`uColor`。从上面的代码看，这个着色器除了把传入的值赋给`gl_FlagColor`外什么都没有做。`uColor`的值在这个着色器的内部不能改变。

**编写着色器程序**

浏览器不能理解 GLSL 程序，因此必须准备好字符串形式的 GLSL 程序，以便编译并链接到着色器程序。为便于使用，通常是把着色器包含在页面的`<script>`标签中，并为该标签指定一个自定义的`type`属性。由于无法识别`type`属性值，浏览器不会解析`<script>`标签中的内容，但这不影响我们读写其中的代码。例如：

``` html
<script type="x-webgl/x-shader" id="vertex-shader">
attribute vec2 aVertexPosition;

void main() {
    gl_Position = vec4(aVertexPosition, 0.0, 1.0);
}
</script>
<script type="x-webgl/x-shader" id="fragment-shader">
uniform vec4 uColor;

void main() {
    gl_FragColor = uColor;
}  
</script>
```

然后可以通过`text`属性取出`<script>`元素中的内容。

``` javascript
var vertexGlsl = document.getElementById("vertex-shader").text,
    fragmentGlsl = document.getElementById("fragment-shader").text;
```

取得 GLSL 字符串之后，接下来就是创建着色器对象。要创建着色器对象，可以调用`gl.createShader()`方法并传入要创建的着色器类型（`gl.VERTEXT_SHADER`或`gl.FRAGMENT_SHADER`）。编译着色器使用的是`gl.compileShader()`。如下：

``` javascript
var vertexShader = gl.createShader(gl.VERTEX_SHADER);
gl.shaderSource(vertexShader, vertexGlsl);
gl.compileShader(vertexShader);

var fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fragmentShader, fragmentGlsl);
gl.compileShader(fragmentShader);
```

上面的代码创建了两个着色器，并分别将它们保存在`vertexShader`和`fragmentShader`中。而使用下列代码，可以把这两个对象链接到着色器程序中。

``` javascript
var program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
gl.linkProgram(program);
```

第一行代码创建了一个程序，然后调用`attachShader()`方法又包含了两个着色器。最后调用`gl.linkProgram()`方法把两个着色器封装到变量`program`中。链接完程序之后，就可以通过`gl.useProgram()`通知 WebGL 使用这个程序了。

``` javascript
gl.useProgram(program);
```

调用`gl.useProgram()`方法后，所有后续的操作都将使用这个程序。

**为着色器传入值**

前面定义的着色器都必须接收一个值才能工作。为了给着色器传入这个值，必须先找到要接收这个值的变量。对于 Uniform 变量，可以使用`gl.getUniformLocation()`，这个方法返回一个对象，表示 Uniform 变量在内存中的位置，然后可以基于变量的位置来赋值。例如：

``` javascript
var uColor = gl.getUniformLocation(program, "uColor");
gl.uniform4fv(uColor, [0, 0, 0, 1]);
```

第一行代码从`program`中找到 Uniform 变量`uColor`，返回了它在内存中的位置。第二行代码使用`gl.uniform4fv()`给`uColor`赋值。

对于顶点着色器中的 Attribute 变量，也差不多是这个赋值过程。先找到 Attribute 变量在内存中的位置，可以调用`gl.getAttribLocation()`。取得位置之后，就可以赋值：

``` javascript
var aVertexPosition = gl.getAttribLocation(program, "aVertexPosition");
gl.enableVertexAttribArray(aVertexPosition);
gl.vertexAttribPointer(aVertexPosition, vertexSetSize, gl.FLOAT, false, 0, 0);
```

我们取得了`aVertexPosition`的位置，然后又通过`gl.enableVertexAttribArray()`来启用它。最后一行创建了指针，指向由`gl.bindBuffer()`指定的缓冲区，并将其保存在`aVertexPosition`中，以便顶点着色器使用。

**调试着色器和程序**

着色器可能会执行失败，而且是静默失败。对于着色器，可以在操作之后调用`gl.getShaderParameter()`，取得着色器的编译状态，用`gl.getShaderInfoLog()`捕获失败消息：

``` javascript
if(!gl.getShaderParameter(vertexShader, gl.COMPILE_STATUS)){
    console.log(gl.getShaderInfoLog(vertexShader));
}
```

程序也会执行失败，可以用`gl.getProgramParameter()`来检测执行状态，用`gl.getProgramInfoLog()`捕获失败消息。最常见的程序失败发生在链接过程中，用以下代码检测链接错误：

``` javascript
if(!gl.getProgramParameter(program, gl.LINK_STATUS)){
    console.log(gl.getProgramInfoLog(program));
}
```

**绘图**

执行绘图操作要调用`gl.drawArrays()`或者`gl.drawElements()`方法，前者用于数组缓冲区，后者用于元素数组缓冲区。第一个参数是一个常量，表示要绘制的形状：

- `gl.POINTS`：将每个顶点当成一个点来绘制。
- `gl.LINES`：将数组当成一系列顶点，在这些顶点间画线。每个顶点既是起点也是重点，因此数组中必须包含偶数个顶点才能完成绘制。
- `gl.LINE_LOOP`：将数组当成一系列顶点，在这些顶点间画线。线条从第一个顶点到第二个顶点，再从第二个顶点到第三个顶点，以此类推，直至最后一个顶点。然后再从最后一个顶点到第一个顶点画一条线，结果就是一个形状的轮廓。
- `gl.LINE_STRIP`：除了不画最后一个顶点与第一个顶点之间的线之外，其他与`gl.LINE_LOOP`相同。
- `gl.TRIANGLES`：将数组当成一系列顶点，在这些顶点间绘制三角形。除非明确指定，每个三角形都单独绘制，不与其他三角形共享顶点。
- `gl.TRIANGLES_STRIP`：除了将前三个顶点之后的顶点当作第三个顶点与前两个顶点共同构成一个新三角形外，其他都与`gl.TRIANGES`相同。
- `gl.TRIANGLES_FAN`：除了将前三个顶点之后的顶点当作第三个顶点与前一个顶点及第一个顶点共同构成一个新三角形外，其他都与`gl.TRIANGES`相同。

接受数组缓冲区中的起始索引作为第二个参数，接受数组缓冲区包含的顶点数作为第三个参数。下面代码绘制了一个三角形。

``` javascript
var vertices = new Float32Array([ 0, 1, 1, -1, -1, -1 ]),
    buffer = gl.createBuffer(),
    vertexSetSize = 2,
    vertexSetCount = vertices.length/vertexSetSize,
    uColor, aVertexPosition;

//put data into the buffer
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);					
gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);

//pass color to fragment shader
uColor = gl.getUniformLocation(program, "uColor");
gl.uniform4fv(uColor, [ 0, 0, 0, 1 ]);

//pass vertex information to shader
aVertexPosition = gl.getAttribLocation(program, "aVertexPosition");
gl.enableVertexAttribArray(aVertexPosition);
gl.vertexAttribPointer(aVertexPosition, vertexSetSize, gl.FLOAT, false, 0, 0);

//draw the triangle
gl.drawArrays(gl.TRIANGLES, 0, vertexSetCount);
```

**纹理**

WebGL 的纹理可以使用 DOM 中的图像，要创建一个新纹理，可以调用`gl.createTexture()`，然后再将一幅图像绑定到该纹理。如果图像尚未加载到内存中，可能需要创建一个 Image 对象的实例，以便动态加载图像。图像加载完成之前，纹理不会初始化，因此必须在`load`事件触发后才能设置纹理。例如：

``` javascript
var image = new Image(),
    texture;

image.src = "smile.gif";
image.onload = function (){
    texture = gl.createTexture();
    gl.bindTexture(gl.TEXTURE_2D, texture);
    gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true);
    gl.textImgae2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, image);
    gl.textParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST);
    gl.textParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
    gl.bindTexture(gl.TEXTURE_2D, null);
}
```

**读取像素**

使用`readPixels()`方法，参数：x、y、宽度、高度、图像格式、数据类型和类型化数组。前 4 个参数指定读取哪个区域中的像素，图像格式参数几乎总是`gl.RGBA`。数据类型参数用于指定保存在类型化数组中的数据类型。如下：

``` javascript
var pixels = new Uint8Array(25*25);
gl.readPixels(0, 0, 25, 25, gl.RGBA, gl.UNSIGNED_BYTE, pixels);
```

以上代码从帧缓冲区中读取了 25 x 25 像素的区域，将读取的信息保存在 pixels 数组中。

在浏览器绘制更新的 WebGL 图像之前调用`readPixels()`不会发生意外。绘制发生后，帧缓冲区会恢复其原始的干净状态，如果想获取绘制发生后的像素数据，那在初始化 WebGL 上下文时必须传入适当的`preserveDrawingBuffer`选项。

``` javascript
var gl = drawing.getContext("experimental-webgl", { preserveDrawingBuffer: true});
```

设置这个标志的意思是让帧缓冲区在下一次绘制之前，保留其最后的状态。