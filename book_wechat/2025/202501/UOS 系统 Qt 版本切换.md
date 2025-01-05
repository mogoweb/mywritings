最近在 UOS 1070 版本上编译Union Code 软件，结果出现如下错误：

```plain
-- process ts file: /home/alex/work/deepin-community/deepin-unioncode/assets/translations/en_US.ts
CMake Error at assets/CMakeLists.txt:25 (message):
  process ts file result : 1, with error: lupdate: could not exec                                                                                                          
  '/usr/lib/loongarch64-linux-gnu/qt4/bin/lupdate': No such file or directory                                    
```

看提示，是 lupdate 命令无法执行。但是系统中是存在 lupdate 命令的，只是它指向了 qtchooser。

```plain
alex@alex-loongson-MiniPC:~$ which lupdate
/usr/bin/lupdate
alex@alex-loongson-MiniPC:~$ ls -la /usr/bin/lupdate 
lrwxrwxrwx 1 root root 9 4月  15 22:18 /usr/bin/lupdate -> qtchooser
```

那什么是 qtchooser 呢？从名字上看，这与 Qt 版本选择有关，事实上也是如此。

`qtchooser` 是一个用于管理系统中多个 Qt 版本的工具，它允许用户选择和切换不同的 Qt 版本，以满足开发或运行环境的需求。通过 `qtchooser`，可以方便地在不同版本的 `qmake` 或其他 Qt 工具之间切换。

## `**qtchooser**`** 的作用**

1. **管理多个 Qt 版本：** 当系统中安装了多个 Qt 版本时，`qtchooser` 提供了一种机制来选择需要的版本。
2. **路径配置：** 配置和切换不同版本的 `qmake`、`uic` 等工具的路径。
3. **开发环境的灵活切换：** 对于开发者来说，可以针对不同的项目需求选择特定的 Qt 版本。

---

## **列出系统中的 Qt 版本**

运行以下命令可以列出系统中配置的 Qt 版本：

```bash
alex@alex-loongson-MiniPC:~$ qtchooser -list-versions
4
5
default
qt4-loongarch64-linux-gnu
qt4
qt5-loongarch64-linux-gnu
qt5
```

其中：

+ `4` 和 `5` 表示已配置的 Qt 4 和 Qt 5。
+ `default` 表示系统当前的默认版本。
+ 其它都是重复，可以忽略

---

## **切换 Qt 版本**

切换 Qt 版本有两种方式：临时切换和永久切换。

### **1. 临时切换**

可以通过环境变量 `QT_SELECT` 指定使用的 Qt 版本：

```bash
alex@alex-loongson-MiniPC:~$ QT_SELECT=5 qmake --version
QMake version 3.1
Using Qt version 5.11.3 in /usr/lib/loongarch64-linux-gnu

alex@alex-loongson-MiniPC:~$ QT_SELECT=4 qmake --version
qmake: could not exec '/usr/lib/loongarch64-linux-gnu/qt4/bin/qmake': No such file or directory
```

这会在当前命令中使用指定的版本，而不会影响其他终端或全局设置。

从命令执行结果上看，虽然系统配置了 4 和 5 两个版本，但实际上 Qt 4 并没有安装，所以执行才会出错。

文章开始的编译错误，也是系统默认将 Qt 版本指向 4，但版本 4 并没有安装，所以执行 lupdate 才会报错。

### **2. 永久切换**

将默认版本更改为某个版本：

```bash
export QT_SELECT=5
```

或者在 `~/.bashrc` 或 `~/.zshrc` 文件中添加这行代码以保持设置。

---

将上述环境变量设置加入到 `~/.bashrc` 后，再开启终端执行 qmake，就可以正常输出：

```bash
alex@alex-loongson-MiniPC:~$ qmake --version
QMake version 3.1
Using Qt version 5.11.3 in /usr/lib/loongarch64-linux-gnu
```

使用 qtchooser 命令输出当前的 Qt 环境变量：

```bash
alex@alex-loongson-MiniPC:~$ qtchooser --print-env
QT_SELECT="5"
QTTOOLDIR="/usr/lib/qt5/bin"
QTLIBDIR="/usr/lib/loongarch64-linux-gnu"
```

## **配置新的 Qt 版本**

如果安装了新的 Qt 版本但 `qtchooser` 中没有列出，可以通过配置文件手动添加：

1. **创建新的配置文件：**

```bash
sudo nano /usr/lib/loongarch64-linux-gnu/qtchooser/qt6.conf
```

在文件中添加两行：

```plain
/path/to/qt6/bin
/path/to/qt6/lib
```

替换 `/path/to/qt6` 为实际的 Qt 安装路径。

2. **测试配置是否成功：**

```bash
alex@alex-loongson-MiniPC:~$ qtchooser -list-versions
4
5
default
qt4-loongarch64-linux-gnu
qt4
qt5-loongarch64-linux-gnu
qt5
qt6
```

## 小结

`qtchooser` 是一个强大的工具，可以有效管理系统中多个 Qt 版本。通过它，开发者能够快速列出、切换或配置不同的 Qt 环境，以满足项目的多样化需求。无论是临时切换还是永久配置，`qtchooser` 都提供了灵活的方式。同时，合理配置新版本路径能够确保工具的扩展性，为开发工作带来更高的效率和便利性。
