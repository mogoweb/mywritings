# AI 作画初体验

连续看了几期和菜头的公众号上关于 AI 作画的文章后，我也产生了一些兴趣。作为一名理科生，立马就行动起来，这篇文章就聊一聊我的尝试过程。

说起 AI 作画，其实已经出现好几年了。最早的新闻有：

> 2018年，10月25日，一幅由人工智能创作出的肖像画在纽约佳士得拍卖会上拍出43.2万美元的高价（约合人民币300万元）。

但之前的 AI 作画，都只存在于谷歌、NVidia这样的顶级实验室中，对硬件配置有着极高的要求，普通人想要尝试，并不太容易。

近一段时间，画家圈刷屏了一个 AI 工具：Disco Diffusion。这个 AI 工具已经进化到 V5.2 版本，具有两个特点：

1. 平民化。普通的用户级显卡就可以运行，而且获得的效果不错，速度也能接受（几分钟到几个小时，取决于显卡）。
2. 易使用。得益于 TTI（Text to Image Generator）技术的发展，人工智能开始“理解”用户输入的文本，只需要提供一些关键词，就能指挥 AI 来生成画作。

现在 AI 作画工具也是百花齐放，远不止 Disco Diffusion 这一款，不过这篇文章仅限于探讨 Disco Diffusion。

#### 在线尝试

Disco Diffusion 是一个部署在 Colab 的开源项目，所以在线就可以作图。Disco Diffusion V 5.2 Colab 地址：

> https://colab.research.google.com/github/alembics/disco-diffusion/blob/main/Disco_Diffusion.ipynb

Colab 是 Google 家的用来在线编写并运行 Python 程序的 notebook，如果使用过用 Jupyter notebook，会觉得非常熟悉。

依次运行 notebook 中的代码，就可以出图。当然 Colab 每天给免费用户使用的计算资源有限制（有资料说是 2 个小时时长，显卡资源随机），经常碰到的情况就是生成到一半，服务器就断开了。下面就是我在线尝试生成的画作：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_trial_01.png)

这是一幅未完成的作品。如果你觉得对你非常有用，可以购买谷歌的云计算资源，完成一幅完整的作品。

不过对一名程序员来说，怎么会这么容易就为云计算买单呢？

接着尝试其它的 AI 工具：CogView。这是中国之光清华大学的项目，据团队发布的论文里称“人工评估的测试中，CogView被选为最好的概率为37.02%，远远超过其他基于GAN的模型。” 

> 来源：CogView: Mastering Text-to-Image Generation via Transformers （https://arxiv.org/abs/2105.13290）


中国团队出品的产品，当然提示词原生支持中文，对中国用户比较友好。试用地址：

> https://wudao.aminer.cn/CogView/index.html

使用提示语：“一棵湖面上的樱花树，花瓣飘在天空中，湖水倒影” 生成的结果如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_trial_02.png)

一次生成 8 张，速度倒是挺快，至于说质量，要看运气。运气好的话，还是能生成不错的图像的。

再次尝试一款国外出品的在线生成工具：Dalle-Mini。试用地址：

> https://huggingface.co/spaces/dalle-mini/dalle-mini

使用 prompt："A lonely glowing door in a beautiful wilderness, by Asher Brown Durand." 生成的图像如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_trial_03.png)

这个生成的速度也很快，一次生成 9 张，但效果比起和菜头本地运行生成出来的图还是差远了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_trial_04.png)

没有办法了，还是得本地部署，虽然显卡比较渣（RTX 2080 TI），但我可以用时间换取质量。

#### 本地部署

关于 Disco Diffusion 的本地部署，网上有许多资料，谷歌也给出了本地部署的官方文档：

> https://research.google.com/colaboratory/local-runtimes.html

但作为一名程序员，我更喜欢批量化的运行，喜欢直接通过脚本搞定一切，所以我选择了通过 Docker 部署。

我的操作系统环境是 Ubuntu 20.04 LTS，NVIDIA 的驱动和 CUDA 已经安装，版本如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_trial_05.png)

照着网上的资料一步步操作，你总会遇到这样或那样的问题，这并不是作者的问题，而是这个世界变化太快。本次部署我也碰到了一些问题，所以记录一下。

1. 上 Disco Diffusion 的 Github 下载代码，其中包含 Dockerfile：

```
git clone https://github.com/alembics/disco-diffusion
```

2. 构建 Docker 镜像。
 
