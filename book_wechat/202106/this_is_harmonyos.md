# 这就是鸿蒙系统？

我手头使用的是一部华为Mate 20 pro手机，快三年时间了。作为一名科技工作者，对于系统升级非常积极，每次收到系统更新通知，都会在第一时间升级。这次升级到鸿蒙系统，有些迫不及待，还是内测版本就申请进行了升级。距离升级到鸿蒙系统2.0版本已经五天时间，我最大的感受是，**没变化**。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/this_is_harmonyos_01.jpg)

升级到新的系统，数据还在，应用还在，所有的系统设置都还在，从界面到操作方式，完全没变。应用商店里的应用，也没有见少，差不多都是安卓的应用程序。安卓的程序运行在鸿蒙系统上，非常欢畅，一点也没感觉到运行在一个不是安卓的操作系统上。使用的这几天，我将我手头的app都使用了一下，没出现崩溃、无法启动、界面异常等情况。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/this_is_harmonyos_02.jpg)

要说变化，感觉运行更加流畅了。当然这只是个人感觉，没有拿软件测试，也没有进行定量的对比分析，可能是心理作用。界面也做了一些小小的调整，比如将通知消息和控制中心分开，但如果不仔细体验，基本上感知不到。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/this_is_harmonyos_03.jpeg)

前几天，媒体报道，从6月2日华为发布HarmonyOS（鸿蒙操作系统）手机系统算起，在不到一周时间里升级的手机用户数已经超过1000万。这无疑是一个非常值得称道的成绩，要知道第一批支持的手机只有Mate 40系列、P40系列、Mate 30系列等少数几个型号。另外，大多数人并不是数码爱好者，对于系统升级并不热衷。从目前舆论反馈来看，负面评价非常少。这也说明，看似没有变化的升级策略，非常有效。人虽然喜新厌旧，但对于一些习惯，却很难适应。Windows Vista因为过于激进，成为史上最差的一个Windows版本，当年 iOS 7 启用扁平化系统风格，也是遭到无数的吐槽。

鸿蒙操作系统与华为之前的安卓定制版本如此之象，对于安卓应用的支持有时如此完美，令人浮想联翩，是不是又一个套壳的安卓系统？关于鸿蒙系统是安卓套壳的言论，网络上已经很多。对于这种看法，我的观点，是不是套壳并不重要。

首先，从科学的发展历程来看，都是一步一步前进的。连牛顿这样的伟大的科学家都说过：

> 如果说我看得比别人更远些，那是因为我站在巨人的肩膀上。

软件开发领域同样如此。Linux借鉴了Unix的设计思想，在接口等方面也与Unix保持一致，用户可以从Unix系统无缝切换到Linux系统。安卓系统并没有从头开发，而是选择了Linux内核，在Linux内核的基础上构建了一套完善的系统框架。

更重要的是，安卓的大部分代码都是采用了Apache许可协议。只要阅读过Apache许可协议的人就知道，这个许可协议相当的宽松，可以修改源码，修改的源码可以闭源，也可以更改为其它的许可协议，还可以用于商业用途。所以基于安卓开发新的手机操作系统完全是可行的，也不存在卡脖子的问题，除非某一天谷歌选择将安卓闭源。即使那样，现在的代码依然可以用。

其实Google和苹果也做过这样的事情。比如浏览器内核，苹果工程师在KHTML的基础上开发了新的内核WebKit。谷歌工程师先是参与WebKit内核的开发，用于安卓系统和Chrome浏览器。后来翅膀硬了之后，同苹果分道扬镳，从WebKit分叉出一个blink内核。这样的项目还有很多，比如boringSSL就是OpenSSL的一个分叉版本。可见先基于现有软件开发，等有实力之后，独立开发出自有的分支，完全可行。

很多中国人都有一些道德洁癖，就和很多男人的处女情结一样，见不得一点国外的技术参杂在里面。这些年来，中国一直把自主知识产权挂在嘴边，但多少人又真正懂得什么是自主知识产权呢？

