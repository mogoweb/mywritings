# 国密SSL通信的调试技巧

前面写了几篇国密开发相关的文章：

* [解读国密非对称加密算法SM2](https://mp.weixin.qq.com/s/qFppMQsy6rsKOwG_AzGLEw)
* [啥？双证书？](https://mp.weixin.qq.com/s/gQmufSLucKj1woN0tKTQUA)
* [详解国密SM2的加密和解密](https://mp.weixin.qq.com/s/Axj_oVvV2g-xSTLXO15c8w)
* [详解国密SSL ECC_SM4_SM3套件](https://mp.weixin.qq.com/s/wT3BSOWkz7PmJep6L_jytw)
* [详解国密SM2的数字签名](https://mp.weixin.qq.com/s/fuQ-UFj3f3Hue4TFa935Vw)

这篇文章来聊一聊国密SSL通信的几个调试技巧。网络通信开发需要开发者具有细致和耐心，对照协议，逐个步骤分析数据，整个过程有些枯燥。特别是网络数据包，差一个字节都可能导致解析出错，只能逐个字节比对。这个时候，一些调试经验和技巧就比较重要了。

下面就说说我在开发支持国密的浏览器产品中使用到的一些技巧。

#### 单步调试

单步调试，这也能算是调试技巧吗？你还别说，我发现很多开发者宁可使用print大法，也不愿意采用单步调试，特别是在嵌入式开发领域、移动端开发及前端开发。有多少人开发网页中的js使用到了chrome和firefox的远程调试工具？

为啥不愿意采用单步调试？主要是单步调试需要配置，稍微有点麻烦。对于嵌入式开发和移动开发来说，通常需要在开发机上交叉编译（C/C++开发），将编译的二进制程序下载到设备上运行，一般没有Windows开发那样的IDE。如果要用gdb调试，有可能还需要用到gdbserver，配置gdb和gdbserver之间的通信，配置代码的调试符号路径等等。

但是，如果将单步调试环境配置好了，对于厘清程序运行流程、调试BUG有非常大的作用。俗话说，工欲利其事，必先利其器。花点时间配置一下调试环境，是非常值得的。

在Linux系统下调试**国密SSL通信**，准备的过程其实非常简单，不涉及交叉编译、gdbserver，只需要稍微掌握几个gdb命令即可。

下面以GmSSL的代码为例，说明如何单步调试。

* 编译带调试符号的二进制程序

  在文章[搭建国密SSL开发测试环境](https://mp.weixin.qq.com/s/rUeOtFjB3QTPJW3RKccKag)中说明了如何编译GmSSL，编译出来的是release版本，不带调试符号，这样就无法进行单步跟踪。要想单步调试，需要编译出debug版本：

  ```
  $ cd gmssl
  $ ./config --prefix=/home/alex/work/gmbrowser/usr/local/gmssl -d
  $ make
  $ make install
  ```

  是的，非常简单，在**config**的时候加上 **-d** 参数即可。

* gdb调试

  gdb其实也有图形界面前端，但使用上并不是很方便，通常情况下，我选择使用命令行，毕竟只需要掌握几个简单的**gdb**命令即可:

  ```
  1. **set args**     设置被调试程序的命令行参数
  2. **b <函数名> 或 b <文件名：行号>**   设置断点
  3. **r**   运行程序
  4. **n**   单步运行
  5. **c**   从断点出运行，直到遇到下一个断点或结束
  6. **p <变量名>**   输出变量值
  7. **bt**   显示断点处的调用栈
  ```

  还有很多其它命令，但常用的就这些，如果碰到不明白的地方，随时输入help，寻求帮助。 
  
  比如，如果我们希望调试客户端的HELLO处理流程，如果对代码结构有些了解，就可以知道是在**tls_process_server_hello**函数中处理的，如何调用到该函数的，函数内部处理过程是怎样的？这个时候单步调试就可以排上用场了。

  ```
  $ gdb gmssl
  (gdb) set args s_client -connect sm2test.ovssl.cn:443 -gmtls -servername sm2test.ovssl.cn
  (gdb) b tls_process_server_hello
  Function "tls_process_server_hello" not defined.
  Make breakpoint pending on future shared library load? (y or [n]) y
  Breakpoint 1 (tls_process_server_hello) pending.
  (gdb) r
  Starting program: /work/gmbrowser/usr/local/gmssl/bin/gmssl s_client -connect sm2test.ovssl.cn:443 -gmtls -servername sm2test.ovssl.cn
  [Thread debugging using libthread_db enabled]
  Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
  CONNECTED(00000003)

  Breakpoint 1, tls_process_server_hello (s=0x55555585a3f0, pkt=0x7fffffffce70) at ssl/statem/statem_clnt.c:988
  988	{
  (gdb) n
  994	    int i, al = SSL_AD_INTERNAL_ERROR;
  (gdb) n
  1002	    if (!PACKET_get_net_2(pkt, &sversion)) {
  (gdb) n
  1008	    protverr = ssl_choose_client_version(s, sversion);
  (gdb) bt
  #0  tls_process_server_hello (s=0x55555585a3f0, pkt=0x7fffffffce70) at ssl/statem/statem_clnt.c:1008
  #1  0x00007ffff7988046 in ossl_statem_client_process_message (s=0x55555585a3f0, pkt=0x7fffffffce70) at ssl/statem/statem_clnt.c:689
  #2  0x00007ffff79865c0 in read_state_machine (s=0x55555585a3f0) at ssl/statem/statem.c:643
  #3  0x00007ffff7986061 in state_machine (s=0x55555585a3f0, server=0) at ssl/statem/statem.c:439
  #4  0x00007ffff7985b34 in ossl_statem_connect (s=0x55555585a3f0) at ssl/statem/statem.c:218
  #5  0x00007ffff7960800 in ssl3_write_bytes (s=0x55555585a3f0, type=23, buf_=0x555555847960, len=0) at ssl/record/rec_layer_s3.c:366
  #6  0x00007ffff796cac3 in ssl3_write (s=0x55555585a3f0, buf=0x555555847960, len=0) at ssl/s3_lib.c:4126
  #7  0x00007ffff797a29c in SSL_write (s=0x55555585a3f0, buf=0x555555847960, num=0) at ssl/ssl_lib.c:1677
  #8  0x00005555555bec06 in s_client_main (argc=0, argv=0x7fffffffdc90) at apps/s_client.c:2271
  #9  0x00005555555a2a39 in do_cmd (prog=0x555555842180, argc=6, argv=0x7fffffffdc90) at apps/gmssl.c:471
  #10 0x00005555555a20fd in main (argc=6, argv=0x7fffffffdc90) at apps/gmssl.c:177
  ```

  这是不是比找到可疑代码，然后一句句加print语句来得快呢？

  当然，如果对于大块的数据，比如我怀疑某个加密算法实现得不对，需要对加密结果进行判断，这个时候还是用print语句来得块，毕竟gdb对于大块数据的显示还不是很直观。所以大部分情况下，是单步调试和print结合使用。

#### 网络抓包

从事网络通信相关开发的朋友，应该对网络抓包很熟悉。通常在PC上使用的抓包软件为wireshark，在Android系统上的抓包工具有tcpdump。但是国密为SSL版本号定义了一个非常坑爹的值：0x0101，而大多数软件包对0x0300以下的值都会被认为是一个无效的版本号，所以如果使用标准版的wireshark抓国密SSL包，结果是这样的：

![标准版wireshark抓包](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202005/images/gmssl_debug_01.png)

这非常不方便分析！

幸运的是，已经有朋友在wireshark上增加了国密SSL支持，抓包结果如下：

![支持国密版wireshark抓包](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202005/images/gmssl_debug_02.png)

这个支持国密的wireshark项目地址为：https://github.com/pengtianabc/wireshark-gm

这个项目的release中有Windows的安装包，如果是Windows开发者，不用过多折腾，安装上即可。

下面说说Ubuntu系统下从源码build出wireshark，并运行之。

1. clone源码
   
   ```
   $ git clone https://github.com/pengtianabc/wireshark-gm
   ```

2. 安装编译源码所需要的软件包
   
   ```
   $ sudo apt install libc-ares-dev flex qt5-default qttools5-dev qtmultimedia5-dev libpcap-dev libcap-dev cmake cmake-curses
   ```

3. 编译源码
   
   ```
   $ cd wireshark-gm
   $ mkdir build
   $ cd build
   $ cmake ..
   $ make
   ```

   编译出的wireshark位于 build/run 目录下。

4. 切换到sudo运行环境
   
   网络抓包需要root权限，否则会提示：

   ```
   Capturing on 'enp3s0'
   dumpcap: The capture session could not be initiated on interface 'enp3s0' (You don't have permission to capture on that device).
   Please check to make sure you have sufficient permissions.
   ```
   在网络上找了一些方法，非root用户也能够抓包，但折腾了半天，也没有成功（系统安装的wireshark倒是可以）。算了，先不折腾，就切换到root环境吧：

   ```
   sudo -i
   ```

5. 设置环境变量
   
   为了避免和系统的wireshark冲突，这里不建议执行 make install 将 wireshark-gm 安装到系统目录下，为了能够让系统找到我们的wireshark和库，设置环境变量：

   ```
   export PATH=/work/gmbrowser/src/wireshark-gm/build/run:$PATH
   export LD_LIBRARY_PATH=/work/gmbrowser/src/wireshark-gm/build/run:$LD_LIBRARY_PATH
   ```

6. 运行wireshark，就可以抓包了。要确定运行的是自己build的版本，而不是系统的wireshark，可以看到自己build的版本有**development build**字样：
   
   ![从源码build出的wireshark](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202005/images/gmssl_debug_03.png)

---

网络抓包对于现场调试，特别是在和第三方对接调试时非常有用。而在国密SSL通信中，主要调试建立国密SSL连接过程，在这个过程中，建立连接的几个阶段，分别发送了和接收了什么数据。有了对国密SSL协议的支持，看起来就清楚多了，否则还需要自己对照协议，一部分一部分的分析数据。

好了，关于国密开发的两点调试技巧就介绍到这里，希望对你有帮助。谢谢！

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)

