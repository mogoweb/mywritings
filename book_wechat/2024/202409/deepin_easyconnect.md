# Deepin V23下 EasyConnect VPN 问题之解决

今天是周末，在家休息，突然想起有个文档还需要修改（苦命的打工人）。文档存放在公司内网，因此我需要通过 VPN 来访问。如今，许多公司都采取内外网隔离的部署方式，但为了满足远程办公的需求，通常需要在外部电脑上安装 VPN 软件。VPN 的主要功能是通过公共网络建立一个专用的加密通道，借助 VPN 网关实现数据加密传输和远程访问。

按照 IT 支持部提供的指南，需要下载一个 EasyConnect 软件。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_easyconnect_01.png)

令人惊喜的是，EasyConnect 支持国产的统信UOS系统，这样我就不必非得切换到 Windows 下去修改文档了。

下载完成后，双击安装，看到该软件的版本号是 7.6.7.6。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_easyconnect_02.png)

然而，安装完成后，启动 EasyConnect 却没有任何反应。大多数用户遇到这种情况可能会抱怨：”国产系统果然垃圾“。但作为一名程序员，当然要看看到底怎么回事，是软件的问题还是系统的问题。

首先，找到 EasyConnect 的启动命令：

```bash
$ cat /usr/share/applications/EasyConnect.desktop 
[Desktop Entry]
Name=EasyConnect
GenericName=vpn client
Comment=Sangfor ssl vpn client
Exec=/usr/share/sangfor/EasyConnect/EasyConnect --enable-transparent-visuals --disable-gpu
Icon=EasyConnect
Terminal=false
Type=Application
StartupNotify=false
Categories=Network;WebBrowser;
X-Desktop-File-Install-Version=0.22
```

可以看到软件的启动指令是：/usr/share/sangfor/EasyConnect/EasyConnect --enable-transparent-visuals --disable-gpu，在终端中输入该命令，提示段错误，去掉命令行参数执行，依然是段错误。

```bash
$ /usr/share/sangfor/EasyConnect/EasyConnect --enable-transparent-visuals --disable-gpu
段错误
$ /usr/share/sangfor/EasyConnect/EasyConnect
段错误
```

有经验的程序员都知道，段错误通常是内存访问错误，可能是由于程序的兼容性问题，这种问题比包依赖问题更复杂，往往只有开发方才能有效解决。

我注意到页面上还提供了 Ubuntu 的安装包，于是在 Ubuntu 上也进行了一次尝试。然而，同样无法启动，不过这次的错误信息不同：

```
Pango-ERROR **: Harfbuzz version too old(1.3.1)。
```

这种系统包依赖导致的问题也是挺头大，有可能解决了这个软件的问题，导致别的软件出问题。关于在 Ubuntu 20.04 下该问题的解决方法可以参考：

> https://blog.csdn.net/weixin_37926734/article/details/123068318

还是回到 Deepin 系统，去应用商店浏览了一圈，发现应用商店有 EasyConnect，版本 7.6.7.7，比 IT 支持部提供的版本要新。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_easyconnect_03.png)

安装应用商店的 EasyConnect 之后，一切顺利，软件启动成功。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_easyconnect_04.png)

登录 VPN 后，我能够正常访问内网和外网，并没有遇到应用商店里用户反馈的“外网无法访问”的问题。经过查阅帖子，这种问题可能是由于没有安装 iptables 包导致的。

> https://bbs.deepin.org/post/278436

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_easyconnect_05.png)

解决方法也很简单，只需执行以下命令安装 iptables：

```bash
$ sudo apt install iptables
```

当然更好的方法是软件开发方重新打包，将 iptables 加入到依赖项中，这样在安装 EasyConnect 的同时会将 iptables 包安装进系统。

大家平时在家会连公司内网吗？你们用的是什么软件？遇到过什么问题？欢迎交流！
