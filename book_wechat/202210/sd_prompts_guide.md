# 一步步指导 AI 画一幅中国山水画

在 「[AI 作画第二弹](https://mp.weixin.qq.com/s/BId5ssoAOfPMYgU6A87eEA)」这篇文章中，我给大家介绍了 AI 作画工具在 Linux 系统上的部署。如果对 Linux 系统不熟，或者显卡比较低端，也可以考虑一些在线网站。国内比较好的网站有：

* 文心一格 (https://yige.baidu.com/)
* 6pen (https://6pen.art/)
* Draft (https://draft.art/)

有了工具之后，是否就意味着能画出一幅不错的作品呢？我猜想大部分人兴致勃勃的尝试之后，会反问一句：“就这？”

如果就此浅尝辄止，那我们就会失去一项探索的机会。这如同一个人拿着单反相机，初拍的照片还比不上手机拍的。问题不在于单反相机，而在于使用的人。AI 作画工具也是如此，最重要的是要有想法，比如心中要有一个画面，希望表达什么，最后还要让 AI 理解你的想法。

在 AI 作画术语中，Prompt（文字描述）就是你和 AI 沟通的桥梁。简单说，Prompt 就是一段话，告诉 AI 你的作画需求，所以 Prompt 的好坏，决定了最终作品的是否合你心意。

看到这里，估计有人心里会犯怵。其实完全不用担心，AI 已经相当智能，你只要给几个关键词，就能画出一幅不错的作品。

下面就以一幅中国山水画为例，如何一步步指导 AI 画出一幅作品。

#### 一、指定主体

作画的第一步，就是要指定主体内容。当然什么不指定，AI 也能画，只是 AI 太全能了，各种画都能画，画出来的不见得合乎你的心意。主体内容一般就是名词，比如山、瀑布、树、风景等等。

**注：我在本机上部署的是 Stable Diffusion，只支持英文 Prompt，如果不知道英文如何表达，可以借助在线翻译。国内的在线网站都直接支持中文 Prompt，用起来更方便。**

本文的主题是中国山水画，主体就先指定为山，在 Prompt 处输入 "mountain"。下面就是生成的九张图：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_01.png)

可以看到，生成的画作更多的类似照片。这是由于 AI 训练时大多数数据来源都是各种照片，自然就会偏好写实的照片风格。这就好比让小孩画电视机，画出来大概率是平板电视，而不是我们童年时代那种电子显像管电视机，除非我们指导他画老旧的电视机。

当然 AI 学习的作品相当多，如果让 AI 不停的画下去，可能会出现几幅中国山水画，但这无异于大海捞针，所以接下来的步骤就是指定风格。

#### 二、指定风格

不同的作画工具，形成了不同的绘画风格，比如西方的油画和中国的水墨画，给人的观感就截然不同。绘画风格主要有：

* 中国水墨画 (chinese ink painting)
* 彩色中国画 (chinese watercolor painting)
* 油画 (oil painting)
* 水彩画 (watercolor painting)
* ...

这里不一一列举，也不用记，有需要上网搜索即可。加上风格限定的 Prompt 为：

> A chinese watercolor painting of a mountain

Prompt 的语法并不重要，只要带上 “chinese watercolor painting” 这样的关键词，AI就能理解。生成的九张图如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_02.png)

可以看到，生成的画作确实有国画味。特别是下面这一张，古色古香：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_03.png)

如果仔细看，画面上还有文字和印章，但生成的文字和印章完全是无法识别的字。这是因为中国画大多有题字和盖章的传统，AI 也学习到这一点，但并没有理解这上面的文字是啥含义，所以生成出像汉字但又不是汉字的画面来。尝试了几家国内的 AI 工具，同样存在这种问题。我想这也是中国创业公司的机会，如果能针对中国画进行优化，想必可以扩大受众群体。

中国山水画很少采用方形构图，一般是横卷或者竖轴。所以接下来修改画面尺寸。

#### 三、指定画面尺寸

就如同拍照，有 1:1, 2:3, 3:4, 9:16 之类的尺寸，画作同样也有不同的画面比例。中国山水画一般是长卷，所以下面就采用 1:2 (512 x 1024) 和 2:1 (1024 x 512) 的比例。当然你也可以选择其它的尺寸和比例，根据你的个人喜好选择。

画面尺寸并不是 Prompt 的一部分，而是作画参数，不管是命令行方式，还是图形界面，通常会提供这个选项。比如我的作画界面就提供宽和高的选项：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_04.png)

512 x 1024 尺寸的作品如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_05.png)

1024 x 512 尺寸的作品如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_06.png)

有几幅画作中，点缀了一些房子，但总体而言，画面不算很丰富。一般来说，中国山水画有山有水，下面就添加一些元素，丰富一下画面。

#### 四、丰富画面

我们可以往山水画中添加一些元素，如瀑布、人物，丰富一下画面。比如下面的 prompt 添加了**小男孩**、**瀑布**两个元素:

> A little boy is standing in front of a waterfall, mountain, chinese watercolor-wash

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_07.png)

添加**庙宇**元素：

> a beautiful chinese watercolor painting of the mountainous landscape of huangshan with a buddisht temple on the hilltop on a rainy day

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_08.png)

在生成一张还不错的作品后，接下来就需要生成一张高清作品。这是从生成效率和质量作的平衡，一般民用级显卡显存都不算大，如果生成超大尺寸的图片，不仅可能出现显存不足的问题，还会造成生成时间加长。AI 生成画作的质量并不十分稳定，可能需要生成多张才能挑出一幅满意的作品。目前在我的系统上，输出 512 x 512 大小的图像，大约需要 20 秒，这样一次生成 9 张图片也不到 3 分钟，还能接受。

图片放大不能仅仅通过拉伸来实现，这样得到的图像虽然像素多，但照样不清晰，这个时候就需要借助 AI 技术来实现高清放大。

#### 五、高清放大

AI 高清放大的方法有多种，我本机部署的 Stable Diffusion 本身就集成了放大功能。下面是操作界面：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_09.png)

可以选择放大 2 倍和 4 倍，这里放大 2 倍是长和宽各放大 2 倍，清晰度还是不错的。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_10.png)

还有一种方法就是在生成画作的时候在 **Advanced Options** 部分指定放大倍数：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_11.png)

但是不太建议在这里指定，这相当于每幅作品都作一个放大操作，比较费时间，毕竟 AI 生成的画作并不是每幅都令人满意，针对满意作品进行高清放大，更加高效。

#### 六、小结

综合下来，使用 AI 作画是不是非常简单？当然要创作复杂的作品，那就需要在 Prompt 和调参上花费更多的心思。我对 Stable Diffusion 的各种参数研究不多，都是使用默认参数。Prompt 则可以搜索其他高手的作品，包括：

https://lexica.art/

https://www.krea.ai/

https://openart.ai/

下面展示一下我创作的国画：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_12.png)

> A little boy is standing in front of a waterfall, mountain, trees, chinese watercolor-wash

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_13.png)

> Osmanthus fragrans, mid-shot, Chinese watercolor-wash, art by Xiagui

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_14.png)

> bamboo in strong wind, style of Chinese ink painting, art by zhengbanqiao

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_15.png)

> a river across Taishan mountains, sunrise, foggy, trees, chinese watercolor-wash

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202210/images/sd_prompts_guide_16.png)

> a beautiful chinese watercolor painting of the mountainous landscape of huangshan with a buddisht temple on the hilltop on a rainy day