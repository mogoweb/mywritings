# 龙芯新世界之路，道长且阻

在上一篇[《龙芯迷你主机，用来办公怎么样？》](https://mp.weixin.qq.com/s/4WtgcfzNDiaq_9M-KG9G8A)中，我分享了用龙芯迷你主机搭配统信 UOS 进行日常办公的体验。这一体验，半年时间就过去了。半年下来，总体感觉差强人意，搭配统信 UOS 系统，日常文档编辑、网页浏览、在线视频会议等常见办公任务运行稳定。但是由于生态起步阶段，多数软件尚未提供 LoongArch 原生版本，阵容不仅不及 x86，甚至还落后于 ARM 平台。作为 Linux 生态的重要补充 Wine 应用，在龙芯上存在很多兼容问题。

最近了解到龙芯有“新世界”与“旧世界”之分，特意去做了一下功课：

> 旧世界（ABI 1.0）
> 诞生于龙芯早期过渡阶段，以 MIPS 架构代码为基础改造而成。为加速商业化推广，龙芯在保留部分 MIPS 逻辑的前提下，用 LoongArch 指令集替换了底层架构，形成了以统信 UOS、银河麒麟等商业发行版为代表的“MIPS 延伸”生态。  
>
> 新世界（ABI 2.0）
> 自 2021 年起，龙芯完全剔除历史遗留的 MIPS 组件，从指令集到软件栈全面重构，严格遵循开源社区规范。新世界架构已被 Linux 5.19 内核及主流编译器（如 GCC）原生支持，社区发行版（如 Arch Linux LoongArch 社区版）也相继涌现。  

都说要打破旧世界，拥抱新世界，作为程序员，忍不住也要尝尝鲜。

说做就做，立即行动起来，给这台龙芯迷你主机装上了 deepin v23。目前 deepin v23 龙芯版还是 preview 阶段，尝鲜可以。

安装过程很顺利，然后就进入了 deepin v23 的系统，还是熟悉的界面，系统自带的应用都还在，但没有应用商店，浏览器是 firefox。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202504/images/loongson_new_world_01.png)

没有应用商店，对于普通用户来说是麻烦了一些，但这难不倒程序员，自己动手吧。

首先想到要安装的应用是微信，微信发布了 Linux 原生版，而且还提供了龙芯架构，这点还不错。下载地址是：

> https://linux.weixin.qq.com/

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202504/images/loongson_new_world_02.png)

选择 LoongArch 版下载，但是安装时却出现如下错误：

```
uos@uos-loongson-PC:~/Downloads$ sudo dpkg -i WeChatLinux_LoongArch.deb 
请输入密码:
验证成功
dpkg: 处理归档 WeChatLinux_LoongArch.deb (--install)时出错：
 软件包体系结构(loongarch64)与本机系统体系结构(loong64)不符
在处理时有错误发生：
 WeChatLinux_LoongArch.deb
```

查了一下原因，是因为新世界和旧世界两大生态之间不兼容，旧世界的软件需通过 **liblol** 兼容层才能在新世界环境中运行。现在的龙芯架构应用基本上都是为龙芯旧世界开发的。

要在新世界上运行旧世界的应用，首先需要修改安装包的架构，这个可以通过 AOSC 社区提供的一个脚本来实现。

> https://raw.githubusercontent.com/AOSC-Dev/scriptlets/refs/heads/master/loong64-it/loong64-it.bash

这个脚本会直接修改 deb 包里的架构为 loong64，这样就可以在新世界上安装该 deb 包了。

```
uos@uos-loongson-PC:~$ chmod a+x loong64-it.bash 
uos@uos-loongson-PC:~$ ./loong64-it.bash Downloads/WeChatLinux_LoongArch.deb
```

在脚本处理的最后，会有一行提示：

```
[INFO]:  Your requested package:

    Downloads/WeChatLinux_LoongArch.deb

Has been successfully converted as a loong64 package!

However, you may still need to install libLoL for old-world applications to
work properly. Please refer to the libLoL home page:

    https://liblol.aosc.io

For details on how to install and configure libLoL.
```

也就是说还需要安装 liblol 库，才能运行。deepin v23 系统仓库已经带了 liblol 库，所以直接安装即可。

