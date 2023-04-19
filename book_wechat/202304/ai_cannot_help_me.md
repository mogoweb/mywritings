# 这次， AI 也帮不了我

这段时间一直在改造 Chromium for Android，详情参考前文：

* [选择最新 Chromium，支持 H264 / H265](https://mp.weixin.qq.com/s/IpioXG-_NaGOc9nnKe6xbQ)
* [Chromium 改造实录：增加 MPEG TS 格式支持](https://mp.weixin.qq.com/s/enrzjVLy_VACqTKavuam7Q)
* [Chromium 改造实录：增加 MP2 音频支持](https://mp.weixin.qq.com/s/ZJjG_JYS51WM-WiY8XejVA)

在增加 TS 格式支持的时候，还参考了一下 AI 的意见：

* [工作上的问题，我问了问 AI](https://mp.weixin.qq.com/s/DuRnIfcdcFtWjpJmMNxaqA)

在增加了所要支持的音视频格式后，正准备收工时，突然发现一个更大的麻烦：RTSP 协议支持。

先简单介绍一下 RTSP 协议：

> RTSP（Real Time Streaming Protocol）是由 Real Network 和 Netscape 共同提出的如何有效地在 IP 网络上传输流媒体数据的应用层协议。RTSP 对流媒体提供了诸如暂停、快进等控制，而它本身并不传输数据，RTSP 的作用相当于流媒体服务器的远程控制。服务器端可以自行选择使用 TCP 或 UDP 来传送串流内容，它的语法和运作跟HTTP 1.1 类似，但并不特别强调时间同步，所以比较能容忍网络延迟。而且允许同时多个串流需求控制（Multicast），除了可以降低服务器端的网络用量，还可以支持多方视频会议（Video onference）。 因为与 HTTP 1.1 的运作方式相似，所以代理服务器的快取功能也同样适用于 RTSP ，并因 RTSP 具有重新导向功能，可视实际负载情况来转换提供服务的服务器，以避免过大的负载集中于同一服务器而造成延迟。

关于 RTSP 协议，了解到的情况是：

1. Chromium net 模块不支持 RTSP 协议。
2. FFmpeg 支持 RTSP 协议。
3. Chromium media 模块的视频流的网络获取是通过 Chromium net 模块，FFmpeg 只是进行 demuxer 和解码。

有了上次的经验，这次，我也求助一下 AI。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_01.png)

这个回答有点不靠谱，特别是参考链接 3，指向的是一篇关于健康的文章，显然 AI 把 Chromium 认作**铬**这种微量元素了。接着提问：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_02.png)

这个回答还可以，至少点明了修改的思路。其中链接 1 给的文章，似乎某位开发者已经修改过 Chromium net 的代码以支持 RTSP，但这篇文章明显是搬运过来的。链接 2 的文章是分析 Chromium net 源码的，对于理解 chromium 处理 HTTP 请求有所帮助。链接 4 是一个开源库，将 Chromium net 抽出来单独编译，进去看了一下，没有 RTSP 协议支持，参考价值不大。

我想看看链接 1 是否也有源码，于是继续提问：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_03.png)

AI 误会了我的意思，RTSP 这种成熟的协议，肯定有不少开源实现，我这不是为了省事吗？接着提问：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_04.png)

链接 1 和 2 实际上是重复了第一个回答，而且第二点概括错误，并没有利用 libavformat 库。链接 3 的插件模式并不适合本项目。既然中文世界没有找到答案，那有没有英文资料呢？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_05.png)

又回答偏了，再次拉到 Chromium net 上来：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_06.png)

看来真问不出什么内容了。突然冒出一个主意，绕过 Chromium net，让 FFmpeg 直接处理呢？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_07.png)

这个答案不行，接着问：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_08.png)

到此，我已经明白，再也问不出什么实质性的内容来。

前面提到一位老哥已经做了 Chromium net 中添加 RTSP 的功能，只可惜那是一篇搬运的文章，找不到原主，那就从这条线索再追问下去吧。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_09.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202304/images/bing_10.png)

看来微软的 bing 也对 CSDN 情有独钟，搜去搜来都是 CSDN 上的那几篇文章。

最后，还是抬出谷歌，在下面的链接找到了原文章：

> https://www.cswamp.com/post/41

遗憾的是，没有源码，而且貌似所使用的 Chromium 也是比较老的版本。看来只能自己啃一啃 Chromium net 的源码。

一想到协议实现就头大，没有什么捷径，必须参考 RFC 文档，按着规范来，一丝一毫都不能出错。光是 RFC 文档看起来就挺头疼，细节太多。不过目前也没有更好的办法，指望 AI 来帮忙写代码是不可能的，工作上的问题，没有条件可讲，没有条件也要创造条件上。唯一值得庆幸的是，这肯定是一条可行的道路，不像做预研，能做到什么程度心里没底。

改造 Chromium net 又会碰到哪些坑呢？欢迎围观。

