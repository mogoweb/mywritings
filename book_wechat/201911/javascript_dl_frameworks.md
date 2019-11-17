### 三种Javascript深度学习框架介绍

谈到机器学习，我们脑海首先蹦出的编程语言是什么？一定是python。其实除了python，Javascript也是不错的选择。都说现在是大前端时代，从移动开发、服务器端，甚至桌面软件开发（比如大名鼎鼎的VS Code），都有Javascript的身影。我在之前写过几篇有关浏览器中的机器学习的文章：

* [浏览器中的手写数字识别](https://mp.weixin.qq.com/s/5uNizYbLtOYLJfWU8T6wHw)
* [浏览器中的机器学习：使用预训练模型](https://mp.weixin.qq.com/s/LyzDKLZFlxNqv71eAvV1ig)
* [TensorFlow.js简介](https://mp.weixin.qq.com/s/aaudWL156X4DsxwHw0LDLA)

还写过一些关于微信小程序中的机器学习的文章：

* [当微信小程序遇上TensorFlow - tensorflow.js篇](https://mp.weixin.qq.com/s/M1pa1H1qVNC4kr58mDxC6g)
* [当微信小程序遇上TensorFlow - 本地缓存模型](https://mp.weixin.qq.com/s/CwrFWO7SIVaaB5NGw_2h-g)
* [当微信小程序遇上TensorFlow - 官方文档](https://mp.weixin.qq.com/s/ly4byGD34nOiXHC6LJnlKg)
* [TensorFlow助力微信小程序，来自谷歌开发者大会上的商用案例](https://mp.weixin.qq.com/s/eXvSHnyDmDMJCxGM2UJDIA)

用Javascript写机器学习应用，当然不会从头开始手写机器学习算法和模型，通常会借助现有框架。我之前接触的都是TensorFlow.js，其实除了TFJS，还有其它的深度学习框架。下面就介绍三种常用的Javascript深度学习框架。

#### TensorFlow.js

Tensorflow.js是业界的大哥大，Google出品，值得信赖。其实TensorFlow.js发布得很晚，到2017年中期才公开发布第一个beta版本，其前身是Deeplearn.js。不过TensorFlow.js是第一个在浏览器中提供硬件加速的开源深度学习框架，它利用了WebGL进行加速。

在浏览器中的机器学习，用户可以直接在浏览器中提供数据，进行实时训练学习，而不用额外安装软件。TensorFlow.js也无需用单独的深度学习框架构建离线的模型，随着浏览器对硬件能力的支持度越来越高（比如摄像头、麦克风等），我们可以在浏览器中运行越来越丰富的机器学习应用。

关于TensorFlow.js的更多介绍，请参考我之前写的文章：

* [TensorFlow.js简介](https://mp.weixin.qq.com/s/aaudWL156X4DsxwHw0LDLA)

#### WebDNN

WebDNN是由东京大学的机器智能实验室开发的，虽然它没有TensorFlow.js那么流行，但是它支持更多种类的深度学习框架：
* TensorFlow
* Keras
* PyTorch
* Chainer
* Caffe

如果你已经有了这些深度学习框架的模型，你可以用WebDNN很容易地导入这些模型。WebDNN有一个优化器管道，它看似一个编译器，将一个训练模型转换为一个WebDNN的中间表示的格式。在WebDNN优化中间表达之后，优化过的模型生成一个核操作图，如下图所示：

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/javascript_dl_frameworks_01.png)

WebDNN也能通过WebGL进行硬件加速，如果你的浏览器支持WebAssembly和WebGPU，还可以通过这些API加速。

TensorFlow.js和WebDNN的主要不同在于WebDNN只支持任务的推断阶段，而不能用在训练阶段。因此除了WebDNN，你还要熟悉前面介绍的几种深度学习框架之一。我们可以将WebDNN看做一个优化器，它能让预训练的模型在浏览器上运行得更快。

你可以使用pip安装WebDNN：

```
$ pip install webdnn
```

#### Keras.js

Keras.js只支持Keras生成的模型，但因为Keras本身支持多种深度学习框架后端，所以Keras.js间接支持Keras支持的深度学习框架后端，比如Tensorflow、CNTK、MXNet等。

像TensorFlow.js一样，Keras.js实现各种核函数。Keras.js同样不支持模型训练，所以你需要为Keras.js准备预训练模型来创建应用。

Keras.js可以运行在独立于主线程外的WebWorker，这可以避免阻塞渲染UI，对于提升良好的用户体验至关重要。

Keras.js提供很多使用的例子，你可以访问： https://github.com/transcranial/keras-js.git 了解更多。

#### 小结

三种Javascript深度模型框架之中，TensorFlow.js无疑是最流行的框架，无论是从功能、社区支持还是活跃度上，都碾压其它两个。但其它两个也各有特点，支持的后端框架更多，支持更多的模型类型，更容易和已有的资源整合。重要的是哪种工具适合哪类问题，深度学习框架还很年轻、不成熟，希望在看完这篇文章之后，可以帮助你更好的选择深度学习框架。

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)