# Android NN API，谷歌意在收编各路诸侯？

在过去的几年中，诸如智能手机和平板电脑之类的移动设备的计算能力得到了飞速增长，接近几年前主流台式机的水平。如果仅仅是运行普通的应用程序，那的确存在性能过剩。于是，芯片厂商将目光转向人工智能。

人工智能是当前的发展热点之一，其对计算能力的需求大力推进了GPU的发展，在这个过程中，NVIDIA无疑是最大的赢家，几乎垄断了桌面显卡市场。然而在移动端，依然是群雄逐鹿，高通、海思、联发科和三星等公司，均在手机芯片中集成了对 AI 的加速支持。NVIDIA曾经推出过手机芯片，但以失败告终。此外手机芯片领域还有一个大腕：苹果，由于其封闭的专有系统，不在本文讨论之列。

为了给芯片增加 AI 支持，各手机芯片厂商各显神通，比如高通，其芯片，除了 ARM 核的 CPU，还集成了 GPU 和 DSP。而海思半导体，除了 CPU、GPU 和 DSP 外，还增加了一个 NPU ( Neural Processing Unit)，专门加速在人工智能算法中广泛使用的基于向量和矩阵的计算。如下图所示：

![左图：高通Snapdragon 845，右图：海思麒麟970](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202007/images/nnapi_01.png)

同样，联发科和三星也有自己的架构。为了充分发挥这些专用的硬件加速，芯片厂商推出了 SDK 开发包。高通发布了专用的 Snapdragon Neural Processing Engine (SNPE) SDK ，该SDK支持常见的深度学习模型框架，例如Caffe / Caffe2、TensorFlow、PyTorch、Chainer、MxNet、CNTK 等。华为、联发科同样也分别推出了HiAI SDK、NeuroPilot SDK。理论上，这些 SDK 都支持主流的深度学习模型框架，但是第三方应用想要充分利用专用的硬件加速，必须使用厂商提供的 SDK，因为主流框架并未 build 出专门集成第三方 SDK 的版本，除非手机芯片厂商提供专用的编译版本。

就市场层面而言，百花齐放是好事，互相竞争，受益的是消费者，但对开发人员而言，则是一场灾难。为华为海思开发的 app 无法用在高通芯片的手机上，反之依然。为不同手机芯片厂商开发版本，不仅加大了开发工作量，也加大了后期维护工作量。对用户而言也是一个负担，一般用户哪里清楚芯片型号而选择正确的版本呢？

面对这种局面，Android 生态的盟主谷歌出手了，推出了 Android Neural Networks API，试图结束群雄各自为战的局面。

#### Android Neural Networks API

Android Neural Networks API (NNAPI) 是一个 Android C API，专为在 Android 设备上运行计算密集型运算从而实现机器学习而设计。NNAPI 旨在为更高层级的机器学习框架（如 TensorFlow Lite 和 Caffe2）提供一个基本功能层，用来建立和训练神经网络。搭载 Android 8.1（API 级别 27）或更高版本的所有 Android 设备上都提供该 API。

下图为 NNAPI 的简要系统架构。

![Android NN API 的系统架构](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202007/images/nnapi_02.png)

Android NN API 的架构有点类似于 OpenGL，谷歌制定标准和定义接口，硬件厂商提供驱动，合力完成，这比各厂商各自为战更符合技术发展趋势。简单说，就是谷歌把控标准，芯片厂商则专注于提高硬件性能。

根据上面提供的信息，我们还可以推断如下结论：

1. 因为 Android NNAPI 是一个 Android C API，所以应用一般不会直接使用 NNAPI，适配的任务落在机器学习框架上。这对于应用程序开发而言是一件好事，无需修改应用程序（或仅仅修改某个参数），即可享受 NNAPI 带来的硬件加速。
2. 并不是所有 Android 设备都支持（需要 Android 8.1 及以上版本） NNAPI，应用程序开发者需要考虑不支持 NNAPI 的设备。
3. 除了需要 Android 系统支持 NNAPI，芯片厂商需要提供 NN 驱动才行。虽然即使厂商不提供 NN 驱动，NNAPI也可以走 CPU 的路径，但这意味着并没有充分利用到 GPU 或 DSP。应用开发者需要做出选择，到底是选择 NNAPI 还是直接 GPU 加速。

---

是否谷歌就此一统江湖呢？也未必，因为从 **Android NN API 的系统架构**可以看出，架构落地还需要芯片厂商的通力支持，提供 NN 驱动。但从目前的状况来看，似乎芯片厂商对于 NN API的支持并不积极。起码我在华为手机上的测试，启用 NNAPI 并没有什么作用。但从长远来看，如果能够形成 OpenGL 这样的组织联盟，无论是对厂商，还是对开发者，都是有利的。

另外需要指出，虽然一般应用开发者不会直接使用 NNAPI，但通过 NDK，应用程序直接使用 NNAPI 也是可以的，具体可以参考官方文档：

https://developer.android.com/ndk/guides/neuralnetworks

github上也有一个 NNAPI 的完整例子：

https://github.com/android/ndk-samples/tree/master/nn-samples

有兴趣的同学可以研究一下。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)