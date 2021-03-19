<!--
 * @Author: 陈正勇
 * @Date: 2021-02-22 10:54:30
 * @LastEditTime: 2021-02-22 11:17:38
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: /mywritings/book_wechat/202102/practical_cryptography_for_developers_asymmetric.md
-->
# 写给开发人员的实用密码学 - 非对称加密算法

在前面的文章《[写给开发人员的实用密码学 - 对称加密算法](https://mp.weixin.qq.com/s/hsQOA79Bk0oUDlBHVucvng)》中，介绍了现代密码学中非常重要的加密解密算法。从工程学的角度，选取密钥足够长的加密算法（比如AES 256、AES 512），是无法破解的。但在对称加密算法中，存在明显的薄弱环节，那就是密钥的存储与分发。因为算法是公开的，那么决定加密系统是否安全的因素就是密钥。

然而在开放的互联网环境下，如何高效分发密钥成了一个难题。比如一个服务器，可能会服务千万级的用户，如果为每个用户分配不同的密钥，那密钥的维护是个大问题。密钥发放也是问题，如果是企业内部，还可以通过邮件发送密钥，但在开放的互联网上，这种方式行不通。

公开密钥算法（Public KeyCryptography），也称为非对称加密算法（Asymmetrical Cryptography），可以用于解决这一难题。

顾名思义，非对称加密算法就是加密密钥和解密密钥不是同一个。算法的密钥是一对，分别是公钥（public key）和私钥（private key）。公一般私钥由密钥对的生成方（比如服务器端）持有，避免泄露，而公钥任何人都可以持有，也不怕泄露。这里没有使用加密密钥和解密密钥，是因为公钥和私钥都可以用来加密，也可以用来解密。如果使用公钥加密，就要使用私钥解密，反之依然。

公开密钥算法不是一个算法而是一组算法，通常提供如下算法：

* 密钥对生成：生成随机对的私钥+对应的公钥。
* 加密/解密：通过公钥加密数据，并通过私钥解密数据（通常使用混合加密方案）。
* 数字签名（消息认证）：通过私钥对消息签名，并通过公钥验证签名。
* 密钥交换算法：通过不安全的通道在两方之间安全地交换加密密钥。

如果公开密钥算法用于加密解密运算，习惯上称为非对称加密算法。非对称加密算法的加密解密过程如下图所示。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202103/images/practical_cryptography_for_developers_asymmetric_01.png)

目前非对称加密算法还无法取代对称加密算法，相比对称加密算法来说，公开密钥算法尤其是RSA算法运算非常缓慢，一般情况下，需要加密的明文数据都非常大，如果使用公开密钥算法进行加密，运算性能会惨不忍睹。公开密钥算法在密码学中一般进行密钥协商或者数字签名，因为这两者运算的数据相对较小。

目前，最重要和最常用的公钥密码系统是 RSA 和 椭圆曲线密码算法（ECC）。推荐使用 ECC，因为它具有较小的密钥，较短的签名和更好的性能。

国密 SM2 算法就是一种 ECC 算法，所以下面着重讲一讲 ECC。

#### ECC 算法

ECC是比离散对数类算法（比如RSA和DH算法）更复杂的算法，非常难于理解，本身也是很复杂的一个结构体，在理解起来的时候有一定的难度。不理解ECC理论知识没有关系，但需要了解以下这张图：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202103/images/practical_cryptography_for_developers_asymmetric_02.png)

ECC椭圆曲线由很多点组成，这些点由特定的方程式组成的，比如方程式可以是y^2 = x^3 + ax + b，这些点连接起来就是一条曲线，但曲线并不是一个椭圆。

椭圆曲线有个特点，任意两个点能够得到这条椭圆曲线上的另外一点，这个操作称为打点，经过多次（比如d次）打点后，能够生成一个最终点（F）。

在上面的图中，A点称为基点（G）或者生成器。A可以和自己打点从而生成B点，在实际应用的时候，一般有基点就可以了。经过多次打点，就得到了最终点G。

ECC密码学的关键点就在于就算知道具体方程式、基点（G）、最终点（F），也无法知晓一共打点了多少次（d）。

ECC中，打点次数(d)就是私钥，这通常是一个随机数，公钥就是最终点（F)，包含(x，y)两个分量，通常组合成一个数字来传输和存储。

