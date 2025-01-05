# FUSE 文件系统和 libfuse 介绍

## 前言

这几天为了解决浏览器产品中的一个问题，研究了一下 FUSE(Filesystem in Userspace)。通常，应用程序开发只需要使用系统 API 进行文件读写操作，不需要了解文件系统的细节。在 Chromium 中为了实现跨平台，甚至对各操作系统的文件 API 进行了封装。在代码中使用封装 API，都不需要了解各操作系统所提供的文件系统 API。但由于我们的浏览器产品中使用了 FUSE 进行加密存储，所以有必要了解 FUSE 和 libfuse。

## FUSE 介绍

文件系统为应用程序提供了一个访问数据的通用接口。文件系统可以在用户态中实现，比如像鸿蒙这样的微内核。但大多数宏内核操作系统（如 Linux），文件系统是在内核态中实现，以保证性能。

然而，随着文件系统复杂性的增加，用户态文件系统的使用逐渐增多，特别是在快速开发和实验性研究领域中。采用用户态文件系统有如下优势：

* 开发效率更高，易于维护和移植。
* 用户态中的错误影响范围有限，降低了系统崩溃的风险。
* 支持多平台的丰富编程语言和库。

而用户态文件系统最大的缺点是性能开销较大，特别是在用户态和内核态之间的通信和上下文切换时。这一点在后面介绍 FUSE 的架构时会有更详细的说明。

FUSE（Filesystem in Userspace）是当前最广泛使用的用户态文件系统框架。据保守估计，至少有 100 种基于 FUSE 的文件系统可以在互联网上找到。

FUSE 得到广泛使用主要得益于其提供的简单 API。FUSE 项目由两个组件组成：由常规内核代码库维护的 fuse 内核模块和 libfuse 用户空间库。libfuse 提供了与 FUSE 内核模块通信的参考实现。

对于应用开发者而言，通常只需要使用 libfuse，无需了解 fuse 内核模块。不过为了更好的使用 libfuse 开发文件系统，最好理解 FUSE 的高层设计，了解其实现的一些细节。

所以，本文先介绍 FUSE 的高层架构，并解释一些重要的实现细节，接着重点介绍 libfuse 的 API，并介绍在 UOS/Deepin 上的编译与运行示例。

## FUSE 高层架构

FUSE 由内核部分和用户级守护进程组成。内核部分实现为一个 Linux 内核模块，当加载时，会向 Linux 的虚拟文件系统（VFS）注册一个 FUSE 文件系统驱动程序。该 FUSE 驱动程序充当由不同用户级守护进程实现的各种特定文件系统的代理。

除了注册一个新文件系统外，FUSE 的内核模块还注册了一个 `/dev/fuse` 块设备。该设备作为用户态 FUSE 守护进程与内核之间的接口。通常，守护进程从 `/dev/fuse` 读取 FUSE 请求，处理后将回复写回 `/dev/fuse`。

![图1 FUSE 架构](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202501/images/fuse_arch_01.png)

FUSE 的高层架构如图 1 所示。当用户空间程序对挂载的 FUSE 文件系统发起操作时，其调用流程如下：

1. 用户程序通过系统调用请求文件系统操作（如读取文件）。

2. Linux 虚拟文件系统（VFS）将请求转发到 FUSE 的内核模块。

3. FUSE 内核模块将请求打包成 `FUSE request` 数据结构，存入内核的 FUSE 队列，同时将调用进程挂起等待结果。

4. 用户空间守护进程（FUSE daemon）从 `/dev/fuse` 设备读取请求，处理后再将结果写回 `/dev/fuse`。

5. FUSE 内核模块从 `/dev/fuse` 接收到结果后，将结果返回给挂起的进程。

6. 用户程序接收到文件系统操作的结果。

从上面的流程可以看出，每次文件操作都需要从用户态程序进入内核态，然后通过 FUSE 内核模块将请求发送到用户态的 FUSE 守护进程，处理完成后再切换回内核态，最后返回给用户程序。这个调用过程不仅是增加了调用层次，而且系统调用和上下文切换增加了额外的开销。而且请求被放入 FUSE 队列后，必须等待用户态守护进程来处理。守护进程可能由于调度延迟或处理能力不足而导致请求滞后。

## libfuse 介绍

`libfuse` 是一个用户空间库，作为用户空间程序与 Linux 内核中的 FUSE 模块之间的接口。`libfuse` 为开发者提供了操作文件系统所需的功能，使其能够通过一个简洁的 API 实现复杂的文件系统行为。

