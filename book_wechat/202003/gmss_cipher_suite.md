# SSL通信双方如何判断对方采用了国密？

SSL通信涉及两方的参与者，通常采用的模型是Client/Server。如果我们开发Client端产品（比如浏览器），可能会和多方的Server产品对接。那么问题来了，双方是如何知道对方使用了国密算法呢？这个问题非常重要，只有双方采用同样的加密标准，才能正常通信。难道采用“天王盖地虎，宝塔镇河妖”的接头暗号？

在揭开这个谜底之前，我们先了解一下OID（Object Identifier，对象标志符），然后再聊一聊密码套件（Cipher Suite），就可以慢慢理解SSL通信双方是如何接上头的。当然，整个SSL通信涉及到很多标准和协议，不是一篇文章能讲完的，所以本文只是抛砖引玉，探讨一些基本的概念。可能对于资深人士而言，不值得一提，但我初入门时，也是摸索了好久，才慢慢能深入到细节。

### OID

OID是由ISO/IEC、ITU-T国际标准化组织上世纪80年代联合提出的标识机制，其野心很大，为任何类型的对象（包括实体对象、虚拟对象和组合对象）进行全球唯一命名。通过唯一的编码，我们就可以识别出对象。但要为所有对象进行唯一命名，其难度和工作量都很大，所以它采用了分层树形结构。

OID对应于“OID树”或层次结构中的一个节点，该节点是使用ITU的OID标准X.660正式定义的。树的根包含以下三个起点：

```
0：ITU-T
1：ISO
2：ITU-T/ISO联合发布
```

树中的每个节点都由一系列由句点分隔的整数表示。比如，表示英特尔公司的OID如下所示：

> 1.3.6.1.4.1.343

对应于OID树中的含义：

```
1               ISO
1.3             识别组织
1.3.6           美国国防部
1.3.6.1         互联网
1.3.6.1.4       私人
1.3.6.1.4.1     IANA企业编号
1.3.6.1.4.1.343 英特尔公司
```
这里采用分而治之的策略，解决编码重复问题。树中的每个节点均由分配机构控制，该机构可以在该节点下定义子节点，并为子节点委托分配机构。 

在上面的例子中，根节点“1”下的节点号由ISO分配，“1.3.6”下的节点由美国国防部分配，“1.3.6.1.4.1”下的节点由IANA分配，“1.3.6.1.4.1.343”下的节点由英特尔公司分配，依此类推。只要有需求，可以一直往下分配下去，也解决了编码不够的问题。

在实际应用中，ISO/IEC国际标准化机构维护顶层OID标签，各个国家负责该国家分支下的OID分配、注册和解析等工作，实现自我管理和维护。

相应的，针对国密，国家密码局也定义了各类对象的标识符：

![国密相关OID定义](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_cipher_suite_01.png)

详细可参考 GM/T 0006-2012 这份标准。

像"1.2.156.10197.1.100"这种字符串，人读起来比较直观，但对于计算机处理而言，却是大大的不方便，要知道字符串处理的效率非常低，所以在程序代码中，对OID又进行了一次编码。

在GmSSL源码中，原始的OIDs定义在objects.txt文件中，在文件的尾部，我们可以看到国密的相关定义：

```
# GM/T
member-body 156		: ISO-CN		: ISO CN Member Body
ISO-CN 10197		: oscca
oscca 1			: sm-scheme

sm-scheme 103 1		: SSF33-ECB		: ssf33-ecb
sm-scheme 103 2		: SSF33-CBC		: ssf33-cbc
!Cname ssf33-ofb128
sm-scheme 103 3		: SSF33-OFB		: ssf33-ofb
!Cname ssf33-cfb128
sm-scheme 103 4		: SSF33-CFB		: ssf33-cfb
sm-scheme 103 5		: SSF33-CFB1		: ssf33-cfb1
sm-scheme 103 6		: SSF33-CFB8		: ssf33-cfb8
sm-scheme 103 7		: SSF33-CBC-MAC		: ssf33-cbc-mac

sm-scheme 102 1		: SM1-ECB		: sm1-ecb
sm-scheme 102 2		: SM1-CBC		: sm1-cbc
!Cname sm1-ofb128
sm-scheme 102 3		: SM1-OFB		: sm1-ofb
!Cname sm1-cfb128
sm-scheme 102 4		: SM1-CFB		: sm1-cfb
sm-scheme 102 5		: SM1-CFB1		: sm1-cfb1
sm-scheme 102 6		: SM1-CFB8		: sm1-cfb8


# SM2 OIDs
sm-scheme 301		: sm2p256v1
sm-scheme 301 1		: sm2sign
sm-scheme 301 2		: sm2exchange
sm-scheme 301 3		: sm2encrypt

```

通过 objects.pl 脚本，生成 obj_data.h文件，这个才是在代码中使用到的OID编码。

boringssl也类似，不过其采用了 go 语言编写的转换脚本。

### 密码套件

仅仅定义了OID还不够，因为国密并不是一个单一的标准，包含了很多加密、解密、哈希等算法，可以形成很多种组合，不能简单假定对方采用了国密就可以建立通信。

在SSL通信开始，双方就需要进行协商，采用何种算法进行通信。这里需要重点理解密码套件（CipherSuite）的概念。

密码套件是一系列密码学算法的组合，主要包含多个密码学算法：