ECC由方程式（比如a、b这样的方程式参数）、基点（G）、质数（P）组成。理论上方程式和各种参数组合可以是任意的，但是在密码学中，为了安全，系统预先定义了一系列的曲线，称为命名曲线（name curve），比如secp256k1就是一个命名曲线。对于开发者而言，在使用ECC密码学的时候，就是选择具体的命名曲线。

#### 命名曲线

ECC本质上就是一个数学公式，任何人基于公式都可以设计出椭圆曲线，在实现的时候一定要注意ECC离散对数问题（Elliptic-Curve Discrete-Logarithm Problem，简称ECDLP），如果实现不当，那么ECC公式就会存在安全风险。为了简化ECC的使用，可以选用设计规范的命名曲线，命名曲线中包含了ECC椭圆曲线的参数，比如基点、有限域等，对于大部分开发者来说，如果要使用ECC椭圆曲线，要做的就是选择一条安全且性能高的命名曲线。

常见的命名曲线有 secp192r1、secp256r1、secp384r1、secp512r1。

使用GmSSL命令行可以查看所实现的命名曲线。

```bash
$ gmssl ecparam -list_curves
  secp112r1 : SECG/WTLS curve over a 112 bit prime field
  secp112r2 : SECG curve over a 112 bit prime field
  secp128r1 : SECG curve over a 128 bit prime field
  secp128r2 : SECG curve over a 128 bit prime field
  secp160k1 : SECG curve over a 160 bit prime field
  secp160r1 : SECG curve over a 160 bit prime field
  secp160r2 : SECG/WTLS curve over a 160 bit prime field
  secp192k1 : SECG curve over a 192 bit prime field
  secp224k1 : SECG curve over a 224 bit prime field
  secp224r1 : NIST/SECG curve over a 224 bit prime field
  secp256k1 : SECG curve over a 256 bit prime field
  secp384r1 : NIST/SECG curve over a 384 bit prime field
  secp521r1 : NIST/SECG curve over a 521 bit prime field
  prime192v1: NIST/X9.62/SECG curve over a 192 bit prime field
  prime192v2: X9.62 curve over a 192 bit prime field
  prime192v3: X9.62 curve over a 192 bit prime field
  prime239v1: X9.62 curve over a 239 bit prime field
  prime239v2: X9.62 curve over a 239 bit prime field
  prime239v3: X9.62 curve over a 239 bit prime field
  prime256v1: X9.62/SECG curve over a 256 bit prime field
  sect113r1 : SECG curve over a 113 bit binary field
  sect113r2 : SECG curve over a 113 bit binary field
  sect131r1 : SECG/WTLS curve over a 131 bit binary field
  sect131r2 : SECG curve over a 131 bit binary field
  sect163k1 : NIST/SECG/WTLS curve over a 163 bit binary field
  sect163r1 : SECG curve over a 163 bit binary field
  sect163r2 : NIST/SECG curve over a 163 bit binary field
  sect193r1 : SECG curve over a 193 bit binary field
  sect193r2 : SECG curve over a 193 bit binary field
  sect233k1 : NIST/SECG/WTLS curve over a 233 bit binary field
  sect233r1 : NIST/SECG/WTLS curve over a 233 bit binary field
  sect239k1 : SECG curve over a 239 bit binary field
  sect283k1 : NIST/SECG curve over a 283 bit binary field
  sect283r1 : NIST/SECG curve over a 283 bit binary field
  sect409k1 : NIST/SECG curve over a 409 bit binary field
  sect409r1 : NIST/SECG curve over a 409 bit binary field
  sect571k1 : NIST/SECG curve over a 571 bit binary field
  sect571r1 : NIST/SECG curve over a 571 bit binary field
  c2pnb163v1: X9.62 curve over a 163 bit binary field
  c2pnb163v2: X9.62 curve over a 163 bit binary field
  c2pnb163v3: X9.62 curve over a 163 bit binary field
  c2pnb176v1: X9.62 curve over a 176 bit binary field
  c2tnb191v1: X9.62 curve over a 191 bit binary field
  c2tnb191v2: X9.62 curve over a 191 bit binary field
  c2tnb191v3: X9.62 curve over a 191 bit binary field
  c2pnb208w1: X9.62 curve over a 208 bit binary field
  c2tnb239v1: X9.62 curve over a 239 bit binary field
  c2tnb239v2: X9.62 curve over a 239 bit binary field
  c2tnb239v3: X9.62 curve over a 239 bit binary field
  c2pnb272w1: X9.62 curve over a 272 bit binary field
  c2pnb304w1: X9.62 curve over a 304 bit binary field
  c2tnb359v1: X9.62 curve over a 359 bit binary field
  c2pnb368w1: X9.62 curve over a 368 bit binary field
  c2tnb431r1: X9.62 curve over a 431 bit binary field
  wap-wsg-idm-ecid-wtls1: WTLS curve over a 113 bit binary field
  wap-wsg-idm-ecid-wtls3: NIST/SECG/WTLS curve over a 163 bit binary field
  wap-wsg-idm-ecid-wtls4: SECG curve over a 113 bit binary field
  wap-wsg-idm-ecid-wtls5: X9.62 curve over a 163 bit binary field
  wap-wsg-idm-ecid-wtls6: SECG/WTLS curve over a 112 bit prime field
  wap-wsg-idm-ecid-wtls7: SECG/WTLS curve over a 160 bit prime field
  wap-wsg-idm-ecid-wtls8: WTLS curve over a 112 bit prime field
  wap-wsg-idm-ecid-wtls9: WTLS curve over a 160 bit prime field
  wap-wsg-idm-ecid-wtls10: NIST/SECG/WTLS curve over a 233 bit binary field
  wap-wsg-idm-ecid-wtls11: NIST/SECG/WTLS curve over a 233 bit binary field
  wap-wsg-idm-ecid-wtls12: WTLS curve over a 224 bit prime field
  Oakley-EC2N-3: 
        IPSec/IKE/Oakley curve #3 over a 155 bit binary field.
        Not suitable for ECDSA.
        Questionable extension field!
  Oakley-EC2N-4: 
        IPSec/IKE/Oakley curve #4 over a 185 bit binary field.
        Not suitable for ECDSA.
        Questionable extension field!
  brainpoolP160r1: RFC 5639 curve over a 160 bit prime field
  brainpoolP160t1: RFC 5639 curve over a 160 bit prime field
  brainpoolP192r1: RFC 5639 curve over a 192 bit prime field
  brainpoolP192t1: RFC 5639 curve over a 192 bit prime field
  brainpoolP224r1: RFC 5639 curve over a 224 bit prime field
  brainpoolP224t1: RFC 5639 curve over a 224 bit prime field
  brainpoolP256r1: RFC 5639 curve over a 256 bit prime field
  brainpoolP256t1: RFC 5639 curve over a 256 bit prime field
  brainpoolP320r1: RFC 5639 curve over a 320 bit prime field
  brainpoolP320t1: RFC 5639 curve over a 320 bit prime field
  brainpoolP384r1: RFC 5639 curve over a 384 bit prime field
  brainpoolP384t1: RFC 5639 curve over a 384 bit prime field
  brainpoolP512r1: RFC 5639 curve over a 512 bit prime field
  brainpoolP512t1: RFC 5639 curve over a 512 bit prime field
  sm2p256v1 : SM2 curve over a 256 bit prime field
  wapip192v1: WAPI curve over a 192 bit prime field
  sm9bn256v1: SM9 BN curve over a 256 bit prime field
```
其中 sm2p256v1、wapip192v1、sm9bn256v1 就是国密定义的命名曲线。