```
sudo apt install liblol
```

安装了 liblol 后，从启动菜单点击**微信**图标，又是一点反应都没有。作为程序员，当然不甘心止步如此，于是就从命令行启动微信：

```
uos@uos-loongson-PC:~$ /usr/bin/wechat
/usr/bin/wechat: error while loading shared libraries: libtiff.so.5: cannot open shared object file: No such file or directory
```

原来是缺少 so，顺便看看还缺不缺其它的库。不查不知道，一查吓一跳，好些库找不到：

```
~$ ldd /usr/bin/wechat
/usr/bin/wechat: /lib/loongarch64-linux-gnu/libdl.so.2: version `GLIBC_2.27' not found (required by /usr/bin/wechat)
/usr/bin/wechat: /lib/loongarch64-linux-gnu/libpthread.so.0: version `GLIBC_2.0' not found (required by /usr/bin/wechat)
/usr/bin/wechat: /lib/loongarch64-linux-gnu/libpthread.so.0: version `GLIBC_2.2' not found (required by /usr/bin/wechat)
/usr/bin/wechat: /lib/loongarch64-linux-gnu/libpthread.so.0: version `GLIBC_2.2.3' not found (required by /usr/bin/wechat)
/usr/bin/wechat: /lib/loongarch64-linux-gnu/libpthread.so.0: version `GLIBC_2.3.2' not found (required by /usr/bin/wechat)
/usr/bin/wechat: /lib/loongarch64-linux-gnu/libpthread.so.0: version `GLIBC_2.3.3' not found (required by /usr/bin/wechat)
/usr/bin/wechat: /lib/loongarch64-linux-gnu/libpthread.so.0: version `GLIBC_2.12' not found (required by /usr/bin/wechat)
/usr/bin/wechat: /lib/loongarch64-linux-gnu/libc.so.6: version `GLIBC_2.27' not found (required by /usr/bin/wechat)
/usr/bin/wechat: /lib/loongarch64-linux-gnu/libc.so.6: version `GLIBC_2.28' not found (required by /usr/bin/wechat)
        linux-vdso.so.1 (0x00007ffffd344000)
        libglib-2.0.so.0 => /lib/loongarch64-linux-gnu/libglib-2.0.so.0 (0x00007fffe7fa0000)
        libatomic.so.1 => /lib/loongarch64-linux-gnu/libatomic.so.1 (0x00007ffff2a04000)
        libXcomposite.so.1 => /lib/loongarch64-linux-gnu/libXcomposite.so.1 (0x00007fffe7f94000)
        libXrender.so.1 => /lib/loongarch64-linux-gnu/libXrender.so.1 (0x00007fffe7f80000)
        libXrandr.so.2 => /lib/loongarch64-linux-gnu/libXrandr.so.2 (0x00007fffe7f6c000)
        libudev.so.1 => /lib/loongarch64-linux-gnu/libudev.so.1 (0x00007fffe7f2c000)
        libandromeda.so => not found
        libconfService.so => not found
        libilink2.so => not found
        libilink_network.so => not found
        libilink_protobuf.so => not found
        libowl.so => not found
        libvoipChannel.so => not found
        libvoipCodec.so => not found
        libvoipComm.so => not found
        libmmmojo.so => not found
        libz.so.1 => /lib/loongarch64-linux-gnu/libz.so.1 (0x00007fffe7ef8000)
        libdl.so.2 => /lib/loongarch64-linux-gnu/libdl.so.2 (0x00007fffe7eec000)
        libxkbcommon.so.0 => /lib/loongarch64-linux-gnu/libxkbcommon.so.0 (0x00007fffe7e9c000)
        libxkbcommon-x11.so.0 => /lib/loongarch64-linux-gnu/libxkbcommon-x11.so.0 (0x00007fffe7e8c000)
        libxcb-glx.so.0 => /lib/loongarch64-linux-gnu/libxcb-glx.so.0 (0x00007fffe7e68000)
        libxcb-xkb.so.1 => /lib/loongarch64-linux-gnu/libxcb-xkb.so.1 (0x00007fffe7e44000)
        libxcb-randr.so.0 => /lib/loongarch64-linux-gnu/libxcb-randr.so.0 (0x00007fffe7e2c000)
        libxcb-icccm.so.4 => /lib/loongarch64-linux-gnu/libxcb-icccm.so.4 (0x00007fffe7e20000)
        libxcb-shm.so.0 => /lib/loongarch64-linux-gnu/libxcb-shm.so.0 (0x00007fffe7e14000)
        libxcb-render.so.0 => /lib/loongarch64-linux-gnu/libxcb-render.so.0 (0x00007fffe7dfc000)
        libxcb-image.so.0 => /lib/loongarch64-linux-gnu/libxcb-image.so.0 (0x00007fffe7df0000)
        libxcb-xfixes.so.0 => /lib/loongarch64-linux-gnu/libxcb-xfixes.so.0 (0x00007fffe7de0000)
        libxcb-shape.so.0 => /lib/loongarch64-linux-gnu/libxcb-shape.so.0 (0x00007fffe7dd4000)
        libxcb-sync.so.1 => /lib/loongarch64-linux-gnu/libxcb-sync.so.1 (0x00007fffe7dc4000)
        libxcb-render-util.so.0 => /lib/loongarch64-linux-gnu/libxcb-render-util.so.0 (0x00007fffe7db0000)
        libxcb-keysyms.so.1 => /lib/loongarch64-linux-gnu/libxcb-keysyms.so.1 (0x00007fffe7da4000)
        libxcb.so.1 => /lib/loongarch64-linux-gnu/libxcb.so.1 (0x00007fffe7d70000)
        libX11.so.6 => /lib/loongarch64-linux-gnu/libX11.so.6 (0x00007fffe7c18000)
        libX11-xcb.so.1 => /lib/loongarch64-linux-gnu/libX11-xcb.so.1 (0x00007fffe7c0c000)
        libfontconfig.so.1 => /lib/loongarch64-linux-gnu/libfontconfig.so.1 (0x00007fffe7bb8000)
        libdbus-1.so.3 => /lib/loongarch64-linux-gnu/libdbus-1.so.3 (0x00007fffe7b58000)
        libtiff.so.5 => not found
        libgcc_s.so.1 => /lib/loongarch64-linux-gnu/libgcc_s.so.1 (0x00007fffe7af4000)
        libpthread.so.0 => /lib/loongarch64-linux-gnu/libpthread.so.0 (0x00007fffe7ae8000)
        libc.so.6 => /lib/loongarch64-linux-gnu/libc.so.6 (0x00007fffe7938000)
        ld.so.1 => not found
        libm.so.6 => /lib/loongarch64-linux-gnu/libm.so.6 (0x00007fffe78a0000)
        libpcre2-8.so.0 => /lib/loongarch64-linux-gnu/libpcre2-8.so.0 (0x00007fffe783c000)
        /lib64/ld.so.1 => /lib64/ld-linux-loongarch-lp64d.so.1 (0x00007ffff2a14000)
        libXext.so.6 => /lib/loongarch64-linux-gnu/libXext.so.6 (0x00007fffe7820000)
        libcap.so.2 => /lib/loongarch64-linux-gnu/libcap.so.2 (0x00007fffe7808000)
        libxcb-util.so.1 => /lib/loongarch64-linux-gnu/libxcb-util.so.1 (0x00007fffe77f8000)
        libXau.so.6 => /lib/loongarch64-linux-gnu/libXau.so.6 (0x00007fffe77ec000)
        libXdmcp.so.6 => /lib/loongarch64-linux-gnu/libXdmcp.so.6 (0x00007fffe77dc000)
        libfreetype.so.6 => /lib/loongarch64-linux-gnu/libfreetype.so.6 (0x00007fffe7710000)
        libexpat.so.1 => /lib/loongarch64-linux-gnu/libexpat.so.1 (0x00007fffe76dc000)
        libsystemd.so.0 => /lib/loongarch64-linux-gnu/libsystemd.so.0 (0x00007fffe75d8000)
        libbsd.so.0 => /lib/loongarch64-linux-gnu/libbsd.so.0 (0x00007fffe75b4000)
        libbz2.so.1.0 => /lib/loongarch64-linux-gnu/libbz2.so.1.0 (0x00007fffe7598000)
        libpng16.so.16 => /lib/loongarch64-linux-gnu/libpng16.so.16 (0x00007fffe7550000)
        libbrotlidec.so.1 => /lib/loongarch64-linux-gnu/libbrotlidec.so.1 (0x00007fffe7538000)
        libgcrypt.so.20 => /lib/loongarch64-linux-gnu/libgcrypt.so.20 (0x00007fffe7444000)
        liblz4.so.1 => /lib/loongarch64-linux-gnu/liblz4.so.1 (0x00007fffe7420000)
        liblzma.so.5 => /lib/loongarch64-linux-gnu/liblzma.so.5 (0x00007fffe73e8000)
        libzstd.so.1 => /lib/loongarch64-linux-gnu/libzstd.so.1 (0x00007fffe7318000)
        libmd.so.0 => /lib/loongarch64-linux-gnu/libmd.so.0 (0x00007fffe7304000)
        libbrotlicommon.so.1 => /lib/loongarch64-linux-gnu/libbrotlicommon.so.1 (0x00007fffe72dc000)
        libgpg-error.so.0 => /lib/loongarch64-linux-gnu/libgpg-error.so.0 (0x00007fffe72ac000)
