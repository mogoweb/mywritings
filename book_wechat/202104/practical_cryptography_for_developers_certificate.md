# 写给开发人员的实用密码学 - 数字证书

在数字签名部分，我们讲到数字签名可以起到“防抵赖”的作用。然而，在开放的互联网环境中，通信的双方通常是互不相识，数字签名并不能解决身份认证的问题。比如在数字签名中，私钥签名，公钥验证签名。如果有人冒充淘宝给了你公钥，对方持有假冒公钥对应的私钥，这种情况下签名、验签都没问题，但你是在和一个假的淘宝通信。退一步说，你开始拿到的确实是淘宝发布的公钥，如果有人偷偷替换掉了你的机器上的公钥，这样你实际拥有的是李鬼的公钥，但是还以为这是淘宝的公钥。因此，李鬼就可以冒充淘宝，用自己的私钥做成"数字签名"，写信给你，而你则使用假的公钥进行解密。

在现实生活中，我们通常使用身份证或者护照来证明自己的身份，而在虚拟的网络世界，则需要使用数字证书。

很多人可能听说过 X.509 证书，它实际上就是一种数字证书。数字证书标准有很多，但使用最广泛的就是 X.509 标准，以至于现在一般将数字证书等同于X.509证书。

#### X.509标准

PKI（Public Key Infrastructure，称为公钥基础设施）是一个集合体，由一系列的软件、硬件、组织、个体、法律、流程组成，主要目的就是向客户端提供服务器身份认证。

为了规范化运用PKI技术，出现了很多标准，HTTPS中最常用的标准就是X.509标准。X.509标准来自国际电信联盟电信标准（ITU-T）的X.500标准，1995年国际互联网工程任务组（IETF）的PKIX小组成立，用来建设互联网的PKI公钥基础设施标准，建立的标准就是X.509。

互联网大部分应用（比如HTTPS协议、S/MIME邮件协议）使用的证书标准就是X.509标准，该标准可以参考RFC 5280文档。其他的组织也会基于X.509标准构建自己的PKI标准，比如IPsec使用自己的PKI标准，该标准定义在RFC 4945文档。

从中可以看出，PKI涉及的领域比较广泛，是一个相对松散的概念，一般来说，我们重点关注X.509的PKI标准即可。

* PKI的组成

根据PKI X.509标准，PKI组成下所示。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202103/images/practical_cryptography_for_developers_certificate_01.png)

（1）服务器实体（end entity），就是需要申请证书的实体，比如www.example.com域名的拥有者可以申请一张证书，证书能够证明www.example.com域名所有者的身份。

（2）CA机构，CA是证书签发机构，在审核服务器实体的有效身份后，给其签发证书，证书是用CA机构的密钥对（比如RSA密钥对）对服务器实体证书进行签名。

（3）RA机构，注册机构，主要审核服务器实体的身份，一般情况下，可以认为CA机构包含了RA机构。

（4）证书仓库，CA机构签发的证书全部保存在仓库中，证书也可能过期或者被吊销，CA机构吊销的证书称为证书吊销列表CRL（Certificate Revocation List）。

（5）证书校验方（relying party），校验证书真实性的软件，在Web领域，读者最熟悉的证书校验方就是浏览器。在本书中，浏览器、客户端、证书校验方可以认为是同一个概念。为了进行校验，证书校验方必须充分信任第三方CA机构，证书校验方集成了各个CA机构的根证书。

* X.509标准的内容

X.509标准包含的内容非常广泛，内容如下：

（1）证书的作用，第三方认证机构为服务器实体（end entity）签发证书，证书校验方可以使用证书对服务器实体的身份进行认证。

（2）证书文件的结构，证书是一个文件，理解证书的结构、属性、值非常重要。

（3）管理证书，服务器实体（end entity）向CA机构申请证书的流程，CA机构审核服务器实体身份的标准，签发证书的流程。

（4）校验证书，通过严谨的步骤校验证书，或者说校验服务器实体（end entity）身份，涉及两部分内容，一部分是证书签名校验，涉及证书链的概念。另外一部分是校验服务器实体属性，比如证书包含的域名、证书有效期等。

