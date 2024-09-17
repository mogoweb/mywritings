# 国产芯片+国产操作系统打造办公系统

在《[使用国产操作系统作为开发系统](https://mp.weixin.qq.com/s/pvwNQHaguXs2X1XnTDE7QA)》一文中，我介绍了将开发系统从 Ubuntu 替换为 Deepin 系统的过程。经过一个多月的使用，Deepin 系统已然成为我的主力开发平台，其顺手程度让我对国产操作系统的信心大增。于是，我开始将目光瞄向公司的办公电脑。

在公司，每位开发人员都有两台电脑：一台配置较高的开发机，另一台则是用于普通办公的 Mail 机。Mail 机配置相对低下，M是那种低配高价的办公电脑，而且还是不知道转手了多少道员工留下来的老古董。运行 Windows 系统时，启动速度缓慢，日常只能处理邮件，连写个文档都显得吃力。

恰好我手头有一台闲置的迷你主机，体积与电视盒相仿，原本是用作下载机的。这几年其实对高清电影兴趣不大，这台设备基本在吃灰。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_01.jpg)

这台迷你主机是几年前购买的，搭载兆芯 CPU（X86架构）和 8GB 内存，虽然在如今看来配置有些落伍，但相较于公司的 Mail 机，还是强不少。于是，我决定来个“废物利用”。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_02.png)

在操作系统方面，我选择了 UOS 专业版。与 Deepin（UOS 社区版）相比，UOS 专业版的更新频率虽然较低，但稳定性更强，特别是对外设的支持更为全面。在公司环境中，打印机是常用设备，UOS 专业版无疑更适合作为办公系统。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_03.png)

关于 UOS 系统的安装，之前我已有过详细介绍，这里不再赘述。接下来，我将分享安装一些办公相关软件的过程。

## 输入法

对于中国用户来说，电脑使用过程中离不开输入法。众所周知，在 Linux 系统下中文输入法非常搞人。比如输入法框架就有 fctix 和 iBus，一般用户哪弄得请区别。即使像我这样的 Linux 老手，在 Ubuntu 下没少折腾过输入法。直到现在，搜索 "Linux input method" 仍能看到许多用户在求助。幸运的是，UOS 专业版集成了搜狗输入法，开箱即用。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_04.png)

更让人欣喜的是，UOS 版的搜狗输入法比 Windows 版简洁许多，没有那些冗余的功能和广告，使用体验非常顺畅。

## Office 软件

办公软件我选择了 WPS Office。在 Windows 环境下，我们已经全面转向 WPS Office，相信许多中国用户也已抛弃了微软 Office。UOS 应用商店提供的是 WPS Office 专业版，如果公司有集采需求，推荐使用该版本。对于个人用户，可以前往 WPS 官网下载个人版。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_05.png)

选择 Linux 版进行下载。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_06.png)

点击**立即下载**。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_07.png)

选择 64 位 Deb 格式。下载完成后，在下载目录中双击安装包。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_08.png)

安装时系统会要求输入用户密码。对于从 Windows 转过来的用户来说，可能会觉得不适应——安装个软件还要输入密码？但反过来想，这一设计能有效防止流氓软件的偷偷安装。其实在应用商店安装软件时并不需要密码，只需点击安装即可，这是因为应用商店的应用都经过审核。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_09.png)

第一次安装可能会出现如下提示，这反映了 UOS 系统对安全的严苛设置，防止未经签名的应用程序随意安装。现在的智能手机上也有类似机制，从应用商店安装没问题，但自己下载的 App 则需要修改设置，降低安全等级。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_10.png)

点击**前往**，进入应用安全设置。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_11.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_12.png)

选择**允许任意应用**，然后再次尝试安装软件包，就可以顺利安装成功了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_13.png)

安装成功后，打开 WPS Office，文档字体齐全，不再像在 Ubuntu 下使用 WPS Office 那样遇到缺少字体的问题。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_14.png)

## 浏览器

在桌面系统下，浏览器的重要性不言而喻，许多软件系统都采用 B/S 架构。例如我在工作中使用的 OA 系统、PMS（项目管理系统）、Gerrit、文档协作系统等，均无需安装客户端，只需通过浏览器即可访问。

UOS 自带的浏览器基于 Chromium 内核，支持国密和账号同步，一般来说，完全能够满足日常办公需求。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_15.png)

如果你习惯使用特定浏览器，也可以前往应用商店下载。从谷歌 Chrome、Edge 到 Firefox，总有一款适合你。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_16.png)

## 通讯软件

在公司工作，少不了与人沟通，常用的通讯工具离不开微信和企业微信。有些公司可能使用钉钉，UOS 应用商店同样提供。

腾讯会议也是目前广泛使用的软件，特别是在疫情之后，大家习惯了线上开会，即便在公司内部，也常用腾讯会议。如今，腾讯会议对国产系统的支持力度很大，提供了原生版本。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_17.png)

## 打印机

虽然国产系统在打印机支持方面曾经备受诟病，但经过多年的努力，支持情况已有显著改善。主流打印机的支持已相对完善，可以前往 [https://ecology.chinaos.com](https://ecology.chinaos.com) 查看适配情况。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_18.png)

我们使用的是 HP 打印机，首先需要从应用商店下载惠普的打印驱动。如果不确定该下载哪个驱动，可以尝试下载一个，若不行再换另一个。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_19.png)

安装好驱动后，打开 **打印管理器**，这在 UOS 专业版中已经预装。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_20.png)

点击添加打印机后，系统会自动查找可用的打印机。我非常幸运，连接的打印机立即被找到。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_21.png)

勾选找到的打印机后，系统会要求选择驱动程序。我这里系统显示了两个驱动选项，遵循“哪个能用就用哪个”的原则，经过测试，第一个驱动能正常工作。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/uos_as_office_22.png)

安装完成后，可以打印一份测试页，以确保打印机和驱动程序正常工作。一定要进行这个步骤，因为有时候驱动虽然安装成功了，但发送打印指令后，打印机并未实际打印。例如，我开始选择 hpcups 3.18.12 这个通用驱动就遇到了这个问题。

## 小结

以上就是我将 Windows 办公电脑改造成国产芯片和操作系统组合的方案。如果你在办公中还使用了其他软件，或者使用了其他国产系统或软件，有任何心得或疑问，欢迎留言交流！
