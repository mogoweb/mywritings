**龙芯迷你主机，用来办公怎么样？**

在《[国产芯片+国产操作系统打造办公系统](https://mp.weixin.qq.com/s/6vI8zfO2okOyGR1aqNzEAg)》这篇文章中，我分享了用国产统信 UOS 系统替代 Windows 系统办公的体验。当时使用的是一台搭载兆芯 CPU 的迷你主机。最近关注国产 CPU 的进展时，发现有一款搭载龙芯 CPU 的迷你主机上市了，顿时心动不已。本想趁双十一薅点羊毛，搭上政府购物补贴的便车，但等了许久，这款主机始终未列入补贴名单。最终还是原价拿下——虽然说是原价，但实际上稍有优惠，原价 2799 元，使用白条 3 期免息后，2768.79 元到手。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_01.jpg)

主机到手非常快。包装盒设计简单，但上面印着的口号“为未来而来”颇有意境。龙芯默默耕耘多年，这句话似乎也传递了一丝希望和决心。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_02.jpg)

打开包装，拿出迷你主机，整体设计方正硬朗，通体黑色。中央的 Logo 简洁有力，唯一美中不足的是底部的“MOREFINE”字样，略显多余，有些破坏整体的美感。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_03.png)

主机正面布局简洁，配备电源按钮、两个 USB 3.0 接口和一个 Type-C 接口。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_04.jpg)

背面接口更为丰富：两个 HDMI、两个网口、四个 USB 接口和一个耳机插孔。双网口和双 HDMI 的设计似乎成为国产迷你主机的标配（我的上一台兆芯主机也是如此）。现在大部分情况下不都是使用无线网络吗？双网口既占空间又增加成本，不是很理解。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_05.jpg)

侧面和底部采用镂空设计，散热效果理想。但风扇运行时的噪音有些大。后面有时间测一下功耗，按道理精简指令集的 CPU 应该功耗很小，还需要风扇主动散热吗？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_06.jpg)

这款主机可以平放在桌面，也支持立式摆放，空间利用灵活。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_07.jpg)

此外，它还可挂在显示器背面，有效节省桌面空间。这点设计相当贴心，配件一应俱全。摆一张主机配件全家福。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_08.jpg)

**系统体验：开机与基本功能**

开机后，红色龙芯 Logo 显示在屏幕中央（由于光线原因，拍照略显橙色）。主机搭载统信 UOS 试用版，版本不是最新，我第一时间更新了系统。

硬件信息显示，这台主机配备龙芯 3A6000 CPU 和 7A2000 GPU，16GB 内存、256GB 固态硬盘，内置蓝牙和无线网卡。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_10.png)

能否用来办公，应用软件才是关键。打开UOS 应用商店，发现有 WPS、微信、QQ、钉钉、腾讯视频等常见办公软件，满足日常办公需求绰绰有余。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_11.png)

目前多数 OA 系统均为 Web 应用，因此浏览器的重要性不言而喻。除系统自带浏览器外，还有龙芯浏览器、360 浏览器、奇安信浏览器等多种选择。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_12.png)

**打印设备的兼容性**

打印机支持是办公电脑的关键。令人欣喜的是，我的 HP 打印机插上就识别到了，无需任何额外安装与设置，表现令人满意。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_13.png)

对于其他打印机和扫描仪，可在 UOS 应用商店的外设专区查找驱动与帮助文档，资源丰富。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_15.png)

**初步体验与小问题**

整体试用下来，这台迷你主机作为办公电脑毫无问题。不过也遇到一些小插曲，例如登录企业微信时，系统提示版本过低。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_16.jpg)

经过研究，发现微信、QQ、钉钉等均为原生版本，而企业微信则是通过 Wine 运行的 Windows 版。能用 Wine 运行 Windows 软件无疑增强了使用国产系统的信心，某些未适配的软件可通过“曲线救国”用上。企业微信版本偏低的问题也不难解决：从官网下载最新版，通过 Wine 启动即可。

**总结**

以上是这台龙芯迷你主机的初步体验分享。后续我会继续深入使用，并总结更多使用经验与技巧。你会考虑尝试使用国产芯片吗？欢迎在评论区留言讨论！