```
然后搜了一下，有些so，比如 libvoipCodec.so 等，是微信 deb 包安装的，安装在 /opt/wechat 下，有些是 /usr/lib64/，加上这些加载路径：

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/wechat：/usr/lib64/
```

但是搜遍了系统目录，都没有 libtiff.so.5 这个 so，倒是系统有一个 libtiff.so，那就建一个软链接吧。

```
sudo ln -s /lib/loongarch64-linux-gnu/libtiff.so libtiff.so.5
```

经过这一番操作，微信总算是能够启动了。虽然还有一些错误提示，但不影响功能使用。

```
$ /usr/bin/wechat 
[0429/133509.170592:ERROR:filesystem_posix.cc(63)] mkdir /home/uos/.xwechat/crashinfo: 没有那个文件或目录 (2)
pci id for fd 113: 0014:7a36, driver (null)
pci id for fd 114: 0014:7a36, driver (null)
pci id for fd 115: 0014:7a36, driver (null)
glx: failed to create dri3 screen
failed to load driver: loonggpu
pci id for fd 115: 0014:7a36, driver (null)
pci id for fd 116: 0014:7a36, driver (null)
pci id for fd 117: 0014:7a36, driver (null)
KMS: DRM_IOCTL_MODE_CREATE_DUMB failed: Permission denied
KMS: DRM_IOCTL_MODE_CREATE_DUMB failed: Permission denied
KMS: DRM_IOCTL_MODE_CREATE_DUMB failed: Permission denied
KMS: DRM_IOCTL_MODE_CREATE_DUMB failed: Permission denied
```
![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202504/images/loongson_new_world_03.png)

