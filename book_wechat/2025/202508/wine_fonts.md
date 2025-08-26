# Wine 字体加载机制探秘：Windows 应用字体缺失问题分析

在国产系统上运行 Windows 应用，字体无法显示，或者字体没按期望的字体显示，是比较常见的一类问题。

比如，使用如下指令创建一个 Wine 容器，然后打开注册表，在某些 UOS 系统上就会出现“方块”字。

```
mkdir -p ~/.test/wine
export WINEPREFIX=~/.test/wine
deepin-wine8-stable regedit
```
![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_fonts_01.png)

本文将深入剖析 Wine 内部复杂的字体查找与替换机制，并在此基础上，清晰地解释方框字问题的根本原因及解决方案。

Windows 和 Linux 拥有两套截然不同的字体管理系统。Wine 的使命就是在其中搭建一座坚固的桥梁，将 Windows 应用的字体请求（API调用）无缝地“翻译”给 Linux 底层的字体渲染引擎（通常是FreeType）。为了实现这一目标，Wine 设计了一套精巧且层次分明的字体查找流程。

当一个 Windows 应用需要显示一段文字时，Wine 会先处理 Windows 的字体处理逻辑，并加上 Wine 自己的一些逻辑，以确保找到最合适的字体。

## 第一棒：名称翻译 - 从逻辑字体到物理字体

应用程序的字体请求通常是字体家族（font family）或则一个逻辑字体名，如MS Shell Dlg 2（常用于对话框）或System，而不会指定具体的字体文件。

Wine 的第一项任务是查阅注册表，将这个逻辑名称翻译成一个具体的物理字体族名称（如 Tahoma ）。这个翻译过程遵循明确的优先级：

1. Wine首先检查当前用户的专属配置：`HKEY_CURRENT_USER\Software\Wine\Fonts\Replacements`。这个是 Wine 特有的注册表配置，这里的设置会覆盖 Windows 系统的默认规则。这样可以通过配置 Wine 容器来定义字体匹配规则，而不是直接修改 Windows 系统本身的规则。如果用户没有特别指明，比如使用上面的默认指令创建容器，不会写入该处的注册表。
2. 如果上述的用户专属配置没有定义， Wine 会模拟 Windows 行为，Windows系统 是通过查找 `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes` 注册表，将一个逻辑字体名转换成字体族名。例如，在这里，MS Shell Dlg 2会被默认指向Tahoma。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_fonts_02.png)

## 第二棒：文件搜寻 - 寻找最佳“选手”

经过第一步的查询注册表，Wine 知道了它需要 Tahoma 这个“选手”。接下来，它会在字体库中搜寻对应的字体文件（如tahoma.ttf）。一般情况下会从三个地方进行查找：

* Wine自带字体库: 位于Wine安装目录下的.../share/wine/fonts/，提供最基础的兼容性保障。
* Linux系统字体库: 通过Linux的FontConfig系统扫描到的所有标准目录，如/usr/share/fonts/和~/.local/share/fonts/。
* 用户注册的外部字体: 在HKEY_CURRENT_USER\Software\Wine\Fonts\External Fonts中手动指定的、位于任何位置的字体文件。

在这个搜寻过程中，Wine 会择优录取。比如，如果同时在 Wine 自带库和 Linux 系统库中找到了 Tahoma，它会优先选用 Linux 系统库的版本，因为它通常质量更高。

## 第三棒：字形渲染

找到了最佳的字体文件后，Wine的图形引擎（利用FreeType）便开始工作，从文件中提取字符的字形数据（glyph），并将其绘制到屏幕上。对于大多数英文字符，到这一部分就结束了，因为几乎所有字体中都包含了英文及数字字符。但如果是中文字符，就不一定包含了对应的字形数据。这时，就需要进行第四步。

## 第四棒：字体链接 (FontLink) 

当遇到特殊情况，比如需要用 Tahoma 字体显示一个中文字符“你”时，tahoma.ttf 中根本不包含中文字形。

这时，为了避免显示方块，Wine 会启动字体链接 (FontLink) 应急预案。Wine 通过查阅注册表中的 HKEY_LOCAL_MACHINE\...\FontLink\SystemLink，查找替代规则，比如，指示 Tahoma 在遇到困难时可以求助于 SIMSUN.TTC,SimSun（宋体）。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_fonts_03.png)

如果系统中存在宋体字体文件，Wine 就会从宋体文件中提取“你”的字形，并成功完成显示。

## 为何会出现“方块字”？

理解了上述流程，方框字的成因便一目了然。方框字的出现，就是到第四步也未能找到正确的字形文件。

就拿注册表编辑器这个应用来说，界面上使用的是 "MS Shell Dlg 2"，通过第一步，知道该使用 "Tahoma"，然后查找到字体文件是 tahoma.ttf，我们知道这个字体文件中是不包含中文字形的，这时通过 FontLink 机制，Wine 会去搜索 simsun.ttc这个字体文件，或则 SimSun 字体，这在 Windows 系统下通常是可以找到的，因为中文 Windows 出厂就有宋体字库，但对于 Linux 系统却未必，因为涉及到字体版权。虽然 Linux 系统可以通过 ttf-mscorefonts-installer 安装 Windows 字体，但大多数用户并不会主动去安装。于是中文字形没有找到，就会显示成方块。

## 解决方案

方案之一就是通过 ttf-mscorefonts-installer 安装 Windows 字体，或者直接拷贝 Windows 系统 C:\Windows\Fonts 目录下的 .ttf / .ttc 文件到 Linux：
```
mkdir -p ~/.local/share/fonts
cp /path/to/Windows/Fonts/*.ttf ~/.local/share/fonts/
fc-cache -fv
```

这对于普通用户来说不太方便，而且在商业场合使用，还可能存在字体版权问题。

这个时候就推荐使用 `统信Windows应用兼容引擎`，使用 `统信Windows应用兼容引擎`，会默认为容器配置替代字体。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_fonts_04.png)

在这个界面，还可以手动添加、修改和删除字体替代，实现个性化要求。

## 小结

本文通过深入理解 Wine 这套从替换、搜寻到链接的完善字体处理流程，我们不仅能轻松解决恼人的方框字问题，更能随心所欲地定制字体配置，从而在国产系统上获得极致的 Windows 应用体验。

