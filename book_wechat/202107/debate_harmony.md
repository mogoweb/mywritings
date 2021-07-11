# 吐槽一下开源鸿蒙系统

最近一直在研究开源鸿蒙系统，但碰到一个问题，卡壳了，弄得我茶不思饭不想。在上一篇文章[鸿蒙系统研究之四：根文件系统](https://mp.weixin.qq.com/s/NBVs6Wa8jKE5x8GcSSusvQ)中，碰到一个难题：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/harmonyos_rootfs_01.png)

这个问题的原因是 Linux 内核编译时，没有开启 SELINUX。然而开启 SELINUX 选项后，碰到了更大的麻烦：

```
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
SELinux: (dev mmcblk0p2, type ext4) has no security xattr handler
audit: type=1403 audit(1625477182.800:2): auid=4294967295 ses=4294967295 lsm=selinux res=1
selinux: SELinux: Loaded policy from /vendor/etc/selinux/precompiled_sepolicy

audit: type=1404 audit(1625477182.800:3): enforcing=1 old_enforcing=0 auid=4294967295 ses=4294967295 enabled=1 old-enabled=1 lsm=selinux res=1
audit: type=1400 audit(1625477182.810:4): avc:  denied  { read } for  pid=1 comm="init" name="plat_file_contexts" dev="mmcblk0p2" ino=382 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=0
selinux: No path given to file labeling backend

selinux: selinux_android_file_context: Error getting file context handle (No such file or directory)

audit: type=1400 audit(1625477182.820:5): avc:  denied  { read } for  pid=1 comm="init" name="product" dev="mmcblk0p2" ino=35 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=lnk_file permissive=0
audit: type=1400 audit(1625477182.820:6): avc:  denied  { read } for  pid=1 comm="init" name="vendor_file_contexts" dev="mmcblk0p2" ino=1259 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=0
audit: type=1400 audit(1625477182.820:7): avc:  denied  { read } for  pid=1 comm="init" name="etc" dev="mmcblk0p2" ino=27 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=lnk_file permissive=0
init: execv("/system/bin/init") failed: Permission denied
audit: type=1400 audit(1625477182.840:8): avc:  denied  { execute } for  pid=1 comm="init" name="init" dev="mmcblk0p2" ino=156 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=0
audit: type=1400 audit(1625477182.860:9): avc:  denied  { read } for  pid=1 comm="init" name="libbacktrace.so" dev="mmcblk0p2" ino=770 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=0
audit: type=1400 audit(1625477182.880:10): avc:  denied  { read } for  pid=1 comm="init" name="init" dev="mmcblk0p2" ino=156 scontext=u:r:kernel:s0 tcontext=u:object_r:unlabeled:s0 tclass=file permissive=0
init: #00 pc 00048b13  /system/bin/init
init: #01 pc 00068dcf  /system/bin/init
init: #02 pc 0000857d  /system/lib/libbase.so
init: #03 pc 0004b1fd  /system/bin/init
init: #04 pc 00059213  /system/lib/bootstrap/libc.so
```

SeLinux 是用来增强 Linux 系统安全的，在[聊一聊可信执行环境](https://mp.weixin.qq.com/s/0RQkQO8YFFf9V2y2TGw2Tg)这篇文章中，我谈到过，有时太过于安全，会给用户带来不便。就如同有些系统强制要求用户设置复杂的密码，但复杂的密码又不便于记忆。SeLinux 也是如此，很安全，但特别复杂，稍微弄错一点规则，就造成程序无法执行。在我以往做的产品中，由于 SeLinux 引起的麻烦数不胜数，最后干脆关闭掉。

带着这样的思路，我也是打算把 SeLinux 模式设置为 Permissive （只打印 log ，但不做权限限制）。从网上找了很多资料，也求助过鸿蒙系统的开发人员，还是未能解决问题。当然这也是我水平不足的原因，我之前做系统开发，但并没有涉及内核这一块，隔行如隔山，还需要进一步学习和研究。

这段时间一直沉浸在开源鸿蒙系统中，所谓爱之切，责之深，在此忍不住要吐槽一下开源鸿蒙系统。

首先是**文档问题**。大多数开源系统都存在文档不足的问题，而且很多开发人员信封源码就是最好的文档。但开源鸿蒙系统的问题并不在于文档少，而在于有些混乱，比如：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202107/images/debate_harmony_01.png)

Ubuntu 编译环境准备就有三个文档，内容也各不相同，到底应该读哪个文档。就算是有的文档适合标准系统、有的适合 Lite OS，那也应该写清楚。这样弄几个文档，还不如没有文档。

其次，**开源鸿蒙系统中使用了 AOSP 的预编译库和程序**。开源鸿蒙系统使用 AOSP 的源码没问题，但像这样基础系统都使用 AOSP，似乎有些说不过去。而且也没有说明使用哪个版本的 AOSP ，就在源码系统中放入了二进制文件，这对于第三方移植非常不友好。我在前面碰到的 init 执行问题，迟迟不好解决，就因为这个超级程序 init 是用 AOSP 预编译出来的，出了问题，我也没法通过源码去查问题。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202107/images/debate_harmony_02.png)

再次，还是要吐槽一下开源鸿蒙系统的构建系统，混杂了 GN、Make、JSON、Python脚本、Bash脚本等等，这是逼迫程序员拥有十八般武艺，才能把这些整明白。更让人痛苦的是，有些构建文件又是通过脚本生成的，这对于追踪问题又竖了一道障碍。

最后，开源鸿蒙系统并没有提供一个标准的参考平台，而是以海思的 3516DV3000 作为参考产品。研究 3516 的 kernel patch 和 kernel config 就让人痛苦，到底哪些是针对 3516 这个特定硬件的 patch，哪些是针对鸿蒙系统的 patch，让人难以分辨。patch 中还引入了符号链接，链接到 开源鸿蒙系统的 driver，没有像 AOSP 那样，kernel 和系统可以分开编译。也许如果和华为公司合作，这些都不是事儿。但对于普通开发者或者小公司，想尝鲜鸿蒙系统，太困难了。

前几天看到消息，荣耀系列也开始收到鸿蒙系统的推送，鸿蒙系统接入用户破 3000 万，的确是一个了不起的成绩。也许华为现在全部的力量都投入到对现有产品的适配上，来不及顾及开源项目。但对于操作系统而言，生态无疑更加重要，而生态有赖于更多玩家的入场，仅仅靠华为一家无法构建整个操作系统生态。希望华为能投入更多的力量在开源项目上，将鸿蒙系统移植到更多的产品上。

接下来我还会继续开源鸿蒙系统的移植，敬请关注！