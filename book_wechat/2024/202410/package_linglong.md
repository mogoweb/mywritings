# 将 QT 应用程序打包成如意玲珑软件包

在上一篇文章《[国产系统之如意玲珑](https://mp.weixin.qq.com/s/VE41M9pCsEEMCJGyRwaroQ)》中，我为大家介绍了一种创新的国产软件包 - 如意玲珑。如意玲珑（Linyaps）是一种新型的独立包管理工具集，专注于解决Linux系统下由传统软件包格式的复杂性和交叉依赖关系引起的兼容性问题。

这篇文章通过将一个简单的 QT 应用程序打包成玲珑包，给大家展示一下玲珑包的制作过程。如意玲玲项目的官网提供了一些文档和示例，但在项目实际中仍然碰到了许多问题，这个探索过程中踩了不少坑，希望能对大家有所帮助。

本文使用的开发环境为 Deepin V23 系统，Qt Creator 13.0.2，Qt 库使用的版本为 5.15.2。源码托管在

>  https://e.coding.net/mogoweb/qt-in-action/qt-in-action.git

本文使用的源码在 LingLongDemo 目录下，供参考。

## 1. 创建一个简单的 QT 工程

因为本文只是为了展示玲珑包的制作过程，所以就以一个简单的 QT Widget 应用为例。

第一步，在 Qt Creator 中新建一个 Qt Widgets Application。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202410/images/package_linglong_01.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202410/images/package_linglong_02.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202410/images/package_linglong_03.png)

简单修改以下窗体里的显示，最后运行显示如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202410/images/package_linglong_04.png)

## 2. 添加一个 .desktop 文件

在 Linux 系统中，.desktop 文件用来描述应用程序的名称、图标、可执行程序等等，应用程序只有建立了 .desktop 文件，并放在系统的正确位置，才会在起始菜单项中出现。

在Qt Creator 中新建一个空文件：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202410/images/package_linglong_05.png)

文件命名为 hello-linglong.desktop:

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202410/images/package_linglong_06.png)

文件的内容如下：

```
[Desktop Entry]
Exec=LingLongDemo
GenericName=LingLong Demo
Hidden=false
Name=LingLong Demo
StartupNotify=false
Type=Application
```

文件内容很好理解，Exec 指定了可执行程序的文件名，Name 指定应用程序名称。

## 修改 qmake 文件

修改 LingLongDemo.pro 文件的内容：

```
QT       += core gui

greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

CONFIG += c++17

# You can make your code fail to compile if it uses deprecated APIs.
# In order to do so, uncomment the following line.
#DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0x060000    # disables all the APIs deprecated before Qt 6.0.0

SOURCES += \
    main.cpp \
    mainwindow.cpp

HEADERS += \
    mainwindow.h

FORMS += \
    mainwindow.ui

# Default rules for deployment.
#qnx: target.path = /tmp/$${TARGET}/bin
#else: unix:!android: target.path = /opt/$${TARGET}/bin
target.path = $$PREFIX/bin
!isEmpty(target.path): INSTALLS += target

CONFIG(release) {
    message("Release build")
    desktop.files += hello-linglong.desktop
    desktop.path = $${PREFIX}/share/applications

    INSTALLS += desktop
}
```

其中比较关键的是后面几行，指定 target.path、desktop.files 和 desktop.path。这其中 $$PREFIX 是 qmake 命令行传入的变量定义，可以理解为最后的安装路径，后面的构建步骤将会说明。

## 创建 linglong.yaml

构建玲珑包的关键就是 linglong.yaml 配置文件，我为这个应用程序创建的 linglong.yaml 文件内容如下：

```
version: "1"
package:
  id: net.mogoweb.qt-in-action.linglongdemo
  name: LingLongDemo
  version: 0.0.0.1
  kind: app
  description: |
    simple LingLong Demo.
command:
  - /opt/apps/net.mogoweb.qt-in-action.linglongdemo/files/bin/LingLongDemo

base: org.deepin.foundation/23.0.0
runtime: org.deepin.Runtime/23.0.1
source:
  - kind: local
build: |
    qmake LingLongDemo.pro CONFIG+=release PREFIX=${PREFIX} 
    make -j${JOBS}
    make install
```

其中：

* version: 版本，注意这里是玲珑的版本，而不是应用程序的版本，当前玲珑的版本为1
* package
  * id: 应用程序的唯一id,为了避免冲突，一般用域名倒置加上应用程序包名
  * name: 应用程序名，这个名称是在玲珑包管理中展现的名称，并不是在开始菜单项中展现的名称。
  * version: 应用程序的版本号，一般要求4个用点分隔的数字
  * kind: 应用程序的类型，app 代表应用，runtime 则代表运行时
  * description: 应用程序描述