* 身份验证算法。
* 密码协商算法。
* 加密算法或者加密模式。
* HMAC算法的加密基元。
* PRF算法的加密基元，需要注意的是，不同的TLS/SSL协议版本、密码套件，PRF算法最终使用的加密基元和HMAC算法使用的加密基元是不一样的。

密码套件的构成如下图所示:

![密码套件结构](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_cipher_suite_02.png)

密码套件决定了本次连接采用哪一种加密算法、密钥协商算法、HMAC算法，协商的结果就是双方都认可的密码套件。

密码套件协商过程有点类似于客户采购物品的过程，客户（客户端）在向商家（服务器端）买东西之前需要告知商家自己的需求、预算，商家了解用户的需求后，根据用户的具体情况（比如客户愿意接受的价格，客户期望物品的使用年限）给用户推荐商品，只有双方都满意了，交易才能完成。对于TLS/SSL协议来说，只有协商出密码套件，才能进行下一步的工作。

除了现有国际上标准的密码套件，国密还定义了一系列的密码套件：

![国密密码套件列表](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_cipher_suite_02.png)

代码可一参考 gmtls.h 文件，也可以使用GMSSL的命令查看所支持的密码套件：

```bash
openssl ciphers -V | column -t
```

与国密相关的密码套件有：

```
0xE1,0x07  -  ECDHE-SM2-WITH-SMS4-GCM-SM3    TLSv1.2    Kx=ECDH      Au=SM2    Enc=SMS4GCM(128)            Mac=AEAD
0xE1,0x02  -  ECDHE-SM2-WITH-SMS4-SM3        TLSv1.2    Kx=ECDH      Au=SM2    Enc=SMS4(128)               Mac=SM3
0xF1,0x20  -  ECDHE-PSK-WITH-SMS4-CBC-SM3    TLSv1      Kx=ECDHEPSK  Au=PSK    Enc=SMS4(128)               Mac=SM3
0xE0,0x17  -  SM9-WITH-SMS4-SM3              GMTLSv1.1  Kx=SM9       Au=SM9    Enc=SMS4(128)               Mac=SM3
0xE0,0x15  -  SM9DHE-WITH-SMS4-SM3           GMTLSv1.1  Kx=SM9DHE    Au=SM9    Enc=SMS4(128)               Mac=SM3
0xE0,0x13  -  SM2-WITH-SMS4-SM3              GMTLSv1.1  Kx=SM2       Au=SM2    Enc=SMS4(128)               Mac=SM3
0xE0,0x11  -  SM2DHE-WITH-SMS4-SM3           GMTLSv1.1  Kx=SM2DHE    Au=SM2    Enc=SMS4(128)               Mac=SM3
0xE0,0x1A  -  RSA-WITH-SMS4-SHA1             GMTLSv1.1  Kx=RSA       Au=RSA    Enc=SMS4(128)               Mac=SHA1
0xE0,0x19  -  RSA-WITH-SMS4-SM3              GMTLSv1.1  Kx=RSA       Au=RSA    Enc=SMS4(128)               Mac=SM3
0xF1,0x01  -  PSK-WITH-SMS4-CBC-SM3          SSLv3      Kx=PSK       Au=PSK    Enc=SMS4(128)               Mac=SM3
```

* 第一列：数值代表密码套件的编号，每个密码套件的编号由IANA定义。
* 第二列：代表密码套件的名称，虽然密码套件编号是一致的，不同的TLS/SSL协议实现其使用的名称可能是不一样的。
* 第三列：表示该密码套件适用于哪个TLS/SSL版本的协议。
* 第四列：表示密钥协商算法。
* 第五列：表示身份验证算法。
* 第六列：表示加密算法、加密模式、密钥长度。
* 第七列：表示HMAC算法。其中AEAD表示采用的是AEAD加密模式（比如AES128-GCM），无须HMAC算法。

值得注意的是，这里的编码又没有采用OID，这也是开发过程中需要注意的，不同的地方使用了不同标准规范，需要在开发中翻阅相应的协议和规范。

可以看出，GmSSL并没有实现所有的国密的密码套件，但同时又扩充了几个标准未定义的密码套件，比如ECDHE-SM2-WITH-SMS4-GCM-SM3、ECDHE-SM2-WITH-SMS4-SM3等。这就体现出协商的重要性了，对双方所支持的密码套件取一个交集，从中选择一个。如果不存在交集，协商也就不成功。注意，协商过程中服务器端可能会禁用一些不太安全的密码套件（比如历史遗留的一些现今已不太安全的算法），这时即使双方都支持，也可能协商不成功。

我们可以测试服务器是否支持某个特定的密码套件:

```bash
openssl s_client -cipher "ECDHE-SM2-WITH-SMS4-SM3" -connect sm2test.ovssl.cn:443 -tls1_2 -servername sm2test.ovssl.cn
```

当然，如何协商出密码套件，涉及到SSL通信协议细节，如果开发中涉及到，就需要翻阅相应的RFC，这里不做过多展开。

### 小结

本文简单介绍了OID和密码套件的概念，这对于理解SSL通信有很重要的帮助，如果从事SSL开发，开始面对一大堆的协议和标准，可能会懵圈，希望本文能给你一个探索国密的入口通道。在和第三方服务进行对接时，由于对这些概念理解不深，也是掉了很多坑，希望本文对你有所帮助。

如果大家有什么问题，欢迎交流。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)