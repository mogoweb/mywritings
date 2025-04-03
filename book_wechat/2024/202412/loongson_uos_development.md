# 在龙芯迷你电脑上搭建开发环境

之前写过一篇文章《(龙芯迷你主机，用来办公怎么样？)[https://mp.weixin.qq.com/s/4WtgcfzNDiaq_9M-KG9G8A]》，到现在已经使用一段时间了。这套系统使用下来，总体感觉可用，但离好用还有些距离，主要问题在于龙芯生态的应用太少了。本来UOS系统下的应用就比Windows下的应用少很多，而龙芯版UOS系统下的应用就更少了。

碰到这种情况，我们当然可以抱怨，但这无济于事。但从另外一个方面思考，龙芯上的应用这么少，而国家又要下定决心推广，是不是意味着程序员供应有缺口，这也未尝不是大家的机会，如果能掌握一些龙芯系统的开发技能，是不是未来在职场上的竞争力会高一些。

说到系统开发，目前的主流还是 C/C++，所以这里介绍一下在龙芯UOS系统上搭建 C/C++ 开发环境。

## 安装编译工具链

虽然龙芯上应用还不丰富，但是对于开发的支持还是比较完备，各种编译器、工具链都有龙芯架构的版本，不足之处是版本可能不是最新。一般情况下，也不会是太大的问题。

首先安装基本的编译工具：

```
$ sudo apt install build-essential
```

build-essential包中主要包括：

* libc6-dev
* gcc
* g++
* make
* dpkg-dev

这套工具中包含常用的编译工具，一般的程序编译基本可以满足。通过如下命令，可以看到系统带的gcc版本为8.3.0，已经是比较新的版本，支持最新的 C++ 20 标准，除非特别的代码，都可以支持。

```
$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/loongarch64-linux-gnu/8/lto-wrapper
Target: loongarch64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Uos 8.3.0.13-deepin1' --with-bugurl=file:///usr/share/doc/gcc-8/README.Bugs --enable-languages=c,c++,fortran --prefix=/usr --with-gcc-major-version-only --program-suffix=-8 --program-prefix=loongarch64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libitm --disable-libsanitizer --disable-libquadmath --disable-libquadmath-support --enable-plugin --with-system-zlib --enable-multiarch --disable-werror --with-arch=loongarch64 --with-abi=lp64 --enable-tls --disable-host-shared --disable-emultls --enable-checking=release --build=loongarch64-linux-gnu --host=loongarch64-linux-gnu --target=loongarch64-linux-gnu
Thread model: posix
gcc version 8.3.0 (Uos 8.3.0.13-deepin1)
```

编译 C/C++ 代码，除了 gcc/g++ 这套工具外，还有更强大的 clang，安装起来也非常简单：

```
$ sudo apt install clang
```

至于版本，可以通过如下命令查到：

```
$ clang --version
clang version 8.0.1-3~bpo10+1.lnd.12 
Target: loongarch64-unknown-linux-gnu
Thread model: posix
InstalledDir: /usr/bin
```

clang 版本为 8，不太新，最新的版本已经到 18.1.8，clang 版本更新特别快，clang 8 并非很老的版本，一般情况下够用。

除了编译器，诸如 ninja、cmake 等工具也是 C/C++ 项目中常用的，可以先安装起来：

```
$ sudo apt install ninja-build cmake git gdb
```

ninja 的版本是 1.10.1，cmake 的版本是 3.22.1。

## 安装 Qt Creator

在国产信创系统下开发 C/C++ 应用，首推 Qt 框架。Qt 框架有多好用不用我多说，而开发 Qt 应用，最好的 IDE 工具是 Qt Creator。

在 Windows 和 Linux x86 架构下，我们通常会去 Qt 官网下载 Qt 社区版安装器，里面可以选择不同的安装组件，但非常可惜的是，并没有龙芯架构。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_uos_development_01.jpg)

