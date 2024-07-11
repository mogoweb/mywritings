# 使用国产操作系统作为开发系统

长期以来，我一直是在 Ubuntu 系统上做开发。近一年来，我们为信创系统（统信 UOS、银河麒麟等）开发应用软件，免不了使用国产操作系统。经过近一年的接触，发现国产系统在易用性、稳定性方面已经相当不错，而且用户界面比起 Ubuntu 还美观很多。系统集成的应用商店，里面的应用非常全面，基本上满足了作为系统开发的需求。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/using_deepin_01.png)

某一天，一个念头出现在我的脑海，何不使用国产操作系统作为开发机系统？说干就干。我立即上京东买了一块 500G 的固态硬盘，作为系统盘。至于数据盘，和原来的 Ubuntu 系统共用。之前在 Ubuntu 系统下下载了不少大模型和源码（Chromium、Android等），在新系统可以直接使用。在**统信 UOS** 和**银河麒麟**之间，我选择了统信 UOS，主要是统信 UOS 背后的社区（Deppin Community）更加活跃。

统信 UOS 有多个版本，我选择社区版（Deepin系统）。一般来说社区版本更新比较快，社区更加活跃，作为一名专业开发人员，有问题也方便去社区交流。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/using_deepin_02.png)

Deepin 是一个基于 Linux 的操作系统，早期基于 Debian 构建。从 2022 年开始，Deepin 脱离 Debian 社区，从 Linux kernel 和其他开源组件而构建，不依赖上游发行版社区，研发全新架构的Deepin 23版本。当前 Deepin V23 还是 RC2 版本，离正式发布不远，所以我选择了这个最新版本。

虽然 Deepin v23 不再基于 Debian 系统，但其依然沿用了 Debian 的那套包管理系统，所以从 Ubuntu 切换过来，非常顺畅。

Deepin 的安装过程相对简单，从官方网站下载 ISO 镜像文件，制作启动 U 盘进行安装，这里不赘述。

值得注意的是，在安装方式那一部，要选择高级安装，自己选择分区划分。如果选择推荐的全盘安装，Deepin 系统会自动进行分区，但是分给根分区的空间过小（只有 15G），这对于开发来说远远不够。有很多软件，特别是 deb 包，会将程序和库安装在 /usr 目录下，如果空间不足，会导致安装失败。我选择将 500 G 的空间，除了 32 G 的交换分区，其它全部挂载在根分区。

## CUDA 的安装



## 安装Deepin