构建 Docker 镜像需要分两步，首先是构建 Prep 镜像，然后 main 镜像，次序不能反，因为 main 镜像依赖于 Prep 镜像。

```
cd docker/prep
docker build -t disco-diffusion-prep:5.1 .
cd ../main
docker build -t disco-diffusion:5.1 .
```

在构建 Prep 镜像时可能会碰到如下错误：

```
Step 5/16 : RUN wget --no-directories --progress=bar:force:noscroll -P /scratch/models https://v-diffusion.s3.us-west-2.amazonaws.com/512x512_diffusion_uncond_finetune_008100.pt
 ---> Running in e777eaa1659f
--2022-05-26 01:36:57--  https://v-diffusion.s3.us-west-2.amazonaws.com/512x512_diffusion_uncond_finetune_008100.pt
Resolving v-diffusion.s3.us-west-2.amazonaws.com (v-diffusion.s3.us-west-2.amazonaws.com)... 52.92.146.186
Connecting to v-diffusion.s3.us-west-2.amazonaws.com (v-diffusion.s3.us-west-2.amazonaws.com)|52.92.146.186|:443... connected.
HTTP request sent, awaiting response... 404 Not Found
2022-05-26 01:36:58 ERROR 404: Not Found.
```

原因就在于模型文件在服务器上已经不存在，研究了一下 colab 中的代码，发现还有一套 fallback 地址（备用地址），所以将 docker/prep/Dockerfile 中的地址修改为：

```
https://huggingface.co/lowlevelware/512x512_diffusion_unconditional_ImageNet/resolve/main/512x512_diffusion_uncond_finetune_008100.pt
```

如果其它的模型还存在问题，可以依葫芦画瓢，修改为备用地址即可。

不过有一个模型死活下载不下来：

```
--2022-05-26 04:47:12--  (try:20)  https://cloudflare-ipfs.com/ipfs/Qmd2mMnDLWePKmgfS8m6ntAg4nhV5VkUyAydYBp8cWWeB7/AdaBins_nyu.pt
Connecting to cloudflare-ipfs.com (cloudflare-ipfs.com)|31.13.81.4|:443... failed: Connection timed out.
Connecting to cloudflare-ipfs.com (cloudflare-ipfs.com)|2001::6ca0:a936|:443... failed: Cannot assign requested address.
Giving up.
```

没有办法，只能上网找一个，先下载下来，地址：

> https://drive.google.com/drive/folders/1nYyaQXOBjNdUJDsmJpcRpu6oE55aQoLA

然后通过 COPY 指令从 host 复制到 Docker 容器：

```
COPY AdaBins_nyu.pt /scratch/pretrained/
```

3. 启动 Docker 容器，运行 Disco Diffusion 脚本。

事先准备好 images_out 和 init_images 两个目录， 前一个目录存放生成的图像，后一个是初始图像存放的位置。然后映射到容器，这样，在 Docker 容器中生成的图像才好被 host 主机访问到：

```
docker run --rm -it \
    -v $(echo ~)/disco-diffusion/images_out:/workspace/code/images_out \
    -v $(echo ~)/disco-diffusion/init_images:/workspace/code/init_images \
    --runtime=nvidia \
    --name="disco-diffusion" --ipc=host \
    --user $(id -u):$(id -g) \
disco-diffusion:5.1 python disco-diffusion/disco.py
```

如果遇到权限问题：

```
PermissionError: [Errno 13] Permission denied: '/workspace/code/images_out/TimeToDisco'
```

可以修改一下 host 下文件的 owner 或者 mode:

```
sudo chown alex:alex ~/disco-diffusion/images_out/
```

然后就是漫长的等待。在没有修改任何参数的情况下，生成了 49 张图片，大小为 1280x768，大家可以欣赏一下其中的几张：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_trial_06.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_trial_07.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_trial_08.png)

值得一提的是，AI 程序对同一个输入，每次输出并不相同，这和传统的计算机程序不一样，所以要获得好的结果，需要多多尝试，挑选出最好的结果。

和菜头自从迷上了 AI 作画后，其公众号的题图都是自己生成，现在网上也有一些大神，通过 AI 画出了不少令人惊艳的作品。下面这篇文章尝试了古风插画，我非常喜欢：

[AI 能理解和表达古诗意境吗？来看看西乔用人工智能辅助创作的古风插画（第一辑）](https://mp.weixin.qq.com/s/oMQ_HR1dJQXLhXv-9qwizA)

你对 AI 作画怎么看，先动手尝试一下再来说说你的看法吧！

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)