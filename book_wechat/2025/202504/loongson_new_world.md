# 龙芯新世界之路，道长且阻

在上一篇[《龙芯迷你主机，用来办公怎么样？》](https://mp.weixin.qq.com/s/4WtgcfzNDiaq_9M-KG9G8A)中，我分享了用龙芯迷你主机搭配统信 UOS 进行日常办公的体验。这一体验，半年时间就过去了。半年下来，总体感觉差强人意，搭配统信 UOS 系统，日常文档编辑、网页浏览、在线视频会议等常见办公任务运行稳定。但是由于生态起步阶段，多数软件尚未提供 LoongArch 原生版本，阵容不仅不及 x86，甚至还落后于 ARM 平台。作为 Linux 生态的重要补充 Wine 应用，在龙芯上存在很多兼容问题。

最近了解到龙芯有“新世界”与“旧世界”之分，特意去做了一下功课：

> 旧世界（ABI 1.0）
> 诞生于龙芯早期过渡阶段，以 MIPS 架构代码为基础改造而成。为加速商业化推广，龙芯在保留部分 MIPS 逻辑的前提下，用 LoongArch 指令集替换了底层架构，形成了以统信 UOS、银河麒麟等商业发行版为代表的“MIPS 延伸”生态。  
>
> 新世界（ABI 2.0）
> 自 2021 年起，龙芯完全剔除历史遗留的 MIPS 组件，从指令集到软件栈全面重构，严格遵循开源社区规范。新世界架构已被 Linux 5.19 内核及主流编译器（如 GCC）原生支持，社区发行版（如 Arch Linux LoongArch 社区版）也相继涌现。  

“打破旧世界，拥抱新世界”——作为程序员，自然要第一时间尝鲜，于是立刻将这台龙芯迷你主机安装了 Deepin V23 龙芯版（目前仍为 Preview 版本，但足够试水）。

## 初探 Deepin V23 龙芯版

安装过程十分顺利，进入系统后依旧是熟悉的 Deepin 界面，内置应用齐全，只是暂未集成应用商店，浏览器默认 Firefox。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202504/images/loongson_new_world_01.png)

对于普通用户来说，没有应用商店确实略显不便，但对程序员而言，这正是动手的好时机。

## 第一战：安装微信

首先想要体验的当然是微信。好在腾讯已经发布了 Linux 原生版，并且提供了 LoongArch 架构支持。登录官网下载 LoongArch 版安装包：

> https://linux.weixin.qq.com/

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202504/images/loongson_new_world_02.png)

选择 LoongArch 版下载，下载后执行：

```
sudo dpkg -i WeChatLinux_LoongArch.deb
```

却报错提示：

```
uos@uos-loongson-PC:~/Downloads$ sudo dpkg -i WeChatLinux_LoongArch.deb 
请输入密码:
验证成功
dpkg: 处理归档 WeChatLinux_LoongArch.deb (--install)时出错：
 软件包体系结构(loongarch64)与本机系统体系结构(loong64)不符
在处理时有错误发生：
 WeChatLinux_LoongArch.deb
```

原来，新世界（ABI 2.0）与旧世界（ABI 1.0）不兼容——旧世界软件需要通过 **libLoL** 兼容层才能在新世界环境中运行。眼下，绝大多数龙芯架构应用都只面向旧世界 ABI 开发。

### 使用脚本修改包架构

通过 AOSC 社区提供的脚本，可将旧世界的 .deb 包内部架构由 loongarch64 修改为 loong64：

> https://raw.githubusercontent.com/AOSC-Dev/scriptlets/refs/heads/master/loong64-it/loong64-it.bash

这个脚本会直接修改 deb 包里的架构为 loong64，这样就可以在新世界上安装该 deb 包了。

```
# 下载并赋予执行权限
chmod a+x loong64-it.bash

# 执行脚本转换安装包
./loong64-it.bash Downloads/WeChatLinux_LoongArch.deb
```

脚本运行完毕，会提示：

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

### 解决依赖：补齐缺失的共享库

安装了 liblol 后，从启动菜单点击**微信**图标，又是一点反应都没有。作为程序员，当然不甘心就此止步，于是就从命令行启动微信：

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
然后搜了一下，一些私有库，比如 libvoipCodec.so 等，是微信 deb 包安装的，安装在 /opt/wechat 下，有些是 /usr/lib64/，需要将它们加入加载路径：

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/wechat：/usr/lib64/
```

但是搜遍了系统目录，都没有 libtiff.so.5 这个 so，倒是系统有一个 libtiff.so，那就建一个软链接吧。

```
sudo ln -s /lib/loongarch64-linux-gnu/libtiff.so libtiff.so.5
```

经过以上配置，微信终于可以启动。尽管终端仍会打印一堆警告和错误信息，但不影响功能。

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

最后，我还编写了一个简单脚本，自动设置环境变量并更新桌面快捷方式，这样就可以直接从应用菜单启动微信，不必每次都输入命令。

## 体验小结

经过这样一番操作，终于搞定了微信的使用问题。这些操作对程序员而言并不复杂，但对于普通用户就极不友好。新旧世界的过渡，注定是一段坎坷历程。正如国内操作系统要从 Windows 生态切换到国产发行版，App 适配与兼容问题同样需要漫长时间来磨合。

值得一提的是，libLoL 是安同开源社区提供的旧世界 ABI 兼容解决方案，其官方说明写道：

> libLoL (LoongArch on LoongArch) 是一款用于提供旧世界 ABI 兼容性的运行时。旧世界 ABI 常用于为龙芯提供的 Loongnix 参考发行版和统信 UOS 设计的商业软件，如腾讯 QQ Linux 版、金山 WPS for Linux 和龙芯浏览器等。由于这些应用程序尚未移植到新世界 ABI 上，本运行时旨在为新世界发行版用户提供运行上述应用程序的便利。

龙芯公司也意识到新旧世界的切换不会是那么顺利的一件事，也提供了自己的解决方案，目前这套方案还未公开，希望龙芯能够从底层完美解决新旧世界应用的兼容问题，再也不要像我运行一个微信应用都要这么折腾了。本来龙芯架构上的应用就少，再这么折腾，更是要吓走用户。

## 延伸知识

1. 旧世界与新世界: https://areweloongyet.com/docs/old-and-new-worlds/
2. libLoL: 为您在新旧世界之间架起桥梁！ https://liblol.aosc.io/