开发者使用 `libfuse` 来实现自定义的文件系统逻辑，而内核通过 `fuse` 模块与用户空间的 `libfuse` 进行通信。

`libfuse` 提供了一套 API，供用户空间文件系统与内核交互。这些 API 主要集中在对文件系统操作的实现上，如打开、读写文件、删除文件等。以下是 `libfuse` 提供的一些常用 API：

### 1. 文件系统操作的回调函数
开发者需要实现一些回调函数，这些回调函数处理用户空间的文件系统操作。每个文件系统操作（如读取、写入、创建文件等）都需要对应的回调函数。

- `fuse_operations` 结构体：  
  该结构体定义了文件系统的各类操作（如 `read`, `write`, `create`, `unlink` 等）对应的回调函数。

  示例：
  ```c
  struct fuse_operations oper = {
      .getattr    = my_getattr,
      .read       = my_read,
      .write      = my_write,
      // 其他操作...
  };
  ```

  每个操作函数都有对应的签名，例如：
  - `getattr`：获取文件属性
    ```c
    int (*getattr)(const char *path, struct stat *stbuf);
    ```
  - `read`：从文件中读取数据
    ```c
    int (*read)(const char *path, char *buf, size_t size, off_t offset, struct fuse_file_info *fi);
    ```

### 2. 文件系统启动与关闭
- `fuse_main`：  
  启动 FUSE 文件系统并进入事件循环。开发者需要传入 `fuse_operations` 结构体和命令行参数来启动文件系统。
  ```c
  int fuse_main(int argc, char *argv[], const struct fuse_operations *op, size_t op_size);
  ```

- `fuse_unmount`：  
  卸载挂载的 FUSE 文件系统。
  ```c
  int fuse_unmount(const char *mountpoint, struct fuse_args *args);
  ```

### 3. 文件和目录操作
- `fuse_mkdir`：  
  创建目录。
  ```c
  int fuse_mkdir(const char *path, mode_t mode);
  ```

- `fuse_rmdir`：  
  删除目录。
  ```c
  int fuse_rmdir(const char *path);
  ```

- `fuse_open`：  
  打开文件，返回文件描述符。
  ```c
  int fuse_open(const char *path, struct fuse_file_info *fi);
  ```

- `fuse_read`：  
  从文件中读取数据。
  ```c
  int fuse_read(const char *path, char *buf, size_t size, off_t offset, struct fuse_file_info *fi);
  ```

- `fuse_write`：  
  向文件中写入数据。
  ```c
  int fuse_write(const char *path, const char *buf, size_t size, off_t offset, struct fuse_file_info *fi);
  ```

- `fuse_unlink`：  
  删除文件。
  ```c
  int fuse_unlink(const char *path);
  ```

### 4. 目录和文件信息
- `fuse_getattr`：  
  获取文件或目录的属性（如权限、大小、类型等）。
  ```c
  int fuse_getattr(const char *path, struct stat *stbuf);
  ```

- `fuse_readlink`：  
  获取符号链接的目标。
  ```c
  int fuse_readlink(const char *path, char *buf, size_t size);
  ```

### 5. 错误处理和返回值
- `fuse_reply_err`：  
  向 FUSE 内核模块返回错误代码。
  ```c
  void fuse_reply_err(struct fuse_req *req, int errcode);
  ```

- `fuse_reply_buf`：  
  返回一个数据缓冲区作为回应。
  ```c
  void fuse_reply_buf(struct fuse_req *req, const void *buf, size_t size);
  ```

## libfuse 源码编译与使用

如果对 libfuse 的版本没有什么要求，可以直接通过 apt install 命令进行安装：

```bash
sudo apt install libfuse-dev
```
deepin v23 上 libfuse 的版本是 2.9.9.1。

由于项目中使用的版本是 libfuse 3.10.5，因此需要手动编译安装。

### 1. 下载源码