* command: 应用程序执行路径，在 UOS 应用打包规范中，app 的 安装路径为: /opt/apps/<app_id>/files/bin/，所以这里的值为：/opt/apps/net.mogoweb.qt-in-action.linglongdemo/files/bin/LingLongDemo
* base: 基础环境，因为玲珑应用是运行在容器中的，这里代表最小根文件系统，由 id 和版本号组成。先按照文档上的写。
* runtime: 应用运行时依赖，我的理解就是一些运行时库，比如 libc、qt 之类的，先按照文档上填。
* source: 在本项目中，是 local类型。但玲珑构建支持从 git 仓库下载资源并进行构建，这里先暂时不讨论。
* build: 构建命令，在本例中是新建的 qmake 工程，所以使用 qmake 命令进行构建。${PREFIX} 是玲珑构建提供的环境变量之一，可在 variable、build 字段下使用，它提供构建时的安装路径，本例中值是 /opt/apps/net.mogoweb.qt-in-action.linglongdemo

## 编译、安装与运行

打开终端，进到 Qt 应用程序源码所在的目录，执行：

```
ll-builder build
```

注意这个过程中会联网下载基础环境和运行时环境，不要挂 VPN。挂 VPN 会导致**基础环境和运行时环境**库无法下载，且没有提示，我在这个地方卡了半天，最后才发现是这个问题。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202410/images/package_linglong_07.png)

构建完成后，可以用命令先尝试者运行一下，看看有没有问题：

```
ll-builder run
```

因为是在 linglong.yaml 文件所在的目录下运行，所以不用指定参数也可以。

运行确定没问题后，就可以导出 玲珑包了。

```
ll-builder export
```

这个过程也是有些坑，要等待个几十秒，没任何输出，搞得我以为哪里又有问题。

导出的文件名为 appid.uab，所以本例中导出的文件为：

> net.mogoweb.qt-in-action.linglongdemo_x86_64_0.0.0.1_main.uab

有了这个 uab 文件，就可以安装到系统中。

```
$ ll-cli install net.mogoweb.qt-in-action.linglongdemo_x86_64_0.0.0.1_main.uab 
(295763) ./libs/linglong/src/linglong/cli/cli.cpp:522 install from file "/work/mywork/qt-in-action/source/LingLongDemo/net.mogoweb.qt-in-action.linglongdemo_x86_64_0.0.0.1_main.uab"
100% install uab successfully

```

安装之后，在系统的起始菜单中就可以看到了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202410/images/package_linglong_08.png)

也可以通过玲珑构建工具查看系统安装了哪些玲珑应用：

```
(base) alex@alex-deepin-os:/work/mywork/qt-in-action/source/LingLongDemo$ ll-cli list
id                               name                             version         arch        channel         module      description
com.163.music                    com.163.music                    1.2.1.5         x86_64      main            runtime     网易云音乐是一款专注于发现与分享的音乐产品，主打发现...
com.baidu.baidunetdisk.linyaps   com.baidu.baidunetdisk           4.17.7.6        x86_64      main            binary      convert from 4.17.7 百度网盘
net.mogoweb.qt-in-action.linglongdemo LingLongDemo                     0.0.0.1         x86_64      main            binary      simple LingLong Demo.

```

通过玲珑构建工具卸载应用：

```
(base) alex@alex-deepin-os:/work/mywork/qt-in-action/source/LingLongDemo$ ll-cli uninstall net.mogoweb.qt-in-action.linglongdemo 
(base) alex@alex-deepin-os:/work/mywork/qt-in-action/source/LingLongDemo$ ll-cli list
id                               name                             version         arch        channel         module      description
com.163.music                    com.163.music                    1.2.1.5         x86_64      main            runtime     网易云音乐是一款专注于发现与分享的音乐产品，主打发现...
com.baidu.baidunetdisk.linyaps   com.baidu.baidunetdisk           4.17.7.6        x86_64      main            binary      convert from 4.17.7 百度网盘
```

以上命令均可以加上 --help 参数，查看更详细的使用方法。

## 小结

如意玲珑打包主要依赖 linglong.yaml 文件，由于文档的不完善，刚开始做玲珑包时还是走了一些弯路。上面用一个简单的Qt 应用程序的例子做一个简单说明。大家可能会发现，这个例子中，并没有打包所依赖的 Qt 库，而是使用了系统的 Qt 库。在实际的项目中，可能会使用比较新的 Qt 库，还有的项目可能会使用复杂的构建系统，我们可能需要先 build 出应用程序，再来构建玲玲包，还可能有更复杂的环境等问题，这个在后面的文章中和大家分享，敬请关注！
