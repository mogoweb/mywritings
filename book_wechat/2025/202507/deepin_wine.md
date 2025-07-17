# 十年磨一剑，让国产系统流畅运行 Windows 应用

在上一篇文章《[兜兜转转，我又开始研究 Windows 系统](https://mp.weixin.qq.com/s/A79_LI0Vxq5aiod51QFLMw)》中，分析了微软多年深耕 Windows，早已筑牢一道深不可破的护城河。即使在国产替代的大潮下，我们还是离不开 Windows 应用。

为了让 Windows 应用运行在国产系统上，有多种方案，目前最常见的方案就是 Wine。

## 什么是 Wine

Wine 是一个开源项目，它在各种 Unix 变体操作系统之上重新实现了微软 Windows 操作系统的部分功能。Wine 主要面向 Linux 和 macOS，但也可以运行于 FreeBSD、NetBSD、Solaris 等系统。对用户而言，这意味着他们可以在非 Windows 系统上运行原本为 Windows 编写的软件。

Wine 不包含任何微软拥有的代码，因此无需 Windows 许可证即可运行 Wine。相反，Wine 开发者将 Windows 操作系统的各个组件重新编写，使得在 Wine 上运行的软件“以为”自己正在 Windows 系统中运行，实际上却是在例如 Linux 的环境中。

举个简单的例子，考虑 Windows 的 CreateFileA API。在 Windows 上，一个应用可能会这样调用它：

```c
CreateFileA(
    "C:\\some_file.txt",    // lpFileName
    GENERIC_WRITE,          // dwDesiredAccess
    0,                      // dwShareMode
    NULL,                   // lpSecurityAttributes
    CREATE_ALWAYS,          // dwCreationDisposition
    FILE_ATTRIBUTE_NORMAL,  // dwFlagsAndAttributes
    NULL                    // hTemplateFile
);
```

而 Wine 会将该调用转换为 Linux 的 open 调用：

```c
open(
    "/home/aeikum/.wine/drive_c/some_file.txt", // path
    O_WRONLY | O_CREAT,                         // oflag
    0644                                        // creation mode
);
```

然后，Wine 会将返回的文件句柄返回给应用程序，应用程序便可使用类似的映射（如将 WriteFile 映射到 Linux 的 write）来写入文件。当然，Wine 中对 CreateFileA 的实际实现要复杂得多（例如路径转换等），但上述示例足以说明 Wine 的基本做法。

## 各种 Wine 发行版

Wine 是一个开源项目，“上游”（Upstream）Wine 是 Wine 的“官方”版本。Wine 项目对代码质量有着严格要求，注重正确性，包含大量单元测试以验证 Windows 的行为。向 “上游”（Upstream）Wine 提交补丁，必须附带单元测试，并且必须通过现有测试。这就导致许多有用但尚未充分验证的补丁无法被及时合并。比如说针对 deepin 系统的整合优化，就很难被合并到上游。此外， Wine 缺乏商业支持，无法及时响应用户需求。因此，Wine 项目衍生出了许多发行版。

目前已知存在数百个 Wine fork，但以下几个 fork 最为知名：

### Wine Staging
* 网站：https://wiki.winehq.org/Wine-Staging

* 由于上游对补丁质量有严格要求，许多有用但尚未充分验证的补丁无法被及时合并。Wine Staging（也称 “wine-staging”）项目旨在汇集并维护这些补丁，让用户可以立即受益。Staging 社区也努力将这些补丁提交到上游，以便所有 Wine 用户和分支都能共享它们，同时减轻自身维护负担。它还可作为尚难以用单元测试验证的补丁的“试验场”。

### CrossOver
* 网站：https://www.codeweavers.com/

* CrossOver 是 CodeWeavers 公司销售的商业版 Wine 发行版。它包含许多针对特定应用的补丁，这些补丁不适合提交到上游。CodeWeavers 还维护一个应用兼容性数据库，可以预安装某些软件组件或对 Wine 环境进行调整，从而提升特定应用的兼容性。

### Proton
* 网站：https://github.com/ValveSoftware/Proton

* Proton 是 Valve 公司在其 Steam 平台上集成的 Wine 发行版，旨在为 Steam 用户提供在 Linux 上运行 Windows 游戏的无缝体验。Proton 专为 游戏优化，兼容 DirectX 9/10/11/12，集成 Vulkan、DXVK（DirectX → Vulkan 转换）、vkd3d，让 Linux 用户也能完美运行 Windows 游戏。我在[一台迷你主机能做什么？编译程序、玩3A 大作、跑本地大模型，样样都行](https://mp.weixin.qq.com/s/e4TSv1_kLqrH-iECPJk0mg)一文中介绍过在 deepin V25 上玩 3A 游戏大作，就是使用了 steam 游戏平台。

### deepin-wine

* 网站：https://wine.deepin.org/

deepin‑wine 是深度操作系统（deepin）团队在上游 Wine 基础上，结合国产化需求与桌面深度集成做的一套定制化 Wine 运行环境，主要特点有：

1. **深度集成 DDE 桌面**

   * 自动将 Windows 应用图标、快捷方式与 deepin 应用菜单、桌面无缝对接；
   * 支持 Deepin 文件管理器右键直接通过“统信Windows应用兼容引擎”运行可执行文件。

2. **多版本并存与一键切换**

   * 内置多个 Wine 版本（如 deepin-wine-staging、deepin‑wine8-stable、deepin-wine10-stable），还支持安装proton，用户可针对不同软件或游戏选择最兼容的版本；
   * 通过“深度应用商店”或命令行工具轻松安装、卸载、切换。

3. **应用兼容性优化**

   * 集成了常用游戏和办公软件的专用补丁（如 DXVK、vkd3d、MSVC 运行库、一键补丁），开箱即用；
   * 定期同步 Wine Staging、Proton 和 CrossOver 的优质补丁。

4. **中文本地化与字体支持**

   * 默认配置了中文字体和输入法支持，能更好地处理中文路径、文件名与界面；
   * 优化了日志输出和错误提示，方便国内开发者调试。

5. **图形化管理工具**

   * “深度应用商店”支持 Wine 应用，一键下载安装 Windows 软件；
   * 提供 winecfg、winetricks 等命令行兼容，也可通过 GUI 进行驱动、组件、库的管理与配置。

6. **开源与社区维护**

   * deepin wine 团队积极向上游提交补丁，持续向 wine 上游社区提交 200 余个补丁，定期对接 Wine 官方上游，同步安全与功能更新。

## 从工具到生态共建

为提升 Linux 系统对 Windows 应用的兼容能力，deepin-wine 团队自 2014 年成立起便持续探索，从技术验证到产品化落地，从单一功能到全场景兼容，十年间的每一步突破都旨在构建更完善的生态闭环。

如今，这一探索迎来重要里程碑 —— 统信 Windows 应用兼容引擎官网正式上线，标志着兼容技术从工具迭代迈向生态共建的新阶段。

### 里程碑​

* deepin-wine团队成立于 2014 年，目标是通过技术探索提升 Linux 系统运行 Windows 应用的能力。团队向 wine 上游社区提交了200多个补丁，推动技术验证到产品化落地。

* 2021年：​团队首次实现wine技术应用化，推出“wine助手”。它允许用户在deepin系统上双击直接安装和运行Windows exe程序，大幅降低技术门槛，适合普通用户。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/deepin_wine_01.png)

* 2024年：​​团队推出“UOS应用迁移助手”，聚焦专业场景，支持将exe程序打包为deb包，并兼容绿色软件打包和ARM架构运行，满足运维人员、工程师和开发者的需求。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/deepin_wine_02.png)

