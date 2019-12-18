# 深度学习的JavaScript基础：矩阵和向量的表示

> 最近在读一本《基于浏览器的深度学习》，书比较薄，但是涉及的内容很多，因此在读的过程中不得不再查阅一些资料，以加深理解。我目前从事的本职工作就是浏览器研发，对于前端技术并不陌生。但是从前段时间开发微信小程序**识狗君**的过程来看，对JavaScript还是掌握得太少，特别是对一些前端框架以及一些比较新的JavaScript语法和编程模型，了解的不够。在修改tfjs-core源码时，就体会到这种痛苦。好吧，既然无法避开，那就正面刚吧。

与Java、C++这样的静态类型语言不同，JS中的变量似乎没有类型，在声明变量时不用指定变量类型。但实际上JS也有字符串、数字、布尔值、对象、数组、未定义等类型，是一种弱类型语言。在深度学习中，矩阵和向量是最基本的数据结构，而高效的矩阵和向量运算是深度学习计算中的关键。在C++中，数组可用于表示矩阵或向量，JS中也有这样的数据结构吗？

在JS中，提供了一种TypedArray的类，它是几种数组类型的统称：

* Int8Array
* Uint8Array
* Uint8ClampedArray
* Int16Array
* Uint16Array
* Int32Array
* Uint32Array
* Float32Array
* Float64Array

前缀中的U表示无符号的值。Uint8Array和Uint8ClampedArray都是保存0 ~ 255之间的值。如果保存的值大于256，Uint8Array会截掉溢出位，而Uint8ClampedArray对值进行限制，大于255的值限定为255，小于0的值限定为0。例如：

```javascript
var arr = new Uint8Array(4);
arr[0] = 256;
console.log(arr);  // [0, 0, 0, 0]

var arrc = new Uint8ClampedArray(4);
arrc[0] = 256;
arrc[1] = -12;
console.log(arrc);  // [256, 0, 0, 0]
```

在最新的JS规范中，还增加了 BigInt64Array 和 BigUint64Array 两种类，但并非每个浏览器都支持，请谨慎使用。

TypedArray可以以类型安全的方式访问数据，而不会造成数据复制的开销。TypedArray使用上有些类似C++中的数组，可以通过 [] 运算符读取或写入值。但实际上TypedArray是类，提供了一种访问数组中每个元素的方法，其实际数据存储在ArrayBuffer中。

#### ArrayBuffer

ArrayBuffer代表内存之中的一段二进制数据，是存储数据的实际数据结构，但它不提供读取或写入数据的任何方式。如何解释这些存放的数据，取决于TypedArray或稍后要讲到的DataView。你可以通过不同的TypedArray访问ArrayBuffer，可以在ArrayBuffer上使用不同的TypedArray，如何解释二进制数据的任务被委托给TypedArray。

```javascript
var buf = new ArrayBuffer(4);
var uint8 = new Uint8Array(buf);
var int16 = new Int16Array(buf);

uint8[0] = 1;
uint8[1] = 1;

console.log(uint8);  // [1, 1, 0, 0]
console.log(int16);  // [257, 0]
```

因为uint8和int16共享同一个底层ArrayBuffer，这样一个修改在两个TypedArray中都反映出来。TypedArray和ArrayBuffer通过避免冗余数据复制提供了一种访问内存数据的高效方法，实现了快速数据访问。

#### DataView

读取和写入ArrayBuffer数据的另一种方式是通过DataView，用TypedArray能做到的事情，一样可以用DataView完成。DataView在ArrayBuffer上提供了一个更低层次的接口，DataView不管理存储数据的类型。每次访问数据时，你需要知道存储的数据类型。

```javascript
var buf = new ArrayBuffer(4);
var d = new DataView(buf);

d.setInt8(0, 10);
console.log(d.getInt8(0));  // 10
```

需要注意的是，在多字节整数存储上，存在“大端”和“小端”的不同，取决于机器的体系结构，这意味着内存中同样的一块内存数据，在不同体系结构的机器上，解释为不同的值。DataView提供了一种显示指定“大端”和“小端”的接口。

```javascript
var buf = new ArrayBuffer(4);
new DataView(buf).setInt16(0, 127, true);  // 按小端保存
console.log(new Uint8Array(buf));  // [127, 0, 0, 0]

new DataView(buf).setInt16(0, 127, false);  // 按大端保存
console.log(new Uint8Array(buf));  // [0， 127, 0, 0]
```

通常情况下，我们无需关心大端和小端，但是如果存在数据在GPU和CPU之间传递，或者模型用于跨体系结构的机器上的情况时，就需要注意这个问题。

#### SharedArrayBuffer

在[深度学习的JavaScript基础：从callbacks到sync/await](https://mp.weixin.qq.com/s/Ctaz-edov5cWK14Rr3LqHw) 这篇文章中，我们提到JS代码是以单线程执行的，但这种说法并非完全正确，因为在HTML5中引入一种新的线程机制web workers。但web workers与其他语言中使用的线程略有不同。默认情况下，它们不共享内存。

这也就意味着，如果你想和其他线程共享数据，那么你就需要将数据从一个地方复制到另外一个地方。这是通过函数postMessage 完成的。postMessage 将所有输入的对象序列化，将其发送到另一个web worker，并将其反序列化并放入内存中。

一眼就可以看出，这种方式相当低效。某些情况下，我们需要更高效的并行策略，希望共享内存单元，ShareArrayBuffer正是为此而生。

通过 ShareArrayBuffer，web worker、不同线程可以在相同的内存块中读写数据。这也意味着你不再需要通过 postMessage 来在不同的线程中通信传递数据。不同的 web worker 都有获取/操作数据的权限。但是这也会带来一些问题，比如两个线程在同一时间对数据进行操作。这也是并发需要解决的问题之一。关于SharedArrayBuffer的并发是一个比较大的话题，这里先不展开讨论。

SharedArrayBuffer 顾名思义就是为线程间共享内存提供了一块内存缓冲区，你可以通过 postMessage 将线程 A 分配的 SharedArrayBuffer 发送给线程 B，然后两个线程就可以共同访问这块内存。

下面的代码通过创建 SharedArrayBuffer 来分配一块共享内存：

```javascript
var sab = new SharedArrayBuffer(1024);  // 1KiB shared memory
```

通过 postMessage 发送给另外一个 Worker 线程：

```javascript
w.postMessage(sab);
```

Worker 接收 SharedArrayBuffer 对象：

```javascript
var sab;
onmessage = function (ev) {
   sab = ev.data;  // 1KiB shared memory, the same memory as in the parent
}
```

同样的，我们可以通过TypedArray或DataView来读写SharedArrayBuffer：

```javascript
const w = new Worker('worker.js'),
buff = new SharedArrayBuffer(1);
var   arr = new Int8Array(buff);
/* setting data */
arr[0] = 9;
/* sending the buffer (copy) to worker */
w.postMessage(buff);
/* changing the data */
arr[0] = 1;
```

#### 小结

本文总结了在JavaScript如何表达深度学习中非常要的矩阵和向量，借助于TypedArray和ArrayBuffer，在JS中，我们也可以高效的处理矩阵数据，为JS中的深度学习提供了坚实的基础。

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)