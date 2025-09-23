# 干得漂亮，Ubuntu 终于干掉了 X11

在《[从 X11 到 Wayland，迈出这一步为何如此艰难？](https://mp.weixin.qq.com/s/_EO5xbY5J1dCukiMhxbQXA)》一文中，我们分析过从 X11 过渡到 Wayland 所面临的诸多挑战。如今，终于有 Linux 发行版吹响了 X11 的丧钟：Ubuntu 25.10 正式弃用 GNOME 的 Xorg/X11 会话，将 GNOME Shell + Mutter 桌面环境完全切换为 Wayland-only。

这个过程可谓漫长而曲折。早在 Ubuntu 17.10，系统就尝试将 GNOME on Wayland 设为默认会话，但当时 Wayland 的兼容性尚不成熟，尤其在 NVIDIA 专有驱动和远程桌面场景下问题频出。于是到了 Ubuntu 18.04，默认会话又回归 Xorg。随着数年改进，Wayland 支持日益成熟，Ubuntu 21.04 再次将 Wayland 会话设置为默认，但在检测到 NVIDIA 专有驱动时仍会自动退回 Xorg。经过社区和 NVIDIA 的努力，自 515 版本起，NVIDIA 驱动对 Wayland 的支持明显改善，从 Ubuntu 22.04.2 开始，Wayland 会话才真正成为默认选项。然而 X11 会话仍被保留，用户可在登录时选择。

到了 25.10，Ubuntu 终于与 X11 彻底告别——不再提供 X11 会话选择。目前 25.10 仍是测试版，根据 Ubuntu 发布节奏，下个月将发布正式版本。这一改变标志着 Ubuntu 在现代化、性能和安全性的新桌面体验上迈出重要一步。

当然，这并不意味着 Ubuntu 放弃对 X11 客户端应用的支持。通过 XWayland，这些应用仍可在 Wayland 会话中运行，也就是说大多数旧应用不会立刻失效。从历史经验来看，即便未来 XWayland 被移除，也将是一个漫长的渐进的过程。

有人可能会担心：在 Wayland 中嵌入 X11，会不会让系统臃肿？答案是否定的。Ubuntu 只保留单一桌面会话路径，使 GNOME 可以集中精力发展 Wayland，而不再维护 Xorg 与 Wayland 的双路线。XWayland 更像是在 Wayland 桌面里“外挂”一个小型 X server，而不是将 X11 的复杂性直接塞进 Wayland。Wayland 本身依然轻量，臃肿的仍然是被隔离在兼容层里的 X11。

每次技术变革都会带来兼容性问题，哪怕是微软在 Windows 系统升级时也无法完全避免。从 X11 迁移到 Wayland，也会出现一些问题：

* 某些依赖 X11 特性的应用（快捷键、工具、窗口管理扩展或插件）在 Wayland 上可能表现不同；
* 远程桌面/显示转发（如 X11 转发、xrdp 或某些 VNC 场景）在 Wayland 下可能更复杂，有时不如在 Xorg 下稳定；
* 依赖 Xorg 特定窗口管理行为或老工具的用户（如脚本、窗口管理器扩展、输入法、截屏工具等）可能在 Wayland 上暂时不便；
* 某些硬件或显卡上，Wayland 的性能、功耗优化和延迟调校可能尚不如 Xorg 完善，尤其在旧显卡或定制图形管线中更明显。

尽管如此，Wayland 替代 X11 是不可逆的趋势。Wayland 提供更现代的安全模型，对输入和窗口隔离更严格，有助于减少潜在安全漏洞。

## 小结

Ubuntu 25.10 “Questing Quokka”的发布，标志着一个里程碑——Ubuntu 彻底“干掉”了 GNOME 桌面的 Xorg 会话，全面采用 Wayland-only 的 GNOME + Mutter 会话。这不仅是技术上的更新，更是方向上的宣示：Ubuntu 正迈向一个更安全、更现代、更统一的 Linux 桌面未来。

对大多数普通用户而言，这意味着日常使用不会受到影响，因为常见应用可通过 XWayland 兼容运行；但对于高度依赖 X11 行为的用户或特定工作流程，短期内可能仍需适应或寻找替代方案。
