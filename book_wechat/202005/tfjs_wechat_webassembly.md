# TensorFlow.js 微信小程序插件开始支持 WebAssembly

我们知道，微信小程序由一个描述整体程序的 app 和多个描述各自页面的 page 组成。小程序主体部分由 app.js、app.json、app.wxss三个文件组成，页面 page 则通常包含 js、wxml、json、wxss 文件。这些文件都是文本文件，由微信小程序引擎解析并解释执行。

然而，随着业务需求越来越复杂，微信小程序的逻辑越来越复杂，相应的代码量随之变的越来越多。对于 js 代码的执行，通常需要进行**词法分析 -> 语法分析 -> 预解析 -> 解释执行**等过程，性能太差。当然，随着 JS 引擎在发展的过程中引入了许多优化手段如字节码缓存，可以省掉每次解释执行时重新遍历语法树的过程。特别是谷歌的 V8 的 JIT 技术，在运行过程中直接生成并缓存机器码，下次执行时可由计算机直接执行，极大的提升了执行速度。然而，由于 JavaScript 这门语言本身的缺陷，使得优化变得越来越困难。由于JavaScript没有静态变量类型，只有动态变量，上一秒可能是Array，下一秒就变成了Object，那么引擎所做的优化就失去了作用，这会导致运行效率降低。

为了应对这一问题， WebAssembly 出现了。

#### WebAssembly

官方对 WebAssembly 的定义如下：

> WebAssembly（wasm）是一个可移植、体积小、加载快并且兼容 Web 的全新格式。

嗯，估计你看了这个定义还是不知道 WebAssembly 是什么。简单来说，WebAssembly是一种新的字节码格式，旨在成为高级语言的编译目标，目前可以使用C、C++、Rust、Go、Java、C#等编译器（未来还有更多）来创建wasm模块（见下图）。该模块以二进制的格式发送到浏览器，并在专有虚拟机上执行，与JavaScript虚拟机共享内存和线程等资源。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202005/images/tfjs_wechat_webassembly_01.jpg)

它是由 Google、Microsoft、Mozilla、Apple 等几家大公司合作发起的一个关于面向Web的通用二进制和文本格式的项目。

关于字节码格式，可以参考 Java，因为 Java 程序就是一种与平台无关的字节码。

首先，字节码是一种经过编译器编译之后的二进制代码，无需经过**解析**和**字节码编译**这两步。其次，WebAssembly强制使用静态类型，在语法上完全脱离JavaScript。其次，WebAssembly强制使用静态类型，在语法上完全脱离JavaScript，同时具有沙盒化的执行环境，安全性更好。最后，WebAssembly可直接和html以及浏览器进行交互。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202005/images/tfjs_wechat_webassembly_02.jpg)

相对于 JS，WebAssembly 有如下优点：

* **体积小**：由于浏览器运行时只加载编译成的字节码，一样的逻辑比用字符串描述的 JS 文件体积要小很多；
* **加载快**：由于文件体积小，再加上无需解释执行，WebAssembly 能更快的加载并实例化，减少运行前的等待时间；
* **兼容性问题少**：WebAssembly 是非常底层的字节码规范，制订好后很少变动，就算以后发生变化,也只需在从高级语言编译成字节码过程中做兼容。可能出现兼容性问题的地方在于 JS 和 WebAssembly 桥接的 JS 接口。

#### 微信小程序对WebAssembly的支持

微信小程序在Android / iOS上用于执行脚本以及渲染组件的环境都不尽相同。

在Android上，微信小程序的 Javascript 引擎采用了 V8，原生支持 WebAssembly，所以微信小程序在 Android 手机上提供 WebAssembly的支持。

在iOS上，微信小程序采用了苹果公司的 JavaScriptCore 引擎，并没有原生支持 WebAssembly，但最新 JavaScriptCore 也已经支持 WebAssembly。在不久的将来，IOS 手机
的微信小程序会支持 WebAssembly。

#### 使用TensorFlow.js的WASM backend

TensorFlow.js的WASM backend非常适合在中低端Android手机上使用。中低端手机的GPU往往相对CPU要弱一些，而WASM backend是跑在CPU上的，这就为中低端手机提供了另一个加速平台。而且WASM的能耗一般会更低。使用WASM backend需要修改package.json文件：

```json
{
  "name": "yourProject",
  "version": "0.0.1",
  "main": "dist/index.js",
  "license": "Apache-2.0",
  "dependencies": {
    "@tensorflow/tfjs-core": "1.7.3"，
    "@tensorflow/tfjs-converter": "1.7.3",
    "@tensorflow/tfjs-backend-wasm": "1.7.3",
    ...
  }
}
```

然后在app.js中设置 wasm backend, 你可以自行在服务器上托管 wasm 文件以提高下载速度, 下面例子中的 wasmUrl 可以替代成你的URL。

```
const info = wx.getSystemInfoSync();
    const wasmUrl = 'https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-backend-wasm@1.7.3/wasm-out/tfjs-backend-wasm.wasm';
    const usePlatformFetch = true;
    console.log(info.platform);
    if (info.platform == 'android') {
      setWasmPath(wasmUrl, usePlatformFetch);
      tf.setBackend('wasm').then(() => console.log('set wasm backend'));
    }
```

**注意** 使用WASM需要导入>= 1.7.3的tfjs库。

#### 小结

本文介绍了 WebAssembly 以及微信小程序对 WebAssembly 的支持情况，最后介绍了如何启用TensorFlow.js的WASM backend。由于在苹果手机上还未能全面支持 WebAssembly，在加上 WebAssembly 技术出现的比较晚（2015年），需要时间的检验，要在项目中全面采用 WebAssembly 不现实。但是如果项目中某个功能模块存在性能瓶颈，使用传统的 JS 实现效率太低，这个时候可以考虑 WebAssembly。如果在低端 Android 手机上运行 tfjs 微信小程序，也可以考虑采用WASM backend。

#### 参考

1. [https://github.com/tensorflow/tfjs-wechat](https://github.com/tensorflow/tfjs-wechat)
2. [WebAssembly进阶系列一：WebAssembly是什么](https://juejin.im/post/5d32e14e6fb9a07ed911fd0e)
   
![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)