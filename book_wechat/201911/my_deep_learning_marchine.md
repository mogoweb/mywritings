# 这个双十一，我配了一台深度学习主机

> 然而，这并不是一篇广告。如果你也打算配一台深度学习主机，可以参考一下。

跑完了杭马，结束了杭州之旅，再无法为中断的深度学习找借口了，所以，我又回来了。研究深度学习，计算机硬件是一个门槛，虽然现在有免费的GPU云计算平台可使用，我之前也写过几篇薅资本主义羊毛的文章：

1. [巧用Kaggle进行模型训练](https://mp.weixin.qq.com/s/pFRrRuDreHp_e6t3sLTsvQ)
2. [谷歌GPU云计算平台，免费又好用](https://mp.weixin.qq.com/s/50WTuFfqrFoQTaNP3BPVPw)

但是，作为免费的云计算平台，或多或少有这样或那样的限制，练练手可以，但对于严肃的软件开发，远远不够。如果付费购买云计算服务，费用也很高。在咬了几次牙之后，决定还是配置一台深度学习主机。关于深度学习主机如何配置，上网搜索可以搜到一大堆，但是绝大部分文章都是几年之前写的。而计算机硬件的发展日新月异，以前主流的一些配置，到今天已经落伍了。接下来我就写写我对于配置深度学习主机的一些考虑。

个人比较懒，不想一块块部件的去购买和装配，所以就上京东搜索了"深度学习主机"，结果很多，接下来就是筛选。当然在做选择之前我也定了几个大件的标准：

首先，这台计算机的主要目的是为了深度学习训练模型，所以显卡要配好一些。如果能够配置多块显卡，那是最好不过。对于个人而言，4块、8块显卡的配，成本太高（土豪随意），所以选择支持两块显卡的主板比较合理。现在手头资金有限，可以先上一块显卡再说，等以后资金充裕了，再添加一块。显卡型号的选择，尽量选择主流、显存大的型号，比如GTX 1080 Ti和RTX 2080 Ti，显存均为11G，但是GTX 1080 Ti貌似已经停产，市面上流通的都是库存，甚至可能是使用过的旧显卡，最好选择RTX 2080 Ti。

其次，有关CPU的选择。通常来讲深度模型的计算主要依赖GPU，所以CPU配置差一点也没有关系。但我在日常工作中，有编译Android系统、Chromium浏览器的需求，所以强劲的CPU还是有必要，最低要求i7 CPU。

最后，就是内存和硬盘，现在内存和硬盘比较便宜，都是越大越好，考虑到性价比，内存选择32G，而硬盘采用SSD+机械硬盘的组合。SSD做系统盘，可以最大限度提升系统运行速度，相对比较贵，现在256G的价格比较亲民。机械硬盘容量大又便宜，作为数据盘，可以尽量多装一些数据集。

最后，筛选出如下的主机：

[商品]

机器配置如下：

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/my_machine_00.jpg)

虽然配置中没有机械硬盘，但预留了8个盘位，以后可以随时扩展，而且SSD免费从256G升级到512G，显卡选配，各方面都符合我的需求。参与活动后，最后以17288元的价格拿下，此外店家还提供了白条6期分期免息的优惠，特别适合我这样的穷人。

这个并非京东自营的商家，原想着可能还得几天才能到，没想到昨天下单，今天货就送到了。京东快递小哥很贴心的送到办公位上，乍一看到，吓了一跳，怎么这么大的包裹，而且超级沉，我一个人都无法从包装箱中拿出。拆开之后，才发现打包了两层，用泡沫固定主机，连机箱中也填充了海绵，主机保护的很周到。拆开通电，一次性点亮。

！[image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/my_machine_01.jpg)

** 是不是霸气侧漏 **

和我原来的开发机对比一下，这个主机的机箱实在太大了。

！[image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/my_machine_02.jpg)

主机中预装的是Windows 10系统，装上鲁大师，进行系统检测，主板是微星的，显卡是技嘉的，都算是一线品牌：

！[image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/my_machine_03.png)

作为一名程序员，最爱的就是跑分，这次也不例外，当然要用鲁大师跑跑分：

！[image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/my_machine_04.png)

能够打败全国99%的电脑，很满足！其实我的主业是软件开发，对于计算机硬件并不是太懂，如果你觉得这里面的配置哪里不合理，可以给我留言，一起探讨一下。

好了，以上就是我在双十一的收获，接下来还要安装Ubuntu Linux系统、深度学习环境，有机会再和大家分享。

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)