从 [libfuse 官方网站](https://github.com/libfuse/libfuse) 下载源码，并切换到 tag/fuse-3.10.5。

```
$ git clone https://github.com/libfuse/libfuse
$ cd libfuse
$ git checkout -b tag/fuse-3.10.5 fuse-3.10.5
```

### 2. 安装 meson 构建系统

libfuse 采用了比较少见的构建系统 meson，这是一套基于 Python3 和 Ninja 的构建系统。首先安装 Python3 和 Ninja：

```bash
$ sudo apt-get install python3 python3-pip python3-setuptools \
                       python3-wheel ninja-build
```
接下来安装 meson:
```
$ pip3 install meson
```

### 3. 编译安装 libfuse

```bash
$ mkdir build
$ cd build
$ meson ..
$ ninja
$ sudo ninja install
```

可以跑一下测试程序：

```
$ sudo chown root:root util/fusermount3
$ sudo chmod 4755 util/fusermount3
$ python3 -m pytest test/
```

## 一个简单示例

为了更好的理解如何使用fuse，我们这里实现一个具体的实例。大多数程序示例都是以 Hello World 开头的，这里我们也实现一个 Hello World 文件系统。

为了简单起见，这里并不实现一个真正的文件系统，也不会访问磁盘，而是在该文件系统的根目录中显示一个固定的文件，也就是 Hello-world 文件。

```
#define FUSE_USE_VERSION 29
#include <stdio.h>
#include <fuse.h>

/* 这里实现了一个遍历目录的功能，当用户在目录执行ls时，会回调到该函数，我们这里只是返回一个固定的文件Hello-world。 */
static int test_readdir(const char* path, void* buf, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info* fi)
{
    printf( "tfs_readdir	 path : %s ", path);
 
    return filler(buf, "Hello-world", NULL, 0);
}

/* 显示文件属性 */
static int test_getattr(const char* path, struct stat *stbuf)
{
    printf("tfs_getattr	 path : %s ", path);
    if (strcmp(path, "/") == 0)
        stbuf->st_mode = 0755 | S_IFDIR;
    else
        stbuf->st_mode = 0644 | S_IFREG;
    return 0;
}

/*这里是回调函数集合，这里实现的很简单*/
static struct fuse_operations tfs_ops = {
    .readdir = test_readdir,
    .getattr = test_getattr,
};

int main(int argc, char *argv[])
{
    int ret = 0;
    ret = fuse_main(argc, argv, &tfs_ops, NULL);
    return ret;
}
```

假设在 /tmp 目录下面有一个 file_on_fuse_fs 目录，如果没有可以手动创建一个。然后执行如下命令可以在该目录挂载新的文件系统。

```
$ ./fuse_user /tmp/file_on_fuse_fs
```

此时实际上已经有一个新的文件系统挂载在 /tmp/file_on_fuse_fs 目录下面，可以通过 mount 命令查看一下。

```
$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=16334592k,nr_inodes=4083648,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,noexec,relatime,size=3279000k,mode=755)
/dev/sdc3 on / type ext4 (rw,relatime)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
efivarfs on /sys/firmware/efi/efivars type efivarfs (rw,nosuid,nodev,noexec,relatime)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=32,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=3173)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
debugfs on /sys/kernel/debug type debugfs (rw,nosuid,nodev,noexec,relatime)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,nosuid,nodev,relatime,pagesize=2M)
tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime)
fusectl on /sys/fs/fuse/connections type fusectl (rw,nosuid,nodev,noexec,relatime)
configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)
/dev/sda1 on /work type ext4 (rw,relatime)
/dev/sdc1 on /boot/efi type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro)
/dev/sdb on /data type ext4 (rw,relatime)
binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=3278996k,nr_inodes=819749,mode=700,uid=1000,gid=1000)
gvfsd-fuse on /run/user/1000/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
portal on /run/user/1000/doc type fuse.portal (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
/dev/nvme1n1p2 on /media/alex/eaf5b6c4-7ee8-4afb-8294-3d73338106f1 type ext4 (rw,nosuid,nodev,relatime,errors=remount-ro,uhelper=udisks2)
/dev/nvme0n1p2 on /media/alex/5C6EC55E6EC53216 type fuseblk (rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other,blksize=4096,uhelper=udisks2)
/work/browser/fuse/hello_fuse on /tmp/file_on_fuse_fs type fuse.hello_fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
```

最后一行就是我们挂载的文件系统。可以通过 ls 命令查看一下这个目录：

```
$ ls -alh /tmp/file_on_fuse_fs
总计 0
-rw-r--r-- 0 root root 0 1970年 1月 1日 Hello-world
```

注意，这里只是一个非常简单的示例代码，并不存在 Hello-world 文件，而是通过虚拟文件系统，让 ls 命令返回这样一个结果。

## 小结

通过 FUSE，开发者可以轻松地在用户空间实现文件系统逻辑，而无需深入内核开发。这种灵活性使得 FUSE 在透明加密、虚拟文件系统等领域得到了广泛应用。而 libfuse 的出现，更是降低了开发 FUSE 文件系统的门槛。

由于篇幅原因，这里并没有给出很复杂的例子，后续会继续介绍 FUSE 的更多用法，敬请关注。