# SDXL 模型之 base、refiner 和 VAE

在上一篇文章《[Stable Diffusion 开源模型 SDXL 1.0 发布](https://mp.weixin.qq.com/s/eWli2YmwGwuGLZGFhLRILQ)》中介绍了 Stable Diffusion 最新模型 SDXL。然而在下载模型时发现，模型有两个，分别是 stable-diffusion-xl-base-1.0、stable-diffusion-xl-refiner-1.0。开始也没有仔细看介绍，想当然的认为 refiner 模型肯定比 base 模型好。于是就下载了 refiner 模型，然而使用模型生成的图像，总感觉有些不对劲：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_01.png)

使用的提示语为：

> masterpiece,(best quality:1.3),ultra high res,raw photo,detailed skin,style: realistic pictures,
> 1girl,chilly nature documentary film photography,snow mountain environment,natural light,a clear face,minor acne,

尝试了多次，生成的图像效果很差，要么比例不对，要么张冠李戴，各种稀奇古怪的问题。

那就试试 base 模型吧，使用相同的提示语，得到的图像为：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_02.png)

感觉图像质量还好一些。这是怎么回事，仔细阅读了一下模型说明，原来已经讲得很清楚了：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_03.png)

也就是说，base 模型是用来做文生图，refiner 模型是用来做图生图的。

首先，使用基础模型（Base）模型生成(有噪音的)潜在变量，然后再由专门用于去噪的精修模型（refiner）进一步处理。

当然，基础模型本身也可以作为独立模块使用，但是串联起效果更好。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_04.png)

可以看出，经过 refiner 模型进一步处理的图像对比度更高，发丝的边缘处理更好，当然效果和 base 模型生成的图像各有千秋，喜欢 base 模型生成的图像还可以省掉后面一步。

如果我们进一步查看模型文件，发现 base 模型和 refiner 模型各有一个 vae 的版本：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_05.png)

此外还有一个单独的 sdxl_vae 模型：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_06.png)

这个 vae 又是怎么一回事？

#### 什么是 VAE？

VAE（Variable Auto Encoder，变量自动编码器）是一个添加到稳定扩散检查点模型中的文件，以获得更鲜艳的颜色和更清晰的图像。 VAE 通常还有改善手部和面部的额外好处。

可以看看经过 VAE 处理的图像对比：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_06.jpg)

从技术上讲，模型可以内置 VAE，可以使用一些通用的外部 VAE，某些外部 VAE 甚至比内置 VAE 工作得更好。

上面提到的 sd_xl_base_1.0_0.9vae.safetensors 模型文件就是内置 VAE 的 sdxl 模型，可以独立使用，而 sdxl_vae 则属于外部 VAE 模型，需要搭配 sdxl 基础模型使用。从文件大小上也可以看出，sd_xl_base_1.0_0.9vae.safetensors 高达 6.9 G，而 sdxl_vae 则只有 300 多 M。

下图是使用 sd_xl_base_1.0_0.9vae.safetensors 模型文件生成的：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_07.png)

使用 sdxl_vae 搭配 sdxl 基础模型生成的图像：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_08.png)

可以看出，使用了 VAE 之后，生成的图像色彩更加饱满，面部细节更加丰富。

#### 使用 VAE（以 stable-diffusion-webui 为例）

对于内置 VAE 的 sdxl 模型，使用方法和基础模型一样，将模型文件放到 stable-diffusion-webui\models\Stable-diffusion 文件夹，然后刷新一下，从下拉列表中选择即可：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_09.png)

对于外部 VAE 模型，步骤稍微多一些。

首先下载上面提到的外部 VAE 文件，并将它们放置在文件夹 stable-diffusion-webui\models\VAE 中。

**注意：是 VAE 目录，和基础模型的位置有所不同。**

接下来，在 WebUI 中，单击 **Settings** > **Show all pages**。

然后在设置项中搜索 **Quicksettings list**，在 sd_model_checkpoint 之后添加 sd_vae。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_10.png)

滚动到设置顶部。 单击“Apply settings”，然后单击“Reload UI”。

现在，在 txt2img 顶部可以看到 SD VAE 下拉列表，在那里选择您的 VAE。如果下载了新的 VAE，并将其放入 VAE 文件夹后，单击蓝色刷新按钮即可在下拉列表中看到它。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_models_11.png)

#### 小结

使用 sdxl 模型的基本流程是先使用基础模型以文生图，然后再使用精炼模型以图生图。但我个人更喜欢基础模型以文生图的图像，反而觉得使用精炼模型后图像对比度太高，当然我目前只尝试了人物肖像，可能在风景、动漫等类型的图像中，精炼模型有更好的表现。

VAE 能获得更鲜艳的颜色和更清晰的图像，更加讨喜，所以我目前倾向于使用内置 VAE 的 SDXL 基础模型。

你喜欢哪种呢？欢迎留言。
