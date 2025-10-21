# 如何开启 NTSync？开启 NTSync 能带来怎样的性能提升？

在上一篇文章《[Wine 10.16 发布：NTSync 让 Windows 应用运行更丝滑了，但还得等等](https://mp.weixin.qq.com/s/qnpYMRdXnFOzG7AuxtFqSQ)》中，我们介绍了 Wine 10.16 开始正式支持 NTSync。不过，NTSync 功能依赖于 Linux 内核模块，因此并不是所有 Linux 系统都能直接启用。

根据 ChatGPT 的回答，Linux 6.6 开始加入了 NTSync。但我后来专门查阅了 Wine 的提交记录，发现实际上是 **Linux 6.14 内核才正式合入了 NTSync**。这两者并不矛盾——Linux 内核对新特性的合入通常比较谨慎，新功能往往会先在多个版本中以可选模块的形式存在，经过充分测试后才正式进入主线代码。因此，**在 6.14 以下版本中，若发行版启用了该模块，也可能已经具备支持**。

下面以 **deepin Linux V25** 和 **Ubuntu 25.10** 为例，介绍如何开启 NTSync。

## deepin V25 上开启 NTSync

首先进入 `控制中心 → 系统 → 关于本机`，查看当前内核版本：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202510/images/turn_on_ntsync_01.png)

然后在终端中确认内核是否已启用 NTSync：

```bash
$ sudo grep -i NTSYNC /boot/config-6.12.41-amd64-desktop-rolling
CONFIG_NTSYNC=m
```

输出结果表明 **deepin Linux 内核已启用 NTSYNC 功能**，但以模块（module）的形式构建，而非直接编译进内核。
默认情况下，该模块并不会自动加载，因此可以手动加载：

```bash
$ sudo modprobe ntsync
$ lsmod | grep -i ntsync
ntsync                 16384  0
```

接着查看设备节点：

```bash
$ ls /dev/ntsync 
/dev/ntsync
```

若能看到 `/dev/ntsync` 设备节点，即表示 NTSync 模块已成功加载并启用。

## Ubuntu 25.10 上开启 NTSync

同样，先查看系统内核版本：

```bash
$ uname -r
6.17.0-5-generic
```

检查是否已启用 NTSync 模块：

```bash
$ grep -i NTSYNC /boot/config-6.17.0-5-generic 
CONFIG_NTSYNC=m
```

可以看到，即使是 Linux 6.14 以上版本，NTSync 也未必默认启用，需要手动加载模块：

```bash
$ sudo modprobe ntsync
$ lsmod | grep -i ntsync
ntsync                 24576  0
```

确认设备节点存在：

```bash
$ ls -la /dev/ntsync 
crw-rw-rw- 1 root root 10, 261 Oct 17 09:03 /dev/ntsync
```

当 `/dev/ntsync` 出现时，说明系统端的支持已准备就绪，Wine 将自动检测并使用该功能。

## NTSync 带来的性能提升

在上一篇文章中，我们简单介绍了 NTSync 的工作原理——它是针对 Windows 的同步原语（如事件、信号量、互斥量）在 Linux 下实现的一种更高效机制，旨在取代早期的 esync 和 fsync 补丁。理论上，它能显著减少系统调用次数，提高线程同步性能，特别是在游戏等多线程场景下效果尤为明显。

不过，想在本地直接测试性能提升并不容易。我曾尝试使用 ChatGPT 生成的基准测试程序，但结果并不理想。幸运的是，国外的 Phoronix 网站已经进行了实际测试：

> 数据来源：[https://www.phoronix.com/news/NT-Sync-Primitive-Driver-v2](https://www.phoronix.com/news/NT-Sync-Primitive-Driver-v2)

根据该文章的评测数据，NTSync 在初步测试中表现出色，**能让部分 Windows 游戏在 Linux 下的运行性能显著提升**。下图展示了其中的对比结果：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202510/images/turn_on_ntsync_02.png)

这项技术由 CodeWeavers 工程师 **Elizabeth Figura** 主导开发，并已在 **Steam Play/Proton** 环境中得到验证。直到 Wine 10.16 版本，它才正式合并进入主线代码。


## 小结

NTSync 的引入，标志着 Wine 在兼容 Windows 应用性能方面迈出了重要一步。它不仅能显著降低线程同步的延迟，还能减少 Wine 与内核之间的切换开销，使复杂的 Windows 程序和 3D 游戏在 Linux 上的运行更加流畅。

目前，**只要内核版本在 6.6 以上且支持 CONFIG_NTSYNC 模块，就可以通过手动加载启用**。未来，随着更多发行版默认集成该模块，用户无需任何额外设置，就能直接享受到更接近原生 Windows 的性能体验。

换句话说，**NTSync 的落地，是 Wine 向“原生级兼容层”迈进的重要一步——它不仅仅是一个补丁，更是一种全新的运行机制。**

