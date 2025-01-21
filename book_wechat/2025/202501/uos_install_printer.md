# 统信UOS系统安装网络打印机

在国产系统替代 Windows 系统的过程中，打印机的支持非常关键，特别是对于办公场景下，能否支持打印机尤为重要。

在前文《[国产芯片+国产操作系统打造办公系统](https://mp.weixin.qq.com/s/6vI8zfO2okOyGR1aqNzEAg)》中，我为大家介绍了如何在统信 UOS 系统上安装本地打印机。但办公环境下，网络打印机更为常见，因此本篇将介绍如何在统信 UOS 系统上安装网络打印机。

因为我这边办公室用的是佳能的打印复印一体机，所以这里以佳能打印机的安装为例，使用的是龙芯架构的 UOS V20 系统。如果用的是其他品牌的打印机，安装步骤大同小异。

## 安装打印机驱动

由于佳能打印机的驱动未上架应用商店，因此需要通过命令行安装。

```
alex@alex-loongson-MiniPC:~$ sudo apt install com.canon.ufr2
请输入密码:
验证成功
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
下列【新】软件包将被安装：
  com.canon.ufr2
升级了 0 个软件包，新安装了 1 个软件包，要卸载 0 个软件包，有 1 个软件包未被升级。
需要下载 7,706 kB 的归档。
解压缩后会消耗 110 MB 的额外空间。
获取:1 https://pro-driver-packages.uniontech.com eagle/non-free loongarch64 com.canon.ufr2 loongarch64 5.90-1.4 [7,706 kB]
79% [1 com.canon.ufr2 7,602 kB/7,706 kB 99%]                                                                                                                                                           117 kB/s 0秒106e38664edf1f6322838cd158d24895  /var/cache/apt/archives/partial/com.canon.ufr2_5.90-1.4_loongarch64.deb
106e38664edf1f6322838cd158d24895  /var/cache/apt/archives/partial/com.canon.ufr2_5.90-1.4_loongarch64.deb
已下载 7,706 kB，耗时 1分 44秒 (74.3 kB/s)                                                                                                                                                                          
正在选中未选择的软件包 com.canon.ufr2。
(正在读取数据库 ... 系统当前共安装有 240608 个文件和目录。)
准备解压 .../com.canon.ufr2_5.90-1.4_loongarch64.deb  ...
正在解压 com.canon.ufr2 (5.90-1.4) ...
/var/cache/apt/archives/com.canon.ufr2_5.90-1.4_loongarch64.deb
正在设置 com.canon.ufr2 (5.90-1.4) ...
[ ok ] Restarting cups (via systemctl): cups.service.
正在处理用于 deepin-app-store (7.8.3.0103-1) 的触发器 ...
Rebuilding /usr/share/applications/bamf-2.index...
正在处理用于 desktop-file-utils (0.23-4+sign) 的触发器 ...
正在处理用于 mime-support (3.62+sign) 的触发器 ...
正在处理用于 libc-bin (2.28.31-deepin1) 的触发器 ...
正在处理用于 bamfdaemon (0.5.4.2-1) 的触发器 ...
Rebuilding /usr/share/applications/bamf-2.index...
```

## 打印管理器

驱动安装完毕后，打开打印管理器。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_01.png)

点击`添加打印机`按钮，然后选择`手动查找`，输入打印机的 IP 地址，然后点击`查找`按钮。如果一切顺利，打印机会显示在列表中。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_02.png)

如果打印机没有显示在列表中，请确保打印机的 IP 是否正确，以及网络是否通畅。

一般网络打印机有两种：socket 和 lpd，一般选择 socket 的那个即可。点击之后，会弹出一个提示框，未自动匹配上，没有关系，点击确定即可。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_03.png)

然后再点击下面的`下一步`按钮。在出现的界面中，选择`本地驱动`，然后依次选择厂商、型号，型号可以用模糊匹配，比如我这边输入 2006，就匹配到了我的打印机。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_04.png)

然后可以选择打印测试页，测试一下打印机是否安装成功。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_05.png)

如果打印成功，点击界面上的`是`，打印机就安装成功了，这时打印管理器中就会显示这台打印机。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_06.png)

如果打印失败，可以点击界面上的`故障排查`图标，排查一下问题。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_07.png)

至此，网络打印机安装完毕。是不是非常简单呢？随着国产操作系统生态的不断完善，相信未来会有越来越多的打印机厂商适配国产系统。国产系统也会越来越完善，越来越好用。真正从可用走向好用。