接下来就是写一个脚本，将上面的加载路径加载，然后修改 desktop 文件，后面再打开微信就不用从命令行了，这里不赘述了。

经过这样一番操作，终于搞定了微信的使用问题。这些操作对于程序员来说，没有难度，但是对于普通用户来说，相当不友好。在新旧交替的过渡阶段，总会是相当麻烦的事情。就如同 Windows 系统过渡到国产系统，App 的适配与兼容问题，就是花了好长时间来慢慢磨合。

这里需要提一句，liblol 是安同开源社区提供的一种解决方案，在其官方网站上有如下介绍：

> libLoL (LoongArch on LoongArch) 是一款用于提供旧世界 ABI 兼容性的运行时。旧世界 ABI 常用于为龙芯提供的 Loongnix 参考发行版和统信 UOS 设计的商业软件，如腾讯 QQ Linux 版、金山 WPS for Linux 和龙芯浏览器等。由于这些应用程序尚未移植到新世界 ABI 上，本运行时旨在为新世界发行版用户提供运行上述应用程序的便利。

龙芯公司也意识到新旧世界的切换不会是那么顺利的一件事，也提供了自己的解决方案，目前这套方案还未公开，希望龙芯能够从底层完美解决新旧世界应用的兼容问题，再也不要像我运行一个微信应用都要这么折腾了。本来龙芯架构上的应用就少，再这么折腾，更是要吓走用户。