#### 国密SM2算法

SM2算法就是一种ECC算法，准确来说，就是设计了一条ECC命名曲线。在《GMT 0003-2012》这份标准中，有SM2算法的设计背景知识，有兴趣的可以了解，对于开发者而言，最重要的是《GMT 0005-2012》标准中的曲线参数：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202103/images/practical_cryptography_for_developers_asymmetric_03.png)

现在的网络库，比如NSS、OpenSSL、libtomcrypt等，都有ECC算法的支持，要在网络库中加入SM2算法支持，只需加入命名曲线的参数即可。
比如在GmSSL代码的 ec_curve.c 文件中就有 sm2p256v1 命名曲线的参数定义：

```c
static const struct {
    EC_CURVE_DATA h;
    unsigned char data[0 + 32 * 6];
} _EC_SM2_PRIME_256V1 = {
    {
        NID_X9_62_prime_field, 0, 32, 1
    },
    {
        /* no seed */
        /* p */
        0xFF, 0xFF, 0xFF, 0xFE, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
        0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0x00, 0x00, 0x00, 0x00,
        0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
        /* a */
        0xFF, 0xFF, 0xFF, 0xFE, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
        0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0x00, 0x00, 0x00, 0x00,
        0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFC,
        /* b */
        0x28, 0xE9, 0xFA, 0x9E, 0x9D, 0x9F, 0x5E, 0x34, 0x4D, 0x5A, 0x9E, 0x4B,
        0xCF, 0x65, 0x09, 0xA7, 0xF3, 0x97, 0x89, 0xF5, 0x15, 0xAB, 0x8F, 0x92,
        0xDD, 0xBC, 0xBD, 0x41, 0x4D, 0x94, 0x0E, 0x93,
        /* x */
        0x32, 0xC4, 0xAE, 0x2C, 0x1F, 0x19, 0x81, 0x19, 0x5F, 0x99, 0x04, 0x46,
        0x6A, 0x39, 0xC9, 0x94, 0x8F, 0xE3, 0x0B, 0xBF, 0xF2, 0x66, 0x0B, 0xE1,
        0x71, 0x5A, 0x45, 0x89, 0x33, 0x4C, 0x74, 0xC7,
        /* y */
        0xBC, 0x37, 0x36, 0xA2, 0xF4, 0xF6, 0x77, 0x9C, 0x59, 0xBD, 0xCE, 0xE3,
        0x6B, 0x69, 0x21, 0x53, 0xD0, 0xA9, 0x87, 0x7C, 0xC6, 0x2A, 0x47, 0x40,
        0x02, 0xDF, 0x32, 0xE5, 0x21, 0x39, 0xF0, 0xA0,
        /* order */
        0xFF, 0xFF, 0xFF, 0xFE, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
        0xFF, 0xFF, 0xFF, 0xFF, 0x72, 0x03, 0xDF, 0x6B, 0x21, 0xC6, 0x05, 0x2B,
        0x53, 0xBB, 0xF4, 0x09, 0x39, 0xD5, 0x41, 0x23
    }
};
```

