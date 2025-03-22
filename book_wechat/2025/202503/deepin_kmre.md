# deepin V23 下运行安卓应用程序

在前一篇文章《[Linux 系统运行 Android 应用的几种方案](https://mp.weixin.qq.com/s/ZANEmvhXhelee-6mz1D0mw)》中介绍了几种在 Linux 系统上运行Android 应用的方案。其实统信 UOS/deepin 也有自己的 UEngine 方案来运行 Android 应用。UEngine 方案基于 anbox 二次开发。可惜的是 UEngine 已经不再维护，在 deepin v23 中已经移除。UOS 应用商店中的 Android 应用也越来越少。

在《[Linux 系统运行 Android 应用的几种方案](https://mp.weixin.qq.com/s/ZANEmvhXhelee-6mz1D0mw)》中也介绍过麒麟系统的开源方案 KMRE。既然是开源的方案，是不是也可以将 KMRE 移植到 deepin 上呢？答案是肯定的，而且已经有人这么做了。

原来是 GXDE 社区的大神做了相关的移植工作，按照 https://bbs.deepin.org/post/284668 这篇帖子介绍，在原版 KMRE 的基础上做出了如下修改：

* 新增支持获取 DDE/KDE 缩放比例，不再仅 UKUI 可用
* 生成的 .desktop文件添加X-GXDE-KMREAPP=true、X-GXDE-KMRE-PKGNAME标签以支持一键卸载安卓应用
* 生成的 .desktop 添加 gxme 前缀 ，避免出现与 UOS 标准包名撞车
* 支持通过 CLI 安装或卸载 apk 应用，并定义相应的返回值方便 deb 打包
* 支持双击 apk 一键安装应用
* 将 Arm 翻译层更换为 libhoudini，提升应用兼容性
* 接入 GXDE 构建系统

GXDE 社区又是何方圣神？上网查找了一下资料，原来 deepin 还有一个下游衍生版本 GXDE OS。不过随着 deepin 开始构建自己的根系统，GXDE OS 并没有跟随，依然以 Debian 为基础构建系统。

GXDE OS 是一款基于 ​Debian 的 Linux 发行版，以 ​GXDE 桌面环境为核心特色，旨在提供轻量、优雅且开箱即用的用户体验。​GXDE（Gorgeous eXtended Deepin Environment）​ 是基于 deepin 15 时期 DDE 桌面的重生版本，既保留了经典的 deepin 15 界面和操作逻辑，又新增了多项功能优化和扩展组件。

GXDE OS 依然保持着活跃的开发，由国内开发者 ​**@gfdgd_xi** 主导，星火计划提供生态建设和仓库托管。如果是 deepin 的早期粉丝，也可以尝试一下 GXDE OS 系统，该系统支持国产信创机型。

关于在 deepin V23 上如何编译运行 KMRE，按照帖子上的步骤即可。这里有三点需要注意：

* deepin V23 的内核要升级到最新版本，当前是 6.6.71-amd64-desktop-hwe，我的 deepin v23系统之前的版本是 6.6.63 版本，就提示不满足内核条件。其原因是 Kmre 的内核模块需要使用 can_nice、close_fd_get_file/file_close_fd 等原版内核没有导出的函数。上一个内核版本 6.6.63 没有导出 can_nice 函数。
* 安装源里没有的依赖包时，出现依赖错误，碰到这种问题，可以通过手动执行 apt install 命令，将缺失的包装起来。
* GXDE 的 SSL 证书是 Let‘s encrypt 颁发的，有效期很短，没有及时跟心，可能会出现证书错误。可以修改脚本：

```
diff --git a/build-kmre-deb.sh b/build-kmre-deb.sh
index 8abbce8..8ff6a73 100644
--- a/build-kmre-deb.sh
+++ b/build-kmre-deb.sh
@@ -27,8 +27,8 @@ fi
 # 判断发行版
 arch=$(dpkg --print-architecture)
 kernel_version=$(uname -r)
-gxde_repo_url="https://repo.gxde.top/gxde-os/bixie/"
-gxde_repo_list=$(curl "$gxde_repo_url/Packages")
+gxde_repo_url="http://repo.gxde.top/gxde-os/bixie/"
+gxde_repo_list=$(curl -k "$gxde_repo_url/Packages")

```

构建完重启系统，在启动器找到 ** Android 系统设置** 的图标，点击它然后等待一段时间（第一次运行需要进行初始化），看到系统设置即为成功。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/deepin_kmre_01.png)

系统还增加了一个 ** KMRE APK 安装器** 应用，用于安装安卓 apk 包，找了一个 QQ 音乐安卓版本试了一下，运行正常。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/deepin_kmre_02.png)

KMRE 能否移植到 UOS V20 上呢？估计会有些困难，从代码上看，KMRE 对内核的版本要求较高，目前未提及对 4.x 系列内核的支持，而 UOS V20 的稳定内核版本是 4.19，如果移植过去可能存在比较大的工作。不过 KMRE 对 5.x 系列内核有较好的适配，官方明确支持的版本包括 ​5.4、5.8、5.10、5.11、5.13、5.14，如果 UOS V20 切换到比较新的内核，也可能可以支持起来。

此外，从技术实现看，KMRE 依赖 Linux 容器（LXC）、硬件加速组件（如 OpenGL|ES）及内核模块（如 binder、ashmem）。这些在 4.19 和 5.x 内核上都有支持，所以移植起来应该问题不大。

有兴趣的朋友可以尝试一下，如果遇到问题，可以到 GXDE 社区和 KMRE 社区求助。