不过也不用担心，在龙芯 UOS 系统下安装Qt 开发工具相当简单，只需要一条命令：

```
$ sudo apt install qtcreator qt5-default
```

其中 qt5-default 包中主要包括：

* qtbase: Qt 基础模块的集合，比如 widgets, Gui 等；
* qmake: qmake 是 Qt 项目的构建工具，通过 qmake 将 .pro 文件生成 make 文件，进而完成项目的编译;

qtcreator 包中主要包括：

* qtcreator: Qt官方的 IDE；
* qt助手: Qt 所有模块的说明文档；
* qt linguist: Qt 进行文字内容国际化的可视化工具，帮助开发者对程序中需要翻译的文字内容进行对应语言的翻译；
* qt设计器: Qt 对 UI 文件进行布置的可视化工具。

## 安装 deepin Union Code

在 UOS/Deepin 系统上开发 Qt 应用程序，多了一个选择，就是 deepin Union Code（以前叫 deepin-IDE ）。

deepin Union Code 是深度科技推出的一款集成开发环境（IDE），专为开发者提供高效、简洁的开发体验。它的设计灵感来源于国产操作系统和本地化开发需求，旨在为开发者提供一个更加符合中国市场需求的开发工具。

值得一提的是，deepin Union Code 中的智能插件是和智谱厂商合作，现已实现了智能问答、代码翻译、添加注释、代码生成等功能。

deepin Union Code 的主要特点：

1. **国产化与本土化优化**
   - **深度集成 deepin 生态**：deepin Union Code 完全适配 deepin/UOS 操作系统，能够更好地与 deepin/UOS 系统的桌面环境和工具链集成，如 deepin 的文件管理器、窗口管理器等，提供无缝的使用体验。
   - **支持国产操作系统与硬件**：deepin Union Code 针对国产操作系统进行了优化，能够更好地运行在国产芯片（如飞腾、龙芯等）上。

2. **简洁易用的界面设计**
   - **深度简化的用户界面**：deepin Union Code 的界面简洁、直观，用户界面设计符合国内用户的使用习惯。
   - **完全集成开发工具**：内置了代码编辑、调试、版本管理、终端等工具，开发者无需再安装额外的插件和扩展，减少了插件冲突和管理负担。

3. **开发效率与性能优化**
   - **资源占用更少**：在资源占用方面进行了优化，尤其适合低配置机器，启动速度更快，运行更加流畅。
   - **集成的代码分析与调试工具**：提供了强大的代码分析和调试工具，支持实时查看代码中的潜在问题，并通过图形化界面进行调试，提高开发效率。

3. **多语言支持**
   - **支持多种编程语言**：支持主流的编程语言（如 C/C++、Java、Python、JavaScript 等），提供适应多种开发场景的功能。

4. **AI集成**
   - **智能问答**：开发中遇到的技术问题，可直接向 AI 提问。无需离开 IDE 环境去搜索引擎寻找答案，让开发者更沉浸于开发环境。
   - **代码翻译**：基于 AI 大模型对代码进行语义级翻译，支持多种编程语言互译。
   - **自动添加注释**：支持给代码自动添加行级注释，节省大量开发时间。没有注释的历史代码，也不再是问题。
   - **代码生成和补全**：根据自然语言注释描述的功能自动生成代码，也可以根据已有的代码自动生成后续代码，补全当前行或生成后续若干行，帮助提高编程效率。

要安装 deepin Union Code 也很简单，在应用商店中搜索 deepin-IDE，点击安装即可。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_uos_development_02.jpg)

## 安装 VS Code

除了 Qt Creator 和 deepin Union Code，还有一个流行的编辑器 VS Code。VS Code 严格意义上不是一款IDE，但配合插件扩展，支持多种编程语言，非常适合进行跨平台开发。

在龙芯 UOS 系统上安装 VS Code 也非常简单，在应用商店中搜索 VS Code，点击安装即可。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_uos_development_03.jpg)

