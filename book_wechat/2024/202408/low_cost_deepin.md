# 低成本体验国产系统的几种方式

前段时间，Windows 蓝屏事件引发了全球震动，最终发现问题源自一家安全公司，与微软无关。然而，细细一品，连一家第三方软件公司都能导致 Windows 系统在全球范围内大面积崩溃，那么如果微软自己出手，我们的 Windows 系统岂不是在瞬间就会陷入瘫痪？虽然理论上微软不会这么做，但这种可能性始终存在，类似于核威慑的存在意义：虽然不会轻易使用，但其威慑力始终存在。

如果全世界只有一个国家拥有核武器，其他国家是否会瑟瑟发抖？同样地，假设这个世界上只有一个操作系统，一旦出现问题导致我们的电脑崩溃，想想是不是非常可怕？

近年来，中国不断强调信息技术自主可控的重要性，推动国产操作系统的发展，正是为了防患于未然。

Windows 蓝屏事件提醒我们，依赖单一操作系统存在巨大的风险。通过发展国产操作系统，我们可以在应对突发事件时拥有更多选择和更强的应对能力。即使国外操作系统出现问题，国内系统也可以作为备选方案，确保关键基础设施和重要系统的正常运行。

国产操作系统经过多年的发展，其易用性和可靠性都取得了显著进步，但不少用户在接触后仍抱怨系统不好用、软件少等问题。这一方面确实与国产系统的成长时间较短、软件厂商适配不足有关；但另一方面，用户觉得难用的原因也在于他们固守 Windows 的操作习惯，认为国产系统不够顺手。这就像老一辈人使用智能手机一样，一些我们认为理所当然的操作，对他们来说却缺乏概念。

其实，国产系统在操作设计上尽量沿袭了 Windows，加上应用商店的出现，下载安装应用已经非常方便，不需要懂英文，也不需要使用命令行操作，只需简单的点击操作，如同使用智能手机一样便捷。

如果你对国产系统感兴趣，但担心迁移成本太高，可以继续阅读本文，下面将介绍几种低成本体验国产系统的方法。

## 1. LiveUSB

在 Linux 系统发展初期，开发者推出了一项技术，可以将 Linux 操作系统制作成镜像文件并刻录到光盘上，直接通过 CD 或 DVD 光盘启动并运行系统，这就是 LiveCD。LiveCD 的设计初衷是让用户在不安装系统的情况下，体验 Linux 的界面和功能。

- **试用系统**：无需更改硬盘数据即可体验 Linux 系统，判断是否符合你的需求。
- **系统修复**：若硬盘上的 Linux 操作系统出现问题，LiveCD 可用于修复或备份数据。
- **数据恢复**：当硬盘中的系统无法启动时，可以使用 LiveCD 进入系统，访问并恢复数据。

随着光盘逐渐被淘汰，USB 等移动存储设备取而代之，但这一技术得以延续，只是存储介质换成了 U 盘，即 LiveUSB。相比于 LiveCD，LiveUSB 不仅保留了上述功能，还具有以下优势：

- **更快的运行速度**：USB闪存盘的读写速度远快于 CD/DVD，因此 LiveUSB 的系统响应速度更快。
- **持久存储**：与光盘不同， U 盘是可读写的，能够保存用户数据。
- **便携性**：体积小巧，易于携带。

### 制作和使用LiveUSB的步骤

