# 如何将根证书预置到 firefox 浏览器发布包中

继续写一写国密开发相关的内容。在实现了国密算法后，用生成的 firefox 浏览器可以访问沃通的在线国密测试网站。但还不够完美，首次访问依然会出现如下安全警告：

![安全警告](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202007/images/builtin_certs_01.png)

其原因在于签发该证书的根证书不被 firefox 所信任。要让 firefox 信任该证书，必须将签发该证书的根证书植入到 firefox 中，有两种方法：

1. 在上述的警告界面，点击 **Advanced**，然后点击 **Add Excepiton ...**
2. 下载该根证书，然后通过 Firfox 的 **Preferences** 中的证书管理界面，导入根证书。

通过这样的操作，下次再访问该在线国密测试网站，就不会再出现安全警告。但这种操作也只有专业人员清楚，而且普通人看到安全警告，第一反应肯定是关掉这个网站。所以为了更好的用户体验，我们需要将一些国密证书预置到 firefox 发布包中。其实 firefox 中预置了一些根证书，但都是国际知名 CA 的根证书，而国密 CA 作为后来者，还没有大范围使用，所以国际上主流的浏览器（firefox、chrome、safari等）连国密都不支持，更别提预置国密相关根证书。

在网络上查找资料，兜兜转转找了一圈，也没有查找到有用的资料。钻研 firefox 的启动代码，跟踪了两天导入证书的代码，但初始化过程中并没有调用。正在一筹莫展的时候，忽然看到 NSS 库下的命令行工具 **addbuiltin**, 接着找到 gecko 源码 security/nss/lib/ckfw/builtins/ 目录下的 README 文件，详细说明了如何将自己的证书预置到 firefox 中。正是：

> 山穷水尽疑无路，柳暗花明又一村。

下面就说说如何预置国密根证书到 firefox 中。

#### 下载根证书

目前还没有查到有网站提供国密根证书的下载，所以采用一个笨的方法，通过浏览器访问网站的方式来获得证书。步骤如下：

* 使用国密浏览器访问 https://sm2test.ovssl.cn 。
* 在弹出的警告界面，点击 **Advanced**，然后点击 **Add Excepiton ...** 。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202007/images/builtin_certs_02.png)

* 在 **Add Security Exception** 界面，先点击 **Get Certificate** 按钮，然后再点击 **View ...** 。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202007/images/builtin_certs_03.png)

* 在 **Certificate Viewer** 界面，先点击 **Details**，然后再点击 **Export ...** 。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202007/images/builtin_certs_04.png)

* 导出格式请选择 DER 格式，因为目前 **addbuiltin** 只支持 DER 格式，而不是常见的 PEM 格式。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202007/images/builtin_certs_05.png)

[注]：如果手头是 PEM 格式的证书，可以使用 openssl 命令行工具转化为 DER 格式：

```
openssl x509 -outform der -in certificate.pem -out certificate.der
```

#### 编译出命令行工具 addbuiltin

虽然 NSS 中提供了 **addbuiltin** 的源码，但在构建 firefox 的时候，并没有编译出 **addbuiltin** 这个命令行工具。我们进入到该源码目录，可以看到里面有 Makefile 文件，但如果直接 make，会出现错误：

```
/bin/sh: 1: ../../coreconf/nsinstall/Linux5.3_x86_cc_glibc_PTH_DBG.OBJ/nsinstall: not found
../../coreconf/rules.mk:392: recipe for target 'Linux5.3_x86_cc_glibc_PTH_DBG.OBJ/addbuiltin.o' failed
make: *** [Linux5.3_x86_cc_glibc_PTH_DBG.OBJ/addbuiltin.o] Error 127
```

看样子还得加入到 firefox 的编译系统中才行， firefox 采用了复杂的 **mach** 构建系统，理解起来也相当费劲。在经过一番摸索后，找到了方法。

* 修改 config/external/nss/Makefile.in 文件，加入如下片段：

```
NSS_DIRS += \
  nss/cmd/addbuiltin \
  $(NULL)

```

* 再执行 mach build 命令，在 obj-x86_64-pc-linux-gnu/dist/bin/ 下就可以看到 **addbuiltin**
* 为了方便后续命令的执行，我们可以将 obj-x86_64-pc-linux-gnu/dist/bin/ 路径加入到系统环境变量 **PATH** 中。

#### 预置根证书

所有的预置根证书均存放在 certdata.txt 文件中，这个文件位于源码的 security/nss/lib/ckfw/builtins 目录下，是一个文本文件。firefox 的构建系统中有一个 perl 脚本，会处理该文本文件，然后生成对应的 C 代码，最后编译到 firefox 中。如果了解这个文件的结构，可以手工添加和删除里面的内容，但这样很容易出错，所以还是得借助 **addbuiltin** 这个命令行工具。

进入 security/nss/lib/ckfw/builtins 目录，然后执行命令：

```
addbuiltin -n "sm2 ovssl certificates" -t C,C,C < ~/Downloads/sm2testovsslcn.der >> certdata.txt
```

将 sm2 证书附加到 certdata.txt 文件的最后，再编译 firefox。再次打开 firefox 的证书管理界面，可以看到这个证书已经加入成功。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202007/images/builtin_certs_06.png)

这个时候再访问 https://sm2test.ovssl.cn，就不会再有讨厌的警告了。

#### 小结

本文介绍了 firefox 浏览器预置根证书的方法，虽然是以国密为例进行说明，其实对于某些国际标准证书也同样使用。比如有朋友反映使用 firefox 访问 https://www.bizconf.cn/about.html 这个网站，有安全警告，但是使用 chrome 浏览器就没有。究其原因在于网站并没有向 firefox 发送完整的证书链，解决的方法可以通过将根证书预置到 firefox 中解决。

虽然本文属于一则冷门技巧，但碰到这样的问题，解决起来也挺费劲，特此分享出来，供大家参考。欢迎大家阅读**国密开发**专辑中的其它文章。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)