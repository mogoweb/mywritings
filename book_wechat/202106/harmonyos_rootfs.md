# 鸿蒙系统研究之四：根文件系统

在上一篇文章[鸿蒙系统研究之三：迈出平台移植第一步]()，我们将内核加载并启动，但缺少根文件系统。这篇文章我们来探讨一下根文件系统的制作。

熟悉 Android 系统开发的朋友可能知道，一个 Android 系统镜像通常包括 system.img、userdata.img、recovery.img 等几个系统镜像，这些镜像一般烧录到手机或板子的 Flash 存储上。Vexpress A9 模拟器也可以模拟出 Flash Memory，但容量最多只有 128 M。鸿蒙标准系统面向手机、平板等富资源设备，系统镜像通常有好几百兆，没有办法写入到 Vexpress A9 模拟器的 Flash Memory。但是 Vexpress A9 模拟器支持加载 SD 存储，所以咱们先采取一个变通的方法，将根文件系统和系统文件都放入到 SD 存储中，然后挂载到模拟器中。

鸿蒙系统构建完毕之后，包括 system.img、userdata.img、vendor.img 和 updater.img，但这不是能够挂载到 SD 存储的格式。

研究鸿蒙系统的构建输出，系统文件都输出在 out/ohos-arm-release/packages/phone/images/system 目录下，根文件系统则位于 out/ohos-arm-release/packages/phone/images/root，先把这两个目录的文件整合到一起，生成 ext3 格式的镜像，为此，我们写一个名为 create-fs.sh 的脚本：

```
#!/bin/bash
# 0.设置目录变量
OHOS_ROOTFS=../../../out/ohos-arm-release/packages/phone/images

# 1.先进行清理操作
sudo rm -rf rootfs
sudo rm -rf tmpfs
sudo rm -f rootfs.ext3

sudo mkdir rootfs
# 2.创建Linux中的必要文件夹
sudo mkdir -p rootfs/bin   # /bin包含普通用户和超级用户都能使用的命令
sudo mkdir -p rootfs/sbin  # /sbin包含系统运行的关键可执行文件以及一些管理程序
sudo mkdir -p rootfs/home  # /home普通用户的工作目录，没有普通用户都会在这里建立一个文件夹
sudo mkdir -p rootfs/etc   # /etc存放系统配置文件以及应用程序的配置文件
sudo mkdir -p rootfs/lib   # /lib存放所有应用程序的共享文件以及内核模块
sudo mkdir -p rootfs/proc/ # /proc目录是内核在内存中映射的实时文件系统，存放内核向用户应用程序提供的信息文件
sudo mkdir -p rootfs/sys/  # /sys是文件系统挂载的地方
sudo mkdir -p rootfs/tmp/  # /tmp存放系统或应用程序产生的临时文件
sudo mkdir -p rootfs/root/ # /root是超级用户的用户目录
sudo mkdir -p rootfs/var/  # /var存放假脱机数据以及系统日志
sudo mkdir -p rootfs/mnt/  # /mnt用于加载磁盘分区和硬件设备挂载点
sudo mkdir -p rootfs/usr   # /usr包含所有用户的二进制文件和库文件等
sudo mkdir -p rootfs/dev/  # /dev用于存放设备文件
sudo mkdir -p rootfs/system

# 3.将open harmony 根文件系统文件放到rootfs下
sudo cp -arf ${OHOS_ROOTFS}/root/*  rootfs/
sudo cp -arf ${OHOS_ROOTFS}/system/*  rootfs/system/

# 4.创建设备文件
sudo mknod rootfs/dev/tty1 c 4 1
sudo mknod rootfs/dev/tty2 c 4 2
sudo mknod rootfs/dev/tty3 c 4 3
sudo mknod rootfs/dev/tty4 c 4 4
sudo mknod rootfs/dev/console c 5 1
sudo mknod rootfs/dev/null c 1 3

# 5.生成一个空的文件作为文件系统
sudo dd if=/dev/zero of=rootfs.ext3 bs=1M count=1024
sudo mkfs -t ext3 rootfs.ext3

# 6.将文件系统挂载到tmpfs目录下
sudo mkdir -p tmpfs
sudo mount -t ext3 rootfs.ext3 tmpfs/ -o loop

# 7.将之前创建的文件系统相关的文件放到通过tmpfs放到rootfs.ext3文件系统中去
sudo cp -r rootfs/*  tmpfs/
sudo umount tmpfs
```

脚本文件中使用 sudo 的目的是保证 rootfs.ext3 里面的文件的所有者为 root。在 device/qemu/vexpress-a9 下执行该脚本，会在当前目录下生成 rootfs.ext3，先修改文件属性，让模拟器能够加载：

```
$ sudo chmod 777 rootfs.ext3
```

退回到鸿蒙系统源码的根目录下，执行命令：

```
$ qemu-system-arm -M vexpress-a9 -m 512M -dtb ./out/KERNEL_OBJ/kernel/src_tmp/linux-4.19/arch/arm/boot/dts/vexpress-v2p-ca9.dtb -kernel ./out/KERNEL_OBJ/kernel/src_tmp/linux-4.19/arch/arm/boot/zImage -append "root=/dev/mmcblk0 rw console=ttyAMA0 init=/init" -sd ./device/qemu/vexpress-a9/rootfs.ext3 -nographic
```

可以发现有如下输出：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/harmonyos_rootfs_01.png)

可以看到，内核加载了，根文件系统也加载了，也执行了超级用户进程 init，遗憾的是，执行中还存在错误。不用担心，我们一步一步来解决问题。

可以看出，这篇文章介绍的方法的不足，系统镜像的文件和根文件系统混在一起。一般而言，根文件系统是加载到内存中，这样做出来的根文件系统太大，比较占内存。一般根文件系统和系统镜像分开加载，在下一篇文章中，我们采用 uboot 来加载内核、根文件系统以及系统镜像。

敬请关注！