### 深度学习软件开发环境搭建

这个双十一，我下了一个狠单，配置了一台深度学习主机，详情请参考我之前的文章：

[这个双十一，我配了一台深度学习主机](https://mp.weixin.qq.com/s/roLDbKmP9w62v9ytCJpfBQ)

这几天，我花了一些时间，装系统，装软件，进行系统设置，搭建了一套令我比较满意的深度学习环境。下面就介绍一下我的深度学习软件配置。

在操作系统的选择上，我毫不犹豫的选择了Ubuntu 180.4 LTS（长期支持系统）。Linux虽然对普通用户不友好，但对开发人员却非常友好，各种开发软件，在Linux系统上均首先得到支持（微信小程序开发工具是个例外，这点很让人无语）。Linux的发行版本众多，而Ubuntu系统是我长期使用的发型版本，上手容易，使用方便，所以选择Ubuntu是自然而然的事情。版本上当然选择新版本，但我没有选择Ubuntu 19.04或Ubuntu 19.10，因为Ubuntu版本出的非常勤，每年都会出两个版本，但每隔两年会出一个LTS版本，支持期为4年，所以为了稳定起见，选择LTS版本是一个稳妥的选择，最近的一个LTS版本就是18.04。

选择Linux系统还有一个好处，就是专心开发，虽然这台主机很适合玩大型游戏，但没装Windows系统，玩不了。另外好多常见的软件，都没有Linux版本。总结一下，这台机器就是用来干活的。

这篇文章略过Ubuntu系统的安装，重点说一说各种深度学习软件的安装与配置。

#### 安装基本的开发工具

作为一名Linux开发人员，通常gcc、jdk、git是必不可少的，另外ssh登录，可以方便远程登录。下面是我安装的一些基本软件：

```
sudo apt-get update
sudo apt-get install build-essential git openssh-server vim openjdk-8-jdk bash-completion wget zlib1g-dev unzip
```

接下来设置github访问。 首先，使用您会记住的密码（或者为空）生成一个公共RSA密钥。
```
ssh-keygen -t rsa -b 4096
```
这将在 *~/.ssh/* 目录中生成一个公共密钥 *id_rsa.pub* 和一个标识符 *id_rsa*。现在，用文本编辑器打开 *id_rsa.pub* 文件，将其中的内容复制到剪贴板。

登录到您的GitHub帐户，然后在“设置”下单击SSH和GPG密钥并添加新的SSH密钥。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/deep_learning_software_setup_01.png)

将剪贴板中的内容贴到 *key* 这一栏。尝试一下，是否能够通过SSH协议克隆你的项目代码。

#### CUDA及cuDNN

这个主机主要用于进行深度学习，Nvidia的CUDA肯定首先需要安装的。虽然CUDA的最新版本是10.1，但由于TensorFlow GPU仅和CUDA 10.0兼容，所以不要安装最新的CUDA 10.1，请按照如下命令安装CUDA 10.0:

```bash
# 添加NVIDIA包仓库
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
$ sudo dpkg -i cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
$ sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
$ sudo apt-get update
wget http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/nvidia-machine-learning-repo-ubuntu1804_1.0.0-1_amd64.deb
$ sudo apt install ./nvidia-machine-learning-repo-ubuntu1804_1.0.0-1_amd64.deb
$ sudo apt-get update

# 安装 NVIDIA 驱动
$ sudo apt-get install --no-install-recommends nvidia-driver-418
```

重启。检查GPU是否正常工作，使用命令:

```bash
nvidia-smi
```

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/deep_learning_software_setup_02.png)

接下来安装CUDA开发与运行时库：

```
# 安装大约4GB
$ sudo apt-get install --no-install-recommends \
    cuda-10-0 \
    libcudnn7=7.6.2.24-1+cuda10.0  \
    libcudnn7-dev=7.6.2.24-1+cuda10.0


# 安装TensorRT. 要求以上的libcudnn7已经安装.
$ sudo apt-get install -y --no-install-recommends libnvinfer5=5.1.5-1+cuda10.0 \
    libnvinfer-dev=5.1.5-1+cuda10.0
```

要检查CUDA开发库是否正确安装，可以编译CUDA提供的示例程序。

$ cuda-install-samples-10.0.sh ~
$ cd ~/NVIDIA_CUDA-10.0_Samples
$ make

运行编译示例程序：

```
$ ./bin/x86_64/linux/release/deviceQuery
./bin/x86_64/linux/release/deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce RTX 2080 Ti"
  CUDA Driver Version / Runtime Version          10.1 / 10.0
  CUDA Capability Major/Minor version number:    7.5
  Total amount of global memory:                 10986 MBytes (11519983616 bytes)
  (68) Multiprocessors, ( 64) CUDA Cores/MP:     4352 CUDA Cores
  GPU Max Clock rate:                            1545 MHz (1.54 GHz)
  Memory Clock rate:                             7000 Mhz
  Memory Bus Width:                              352-bit
  L2 Cache Size:                                 5767168 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
  Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  1024
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 3 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Compute Preemption:            Yes
  Supports Cooperative Kernel Launch:            Yes
  Supports MultiDevice Co-op Kernel Launch:      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.1, CUDA Runtime Version = 10.0, NumDevs = 1
Result = PASS
```

输出结果中包含Result = PASS，表明CUDA是正常工作的。

#### Docker

关于Docker以及Docker中的硬件加速支持，请参考我前面一篇文章：

[启用Docker虚拟机GPU，加速深度学习](https://mp.weixin.qq.com/s/LIm357mAmtCZ8c_1AInHfg)

关于Docker虚拟机，这里补充一点知识。docker在安装完成之后，是需要root权限才能执行docker命令。通常情况下，使用sudo是一个危险的操作，应该尽量避免，Docker给出解决方案，将用户加入到名为**docker**的用户组，这个用户组在安装docker软件的过程中会创建。

```
$ sudo usermod -aG docker ${USER}
```

登出机器再登入，或者执行以下的命令，即可生效：

```
$ su - ${USER}
```

确定当前用户是否属于**docker**用户组:

```
$ id -nG
alex adm cdrom sudo dip plugdev lpadmin sambashare docker
```

这个时候直接运行 docker 命令就不会出错了，可以运行以下命令验证一下：

```
docker run hello-world
```

#### Anaconda

谈到机器学习，python肯定是首选语言。但是Python语言发展过程中，一直深受Python 2和Python 3两个版本分裂的困扰，虽然Python 3是大势所趋，但Python 2的生命力很顽强，现在依然有许多代码只能运行在python 2下。另外Tensorflow 1.X和Tensorflow 2.0也不兼容，让开发人员头疼不已。所以Python虚拟环境就非常有存在的必要。

前面讲到的docker虚拟机能解决这一问题，但是docker虚拟机相对而言比较“重”，更优雅的方案是Python虚拟环境，虽然只适用于python编程，但足够轻量。Python虚拟环境也有很多方案，这里我推荐Anaconda。

Anaconda就是可以便捷获取包且对包能够进行管理，同时对环境可以统一管理的发行版本。Anaconda包含了conda、Python在内的超过180个科学包及其依赖项。

Anaconda具有如下特点：

* 开源

* 安装过程简单

* 高性能使用Python和R语言

* 免费的社区支持

我之前针对Win 10操作系统写过一篇，有兴趣可以看看：

[Win10下配置机器学习python开发环境](https://mp.weixin.qq.com/s/j2e1PUmD69v0krxg-vq7vg)

Ubuntu下的安装和Win 10下差不多，首先去Anaconda官网下载安装包，下载地址为：

> https://www.anaconda.com/distribution/#download-section

下载得到一个自解压shell包，用shell执行即可：

```
$ sh Anaconda3-2019.10-Linux-x86_64.sh
```

按照安装说明：

* *回车* 通读许可条款。
* *yes* 同意许可条款。
* *回车* 接受默认安装位置（/home/{User}/anaconda3），或指定其他目录
* *yes* 将Anaconda3安装位置添加到 *~/.bashrc* 文件中

为了方便后续使用anaconda中的命令，登出再登入当前会话，或者简单的使 *~/.bashrc* 生效：

```
source ~/.bashrc
```

有了anaconda，接下来可以创建各种python虚拟环境，比如为python 2的代码建立一个名为py2的虚拟环境，并激活这个虚拟环境：

```
$ conda create --name py2 python=2.7
$ conda activate py2

(py2) alex@alex-MS-7C22:~/Downloads$ python
Python 2.7.17 |Anaconda, Inc.| (default, Oct 21 2019, 19:04:46) 
[GCC 7.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 

```
可以看出现在python解释器的版本是2.7.17。

还可以尝试为tensorflow 2.0 GPU创建一个虚拟环境，python版本可以选择3.6。

```
$ conda create --name tf2-gpu python=3.6
$ conda activate tf2-gpu
(tf2-gpu) alex@alex-MS-7C22:~/Downloads$ conda install tensorflow-gpu
```

创建python虚拟环境是如此轻量，如此方便。你可以查看现有所创建的虚拟环境列表：

```
(tf2-gpu) alex@alex-MS-7C22:~/Downloads$ conda env list
# conda environments:
#
base                     /data/ai/anaconda3
py2                      /data/ai/anaconda3/envs/py2
py38                     /data/ai/anaconda3/envs/py38
tf2-gpu               *  /data/ai/anaconda3/envs/tf2-gpu
```

前面有 \* 标记的表明当前激活的虚拟环境。

验证 *tf2-gpu* 虚拟环境是否启用了GPU加速：

```
import tensorflow as tf
from tensorflow.python.client import device_lib

print(tf.__version__)
tf.test.is_gpu_available()
tf.test.gpu_device_name()
device_lib.list_local_devices()
```

#### Jupyter notebook

在很多深度学习教程中，我们都可以看到Jupyter notebook，作为一种WEB交互环境，做演示、写samples非常方便。

我们可在 *tf2-gpu* 虚拟环境中安装Jupyter notebook，当然也可以为Jupyter notebook新建一个虚拟环境，看情形而定。

```
$ conda activate tf2-gpu # 'source activate jnb' deprecated
$ conda install -y -q -c numpy matplotlib jupyter nb_conda
```

运行 *jupyter notebook* 命令即可启动服务。

#### Visual Studio Code

作为一名开发人员，拥有一个趁手的代码编写工具无疑能使我们心情愉悦，提高开发效率。在此我推荐Visual Studio Code，由微软出品。虽然打上了Visual Studio标记，但是和Windows下的Visual Studio开发套件并没有什么关系，而是一款免费开源的现代化轻量级代码编辑器，支持几乎所有主流的开发语言的语法高亮、智能代码补全、自定义快捷键、括号匹配和颜色区分、代码片段、代码对比 Diff、GIT命令 等特性，支持插件扩展。VS Code 基于 Electron 开发。Electron 是一个基于 Chromium 的项目，可用于开发基于 Node.js 的本地应用程序。软件跨平台支持 Win、Mac 以及 Linux，运行流畅。

前往 https://code.visualstudio.com/download# 下载deb包，然后安装：

```
sudo dpkg -i ~/Downloads/code_1.40.1-1573664190_amd64.deb
```

VS Code的精华在于其丰富的插件，正式借助于插件，才能将一款文本编辑器打造成无所不能的开发环境，只要你愿意折腾，这点应该挺对程序员的胃口。借助Markdown插件，我现在写公众号、写文档都用VS Code。下面介绍几个Python开发插件：

* Python: 请认准微软出品，提供了代码分析，高亮，规范化等很多基本功能，装好这个就可以开始愉快的写python了。

* Anaconda Extension Pack: 依然是微软出品，配合前面推荐的Anaconda使用，非常方便切换Anaconda虚拟环境，还大大增强了代码提示功能，各种第三方库基本都能实现代码提示了，并且还会额外显示每个方法的帮助。

* Bracket Pair Colorizer: 代码颜色高亮一般只会帮你区分不同的变量，这款插件给不同的括号换上了不同的颜色，括号的多的时候非常实用。

* filesize: 一款在左下角显示文件大小的插件，还是挺实用的
* Trailing Spaces: 自动删除行尾的空格，代码提交到gerrit上，如果代码行存在空格符，就会出现刺眼的红色，这个插件可以解决这一问题。
* Project Manager: 顾名思义，就是管理VS Code的多个项目，方便在不同项目间切换。

当然VS Code还有很多主题、皮肤，只要你愿意折腾，可以打造一个非常炫酷的开发环境，这里就不过多介绍。

---

至此，我的深度学习开发环境介绍完毕，你觉得还有哪些必备软件呢，欢迎留言。

#### 参考

1. [How To Install and Use Docker on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)
2. [TensorFlow GPU support](https://www.tensorflow.org/install/gpu)

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)