# TensorFlow.js 为何要引入 WASM 后端

在前面的一篇文章《TensorFlow.js 微信小程序插件开始支持 WebAssembly》中，我们谈到了 Tensorflow.js（tfjs） 的新后端 WebAssembly（WASM）。这篇文章进一步挖掘 tfjs WASM 后端的更多信息，并探讨一下 tfjs 为何要引入 WASM 后端。

首先澄清一个概念，WASM 后端只是 tfjs 的一种实现，就和 WebGL 实现的后端一样，对于微信小程序或 WebApp 而言，并不与后端直接打交道，并不会感知到 WASM 后端的存在。

其次，就实现而言，WASM 后端和 WebGL 后端是平行存在的，不存在谁取代谁的问题，针对不同的用户场景，选择合适的后端。而实现上的区别， WebGL 后端需要 GPU 支持，并且需要 WebGL 支持，而 WASM 完全是基于 CPU 运算的。当然，除了这两个后端，还有一个基于 CPU 运算的普通 Javascript 实现。

我们知道， GPU 是非常适合机器学习的运算，不论是模型训练还是推导， GPU 能从性能上提升好几个数量级，一般而言机器学习离不开 GPU。那 Google 为何又要为 tfjs 开发出一个 WASM 后端呢？这不是在开历史倒车吗？

查看了 Google 的官方资料后，总结出如下几点理由：

* 大量的低端移动设备缺乏 WebGL 支持，或者有 GPU 但速度很慢。而 WASM 是一种跨浏览器工作、可移植汇编和兼容 Web 的二进制文件格式，可在 Web 上实现接近原生代码的执行速度。全球 90％设备 都支持 WASM。
* 处于速度上的考虑。普通 Javascript 实现适应面更广， 但 WASM 的原生解码速度比 JavaScript 的解析速度 快 20 倍。在前面的文章中也分析过，由于 JavaScript 是动态类型语言，并且会执行垃圾回收，因此可能会在运行时导致明显的速度减缓。与 WebGL 后端进行对比，对于大多数模型，WebGL 后端的性能仍然优于 WASM 后端。 但 WASM 在超精简模型（小于 3MB 的模型）上的速度则更快。这个时候， GPU 并行化带来的收益无法抵消执行 WebGL 着色器的固定开销成本。Google 在 2018 版 MacBook Pro（Intel i7 2.2GHz，Radeon 555X）上，做过 WebGL、WASM 和普通 JS (CPU) 在 Chrome 上的推理时间的对比，结果如下：

  ![性能对比](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202006/images/tfjs_wasm_01.png)

  从上表可以看出 WASM 后端比普通 JS（CPU）后端快 10-30 倍。与 WebGL 的 PK 则互有胜负，对于类似 MediaPipe 的 BlazeFace 和 FaceMesh 等超轻量级模型，WASM 与 WebGL 速度相当，或比之更快。而对于类似 MobileNet、BodyPix 和 PoseNet 的中型边缘模型，WASM 的速度比 WebGL 慢 2-4 倍。
* 可移植性和稳定性方面，WASM 原生支持浮点运算，而 WebGL 后端则需要 OES_texture_float 扩展，但并非所有设备都支持此扩展。GPU 驱动程序可能与特定硬件相关，而且不同的设备也可能存在精度问题。比如在 iOS 上，GPU 不支持 32 位浮点数，因此只能退化到 16 位浮点数，从而导致精度问题。在 WASM 中，将始终以 32 位浮点数进行计算，因此在所有设备之间都能实现一致的精度。

* WASM 仍然存在优化的空间。值得一体的方案有　SIMD（Single Instruction Multiple Data，单指令流多数据流） WASM 扩展方案。在热门机器学习模型上使用 SIMD-WASM 进行的基准测试表明，其速度相比非 SIMD WASM 提高了 2-3 倍，而采用　LLVM　优化 SIMD 指令后，还可额外提速 26-50％。另外多线程的支持更加完善后，可充分利用线程提升性能，而无需修改　tfjs 用户代码。

#### 小结

tfjs WASM　后端因其设备支持官方，在性能上又能取得不错的平衡，特别是对于微信小程序而言，通常不会选用太复杂的模型，性能优势更明显。而　SIMD 和线程之类的新扩展，将如虎添翼，让　tfjs WASM 后端越来越受欢迎。作为一名　C++　程序员，我也希望我的 C++ 编程技能也能在 Web 应用开发方面一展拳脚。

#### 参考

1. [Introducing the WebAssembly backend for TensorFlow.js](https://blog.tensorflow.org/2020/03/introducing-webassembly-backend-for-tensorflow-js.html)
2. [Face and hand tracking in the browser with MediaPipe and TensorFlow.js](https://blog.tensorflow.org/2020/03/face-and-hand-tracking-in-browser-with-mediapipe-and-tensorflowjs.html)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)