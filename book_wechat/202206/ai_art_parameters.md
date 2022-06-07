# AI 作画调参说明

在上一篇文章《[AI 作画初体验](https://mp.weixin.qq.com/s/-oVd2ZnId4BSGC2Q2emQYA)》中给大家介绍了一款 AI 作画工具 DD （Disco Diffusion） 及其本地部署方法。初次尝试，感觉 DD 生成的画作效果还不错，就是每次运行的时间比较长，为此花了一些时间研究如何提高 DD 作画的效率。

注意本文并不是探讨如何优化算法或者优化 GPU 来提高效率，这里面的水太深，不是我等普通程序员能够驾驭得了的事情。这里探讨的是通过调整一些参数，在牺牲一些体验的基础上提高生成一幅画的时间。

DD 包含很多参数，用来调整图像的生成过程，参数可以在 Disco_Diffusion.ipynb 的 **3. Settings** 部分找到:

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_parameters_01.png)

还有一部分运行控制参数位于 **4. Diffuse!** 部分：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_parameters_02.png)

如果是通过 Docker 部署，可以在 **disco.py** 文件中找到对应的参数。

关于 DD 详细的参数调整说明可以参考这篇文档:
> https://docs.google.com/document/d/1l8s7uS2dGqjztYSjPpzlmXLjl5PM3IGkRWI3IiCuK7g/edit)

这里仅介绍几个常用的参数。

* n_batches: (50|1-100) 
  
  这个变量决定 DD 生成的静态图片的数量，默认值是 50，也就是说跑一次一下子给你作 50 张画。其实大部分时候我们并不需要这么多，如果把这个值改为 1 ，其它参数不变，在我的 RTX 2080 TI 的显卡上，创作一张画的时间大约为 7 分钟，这算是一个比较能让人接受的时长。在 Google Colab 上运行，时间大约为 12 分钟，无需购买 GPU 资源也能作画。

  那为何会设计这样一个参数呢？主要是因为 AI 作图，有时会创作出一些莫名其妙的作品，生成多张，可以从中挑选满意的一张。如果显卡性能强劲，可以考虑增加这个值，一般设置为 9 就足够了。

  需要注意的是，DD 还能生成视频，如果是生成视频，这个参数会被忽略。

* width_height: (\[1280,768\])

  该变量决定最终生成的图像大小，以像素为单位，默认值是 1280x768。在手机摄像头动辄千万像素的年代，这个分辨率似乎有点低，但对于 AI 而言，[1024x768] 都是相当大的图像。如果这个尺寸设置过大，可能会导致 OOM（内存不足）错误，导致 DD 崩溃！可以从 [512x768] 开始尝试，如果运行没问题，再增大。比如在我的 2080 TI 显卡上，使用默认值运行起来完全无压力。

  AI 创作的图像可以是正方形、或宽或高，但每个边应设置为 64 像素的倍数。当然，如果设置的尺寸不是 64 像素的倍数，DD 将调整图像的尺寸变为 64 的倍数。

  如果你希望产生更高清的图像，可以借助于其它的 AI 图片放大工具。

* steps: (250|50-10000) 
  
  AI 生成图像有点类似负反馈系统，系统先生成一个图像，然后评估，引导图像生成的“方向”，再次生成图像，如此轮回。每一个轮回就是一个 step。增加步长将为 AI 提供更多调整图像的机会，并且每次调整幅度都会更小，从而产生更精确、更细腻的图像。

  当然，增加 steps 是意味着更长的渲染时间。这个值的选择取决与你希望生成图像的质量以及图像的复杂程度，没有一个固定值。一般而言，使用默认值就是一个比较好的选择。

  如果你只是验证提示词（prompts）和生成画作的相关度，可以先把这个值设置小一些，等提示词设计确定下来，再来增加这个参数值，以生成更精确的图像。

* cutn_batches: (4|1-8) 
  
  每次迭代，AI 将图像分割成小块，并将每个小块与提示（prompts）进行比较，以决定如何指导下一个生成步骤。更多的小块通常可以产生更好的图像，因为 DD 在每个时间步有更多的机会微调图像精度。
 
  小块划分得越多，内存消耗越大。
  
  在默认设置下，DD 每个步骤执行的切割数量为 cutn_batches x 16。如果 cutn_batches 为 4，DD 将分为 4 个连续批次，每批次切割 16 块，因为一次只评估 16 个小块，DD 只使用 16 个小块所需的内存，但提供 64 个小块的质量优势，但带来的不足之处是，渲染每个图像需要大约 4 倍的时间。
 
  所以，增加 cutn_batches 会增加渲染时间，因为工作是按顺序完成的。

* skip_augs：(False)

  DD 使用了一些视觉效果增强技巧，在图像创建过程中引入随机图像缩放、透视和其他调整方法，以提高图像质量。这样产生的图像更加自然、边缘更加平滑。通过将 skip_augs 设置为 True，可以跳过这些增强，这样可以稍微加快渲染速度。

* display_rate: (50|5-500)

  前面讲到，DD 是通过多次迭代生成最终图像，在运行期间，DD 提供一个机会让你监控正在创建的每个图像。 如果 display_rate 设置为 50，DD 将每 50 步显示一次图像。
 
  将此值设置为较低的值，例如 5 或 10，可以仔细观察出图像是如何一步一步生成的。如果你对中间过程不敢兴趣，可以将 display_rate 设置为等于前面的 steps 参数值，这样可以稍稍提高在 Colab 中运行的速度，毕竟显示图像也会花费一点时间。


随后，我尝试了几种参数组合对图像生成速度的影响（本地部署，显卡 RTX 2080 TI ）。

第一次，我只修改了 n_batches 参数值，将其设为 1，耗时 07:18，得到如下图像：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_parameters_03.png)

第二次，我修改的参数 n_batches=1 cutn_batches=1，结果耗时 04:55，得到如下图像：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_parameters_04.png)

第三次，我修改的参数 n_batches=1 cutn_batches=1 skip_augs=True steps=150，耗时降低到 02:45，得到如下图像：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_parameters_05.png)

通过这些尝试，发现适当降低画作的质量，时间就能显著降低。在家用级显卡上，不到三分钟就能渲染出一幅还算不错的作品，这也给了大众参与创作的机会。而且通过参数调整，我们也可以借助于 Colab 创作，无需购买谷歌的 GPU 运算资源。

后续我会研究一下 提示语（prompts）对创作的影响，尝试创作一些古风山水画。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)
