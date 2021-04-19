# 其实，我们也在努力做CPU

全球的缺芯风潮愈演愈烈，也让很多中国人开始关注起芯片。谈到中国的国产CPU，很多人都恨铁不成钢，“泱泱大国，怎么小小的芯片也做不好？人家的芯片都做到i9了，你们怎么还只有i5的水平？”

细细品味这些批评，是否有中国家长埋汰孩子的那种味道？

> 隔壁的xxx，回回考试得第一，你怎么连前十都进不了？
>
> 我同事的儿子都考上清华了，你才考一个普通一本，以后怎么办啊？

大可不必这样，连普通的老百姓都认识到芯片的重要，国家会不知道吗？你以为国家智囊团都是吃白饭的吗？

其实，中国在多年前就已经开始布局CPU芯片，历经跌宕曲折，但总体而言还是进步很大。目前中国已经有好几家做 CPU 芯片，龙芯、鲲鹏、飞腾、申威、兆芯、海光等等，都在向不同的方向发力，如果再算上ARM芯片，厂家就更多了。

在中国人的智慧中，集中力量办大事，但现在这种乱战的局面，会不会引起资源浪费？只能说，摸索中走些弯路是不可避免的，只要不是故意骗取国家经费，即使失败了，也会培养出芯片人才。想想八九十年代，中国引进了好多条冰箱、彩电生产线，你如果去翻那个时候的报纸，也招来了很多非议，“**一哄而上，重复建设**”，不绝于耳。最后怎样呢？这些厂家之间互相竞争，变得一个比一个强悍，现在中国家电已经是世界当之无愧的No. 1。

如果是面向百姓的产品，最终还是得靠性价比取得市场。就像中国手机，可不是打爱国牌才取得目前的市场地位。

CPU同样也是如此，只有在残酷的市场中杀出来，才能生存下去。

前段时间在网上看到一款产品 - 零刻LZX，这是一款搭载了国产KX-6640MA处理器的迷你主机，价格才1000元，买回来之后接上鼠标、键盘和显示器就可以使用。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/we_work_hard_to_make_chip_01.jpg)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/we_work_hard_to_make_chip_01-2.jpg)

这台迷你主机预装了国产统信 UOS 操作系统，UOS操作系统实际上就是 Deepin Linux 系统的商业版本，而 Deepin 是 Linux 社区广受欢迎的发行版本。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/we_work_hard_to_make_chip_02.png)

如果是普通用户，对 Linux 系统不熟悉，也可以安装 Windows 10 操作系统。因为这款主机搭载的是兆芯开发的 X86 架构的 CPU，和英特尔的 CPU 完全兼容。

这款 KX-6640MA 处理器，台积电 16nm 的 CMOS 的工艺，X86 64位架构，四核四线程，主频2.2G Hz，睿频2.6G Hz。集成了核显、多媒体音频IO、DDR4内存控制器、储存和数据总线等多种功能，同时还支持4K 60赫兹分辨率的输出和视频解码。

UOS应用商店有安兔兔评测工具，运行了一下，测试结果为 167657 分，也不知道这个分数处在怎样的水平。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/we_work_hard_to_make_chip_03.png)

其它的硬件参数有：

* DDR4 2666M Hz 8G内存，最大支持64 G；
* 256G SSD硬盘，另有两个固态硬盘，一个2.5吋机械硬盘可供扩展；
* 支持双 HDMI 输出，分辨率可达4K@60Hz；
* WIFI，2.4G/5G双频；
* 双千兆以太网网口；
* 2个USB 3.0口，4个USB 2.0口；

最关键的是迷你，尺寸仅有（168×120×40）mm，放在电脑桌上完全不占位置，携带也方便。

当初买这个迷你主机是打算做个家用NAS，买回后抱着测试的目的，发现用它来写代码也挺好。作为一名程序员，使用 UOS 当然毫无压力，各种开发工具都能顺利安装。期间碰到一些问题，网上也有很多答案。因为 Deepin 本身基于 DEBIAN 系统开发，拥有丰富的软件资源，而且这个发行版本已经耕耘多年，拥有很多用户，求助也容易得到回应。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/we_work_hard_to_make_chip_04.png)

在这期间，我还使用它测试过各种格式的 4K 视频，虽然开始对其 CPU 性能还有疑虑，但测试下来，播放 4K 视频绰绰有余。所以在工作之余，我也用它来看看电影。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/we_work_hard_to_make_chip_05.png)

在我看来，在千元级的价位上，零刻LZX迷你主机还是有性价比的，在大多数情况下，我们可能也只是上上网，用 Office 处理一下文件，这样一台主机性能完全足够应付。作为程序员，也可以把它当做生产力工具，写写代码也不错，当然对于重度开发，比如机器学习、大数据、大型软件开发，这个性能肯定不够。

重度使用了一个多月，这台主机一直表现稳定，没有出现过死机、重启，应用程序使用也很流畅，可以说，​完全达到了商用水平。

我们国产芯片才刚刚起步，需要得到支持。但大家也不要被爱国情操所绑架，毕竟大家的钱也不是大风刮来的，靠爱国支撑的产品也走不远，只要大家不要带着有色眼镜看待国产CPU，一味地和世界最先进的产品对比。给时间以力量，相信我们会越做越好。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)