# Stable Diffusion 开源模型 SDXL 1.0 发布

关于 SDXL 模型，之前写过两篇：

* [Stable Diffusion即将发布全新版本](https://mp.weixin.qq.com/s/ZYjLNSa8klhlBOqMiPf-3w)
* [Stable Diffusion XL 带来哪些新东西？](https://mp.weixin.qq.com/s/Bm_JMhD3buAJMIaKszsPag)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202307/images/sdxl1.0_01.png)

一晃四个月的时间过去了，Stability AI 团队终于发布了 SDXL 1.0。当然在这中间发布过几个中间版本，分别是 SDXL beta 和 SDXL 0.9。相较于 SDXL 0.9 的仅供研究的版本，这次的完整版本进步明显，是目前最好的开放图像生成模型。经过 Discord 上收集的实验数据，与其他开放模型相比，人们更喜欢 SDXL 1.0 生成的图像。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202307/images/sdxl1.0_02.png)

SDXL 可以生成几乎任何艺术风格的高质量图像，尤其擅长照片写实主义。SDXL 1.0 特别针对鲜艳而准确的色彩进行了精心调校，与之前的 Stable Diffusion 1.x 和 2.x 模型相比，具有更好的对比度和明暗表现。

此外，SDXL 可以生成图像模型难以渲染的概念，例如手和文本或空间排列的构图（例如，背景中的女人在前景中追逐狗）。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202307/images/sdxl1.0_03.png)

SDXL 只需要几句话就可以创建复杂、详细且美观的图像。 用户不再需要调用“杰作”之类的限定词来获得高质量的图像。 此外，SDXL 可以理解“红场”（著名的地方）与“红场”（形状）等概念之间的差异。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202307/images/sdxl1.0_04.png)

SDXL 1.0 是最大的开放图像模型，但对显存的要求并不高，在 8GB 显存的 GPU 上可以正常工作，覆盖了大多数消费级显卡和 GPU 云服务。

为了获得更加稳定的输出结果，我们通常借助 ControlNet，通过添加额外控制条件，来引导 Stable Diffusion 按照创作者的创作思路生成图像，从而提升 AI 图像生成的可控性和精度。目前还没有针对 SDXL 1.0 的 ControlNet 模型。好消息是，针对 SDXL 1.0 自定义数据微调模型比以往更加容易。Stability AI 团队正在构建下一代特定于任务的结构、风格和组合控件，其中 T2I / ControlNet 专门用于 SDXL，这些功能目前处于测试版预览阶段。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202307/images/sdxl1.0_06.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202307/images/sdxl1.0_07.png)

#### 使用 SDXL

有多种方法可以开始使用 SDXL 1.0：

* SDXL 1.0 已在 Clipdrop 上上线。网址：https://clipdrop.co/stable-diffusion

* SDXL 1.0 的权重和相关源代码已在 Stability AI GitHub 页面上发布。网址：https://github.com/Stability-AI/generative-models

* DreamStudio。网址：http://dreamstudio.ai/

如果要在本地部署，推荐使用 Stable Diffusion WebUI (https://github.com/AUTOMATIC1111/stable-diffusion-webui)。关于 WebUI 的部署，网上的资料多如牛毛，这里就不赘述。

前往 https://huggingface.co/stabilityai，可以看到 SDXL 1.0 的模型已经有了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202307/images/sdxl1.0_09.png)

点击 **Files and versions**，下载 sd_xl_base_1.0.safetensors 文件即可。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202307/images/sdxl1.0_10.png)

将下载到的模型文件放到 WebUI 的 models/Stable-diffusion 目录下即可。在 Web 界面上刷新并选择新模型即可。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202307/images/sdxl1.0_11.png)

#### 许可协议

可能会有人担心版权问题，这个不用担心，SDXL 1.0 根据 CreativeML OpenRAIL++-M 许可证发布。详细条款请参考：

> https://github.com/Stability-AI/generative-models/blob/main/model_licenses/LICENSE-SDXL1.0

当然这份 license 读起来很是生涩。可以看看知乎上的解读：

> https://zhuanlan.zhihu.com/p/626686691

涉及法律的条文，很难读，这里划重点：可商用，可以复制、使用和再分发。

