
1. 下载源码
   
   ```bash
   git clone git clone ssh://<your_user_name>@gerrit.china-liantong.com:29000/gmbrowser/gmssl
   ```

2. build源码
   
   为了避免gmssl安装到系统目录，影响现有openssl，可以指定一个安装路径，比如 $HOME/bin/gmssl。

   ```bash
   ./config --prefix=/home/alex/bin/gmssl
   make
   ```

3. 安装
   
   如果上一步prefix指定的目录当前用户有写入权限：

   ```bash
   make install
   ```

   如果是系统路径，前面加上 sudo：

   ```bash
   sudo make install
   ```


国密标准文档 https://github.com/guanzhi/GM-Standards

沃通搭建了SM2 SSL证书的测试网站：https://sm2test.ovssl.cn

只支持国密的测试网站： https://sm2only.ovssl.cn/

apache-tomcat-7.0.85国密SSL的配置方法：https://blog.csdn.net/upset_ming/article/details/80360507

版本列表：

Firefox： ESR 52.7.2
NSS：     NSS_3_28_6_RTM


编译CURL，第一步使用nss作为ssl库，接着加入GmSSL：

下载源码包：

[curl-7.58.0.tar.gz](https://curl.haxx.se/download/curl-7.58.0.tar.gz)
[nss-3.28.6-with-nspr-4.16.tar.gz](https://ftp.mozilla.org/pub/security/nss/releases/NSS_3_28_6_RTM/src/nss-3.28.6-with-nspr-4.16.tar.gz)

编译代码：

1. build nss：
   
   ```bash
   nss/build.sh
   ```

2. 复制nss和nspr头文件
   
   ```bash
   cp dist/public/nss/* dist/Debug/include/
   mv dist/Debug/include/nspr/* dist/Debug/include/

   
   ```

3. build curl：
   
   ```bash
   ./configure --prefix=/data/gmbrowser/src/exp/opt/curl --without-ssl --with-nss=/data/gmbrowser/src/exp/nss-3.28.6/dist/Debug
   make && make install
   ```

验证curl是否使用了NSS：

```bash
alex@alex-MS-7C22:/data/gmbrowser/src/exp/opt/curl/bin$ ./curl -V
curl 7.58.0 (x86_64-pc-linux-gnu) libcurl/7.58.0 NSS/3.28.6 zlib/1.2.11
Release-Date: 2018-01-24
Protocols: dict file ftp ftps gopher http https imap imaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS IPv6 Largefile NTLM NTLM_WB SSL libz UnixSockets HTTPS-proxy
```

### 编译NSS samples

1. 下载NSS samples源码
   ```bash
   mkdir /work/gmbrowser/src/nss-samples
   cd /work/gmbrowser/src/nss-samples
   hg clone https://hg.mozilla.org/projects/nss
   cd nss
   hg update SAMPLES_BRANCH
   ```

2. 复制nss和nspr头文件

   ```bash
   # for nss sample
   cd /work/gmbrowser/src/dist
   cp -r Debug/include/ .
   ```

3. 修改 makesample.sh 脚本
国密SSL协议的握手过程如下：

(1)交换Hello消息来协商密码套件，交换随机数，决定是否会话重用；

(2)交换必要的参数，协商预主密钥

(3)交换证书信息，用于验证对方

(4)使用预主密钥和交换的随机数生成主密钥

(5)向记录层提供安全参数

(6)验证双方计算的安全参数的一致性、握手过程的真实性和完整性


基于UKey数字证书实现身份认证：

https://blog.csdn.net/helihongzhizhuo/article/details/38819837