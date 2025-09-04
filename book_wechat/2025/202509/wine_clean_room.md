# 戴着镣铐起舞，Wine 开发中的净室原则

今天在查阅 Wine 项目的开发者指南中，发现有这样一句：

> 开发者都应当注意不要违反“净室”指导原则（ Clean Room Guidelines）。

那什么是净室原则？Wine 项目也给出了解释：

* 可以做的事

以下是一些被认为对 Wine 贡献者来说是安全的做法：

✅: 在尝试理解某个 Windows API 函数时，可以编写一个测试程序来验证其行为，并将其贡献到 Wine 的一致性测试套件中。

✅: 查阅 MSDN 也是可以的（但要有所保留地看待）。

✅: 为了保险起见——如果写一个测试程序就足够了，就不要去看微软的文档或头文件。

✅: 如果拿不准，去 wine-devel 邮件列表上提问！

* 不可以做的事

以下是 Wine 贡献者需要避免的一些行为。这绝不是完整的清单；所有贡献者都需要对版权问题保持谨慎，并避免触犯任何法律。

❌: 不要写测试程序去打印某个内部表的值。

❌: 不要反汇编微软的代码。

❌: 不要查看任何微软的源代码，即使它在某些许可证下“公开”了。例如，不要查看他们 C 编译器附带的 C 运行时库源码。需要注意的是，例外情况是：如果代码是以 MIT 许可证（或其他与 LGPL 兼容的许可证）发布的，那么查看和借鉴是可以的（需进行适当的署名）。

❌: 不要在原生组件上使用 +relay。总体而言，应尽量避免用原生组件进行调试，因为那样会泄露这些组件所调用的函数信息。

❌: 不要查看微软代码的调试符号。

❌: 不要查看 ReactOS 的代码（甚至连头文件都不要看）。它的大部分内容是通过并不适用于 Wine 的方法逆向工程得到的，因此对我们来说并不是可靠的信息来源。

这里面还专门提到不要查看 ReactOS 的代码，ReactOS 又是何方神圣？

ReactOS 是一个 自由开源的操作系统，目标是实现与 Windows 二进制兼容。换句话说，它希望能够直接运行 Windows 应用程序和驱动程序，而不需要修改这些软件。

ReactOS 项目最早始于 1996 年，当时叫 FreeWin95，后来在 1998 年更名为 ReactOS。

值得一提的是，ReactOS 不是基于 Linux，而是一个从零开始编写的操作系统内核和用户态组件，力求在架构上与 Windows NT 系列（如 Windows XP/Server 2003）兼容。所以可以直接运行部分 Windows 程序（通过自身实现的 Win32 API），也可以加载部分 Windows 驱动程序。不过由于开发滞后，外观和使用习惯类似早期的 Windows（如 Windows 2000/XP）。

截至 2025 年，ReactOS 仍处于 Alpha 阶段，在业界也没有什么影响力。看起来 ReactOS 如果能够发展壮大，就可以打破微软帝国的垄断，那为什么没发展起来呢？因为这其中存在巨大的版权风险：

1. 逆向工程方式

* ReactOS 的部分实现是通过对 Windows 进行 逆向工程 得到的，比如分析 Windows 的行为、API 调用、甚至二进制接口。

* 在欧美法律体系下，逆向工程虽然在某些情况下合法（例如出于兼容性目的），但往往存在 法律灰色地带，尤其当涉及微软这种大型厂商时。

2. 代码来源不完全透明

* ReactOS 社区虽声称遵守合法逆向原则，但外界很难百分百确认其代码中是否混入了来源可疑的实现（例如从 Windows SDK、Windows 源代码泄露版里参考来的东西）。

* 微软过去也曾经对 ReactOS 提出质疑。

正因为存在法律风险，所以没有大型基金会或公司参与 ReactOS 项目的发展，导致社区非常小众，论坛和 IRC 上活跃的开发者和测试用户为主。主要用户为系统爱好者、黑客、研究操作系统的人。

出于长期发展的考虑， Wine 选择了更艰难的一条道，采用净室原则开发 Windows API 兼容层，难度要增加很多倍：

* 黑盒测试 vs 白盒逆向

  * 逆向方式：直接反汇编 DLL，就能知道 API 内部调用了哪些函数，参数怎么传，结构体怎么布局。

  * 黑盒方式：只能写测试程序，用各种输入跑出结果，再从外部行为反推内部规则。

  * 黑盒需要大量实验，才能逼近真实逻辑 → 难度指数级增加。

* 文档不完整

  * Windows API 文档（MSDN）只覆盖一部分常用接口，而且经常模糊不清。

  * 很多“边界情况”文档没写，只能靠写测试程序，一个一个试出来。

比如 GetSystemMetrics(SM_CXVSCROLL) 这个 API：

* 逆向方式：直接看 Windows 源码或反汇编，知道它返回了一个固定常量值（比如 17）。

* 黑盒方式：只能写程序跑 Windows，然后调用 API 打印结果，再在不同环境下测试，看是不是固定值，或者和 DPI/主题有关。

可以想象得到，黑盒方式要做很多测试，相当于通过编写测试用例来逼近 API 的真实行为。

参与过 Wine 项目的人会有比较深的感受，向 Wine 上游代码提交一个 PR 相当困难，要编写很多的测试用例来证明修改是推导出来，而不是通过研究 Windows 系统而得到的。有很多人不明其中的缘故，总认为国内的开发者不主动向上游贡献，真实的情况是，deepin wine 团队一直在向上游积极提交 PR，但是每个合并都要耗费大量的精力，经常需要和外国开发者来回拉扯，即使这样，deepin wine 团队还是向上游提交了不少补丁，以下是统计数据：
```
     14 chenhaoyang@uniontech.com
      7 yuhaidong@uniontech.com
      6 yeyeshun@uniontech.com
      6 luotingzhong@uniontech.com
      5 zhaoyi@uniontech.com
      5 longchao@uniontech.com
      5 cuijiajin@uniontech.com
      4 dongpengpeng@uniontech.com
      3 zhaozhipeng@uniontech.com
      3 yaoyongjie@uniontech.com
      3 chenjiangyi@uniontech.com
      1 yangkun@uniontech.com
      1 xiewei@uniontech.com
```
由于 Wine 项目对于合并补丁太过于严格，由此还产生了一个 Wine-Staging 项目。Wine Staging 相当于 Wine 的测试分支（experimental branch），专门收集和维护尚未被合并进主线 Wine 的补丁（patches）。它是 Wine 的“实验场”，新的功能、改进、bug 修复会先放在 Wine Staging 中进行测试。

Wine Staging 通常比 Wine 官方版本支持更多功能。例如：某些 Windows API、图形（D3D、Vulkan）、性能优化、兼容性修复。

一般来说，Wine Staging 有更好的兼容性，但同时也可能引入新问题。如果某个应用在 Wine 下运行有问题，不妨尝试一下 Wine Staging，也许有惊喜。
