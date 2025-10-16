# Wine 10.16 发布：NTSync 让 Windows 应用运行更丝滑了，但还得等等

其实这条消息已经不算新鲜了。早在国庆节期间，也就是 10 月 3 日，Wine 10.16 就正式发布了。
这次的更新日志看起来很简短：

> **Wine 10.16 Released (October 3, 2025)**
>
> What’s new in this release:
>
> * Fast synchronization support using NTSync
> * 16-bit apps supported in new WoW64 mode
> * Initial support for D3DKMT objects
> * WinMD (Windows Metadata) files generated and installed
> * Various bug fixes

看似平常的更新中，最值得关注的改进是 **NTSync**。那它到底是什么？为什么说它对 Wine 乃至整个 Linux 上的 Windows 兼容层都有重大意义？这篇文章就带你梳理一下。

## 一、什么是 NTSync

`NtSync` 是 Wine 团队近年来开发的一个**内核同步接口层（Kernel Synchronization Layer）**。
它的名字来源于 Windows 内核函数前缀 “Nt”（即 **Native API**），比如：

* `NtWaitForMultipleObjects`
* `NtCreateEvent`
* `NtReleaseSemaphore`

这些函数在 Windows 系统中用于操作 **NT 内核对象**（如事件、互斥量、信号量等），是操作系统同步机制的核心。

在此之前，Wine 中这些同步对象都是通过 **wineserver**（Wine 的后台用户态服务进程）来模拟实现的。
也就是说，Wine 自己构建了一个“伪内核”，负责协调线程等待与唤醒等操作：

```
Windows app
   ↓
Win32 API (WaitForSingleObject)
   ↓
Wine kernel32.dll -> wineserver
   ↓
Linux futex
```

## 二、为什么要引入 NTSync

NTSync 的出现，是为了**更高效、更精确地还原 Windows 的内核同步行为**。

过去，Wine 的同步逻辑完全由 wineserver 处理，每次 Wait 或 Signal 操作都需要跨进程通信。这带来了两个明显问题：

* **上下文切换频繁，性能损耗大**；
* **复杂的同步语义难以精确模拟**，尤其在多线程或多进程应用中。

这使得一些复杂程序（如大型游戏、Photoshop、Steam 客户端、Office 等）在 Linux 上运行时，会出现轻微卡顿甚至线程死锁的情况。

NTSync 的引入，就是要把这些同步逻辑**下放到内核层**，实现真正意义上的“内核级同步”。

## 三、NTSync 的设计思路

Wine 为此在 Linux 上增加了一个新的 **内核接口层**：`ntsynchronization`（简称 NTSync）。

这个接口：

* 通过 **内核模块或系统调用** 实现；
* 将原本在 wineserver 中模拟的同步逻辑迁移到 **Linux 内核的 futex / io_uring** 机制；
* 能够提供与 Windows NT 几乎一致的同步对象语义。

新的调用路径变成了这样：

```
Windows app
   ↓
Win32 API (WaitForSingleObject)
   ↓
Wine kernel32.dll -> NTSYNC (kernel driver)
   ↓
Linux futex / io_uring / epoll
```

这样，Wine 不再频繁进入 wineserver，性能显著提升，同时线程等待行为也更加接近原生 Windows。

下面放一章两种调用栈的对比图：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202510/images/wine_ntsync_01.png)


## 四、NTSync 的意义与效果

| 对比项   | 传统机制（wineserver） | NTSync 机制     |
| ----- | ---------------- | ------------- |
| 实现层级  | 用户态              | 内核态           |
| 上下文切换 | 频繁               | 显著减少          |
| 行为精确度 | 与 Windows 存在差异   | 几乎完全一致        |
| 性能表现  | 可能成为瓶颈           | 更接近原生 Windows |
| 游戏兼容性 | 可能出现卡顿、死锁        | 明显改善          |

NTSync 的研发历时多年，早期已经在 **Wine Staging** 和 **Proton（Steam 使用的 Wine 分支）** 中验证过，
对多线程游戏、图形引擎（DXVK、vkd3d-proton）等性能提升明显。
如今它正式合入主线，意味着这项特性已经趋于稳定，将逐步惠及所有 Wine 用户。

不过，目前的 NTSync 仍然存在一个现实问题：**它依赖 Linux 内核支持**。
也就是说，即便 Wine 已经准备好，如果系统内核尚未集成 NTSync 驱动，功能仍无法启用。

## 五、对 Linux 内核的要求

NTSync 依赖于 Linux 内核中的 `ntsynchronization` 接口，这个接口直到最近才被上游社区接受。
目前仅部分开发版内核（预计从 Linux 6.14 起）才会默认包含这一特性。

| 项目       | 要求                                        |
| -------- | ----------------------------------------- |
| **内核版本** | Linux 6.6 或更新版本（推荐 6.8+，完整支持将在 6.14）      |
| **内核配置** | 需启用 `CONFIG_NTSYNC`                       |
| **设备节点** | `/dev/ntsynchronization`（或 `/dev/ntsync`） |
| **支持情况** | deepin V25、Ubuntu 25.10 等主流发行版尚未默认启用      |

如果系统不支持 NTSync，Wine 会自动回退到传统的 wineserver 同步机制，因此仍可运行，只是性能提升无法体现。

检测方法也很简单：

```bash
$ ls /dev/ntsynchronization
```

如果该设备存在，就说明内核支持。否则，即代表当前系统尚未启用此功能。

---

真的是高兴太早了，看到 Wine 的发布日志，我还以为马上就可以用上了。目前连 Ubuntu 对于新特性支持比较激进的版本，都没有启用，那估计还需要再等一段时间，毕竟，每次的内核更新，都需要经过很长时间的测试。

当然，这个应该不会太久，等未来主流发行版的内核全面启用 NTSync 后，Linux 上的 Windows 应用将更加流畅。

最后，wine 10.16 的贡献者名单上已经有我的名字，虽然只是一个小小的提交，争取以后能够贡献更多的代码。