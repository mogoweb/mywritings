# 动动小手，小众应用也可以在国产系统上运行起来

在上篇讨论国产操作系统生态挑战的文章中[从纯血鸿蒙允许回退版本，聊一聊国产操作系统面临的困局](https://mp.weixin.qq.com/s/dcTbuD3r02fpvW81F5SghA)，我们聊到应用移植的复杂性。但推广国产系统不能被动等待生态成熟，因此统信 UOS 等国产系统提供了​​兼容过渡方案​​，让许多未适配的 Windows 应用也能顺畅运行。

统信 UOS 官方应用商店上架了大量经过兼容性处理的软件，其中不乏小众工具。例如搜索“围棋”，会显示多个应用：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/using_wine_01.png)

这其中就有我经常使用的**野狐围棋**。实际上**野狐围棋**这款应用并没有原生适配国产系统，而是通过 ​​Windows 兼容引擎​​实现运行。商店中会明确标注“Windows 应用”标识以便识别：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/using_wine_02.png)

虽然是 Windows 应用，但是经过转成deb包，在应用商店直接下载安装，省去了用户自己还要配置兼容引擎，从命令行启动的麻烦。

从应用商店安装的 Windows 应用，有些在第一次启动时会像 Windows 系统下那样进行安装，这个一步步往下进行即可。一般应用安装后，会在桌面创建快捷方式，就像在 Windows 下那样使用，可以减少用户切换到新系统的不适应。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/using_wine_03.png)

由于人力有限，应用商店没法上架所有应用，那如果遇到自己常用的软件在应用商店搜索不到怎么办？比如野狐围棋和腾讯围棋分家后，我在两边都会去下棋，但腾讯围棋在统信UOS应用商店搜索不到。

这里还有一个办法，就是在统信UOS应用商店下载一个名为**统信Windows应用兼容引擎**的软件：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/using_wine_04.png)

安装后，打开**统信Windows应用兼容引擎**，从界面上感觉又是一个应用商店：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/using_wine_05.png)

点击其中的安装按钮，会弹出如下提示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/using_wine_06.png)

可以看出，这里无法做到一键安装，而是引导用户去网站下载。也就是说，这里只是列出经过验证，可以通过统信Windows应用兼容引擎运行的Windows应用程序，应用本身仍需要用户手动去下载，然后通过兼容引擎打开。为什么没有做成能够一键安装运行？主要出于软件版权考虑，这些应用分散在各个地方，而且下载链接可能会随着版本更新而变化，维护起来有很多工作量。如果收集起来放服务器上供用户直接下载，可能会侵权（不要和那些下载网站相提并论）。后期可能会考虑和一些第三方Windows应用商店合作，这样就可能做到在统信应用商店那样一键下载和安装，拥有更好的体验。

那如果这两个应用商店都没有的应用呢？一样可以运行。就拿腾讯围棋这个应用来说，我们先上腾讯的官网下载软件安装包Txwq_Setup.exe，然后在统信Windows应用兼容引擎中点击左边栏的**我的应用**。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/using_wine_07.png)

然后点击**添加应用**按钮，在弹出文件选择对话框中选择刚才下载的安装包文件：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/using_wine_08.png)

然后类似Windows那样安装应用程序，最后运行程序，就可以看到腾讯围棋的界面了：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/using_wine_09.png)

至此，一款小众应用就可以在 UOS/deepin 系统上用起来了。

如果你觉得自己使用的软件在统信Windows应用兼容引擎中运行得不错，还可以考虑去**统信 Windows 应用兼容引擎**官网投递，这样应用可能会纳入到统信应用商店中，别的小伙伴就可以直接从应用商店下载，如果能够因此结识一些同好，也是美事一件。详情请访问官网：

> https://wine.deepin.org/

这里有详细的文档，还有论坛交流，遇到某些小众软件运行存在兼容问题，可以在论坛中反馈，也许有人已经解决了。

## 小结

通过 ​​应用商店适配版​​、​​兼容引擎预验证应用​​ 和 ​​手动添加 EXE 文件​​ 三种方式，即使是小众 Windows 应用也能在统信 UOS/deepin 上顺畅运行。这种兼容策略显著降低了用户切换国产系统的门槛，为生态建设提供了灵活过渡方案。