* 2024年11月：​UOS应用迁移助手更名为“统信Windows应用兼容引擎”，并升级至V3.0.4版本。功能从“打包工具”转向“全场景兼容引擎”，支持双击直接运行exe程序，同时打包功能整合为延伸选项，优先保障应用运行成功率。引擎面向普通用户提供一键运行便利，为技术用户提供图形化迁移工具，覆盖多架构（如ARM、X86）场景。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/deepin_wine_03.png)

* 2025年迭代升级​​
  * ​​Proton支持与架构优化（V3.3.1）​​：适配Proton技术，支持在deepin 25系统中运行游戏，提升性能和兼容性（如新增wow64支持，让64位系统运行32位游戏）。

  ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/deepin_wine_04.png)

  * 应用清单与标准化（V3.3.0）​​：新增“全部应用”模块，提供团队验证的可兼容应用清单；默认wine版本升级为deepin-wine10-stable，统一容器标准。

  ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/deepin_wine_05.png)

  * 内存优化突破（V3.3.1）​​：针对64位Electron框架应用，优化后内存占用减少90%，接近Windows原生水平。

  ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/deepin_wine_06.png)

## 生态共建新起点​

统信 Windows 应用兼容引擎官网正式启用，标志着兼容技术从工具迭代迈向生态共建！

官网（https://wine.deepin.org/）提供详细教程、开发文档和论坛入口，支持用户从“确认兼容”到“一键安装”的全流程。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/deepin_wine_07.png)

欢迎各位朋友加入 deepin-wine 用户队伍，参与 X86 Wine 应用的迁移投递，齐心协力推动 Linux 系统上 Windows 应用兼容技术的持续进步，为用户打造更加多元、完善的应用生态体验。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/deepin_wine_08.png)

