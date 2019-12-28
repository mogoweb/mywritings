# 手把手教你开发人工智能微信小程序(1)：Hello WeChat！

每个开始学习编程的程序员，大约是从“Hello World！”开始的吧。就这样一个简简单单在屏幕上输出“Hello World！”字样的程序，帮助我们进入编程世界。这里我也以一个最简单的小程序开启人工智能微信小程序之旅。致敬经典，Hello WeChat！

在本文，您将掌握：

1. 如何申请微信小程序。
2. 新建支持tfjs的微信小程序工程。
3. 编写简单的tfjs代码。

#### 申请微信小程序

在微信公众平台官网首页（mp.weixin.qq.com）点击右上角的“立即注册”按钮。

![微信公众平台首页](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/wechat_ml_01_hello_wechat_01.png)

然后按照向导一步步进行下去即可，如果中间遇到什么疑问，参考小程序注册流程：[https://kf.qq.com/faq/170109iQBJ3Q170109JbQfiu.html](https://kf.qq.com/faq/170109iQBJ3Q170109JbQfiu.html)

注册成功之后，登入微信公众平台。然后点击**设置**，记下小程序的ID，这个在后面新建小程序工程时需要：

![小程序ID](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/wechat_ml_01_hello_wechat_02.png)

#### 新建微信小程序工程

1. 打开**微信小程序开发者工具**，新建项目，在**AppID**那一栏填入在上一步记下的小程序ID：
   
   ![新建小程序项目](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/wechat_ml_01_hello_wechat_03.png)

2. 点击**新建**按钮后，**微信小程序开发者工具**会自动生成小程序的基本框架代码，项目布局如下：
   
   ![小程序项目布局](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/wechat_ml_01_hello_wechat_04.png)

3. 添加**tfjs微信小程序插件**
   
   登录小程序管理后台,也就是前面步骤中的微信公众平台： 设置 -> 第三方设置 -> 添加插件，搜索框中输入 wx6afed118d9e81df9，查找插件并添加。

   ![tfjs插件](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/wechat_ml_01_hello_wechat_05.png)

#### 配置微信小程序工程

1. 增加npm支持，修改基础库版本。
   
   点击 **设置** -> **项目设置**，打开项目详情界面。**调试基础库**请选择 2.7.0以上版本，勾选**增强编译**和**使用 npm 模块**。

   ![项目详情](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/wechat_ml_01_hello_wechat_06.png)

2. 打开操作系统的命令行程序，进入项目的根目录，执行：

   ```
   $ npm init
   ```

3. 在后续的提示里，一直按回车键，使用默认值。
4. 经过第3步，在项目下会多出一个package.json文件，往该文件中添加：
   
   ```
   "devDependencies": {
     "miniprogram-api-typings": "^2.6.5-2"
   },
   "dependencies": {
     "@tensorflow/tfjs-core": "1.3.1",
     "fetch-wechat": "0.0.3"
   }
   ```
5. 再到操作系统的命令行，进入项目的根目录，接着执行：

   ```
   $ npm install
   ```

6. 点击微信开发者工具中的**工具** -> **构建npm** ，就可以继续下去。这时可能会弹出提示：

   ![警告](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/wechat_ml_01_hello_wechat_07.png)
   
   忽略之。

7. 引入插件代码包，往项目文件的app.json文件中加入：
   
   ```
   "plugins": {
     "tfjsPlugin": {
       "version": "0.0.7",
       "provider": "wx6afed118d9e81df9"
     }
   }
   ```

   tfjs插件的最新版本是0.07。

#### 微信小程序中添加tfjs代码

1. 在app.js的onLaunch里调用插件configPlugin函数

   ```javascript
   //app.js
   const fetchWechat = require('fetch-wechat');
   const tf = require('@tensorflow/tfjs-core');
   const plugin = requirePlugin('tfjsPlugin');

   App({
     onLaunch: function () {
       plugin.configPlugin({
         // polyfill fetch function
         fetchFunc: fetchWechat.fetchFunc(),
         // inject tfjs runtime
         tf,
         // provide webgl canvas
         canvas: wx.createOffscreenCanvas()
       });

       // 其它初始化代码
     }
   });
   ```

2. 组件设置完毕就可以开始使用 TensorFlow.js库的API了。比如加入如下简单的代码：
   
   ```javascript
   tf.tensor("Hello WeChat!").print();
   ```
   
   这段代码将在微信开发者工具的调试控制台输出：

   ![控制台输出](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/wechat_ml_01_hello_wechat_08.png)


至此，一个非常非常简单的人工智能微信小程序就完成了，虽然它啥事也没干。别着急，在后面的文章将逐步带你深入人工智能小程序的开发。

#### 小结

本章主要讲解了建立一个支持tfjs的微信小程序工程，包括申请微信小程序，配置微信小程序工程，以及添加一段简单的tjfs代码。在下一篇文章中，我将以一个线性回归的例子，说明微信小程序中的tfjs。如果你有什么建议，欢迎留言。

本系列文章的源码请访问：

https://github.com/mogotech/wechat-tfjs-examples

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)