首先需要明白的一点是，专利保护的是思想，而不是实现。也就是说，即使把安卓重写一遍，同样可能面临专利问题。微软写了一行的安卓代码了吗？为什么因安卓系统每年要向微软支付几十亿的专利费？还有前不久结束的谷歌和甲骨文关于Java的世纪官司，虽然最终谷歌赢得了官司，但这还是能说明，**不是重写代码就能规避专利问题的**。

有些基础专利，是很难绕过去的。比如，如果”鼠标之父“为鼠标申请了专利，不管把鼠标做成方的、还是圆的，都规避不了。还有一些塞到标准中的专利，也很难绕开。如果不遵循标准，你没法和别人的产品互通，如果遵循了标准，专利就没法绕开。这也是为啥那些科技巨头都非常积极把自己的技术纳入到国际标准中去的原因。当年华为在通信领域和爱立信、诺基亚、朗讯、思科等公司竞争时，就碰到这样的困境。没有办法，只能按照别人的规则来玩，每年要付不少的专利费。但只要不下牌桌，就有机会。华为积极的申请专利，加入各种国际标准组织，最终手中持有的专利越来越多，谈判的筹码有了，就可以进行专利的交叉授权，不再受制于人。目前，在 5G 通信领域，华为还可以向别的跨国公司收取专利费。

说句实话，在软件领域，操作系统也研究这么多年，你能想到的方法，能写成专利的，别人早就捷足先登。除非有石破天惊的大突破，但从文明的进程来看，跳跃式发展从来都是偶然的。

所以说，套不套壳不重要，重要的是能够把现有的操作系统（不管是Linux，还是安卓）吃透，拥有自我发展的能力。至于创新，重点并不在于颠覆现有技术，而是多积累一些新的专利，持有的专利越多，谈判的筹码也就越多。

现在Harmony OS 2.0的源码也开放出来了，是不是套壳，大家可以去看看。当然，仍然有人质疑开放出来的源码和华为使用的系统并不是同一套代码。这个既无法证实，也无法证伪，我选择保持沉默，让子弹飞一会，先不着急下结论。

要说，鸿蒙系统前途一片光明？这我还真不是这样认为，当前鸿蒙面临的生态建设问题依然很大。

1. 鸿蒙系统对安卓系统的兼容性太好，让人感觉不到换了一个操作系统。这对于用户而言是一件好事，但对于软件开发商而言，就更没有动力去开发鸿蒙版应用。本来开发者为了跨桌面、安卓、iOS系统开发，就设计了不少的方案，就是为了减少开发和维护成本，现在多了一个鸿蒙系统要适配，而安卓app既可以在鸿蒙上运行，又能在安卓系统上跑，为什么要开发两套？华为应用商店里的鸿蒙应用只有几款，比如新浪新闻、央视影音，功能和界面都很简陋。

2. 鸿蒙应用开发有些门槛，目前支持Java、JS和C/C++，看起来支持的不错。但Java系统接口和安卓存在一定的差异，将安卓应用改写成鸿蒙应用还有一定的工作量。JS开发的门槛虽然很低，但鸿蒙的JS和通常的前端开发还有些区别，很多前端框架还不支持。至于C/C++开发应用，其难度更大，不可能成为应用开发的主力。更为关键的是，目前鸿蒙应用开发的资料很少，组织的也不好，完全不比不上微软、谷歌和苹果等公司出的文档。这一点情有可原，毕竟华为以前主要是做硬件的，做软件也是赶鸭子上架。

3. 鸿蒙系统如果只是华为一家玩，肯定是玩不下去。华为硬件方面的业务受到制裁，就靠一些存量设备，没法支撑下去。发动友商支持，也不太现实，毕竟存在产品线的竞争。如果破这个局，也不是我这个级别的人能想到的，只能静观其变吧。

反正不管怎么看，鸿蒙系统都很难，但也不能失去信心。中国每前进一步，都很难，但我们还是做到了。所以，这里还是要给鸿蒙系统加油。

在研究鸿蒙系统的同时，我也拿到了鸿蒙应用开发的中级证书，等想好做什么应用的时候，再来试一试。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/this_is_harmonyos_04.png)