# 鸿蒙系统研究之六：U-Boot引导

这是我的鸿蒙系统研究系列文章的第五篇，有兴趣还可以看看前面的文章：

* [鸿蒙系统研究第一步：从源码构建系统镜像](https://mp.weixin.qq.com/s/LofgJs_-oFP9cUjovXaiBQ)
* [鸿蒙系统研究之二：内核编译](https://mp.weixin.qq.com/s/DalVCNVyBqJWRK6vTIYRdQ)
* [鸿蒙系统研究之三：迈出平台移植第一步](https://mp.weixin.qq.com/s/7vUZqDI6f79kComjeG0Q_A)
* [鸿蒙系统研究之四：根文件系统](https://mp.weixin.qq.com/s/NBVs6Wa8jKE5x8GcSSusvQ)
* [鸿蒙系统研究之五：替换 AOSP 预编译库，关闭 SELinux](https://mp.weixin.qq.com/s/h2Rdnvt4NogN827VeGKVHQ)

另外，还有关于鸿蒙系统的看法：

* [我看鸿蒙系统](https://mp.weixin.qq.com/s/SeMGA1EYfcENcaeBfgG2hA)
* [这就是鸿蒙系统？](https://mp.weixin.qq.com/s/d1PzXcBEUdvzoKvRuQeMcQ)
* [吐槽一下开源鸿蒙系统](https://mp.weixin.qq.com/s/Hhq4k6mclEMTUKPjLzUZOQ)

U-Boot 的全称是 **Universal Boot Loader**，其作用就是引导系统。对于我们熟悉的 PC，上电后，通过 BIOS 引导操作系统 （Windows、Linux等）。对于嵌入式系统一般将这个引导程序称作 BootLoader，U-Boot 就是目前使用得最广泛的 BootLoader。

在前面的文章中，QEMU 直接引导鸿蒙系统的 Linux 内核，这种方式缺少灵活性，关键是在实际产品中，嵌入式系统是不会直接上电引导 Linux 内核的（整个软件系统一般位于 Flash，需要能够读写 Flash）。所以本文探讨使用 U-Boot 来引导内核。

开始之前，也做一个准备工作，安装 mkImage 工具:

```
$ sudo apt install u-boot-tools
```

U-Boot 既然是用来引导内核，那肯定与具体硬件相关，所以需要从源码编译，并且还需要针对具体产品做一些修改。

#### 下载 U-Boot 源码，并编译

U-Boot 源码选择哪个版本关系并不大，这里选择 OpenHarmony 标准系统参考实现相同的版本：u-boot-2020.01。直接去官网下载，不要直接使用 device/hisilicon/third_party/uboot/u-boot-2020.01 这个代码，因为这个版本有华为针对 Hi3516DV300 等产品进行的修改。

```
$ wget https://codeload.github.com/u-boot/u-boot/zip/refs/tags/v2020.01
```

解压源码，然后进入 U-Boot 源码目录，编译 u-boot

```
$ export ARCH=arm
$ export CROSS_COMPILE=arm-linux-gnueabi-
$ export LOADADDR=0x80008000
$ make vexpress_ca9x4_defconfig
$ make all
```

编译后得到一个 u-boot 文件，这个在后面 QEMU 加载时用到。 

#### 制作 SD 卡镜像

在[鸿蒙系统研究之四：根文件系统](https://mp.weixin.qq.com/s/NBVs6Wa8jKE5x8GcSSusvQ)这篇文章中，我曾写到，要将根文件系统和system镜像分开，但仔细研究鸿蒙的文件系统后发现不行，因为根文件系统中的很多文件都是指向system的符号链接，特别是 init 这个超级进程。kernel 加载之后，首先要执行 init 进程，但此时还没有机会挂载 system 镜像，导致 init 是一个空链接。

此外，鸿蒙系统启用了SeLinux，这个也是超级折磨人的一个安全特性，稍不注意就会有程序无法执行的问题。因此这里解包 system.img ，得到根文件系统，另外为了简单起见，将 vendor 镜像里面的内容也复制过来。

Vexpress A9 最多只有 128 M 的 flash，没法容纳整个鸿蒙标准系统。好在 Vexpress A9 支持 SD 卡挂载，所以这里制作一个 SD 卡镜像，通过 QEMU 模拟器挂载。

下面的内容主要查阅网上的资料，为此写了一个脚本，将整个过程自动化完成。

```bash
#!/bin/bash
# 0.设置目录变量
OHOS_SRC_ROOT=$(cd "../../.." && pwd)

# 1. 解包系统镜像，得到鸿蒙系统文件
cd ${OHOS_SRC_ROOT}/out/ohos-arm-release/packages/phone/images
# Android sparse image 和 ext4 文件系统之间转换
simg2img system.img system.ext4
# 挂载 system 镜像
mkdir system-img
sudo mount system.ext4 system-img
simg2img vendor.img vendor.ext4
# 挂载 vendor 镜像
mkdir vendor-img
sudo mount vendor.ext4 vendor-img

# 2. 制作 SD 卡镜像(1GB)
dd if=/dev/zero of=uboot.disk bs=1M count=1024
# 创建 4 个分区
sgdisk -n 0:0:+16M -c 0:kernel uboot.disk
sgdisk -n 0:0:+512M -c 0:rootfs uboot.disk
sgdisk -n 0:0:+64M -c 0:vendor uboot.disk
sgdisk -n 0:0:0 -c 0:userdata uboot.disk

# 3. 寻找一个空闲的loop设备，挂载 SD 卡镜像
loop=$(sudo losetup -f)
echo "found idle loop device: ${loop}"
sleep 1
sudo losetup ${loop} uboot.disk
sleep 3
sudo partprobe ${loop}
sleep 3
# 格式化分区
sudo mkfs.ext4 ${loop}p1
sudo mkfs.ext4 ${loop}p2
sudo mkfs.ext4 ${loop}p3
sudo mkfs.ext4 ${loop}p4

# 4. 将 kernel 和系统文件复制到 SD 卡镜像
mkdir p1
mkdir p2
sudo mount -t ext4 ${loop}p1 p1/
sudo mount -t ext4 ${loop}p2 p2/
sudo cp ${OHOS_SRC_ROOT}/out/KERNEL_OBJ/kernel/src_tmp/linux-4.19/arch/arm/boot/zImage p1/
sudo cp ${OHOS_SRC_ROOT}/out/KERNEL_OBJ/kernel/src_tmp/linux-4.19/arch/arm/boot/dts/vexpress-v2*.dtb p1/
sudo cp -raf system-img/* p2/
sudo cp -raf vendor-img/* p2/vendor/

# 卸载镜像
sudo umount system-img
sudo umount vendor-img
sudo umount p1 p2
sudo losetup -d ${loop}

rm -rf system-img
rm -rf vendor-img
rm -rf p1
rm -rf p2
```

执行该脚本，将在 ${OHOS_SRC_ROOT}/out/ohos-arm-release/packages/phone/images 目录下生成一个 SD 卡镜像文件： uboot.disk，包含四个分区，其中 vendor 分区和 userdata 分区暂时没有用上，这个将在以后挂载。

#### 启动uboot

将前面生成的 u-boot 文件和 uboot.disk 文件放到同一个目录下，然后运行：

```
$ qemu-system-arm -M vexpress-a9 -m 512M -nographic -kernel ./u-boot -sd ./uboot.disk
U-Boot 2020.01 (Jul 05 2021 - 16:43:03 +0800)

DRAM:  512 MiB
WARNING: Caches not enabled
Flash: 128 MiB
MMC:   MMC: 0
*** Warning - bad CRC, using default environment

In:    serial
Out:   serial
Err:   serial
Net:   smc911x-0
Hit any key to stop autoboot:  0
```

倒计时结束前敲任意键，进入手工运行状态。

默认SD卡就是出于可用状态，也可以用下面的命令查看：

```
=> mmc dev 0
switch to partitions #0, OK
mmc0 is current device
=> mmc info
Device: MMC
Manufacturer ID: aa
OEM: 5859
Name: QEMU! 
Bus Speed: 6250000
Mode: SD Legacy
Rd Block Len: 512
SD version 1.0
High Capacity: Yes
Capacity: 4 GiB
Bus Width: 1-bit
Erase Group Size: 512 Bytes
```

查看分区内容：

```
=> part list mmc 0

Partition Map for MMC device 0  --   Partition Type: EFI

Part	Start LBA	End LBA		Name
	Attributes
	Type GUID
	Partition GUID
  1	0x00000800	0x000087ff	"kernel"
	attrs:	0x0000000000000000
	type:	0fc63daf-8483-4772-8e79-3d69d8477de4
	guid:	016c761f-2f11-42ab-8b29-0b696041d2e4
  2	0x00008800	0x001087ff	"rootfs"
	attrs:	0x0000000000000000
	type:	0fc63daf-8483-4772-8e79-3d69d8477de4
	guid:	9f3de605-f8b5-4f09-8d76-1c126ed12da1
  3	0x00108800	0x001287ff	"vendor"
	attrs:	0x0000000000000000
	type:	0fc63daf-8483-4772-8e79-3d69d8477de4
	guid:	4584a2dd-d969-4a49-a0c7-b8235795dd66
  4	0x00128800	0x001fffde	"userdata"
	attrs:	0x0000000000000000
	type:	0fc63daf-8483-4772-8e79-3d69d8477de4
	guid:	8e11ebef-5cfc-4edd-8cc8-a7d9404e9a03

=> ls mmc 0:1
<DIR>       1024 .
<DIR>       1024 ..
<DIR>      12288 lost+found
         4492968 zImage
           18595 vexpress-v2p-ca15_a7.dtb
           13218 vexpress-v2p-ca15-tc1.dtb
           12796 vexpress-v2p-ca5s.dtb
           14430 vexpress-v2p-ca9.dtb
=> ls mmc 0:2
<DIR>       4096 .
<DIR>       4096 ..
<DIR>      16384 lost+found
<DIR>       4096 acct
<DIR>       4096 apex
<SYM>         11 bin
<SYM>         50 bugreports
<DIR>       4096 cache
<SYM>         19 charger
<DIR>       4096 config
<SYM>         17 d
<DIR>       4096 data
<DIR>       4096 debug_ramdisk
<SYM>         23 default.prop
<DIR>       4096 dev
<SYM>         11 etc
<SYM>         16 init
           30389 init.cfg
             987 init.environ.rc
           33152 init.rc
            7690 init.usb.configfs.rc
            5623 init.usb.rc
<DIR>       4096 mnt
<DIR>       4096 odm
<DIR>       4096 oem
<DIR>       4096 proc
<SYM>         15 product
<SYM>         24 product_services
<DIR>       4096 sbin
<SYM>         21 sdcard
<DIR>       4096 storage
<DIR>       4096 sys
<DIR>       4096 system
            2608 ueventd.rc
<DIR>       4096 vendor

```

加载kernel、设备树

```
=> load mmc 0:1 0x60008000 zImage
4426496 bytes read in 463 ms (9.1 MiB/s)
=> load mmc 0:1 0x61000000 vexpress-v2p-ca9.dtb
14430 bytes read in 65 ms (216.8 KiB/s)
```

设置bootargs

```
=> setenv bootargs 'root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait earlycon console=tty0 console=ttyAMA0 init=/init ignore_loglevel androidboot.selinux=permissive'
```
 
引导内核

```
=> bootz 0x60008000 - 0x61000000
```

输出如下（当然还存在一些问题，这个后面慢慢来解决）：

```
Starting kernel ...

Booting Linux on physical CPU 0x0
Linux version 4.19.155+ (alex@alex-MS-7C22) (Android (dev based on r353983c) clang version 9.0.3 (https://android.googlesource.com/toolchain/clang 745b335211bb9eadfa6aa6301f84715cee4b37c5) (https://android.googlesource.com/toolchain/llvm 60cf23e54e46c807513f7a36d0a7b777920b5881) (based on LLVM 9.0.3svn)) #1 SMP Mon Jul 12 09:17:21 CST 2021
CPU: ARMv7 Processor [410fc090] revision 0 (ARMv7), cr=10c5387d
CPU: PIPT / VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
OF: fdt: Machine model: V2P-CA9
Malformed early option 'earlycon'
debug: ignoring loglevel setting.
Memory policy: Data cache writeback
cma: dma_contiguous_reserve(limit ffffffff)
cma: dma_contiguous_reserve: reserving 64 MiB for global area
cma: cma_declare_contiguous(size 0x04000000, base 0x00000000, limit 0xffffffff alignment 0x00000000)
cma: Reserved 64 MiB at 0x7bc00000
On node 0 totalpages: 131072
  Normal zone: 1024 pages used for memmap
  Normal zone: 0 pages reserved
  Normal zone: 131072 pages, LIFO batch:31
CPU: All CPU(s) started in SVC mode.
random: get_random_bytes called from start_kernel+0x88/0x39c with crng_init=0
percpu: Embedded 14 pages/cpu s28108 r8192 d21044 u57344
pcpu-alloc: s28108 r8192 d21044 u57344 alloc=14*4096
pcpu-alloc: [0] 0 [0] 1 [0] 2 [0] 3 
Built 1 zonelists, mobility grouping on.  Total pages: 130048
Kernel command line: root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait earlycon console=tty0 console=ttyAMA0 init=/init ignore_loglevel androidboot.selinux=permissive
log_buf_len individual max cpu contribution: 4096 bytes
log_buf_len total cpu_extra contributions: 12288 bytes
log_buf_len min size: 16384 bytes
log_buf_len: 32768 bytes
early log buf free: 14280(87%)
Dentry cache hash table entries: 65536 (order: 6, 262144 bytes)
Inode-cache hash table entries: 32768 (order: 5, 131072 bytes)
Memory: 442076K/524288K available (8192K kernel code, 252K rwdata, 1656K rodata, 1024K init, 195K bss, 16676K reserved, 65536K cma-reserved, 0K highmem)
Virtual kernel memory layout:
    vector  : 0xffff0000 - 0xffff1000   (   4 kB)
    fixmap  : 0xffc00000 - 0xfff00000   (3072 kB)
    vmalloc : 0xa0800000 - 0xff800000   (1520 MB)
    lowmem  : 0x80000000 - 0xa0000000   ( 512 MB)
    pkmap   : 0x7fe00000 - 0x80000000   (   2 MB)
    modules : 0x7f000000 - 0x7fe00000   (  14 MB)
      .text : 0x(ptrval) - 0x(ptrval)   (9184 kB)
      .init : 0x(ptrval) - 0x(ptrval)   (1024 kB)
      .data : 0x(ptrval) - 0x(ptrval)   ( 253 kB)
       .bss : 0x(ptrval) - 0x(ptrval)   ( 196 kB)
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
rcu: Hierarchical RCU implementation.
rcu: 	RCU restricting CPUs from NR_CPUS=8 to nr_cpu_ids=4.
rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=4
NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
GIC CPU mask not found - kernel will fail to boot.
GIC CPU mask not found - kernel will fail to boot.
sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 89478484971ns
clocksource: arm,sp804: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 1911260446275 ns
Failed to initialize '/smb@4000000/motherboard/iofpga@7,00000000/timer@12000': -22
smp_twd: clock not found -2
Console: colour dummy device 80x30
console [tty0] enabled
Calibrating local timer... 96.04MHz.
Calibrating delay loop... 1208.32 BogoMIPS (lpj=6041600)
pid_max: default: 32768 minimum: 301
Security Framework initialized
SELinux:  Initializing.
AppArmor: AppArmor disabled by boot time parameter
Mount-cache hash table entries: 1024 (order: 0, 4096 bytes)
Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes)
CPU: Testing write buffer coherency: ok
CPU0: Spectre v2: using BPIALL workaround
CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
Setting up static identity map for 0x60100000 - 0x60100060
rcu: Hierarchical SRCU implementation.
smp: Bringing up secondary CPUs ...
smp: Brought up 1 node, 1 CPU
SMP: Total of 1 processors activated (1208.32 BogoMIPS).
CPU: All CPU(s) started in SVC mode.
devtmpfs: initialized
VFP support v0.3: implementor 41 architecture 3 part 30 variant 9 rev 0
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
futex hash table entries: 1024 (order: 4, 65536 bytes)
pinctrl core: initialized pinctrl subsystem
NET: Registered protocol family 16
cma: cma_alloc(cma (ptrval), count 64, align 6)
cma: cma_alloc(): returned (ptrval)
DMA: preallocated 256 KiB pool for atomic coherent allocations
audit: initializing netlink subsys (disabled)
audit: type=2000 audit(0.200:1): state=initialized audit_enabled=0 res=1
cpuidle: using governor ladder
Serial: AMBA PL011 UART driver
10009000.uart: ttyAMA0 at MMIO 0x10009000 (irq = 25, base_baud = 0) is a PL011 rev1
console [ttyAMA0] enabled
1000a000.uart: ttyAMA1 at MMIO 0x1000a000 (irq = 26, base_baud = 0) is a PL011 rev1
1000b000.uart: ttyAMA2 at MMIO 0x1000b000 (irq = 27, base_baud = 0) is a PL011 rev1
1000c000.uart: ttyAMA3 at MMIO 0x1000c000 (irq = 28, base_baud = 0) is a PL011 rev1
OF: amba_device_add() failed (-19) for /smb@4000000/motherboard/iofpga@7,00000000/wdt@f000
OF: amba_device_add() failed (-19) for /memory-controller@100e0000
OF: amba_device_add() failed (-19) for /memory-controller@100e1000
OF: amba_device_add() failed (-19) for /watchdog@100e5000
SCSI subsystem initialized
libata version 3.00 loaded.
usbcore: registered new interface driver usbfs
usbcore: registered new interface driver hub
usbcore: registered new device driver usb
videodev: Linux video capture interface: v2.00
Advanced Linux Sound Architecture Driver Initialized.
clocksource: Switched to clocksource arm,sp804
NET: Registered protocol family 2
tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 6144 bytes)
TCP established hash table entries: 4096 (order: 2, 16384 bytes)
TCP bind hash table entries: 4096 (order: 3, 32768 bytes)
TCP: Hash tables configured (established 4096 bind 4096)
UDP hash table entries: 256 (order: 1, 8192 bytes)
UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
NET: Registered protocol family 1
RPC: Registered named UNIX socket transport module.
RPC: Registered udp transport module.
RPC: Registered tcp transport module.
RPC: Registered tcp NFSv4.1 backchannel transport module.
workingset: timestamp_bits=30 max_order=17 bucket_order=0
squashfs: version 4.0 (2009/01/31) Phillip Lougher
NFS: Registering the id_resolver key type
Key type id_resolver registered
Key type id_legacy registered
jffs2: version 2.2. (NAND) © 2001-2006 Red Hat, Inc.
io scheduler noop registered (default)
io scheduler mq-deadline registered
io scheduler kyber registered
clcd-pl11x 1001f000.clcd: PL111 designer 41 rev2 at 0x1001f000
clcd-pl11x 1001f000.clcd: clcd@1f000 hardware, 640x480@59 display
Console: switching to colour frame buffer device 80x30
clcd-pl11x 10020000.clcd: PL111 designer 41 rev2 at 0x10020000
cma: cma_alloc(cma (ptrval), count 384, align 8)
cma: cma_alloc(): returned (ptrval)
clcd-pl11x 10020000.clcd: clcd@10020000 hardware, 1024x768@59 display
brd: module loaded
40000000.flash: Found 2 x16 devices at 0x0 in 32-bit bank. Manufacturer ID 0x000000 Chip ID 0x000000
Intel/Sharp Extended Query Table at 0x0031
Using buffer write method
erase region 0: offset=0x0,size=0x40000,blocks=256
40000000.flash: Found 2 x16 devices at 0x0 in 32-bit bank. Manufacturer ID 0x000000 Chip ID 0x000000
Intel/Sharp Extended Query Table at 0x0031
Using buffer write method
erase region 0: offset=0x0,size=0x40000,blocks=256
Concatenating MTD devices:
(0): "40000000.flash"
(1): "40000000.flash"
into device "40000000.flash"
libphy: Fixed MDIO Bus: probed
usbcore: registered new interface driver r8152
isp1760 4f000000.usb: bus width: 32, oc: digital
isp1760 4f000000.usb: NXP ISP1760 USB Host Controller
isp1760 4f000000.usb: new USB bus registered, assigned bus number 1
isp1760 4f000000.usb: Scratch test failed.
isp1760 4f000000.usb: can't setup: -19
isp1760 4f000000.usb: USB bus 1 deregistered
usbcore: registered new interface driver usb-storage
mousedev: PS/2 mouse device common for all mice
usbcore: registered new interface driver xpad
rtc-pl031 10017000.rtc: rtc core: registered pl031 as rtc0
i2c /dev entries driver
usbcore: registered new interface driver uvcvideo
USB Video Class driver (1.1.1)
mmci-pl18x 10005000.mmci: Got CD GPIO
mmci-pl18x 10005000.mmci: Got WP GPIO
mmci-pl18x 10005000.mmci: Linked as a consumer to regulator.1
mmci-pl18x 10005000.mmci: mmc0: PL181 manf 41 rev0 at 0x10005000 irq 21,22 (pio)
usbcore: registered new interface driver usbhid
usbhid: USB HID core driver
ashmem: initialized
aaci-pl041 10004000.aaci: ARM AC'97 Interface PL041 rev0 at 0x10004000, irq 20
aaci-pl041 10004000.aaci: FIFO 512 entries
oprofile: hardware counters not available
oprofile: using timer interrupt.
NET: Registered protocol family 17
8021q: 802.1Q VLAN Support v1.8
9pnet: Installing 9P2000 support
Key type dns_resolver registered
Registering SWP/SWPB emulation handler
input: AT Raw Set 2 keyboard as /devices/platform/smb@4000000/smb@4000000:motherboard/smb@4000000:motherboard:iofpga@7,00000000/10006000.kmi/serio0/input/input0
mmc0: new SD card at address 4567
rtc-pl031 10017000.rtc: setting system clock to 2021-07-15 08:08:16 UTC (1626336496)
ALSA device list:
  #0: ARM AC'97 Interface PL041 rev0 at 0x10004000, irq 20
mmcblk0: mmc0:4567 QEMU! 1.00 GiB 
random: fast init done
 mmcblk0: p1 p2 p3 p4
input: ImExPS/2 Generic Explorer Mouse as /devices/platform/smb@4000000/smb@4000000:motherboard/smb@4000000:motherboard:iofpga@7,00000000/10007000.kmi/serio1/input/input2
EXT4-fs (mmcblk0p2): Filesystem with huge files cannot be mounted RDWR without CONFIG_LBDAF
EXT4-fs (mmcblk0p2): mounted filesystem with ordered data mode. Opts: (null)
VFS: Mounted root (ext4 filesystem) readonly on device 179:2.
devtmpfs: mounted
Freeing unused kernel memory: 1024K
Run /init as init process
random: crng init done
init: init first stage started!
init: [libfs_mgr]ReadFstabFromDt(): failed to read fstab from dt
init: [libfs_mgr]ReadDefaultFstab(): failed to find device default fstab
init: Failed to fstab for first stage mount
init: Using Android DT directory /proc/device-tree/firmware/android/
init: [libfs_mgr]ReadDefaultFstab(): failed to find device default fstab
init: First stage mount skipped (missing/incompatible/empty fstab in device tree)
init: Skipped setting INIT_AVB_VERSION (not in recovery mode)
init: Loading SELinux policy
SELinux:  policy capability network_peer_controls=1
SELinux:  policy capability open_perms=1
SELinux:  policy capability extended_socket_class=1
SELinux:  policy capability always_check_network=0
SELinux:  policy capability cgroup_seclabel=0
SELinux:  policy capability nnp_nosuid_transition=1
audit: type=1403 audit(1626336498.640:2): auid=4294967295 ses=4294967295 lsm=selinux res=1
selinux: SELinux: Loaded policy from /vendor/etc/selinux/precompiled_sepolicy

selinux: SELinux: Loaded file_contexts

init: init second stage started!
init: Using Android DT directory /proc/device-tree/firmware/android/
selinux: SELinux: Loaded file_contexts

init: Running restorecon...
selinux: SELinux:  Could not stat /dev/block: No such file or directory.

init: Overriding previous 'ro.' property 'ro.control_privapp_permissions':'disable' with new value 'enforce'
init: Overriding previous 'ro.' property 'ro.config.notification_sound':'OnTheHunt.ogg' with new value 'pixiedust.ogg'
init: Couldn't load property file '/product_services/build.prop': open() failed: No such file or directory: No such file or directory
init: Couldn't load property file '/factory/factory.prop': open() failed: No such file or directory: No such file or directory
init: Setting product property ro.product.brand to 'Android' (from ro.product.odm.brand)
audit: type=1400 audit(1626336499.180:3): avc:  denied  { mounton } for  pid=1 comm="init" path="/" dev="mmcblk0p2" ino=2 scontext=u:r:init:s0 tcontext=u:object_r:unlabeled:s0 tclass=dir permissive=1
audit: type=1400 audit(1626336499.210:4): avc:  denied  { search } for  pid=829 comm="init" name="/" dev="mmcblk0p2" ino=2 scontext=u:r:vendor_init:s0 tcontext=u:object_r:unlabeled:s0 tclass=dir permissive=1
audit: type=1400 audit(1626336499.910:5): avc:  denied  { search } for  pid=831 comm="ueventd" name="/" dev="mmcblk0p2" ino=2 scontext=u:r:ueventd:s0 tcontext=u:object_r:unlabeled:s0 tclass=dir permissive=1
audit: type=1400 audit(1626336499.980:6): avc:  denied  { search } for  pid=832 comm="apexd" name="/" dev="mmcblk0p2" ino=2 scontext=u:r:apexd:s0 tcontext=u:object_r:unlabeled:s0 tclass=dir permissive=1
apexd: Bootstrap subcommand detected
ueventd: ueventd started!
apexd: Scanning /system/apex for embedded keys
apexd: Scanning /product/apex for embedded keys
apexd: ... does not exist. Skipping
selinux: SELinux: Loaded file_contexts

ueventd: Parsing file /ueventd.rc...
apexd: Scanning /system/apex looking for APEX packages.
apexd: Found /system/apex/com.android.runtime-hos.debug
ueventd: Parsing file /vendor/ueventd.rc...
ueventd: Unable to read config file '/vendor/ueventd.rc': open() failed: No such file or directory
ueventd: Parsing file /odm/ueventd.rc...
apexd: Successfully bind-mounted flattened package /system/apex/com.android.runtime-hos.debug on /apex/com.android.runtime@1
ueventd: Unable to read config file '/odm/ueventd.rc': open() failed: No such file or directory
ueventd: Parsing file /ueventd.unknown.rc...
ueventd: Unable to read config file '/ueventd.unknown.rc': open() failed: No such file or directory
apexd: Activated 1 packages. Skipped: 0
apexd: Bootstrapping done
ueventd: [libfs_mgr]ReadDefaultFstab(): failed to find device default fstab
```

#### 固化启动命令

每次进入到 U-Boot 都要敲这么多命令，也是相当麻烦。如果上面 引导内核 没有问题的话，那么可以在 U-Boot 的代码中将上面的命令固化进 u-boot 程序中。

进入 U-Boot 源码，打开 include/configs/vexpress_common.h 文件，增加一行 CONFIG_BOOTCOMMAND 定义：
改成

```
#define CONFIG_BOOTCOMMAND  "load mmc 0:1 0x60008000 zImage ;load mmc 0:1 0x61000000 vexpress-v2p-ca9.dtb; setenv bootargs \"root=/dev/mmcblk0p2 earlycon console=ttyAMA0\"; bootz 0x60008000 - 0x61000000"
```

重新编译 U-Boot，再次使用 QEMU 运行，在启动倒计时过程中，不按任何键，就会自动执行这一行启动命令。

#### 小结

本文介绍了从 U-Boot 源码编译 u-boot，接着介绍了从鸿蒙标准系统的 system 镜像和 vendor 镜像文件获取根文件系统，并制作 SD 卡镜像，然后介绍 QEMU 使用编译出来的 u-boot 启动，并挂载 SD 卡镜像，从 SD 卡镜像中读取并加载 Linux 内核，最后介绍了将命令固化到 u-boot 程序中。

最后，系统启动仍然存在问题，下一步解决设备挂载问题，敬请关注！
