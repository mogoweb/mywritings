# Stable Diffusion XL 带来哪些新东西？

前几天写了一篇小短文《 Stable Diffusion 即将发布全新版本》，很快，Stability AI 的创始人兼首席执行官 Emad Mostaque 在一条推文中宣布，Stable Diffusion XL 测试现已可用于公开测试。那么这样一个全新版本会带来哪些新东西，让我们眼见为实吧。

不过在开始之前，简单说明一下：

* XL 并不是新 AI 模型的正式名称。 一旦 Stability AI 正式宣布，它可能会改变
* 与以前的版本相比，图像质量有所提高
* 图像生成比以前的版本快很多

#### 体验方法

目前，新的开源模型还没有发布，无法进行本地部署，所以只能在线体验，地址如下：

> https://beta.dreamstudio.ai/generate

Stream Studio 的体验门槛很低，不需要邀请，也不需要等待，对于地区也没有限制，直接使用电子邮箱注册即可。

登入后主界面如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/sdxl_04.png)

可以看到，可以选择不同的模型来生成图像，这样也方便对比新旧模型之间的差异。

#### 费用

由于支撑 AI 模型运行需要消耗巨大的 GPU 算力，所以几乎所有的在线 AI 都会向用户收取一定的费用，这个可以理解。

看到这里也别慌，Stream Studio 为新用户提供了一定的免费额度，大约可以画 108 张图。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/sdxl_10.png)

Stream Studio 使用了一种信用分进行计费，大抵是：

10 美元 = 1000 信用分

生成一张图使用多少信用分，具体取决于生成图像配置（如图像大小，迭代步数等），参见如下表格：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/sdxl_06.png)


#### 示例图像

有了 108 张图像免费额度，那就下场体验一下，看看新旧AI模型在结果上的差异。

> Prompt: Luxury sports car with aerodynamic curves, shot in a high contrast, high key lighting with shallow depth of field, exotic, detailed, sporty, studio lighting, HQ, 4k

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/sdxl_01.png)

> Prompt: Bowl of steaming hot ramen with a perfect egg in the center, surrounded by thin slices of meat, green onions, and nori, with a flavorful broth and perfect noodles, high detail, focused on texture and steam

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/sdxl_02.png)

> Prompt: Enchanting waterfall in a lush jungle, surrounded by exotic plants and wildlife, tranquil, serene, high detail, tropical landscape

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/sdxl_03.png)

看到这里，你可能觉得新的 AI 模型并没有什么显著改进，可能生成的图像更好看，但并没有令人惊叹的效果。

至于在速度方面，因为是在线体验，感觉不到，这需要等到开源模型发布之后，在本地部署，才会有比较明显的感受。

#### 最后

可能从效果上看，Stable Diffusion 与商业产品 MidJourney、Leonardo AI 和 Microsoft Image Generator 相比有一定的差距。但它胜在开源，质量上也过得去。

尽管 Stable Diffusion XL 与之前的 AI 模型相比似乎没有显着改进，但它仍然是向前迈进了一步，相信今后还有进一步改进的空间。

我还是很期待新模型发布。