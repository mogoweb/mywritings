# 鸿蒙系统研究之五：替换 AOSP 预编译库，关闭 SELinux

这是我的鸿蒙系统研究系列文章的第五篇，有兴趣还可以看看前面的文章：

* [鸿蒙系统研究第一步：从源码构建系统镜像](https://mp.weixin.qq.com/s/LofgJs_-oFP9cUjovXaiBQ)
* [鸿蒙系统研究之二：内核编译](https://mp.weixin.qq.com/s/DalVCNVyBqJWRK6vTIYRdQ)
* [鸿蒙系统研究之三：迈出平台移植第一步](https://mp.weixin.qq.com/s/7vUZqDI6f79kComjeG0Q_A)
* [鸿蒙系统研究之四：根文件系统](https://mp.weixin.qq.com/s/NBVs6Wa8jKE5x8GcSSusvQ)

另外，还有关于鸿蒙系统的看法：

* [我看鸿蒙系统](https://mp.weixin.qq.com/s/SeMGA1EYfcENcaeBfgG2hA)
* [这就是鸿蒙系统？](https://mp.weixin.qq.com/s/d1PzXcBEUdvzoKvRuQeMcQ)
* [吐槽一下开源鸿蒙系统](https://mp.weixin.qq.com/s/Hhq4k6mclEMTUKPjLzUZOQ)

言归正传，在我的上一篇文章 [吐槽一下开源鸿蒙系统](https://mp.weixin.qq.com/s/Hhq4k6mclEMTUKPjLzUZOQ) 中，我提到过，开源鸿蒙标准系统的系统文件主要来自 AOSP 的预编译文件，这对于追踪启动过程中的问题非常不友好。我在 SeLinux 的问题上就卡壳了很久。

前几天在 gitee 上咨询鸿蒙系统的软件工程师，得知 Open Harmony 2.0 的 AOSP 的预编译文件来自 Android 10.0.0_r2 版本。为此下载 Android 10.0.0_r2 源码，编译之后替换，终于找到问题所在，顺利解决了 SeLinux 的问题。

Android 源码经过这么长时间的发展，代码越来越庞大，再加上众多分支，所以整个 repo 库非常庞大。整个源代码克隆下来有两三百个 G，如果网络不稳定，git 又没有断点续传，那就更痛苦了。建议使用国内的 AOSP 镜像站点，比如清华大学的 AOSP mirror 就不错。

```
$ repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-10.0.0_r2
$ repo sync
```

代码 sync 完毕之后，按照 Android 文档编译系统。

```
$ source build/envsetup.sh
$ lunch
You're building on Linux

Lunch menu... pick a combo:
     1. aosp_arm-eng
     2. aosp_arm64-eng
     3. aosp_blueline-userdebug
     4. aosp_bonito-userdebug
     5. aosp_car_arm-userdebug
     6. aosp_car_arm64-userdebug
     7. aosp_car_x86-userdebug
     8. aosp_car_x86_64-userdebug
     9. aosp_cf_arm64_phone-userdebug
     10. aosp_cf_x86_64_phone-userdebug
     11. aosp_cf_x86_auto-userdebug
     12. aosp_cf_x86_phone-userdebug
     13. aosp_cf_x86_tv-userdebug
     14. aosp_crosshatch-userdebug
     15. aosp_marlin-userdebug
     16. aosp_sailfish-userdebug
     17. aosp_sargo-userdebug
     18. aosp_taimen-userdebug
     19. aosp_walleye-userdebug
     20. aosp_walleye_test-userdebug
     21. aosp_x86-eng
     22. aosp_x86_64-eng
     23. beagle_x15-userdebug
     24. fuchsia_arm64-eng
     25. fuchsia_x86_64-eng
     26. hikey-userdebug
     27. hikey64_only-userdebug
     28. hikey960-userdebug
     29. hikey960_tv-userdebug
     30. hikey_tv-userdebug
     31. m_e_arm-userdebug
     32. mini_emulator_arm64-userdebug
     33. mini_emulator_x86-userdebug
     34. mini_emulator_x86_64-userdebug
     35. poplar-eng
     36. poplar-user
     37. poplar-userdebug
     38. qemu_trusty_arm64-userdebug
     39. uml-userdebug

Which would you like? [aosp_arm-eng]  

============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=10
TARGET_PRODUCT=aosp_arm
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv7-a-neon
TARGET_CPU_VARIANT=generic
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-5.4.0-77-generic-x86_64-Ubuntu-18.04.5-LTS
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=QP1A.190711.020
OUT_DIR=out
============================================
$ make -j16
```

这里选择 aosp_arm-eng，这是针对 ARM 32 位处理器的最通用版本，eng 版本的日志输出也更多，比较适合查问题。

编译完毕后，在 OpenHarmony 2.0 源码目录下的 **prebuilts\/aosp_prebuilt_libs\/** 目录，有一个脚本 **update_prebuilts.sh**，可以更新预置版本。

```
$ ./update_prebuilts.sh --source_dir ${AOSP_SRC_ROOT}/out/target/product/generic --prebuilts_dir .
```

接下来编译 OpenHarmony 2.0 系统，得到的镜像就是更新过 AOSP 预编译库的版本了。在 Android 系统中，SeLinux 可以有两种模式：permissive 和 enforcing。permissive 模式碰到 SeLinux 安全问题，会打印出警告，但不会阻止执行，比较适合产品开发阶段。如果希望正式产品中拥有更高的安全，将模式设置为 enforcing，将会进入严格的安全模式。

Android 系统中，init 程序会检查一个 androidboot.selinux 的内核参数值，如果其值为 permissive，那么 SeLinux 的模式设置为 permissive，否则设置为 enforcing。相关代码为：

```
// system/core/init/selinux.cpp

EnforcingStatus StatusFromCmdline() {
    EnforcingStatus status = SELINUX_ENFORCING;

    import_kernel_cmdline(false,
                          [&](const std::string& key, const std::string& value, bool in_qemu) {
                              if (key == "androidboot.selinux" && value == "permissive") {
                                  status = SELINUX_PERMISSIVE;
                              }
                          });

    return status;
}
```

找到问题就好办，解决的方法分两步。

首先，Linux 内核开启 SeLinux 支持，否则会出现 **init: mount("selinuxfs", "/sys/fs/selinux", "selinuxfs", 0, NULL) failed No such file or directory** 这样的错误，方法是修改内核编译选项，加入如下选项：

```
CONFIG_SECURITY=y
CONFIG_SECURITYFS=y
CONFIG_SECURITY_NETWORK=y
# CONFIG_SECURITY_NETWORK_XFRM is not set
CONFIG_SECURITY_PATH=y
CONFIG_LSM_MMAP_MIN_ADDR=32768
CONFIG_HAVE_HARDENED_USERCOPY_ALLOCATOR=y
# CONFIG_HARDENED_USERCOPY is not set
# CONFIG_FORTIFY_SOURCE is not set
# CONFIG_STATIC_USERMODEHELPER is not set
CONFIG_SECURITY_SELINUX=y
CONFIG_SECURITY_SELINUX_BOOTPARAM=y
CONFIG_SECURITY_SELINUX_BOOTPARAM_VALUE=1
# CONFIG_SECURITY_SELINUX_DISABLE is not set
CONFIG_SECURITY_SELINUX_DEVELOP=y
CONFIG_SECURITY_SELINUX_AVC_STATS=y
CONFIG_SECURITY_SELINUX_CHECKREQPROT_VALUE=1
# CONFIG_SECURITY_SMACK is not set
# CONFIG_SECURITY_TOMOYO is not set
CONFIG_SECURITY_APPARMOR=y
CONFIG_SECURITY_APPARMOR_BOOTPARAM_VALUE=1
CONFIG_SECURITY_APPARMOR_HASH=y
CONFIG_SECURITY_APPARMOR_HASH_DEFAULT=y
# CONFIG_SECURITY_APPARMOR_DEBUG is not set
# CONFIG_SECURITY_LOADPIN is not set
# CONFIG_SECURITY_YAMA is not set
CONFIG_INTEGRITY=y
# CONFIG_INTEGRITY_SIGNATURE is not set
CONFIG_INTEGRITY_AUDIT=y
# CONFIG_IMA is not set
# CONFIG_EVM is not set
CONFIG_DEFAULT_SECURITY_SELINUX=y
# CONFIG_DEFAULT_SECURITY_APPARMOR is not set
# CONFIG_DEFAULT_SECURITY_DAC is not set
CONFIG_DEFAULT_SECURITY="selinux"
```

接下来，在 QEMU 的启动参数中增加 androidboot.selinux=permissive ：

```
$ qemu-system-arm -M vexpress-a9 -m 512M -dtb ./out/KERNEL_OBJ/kernel/src_tmp/linux-4.19/arch/arm/boot/dts/vexpress-v2p-ca9.dtb -kernel ./out/KERNEL_OBJ/kernel/src_tmp/linux-4.19/arch/arm/boot/zImage -append "root=/dev/mmcblk0 rw console=ttyAMA0 init=/init androidboot.selinux=permissive" -sd ./device/qemu/vexpress-a9/rootfs.ext3 -nographic
WARNING: Image format was not specified for './device/qemu/vexpress-a9/rootfs.ext3' and probing guessed raw.
         Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.
         Specify the 'raw' format explicitly to remove the restrictions.
pulseaudio: set_sink_input_volume() failed
pulseaudio: Reason: Invalid argument
pulseaudio: set_sink_input_mute() failed
pulseaudio: Reason: Invalid argument
Booting Linux on physical CPU 0x0
Linux version 4.19.155+ (alex@alex-MS-7C22) (Android (dev based on r353983c) clang version 9.0.3 (https://android.googlesource.com/toolchain/clang 745b335211bb9eadfa6aa6301f84715cee4b37c5) (https://android.googlesource.com/toolchain/llvm 60cf23e54e46c807513f7a36d0a7b777920b5881) (based on LLVM 9.0.3svn)) #1 SMP Mon Jul 12 09:17:21 CST 2021
CPU: ARMv7 Processor [410fc090] revision 0 (ARMv7), cr=10c5387d
CPU: PIPT / VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
OF: fdt: Machine model: V2P-CA9
Memory policy: Data cache writeback
cma: Reserved 64 MiB at 0x7c000000
CPU: All CPU(s) started in SVC mode.
random: get_random_bytes called from start_kernel+0x88/0x39c with crng_init=0
percpu: Embedded 14 pages/cpu s28108 r8192 d21044 u57344
Built 1 zonelists, mobility grouping on.  Total pages: 130048
Kernel command line: root=/dev/mmcblk0 rw console=ttyAMA0 init=/init androidboot.selinux=permissive
log_buf_len individual max cpu contribution: 4096 bytes
log_buf_len total cpu_extra contributions: 12288 bytes
log_buf_len min size: 16384 bytes
log_buf_len: 32768 bytes
early log buf free: 14448(88%)
Dentry cache hash table entries: 65536 (order: 6, 262144 bytes)
Inode-cache hash table entries: 32768 (order: 5, 131072 bytes)
Memory: 442044K/524288K available (8192K kernel code, 252K rwdata, 1656K rodata, 1024K init, 195K bss, 16708K reserved, 65536K cma-reserved, 0K highmem)
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
Calibrating local timer... 96.87MHz.
Calibrating delay loop... 1162.44 BogoMIPS (lpj=5812224)
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
SMP: Total of 1 processors activated (1162.44 BogoMIPS).
CPU: All CPU(s) started in SVC mode.
devtmpfs: initialized
VFP support v0.3: implementor 41 architecture 3 part 30 variant 9 rev 0
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
futex hash table entries: 1024 (order: 4, 65536 bytes)
pinctrl core: initialized pinctrl subsystem
NET: Registered protocol family 16
DMA: preallocated 256 KiB pool for atomic coherent allocations
audit: initializing netlink subsys (disabled)
audit: type=2000 audit(0.220:1): state=initialized audit_enabled=0 res=1
cpuidle: using governor ladder
Serial: AMBA PL011 UART driver
10009000.uart: ttyAMA0 at MMIO 0x10009000 (irq = 29, base_baud = 0) is a PL011 rev1
console [ttyAMA0] enabled
1000a000.uart: ttyAMA1 at MMIO 0x1000a000 (irq = 30, base_baud = 0) is a PL011 rev1
1000b000.uart: ttyAMA2 at MMIO 0x1000b000 (irq = 31, base_baud = 0) is a PL011 rev1
1000c000.uart: ttyAMA3 at MMIO 0x1000c000 (irq = 32, base_baud = 0) is a PL011 rev1
OF: amba_device_add() failed (-19) for /smb@4000000/motherboard/iofpga@7,00000000/wdt@f000
OF: amba_device_add() failed (-19) for /memory-controller@100e0000
OF: amba_device_add() failed (-19) for /memory-controller@100e1000
OF: amba_device_add() failed (-19) for /watchdog@100e5000
irq: type mismatch, failed to map hwirq-75 for interrupt-controller@1e001000!
SCSI subsystem initialized
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
clcd-pl11x 10020000.clcd: clcd@10020000 hardware, 1024x768@59 display
brd: module loaded
40000000.flash: Found 2 x16 devices at 0x0 in 32-bit bank. Manufacturer ID 0x000000 Chip ID 0x000000
Intel/Sharp Extended Query Table at 0x0031
Using buffer write method
40000000.flash: Found 2 x16 devices at 0x0 in 32-bit bank. Manufacturer ID 0x000000 Chip ID 0x000000
Intel/Sharp Extended Query Table at 0x0031
Using buffer write method
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
mmci-pl18x 10005000.mmci: mmc0: PL181 manf 41 rev0 at 0x10005000 irq 25,26 (pio)
usbcore: registered new interface driver usbhid
usbhid: USB HID core driver
ashmem: initialized
aaci-pl041 10004000.aaci: ARM AC'97 Interface PL041 rev0 at 0x10004000, irq 24
aaci-pl041 10004000.aaci: FIFO 512 entries
oprofile: hardware counters not available
oprofile: using timer interrupt.
NET: Registered protocol family 17
8021q: 802.1Q VLAN Support v1.8
9pnet: Installing 9P2000 support
Key type dns_resolver registered
Registering SWP/SWPB emulation handler
rtc-pl031 10017000.rtc: setting system clock to 2021-07-12 08:36:39 UTC (1626078999)
ALSA device list:
  #0: ARM AC'97 Interface PL041 rev0 at 0x10004000, irq 24
input: AT Raw Set 2 keyboard as /devices/platform/smb@4000000/smb@4000000:motherboard/smb@4000000:motherboard:iofpga@7,00000000/10006000.kmi/serio0/input/input0
mmc0: new SD card at address 4567
mmcblk0: mmc0:4567 QEMU! 1.00 GiB 
input: ImExPS/2 Generic Explorer Mouse as /devices/platform/smb@4000000/smb@4000000:motherboard/smb@4000000:motherboard:iofpga@7,00000000/10007000.kmi/serio1/input/input2
EXT4-fs (mmcblk0): mounting ext3 file system using the ext4 subsystem
random: fast init done
EXT4-fs (mmcblk0): mounted filesystem with ordered data mode. Opts: (null)
VFS: Mounted root (ext3 filesystem) on device 179:0.
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
init: Compiling SELinux policy
init: Loading compiled SELinux policy
SELinux:  policy capability network_peer_controls=1
SELinux:  policy capability open_perms=1
SELinux:  policy capability extended_socket_class=1
SELinux:  policy capability always_check_network=0
SELinux:  policy capability cgroup_seclabel=0
SELinux:  policy capability nnp_nosuid_transition=1
audit: type=1403 audit(1626079005.990:2): auid=4294967295 ses=4294967295 lsm=selinux res=1
selinux: SELinux: Loaded policy from /dev/sepolicy.wMgsPa

audit: type=1400 audit(1626079006.000:3): avc:  denied  { read } for  pid=1 comm="init" name="plat_file_contexts" dev="mmcblk0" ino=57703 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=1
audit: type=1400 audit(1626079006.000:4): avc:  denied  { read } for  pid=1 comm="init" name="product" dev="mmcblk0" ino=22 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=lnk_file permissive=1
audit: type=1400 audit(1626079006.010:5): avc:  denied  { getattr } for  pid=1 comm="init" path="/system/etc/selinux/plat_file_contexts" dev="mmcblk0" ino=57703 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=1
audit: type=1400 audit(1626079006.010:6): avc:  denied  { open } for  pid=1 comm="init" path="/system/etc/selinux/plat_file_contexts" dev="mmcblk0" ino=57703 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=1
audit: type=1400 audit(1626079006.030:7): avc:  denied  { map } for  pid=1 comm="init" path="/system/etc/selinux/plat_file_contexts" dev="mmcblk0" ino=57703 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=1
selinux: SELinux: Loaded file_contexts

audit: type=1400 audit(1626079006.040:8): avc:  denied  { getattr } for  pid=1 comm="init" path="/system" dev="mmcblk0" ino=57348 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=dir permissive=1
audit: type=1400 audit(1626079006.060:9): avc:  denied  { relabelfrom } for  pid=1 comm="init" name="init" dev="mmcblk0" ino=57387 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=1
audit: type=1400 audit(1626079006.060:10): avc:  denied  { execute } for  pid=1 comm="init" name="linker" dev="mmcblk0" ino=57642 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=1
audit: type=1400 audit(1626079006.070:11): avc:  denied  { execute } for  pid=1 comm="init" path="/system/bin/bootstrap/linker" dev="mmcblk0" ino=57642 scontext=u:r:init:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=1
init: init second stage started!
init: Using Android DT directory /proc/device-tree/firmware/android/
selinux: SELinux: Loaded file_contexts

init: Running restorecon...
selinux: SELinux:  Could not stat /dev/block: No such file or directory.

init: Couldn't load property file '/product_services/build.prop': open() failed: No such file or directory: No such file or directory
init: Couldn't load property file '/factory/factory.prop': open() failed: No such file or directory: No such file or directory
init: Setting product property ro.product.brand to 'Android' (from ro.product.odm.brand)
init: Setting product property ro.product.device to 'generic' (from ro.product.odm.device)
init: Setting product property ro.product.manufacturer to 'unknown' (from ro.product.odm.manufacturer)
ueventd: ueventd started!
selinux: SELinux: Loaded file_contexts

ueventd: Parsing file /ueventd.rc...
ueventd: Parsing file /vendor/ueventd.rc...
ueventd: Unable to read config file '/vendor/ueventd.rc': open() failed: No such file or directory
ueventd: Parsing file /odm/ueventd.rc...
ueventd: Unable to read config file '/odm/ueventd.rc': open() failed: No such file or directory
ueventd: Parsing file /ueventd.unknown.rc...
ueventd: Unable to read config file '/ueventd.unknown.rc': open() failed: No such file or directory
ueventd: [libfs_mgr]ReadDefaultFstab(): failed to find device default fstab

```

可以看到，依然存在如下 SeLinux 的错误信息，但不会阻止程序的运行。

```
audit: type=1400 audit(1626079006.000:3): avc:  denied  { read } for  pid=1 comm="init" name="plat_file_contexts" dev="mmcblk0" ino=57703 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=1
```

目前尚不清楚 OpenHarmony 在 AOSP 库上做了哪些修改，官方也没有提供 patch。理论上应该做了修改，否则 init 程序启动的不就成了 Android 系统吗？

所以这里替换 AOSP 预编译库，仅仅是作为一种查找问题的手段，看后续 OpenHarmony 是否会修改这一部分的实现，或者提供 patch。

最后，如上面的输出所看到的，系统依然启动存在问题，这个没有关系，遇到问题解决问题，敬请关注后续的研究！