（5）证书的撤销问题，包括CRL和OCSP协议等概念。

#### 证书

证书是PKI中最核心的部分，理解了证书等同于理解了PKI的工作原理，证书中包含了很多信息，由签名、服务器实体（end entity）信息、CA机构信息三部分组成。

数字证书可以建立公钥与用户之间的对应关系。数字证书实际上是一种特殊的文件格式，包含用户身份信息、用户公钥和CA私钥的数字签名。数字证书中只包含公钥，并不包括私钥，可以公开。而数字证书中包含CA私钥的签名，所以具有防伪性。

证书是一个文件，用记事本打开，是一堆无意义的数据。理解证书内容必须先明白ASN.1（Abstract Syntax Notation One）的概念。

##### ASN.1

ASN.1是国际电信联盟电信标准（ITU-T）定义的标准，用来结构化描述证书。ASN.1类似于JSON或者XML这样的数据结构。ASN.1定义了复杂的数据结构，通常现有的加密库都包含了ASN.1的编码与解析，网上也可以找到源码，一般没必要完全理解ASN.1内部结构。我们可以将ASN.1看作一种伪代码，是用来描述证书结构的。

X.509是标准，ASN.1也是标准。X.509标准定义了证书应该包含的内容，而借助ASN.1标准来描述X.509标准（或者说证书），可以让机器和人更好地理解和组织X.509标准。

##### 证书结构

证书的主要结构如下：

```
 Certificate ::= SEQUENCE {
     tbsCertificate       TBSCertificate,
     signatureAlgorithm   AlgorithmIdentifier,
     signature            BIT STRING
 }
```

SEQUENCE是ASN.1中的一个结构体，相当于一个多维数组，数组由多个元素组成，每个元素有一个名称（比如tbsCertificate），名称有对应的属性（比如TBSCertificate），名称的值取决于属性。每个元素还可以嵌套其他的ASN.1结构，比如再嵌套一个SEQUENCE, TBSCertificate结构就是一个SEQUENCE结构。

CA机构对证书进行签名，为了让证书校验方（比如浏览器）进行校验，必须在证书中说明CA机构使用的签名算法。结构中的signatureAlgorithm代表的就是签名算法，signature就是签名值，tbsCertificate就是需要签名的内容，也是证书的核心，包括了服务器实体和CA机构的信息。

接下来看TBSCertificate结构，它是证书内容的核心部分。

```
 TBSCertificate ::= SEQUENCE {
     version             [0]  Version DEFAULT v1,
     serialNumber             CertificateSerialNumber,
     signature                AlgorithmIdentifier,
     issuer                   Name,
     validity                 Validty,
     subject                  Name,
     subjectPublicKeyInfo     SubjectPublicKeyInfo,
     issuerUniqueID      [1]  IMPLICIT UniqueIdentifier OPTIONAL,
                              -- If present, version MUST be v2 or v3
     subjectUniqueID     [2]  IMPLICIT UniqueIdentifier OPTIONAL,
                              -- If present, version MUST be v2 or v3
     extensions          [3]  Extensions OPTIONAL
                              -- If present, version MUST be v3
 }
```

1）version

version表示证书的版本号，目前有3个版本（v1, v2, v3），证书校验方（比如浏览器）根据版本进行校验，如果一个证书是v3版本，而证书校验方使用v1版本标准校验必然是错误的。version的类型是Version，结构定义如下：

```
 Version ::= INTEGER { v1(0), V2(1), V3(2) }
```

Version类型相当于一个枚举整型，有三个整数值可以选择。

2）serialNumber

每个证书都有唯一的编号，对于不同的CA机构来说，编号是无法预测的，CertificateSerialNumber是一个整型类型。

3）signature

证书是通过签名算法进行签名的，为了让证书校验方（比如浏览器）验证签名，必须告诉其签名算法，签名算法信息也在TBSCertificate结构体中。签名算法包含两个部分，分别是摘要算法和签名算法。对于ECDSAWithSHA256签名算法来说，ECDSA是签名算法，SHA256是摘要算法。接下来了解在ASN.1中是如何定义的AlgorithmIdentifier类型的。