1. 下载国产系统的ISO镜像文件，例如可以从[Deepin OS官网](https://www.deepin.org/zh/download/)下载。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/low_cost_deepin_01.png)

2. 使用专门的软件（如深度提供的启动盘制作工具）将 ISO 文件写入 U 盘。

3. 设置计算机从 USB 设备启动。

通过 LiveUSB，你可以在不安装系统的情况下直接体验国产操作系统，这非常适合快速测试国产系统发行版或在不同电脑上临时使用。它不会影响现有系统，也不会更改硬盘上的任何数据，可以放心大胆地探索国产系统的各种功能。

不过需要注意的是，虽然 USB 的速度远快于 CD/DVD，但相较于硬盘（尤其是 SSD ），其读写速度仍较慢，因此性能可能不如硬盘运行时那样理想。此外，通常情况下，LiveUSB 没有持久化存储功能，无法保存用户数据。如果需要持久化存储，需要使用特定的软件工具（如Rufus）来制作 LiveUSB，稍微有些复杂，有需求再展开说说。

## 2. 虚拟机

虚拟机可以在现有系统上虚拟出一个主机，从而安装和运行国产系统，仿佛在一台独立的物理主机上操作。早期虚拟机性能不佳，但随着虚拟化技术的发展，尤其是硬件虚拟化的支持，虚拟机的性能得到了显著提升。如果你的 Windows 主机性能较好（如高主频 CPU、大内存），也可以尝试在虚拟机上试用国产系统。

虚拟机软件可以使用 VirtualBox，这是一款免费开源的软件，支持硬件虚拟化。安装和运行国产系统的基本步骤如下：

### 2.1 下载国产系统ISO镜像

例如可以从[Deepin OS官网](https://www.deepin.org/zh/download/)下载。

### 2.2 安装VirtualBox

访问[VirtualBox官网](https://www.virtualbox.org/)，下载最新版本的 Windows 安装包并完成安装。

### 2.3 创建Ubuntu虚拟机

- 打开 VirtualBox 软件，点击“新建”按钮。
- 在弹出的窗口中，输入虚拟机的名称（例如Deepin v23），**虚拟光盘** 选择之前下载的 ISO 镜像文件。

  ![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/low_cost_deepin_02.png)

- 点击“下一步”，设置虚拟机的内存大小和 CPU 数量。内存推荐8GB（8192 MB），根据你系统的内存大小自行调整，CPU 数量同理。

  ![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/low_cost_deepin_03.png)

- 点击“下一步”，选择“现在创建虚拟硬盘”，建议设置 100 GB 以上的存储空间。

  ![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/low_cost_deepin_04.png)

- 点击“下一步”后，会显示虚拟机的概要信息，确认无误后点击“完成”。

  ![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/low_cost_deepin_05.png)

### 2.4 安装Deepin系统

- 在 VirtualBox 主界面中，选择刚创建的 Deepin V23 虚拟机，然后点击“启动”按钮。

接下来的步骤基本无需干预，均选择默认选项即可。整个系统将安装在Windows下的一个文件中，不会影响到现有系统，可以放心尝试各种安装选项。

- 安装完成后，系统会提示你移除安装介质并点击“立即重启”。

  ![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/low_cost_deepin_06.png)

此时需要进入虚拟机设置中，移除挂载的 ISO 文件。

  ![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/low_cost_deepin_07.png)

系统启动后，设置用户名和密码，系统会自动进行一些配置，无需干预。几分钟后，系统安装完成，输入用户名和密码即可进入系统。

如果希望在虚拟机中深入体验 Deepin 系统，还可以进行虚拟机设置优化，比如调整分辨率、安装 VirtualBox Guest Additions、设置共享文件夹等。

### 虚拟机运行国产系统的优点：
- **安全无风险**：虚拟机允许在不修改现有系统的情况下运行国产系统，是测试或学习国产系统的理想选择。
- **便捷的快照功能**：虚拟机软件（如 VirtualBox ）支持快照功能，能够轻松恢复到之前的状态。

### 缺点：
- **性能影响**：由于国产系统在虚拟机内运行，可能会受到宿主操作系统资源的限制，导致性能下降，尤其是在资源密集型应用中。
- **硬件支持受限**：虚拟机环境对硬件的支持有限，某些外设或硬件特性可能无法在虚拟机中完全利用。

## 3. 加装硬盘，多系统启动

很多教程指导在现有系统上划分出一部分硬盘空间来安装 Linux 系统，但对初学者来说，风险较大，容易导致数据丢失。因此，强烈建议在条件允许的情况下，加装一块独立的 SSD 硬盘。如今，SSD 硬盘的价格已经是白菜价，256 GB 的 SSD 价格仅为一百多元。只要你的电脑有空余的硬盘插槽，安装一块额外的 SSD 硬盘会是更安全、更高效的选择。

在新硬盘上安装国产操作系统，可以充分利用硬件资源，获得最佳的系统性能，同时外设和硬件的支持也会更加完善。安装完成后，你可以在启动时通过启动管理器自由选择进入 Windows 或国产操作系统。这对于那些仍然依赖 Windows 进行某些工作的用户来说，是一种非常灵活的方案。你可以在不影响现有工作的前提下，体验国产系统的各种功能与优势。

**安装步骤概述：**

1. **硬盘安装**：首先，将新购的 SSD 硬盘安装到电脑中。确保连接稳固，并检查主板是否支持所购硬盘的接口类型（如 SATA 或 M.2）。

2. **制作启动盘**：下载你想要安装的国产系统的 ISO 镜像文件，并使用工具（如深度提供的启动盘制作工具）将其写入到一个 U 盘中，制作启动盘。

3. **启动系统**：将 USB 启动盘插入电脑，重启电脑并进入 BIOS/UEFI 设置，将启动顺序调整为从 USB 启动。保存设置并重启。

4. **系统安装**：在安装界面中，选择新安装的硬盘作为目标硬盘进行操作系统的安装。根据系统安装向导完成安装过程。安装完成后，启动菜单会自动生成，你可以在启动时选择进入哪个操作系统。

这种方法不仅可以充分发挥新硬盘的速度优势，还能在不影响现有 Windows 系统的情况下，为你提供独立的国产操作系统运行环境。此外，安装前切记备份重要数据，以防操作失误导致数据丢失。

## 4. 迷你主机

几年前，我购入了一款采用国产CPU（兆芯 KX-6640MA处理器）的Mini主机，详细体验可以参考我当时写的文章《[其实，我们也在努力做CPU](https://mp.weixin.qq.com/s/Cy0NITwMy07ySpmvZB4mOA)》。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/low_cost_deepin_08.png)

如今，迷你主机已然成为一个受欢迎的品类，市场上产品更加丰富，选择也更多。打开京东，搜索“迷你主机”，你会发现价格从几百元到几千元不等，性能覆盖从办公入门到高性能游戏主机，各种需求都能满足。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/low_cost_deepin_09.png)

别看这些迷你主机体积小巧，仅有一个电视盒子大小，但其性能表现非常出色。对于日常办公、浏览网页、观看视频，迷你主机完全能够胜任。即使是轻度的开发工作，迷你主机也能提供足够的支持。

如果你有意尝试国产操作系统，一款价格在千元左右的迷你主机会是一个理想选择。它占用桌面空间极少，甚至还可以挂在显示器背后。此外，迷你主机通常功耗较低，即使长时间开机运行，如下载电影、进行轻量级的服务器任务，也不用心疼电费。

更重要的是，迷你主机独立于你的主 PC 之外，确保在尝试新系统时不会影响主 PC 的日常工作。如果你选择和主 PC 共用显示器，通过切换输入源，你可以在主 PC 和国产系统之间无缝切换，享受多系统带来的便利与灵活性。这种设置既保障了工作流的稳定性，又为你探索国产系统提供了更多可能。

## 小结

本文介绍了四种使用国产操作系统的方式，每种方案都有其独特的优势和适用场景。

* **LiveUSB**：这是一个简单、快捷的方式，让你无需对现有系统做任何更改，就能体验国产操作系统的功能与界面。通过 USB 启动盘，你可以在不同电脑上随时随地体验国产系统，但需要注意的是，LiveUSB 通常不具备持久化存储功能。

* **虚拟机**：如果你希望在不影响现有系统的前提下深入体验国产操作系统，虚拟机是一个理想选择。虚拟机允许你在现有系统内创建一个虚拟环境，进行全面的系统测试与学习，而不会对现有数据造成任何风险。然而，虚拟机的性能可能略逊于直接安装在物理硬盘上的系统。

* **加装硬盘，多系统启动**：对于希望在硬件上获得最佳性能并充分利用系统资源的用户，加装硬盘并启用多系统启动是最合适的选择。通过独立硬盘安装国产系统，你可以在不影响现有 Windows 系统的情况下，全面体验国产操作系统的全部功能。这种方式兼顾了性能和安全性，是较为专业的解决方案。

* **迷你主机**：如果你希望完全独立于现有 PC 系统，同时又不想占用太多空间，迷你主机将是一个绝佳的选择。这种小巧且功耗低的设备不仅适合安装国产操作系统，还可以作为挂机下载或轻度软件开发环境。迷你主机的独立性使得它不会干扰你日常的 PC 使用，同时又能为你提供稳定的国产系统体验。

如果你还有更好的方案，欢迎交流。