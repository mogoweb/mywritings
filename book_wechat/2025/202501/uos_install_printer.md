# 在统信 UOS 系统上安装网络打印机

在国产操作系统逐步替代 Windows 的过程中，打印机的支持成为关键因素之一，尤其是在办公场景下，打印功能的完善至关重要。

在此前的文章《[国产芯片+国产操作系统打造办公系统](https://mp.weixin.qq.com/s/6vI8zfO2okOyGR1aqNzEAg)》中，我讲过如何在统信 UOS 系统上安装本地打印机。然而，办公环境中网络打印机的使用更为普遍，因为我这边办公室用的是佳能的打印复印一体机，所以这里以佳能打印机的安装为例，介绍如何在统信 UOS 系统上安装网络打印机。此处演示基于龙芯架构的 UOS V20 系统，其他品牌或型号的打印机安装方法大同小异。

## 安装打印机驱动

由于部分打印机驱动（如佳能驱动）尚未上架应用商店，我们需要通过命令行安装驱动程序。以下是具体操作步骤：

```bash
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
安装完成后，打印服务将自动重启。

## 配置网络打印机

### 打开打印管理器

安装驱动后，打开统信 UOS 的打印管理器界面：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_01.png)

点击 **“添加打印机”** 按钮，在弹出的窗口中选择 **“手动查找”**，然后输入打印机的 IP 地址，点击 **“查找”** 按钮。如果连接成功，打印机将显示在列表中：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_02.png)

**注意**：如果打印机未显示，请检查输入的 IP 地址是否正确，以及网络连接是否正常。

### 选择连接方式

大多数网络打印机支持两种连接方式：**Socket** 和 **LPD**。一般选择 **Socket** 方式。选定后，可能会弹出“未自动匹配驱动”的提示框，无需担心，直接点击“确定”即可：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_03.png)

随后点击 **“下一步”**，进入驱动选择界面。

### 设置打印机型号

在驱动选择界面，选择 **“本地驱动”**，然后依次设置厂商和型号。您可以通过模糊匹配快速定位型号，例如输入“2006”即可找到佳能打印机对应的驱动：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_04.png)

完成后，建议打印测试页以确认打印机安装是否成功：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_05.png)

如果打印成功，请点击 **“是”**，打印机将成功添加至打印管理器列表中：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_06.png)

### 故障排查

若测试页打印失败，可以点击界面上的 **“故障排查”** 图标，根据提示排除问题：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/uos_install_printer_07.png)

## 小结

至此，网络打印机的安装过程便完成了，是不是很简单呢？随着国产操作系统生态的不断发展，将会有越来越多的打印机厂商适配国产系统。未来，国产系统必将从“可用”走向“好用”，成为更好的选择。