签名算法标识符AlgorithmIdentifier类型本身也是一个SEQUENCE结构，由两个子元素构成：

```
 AlgorithmIdentifier ::= SEQUENCE {
     algorithm    OBJECT IDENTIFIER,
     parameters   ANY DEFINED BY algorithm OPTIONAL
 }
```

algorithm类型是OBJECT IDENTIFIER(OID)，在X.509中是非常普遍的一种数据结构，简单地理解就是一个字符数组，在《GM/T 0006-2012 密码应用标识规范》中就为国密规范定义了很多OID。

比如基于SM2算法和SM3算法的签名的OID为 1.2.156.10197.501

4）issuer

代表CA机构的名称，Issuer主要由国家（C）、组织（O）、子组织名（CN）组成

5）validity

CA机构是赢利的组织，证书使用期限越长价格越高，在证书中包括了证书的有效期，证书校验方需要校验证书有效期，如果证书有效期失效，表明证书不能代表服务器实体身份。

6）subject

代表服务器实体的名称，该组织向CA机构申请证书，其对应的Name类型和issuer的Name类型是一样的，CN表示服务器实体的域名（比如CN =www.example.com）。对于Web网站来说，每个网站都有一个域名，证书和域名息息相关，早期证书校验方校验证书的时候是将URL中的域名和证书subject值中的CN比较，如果一致，代表证书校验成功。随着时间的推移，一张证书可能包含多个域名，所以不再使用CN来校验证书域名了，而使用SAN证书扩展进行域名校验。

7）subjectPublicKeyInfo

服务器实体申请证书的时候，包含的一个重要属性就是服务器公钥，该公钥对应的算法就是公开密钥算法。

subjectPublicKeyInfo包含两部分信息，分别是公开密钥算法和公钥值，其对应的类型就是SubjectPublicKeyInfo类型：

```
 SubjectPublicKeyInfo ::= SEQUENCE {
     algorithm        AlgorithmIdentifier,
     subjectPublicKey BIT STRING
 }
```

8）issuerUniqueID和subjectUniqueID

这两个分别代表CA机构和服务器实体的唯一编号，目前已经被相应的证书扩展替代。

9）extension

扩展是X.509 V3版本引入的，主要是为了扩展证书的含义，在不改变X.509版本的情况下，可以相对方便地增加证书新属性，新添加的扩展是否生效取决于证书校验方。

通过证书扩展，CA机构和证书校验方可以在不修改（或者较少修改）代码的前提下使用该扩展，前提是双方都认这个扩展。X.509 V3定义了14个扩展，如果需要额外添加扩展，就需要双方都支持。

##### CSR

CSR是Certificate Signing Request的缩写，服务器实体为了证明自己的身份，需要向CA机构申请证书，在申请证书之前，必须先生成一个CSR文件，然后将CSR文件发送给CA机构。

CSR文件包括两部分内容：

* 生成证书必需的信息，比如域名、公钥。
* 服务器实体的证明材料，比如企业的纳税编号等信息。

CSR文件采用的标准是PKCS#10，请参考RFC2986。CSR用ASN.1标准描述，整体结构如下：

```
CertificationRequest ::= SEQUENCE {
    certificationRequestInfo   CertificationRequestInfo,
    signatureAlgorithm         AlgorithmIdentifier,
    signature                  BIT STRING
}
```

该结构包含两部分：

* certificationRequestInfo结构表示证书的请求信息。
* 签名信息，对certificationRequestInfo的签名。

CertificationRequestInfo的信息需要重点关注，其结构如下：

```
CertificationRequestInfo ::= SEQUENCE {
    version           INTEGER { v1(0) } (v1, ...),
    subject           Name,
    subjectPKInfo     SubjectPublicKeyInfo,
    attributes    [0] Attributes
}
```

* version表示PKCS#10标准的版本号。
* subject表示服务器主体的可分辨名称DN，最重要的是CN属性值，表示证书需要包含的域名，可以包含多个。
* subjectPKInfo表示服务器密钥对的公钥，可以是RSA公钥或国密SM2公钥。
* attributes表示可选信息，比如服务器主体的邮件地址等相关信息。

