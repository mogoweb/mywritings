# 鸿蒙系统研究第一步：从源码构建系统镜像

周末下载了 OpenHarmony OS 2.0 的源码，并 build 成功。虽然大部分的步骤都是来自官方文档，但还是碰到了一些问题，所以决定还是写下来，当作一个备忘录。

我平常使用的开发环境是 Ubuntu Linux 系统，但这次切换到了 Windows 系统，原因是鸿蒙的开发工具 DevEcoStudio 和烧写工具 HiTool 只有 Windows 版本和 Mac 版。好在 Windows 10 对 Linux 的支持非常好，其中 WSL (Windows Subsystem for Linux) 可以像 Windows 应用程序那样安装与运行，比使用虚拟机高效。WSL 已经进化到第二代，简称为 WSL2。关于 WSL2 的安装与配置，请参考相关文档。

我从 Microsoft Store 安装的 Linux 发行版本为 Ubuntu 18.04 LTS 版本。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/harmonyos_building_01.png)

需要注意的是，WSL2 Linux 的系统镜像文件默认放置在 C 盘，如果 C 盘空间预留不是很足够的话，建议移动到其它空间比较足的盘上。具体方法如下：

1. 首先找到 WSL2 Linux 的系统镜像文件位置，默认为 *C:\Users\<用户>\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState* ，将其中的 **<用户>** 替换为你的用户名。
2. 将 *C:\Users\<用户>\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState* 移动到其它盘，比如 *D:\VirtualMachines\WSL2\Ubuntu18.04\LocalState*
3. 建立符号链接（类似 Linux 下的软链接）。

```
mklink /j C:\Users\chenz\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState D:\VirtualMachines\WSL2\Ubuntu18.04\LocalState
```

做好上述准备后，就不会担心 C 盘被撑爆了。

言归正传，下面就说说在 Ubuntu 18.04 LTS 下如何下载和编译 OpenHarmony OS 2.0 的源码。

### 安装依赖工具

安装命令如下：

```
sudo apt-get update
sudo apt-get install binutils git-core git-lfs gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip m4 python2.7 python-minimal
```

### 获取标准系统源码

官方文档给了三种获取系统源码的方式，如果是研究鸿蒙系统，最好直接从软件仓库下载，这样有比较完善的提交信息。所以这里只介绍如何从软件仓库克隆系统源码。

1. 配置 git 用户信息

```
git config --global user.name "yourname"
git config --global user.email "your-email-address"
git config --global credential.helper store
```

2. 安装 repo 工具，可以执行如下命令。

```
$ curl https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 > repo
$ chmod a+x repo
$ sudo mv repo /usr/local/bin/repo
$ sudo apt install python3-pip
$ pip3 install -i https://repo.huaweicloud.com/repository/pypi/simple requests
```

3. 获取标准系统源码

```
$ mkdir OpenHarmony
$ cd OpenHarmony

$ repo init -u https://gitee.com/openharmony/manifest.git -b master --no-repo-verify

$ repo sync -c

$ repo forall -c 'git lfs pull'

```

注意：**repo sync** 命令后面的 **-c** 参数表示只获取当前分支的源码，也就是说并不是所有分支的代码。我尝试不加这个 **-c** 参数，可能是 gitee 的配置问题，超过 1G 的软件仓库，比如 linux kernel 就出现如下错误，网上搜索了很多方法也未能解决。

```
error: RPC failed; curl 56 GnuTLS recv error (-110): The TLS connection was non-properly terminated.
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

### 获取预编译工具

```
$ curl https://gitee.com/landwind/script-tools/raw/master/Shell/OpenHarmony/OpenHarmony_2.0_canary_prebuilts_download.sh >./prebuilts_download.sh
$ bash ./prebuilts_download.sh
```

### 配置NodeJS环境和获取Node\_modules依赖包

1. 下载Nodejs。

```
$ mkdir -p prebuilts/build-tools/common/nodejs
$ cd prebuilts/build-tools/common/nodejs
$ wget --no-check-certificate https://nodejs.org/download/release/v12.18.4/node-v12.18.4-linux-x64.tar.gz
$ tar -zxvf node-v12.18.4-linux-x64.tar.gz
$ cd -
```

2. 下载node\_modules包。

```
$ cd third_party/jsframework
$ export PATH=../../prebuilts/build-tools/common/nodejs/node-v12.18.4-linux-x64/bin:${PATH}
$ npm install
$ cd -
```

3. 把下载的node\_modules包放入OpenHarmony代码的prebuilts/build-tools/common/js-framework目录下。

```
$ mkdir -p prebuilts/build-tools/common/js-framework
$ cp -rp third_party/jsframework/node_modules prebuilts/build-tools/common/js-framework/
```

### 安装hc-gen工具

hc-gen用于进行驱动编译，具体安装步骤如下：

1.  下载 hc-gen 工具。

    ```
    $ wget https://repo.huaweicloud.com/harmonyos/compiler/hc-gen/0.65/linux/hc-gen-0.65-linux.tar
    ```

2.  解压 hc-gen 安装包到 ~/hc-gen 路径下。

    ```
    tar -xvf hc-gen-0.65-linux.tar -C ~/
    ```

3.  设置环境变量。

    ```
    vim ~/.bashrc
    ```

    将以下命令拷贝到.bashrc文件的最后一行，保存并退出。

    ```
    export PATH=~/hc-gen:$PATH
    ```

4.  生效环境变量。

    ```
    source ~/.bashrc
    ```

### 编译系统

执行脚本 build.sh 即可，后面必须加上 **--product-name Hi3516DV300** 参数，目前只支持这一种产品形态的构建。

```
$ ./build.sh --product-name Hi3516DV300 --ccache
```

构建成功以后，输出如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/harmonyos_building_02.png)

手头还没有 Hi3516DV300 的板子，所以无法烧写体验。

非常意外的是，OpenHarmony OS 2.0 没有提供模拟器的 build 选择，这对开发者相当不友好。后面我会研究一下 QEMU 模拟器，看能否在 QEMU 上把 OpenHarmony OS 2.0 运行起来，敬请关注。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)