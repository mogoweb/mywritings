# 详解国密SSL ECC_SM4_SM3套件

国密算法最好的应用场景应该是SSL/TLS通信，然而国密文档中并没有单独规范SSL/TLS协议，我们能参考的只有《GM/T 0024-2014 SSL VPN 技术规范》。这份文档并没有像RFC那样描述得很详细，在实现上可能会存在很多不清楚的地方。很多时候，我们还要去翻看标准TLS 1.1的RFC4346。

本文主要总结国密SSL ECC_SM4_SM3密码套件的实现需要注意的地方。

因为国密SSL是以TLS 1.1标准为蓝本制定的，所以这里主要总结国密SSL协议和标准的TLS协议之间的区别。

在SSL通信中，最重要的是通信握手，握手成功后，就可以通过加密通道进行通信，握手过程如下：

![SSL握手](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_gmtls_01.png)

#### 1. 协议版本

TLS协议定义有三个版本号，为0x0301、0x0302、0x0303，分别对应TLS 1.0、1.1、1.2。国密SSL估计是担心与未来的TLS版本号冲突，选择了0x0101。这在实现上带来一定的麻烦，因为现有很多网络库会认为这是一个无效的协议版本，需要一一将判断修改过来。

国密SSL协议规范是TLS 1.1和TLS 1.2的混合体，大部分情况下参考TLS 1.1就可以了，少数地方又参考了TLS 1.2。在后面会指出哪些是参考了TLS 1.2的标准。

#### 2. 加密算法

非对称加密、对称加密、摘要等算法都替换为国密标准。在ECC_SM4_SM3套件中，非对称加密算法为SM2，对称加密算法为SM4，摘要算法为SM3。

注意，PRF算法和TLS 1.2类似，而不是像TLS 1.1那样，实现时需要注意：

![国密PRF函数定义](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_gmtls_02.png)

TLS1.2下，PRF中所使用的哈希算法可根据选择，比如SHA256，而GM SSL的算法固定为SM3。

#### 3. Certificate报文

国密规范规定发送证书时需要发送两个证书：签名证书和加密证书（双证书体系）。关于双证书请参考我之前的文章：

[啥？双证书？](https://mp.weixin.qq.com/s/gQmufSLucKj1woN0tKTQUA)

与标准TLS报文格式一样，但至少要包含两个证书，签名证书在前，加密证书在后。但如果牵扯到证书链，问题就复杂了，而且协议这里也没有规定清楚。是签名证书 + 证书链 + 加密证书，还是签名证书 + 加密证书 + 证书链？在实现中发现　TASSL　采用的是前者，而**沃通测试网站**采用后者。在编码时请注意，最好是两者都兼容。

#### ４. Certificate Verify

证书的验签流程和标准ECC证书的验签流程也不同，请参考《GMT 0003.2-2012 SM2椭圆曲线公钥密码算法第2部分：数字签名算法》，在后面我会详细写一写SM2的数字签名算法。

#### ５. 密钥交换

实现时请参考RSA的密钥交换，而不要参考椭圆曲线密钥交换算法ECDH或ECDHE，这点需要注意。具体过程为：

1. 服务器发送SM2公钥（在**加密证书**中）到客户端。
2. 客户端生成Pre-Master Secret，用SM2公钥加密后传送给服务器。
3. 服务器使用SM2私钥解密，得到Pre-Master Secret，通过一定的处理得到Master Secret，再次处理得到会话密钥，这个也是一个SM4对称加密算法的密钥。
4. 客户端使用同样推导算法从Pre-Master Secret计算出Master Secret，然后再计算得到会话密钥。因为双方都是从同样的Pre-Master Secret值，使用同样的密钥派生算法计算出会话密钥，所以最后会得到同样的会话密钥。

在握手协议的最后一步，双方会互相发送Finished消息，其中就包含VerifyData，加密发送给对方。如果双方的密钥不同，校验不会通过。

关于SM2加密　Pre-Master Secret，请参考我之前的文章：

[详解国密SM2的加密和解密](https://mp.weixin.qq.com/s/Axj_oVvV2g-xSTLXO15c8w)

Server Key Exchange消息中包含的数据如下：

![Server Key Exchange消息](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_gmtls_03.png)

刚开始还不明白其中的　

```
Opaque ASN.1Cert<1, 2^24-1>;　
```

指的是什么，该如何处理这个消息。后来才明白，对于ECC_SM4_SM3套件而言，会话密钥其实主要由客户端决定。对于客户端而言，这个消息不处理也没有问题，所以我把这个消息的处理略过了。开发服务器端的朋友可以补充一下，说说这个证书指的是哪个证书，我估计是加密证书。

#### 6. Finished报文

Finish报文的内容也和TLS 1.1有差别，TLS 1.1的定义为：

```
      struct {
          opaque verify_data[12];
      } Finished;

      verify_data
          PRF(master_secret, finished_label, MD5(handshake_messages) +
          SHA-1(handshake_messages)) [0..11];
```

而国密SSL的Finished报文定义为：

![Finished消息](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_gmtls_04.png)

和TLS 1.2标准类似，但TLS 1.2可以使用标准的SHA1、SHA256、SHA384等进行hash运算，而国密SSL中，hash运算固定为SM3。

#### 小结

在实现国密SSL的过程中，碰到不少的坑，其实有些地方要是有人稍微点拨一下，也不至于摸索得那么辛苦。现在我把总结出来的问题列出来，希望对大家有所帮助。

如果大家有什么问题，欢迎交流。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)