在证书中，服务器公钥、域名是服务器主体最重要的内容。服务器主体使用该密钥对的私钥对certificationRequestInfo进行数字签名。

##### 证书生成格式

ASN.1标准用于描述证书结构，而证书本质上是一个文件，需要一种专门的格式，才能在互联网中传输，证书需要通过一个规则将ASN.1转换为二进制文件。在X.509证书中，使用的编码方式是Distinguished Encoding Rules（DER），ASN.1和DER的关系类似于字符集和编码的关系。

DER是一个二进制文件，为了方便传输，可以将DER转换为PEM（Privacy-enhanced Electronic Mail）格式，PEM是Base64编码方式，以-----BEGINCERTIFICATE-----开头、-----END CERTIFICATE-----结尾。

还有一些其它不常用的格式：

* BER。Basic Encoding Rules（BER）是DER的一个子集，
* CER。Canonical Encoding Rules（CER）是另外一种编码标准，用来编码ASN.1结构。
* PKCS#12格式。微软发布的一种格式，文件后缀一般是．pkcs12、.pfx、.p12。
* PKCS#7格式。证书的另外一种格式，主要用来进行数字签名和数据加密，文件后缀一般是．p7b或者．p7c

##### 证书生成过程

一般生成证书的流程为：

* 用户生成一对密钥对，比如SM2密钥对。
* 生成CSR文件，包含域名、公钥、签名等，发送给CA机构。
* CA机构根据CSR文件生成证书，然后将证书下发给用户。

下面制作一张自签名证书为例，说明证书的制作过程。

1. 生成SM2私钥

```
$ gmssl ecparam -genkey -name sm2p256v1 -text -out sm2_user.key
```

ecparam 表示采用椭圆曲线算法 -name sm2p256v1 指定椭圆曲线名称，这里指定了国密定义的椭圆曲线。

最后SM2私钥保存到 sm2_user.key

```
ASN1 OID: sm2p256v1
NIST CURVE: SM2
-----BEGIN EC PARAMETERS-----
BggqgRzPVQGCLQ==
-----END EC PARAMETERS-----
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIONlieRy6aZ0q9o1z7LRii5HScKYIvLVcgaeM0x+FtyXoAoGCCqBHM9V
AYItoUQDQgAECvItoLwmG+Ws1GvLXsJbM+DgRbdY37MRloE6WDn6X/S3l5lKHIAQ
6ZH0QMSqQwg/LdvoI3s80/CMZJHnWqn6pA==
-----END EC PRIVATE KEY-----
```

注意：SM2私钥本质上是一串随机数，这里还包含了椭圆曲线的一些参数，并进行了PEM编码。

2. 创建证书请求：

