# 手把手教你开发人工智能微信小程序(3)：加载数据

在上篇文章《[手把手教你开发人工智能微信小程序(2)：线性回归模型](https://mp.weixin.qq.com/s/4ECPgLplLSJRdDXD2ebZ5A)》，我们在代码中给定了一组训练数据，对于机器学习而言，这点数据是不够的。数据集可以有多种来源，本文就来说说如何从网络加载数据。

读完本文，你将学习到：

* 如何通过fetch加载网络数据
* 数据归范化

#### 加载网络数据

本文以网络上的公开数据集 **Boston House price** 为例，这个数据集有多种格式，为了简单期间，我们先以 JSON 格式为例。这个 house.json 文件我放到了我的个人网站上，在文章配套的源码库中你也可以找到它。

在 Javascript 中，有一个非常方便的 fetch API 用来获取网络数据，但非常遗憾的是，不知道出于什么原因，在微信小程序中，这个 API 被裁掉了。在《[手把手教你开发人工智能微信小程序(1)：Hello WeChat！](https://mp.weixin.qq.com/s/F3lYoBCKgSwCXYi3n_ElRA)》这篇文章中，为了使用tfjs，需要导入一个 fetch-wechat 模块，这实际上是一个采用微信小程序API实现 fetch API的模块。在代码中，我们可以使用这个模块获取网络数据。

```javascript
async function getData() {
  const fetch = fetchWechat.fetchFunc();
  const houseDataReq = await fetch('https://ilego.club/ai/dataset/house.json');
  const houseData = await houseDataReq.json();
  const cleaned = houseData.map(house => ({
    price: house.Price,
    rooms: house.AvgAreaNumberofRooms,
  }))
    .filter(house => (house.price != null && house.rooms != null));

  return cleaned;
}
```

上面的代码在获取到数据后，进行了两个处理：

1. 房屋价格和多个因素有关，这里为了简化问题起见，假设房价只与房间数量有关，所以只保留了房间数量及价格这两项数据。
2. 过滤掉没有定义价格或房间数量的条目。

#### 规范化特征数据

规范化数据是机器学习中一种常见的处理数据的一种技巧，目的是消除数据量纲对模型的影响，减少过拟合。最简单的规范化方法就是对数据进行归一化，就是将数据处理为[0, 1]之间的范围，其处理公式为：

```
Xnorm = (X - Xmin) / (Xmax - Xmin)
```

看看代码是如何实现的：

```javascript
function convertToTensor(data) {
  return tf.tidy(() => {
    // Step 1\. Shuffle the data    
    tf.util.shuffle(data);
    // Step 2\. Convert data to Tensor
    const inputs = data.map(d => d.rooms)
    const labels = data.map(d => d.price);
    const inputTensor = tf.tensor2d(inputs, [inputs.length, 1]);
    const labelTensor = tf.tensor2d(labels, [labels.length, 1]);
    //Step 3\. Normalize the data to the range 0 - 1 using min-max scaling
    const inputMax = inputTensor.max();
    const inputMin = inputTensor.min();
    const labelMax = labelTensor.max();
    const labelMin = labelTensor.min();
    const normalizedInputs = inputTensor.sub(inputMin).div(inputMax.sub(inputMin));
    const normalizedLabels = labelTensor.sub(labelMin).div(labelMax.sub(labelMin));
    return {
      inputs: normalizedInputs,
      labels: normalizedLabels,
      // Return the min/max bounds so we can use them later.
      inputMax,
      inputMin,
      labelMax,
      labelMin,
    }
  });
}
```

第一步将数据随机打乱，也是一种减少过拟合的技巧。

第二步将数组转化为tensor

第三步对数据进行归一化。

#### 构建模型并训练

这个步骤和上篇文章中讲到的步骤是一样的，这里模型稍微修改一下，增加一个层：

```javascript
function createModel() {
  // Create a sequential model
  const model = tf.sequential();

  // Add a single hidden layer
  model.add(tf.layers.dense({ inputShape: [1], units: 1, useBias: true }));

  // Add an output layer
  model.add(tf.layers.dense({ units: 1, useBias: true }));
  return model;
}
```

接下来训练模型，因为数据比较多，一次性训练所有数据，可能会出现内存溢出，所以需要指定一个batch size

```javascript
async function trainModel(model, inputs, labels) {
  // Prepare the model for training.  
  model.compile({
    optimizer: tf.train.adam(),
    loss: tf.losses.meanSquaredError,
    metrics: ['mse'],
  });

  const batchSize = 28;
  const epochs = 50;

  return await model.fit(inputs, labels, {
    batchSize,
    epochs
  });
}
```

注意代码中指定优化器和损失函数的方式和上篇文章也有所不同，不是以字符串的形式指定，两种方法都可以，你可以根据自己的偏好选择。

#### 推理

需要注意的是，因为模型是通过规范化的数据训练的，所以在推理是，输入数据需要进行归一化处理，而结果需要反归一化：

```javascript
    const inputTensor = tf.tensor2d([5], [1, 1]);
    const normalizedInputs = inputTensor.sub(inputMin).div(inputMax.sub(inputMin));
    const preds = model.predict(normalizedInputs);
    const unNormPreds = preds.mul(labelMax.sub(labelMin)).add(labelMin);
```

#### 小结

本文探讨了如何从网络加载数据集，并采用归一化对数据进行处理。例子做了简化处理，仍然算不上一个实用的例子，在下篇文章中，我将介绍一个稍微复杂的例子：手写数字识别。如果你有什么建议，欢迎留言。

本系列文章的源码请访问：

https://github.com/mogotech/wechat-tfjs-examples

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)