## 总结

至此，一套完整的 C/C++ 开发环境已经搭建完成，接下来就可以开始编写程序了。

龙芯 UOS 系统虽然生态还不完善，但作为国产操作系统，其发展潜力巨大。通过学习和掌握这套系统的开发技能，不仅可以提升个人能力，还能为推动国产软件的发展做出贡献。

希望这篇文章能帮助你在龙芯迷你电脑上成功搭建起一个高效的开发环境，祝你编码愉快！



----------------------------------------------------

以下是润色后的文章：

---

# 在龙芯迷你电脑上搭建开发环境

之前我写过一篇文章《[龙芯迷你主机，用来办公怎么样？](https://mp.weixin.qq.com/s/4WtgcfzNDiaq_9M-KG9G8A)》，到现在已经使用了一段时间。整体体验下来，系统是可用的，但离完美仍有差距，主要原因是龙芯生态中的应用还非常匮乏。原本在 UOS 系统下，应用就比 Windows 少很多，而龙芯版 UOS 系统的应用更加稀缺。

面对这样的困境，我们可以抱怨，但并没有太大意义。反过来思考，龙芯上的应用稀缺，国家又决心推广，这是否意味着开发人才存在缺口？这或许是一个机遇。如果能掌握一些龙芯系统的开发技能，未来在职场上的竞争力或许会大大提升。

既然如此，接下来就介绍如何在龙芯 UOS 系统上搭建 C/C++ 开发环境。

## 安装编译工具链

尽管龙芯生态尚不成熟，但其开发支持相对完备，已有多种编译器和工具链版本适配龙芯架构。唯一不足之处在于版本可能不是最新的，但通常这并不妨碍使用。

首先，安装基本的编译工具：

```bash
$ sudo apt install build-essential
```

`build-essential` 包含以下常用工具：

- `libc6-dev`
- `gcc`
- `g++`
- `make`
- `dpkg-dev`

这些工具可以满足大多数程序编译需求。通过以下命令查看系统自带的 GCC 版本：

```bash
$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/loongarch64-linux-gnu/8/lto-wrapper
Target: loongarch64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Uos 8.3.0.13-deepin1' --with-bugurl=file:///usr/share/doc/gcc-8/README.Bugs --enable-languages=c,c++,fortran --prefix=/usr --with-gcc-major-version-only --program-suffix=-8 --program-prefix=loongarch64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libitm --disable-libsanitizer --disable-libquadmath --disable-libquadmath-support --enable-plugin --with-system-zlib --enable-multiarch --disable-werror --with-arch=loongarch64 --with-abi=lp64 --enable-tls --disable-host-shared --disable-emultls --enable-checking=release --build=loongarch64-linux-gnu --host=loongarch64-linux-gnu --target=loongarch64-linux-gnu
Thread model: posix
gcc version 8.3.0 (Uos 8.3.0.13-deepin1)
```

该版本的 GCC 为 8.3.0，支持 C++ 20 标准，除非有特殊需求，通常足够使用。

除了 GCC/G++，Clang 也是一个强大的编译器，安装也非常简单：

```bash
$ sudo apt install clang
```

查看 Clang 版本：

```bash
$ clang --version
clang version 8.0.1-3~bpo10+1.lnd.12
Target: loongarch64-unknown-linux-gnu
Thread model: posix
InstalledDir: /usr/bin
```

Clang 版本为 8，尽管更新版本已经达到 18.1.8，但 Clang 8 在大多数情况下仍然足够使用。

除了编译器，像 Ninja 和 CMake 等工具也在 C/C++ 项目中广泛使用，可以通过以下命令安装：

```bash
$ sudo apt install ninja-build cmake git gdb
```

其中 Ninja 的版本为 1.10.1，CMake 为 3.22.1。

## 安装 Qt Creator

对于国产信创系统，Qt 框架是开发 C/C++ 应用的首选。Qt 强大的跨平台特性不言而喻，而开发 Qt 应用的最佳 IDE 工具便是 Qt Creator。

在 Windows 和 Linux x86 架构下，我们通常可以从 Qt 官网下载 Qt 社区版安装器，选择所需组件进行安装，但遗憾的是，龙芯架构并未在官网提供支持。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_uos_development_01.png)