```
$ gmssl req -new -key sm2_user.key -out sm2_user.req
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [CN]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

这里会生成一个CSR，并保存到一个文件中。前面讲过，需要对CertificationRequestInfo进行数字签名，所以命令行需要传入SM2私钥。

可以看到，生成CSR时还要提供一些信息，比如国家、城市、公司名之类的信息，以后查看证书信息时，显示的就是这些信息。这里只是开发测试，所以填写什么内容无关紧要，直接使用默认值。

最后生成的CSR文件如下：

```
-----BEGIN CERTIFICATE REQUEST-----
MIIBATCBpwIBADBFMQswCQYDVQQGEwJDTjETMBEGA1UECAwKU29tZS1TdGF0ZTEh
MB8GA1UECgwYSW50ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMFkwEwYHKoZIzj0CAQYI
KoEcz1UBgi0DQgAECvItoLwmG+Ws1GvLXsJbM+DgRbdY37MRloE6WDn6X/S3l5lK
HIAQ6ZH0QMSqQwg/LdvoI3s80/CMZJHnWqn6pKAAMAoGCCqBHM9VAYN1A0kAMEYC
IQDlPL8JQRMBEM89hOMd2gOSG7bZZjI/bKnub1YO0wrEVAIhAOIR1GsrCG8e1MzU
NQpsv0wKypISh1wdmN86kHzQE/20
-----END CERTIFICATE REQUEST-----
```

3. 生成证书：

```
$ gmssl x509 -req -days 365 -in sm2_user.req -signkey sm2_user.key -out sm2_user_cert.pem
Signature ok
subject=C = CN, ST = Some-State, O = Internet Widgits Pty Ltd
Getting Private key
```

参数中 -days 365 指的是证书的有效期为365天。生成的证书文件如下：

```
-----BEGIN CERTIFICATE-----
MIIBejCCASACCQDzHKj1UYRCVjAKBggqgRzPVQGDdTBFMQswCQYDVQQGEwJDTjET
MBEGA1UECAwKU29tZS1TdGF0ZTEhMB8GA1UECgwYSW50ZXJuZXQgV2lkZ2l0cyBQ
dHkgTHRkMB4XDTIxMDMzMTA3Mjk0NloXDTIyMDMzMTA3Mjk0NlowRTELMAkGA1UE
BhMCQ04xEzARBgNVBAgMClNvbWUtU3RhdGUxITAfBgNVBAoMGEludGVybmV0IFdp
ZGdpdHMgUHR5IEx0ZDBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABAryLaC8Jhvl
rNRry17CWzPg4EW3WN+zEZaBOlg5+l/0t5eZShyAEOmR9EDEqkMIPy3b6CN7PNPw
jGSR51qp+qQwCgYIKoEcz1UBg3UDSAAwRQIhAJRL6l3CeSyoUrWbqFuGqKrIyIjr
5zbtoBM4/26jOPb+AiBohiMRwPnAqIkxG8PS3FFrtsZH11uBZ3vrq3ozhV2G7A==
-----END CERTIFICATE-----
```

因为进行了PEM编码，看不出里面的内容，我们可以借助工具查看证书内容。

```
$ gmssl x509 -text -in sm2_user_cert.pem -noout
Certificate:
   Data:
       Version: 1 (0x0)
       Serial Number:
           f3:1c:a8:f5:51:84:42:56
   Signature Algorithm: sm2sign-with-sm3
       Issuer: C = CN, ST = Some-State, O = Internet Widgits Pty Ltd
       Validity
           Not Before: Mar 31 07:29:46 2021 GMT
           Not After : Mar 31 07:29:46 2022 GMT
       Subject: C = CN, ST = Some-State, O = Internet Widgits Pty Ltd
       Subject Public Key Info:
           Public Key Algorithm: id-ecPublicKey
               Public-Key: (256 bit)
               pub:
                   04:0a:f2:2d:a0:bc:26:1b:e5:ac:d4:6b:cb:5e:c2:
                   5b:33:e0:e0:45:b7:58:df:b3:11:96:81:3a:58:39:
                   fa:5f:f4:b7:97:99:4a:1c:80:10:e9:91:f4:40:c4:
                   aa:43:08:3f:2d:db:e8:23:7b:3c:d3:f0:8c:64:91:
                   e7:5a:a9:fa:a4
               ASN1 OID: sm2p256v1
               NIST CURVE: SM2
   Signature Algorithm: sm2sign-with-sm3
        30:45:02:21:00:94:4b:ea:5d:c2:79:2c:a8:52:b5:9b:a8:5b:
        86:a8:aa:c8:c8:88:eb:e7:36:ed:a0:13:38:ff:6e:a3:38:f6:
        fe:02:20:68:86:23:11:c0:f9:c0:a8:89:31:1b:c3:d2:dc:51:
        6b:b6:c6:47:d7:5b:81:67:7b:eb:ab:7a:33:85:5d:86:ec
```

可以对照着前面的证书结构，看看上面的证书内容。

发一个数字证书这么容易吗？对的，借助工具任何人都可以很容易颁发一个数字证书，至于这个证书别人是不是承认，这个就不是证书本身能解决的了。。。这就如同现实世界中，你也可以给自己颁发一个国际程序员大师，你也可以盖上自己的印，表明此证书确实是你颁发的。剩下的问题就是如何让这个证书有价值了。。。

要让用户信任颁发的数字证书，这里就需要引入 CA 中心了。

在下一篇文章中，我将详细阐述 CA 中心，敬请关注。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)