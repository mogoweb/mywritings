# 都是软件版本兼容惹的祸：龙芯 UOS 系统上 Qt Creator 启动失败

在《[在龙芯迷你电脑上搭建开发环境](https://mp.weixin.qq.com/s/rNUTJZmh29qlU8vyvjAftg)》一文中，我详细介绍了如何在龙芯 UOS 系统上搭建开发环境，这其中就介绍了 Qt 开发工具 Qt Creator 的安装过程。然而，Qt Creator 安装之后，从菜单上启动，没有任何反应，从终端上启动，提示如下：

```
alex@alex-loongson-MiniPC:~$ qtcreator 
mesa: CommandLine Error: Option 'help-list' registered more than once!
LLVM ERROR: inconsistency in registered CommandLine options
```
从错误提示来看，问题似乎与 Clang 或 LLVM 有关。然而，检查 Qt Creator 的依赖后发现，它并未直接依赖于 LLVM：

```
$ ldd /usr/bin/qtcreator
        linux-vdso.so.1 (0x000000ffffe90000)
        libExtensionSystem.so.4 => /usr/bin/../lib/loongarch64-linux-gnu/qtcreator/libExtensionSystem.so.4 (0x000000fff47c4000)
        libAggregation.so.4 => /usr/bin/../lib/loongarch64-linux-gnu/qtcreator/libAggregation.so.4 (0x000000fff47b4000)
        libUtils.so.4 => /usr/bin/../lib/loongarch64-linux-gnu/qtcreator/libUtils.so.4 (0x000000fff454c000)
        libQt5Widgets.so.5 => /lib/loongarch64-linux-gnu/libQt5Widgets.so.5 (0x000000fff3e3c000)
        libQt5Gui.so.5 => /lib/loongarch64-linux-gnu/libQt5Gui.so.5 (0x000000fff3884000)
        libQt5Concurrent.so.5 => /lib/loongarch64-linux-gnu/libQt5Concurrent.so.5 (0x000000fff3874000)
        libQt5Network.so.5 => /lib/loongarch64-linux-gnu/libQt5Network.so.5 (0x000000fff36ac000)
        libQt5Core.so.5 => /lib/loongarch64-linux-gnu/libQt5Core.so.5 (0x000000fff316c000)
        libGL.so.1 => /lib/loongarch64-linux-gnu/libGL.so.1 (0x000000fff3010000)
        libpthread.so.0 => /lib/loongarch64-linux-gnu/libpthread.so.0 (0x000000fff2fe4000)
        libstdc++.so.6 => /lib/loongarch64-linux-gnu/libstdc++.so.6 (0x000000fff2e38000)
        libm.so.6 => /lib/loongarch64-linux-gnu/libm.so.6 (0x000000fff2d78000)
        libgcc_s.so.1 => /lib/loongarch64-linux-gnu/libgcc_s.so.1 (0x000000fff2d1c000)
        libc.so.6 => /lib/loongarch64-linux-gnu/libc.so.6 (0x000000fff2b94000)
        /lib64/ld.so.1 (0x000000fff4819900)
        libdl.so.2 => /usr/bin/../lib/loongarch64-linux-gnu/qtcreator/../libdl.so.2 (0x000000fff2b88000)
        libQt5Qml.so.5 => /usr/bin/../lib/loongarch64-linux-gnu/qtcreator/../libQt5Qml.so.5 (0x000000fff2764000)
        libpng16.so.16 => /lib/loongarch64-linux-gnu/libpng16.so.16 (0x000000fff2728000)
        libharfbuzz.so.0 => /lib/loongarch64-linux-gnu/libharfbuzz.so.0 (0x000000fff2614000)
        libz.so.1 => /lib/loongarch64-linux-gnu/libz.so.1 (0x000000fff25f0000)
        libicui18n.so.63 => /lib/loongarch64-linux-gnu/libicui18n.so.63 (0x000000fff22b4000)
        libicuuc.so.63 => /lib/loongarch64-linux-gnu/libicuuc.so.63 (0x000000fff20a4000)
        libpcre2-16.so.0 => /lib/loongarch64-linux-gnu/libpcre2-16.so.0 (0x000000fff204c000)
        libdouble-conversion.so.1 => /lib/loongarch64-linux-gnu/libdouble-conversion.so.1 (0x000000fff2030000)
        libglib-2.0.so.0 => /lib/loongarch64-linux-gnu/libglib-2.0.so.0 (0x000000fff1ef0000)
        libGLX.so.0 => /lib/loongarch64-linux-gnu/libGLX.so.0 (0x000000fff1eb4000)
        libGLdispatch.so.0 => /lib/loongarch64-linux-gnu/libGLdispatch.so.0 (0x000000fff1d34000)
        libfreetype.so.6 => /lib/loongarch64-linux-gnu/libfreetype.so.6 (0x000000fff1c68000)
        libgraphite2.so.3 => /lib/loongarch64-linux-gnu/libgraphite2.so.3 (0x000000fff1c34000)
        libicudata.so.63 => /lib/loongarch64-linux-gnu/libicudata.so.63 (0x000000fff01fc000)
        libpcre.so.3 => /lib/loongarch64-linux-gnu/libpcre.so.3 (0x000000fff01ac000)
        libX11.so.6 => /lib/loongarch64-linux-gnu/libX11.so.6 (0x000000fff0058000)
        libXext.so.6 => /lib/loongarch64-linux-gnu/libXext.so.6 (0x000000fff0038000)
        libxcb.so.1 => /lib/loongarch64-linux-gnu/libxcb.so.1 (0x000000fff0000000)
        libXau.so.6 => /lib/loongarch64-linux-gnu/libXau.so.6 (0x000000ffefff4000)
        libXdmcp.so.6 => /lib/loongarch64-linux-gnu/libXdmcp.so.6 (0x000000ffeffe4000)
        libbsd.so.0 => /lib/loongarch64-linux-gnu/libbsd.so.0 (0x000000ffeffc4000)
        librt.so.1 => /lib/loongarch64-linux-gnu/librt.so.1 (0x000000ffeffb4000)
```
既然主体程序与 LLVM 无直接关联，那么问题很可能出在插件上。经过一番排查，最终定位到 /usr/lib/loongarch64-linux-gnu/qtcreator/plugins/libClangTools.so 插件。解决的方法就是重命名，不加载这个插件：

```
$ sudo mv /usr/lib/loongarch64-linux-gnu/qtcreator/plugins/libClangTools.so /usr/lib/loongarch64-linux-gnu/qtcreator/plugins/libClangTools.so.bak
```
完成上述操作后，重新启动 Qt Creator，熟悉的界面终于出现了：

![图1 Qt Creator界面](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/loongson_qtcreator_01.png)

至此，问题得以部分解决。尽管缺少 ClangTools 插件，但对 Qt 开发影响不大。

## 软件兼容性问题的思考

在这里，我想额外聊聊软件兼容性问题。这不是一个新话题，我之前写过的一些文章中也涉及到相关内容，例如：

* 《[龙芯 UOS 系统升级 Python](https://mp.weixin.qq.com/s/-75AGX9y_cSH35rG59M3ug)》
* 《[UOS 系统 Qt 版本切换](https://mp.weixin.qq.com/s/uVmU6tM26NXUVJmYnRTC9w)》
* 《[从 X11 到 Wayland，迈出这一步为何如此艰难？](https://mp.weixin.qq.com/s/_EO5xbY5J1dCukiMhxbQXA)》

这些问题无一例外都与软件版本的兼容性密切相关。可以说，兼容性问题贯穿了软件产品的整个生命周期。Windows 在这方面相对做得不错，但也无法完全避免。例如，Windows 系统中多版本的 VC++ 运行库依然困扰着不少开发者。

软件的持续迭代和版本共存，难免带来兼容性挑战。新功能的引入往往需要修改底层接口或引入新的依赖，而这些变动可能与现有系统中的其他组件或应用程序产生冲突。尽管向后兼容是理想状态，但在实际开发中，往往难以做到尽善尽美。资源有限、技术债务积累、开发周期紧张等因素，都可能导致开发者不得不在兼容性与交付之间权衡取舍。

尤其是在 Linux 系统中，包管理机制进一步加剧了这一问题。以 Debian 系的 Linux 发行版为例，deb 包丰富性，安装便捷，但也不得不面对 deb 包版本不兼容的烦恼。一个依赖的更新可能导致链式反应，使得多个软件无法正常工作。这种情况在开发和部署过程中尤为常见，甚至可能让用户和管理员望而却步。

既然软件的兼容问题这么难以解决，那么能否借鉴 Docker 的做法，为应用软件建立一个独立的运行环境？通过将应用程序及其依赖隔离运行，可以最大程度地降低因系统环境变动带来的不确定性。

事实上，许多 Linux 厂商已经开始尝试这种做法，比如统信推出的玲珑包格式。关于玲珑，请参考我之前写过的一篇文章：《[国产系统之如意玲珑](https://mp.weixin.qq.com/s/VE41M9pCsEEMCJGyRwaroQ)》。

玲珑包通过将应用程序与其依赖完整打包在一起，提供了一个自包含的运行环境。理论上，它可以在任何支持玲珑的 Linux 发行版上运行，有效减少跨发行版的兼容性问题，同时也避免了同一系统上软件组件版本冲突的困扰。

更重要的是，玲珑包的理念与现代软件分发的趋势高度契合。它不仅提高了软件的可移植性，也简化了开发者的发布流程。对于终端用户而言，安装软件不再依赖于繁杂的系统环境配置，而是更接近于“开箱即用”的体验。这种模式还减少了对系统管理员的依赖，为个人用户、开发者和企业提供了更多的自由和便利。

当然，玲珑的解决方案也并非完美。至少目前还存在如下问题：

1. **自包含的打包格式可能导致磁盘空间使用的增加**  
   自包含的打包方式本质上是通过将应用程序及其所有依赖统一打包，确保在隔离的运行环境中正常工作。然而，这种方式的一个直接后果是可能会显著增加磁盘空间的占用。特别是在多个应用程序依赖相同的底层组件时，每个玲珑包都会各自携带一份相同的依赖库，这种冗余无法像传统包管理系统一样通过共享库机制减少存储开销。尽管后续可以通过技术手段进行优化，例如压缩包体、分层存储以及模块化依赖，但无论如何，相比传统的 deb 包，玲珑包的体积仍然会更大。当然这个问题可能会随着存储空间越来越廉价，不那么突出。

2. **应用程序运行在沙箱中，访问系统资源受限**  
   为了增强隔离性和安全性，玲珑包中的应用程序通常运行在沙箱环境中，限制其对系统资源的直接访问。这种机制虽然提高了安全性，但也为某些应用的运行带来了障碍，尤其是那些需要深入访问系统核心功能的应用。例如，某些系统工具或底层服务类程序在沙箱中可能无法正常访问硬件接口、系统日志或特定的配置文件。这种限制在其他平台上也存在，比如 macOS 的沙箱机制就经常导致一些系统应用无法通过 App Store 的审核，开发者只能通过分发独立安装包或请求额外权限来解决问题。对于玲珑包而言，如何在安全性与功能性之间取得平衡，将是一个长期需要攻克的难题。

3. **玲珑包中运行的应用程序，调试起来不太方便**  
   调试是软件开发过程中至关重要的环节，而在沙箱环境中运行的应用程序，由于与宿主系统环境隔离，调试起来可能面临更多挑战。例如，开发者可能无法直接访问沙箱内部的日志文件、调试信息或运行状态，需要借助额外的工具或命令来进入沙箱环境进行调试。这不仅增加了调试的复杂度，也可能导致开发效率的降低。此外，沙箱机制的隔离性还可能干扰一些调试工具的正常运行，例如传统的调试器可能无法跨越沙箱的边界直接与目标进程交互。尽管目前已经有针对沙箱环境的调试工具，但这些工具的学习成本和操作复杂性，可能会让一些开发者望而却步。

4. **生态系统的构建和适配仍在初期**  
   玲珑包作为一种相对较新的技术，其生态系统建设还在起步阶段。与传统的 deb 包相比，玲珑包的构建工具链、最佳实践和社区支持尚未成熟。这可能会让部分开发者在迁移过程中遇到阻力，尤其是对于复杂应用程序的适配。

期待随着技术的不断迭代和社区的持续投入，这些问题都得到解决，再也不要被这些软件版本兼容问题所折磨。