不过别担心，在龙芯 UOS 系统上安装 Qt 开发工具非常简单，只需执行以下命令：

```bash
$ sudo apt install qtcreator qt5-default
```

`qt5-default` 包含以下内容：

- `qtbase`: Qt 基础模块集合（如 Widgets、Gui 等）
- `qmake`: Qt 项目构建工具，将 `.pro` 文件转换为 Makefile 以进行编译。

`qtcreator` 包含：

- `qtcreator`: Qt 官方 IDE
- `qt助手`: Qt 模块的文档
- `qt linguist`: 文字内容国际化工具
- `qt设计器`: UI 布局设计工具

## 安装 deepin Union Code

在 UOS/deepin 系统中，除了 Qt Creator 外，还有一个不错的选择，那就是 deepin Union Code（前身为 deepin-IDE）。它是深度科技推出的一款集成开发环境（IDE），专为开发者提供高效、简洁的开发体验。

deepin Union Code 的特色如下：

1. **国产化与本土化优化**
   - **深度集成 deepin 生态**：与 deepin/UOS 系统无缝集成，提供卓越的使用体验。
   - **支持国产操作系统与硬件**：针对国产操作系统进行了优化，能够在飞腾、龙芯等国产芯片上更好运行。

2. **简洁易用的界面设计**
   - **深度简化的用户界面**：符合国内用户的使用习惯，操作直观。
   - **完全集成开发工具**：集成了代码编辑、调试、版本管理、终端等工具，减少插件冲突和管理负担。

3. **开发效率与性能优化**
   - **资源占用少**：优化了资源占用，尤其适合低配置机器。
   - **集成的代码分析与调试工具**：支持实时查看潜在问题并通过图形化界面调试，提升开发效率。

4. **多语言支持**
   - **支持多种编程语言**：支持 C/C++、Java、Python、JavaScript 等多种编程语言。

5. **AI集成**
   - **智能问答**：开发中遇到的技术问题，可直接向 AI 提问。无需离开 IDE 环境去搜索引擎寻找答案，让开发者更沉浸于开发环境。
   - **代码翻译**：基于 AI 大模型对代码进行语义级翻译，支持多种编程语言互译。
   - **自动添加注释**：支持给代码自动添加行级注释，节省大量开发时间。没有注释的历史代码，也不再是问题。
   - **代码生成和补全**：根据自然语言注释描述的功能自动生成代码，也可以根据已有的代码自动生成后续代码，补全当前行或生成后续若干行，帮助提高编程效率。

安装 deepin Union Code 同样简单，在应用商店搜索 `deepin-IDE` 即可。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_uos_development_02.png)

## 安装 VS Code

除了 Qt Creator 和 deepin Union Code，另一个流行的开发工具是 VS Code。虽然 VS Code 严格来说是一个编辑器，而非 IDE，但配合插件，它能支持多种编程语言，适合跨平台开发。

VS Code官方并没有提供龙芯架构的支持，不过在龙芯 UOS 系统中，安装 VS Code 也非常简单，只需在应用商店中搜索并安装。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202412/images/loongson_uos_development_03.png)

## 总结

至此，我们已经成功搭建了一套完整的 C/C++ 开发环境，接下来可以开始编写代码了。

尽管龙芯 UOS 系统的生态还在发展中，但作为国产操作系统，它具有巨大的发展潜力。通过掌握这套系统的开发技能，既能提升个人能力，也能为国产软件的发展贡献力量。

希望这篇文章能帮助你在龙芯迷你电脑上成功搭建高效的开发环境，祝你编码愉快！