代码中的order就是参数表中的 n。对于大部分ECC操作来说，不需要该值，但在计算签名的时候会对n取模。

我们也可以通过GmSSL查看命名曲线的参数：

```bash
$ gmssl ecparam -name sm2p256v1 -out sm2p256v1.pem
$ gmssl ecparam -in sm2p256v1.pem -text -param_enc explicit -noout
Field Type: prime-field
Prime:
    00:ff:ff:ff:fe:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:00:00:00:00:ff:ff:ff:ff:ff:
    ff:ff:ff
A:   
    00:ff:ff:ff:fe:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:00:00:00:00:ff:ff:ff:ff:ff:
    ff:ff:fc
B:   
    28:e9:fa:9e:9d:9f:5e:34:4d:5a:9e:4b:cf:65:09:
    a7:f3:97:89:f5:15:ab:8f:92:dd:bc:bd:41:4d:94:
    0e:93
Generator (uncompressed):
    04:32:c4:ae:2c:1f:19:81:19:5f:99:04:46:6a:39:
    c9:94:8f:e3:0b:bf:f2:66:0b:e1:71:5a:45:89:33:
    4c:74:c7:bc:37:36:a2:f4:f6:77:9c:59:bd:ce:e3:
    6b:69:21:53:d0:a9:87:7c:c6:2a:47:40:02:df:32:
    e5:21:39:f0:a0
Order: 
    00:ff:ff:ff:fe:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:72:03:df:6b:21:c6:05:2b:53:bb:f4:09:39:
    d5:41:23
Cofactor:  1 (0x1)

```

#### 小结

本文从非对称密码算法开始，逐步介绍到国密SM2算法。我们可以看到，SM2并不是一个全新设计的算法，而是借助现有的ECC理论，设计了一条命名曲线。这样，在已经实现了ECC算法的网络库上增加SM2算法的支持就非常简单，只需要将曲线参数添加即可。

一套完整的公开密码算法，更重要的作用是用于密钥交换和数字签名，在后面的文章将继续探讨这些话题，敬请关注！

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)