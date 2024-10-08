# 什么是TPM？ Windows 11 是否会重蹈 Windows Vista 覆辙？

2021 年 6 月 25 日，微软举行发布会，正式推出新一代操作系统 **Windows 11**。相比手机操作系统差不多一年更新一个版本，Windows 的迭代节奏还是慢很多。Windows 11 和 Windows 10 之间隔了 6 年多时间，这在手机领域无法想象。

对于这次 Windows 系统的一次大版本更新，无论是微软，还是用户，都充满着期待。发布会上，Windows 11 也没有让大家失望，全新的设计语言、全面优化的触控体验、脱胎换骨的商店、可直接运行 Android 应用、革新的游戏性能... 可以说，Windows 11 算是憋了一个大招。

想升级 Windows 11，先来看看微软官方给出的 Windows 11 最低配置硬件需求：

> 处理器: 1 GHz 或更快的支持 64 位的处理器（双核或多核）或系统单芯片 (SoC)
>
> 内存: 4GB
>
> 存储: 64GB 或更大的存储设备
>
> 系统固件: 支持 UEFI 安全启动
>
> TPM: 受信任的平台模块 （TPM） 版本2.0
>
> 显卡: 支持 DirectX 12 或更高版本，支持 WDDM2.0 驱动程序
> 
> 显示屏: 对角线长大于 9 英寸的高清 （720p） 显示屏，每个颜色通道为 8 位
>

可以看到，Windows 11 对处理器、内存、显卡的要求并不高，基本上近 10 年出的电脑配置都可以满足。但清单中多了一个 TPM，而且还要求 2.0 版本，这个就不是很多计算机能够满足的。要知道 TPM 2.0 正式成为 ISO/IEC 国际标准是 2015 年，考虑到标准制定到实施有一定的滞后，可能 2016 年之后的电脑才支持这一标准。

那什么是 TPM 呢？对于 Windows 11 升级会带来什么影响？

我之前写过一篇文章[聊一聊可信执行环境]()，介绍了智能设备安全相关的**可信执行环境**。与之相对应的，在计算机领域也有这样一个安全模块，这就是 TPM。

TPM 的全称是 Trusted Platform Module（可信平台模块），是根据国际行业标准组织可信计算组(TCG，其中包括微软、英特尔和惠普等公司)规范制作的模块。TPM 技术旨在提供基于硬件的安全相关功能。TPM 芯片是一种安全的加密处理器，旨在执行加密操作。芯片包含多重物理安全机制，具有防篡改功能，恶意软件无法篡改 TPM 的安全功能。TPM 技术的主要作用有：

* 生成、存储和限制使用加密密钥。
* 通过使用 TPM 的唯一 RSA 密钥，将 TPM 技术用于平台设备身份验证，该密钥已刻录到自身中。
* 通过采取和存储安全措施帮助确保平台完整性。

简单说来，和手机上的 TEE 相似，TPM 实际上是一个含有密码运算部件和存储部件的小型片上的系统，具有硬件级别的安全防护，再加上功能比较单一，相比复杂的计算机系统，更难于攻破。

怎么判断自己的电脑是否支持 Windows 11 所要求的 TPM 呢？

1. 按 **Windows + R** 组合键调出**运行对话框**，输入 tpm.msc; 

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/what_is_tpm_01.png)

2. 在弹出的对话框中**状态**一栏查看是否显示为 **TPM 已就绪**；

3. 查看 TPM 模块对应的规范版本是否为 2.0；

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/what_is_tpm_02.png)

如果状态显示的不是 **TPM 已就绪**，也可能是 BIOS 中未开启。在 BIOS 中开启 TPM，各计算机厂商的菜单各不相同，在我的电脑上菜单是这样的：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/what_is_tpm_03.png)

令人意外的是，开启了 TPM 后，系统有一个警告：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/what_is_tpm_04.png)

有人可能会纳闷，这个模块不是用来加强计算机安全的吗？为什么还会有这样的警告呢？

因为，有时太安全对某些部门来说不是好事，比如美国长期以来就禁止向中国出口高级别的加密技术，特别是一些军用级的加密技术。所以，由于某些国家的法律法规影响，TPM并没有推广到全球。这也是为什么大部分计算机在 BIOS 中，这一项是默认关闭的。

这不禁让我们想起了 Windows Vista 的命运。要说微软史上最悲剧的操作系统，那应该是非Windows Vista莫属了。Windows Vista在安全上的改进非常多，增加了使用者帐户控制 (UAC) 和 BitLocker 等重要安全功能。但由于改进过于激进，设计太过于超前、兼容性太差、配置要求高，用户纷纷拒绝升级，最后惨淡收场，直到 Windows 7 的出现，才给 Windows 续上命。也有阴谋论者宣称，Windows Vista 的出现是是微软和英特尔的阴谋，用来淘汰以前的主流电脑。

这次 Windows 11 的重大更新，不知道会引起多少兼容问题，且不说有多少电脑支持 TPM 2.0，就单是开启 TPM，估计会难倒一大批用户吧？非专业用户，有多少直到如何进入 BIOS（BIOS界面各厂商都有自己的一套，进入方法也各不相同），修改 BIOS 设置呢？修改之后，看到那样一个大大的警告，不知道有多少人会心里不发虚。

个人认为，这次的 Windows 11 的升级，不会那么顺利。反正 Windows 10 使用得还比较顺手，有什么必要升级到 Windows 11 呢？

Windows 11 要到今年秋季才会推送给用户，届时，你会升级到新的系统吗？

