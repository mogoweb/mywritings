# Deepin V23下 EasyConnect VPN 问题之解决

今天周末在家，忽然想起有个文档需要修改一下（苦命的打工人）。文档在公司内网，这种情况下需要通过 VPN 来访问。现在很多公司都这样部署，内网和外网隔离，但又要满足在外随时随地能工作的需求，这个时候需要在外网电脑上安装一个 VPN 软件。VPN 的功能是在公用网络上建立专用网络，进行加密通讯，VPN 网关通过对数据包的加密和数据包目标地址转换实现远程访问。

按照 IT 支持部提供的指南，需要下载一个 EasyConnect 软件。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_easyconnect_01.png)

非常惊喜的是，EasyConnect 这款软件支持国产的统信UOS系统，这样我就不必非得切换到 Windows 下去修改文档了。

下载之后双击安装，可以看到，该软件的版本号是 7.6.7.6

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_easyconnect_02.png)

安装成功之后，启动 EasyConnect，啥反应没有。一般用户这个时候可能会开喷：”国产系统果然垃圾“。但作为一名程序员，当然要看看到底怎么回事，是软件的问题还是系统的问题。

首先需要找出 EasyConnect 软件的启动命令：

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

可以看到软件的启动指令是：/usr/share/sangfor/EasyConnect/EasyConnect --enable-transparent-visuals --disable-gpu，在终端中输入该指令，提示段错误，去掉命令行参数执行，依然是段错误。

```bash
$ /usr/share/sangfor/EasyConnect/EasyConnect --enable-transparent-visuals --disable-gpu
段错误
$ /usr/share/sangfor/EasyConnect/EasyConnect
段错误
```

有经验的程序员知道，这个段错误就是内存访问错误，应该是程序兼容性出了问题，这个问题比什么包依赖导致的错误还比较难查，只有软件开发方才好定位。

看到页面上还提供了 Ubuntu 的软件包，那在 Ubuntu 上也试试。安装之后，一样无法打开，不过在 Ubuntu 上提示的错误信息不一样。

```
Pango-ERROR **: Harfbuzz version too old(1.3.1)。
```

这种系统包依赖导致的问题也是挺头大，有可能解决了这个软件的问题，导致别的软件出问题。关于在 Ubuntu 20.04 下该问题的解决方法可以参考：

> https://blog.csdn.net/weixin_37926734/article/details/123068318

还是回到 Deepin 系统，去应用商店浏览了一圈，发现应用商店有 EasyConnect，版本 7.6.7.7，比 IT 支持部提供的版本要新。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_easyconnect_03.png)

安装应用商店的 EasyConnect 之后，Bingo，一次性成功。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_easyconnect_04.png)

登录上 VPN 之后，内网和外网都能正常访问，并没有出现应用商店有用户反馈的外网无法访问的问题。关于这个问题，有个帖子探讨了这件事情，应该是由于 iptables 这个包没有安装的缘故。

> https://bbs.deepin.org/post/278436

解决的方法也简单，就是使用命令安装一下：

```bash
$ sudo apt install iptables
```

当然更好的方法是软件开发方重新打包，将 iptables 加入到依赖项中，这样在安装 EasyConnect 的同时会将 iptables 包安装进系统。

你们在家会连接公司内网吗？使用什么软件，遇到过什么问题？欢迎探讨。
