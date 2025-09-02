# 从点击 Wine 应用，到应用程序启动，这中间发生了什么？

说起与 UOS 结缘，还要回溯到五年前，那时我在兆芯的关联公司工作，人事关系挂在兆芯，也和兆芯的员工在同一间办公室上班。兆芯的主营业务是做 X86 架构的国产CPU，当时我工位隔壁坐着测试的妹子，整天在测试 UOS 系统。我在旁边看着 UOS 系统，那 UI 界面打动了我（我的主力系统是 Ubuntu，朴实无华，但超级稳定）。出于支持国产芯片的想法，购买了一台搭载兆芯 CPU 的迷你主机，安装的操作系统是 UOS。就是下面这台，陪伴我快 5 年时间了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202509/images/wine_app_01.png)

详情参见：[其实，我们也在努力做CPU](https://mp.weixin.qq.com/s/Cy0NITwMy07ySpmvZB4mOA)

那个时候 UOS 已经有应用商店，令我比较好奇的是应用商店中的一些 Windows 应用，是如何打包成 debian 包的。因为我当时使用的一款围棋客户端软件，只有 Windows 版本，而应用商店当时并没有，所以我也希望能够做一个 debian 包，一键安装。于是我就下载了应用商店的 QQ（Windows 版本），研究了包的内容，和一些脚本，然后依葫芦画瓢，也做了一个 debian 包，还提交到 UOS 应用商店。

如今，有了统信 Windows 应用兼容引擎，可以一键打包，方便不少，不需要像以前那样折腾。不过我觉得对于程序员来说，了解这个过程还是挺有用的。比如我在 deepin v25 系统上从应用商店下载安装了剪映（Windows）版，点击时出现闪退。作为一名程序员，自然就会像弄清楚到底发生了什么？虽然应用商店上架应用之前都会经过测试，但个人的环境各不相同，测试并不能面面俱到。如果了解一点程序的运行原理，就可以根据实际环境分析，看看哪里出了问题。

下面就以剪映这款应用，分析一下从点击应用，到应用程序启动这个过程中，会有哪些处理流程。如下分析借助了 AI。

在安装应用程序之后，会在启动器上添加一项程序入口，这是通过在 /usr/share/applications/ 目录添加一个 .desktop 实现的。剪映这个应用对应的就是 com.ulikecam.jianying.deepin.desktop。

我们用文本编辑器打开这个 desktop 文件，可以看到如下内容：

```
#!/usr/bin/env xdg-open

[Desktop Entry]
Encoding=UTF-8
Type=Application
X-Created-By=Deepin WINE Team
Categories=Video;
Icon=com.ulikecam.jianying.deepin
Exec="/opt/apps/com.ulikecam.jianying.deepin/files/run.sh" -f %f
Name=jianying
Name[zh_CN]=剪映
Comment=剪映专业版：全能易用的桌面端剪辑软件
StartupWMClass=com.ulikecam.jianying.deepin
StartupNotify=true
MimeType=
```

其中 Exec= 这一行指明了，在我们点击启动器的剪映图标时，执行的就是 /opt/apps/com.ulikecam.jianying.deepin/files/run.sh 这个脚本，后面带的参数暂时可以不用管，这是为了右键传入文件名的情形。

我们再来看 run.sh 这个脚本的内容：

```
#!/bin/bash

version_gt() { test "$(echo "$@" | tr " " "\n" | sort -V | head -n 1)" != "$1"; }

ACTIVEX_NAME=""
BOTTLENAME="com.ulikecam.jianying.deepin"
APPVER="3.0.5.8542deepin2"
EXEC_PATH="c:/JianyingPro/JianyingPro.exe"
START_SHELL_PATH="/opt/deepinwine/tools/run_v4.sh"
export MIME_TYPE=""
export MIME_EXEC=""
export DEB_PACKAGE_NAME="com.ulikecam.jianying.deepin"
export APPRUN_CMD="deepin-wine8-stable"
DISABLE_ATTACH_FILE_DIALOG=""
EXPORT_ENVS=""
EXEC_NAME="JianyingPro.exe"
INSTALL_SETUP=""
export BOX86_EMU_CMD="/opt/deepin-box86/stable/box86"
export DEF_EMU_NAME="exagear"
export SPECIFY_SHELL_DIR=${START_SHELL_PATH%/*}
export WINESERVICESNODELAY=""
export WINESERVICESDISABLE=""
export WINE_WMCLASS="com.ulikecam.jianying.deepin"

ARCHIVE_FILE_DIR="/opt/apps/$DEB_PACKAGE_NAME/files"

if [ -z "$APPRUN_CMD" ];then
    export APPRUN_CMD="/opt/deepin-wine6-stable/bin/wine"
fi
if [ -f "$APPRUN_CMD" ];then
    wine_path=$(dirname $APPRUN_CMD)
    wine_path=$(realpath "$wine_path/../")
    export WINEDLLPATH=$wine_path/lib:$wine_path/lib64
else
    export WINEDLLPATH=/opt/$APPRUN_CMD/lib:/opt/$APPRUN_CMD/lib64
fi

export WINEPREDLL="$ARCHIVE_FILE_DIR/dlls"

_SetRegistryValue()
{
    env WINEPREFIX="$BOTTLEPATH" $APPRUN_CMD reg ADD "$1" /v "$2" /t "$3" /d "$4" /f
}

if [ -z "$DISABLE_ATTACH_FILE_DIALOG" ];then
    export ATTACH_FILE_DIALOG=1
fi

if [ -n "$EXPORT_ENVS" ];then
    export $EXPORT_ENVS
fi

# 打包安装程序的情况
if [[ -z "$EXEC_PATH" ]] && [[ -n "$INSTALL_SETUP" ]];then
    $START_SHELL_PATH $BOTTLENAME $APPVER "$EXEC_PATH" -c
    BOTTLEPATH="$HOME/.deepinwine/$BOTTLENAME"
    EXEC_PATH=$(find "$BOTTLEPATH" -name $EXEC_NAME | head -1)
    if [ -z "$EXEC_PATH" ];then
        _SetRegistryValue "HKCU\\Software\\Wine\\DllOverrides" winemenubuilder.exe REG_SZ
        WINEPREFIX="$BOTTLEPATH" $APPRUN_CMD "$ARCHIVE_FILE_DIR/$INSTALL_SETUP"
        EXEC_PATH=$(find "$BOTTLEPATH" -name $EXEC_NAME | head -1)

        cp "$ARCHIVE_FILE_DIR/setup.md5sum" "$BOTTLEPATH"
    fi

    if [ -z "$EXEC_PATH" ];then
        echo "安装失败退出"
        exit
    fi
fi

if [ -n "$EXEC_PATH" ];then
    if [ -z "${EXEC_PATH##*.lnk*}" ];then
        $START_SHELL_PATH $BOTTLENAME $APPVER "C:/windows/command/start.exe" "/Unix" "$EXEC_PATH" "$@"
    elif [ -z "${EXEC_PATH##*.bat}" ];then
        $START_SHELL_PATH $BOTTLENAME $APPVER "cmd" -f /c "$EXEC_PATH" "${@:2}"
    else
        $START_SHELL_PATH $BOTTLENAME $APPVER "$EXEC_PATH" "$@"
    fi
elif [ -n "$ACTIVEX_NAME" ] && [ $# -gt 1 ];then
    $START_SHELL_PATH $BOTTLENAME $APPVER "$1" -f "${@:2}"
else
    $START_SHELL_PATH $BOTTLENAME $APPVER "uninstaller.exe" "$@"
fi
```

脚本的执行逻辑可以分为三大块：

1. 静态配置 (Static Configuration): 定义程序运行所需的核心信息，如程序名、版本号、路径等。这些变量在脚本开头就已设定。
2. 动态环境设置 (Dynamic Environment Setup): 根据静态配置，动态地生成和导出一些环境变量，主要是为了配置 Wine 的运行环境，比如动态链接库 (DLL) 的搜索路径。
3. 执行逻辑 (Execution Logic): 这是脚本的最终动作。它根据不同的条件（例如，是首次安装还是正常启动，启动的是 .exe 文件还是 .bat 文件），选择不同的命令行参数去调用一个更底层的通用启动脚本 (run_v4.sh)。

当用户在 Deepin 系统中点击剪映的图标时，实际就是执行了这个 Shell 脚本。整个过程如下：

1. 初始化: 脚本加载，定义了所有常量，如容器名 com.ulikecam.jianying.deepin 和程序路径 c:/JianyingPro/JianyingPro.exe。
2. 环境配置: 脚本自动计算并导出了 WINEDLLPATH 和 WINEPREDLL 等关键环境变量，为 deepin-wine8-stable 准备好运行环境。
3. 跳过安装: 脚本检测到 EXEC_PATH 已存在，因此跳过了首次安装的逻辑。
4. 最终调用: 脚本进入执行逻辑，判断出 EXEC_PATH 是一个 .exe 文件，于是执行 else 分支。
5. 启动程序: 它最终调用了 /opt/deepinwine/tools/run_v4.sh，并把所有准备好的信息（容器名、版本、程序路径、用户参数）作为命令行参数传递过去。run_v4.sh 脚本会利用这些信息，在正确的 Wine 容器中启动 JianyingPro.exe。

这也可以解释为什么 Wine 应用为什么第一次启动会慢一些，因为第一次启动涉及 Wine 容器的创建等初始化动作，后面再启动就可以省掉这个动作，直接启动应用程序。

上面的脚本，还会调用 run_v4.sh 脚本，其实这个脚本才是最终做事情的。

如果我们查看 /opt/deepinwine/tools/ 这个目录下的文件，发现还有 run.sh、run_v1.sh ~ run_v4.sh 等脚本，这也是不断优化的结果。之前的脚本还保留，是出于兼容性考虑，如果用户的应用没有升级，可能还会使用到以前的脚本。

下面再来分析真正做事情的 run_v4.sh 脚本。

```
#!/bin/bash

#   Copyright (C) 2016 Deepin, Inc.
#
#   Author:     Li LongYu <lilongyu@linuxdeepin.com>
#               Peng Hao <penghao@linuxdeepin.com>
BOTTLENAME="$1"
WINEPREFIX="$HOME/.deepinwine/$1"
APPDIR="/opt/apps/${DEB_PACKAGE_NAME}/files"
APPVER=""
APPTAR="files.7z"
WINE_CMD="deepin-wine"
LOG_FILE="$0"
PUBLIC_DIR="/var/public"
export WINEBANNER=1
BANNER_DATE=$(date +%s)

SHELL_DIR=${0%/*}
if [ $SPECIFY_SHELL_DIR ]; then
    SHELL_DIR="$SPECIFY_SHELL_DIR"
fi

if [ $APPRUN_CMD ]; then
    WINE_CMD="$APPRUN_CMD"
fi

UsePublicDir()
{
    if [ -z "$USE_PUBLIC_DIR" ]; then
        echo "Don't use public dir"
        return 1
    fi
    if [ ! -d "$PUBLIC_DIR" ];then
        echo "Not found $PUBLIC_DIR"
        return 1
    fi
    if [ ! -r "$PUBLIC_DIR" ];then
        echo "Can't read for $PUBLIC_DIR"
        return 1
    fi
    if [ ! -w "$PUBLIC_DIR" ];then
        echo "Can't write for $PUBLIC_DIR"
        return 1
    fi
    if [ ! -x "$PUBLIC_DIR" ];then
        echo "Can't excute for $PUBLIC_DIR"
        return 1
    fi

    return 0
}

_DeleteRegistry()
{
    env WINEPREFIX="$WINEPREFIX" "$WINE_CMD" reg DELETE "$1" /f &> /dev/null
}

init_log_file()
{
    if [ ! -d "$DEBUG_LOG" ];then
        return
    fi

    LOG_DIR=$(realpath "$DEBUG_LOG")
    if [ -d "$LOG_DIR" ];then
        LOG_FILE="${LOG_DIR}/${LOG_FILE##*/}.log"
        echo "" > "$LOG_FILE"
        debug_log "LOG_FILE=$LOG_FILE"
    fi
}

debug_log_to_file()
{
    if [ -d "$DEBUG_LOG" ];then
        echo -e "${1}" >> "$LOG_FILE"
    fi
}

debug_log()
{
    echo "${1}"
}

HelpApp()
{
        echo " Extra Commands:"
        echo " -r/--reset     Reset app to fix errors"
        echo " -e/--remove    Remove deployed app files"
        echo " -h/--help      Show program help info"
}

check_link()
{
    if [ ! -d "$1" ];then
        echo "$1 不是目录，不能创建$2软连接"
        return
    fi
    link_path=$(realpath "$2")
    target_path=$(realpath "$1")
    if [ "$link_path" != "$target_path" ];then
        if [ -d "$2" ];then
            mv "$2" "${2}.bak"
        elif [ -e "$2" ];then
            rm "$2"
        fi
        echo "修复$2软连接为$1"
        ln -s -f "$1" "$2"
    fi
}

FixLink()
{
    if [ -d "${WINEPREFIX}" ]; then
        CUR_DIR=$PWD
        cd "${WINEPREFIX}/dosdevices"
        check_link ../drive_c c:
        check_link / z:
        check_link "$HOME" y:
        cd "../drive_c/users/$USER"
        check_link "$HOME/Desktop" Desktop
        check_link "$HOME/Downloads" Downloads
        if [[ "$WINE_CMD" == *"deepin-wine8-stable"* ]];then
            check_link "$HOME/Documents" Documents
        else
            check_link "$HOME/Documents" "My Documents"
        fi
        cd "$CUR_DIR"
        #ls -l "${WINEPREFIX}/dosdevices"
    fi
}

DisableWrite()
{
    if [ -d "${1}" ]; then
        chmod +w "${1}"
        rm -rf "${1}"
    fi

    mkdir "${1}"
    chmod -w "${1}"
}

is_autostart()
{
    AUTOSTART="/opt/deepinwine/tools/autostart"
    if [ -f "$AUTOSTART.all" ]&&[ -f "/opt/apps/$1/files/run.sh" ];then
        return 0
    fi

    if [ -f "$AUTOSTART" ];then
        grep -c "$1" "$AUTOSTART" > /dev/null
        return $?
    fi

    return 1
}

Test_GL_wine()
{
    gl_wine_path="/opt/deepinwine/tools/gl-wine"

    #如果不支持32的GLX,d3d改为gdi的实现
    if [[ ! -f "${WINEPREFIX}/.init_d3d" ]];then
        if [[ "$WINE_CMD" == *"deepin-wine8-stable"* ]];then
            gl_wine="${gl_wine_path}/gl-wine64"
        else
            gl_wine="${gl_wine_path}/run_gl.sh"
        fi

        run_gl=`${gl_wine} 2>&1`

        #如果opengl测试程序运行失败，所有进程的渲染方式改为gdi渲染模式
        if [ $? != 0 ];then
            WINEPREFIX="$WINEPREFIX" $WINE_CMD regedit /S "${gl_wine_path}/gdid3d.reg"
        fi

        touch "${WINEPREFIX}/.init_d3d"
    fi
}

#arg 1: windows process file path
#arg 2-*: windows process args
CallProcess()
{
    #get file full path
    path="$1"
    path=${path/c:/${WINEPREFIX}/drive_c}
    path=${path//\\/\/}

    #kill bloack process
    is_autostart $DEB_PACKAGE_NAME
    autostart=$?
    if [[ $autostart -ne 0 ]] && [[ "$1" != *"pluginloader.exe" ]];then
        "$SHELL_DIR/kill.sh" "$BOTTLENAME" block
    fi

    #run gl-wine for test opengl
    get_arch=`arch`
    if [[ $get_arch = "x86_64" ]];then
        Test_GL_wine
    fi

    #change current dir to excute path
    path=${path%/*}
    cd "$path"
    #pwd

    #Set default mime type
    if [ -n "$MIME_TYPE" ]; then
        xdg-mime default "$DEB_PACKAGE_NAME".desktop "$MIME_TYPE"
    fi

    if [ -z "$WINEDLLOVERRIDES" ];then
        export WINEDLLOVERRIDES="winemenubuilder.exe="
    fi

    debug_log "Starting process $* ..."

    #start autobottle
    if [ $autostart -eq 0 ];then
        env WINEPREFIX="$WINEPREFIX" $WINE_CMD "$@" &
        "$SHELL_DIR/autostart_wine.sh" $DEB_PACKAGE_NAME
    else
        env WINEPREFIX="$WINEPREFIX" $WINE_CMD "$@"
        "$SHELL_DIR/deepin-wine-banner" check $BANNER_DATE
    fi
}

CallZhuMu()
{
    #change current dir to excute path
    path=$(dirname "$path")
    cd "$path"
    pwd

    #Set default mime type
    if [ -n "$MIME_TYPE" ]; then
        xdg-mime default "$DEB_PACKAGE_NAME".desktop "$MIME_TYPE"
    fi

    debug_log_to_file "Starting process $* ..."
    if [ -n "$2" ];then
        env WINEPREFIX="$WINEPREFIX" $WINE_CMD "$1" "--url=$2" &
    else
        env WINEPREFIX="$WINEPREFIX" $WINE_CMD "$1" &
    fi
}

CallQQGame()
{
    debug_log "run $1"
    "$SHELL_DIR/kill.sh" qqgame block
    env WINEPREFIX="$WINEPREFIX" $WINE_CMD "$1" &
}

CallQQ()
{
    if [ ! -f "$WINEPREFIX/../.QQ_run" ]; then
        debug_log "first run time"
        "$SHELL_DIR/add_hotkeys"
        "$SHELL_DIR/fontconfig"
        touch "$WINEPREFIX/../.QQ_run"
    fi

    DisableWrite "${WINEPREFIX}/drive_c/Program Files/Tencent/QQ/Bin/QQLiveMPlayer"
    DisableWrite "${WINEPREFIX}/drive_c/Program Files/Tencent/QQ/Bin/QQLiveMPlayer1"
    DisableWrite "${WINEPREFIX}/drive_c/Program Files/Tencent/QzoneMusic"

    DisableWrite "${WINEPREFIX}/drive_c/Program Files/Tencent/QQBrowser"
    DisableWrite "${WINEPREFIX}/drive_c/Program Files/Common Files/Tencent/QQBrowser"
    DisableWrite "${WINEPREFIX}/drive_c/users/Public/Application Data/Tencent/QQBrowserBin"
    DisableWrite "${WINEPREFIX}/drive_c/users/Public/Application Data/Tencent/QQBrowserDefault"
    DisableWrite "${WINEPREFIX}/drive_c/users/${USER}/Application Data/Tencent/QQBrowserDefault"

    DisableWrite "${WINEPREFIX}/drive_c/users/Public/Application Data/Tencent/QQPCMgr"
    DisableWrite "${WINEPREFIX}/drive_c/Program Files/Common Files/Tencent/QQPCMgr"

    DisableWrite "${WINEPREFIX}/drive_c/Program Files/Common Files/Tencent/HuaYang"
    DisableWrite "${WINEPREFIX}/drive_c/users/${USER}/Application Data/Tencent/HuaYang"

    CallProcess "$@"
}

CallTIM()
{
    if [ ! -f "$WINEPREFIX/../.QQ_run" ]; then
        debug_log "first run time"
        "$SHELL_DIR/add_hotkeys"
        "$SHELL_DIR/fontconfig"
        # If the bottle not exists, run reg may cost lots of times
        # So create the bottle befor run reg
        env WINEPREFIX="$WINEPREFIX" $WINE_CMD uninstaller --list
        touch $WINEPREFIX/../.QQ_run
    fi

    CallProcess "$@"

    #disable Tencent MiniBrowser
    _DeleteRegistry "HKCU\\Software\\Tencent\\MiniBrowser"
}

CallWeChat()
{
    export DISABLE_RENDER_CLIPBOARD=1
    CallProcess "$@"
}

CallWangWang()
{
    chmod 700 "$WINEPREFIX/drive_c/Program Files/AliWangWang/9.12.10C/wwbizsrv.exe"
    chmod 700 "$WINEPREFIX/drive_c/Program Files/Alibaba/wwbizsrv/wwbizsrv.exe"
    if [ $# = 3 ] && [ -z "$3" ];then
        EXEC_PATH="c:/Program Files/AliWangWang/9.12.10C/WWCmd.exe"
        env WINEPREFIX="$WINEPREFIX" $WINE_CMD "$EXEC_PATH" "$2" &
    else
        CallProcess "$@"
    fi
}

CallWXWork()
{
    if [ -d "${WINEPREFIX}/drive_c/users/${USER}/Application Data/Tencent/WXWork/Update" ]; then
        rm -rf "${WINEPREFIX}/drive_c/users/${USER}/Application Data/Tencent/WXWork/Update"
    fi
    if [ -d "${WINEPREFIX}/drive_c/users/${USER}/Application Data/Tencent/WXWork/upgrade" ]; then
        rm -rf "${WINEPREFIX}/drive_c/users/${USER}/Application Data/Tencent/WXWork/upgrade"
    fi
    #Support use native file dialog

    CallProcess "$@"
}

CallDingTalk()
{
    debug_log "run $1"
    "$SHELL_DIR/kill.sh" DingTalk block

    CallProcess "$@"
}

urldecode() { : "${*//+/ }"; echo -e "${_//%/\\x}"; }

CallMeiTuXiuXiu()
{
    #set -- "$1" "${2#file://*}"
    local path=$(urldecode "$2")
    path=${path/file:\/\//}
    set -- "$1" "$path"
    CallProcess "$@"
}

CallFastReadPDF()
{
    #set -- "$1" "${2#file://*}"
    local path=$(urldecode "$2")
    path=${path/file:\/\//}
    set -- "$1" "$path"
    CallProcess "$@"
}

CallEvernote()
{
    local path=$(urldecode "$2")
    path=${path/file:\/\//}
    set -- "$1" "$path"
    CallProcess "$@"
}

CallTencentVideo()
{
    if [ -f "${WINEPREFIX}/drive_c/Program Files/Tencent/QQLive/Upgrade.dll" ]; then
        rm -rf "${WINEPREFIX}/drive_c/Program Files/Tencent/QQLive/Upgrade.dll"
    fi

    CallProcess "$@"
}

CallFoxmail()
{
    sed -i '/LogPixels/d' ${WINEPREFIX}/user.reg
    CallProcess "$@"
}

CallTHS()
{
    $SHELL_DIR/kill.sh ths block

    debug_log "Start run $1"
    #get file full path
    path="$1"
    path=$(echo ${path/c:/${WINEPREFIX}/drive_c})
    path=$(echo ${path//\\/\/})

    #kill bloack process
    name="${path##*/}"
    $SHELL_DIR/kill.sh "$name" block

    #change current dir to excute path
    path=$(dirname "$path")
    cd "$path"
    pwd

    #Set default mime type
    if [ -n "$MIME_TYPE" ]; then
        xdg-mime default "$DEB_PACKAGE_NAME".desktop "$MIME_TYPE"
    fi

    env WINEPREFIX="$WINEPREFIX" $WINE_CMD "$@" &
}

CallQQGameV2()
{
    debug_log "run $1"
    "$SHELL_DIR/kill.sh" QQMicroGameBox block
    env WINEPREFIX="$WINEPREFIX" $WINE_CMD "$1" -action:force_download -appid:${2} -pid:8 -bin_version:1.1.2.4 -loginuin: &
}

CallPsCs6()
{
    #get file full path
    path="$1"
    path=$(echo ${path/c:/${WINEPREFIX}/drive_c})
    path=$(echo ${path//\\/\/})

    #kill bloack process
    name="${path##*/}"
    "$SHELL_DIR/kill.sh" "$name" block

    #change current dir to excute path
    path=$(dirname "$path")
    cd "$path"
    pwd

    #Set default mime type
    if [ -n "$MIME_TYPE" ]; then
        xdg-mime default "$DEB_PACKAGE_NAME".desktop "$MIME_TYPE"
    fi

    debug_log_to_file "Starting process $* ..."

    env WINEPREFIX="$WINEPREFIX" $WINE_CMD "$@" &
}

UnixUriToDosPath()
{
    OPEN_FILE="$1"
    if [ -f "$OPEN_FILE" ]; then
        OPEN_FILE=$(realpath "$OPEN_FILE")
        OPEN_FILE="z:$OPEN_FILE"
        OPEN_FILE=$(echo $OPEN_FILE | sed -e 's/\//\\\\/g')
    fi
    echo $OPEN_FILE
}

#arg 1: exec file path
#arg 2: autostart ,or exec arg 1
#arg 3: exec arg 2
CallApp()
{
    FixLink
    debug_log "CallApp $BOTTLENAME arg count $#: $*"

    if [ -f "/opt/apps/${DEB_PACKAGE_NAME}/files/pre_run.sh" ];then
        source "/opt/apps/${DEB_PACKAGE_NAME}/files/pre_run.sh"
        CallPreRun "$@"
    fi

    case $BOTTLENAME in
        "Deepin-WangWang")
            CallWangWang "$@"
            ;;
        "Deepin-ZhuMu")
            CallZhuMu "$@"
            ;;
        "Deepin-QQ")
            CallQQ "$@"
            ;;
        "Deepin-TIM")
            CallTIM "$@"
            ;;
        "Deepin-QQGame"*)
            CallQQGame "$@"
            ;;
        "Deepin-ATM")
            CallATM "$@"
            ;;
        "Deepin-WeChat")
            CallWeChat "$@"
            ;;
        "Deepin-WXWork")
            CallWXWork "$@"
            ;;
        "Deepin-Dding")
            CallDingTalk "$@"
            ;;
        "Deepin-MTXX")
            CallMeiTuXiuXiu "$@"
            ;;
        "Deepin-FastReadPDF")
            CallFastReadPDF "$@"
            ;;
        "Deepin-Evernote")
            CallEvernote "$@"
            ;;
        "Deepin-TencentVideo")
            CallTencentVideo "$@"
            ;;
        "Deepin-Foxmail")
            CallFoxmail "$@"
            ;;
        "Deepin-THS")
            CallTHS "$@"
            ;;
        "Deepin-QQHlddz")
            CallQQGameV2 "$1" 363
            ;;
        "Deepin-QQBydr")
            CallQQGameV2 "$1" 1104632801
            ;;
        "Deepin-QQMnsj")
            CallQQGameV2 "$1" 1105856612
            ;;
        "Deepin-QQSszb")
            CallQQGameV2 "$1" 1105640244
            ;;
        "Deepin-CS6")
            CallPsCs6 "$@"
            ;;
        *)
            CallProcess "$@"
            ;;
    esac
}
ExtractApp()
{
    "$SHELL_DIR/deepin-wine-banner" unpack &
        mkdir -p "$1"
        7z x "$APPDIR/$APPTAR" -o"$1"
    if [ $? != 0 ];then
        "$SHELL_DIR/deepin-wine-banner" info "解压失败"
        rm -rf "$1"
        exit 1
    fi
        mv "$1/drive_c/users/@current_user@" "$1/drive_c/users/$USER"
        sed -i "s#@current_user@#$USER#" $1/*.reg
    FixLink
    "$SHELL_DIR/deepin-wine-banner" unpacked
}
DeployApp()
{
        ExtractApp "$WINEPREFIX"

    if UsePublicDir;then
        chgrp -R users "$WINEPREFIX"
        chmod -R 0775 "$WINEPREFIX"
    fi

        echo "$APPVER" > "$WINEPREFIX/PACKAGE_VERSION"
}
RemoveApp()
{
        rm -rf "$WINEPREFIX"
}
ResetApp()
{
        debug_log "Reset $PACKAGENAME....."
        read -p "*      Are you sure?(Y/N)" ANSWER
        if [ "$ANSWER" = "Y" -o "$ANSWER" = "y" -o -z "$ANSWER" ]; then
                EvacuateApp
                DeployApp
                CallApp
        fi
}
UpdateApp()
{
        if [ -d "${WINEPREFIX}.tmpdir" ]; then
                rm -rf "${WINEPREFIX}.tmpdir"
        fi

    if [ -f "/opt/apps/${DEB_PACKAGE_NAME}/files/pre_update.sh" ];then
        source "/opt/apps/${DEB_PACKAGE_NAME}/files/pre_update.sh"
        CallPreUpdate
        return
    fi

    case "$BOTTLENAME" in
        "Deepin-Intelligent" | "Deepin-QQ" | "Deepin-TIM" | "Deepin-WeChat" | "Deepin-WXWork" | "Deepin-Dding")
            "$SHELL_DIR/kill.sh" "$WINEPREFIX"
            rm -rf "$WINEPREFIX"
            DeployApp
            return
            ;;
    esac

        ExtractApp "${WINEPREFIX}.tmpdir"
        "$SHELL_DIR/updater" -s "${WINEPREFIX}.tmpdir" -c "${WINEPREFIX}" -v

    if UsePublicDir;then
        chgrp -R users "$WINEPREFIX"
        chmod -R 0775 "$WINEPREFIX"
    fi

        rm -rf "${WINEPREFIX}.tmpdir"
        echo "$APPVER" > "$WINEPREFIX/PACKAGE_VERSION"
}

RunWithLog()
{
    if [ -z "$WINEDEBUG" ];then
        export WINEDEBUG=trace-all,fixme-all,warn-all
        if [ ! -d "$WINEPREFIX/log" ];then
            mkdir "$WINEPREFIX/log"
        fi
        DEF_LOG_FILE="$WINEPREFIX/log/app.log"
        CallApp "$@" &> "$DEF_LOG_FILE"
    else
        CallApp "$@"
    fi
}

RunApp()
{
    "$SHELL_DIR/deepin-wine-banner"
    if [[ $? != 0 ]]; then
        debug_log "检测到 deepin-wine-banner 运行， exit 1"
        exit 1
    fi

    "$SHELL_DIR/deepin-wine-banner" start $BANNER_DATE &
    sleep 0.3

        if [ -d "$WINEPREFIX" ]; then
            if [ ! -f "$WINEPREFIX/PACKAGE_VERSION" ] || [ "$(cat "$WINEPREFIX/PACKAGE_VERSION")" != "$APPVER" ]; then
                        UpdateApp
            fi
        else
        DeployApp
        fi
    RunWithLog "$@"
}

CreateBottle()
{
    if [ -d "$WINEPREFIX" ]; then
            if [ ! -f "$WINEPREFIX/PACKAGE_VERSION" ] || [ "$(cat "$WINEPREFIX/PACKAGE_VERSION")" != "$APPVER" ]; then
            UpdateApp
            fi
    else
        DeployApp
    fi
}

ParseArgs()
{
    if [ $# -eq 4 ];then
            RunApp "$3"
    elif [ -f "$5" ];then
            if [ -n "$MIME_EXEC" ];then
                    RunApp "$MIME_EXEC" "$(UnixUriToDosPath "$5")" "${@:6}"
            else
                    RunApp "$3" "$(UnixUriToDosPath "$5")" "${@:6}"
            fi
    else
            RunApp "$3" "${@:5}"
    fi
}

#init_log_file

# Check if some visual feedback is possible
if command -v zenity >/dev/null 2>&1; then
        progressbar()
        {
                WINDOWID="" zenity --progress --title="$1" --text="$2" --pulsate --width=400 --auto-close --no-cancel ||
                WINDOWID="" zenity --progress --title="$1" --text="$2" --pulsate --width=400 --auto-close
        }

else
        progressbar()
        {
                cat -
        }
fi

if [ $# -lt 3 ]; then
    debug_log "参数个数小于3个"
    exit 0
fi

if UsePublicDir;then
    WINEPREFIX="$PUBLIC_DIR/$1"
fi

if [ -f "$APPDIR/files.md5sum" ];then
    APPVER="$(cat $APPDIR/files.md5sum)"
else
    APPVER="$2"
fi

# ARM 需要指定模拟器参数
if [ -f "/opt/deepinemu/tools/init_emu.sh" ];then
    source "/opt/deepinemu/tools/init_emu.sh"
    export EMU_CMD="$EMU_CMD"
    export EMU_ARGS="$EMU_ARGS"
fi

debug_log "Run $*"

if [ -z "$WINE_WMCLASS" ]; then
    export WINE_WMCLASS="$DEB_PACKAGE_NAME"
fi

#执行lnk文件通过判断第5个参数是否是“/Unix”来判断
if [ "$4" == "/Unix" ];then
    RunApp "$3" "$4" "$5"
    exit 0
fi

if [ $# -lt 4 ]; then
        RunApp "$3"
        exit 0
fi
case $4 in
        "-r" | "--reset")
                ResetApp
                ;;
        "-cb" | "--create")
                CreateBottle
                ;;
        "-e" | "--remove")
                RemoveApp
                ;;
        "-u" | "--uri")
        ParseArgs "$@"
                ;;
        "-f" | "--file")
        ParseArgs "$@"
                ;;
        "-h" | "--help")
                HelpApp
                ;;
        *)
                echo "Invalid option: $4"
                echo "Use -h|--help to get help"
                exit 1
                ;;
esac
exit 0
```

这段脚本比较长，对于程序员来说不难理解，主要做了如下事情：

1. Wine 容器 (Bottle) 管理:
 * 创建 (DeployApp): 如果用户的 ~/.deepinwine/ 目录下不存在应用的容器，脚本会自动创建一个，并将应用文件从 /opt/apps/.../files.7z 解压进去。
 * 更新 (UpdateApp): 在启动时检查应用版本。如果 DEB 包更新了，脚本会自动执行更新逻辑，替换旧文件。
 * 重置 (ResetApp): 提供 -r 或 --reset 选项，允许用户删除并重新创建容器，用于解决应用配置损坏等问题。
 * 移除 (RemoveApp): 提供 -e 或 --remove 选项，用于彻底删除应用的容器目录。
2. 环境初始化与修复:
 * 修复链接 (FixLink): 确保 Wine 容器内的 C: 盘、Z: 盘 (指向 Linux 根目录 /) 以及 Desktop, Documents 等文件夹能正确地链接到 Linux 用户的家目录，这是实现文件互通的关键。
 * 图形兼容性测试 (Test_GL_wine): 自动检测系统的 OpenGL 支持情况。如果检测到 3D 加速有问题，它会自动向 Wine 注册表写入配置，将应用的渲染模式降级为兼容性更好的 GDI 模式，以避免应用因显卡驱动问题而闪退或黑屏。
3. 执行分发 (Execution Dispatching):
 * 这是脚本的“大脑”。CallApp 函数通过一个巨大的 case 语句，根据传入的容器名 ($BOTTLENAME) 来判断应该执行哪个特定的启动函数。
 * 对于大多数“行为良好”的应用（如剪映），它会调用通用的 CallProcess 函数。
 * 对于 QQ、TIM、微信等需要特殊“照顾”的应用，它会调用专属的函数（如 CallQQ, CallWeChat），在启动前执行一些“黑魔法”，比如删除自动更新文件、修改注册表、设置特定环境变量等。
4. 进程管理与用户体验提升:
 * 在启动应用前，会调用 kill.sh 脚本来终止可能残留的旧进程，防止冲突。
 * 通过 deepin-wine-banner 工具，在解压和启动过程中向用户显示一个美观的“正在启动”的横幅，提升用户体验。


总结一下，一个完整的执行流程（以剪映为例）为：
1. 用户点击剪映图标，执行了用户应用程序目录下的 run.sh 脚本。
2. run.sh 脚本设置了 BOTTLENAME, APPVER, EXEC_PATH 等变量，并最终执行了命令：
  bash /opt/deepinwine/tools/run_v4.sh com.ulikecam.jianying.deepin 3.0.5.8542deepin2 "c:/JianyingPro/JianyingPro.exe"
3. run_v4.sh 开始执行。
4. 它解析参数，设置 WINEPREFIX 为 ~/.deepinwine/com.ulikecam.jianying.deepin。
5. 进入 RunApp 函数。
6. 检查 WINEPREFIX 目录。如果是首次运行，就调用 DeployApp，从 /opt/apps/com.ulikecam.jianying.deepin/files/files.7z 解压剪映的程序文件。如果目录存在但版本旧了，就调用 UpdateApp。
7. 调用 RunWithLog，后者再调用 CallApp。
8. CallApp 首先运行 FixLink 修复目录链接。然后进入 case 语句，发现 com.ulikecam.jianying.deepin 没有特殊规则，于是跳转到默认的 CallProcess 函数。
9. CallProcess 函数执行最后的准备工作：杀掉旧进程、测试 OpenGL。
10. CallProcess 执行最终命令 env WINEPREFIX=... deepin-wine8-stable "c:/JianyingPro/JianyingPro.exe"。
11. 剪映专业版成功启动。

现在，剪映专业版启动闪退，我们可以手动执行一下 /opt/apps/com.ulikecam.jianying.deepin/files/run.sh，看看发生了什么？

```
7-Zip 23.01 (x64) : Copyright (c) 1999-2023 Igor Pavlov : 2023-06-20
 64-bit locale=zh_CN.UTF-8 Threads:20 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 393599447 bytes (376 MiB)                      

Extracting archive: /opt/apps/com.ulikecam.jianying.deepin/files/files.7z
--
Path = /opt/apps/com.ulikecam.jianying.deepin/files/files.7z
Type = 7z
Physical Size = 393599447
Headers Size = 40747
Method = LZMA2:24 BCJ
Solid = +
Blocks = 2

ERROR: Dangerous symbolic link path was ignored : dosdevices/c: : ../drive_c
ERROR: Dangerous link path was ignored : dosdevices/com1 : /dev/ttyS4
ERROR: Dangerous link path was ignored : dosdevices/d:: : /dev/sr0
ERROR: Dangerous link path was ignored : dosdevices/z: : /
ERROR: Dangerous link path was ignored : drive_c/Program Files (x86)/Common Files/System/ADO/msado15.dll : /opt/deepin-wine8-stable/lib/wine/i386-windows/msado15.dll
ERROR: Dangerous link path was ignored : drive_c/Program Files (x86)/Common Files/System/OLE DB/msdaps.dll : /opt/deepin-wine8-stable/lib/wine/i386-windows/msdaps.dll
ERROR: Dangerous link path was ignored : drive_c/Program Files (x86)/Common Files/System/OLE DB/msdasql.dll : /opt/deepin-wine8-stable/lib/wine/i386-windows/msdasql.dll
ERROR: Dangerous link path was ignored : drive_c/Program Files (x86)/Common Files/System/OLE DB/oledb32.dll : /opt/deepin-wine8-stable/lib/wine/i386-windows/oledb32.dll
ERROR: Dangerous link path was ignored : drive_c/Program Files (x86)/Internet Explorer/iexplore.exe : /opt/deepin-wine8-stable/lib/wine/i386-windows/iexplore.exe
```
问了一下 AI，原来是 7z 在新版本中增加了安全限制，对于可能的危险链接会直接拒绝。解决方案：

```
sudo apt remove 7zip
sudo apt install p7zip
```

在厘清了 Wine 应用的运行逻辑后，再碰到无法启动的问题，就可以自己解决啦，你学会了吗？

