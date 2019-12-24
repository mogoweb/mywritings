# 手把手教你开发人工智能微信小程序(0)：构建tfjs

本文将介绍如何从源码构建出 TensorFlow JS 库(tfjs)。对于大多数微信小程序开发者而言，并不需要经历这一步，要做的仅仅是把编译好的 tfjs 库加入微信小程序工程中。但我还是希望说说如何从源码编译出 tfjs 库，为什么呢？主要出于两个原因：

1. 完整的 tfjs 库大约有 900K，看起来是一个很小的体积，但我们要知道，微信小程序包有 2M 的大小限制。微信小程序添加一点图片，加上深度学习模型文件，很容易超过 2M 的大小。所以 tfjs 库能减则减，可能在项目开发中需要对 tfjs 库进行裁剪。
2. tfjs 主要是为浏览器和 NodeJS 而开发，微信小程序虽然也采用基于 web 的技术，但有些地方进行了限制，比如 web 中广泛使用的 fetch API 就被砍掉了。在某些情况下我们可能需要修改 tfjs 的源码，这个时候就需要自行构建 tfjs 库。

TensorFlow JS 主要包含 4 个重要的模块，在老的版本中，它们存在于几个独立的 git 库: tjfs-core, tfjs-layers, tfjs-converter, tfjs-data。在最新的版本中，它们整合到同一个 git 库中:

* TensorFlow.js Core, tfjs核心库，提供灵活的低层次API，由deeplearn.js发展而来。
* TensorFlow.js Layers, 提供高层次API，实现了类似于Keras的函数。
* TensorFlow.js Data, 提供简单的API加载和准备数据，功能类似于tf.data。
* TensorFlow.js Converter, 提供工具导入TensorFlow SavedModel格式模型到TensorFlow.js。

上述4个模块中， tfjs-core 是最基础的组件，其它几个模块都依赖这一模块。该可以单独编译，功能独立，能够单独使用，从而可以减少 tfjs 库的体积。

#### 安装yarn

tfjs 采用了 yarn 构建系统，对于前端开发者而言应该比较熟悉。下面简单说说如何在 Ubuntu 18.04 上安装 yarn ，其它平台上的安装方法，请自行搜索。

1. 导入 yarn 库的GPG key：

```bash
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
```

2. 往 *Ubunut* 系统添加 *yarn* APT库：

```bash
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
```

3. 安装 yarn:

```bash
sudo apt update
sudo apt install yarn
```

4. 检查 yarn 是否安装成功，如下命令输出 yarn 的版本号：

```bash
yarn --version
```

当前输出的版本号为：

```bash
1.21.1
```

需要注意的是， *Ubuntu* 中另外有个包中也有同名程序，如果你之前未安装过 *yarn*，运行yarn命令可能会出现如下提示：

```
Command 'yarn' not found, but can be installed with:

sudo apt install cmdtest
```

请勿安装这个 *cmdtest*，这是另外一个工具，并不是 *yarn* 构建系统。

#### 构建tfjs库

1. 克隆 *tfjs* 源码库：

```bash
git clone https://github.com/tensorflow/tfjs.git
```

2. 构建 *tfjs*

```
cd tfjs/tfjs
yarn build-npm
```

在当前目录下会生成一个 *tensorflow-tfjs-x.x.x.tgz* 的压缩包，其中 *x.x.x* 代表 *tfjs* 的版本号，写这篇文章时最新的版本号为1.5.1。如果我们要在项目中加入 *tfjs* 库，可以直接使用 *dist* 目录下的 *tf.min.js* 文件。

如果在项目中，我们只使用 *tfjs-core*，可以单独构建 *tfjs-core*:

```
cd tfjs/tfjs-core
yarn build-npm
```

编译出的 *tfjs-core* 库文件位于 *dist* 子目录下，名为 *tf-core.min.js* 。

#### 小结

本文介绍了如何从源码构建 tfjs 库，对于大多数微信小程序开发者而言并不需要，但如果你希望裁剪和定制 tfjs ，就可能需要从源码构建。在下一篇文章中，我将介绍人工智能微信小程序 Hello WeChat ，敬请关注！

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)
