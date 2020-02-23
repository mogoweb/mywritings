# TensorFlow.js 微信小程序插件开始支持模型缓存

通常情况下，微信小程序追求的是短小精悍，即开即用，用完即走，适用于一些简单的应用场景。然而，随着微信小程序开放能力的提高，人们发现用微信小程序可以实现越来越多的功能，小程序也越来越复杂，越来越庞大起来。这个可以从小程序的大小限制的变化看出，最开始小程序的大小限制为1M，后来限制为2M，最新微信又给小程序提供了分包加载机制，开发者将小程序划分成不同的子包，用户在使用时按需进行加载，所有分包大小限制提高到8M。

虽然小程序的大小限制已经大大提升，但对于小程序开发者而言，仍然捉襟见肘。随便几个图片资源、js库就可能导致小程序超重，尤其对于人工智能小程序而言，更是如此。现在的深度学习模型，动辄几十M，多则一两百M。这个时候开发人员就需要进行取舍，选择那些模型规模小，但精度不那么高的模型。比如图片分类，我们就不要选择Inception V3或ResNet之类的超大规模模型，而是选择针对移动设备优化的MobileNet，也能取得不错的效果。

不过即使是MobileNet，其模型大小也有好几M，对于精简小程序大小仍是一个很大的负担。一种解决方案是从网络加载模型，不增加小程序的体积，但这并不是一个完美的解决方案，毕竟每次推导都需要从网络下载模型，会有一定的网络延迟。在前端开发中，为了保持系统的流畅，通常会采用一些缓存技巧来避免每次从网络加载图片、JS等文件。那能否将模型也作为资源缓存起来呢？

Google团队显然也意识到了这种需求，先是在TensorFlow.js中增加了对tfjs模型缓存的支持。最近，TensorFlow.js 微信小程序插件也得到了更新，支持微信小程序模型缓存。

模型缓存利用了微信小程序的storage接口，需要注意微信小程序对storage的限制：同一个微信用户，同一个小程序 storage 上限为 10MB。storage 以用户维度隔离，同一台设备上，A 用户无法读取到 B 用户的数据；不同小程序之间也无法互相读写数据。所以我们只能选用小于10M的模型。

启用模型缓存也非常简单，步骤如下：

1. 修改app.json文件，将tfjsPlugin的版本修改为0.0.8.

```json
"plugins": {
  "tfjsPlugin": {
    "version": "0.0.8",
    "provider": "wx6afed118d9e81df9"
  }
}
```

2. 在app.js中提供localStorageHandler函数.

```javascript
var fetchWechat = require('fetch-wechat');
var tf = require('@tensorflow/tfjs-core');
var plugin = requirePlugin('tfjsPlugin');
//app.js
App({
  // expose localStorage handler
  globalData: {localStorageIO: plugin.localStorageIO},
  ...
});
```

3. 在模型加载时加入localStorageHandler逻辑。

```javascript
const LOCAL_STORAGE_KEY = 'mobilenet_model';
export class MobileNet {
  private model: tfc.GraphModel;
  constructor() { }

  async load() {

    const localStorageHandler = getApp().globalData.localStorageIO(LOCAL_STORAGE_KEY);
    try {
      this.model = await tfc.loadGraphModel(localStorageHandler);
    } catch (e) {
      this.model =
        await tfc.loadGraphModel(MODEL_URL);
      this.model.save(localStorageHandler);
    }
  }
```

和浏览器缓存机制有点不同的是，只有在代码包被清理的时候本地缓存才会被清理。如果需要处理缓存，可以通过 wx.setStorage/wx.setStorageSync、wx.getStorage/wx.getStorageSync、wx.clearStorage/wx.clearStorageSync，wx.removeStorage/wx.removeStorageSync 对本地缓存进行读写和清理。

另外需要注意的是，当前tfjs模型托管在tfhub上，需要翻墙访问。项目中的说明文件也提及了这个问题，给出了解决方案，但那是针对以前托管在谷歌云上的模型，建立了中国国内用户可以访问到的镜像。耐心等待吧，相信Google的开发人员会解决tfhub的镜像问题的。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)