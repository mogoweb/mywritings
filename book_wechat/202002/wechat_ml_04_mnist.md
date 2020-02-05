# 手把手教你开发人工智能微信小程序(4)： 训练手写数字识别模型

在上篇文章《[手把手教你开发人工智能微信小程序(3)：加载数据](https://mp.weixin.qq.com/s/2KFuw8B6YXt2OcAzmEt_lQ)》中，我给大家演示了如何通过fetch加载网络数据并进行数据归范化，出于演示的目的，例子做了简化处理，本文中将给大家介绍一个稍微复杂一点的例子：手写数字识别。很多机器学习的教程都以手写数字识别作为上手的示例，我在之前的文章也写过几篇：

* [浏览器中的手写数字识别](https://mp.weixin.qq.com/s/5uNizYbLtOYLJfWU8T6wHw)
* [一步步提高手写数字的识别率(1)](https://mp.weixin.qq.com/s/DJxY_5pyjOsB70HrsBraOA)
* [一步步提高手写数字的识别率(2)](https://mp.weixin.qq.com/s/H9I0KX0CBkHeap5Xpwp-5Q)
* [一步步提高手写数字的识别率(3)](https://mp.weixin.qq.com/s/MTugq-5AdPGik3yJb9yDJQ)

可供参考。在本文中，我将演示如何训练卷积神经网络模型来识别手写数字。

需要说明的是，不建议在微信小程序中训练模型，而且通常的流程是模型训练与模型使用分离，本文的示例在实用性上可能欠缺，仅仅是为了给大家展示一种可能性，同时让大家对整个机器学习的过程有所了解。阅读完本文后，你将了解到：

* 如何通过网络加载图片类型数据
* 如何使用tfjs Layers API定义模型结构
* 如何训练模型以及评估模型

#### 加载MNIST数据

针对手写数字识别问题，网络上已经有公开数据集MNIST。这是一套28x28大小手写数字的灰度图像，包含55000个训练样本，10000个测试样本，另外还有5000个交叉验证数据样本。该数据集有多种格式，如果使用keras、tensorflow之类的python机器学习框架，通常有内置的API加载和处理MNIST数据集，但tensorflow.js并没有提供，所以需要自己编写。

常见的MNIST数据集是以多张通过目录进行归类的图片集，比如手写数字0的图片都放到目录名为0的目录下，手写数字1的图片都放到目录名为1的目录下，依次类推，如下图所示：

![按目录归类的数据集](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202002/images/wechat_ml_04_mnist_01.png)

也有的数据集是将所有图片放到一个目录下，然后加上一个文本文件，描述每个文件对应的标签：

![csv文件](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202002/images/wechat_ml_04_mnist_02.png)

这种形式的数据集并不适合tfjs，因为出于安全的考虑，js无法访问本地文件，大量小的文件的网络访问效率很低。所以有人将65000个图片合并为一张图片，但不是简单的将65000个图片拼接起来，而是将每个图片的二进制像素线性展开，一张手写数字图片供784个像素，占图片中的一行，最后得到的图像尺寸为784 * 65000，最后形成的图像对我们来说像是一张无意义的图片：

![拼接的MNIST图片](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202002/images/wechat_ml_04_mnist_03.png)

加载MNIST图像数据的代码如下：

```javascript
  async load(canvasId, imgWidth, imgHeight) {
    const ctx = wx.createCanvasContext(canvasId);
    
    const datasetBytesBuffer =
      new ArrayBuffer(NUM_DATASET_ELEMENTS * IMAGE_SIZE * 4);

    const chunkSize = 5000;

    let drawJobs = [];
    for (let i = 0; i < NUM_DATASET_ELEMENTS / chunkSize; i++) {
      const datasetBytesView = new Float32Array(
        datasetBytesBuffer, i * IMAGE_SIZE * chunkSize * 4,
        IMAGE_SIZE * chunkSize);
      ctx.drawImage(
        MNIST_IMAGES_SPRITE_PATH, 0, i * chunkSize, imgWidth, chunkSize, 0, 0, imgWidth,
        chunkSize);

      drawJobs.push(new Promise((resolve, reject) => {
        ctx.draw(false, () => {
          // API 1.9.0 获取图像数据
          wx.canvasGetImageData({
            canvasId: canvasId,
            x: 0,
            y: 0,
            width: imgWidth,
            height: chunkSize,
            success(imageData) {
              for (let j = 0; j < imageData.data.length / 4; j++) {
                // All channels hold an equal value since the image is grayscale, so
                // just read the red channel.
                datasetBytesView[j] = imageData.data[j * 4] / 255;
              }
              resolve();
            },
            fail: e => {
              console.error(e);
              resolve();
            },
          });
        });
      }));
    }
    await Promise.all(drawJobs);

    this.datasetImages = new Float32Array(datasetBytesBuffer);

    const fetch = fetchWechat.fetchFunc();
    const labelsResponse = await fetch(MNIST_LABELS_PATH);

    this.datasetLabels = new Uint8Array(await labelsResponse.arrayBuffer());

    // Create shuffled indices into the train/test set for when we select a
    // random dataset element for training / validation.
    this.trainIndices = tf.util.createShuffledIndices(NUM_TRAIN_ELEMENTS);
    this.testIndices = tf.util.createShuffledIndices(NUM_TEST_ELEMENTS);

    // Slice the the images and labels into train and test sets.
    this.trainImages =
      this.datasetImages.slice(0, IMAGE_SIZE * NUM_TRAIN_ELEMENTS);
    this.testImages = this.datasetImages.slice(IMAGE_SIZE * NUM_TRAIN_ELEMENTS);
    this.trainLabels =
      this.datasetLabels.slice(0, NUM_CLASSES * NUM_TRAIN_ELEMENTS);
    this.testLabels =
      this.datasetLabels.slice(NUM_CLASSES * NUM_TRAIN_ELEMENTS);
  }
```

这段代码有几点需要注意：

1. 因为送入模型训练的是像素RGB数据，所以需要先对图片进行解码，提取每个手写数字对应的784个像素值，在微信小程序中是借助Canvas绘制图像这种方式获得，也许有其它更好的直接解码的方法。
2. 因为canvasGetImageData是一个异步方法，所以代码中使用了Promise异步模式，等待所有图像数据获取完毕。而图像分部分绘制，也是避免大图片绘制导致内存问题。
3. 整个数据集拆分为训练数据集和测试数据集，训练数据集包含55000个数据，测试数据集10000个数据。nextTrainBatch(batchSize)方法从训练集中返回一组随机图像及其标签。nextTestBatch(batchSize)方法从测试集中返回一批图像及其标签。

#### 定义模型结构

关于卷积神经网络，可以参阅《[一步步提高手写数字的识别率(3)](https://mp.weixin.qq.com/s/MTugq-5AdPGik3yJb9yDJQ)》这篇文章，这里定义的卷积网络结构为：

CONV -> MAXPOOlING -> CONV -> MAXPOOLING -> FC -> SOFTMAX

每个卷积层使用RELU激活函数，代码如下：

```javascript
function getModel() {
  const model = tf.sequential();

  const IMAGE_WIDTH = 28;
  const IMAGE_HEIGHT = 28;
  const IMAGE_CHANNELS = 1;

  // In the first layer of out convolutional neural network we have
  // to specify the input shape. Then we specify some paramaters for
  // the convolution operation that takes place in this layer.
  model.add(tf.layers.conv2d({
    inputShape: [IMAGE_WIDTH, IMAGE_HEIGHT, IMAGE_CHANNELS],
    kernelSize: 5,
    filters: 8,
    strides: 1,
    activation: 'relu',
    kernelInitializer: 'varianceScaling'
  }));

  // The MaxPooling layer acts as a sort of downsampling using max values
  // in a region instead of averaging.
  model.add(tf.layers.maxPooling2d({poolSize: [2, 2], strides: [2, 2]}));

  // Repeat another conv2d + maxPooling stack.
  // Note that we have more filters in the convolution.
  model.add(tf.layers.conv2d({
    kernelSize: 5,
    filters: 16,
    strides: 1,
    activation: 'relu',
    kernelInitializer: 'varianceScaling'
  }));
  model.add(tf.layers.maxPooling2d({poolSize: [2, 2], strides: [2, 2]}));

  // Now we flatten the output from the 2D filters into a 1D vector to prepare
  // it for input into our last layer. This is common practice when feeding
  // higher dimensional data to a final classification output layer.
  model.add(tf.layers.flatten());

  // Our last layer is a dense layer which has 10 output units, one for each
  // output class (i.e. 0, 1, 2, 3, 4, 5, 6, 7, 8, 9).
  const NUM_OUTPUT_CLASSES = 10;
  model.add(tf.layers.dense({
    units: NUM_OUTPUT_CLASSES,
    kernelInitializer: 'varianceScaling',
    activation: 'softmax'
  }));


  // Choose an optimizer, loss function and accuracy metric,
  // then compile and return the model
  const optimizer = tf.train.adam();
  model.compile({
    optimizer: optimizer,
    loss: 'categoricalCrossentropy',
    metrics: ['accuracy'],
  });

  return model;
}
```

如果有过tensorflow python代码编写经验，上面的代码应该很容易理解。


#### 训练模型

在浏览器中训练，也可以批量输入图像数据，可以指定batch size，epoch轮次。

```javascript
  const metrics = ['loss', 'val_loss', 'acc', 'val_acc'];
  const container = {
    name: 'Model Training', styles: { height: '1000px' }
  };
  // const fitCallbacks = tfvis.show.fitCallbacks(container, metrics);

  const BATCH_SIZE = 512;
  const TRAIN_DATA_SIZE = 5500;
  const TEST_DATA_SIZE = 1000;

  const [trainXs, trainYs] = tf.tidy(() => {
    const d = data.nextTrainBatch(TRAIN_DATA_SIZE);
    return [
      d.xs.reshape([TRAIN_DATA_SIZE, 28, 28, 1]),
      d.labels
    ];
  });

  const [testXs, testYs] = tf.tidy(() => {
    const d = data.nextTestBatch(TEST_DATA_SIZE);
    return [
      d.xs.reshape([TEST_DATA_SIZE, 28, 28, 1]),
      d.labels
    ];
  });

  return model.fit(trainXs, trainYs, {
    batchSize: BATCH_SIZE,
    validationData: [testXs, testYs],
    epochs: 10,
    shuffle: true,
  });
```

tfvis库在微信小程序中不能正常工作，所以无法像在浏览器中训练那样，可视化监控训练过程。这个训练过程比较长，我在微信开发者工具中通过模拟器大概需要半个小时，请耐心等待。

#### 评估训练的模型

评估时喂入测试集：

```javascript
function doPrediction(model, data, testDataSize = 500) {
  const IMAGE_WIDTH = 28;
  const IMAGE_HEIGHT = 28;
  const testData = data.nextTestBatch(testDataSize);
  const testxs = testData.xs.reshape([testDataSize, IMAGE_WIDTH, IMAGE_HEIGHT, 1]);
  const labels = testData.labels.argMax([-1]);
  const preds = model.predict(testxs).argMax([-1]);

  testxs.dispose();
  return [preds, labels];
}
```

计算在测试集上的准确率，也就是统计预测值和真实值匹配的个数：

```javascript
    const predsArray = preds.dataSync();
    const labelsArray = labels.dataSync();
    var n = 0;
    for (var i = 0; i < predsArray.length; i++) {
      console.log(predsArray[i]);
      console.log(labelsArray[i]);
      if (predsArray[i] == labelsArray[i])
        n++;
    }
    const accuracy = n / predsArray.length;
    console.log(accuracy);
```

#### 小结

本文探讨了如何从网络加载MNIST数据集，定义卷积神经网络模型，训练模型及评估模型。这个简单的例子，包含了机器学习的整个过程，虽然在实际中我们可能不会这样用。在下篇文章中，我将介绍如何使用现有模型。如果你有什么建议，欢迎留言。

本系列文章的源码请访问：

https://github.com/mogotech/wechat-tfjs-examples

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)
