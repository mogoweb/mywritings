## 证书

### PKI 和 X.509

PKI（Public Key Infrastructure，称为公钥基础设施）是一个集合体，由一系列的软件、硬件、组织、个体、法律、流程组成，主要目的就是向客户端提供服务器身份认证，认证的基础就是必须找到一个可信的第三方组织，认证的技术方案就是数字签名技术。第三方组织能够使用数字签名技术管理证书，包括创建证书、存储证书、更新证书、撤销证书。

为了规范化运用PKI技术，出现了很多标准，HTTPS中最常用的标准就是X.509标准，证书是PKI最核心、最重要的内容，提到证书也可以认为是X.509标准证书。

参考标准文档：RFC 5280

![PKI结构图](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_work/gmbrowser/images/cert_01.png)

### 证书

证书中包含了很多信息，由签名、服务器实体（end entity）信息、CA机构信息三部分组成。

#### ASN.1

证书是一个文件，用记事本打开，是一堆无意义的数据。理解证书内容必须先明白ASN.1（Abstract Syntax Notation One）的概念，ASN.1是国际电信联盟电信标准（ITU-T）定义的标准，用来结构化描述证书，ASN.1类似于JSON或者XML这样的数据结构。

证书的主要结构如下：

![ASN.1证书结构]](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_work/gmbrowser/images/cert_02.png)

CA机构对证书进行签名，为了让证书校验方（浏览器）进行校验，必须在证书中说明CA机构使用的签名算法，ASN.1中的signatureAlgorithm代表的就是签名算法，signature就是签名值，tbsCertificate就是需要签名的内容，也是证书的核心，包括了服务器实体和CA机构的信息。

```
TBSCertificate  ::=  SEQUENCE  {
    version         [0]  EXPLICIT Version DEFAULT v1,
    serialNumber         CertificateSerialNumber,
    signature            AlgorithmIdentifier,
    issuer               Name,
    validity             Validity,
    subject              Name,
    subjectPublicKeyInfo SubjectPublicKeyInfo,
    issuerUniqueID  [1]  IMPLICIT UniqueIdentifier OPTIONAL,
                         -- If present, version MUST be v2 or v3
                          subjectUniqueID [2]  IMPLICIT UniqueIdentifier OPTIONAL,
                      -- If present, version MUST be v2 or v3
     extensions      [3]  EXPLICIT Extensions OPTIONAL
                      -- If present, version MUST be v3
}
```

其中：

签名算法标识符AlgorithmIdentifier类型本身也是一个SEQUENCE结构，由两个子元素构成：

```
AlgorithmIdentifier ::= SEQUENCE {
    algorithm       OBJECT IDENTIFIER,
    parameters      ANY DEFINED BY algorithm OPTIONAL
}
```

algorithm类型是OBJECT IDENTIFIER(OID)，这个结构有点复杂，简单地理解就是一个字符数组，每个字符或者每组字符代表一种含义。在 NSS 的 secoid.c 文件中定义了所有的OID。

subjectPublicKeyInfo 包含两部分信息，分别是公开密钥算法和公钥值，其对应的类型就是SubjectPublicKeyInfo类型：

```
SubjectPublicKeyInfo  ::=  SEQUENCE  {
        algorithm            AlgorithmIdentifier,
        subjectPublicKey     BIT STRING  }
```

subjectPublicKey的类型是BIT STRING，代表公钥具体的值，它的值取决于algorithm。

#### CSR（Certificate Signing Request）

服务器实体为了证明自己的身份，需要向CA机构申请证书，在申请证书之前，必须先生成一个CSR文件，然后将CSR文件发送给CA机构。CSR文件包括两部分内容：

* 生成证书必需的信息，比如域名、公钥。
* 服务器实体的证明材料，比如企业的纳税编号等信息（可以简单地如此理解）。

CSR文件采用的标准是PKCS#10，该标准定义在RFC 2986文档中。

CSR文件生成过程：

* 服务器主体生成一对密钥对，比如RSA密钥对。
* 生成CertificationRequestInfo结构体，主要包含域名、公钥。
* 使用私钥对CertificationRequestInfo进行数字签名得到签名值。
* 组合CertificationRequestInfo信息和签名信息最终得到CSR文件，发送给CA机构。

OpenSSL库会集成根证书:

```
$ openssl version -a
GmSSL 2.5.4 - OpenSSL 1.1.0d  3 Sep 2019
built on: reproducible build, date unspecified
platform: linux-x86_64
compiler: gcc -DDSO_DLFCN -DHAVE_DLFCN_H -DNDEBUG -DOPENSSL_THREADS -DOPENSSL_NO_STATIC_ENGINE -DOPENSSL_PIC -DOPENSSL_IA32_SSE2 -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_MONT5 -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DRC4_ASM -DMD5_ASM -DAES_ASM -DVPAES_ASM -DBSAES_ASM -DGHASH_ASM -DECP_NISTZ256_ASM -DPADLOCK_ASM -DGMI_ASM -DPOLY1305_ASM -DOPENSSLDIR="\"/home/alex/work/gmbrowser/usr/local/gmssl/ssl\"" -DENGINESDIR="\"/home/alex/work/gmbrowser/usr/local/gmssl/lib/engines-1.1\""  -Wa,--noexecstack
OPENSSLDIR: "/home/alex/work/gmbrowser/usr/local/gmssl/ssl"
ENGINESDIR: "/home/alex/work/gmbrowser/usr/local/gmssl/lib/engines-1.1"
```

* 系统根证书库目录是 /home/alex/work/gmbrowser/usr/local/gmssl/ssl 。
* /home/alex/work/gmbrowser/usr/local/gmssl/ssl/certs下所有．pem结尾的文件就是各个CA机构的根证书。
* /home/alex/work/gmbrowser/usr/local/gmssl/ssl/certs/ca-certificates.crt文件是所有根证书的集合，包含了各个CA机构的根证书。

Mozilla的NSS底层密码库独立维护了可信任的根证书库，不仅仅是其自家的软件，其他很多系统都会使用Mozilla的根证书库。在Windows、Linux操作系统中，Firefox使用的都是Mozilla根证书库。Mozilla根证书库可以在线获取，其他的操作系统和应用软件可以使用它的根证书，比如：

* Linux桌面版的Chrome就使用Mozilla的根证书库。
* Linux操作系统也可以使用Mozilla根证书库，从而替代OpenSSL库默认的根证书库。

#### 证书格式

证书需要通过一个规则将ASN.1转换为二进制文件。在X.509证书中，使用的编码方式是Distinguished Encoding Rules（DER），ASN.1和DER的关系类似于字符集和编码的关系。

DER是一个二进制文件，为了方便传输，可以将DER转换为PEM（Privacy-enhanced ElectronicMail）格式，PEM是Base64编码方式，以-----BEGIN CERTIFICATE-----开头、-----ENDCERTIFICATE-----结尾。

#### 获取线上证书

1. 使用 openssl s_client.

```
$ openssl s_client -connect sm2test.ovssl.cn:443 -showcerts 2>&1 </dev/null
CONNECTED(00000003)
depth=2 C = CN, O = \E6\B2\83\E9\80\9A\E7\94\B5\E5\AD\90\E8\AE\A4\E8\AF\81\E6\9C\8D\E5\8A\A1\E6\9C\89\E9\99\90\E5\85\AC\E5\8F\B8, CN = \E5\9B\BD\E5\AF\86SM2\E6\A0\B9\E8\AF\81\E4\B9\A6
verify error:num=19:self signed certificate in certificate chain
Z=78FD3404CA9C610C29C8015630731A4C3666BD6B98CA2C9B3E1CC2BACD43073D
C=00037B308203773082031DA0030201020210192994D95A2E21599982DFC071EB4EDC300A06082A811CCF550183753064310B300906035504061302434E312D302B060355040A0C24E6B283E9809AE794B5E5AD90E8AEA4E8AF81E69C8DE58AA1E69C89E99990E585ACE58FB83126302406035504030C1DE59BBDE5AF86534D32E69C8DE58AA1E599A8E6A0B9E8AF81E4B9A65633301E170D3139303431323035333032385A170D3231303431323033333935335A307F310B300906035504061302434E3112301006035504080C09E5B9BFE4B89CE79C813112301006035504070C09E6B7B1E59CB3E5B882312D302B060355040A0C24E6B283E9809AE794B5E5AD90E8AEA4E8AF81E69C8DE58AA1E69C89E99990E585ACE58FB83119301706035504030C10736D326F6E6C792E6F7673736C2E636E3059301306072A8648CE3D020106082A811CCF5501822D03420004C6CCB0CB4F8C2000040F0CBACC3A40D0392D92F76797DACA496190485B6E7185B6B661C3A1718EDB3BBB0825B7FA7344C80525D278D461A34F5AE25C369685BFA382019430820190300E0603551D0F0101FF040403020520301D0603551D250416301406082B0601050507030106082B0601050507030230090603551D1304023000301D0603551D0E0416041400679B96D28C9DD2147703486AAF9097D8634BE5301F0603551D230418301680148E8668B92CD8AECB2026F65FB63B5A5EEA9B9966306506082B0601050507010104593057302206082B060105050730018616687474703A2F2F6F6373702E776F747275732E636F6D303106082B060105050730028625687474703A2F2F6169612E776F747275732E636F6D2F77732D736D322D73736C332E63657230360603551D1F042F302D302BA029A0278625687474703A2F2F63726C2E776F747275732E636F6D2F77732D736D322D73736C332E63726C301B0603551D11041430128210736D326F6E6C792E6F7673736C2E636E30580603551D200451304F3008060667810C010202300906072A811C89982A0D303806092A811C89982A030103302B302906082B06010505070201161D687474703A2F2F7777772E776F747275732E636F6D2F706F6C6963792F300A06082A811CCF550183750348003045022009B63060CC9E11C121C5DAA958332FBA8CC2D3F00C52C7A0B8AB5DB078A035700221009AC324BDFF53B77CD8D1CFE1428D6422F34D1AD96ECF93E1BDAD037306
crypto/sm2/sm2_sign.c 504: sm2_do_verify
140186210387712:error:141C907B:SSL routines:gmtls_process_ske_sm2:bad signature:ssl/statem/statem_gmtls.c:895:
write:errno=0
---
Certificate chain
 0 s:/C=CN/ST=\xE5\xB9\xBF\xE4\xB8\x9C\xE7\x9C\x81/L=\xE6\xB7\xB1\xE5\x9C\xB3\xE5\xB8\x82/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=sm2only.ovssl.cn
   i:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6V3
-----BEGIN CERTIFICATE-----
MIIDeDCCAx2gAwIBAgIQS3P355MqTNQ4t0uA8LxLATAKBggqgRzPVQGDdTBkMQsw
CQYDVQQGEwJDTjEtMCsGA1UECgwk5rKD6YCa55S15a2Q6K6k6K+B5pyN5Yqh5pyJ
6ZmQ5YWs5Y+4MSYwJAYDVQQDDB3lm73lr4ZTTTLmnI3liqHlmajmoLnor4HkuaZW
MzAeFw0xOTA0MTIwNTMwNDFaFw0yMTA0MTIwMzM5NTNaMH8xCzAJBgNVBAYTAkNO
MRIwEAYDVQQIDAnlub/kuJznnIExEjAQBgNVBAcMCea3seWcs+W4gjEtMCsGA1UE
Cgwk5rKD6YCa55S15a2Q6K6k6K+B5pyN5Yqh5pyJ6ZmQ5YWs5Y+4MRkwFwYDVQQD
DBBzbTJvbmx5Lm92c3NsLmNuMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAE81hb
wnHKCcykAjjRbTsy3/TjnoN2kCRGhrnd7/MmeauNo+j5CKQ3+6KDgnVvIQXp0ToG
o00i+YPsJA94Dk/wXaOCAZQwggGQMA4GA1UdDwEB/wQEAwIHgDAdBgNVHSUEFjAU
BggrBgEFBQcDAQYIKwYBBQUHAwIwCQYDVR0TBAIwADAdBgNVHQ4EFgQUre/PXukq
xiNWNHEeg84d8PWqVIEwHwYDVR0jBBgwFoAUjoZouSzYrssgJvZftjtaXuqbmWYw
ZQYIKwYBBQUHAQEEWTBXMCIGCCsGAQUFBzABhhZodHRwOi8vb2NzcC53b3RydXMu
Y29tMDEGCCsGAQUFBzAChiVodHRwOi8vYWlhLndvdHJ1cy5jb20vd3Mtc20yLXNz
bDMuY2VyMDYGA1UdHwQvMC0wK6ApoCeGJWh0dHA6Ly9jcmwud290cnVzLmNvbS93
cy1zbTItc3NsMy5jcmwwGwYDVR0RBBQwEoIQc20yb25seS5vdnNzbC5jbjBYBgNV
HSAEUTBPMAgGBmeBDAECAjAJBgcqgRyJmCoNMDgGCSqBHImYKgMBAzArMCkGCCsG
AQUFBwIBFh1odHRwOi8vd3d3LndvdHJ1cy5jb20vcG9saWN5LzAKBggqgRzPVQGD
dQNJADBGAiEA/tXQiSwpXHG0wRmqawoYl5CB29jtPN1En06PG4nT97MCIQCYompX
lKzC7EbeZVPd7GH4IpQP0Vn6+dxWgKFlGp/NIg==
-----END CERTIFICATE-----
 1 s:/C=CN/ST=\xE5\xB9\xBF\xE4\xB8\x9C\xE7\x9C\x81/L=\xE6\xB7\xB1\xE5\x9C\xB3\xE5\xB8\x82/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=sm2only.ovssl.cn
   i:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6V3
-----BEGIN CERTIFICATE-----
MIIDdzCCAx2gAwIBAgIQGSmU2VouIVmZgt/AcetO3DAKBggqgRzPVQGDdTBkMQsw
CQYDVQQGEwJDTjEtMCsGA1UECgwk5rKD6YCa55S15a2Q6K6k6K+B5pyN5Yqh5pyJ
6ZmQ5YWs5Y+4MSYwJAYDVQQDDB3lm73lr4ZTTTLmnI3liqHlmajmoLnor4HkuaZW
MzAeFw0xOTA0MTIwNTMwMjhaFw0yMTA0MTIwMzM5NTNaMH8xCzAJBgNVBAYTAkNO
MRIwEAYDVQQIDAnlub/kuJznnIExEjAQBgNVBAcMCea3seWcs+W4gjEtMCsGA1UE
Cgwk5rKD6YCa55S15a2Q6K6k6K+B5pyN5Yqh5pyJ6ZmQ5YWs5Y+4MRkwFwYDVQQD
DBBzbTJvbmx5Lm92c3NsLmNuMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAExsyw
y0+MIAAEDwy6zDpA0Dktkvdnl9rKSWGQSFtucYW2tmHDoXGO2zu7CCW3+nNEyAUl
0njUYaNPWuJcNpaFv6OCAZQwggGQMA4GA1UdDwEB/wQEAwIFIDAdBgNVHSUEFjAU
BggrBgEFBQcDAQYIKwYBBQUHAwIwCQYDVR0TBAIwADAdBgNVHQ4EFgQUAGebltKM
ndIUdwNIaq+Ql9hjS+UwHwYDVR0jBBgwFoAUjoZouSzYrssgJvZftjtaXuqbmWYw
ZQYIKwYBBQUHAQEEWTBXMCIGCCsGAQUFBzABhhZodHRwOi8vb2NzcC53b3RydXMu
Y29tMDEGCCsGAQUFBzAChiVodHRwOi8vYWlhLndvdHJ1cy5jb20vd3Mtc20yLXNz
bDMuY2VyMDYGA1UdHwQvMC0wK6ApoCeGJWh0dHA6Ly9jcmwud290cnVzLmNvbS93
cy1zbTItc3NsMy5jcmwwGwYDVR0RBBQwEoIQc20yb25seS5vdnNzbC5jbjBYBgNV
HSAEUTBPMAgGBmeBDAECAjAJBgcqgRyJmCoNMDgGCSqBHImYKgMBAzArMCkGCCsG
AQUFBwIBFh1odHRwOi8vd3d3LndvdHJ1cy5jb20vcG9saWN5LzAKBggqgRzPVQGD
dQNIADBFAiAJtjBgzJ4RwSHF2qlYMy+6jMLT8AxSx6C4q12weKA1cAIhAJrDJL3/
U7d82NHP4UKNZCLzTRrZbs+T4b2tA3MGXFWd
-----END CERTIFICATE-----
 2 s:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6V3
   i:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6
-----BEGIN CERTIFICATE-----
MIIDJjCCAsugAwIBAgIQC4LrMsbMVTwXKaCKgOkwlDAKBggqgRzPVQGDdTBZMQsw
CQYDVQQGEwJDTjEtMCsGA1UECgwk5rKD6YCa55S15a2Q6K6k6K+B5pyN5Yqh5pyJ
6ZmQ5YWs5Y+4MRswGQYDVQQDDBLlm73lr4ZTTTLmoLnor4HkuaYwHhcNMTkwNDA0
MDYxODE4WhcNMzQwNDA0MDYxODE4WjBkMQswCQYDVQQGEwJDTjEtMCsGA1UECgwk
5rKD6YCa55S15a2Q6K6k6K+B5pyN5Yqh5pyJ6ZmQ5YWs5Y+4MSYwJAYDVQQDDB3l
m73lr4ZTTTLmnI3liqHlmajmoLnor4HkuaZWMzBZMBMGByqGSM49AgEGCCqBHM9V
AYItA0IABO+ktsEtsJ81WrKNhIaQKfQC6yVpAbHAZKbpzBAb8CQMFp1hvUsGnn16
p449RAYmckmJ1JeGR0bFsVwoHC8HS5+jggFoMIIBZDAfBgNVHSMEGDAWgBQxuBWH
TMw3lzrt702RSiutCzV2IDAdBgNVHQ4EFgQUjoZouSzYrssgJvZftjtaXuqbmWYw
NQYDVR0fBC4wLDAqoCigJoYkaHR0cDovL2NybC53b3RydXMuY29tL3dvdHJ1cy1z
bTIuY3JsMGQGCCsGAQUFBwEBBFgwVjAiBggrBgEFBQcwAYYWaHR0cDovL29jc3Au
d290cnVzLmNvbTAwBggrBgEFBQcwAoYkaHR0cDovL2FpYS53b3RydXMuY29tL3dv
dHJ1cy1zbTIuY2VyMEIGA1UdIAQ7MDkwNwYIKoEciZgqAwEwKzApBggrBgEFBQcC
ARYdaHR0cDovL3d3dy53b3RydXMuY29tL3BvbGljeS8wEgYDVR0TAQH/BAgwBgEB
/wIBADAOBgNVHQ8BAf8EBAMCAQYwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUF
BwMCMAoGCCqBHM9VAYN1A0kAMEYCIQC/DGPAPQeARs/+TxvMfTNiTFjgvKV21drw
zdx7AiqdWAIhAI4YaGSoqdVclOIYMzGvbL92pI4hhS85BSRi0XZYQrzu
-----END CERTIFICATE-----
 3 s:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6
   i:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6
-----BEGIN CERTIFICATE-----
MIIB9DCCAZmgAwIBAgIRAIHDVL1guSy7XoFVo0I4+tQwCgYIKoEcz1UBg3UwWTEL
MAkGA1UEBhMCQ04xLTArBgNVBAoMJOayg+mAmueUteWtkOiupOivgeacjeWKoeac
iemZkOWFrOWPuDEbMBkGA1UEAwwS5Zu95a+GU00y5qC56K+B5LmmMB4XDTE5MDQw
NDA2MTYxNloXDTQ0MDQwNDA2MTYxNlowWTELMAkGA1UEBhMCQ04xLTArBgNVBAoM
JOayg+mAmueUteWtkOiupOivgeacjeWKoeaciemZkOWFrOWPuDEbMBkGA1UEAwwS
5Zu95a+GU00y5qC56K+B5LmmMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAEG2iw
QWmdsI7+MzAWrLdrH4nEzb4q2Ujy/QbGrlque74ieYFEI5xV1J8/RqPH1n9D8BBz
zeYuv9mHJpYgB6jXoqNCMEAwHQYDVR0OBBYEFDG4FYdMzDeXOu3vTZFKK60LNXYg
MA8GA1UdEwEB/wQFMAMBAf8wDgYDVR0PAQH/BAQDAgEGMAoGCCqBHM9VAYN1A0kA
MEYCIQCsobYw4P1XbSuLxvoR3RThjv+OLUHdoGdxTtZdv6jayAIhALBB8scNmtNg
yEUt1z54gasQb8OiCZEPIdRZBDR2GI7A
-----END CERTIFICATE-----
---
Server certificate
subject=/C=CN/ST=\xE5\xB9\xBF\xE4\xB8\x9C\xE7\x9C\x81/L=\xE6\xB7\xB1\xE5\x9C\xB3\xE5\xB8\x82/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=sm2only.ovssl.cn
issuer=/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6V3
---
No client certificate CA names sent
---
SSL handshake has read 3261 bytes and written 203 bytes
Verification error: self signed certificate in certificate chain
---
New, (NONE), Cipher is (NONE)
Server public key is 256 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : GMTLSv1.1
    Cipher    : 0000
    Session-ID: 
    Session-ID-ctx: 
    Master-Key: 
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1582784227
    Timeout   : 7200 (sec)
    Verify return code: 19 (self signed certificate in certificate chain)
    Extended master secret: no
---
```

#### 生成自签名证书

1. 生成 CSR 文件

```
openssl req -new -sha256 -newkey rsa:2048 -nodes -subj '/CN=www.example.com/O=Test, Inc./C=CN/ST=Beijing/L=Haidian' -keyout example_key.pem -out example_csr.pem
```

可以使用以下命令查看 CSR 文件内容：

```
$ openssl req -in example_csr.pem -noout -text
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: CN = www.example.com, O = "Test, Inc.", C = CN, ST = Beijing, L = Haidian
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d0:d8:a4:d3:48:bc:22:58:c0:25:54:fb:26:fe:
                    33:60:0b:74:12:67:97:ab:9d:62:6a:fd:e6:60:e8:
                    03:5d:c9:8a:d9:0c:9a:13:32:e6:f3:de:32:2f:eb:
                    5e:ae:dc:f9:17:de:8d:32:28:65:63:13:ce:41:d2:
                    ef:0c:2a:c6:bf:87:68:66:90:4e:8a:2f:ed:6f:44:
                    84:15:fe:d8:bf:41:96:7c:22:29:63:26:63:65:d2:
                    9c:46:7b:e0:dc:98:80:4a:96:95:c0:b3:8a:f5:be:
                    27:4a:28:47:05:0a:74:58:96:a3:70:fd:52:7f:3c:
                    5a:20:52:e0:d4:56:d0:84:97:d8:48:0b:72:fa:fd:
                    36:b8:93:01:12:e0:29:06:9f:96:3b:78:f3:e4:ef:
                    91:f2:e2:94:25:49:bd:6d:97:4e:67:66:b0:9a:8d:
                    fc:87:2f:df:3f:46:1a:cd:76:15:b7:45:5c:96:06:
                    e5:92:0a:dc:e5:0b:82:a2:ca:cc:a8:80:b6:70:52:
                    a0:36:79:08:d3:87:dd:59:17:8c:4f:4a:31:8f:c5:
                    5c:12:87:8d:af:56:f5:fc:1a:83:2a:97:67:83:2e:
                    50:a4:c7:f9:a0:ad:1f:b7:a2:70:9b:19:d1:f8:52:
                    68:b4:af:4c:cf:9f:e7:7a:cf:58:f9:32:1e:9b:0e:
                    03:27
                Exponent: 65537 (0x10001)
        Attributes:
            a0:00
    Signature Algorithm: sha256WithRSAEncryption
         16:1e:6e:a5:dc:46:8c:bd:57:cf:ea:52:40:d6:48:38:32:18:
         05:04:78:38:e5:ef:73:bb:a3:25:4c:bb:7f:0b:96:2e:b9:d1:
         05:f4:f0:c1:85:59:1e:40:c9:3b:30:f2:37:85:57:24:c3:ab:
         6c:37:e5:8c:5c:5f:44:5d:49:74:6d:c1:ef:34:eb:fd:50:7e:
         9b:cf:aa:da:5e:8f:3f:22:ff:6d:f4:3d:52:f2:d8:7a:59:11:
         fe:4a:b2:7e:2b:cd:a3:14:4f:89:91:6d:58:ae:6a:f2:da:d6:
         80:1f:e4:e9:68:8e:6d:92:2b:ad:92:b2:4a:50:fe:cf:3e:20:
         6f:44:38:0e:0c:c9:4d:71:89:bf:95:21:13:80:e8:37:31:bd:
         b1:91:90:4a:6c:91:48:f6:77:7f:45:fa:33:d6:94:89:3c:30:
         29:3d:36:97:35:a2:ff:01:fa:2d:55:de:eb:a9:0b:ec:bc:11:
         64:bd:ed:76:cc:6d:01:19:5f:83:2a:bd:51:96:9d:af:60:11:
         04:32:26:7f:fc:9f:06:a3:73:c9:a9:21:6a:0a:3d:61:e9:e7:
         aa:77:20:d6:a5:49:b0:ea:6d:50:55:12:cd:19:2d:89:73:35:
         ee:30:ac:3e:dd:70:27:86:02:82:fc:a5:68:69:72:39:08:5e:
         c0:87:01:10
```

2. 创建一个 certext.ext 文本文件，内容为：

```
subjectAltName=DNS:www.example.com, DNS:www.example.com
```

3. 生成证书：

```
$ openssl x509 -req -days 365 -in example_csr.pem -signkey example_key.pem -out example_cert.pem -extfile certext.ext
Signature ok
subject=CN = www.example.com, O = "Test, Inc.", C = CN, ST = Beijing, L = Haidian
Getting Private key
unable to write 'random state'
```

#### 查看证书信息

1. 查看证书完整信息：

```
$ openssl x509 -text -in example_cert.pem -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            ae:17:1a:7e:a0:65:2c:14
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = www.example.com, O = "Test, Inc.", C = CN, ST = Beijing, L = Haidian
        Validity
            Not Before: Feb 27 11:20:48 2020 GMT
            Not After : Feb 26 11:20:48 2021 GMT
        Subject: CN = www.example.com, O = "Test, Inc.", C = CN, ST = Beijing, L = Haidian
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d0:d8:a4:d3:48:bc:22:58:c0:25:54:fb:26:fe:
                    33:60:0b:74:12:67:97:ab:9d:62:6a:fd:e6:60:e8:
                    03:5d:c9:8a:d9:0c:9a:13:32:e6:f3:de:32:2f:eb:
                    5e:ae:dc:f9:17:de:8d:32:28:65:63:13:ce:41:d2:
                    ef:0c:2a:c6:bf:87:68:66:90:4e:8a:2f:ed:6f:44:
                    84:15:fe:d8:bf:41:96:7c:22:29:63:26:63:65:d2:
                    9c:46:7b:e0:dc:98:80:4a:96:95:c0:b3:8a:f5:be:
                    27:4a:28:47:05:0a:74:58:96:a3:70:fd:52:7f:3c:
                    5a:20:52:e0:d4:56:d0:84:97:d8:48:0b:72:fa:fd:
                    36:b8:93:01:12:e0:29:06:9f:96:3b:78:f3:e4:ef:
                    91:f2:e2:94:25:49:bd:6d:97:4e:67:66:b0:9a:8d:
                    fc:87:2f:df:3f:46:1a:cd:76:15:b7:45:5c:96:06:
                    e5:92:0a:dc:e5:0b:82:a2:ca:cc:a8:80:b6:70:52:
                    a0:36:79:08:d3:87:dd:59:17:8c:4f:4a:31:8f:c5:
                    5c:12:87:8d:af:56:f5:fc:1a:83:2a:97:67:83:2e:
                    50:a4:c7:f9:a0:ad:1f:b7:a2:70:9b:19:d1:f8:52:
                    68:b4:af:4c:cf:9f:e7:7a:cf:58:f9:32:1e:9b:0e:
                    03:27
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Alternative Name: 
                DNS:www.example.com, DNS:www.example.com
    Signature Algorithm: sha256WithRSAEncryption
         95:6e:59:98:76:ed:38:aa:e6:67:77:56:fc:c3:ce:53:95:69:
         3a:74:2b:3e:ff:03:c3:34:95:99:ec:0d:62:b7:ee:05:d9:89:
         ac:d1:1e:f5:1e:a5:81:eb:f0:21:dd:3f:27:06:d8:e5:b5:80:
         db:b4:ac:1a:56:85:df:79:96:a9:ce:c7:63:c6:c0:6c:d3:cb:
         03:96:d0:56:77:15:d8:49:58:c6:6b:f3:65:3e:e1:c3:e3:ec:
         31:51:1d:b5:0a:82:f6:ab:15:9c:e5:96:15:d5:11:1d:e9:f5:
         7a:f3:7a:8a:f3:b8:6b:79:82:56:79:ed:16:db:77:e3:05:d2:
         3a:21:dd:7b:d8:1c:8a:30:b7:75:16:0f:d4:81:48:2a:e9:77:
         ef:80:d5:51:07:88:f0:14:a4:57:53:5f:e1:fb:7a:04:a7:11:
         81:17:31:38:0d:11:e7:ef:88:ba:15:38:b9:01:2e:23:ac:c9:
         86:5a:e0:69:10:6a:87:f6:df:51:03:dd:8b:bd:72:fa:3f:a9:
         72:93:9c:3a:4c:00:85:b4:24:05:98:e2:10:6a:c2:16:29:45:
         18:e7:3c:bc:14:01:b8:78:53:7c:c5:40:a3:bd:6e:61:bd:a0:
         be:b1:20:05:03:8d:02:91:60:89:3e:b0:22:11:eb:6b:4f:64:
         45:18:54:d6
```

2. 查看证书包含的公钥：

```
$ openssl x509 -in example_cert.pem -pubkey
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0Nik00i8IljAJVT7Jv4z
YAt0EmeXq51iav3mYOgDXcmK2QyaEzLm894yL+tertz5F96NMihlYxPOQdLvDCrG
v4doZpBOii/tb0SEFf7Yv0GWfCIpYyZjZdKcRnvg3JiASpaVwLOK9b4nSihHBQp0
WJajcP1SfzxaIFLg1FbQhJfYSAty+v02uJMBEuApBp+WO3jz5O+R8uKUJUm9bZdO
Z2awmo38hy/fP0YazXYVt0Vclgblkgrc5QuCosrMqIC2cFKgNnkI04fdWReMT0ox
j8VcEoeNr1b1/BqDKpdngy5QpMf5oK0ft6JwmxnR+FJotK9Mz5/nes9Y+TIemw4D
JwIDAQAB
-----END PUBLIC KEY-----
-----BEGIN CERTIFICATE-----
MIIDcjCCAlqgAwIBAgIJAK4XGn6gZSwUMA0GCSqGSIb3DQEBCwUAMGAxGDAWBgNV
BAMMD3d3dy5leGFtcGxlLmNvbTETMBEGA1UECgwKVGVzdCwgSW5jLjELMAkGA1UE
BhMCQ04xEDAOBgNVBAgMB0JlaWppbmcxEDAOBgNVBAcMB0hhaWRpYW4wHhcNMjAw
MjI3MTEyMDQ4WhcNMjEwMjI2MTEyMDQ4WjBgMRgwFgYDVQQDDA93d3cuZXhhbXBs
ZS5jb20xEzARBgNVBAoMClRlc3QsIEluYy4xCzAJBgNVBAYTAkNOMRAwDgYDVQQI
DAdCZWlqaW5nMRAwDgYDVQQHDAdIYWlkaWFuMIIBIjANBgkqhkiG9w0BAQEFAAOC
AQ8AMIIBCgKCAQEA0Nik00i8IljAJVT7Jv4zYAt0EmeXq51iav3mYOgDXcmK2Qya
EzLm894yL+tertz5F96NMihlYxPOQdLvDCrGv4doZpBOii/tb0SEFf7Yv0GWfCIp
YyZjZdKcRnvg3JiASpaVwLOK9b4nSihHBQp0WJajcP1SfzxaIFLg1FbQhJfYSAty
+v02uJMBEuApBp+WO3jz5O+R8uKUJUm9bZdOZ2awmo38hy/fP0YazXYVt0Vclgbl
kgrc5QuCosrMqIC2cFKgNnkI04fdWReMT0oxj8VcEoeNr1b1/BqDKpdngy5QpMf5
oK0ft6JwmxnR+FJotK9Mz5/nes9Y+TIemw4DJwIDAQABoy8wLTArBgNVHREEJDAi
gg93d3cuZXhhbXBsZS5jb22CD3d3dy5leGFtcGxlLmNvbTANBgkqhkiG9w0BAQsF
AAOCAQEAlW5ZmHbtOKrmZ3dW/MPOU5VpOnQrPv8DwzSVmewNYrfuBdmJrNEe9R6l
gevwId0/JwbY5bWA27SsGlaF33mWqc7HY8bAbNPLA5bQVncV2ElYxmvzZT7hw+Ps
MVEdtQqC9qsVnOWWFdURHen1evN6ivO4a3mCVnntFtt34wXSOiHde9gcijC3dRYP
1IFIKul374DVUQeI8BSkV1Nf4ft6BKcRgRcxOA0R5++IuhU4uQEuI6zJhlrgaRBq
h/bfUQPdi71y+j+pcpOcOkwAhbQkBZjiEGrCFilFGOc8vBQBuHhTfMVAo71uYb2g
vrEgBQONApFgiT6wIhHra09kRRhU1g==
-----END CERTIFICATE-----
```

#### 证书格式转换

1. PEM 转 DER

```
$ openssl x509 -in example_cert.pem -out example_cert.der -outform DER
```

#### NSS中的工具

1. derdump 输出der格式证书内容：

```
derdump -i ~/work/gmbrowser/example_cert.der
```

何为单证书和双证书？

单证书：

用户使用唯一的证书及对应的私钥进行签名和加密操作。

签名时，A用户使用自己的私钥加密信息的摘要（签名），B用户使用A的公钥进行解密，对比该摘要是否正确，若正确，则B就确定了A的身份，即验签成功。

加密时，A用户用B的公钥将信息加密传递给B，B使用自己的私钥解密，进而获得信息。

双证书：

包括签名证书和加密证书。

签名证书在签名时使用，仅仅用来验证身份使用，其公钥和私钥均由A自己产生，并且由自己保管，CA不负责其保管任务。

加密证书在传递加密数据时使用，其私钥和公钥由CA产生，并由CA保管（存根）。

既然单证书可以搞定一切，为何要使用双证书？

逻辑上：

如果签名私钥遗失，大不了用户再产生一对，并无多大影响，因此签名密钥没必要交给CA；

如果加密密钥遗失，别人发过来的信息我就没办法解密了，必须从CA那里获取存根。

单证书的话，如果私钥丢了，你如何恢复之前得到的信息呢？

因此从道理上来说两个密钥具有不同的属性，逻辑上应该分开处理。

安全性上：

单证书增加了用户签名被伪造的风险

国家意志：

国家要保证必要的时候有能力对某些通讯进行监控，如果采用单证书，除了自己谁也无法解密（理论上啦），不利于国家安全。因此很多国家法律规定使用双证书。

双证书签发流程：

虽然是双密钥和双证书，从流程上来看，一遍流程即可搞定。

1，用户产生签名密钥对，发送证书请求给RA/CA（请求中包含1个公钥）

2，RA/CA向KMC请求加密密钥对

3，签发两张证书，连同加密密钥一起发送给用户（采用签名证书加密）

4，用户使用自己的签名私钥解密，获得两张证书+加密密钥。



测试服务器：https://47.89.249.43:4433/

（测试时，先将本机时间设置为2018年7月之前（我证书过期了），然后使用360国密浏览器访问。360国密浏览器会在TLS握手失败后才会发起GMSSL握手，所以访问较慢。出现访问不了的情况，请清除360国密浏览器所有缓存，重启浏览器后再访问）

源码在 https://github.com/mrpre/atls 上可以获得

 

 

 

 

GM/T 没有单独规范 SSL协议的文件，而是在SSL VPN技术规范中定义了国密SSL协议。

 

规范号：GM/T 0024-2014



 

 

这里主要讲的是国密SSL协议和标准的TLS协议之间的区别，根据这些区别，完全可以实现国密SSL握手、通讯，实现效果见文末。

 

 

1：协议号

    TLS协议号为0x0301 0x0302 0x0303，分别表示TLS1.0 1.1 1.2

    而国密SSL版本号为0x0101，其参考了TLS1.1。

    故本篇没有描述的握手、加密细节等全部参考TLS 1.1，本篇只罗列GM SSL 与 标准 TLS 之间的区别。

 

Client hello报文如下：



 

 Wireshark是无法解析的该报文的（不知道新版本是否能够解析），若想解析，还需要手动把pcap报文中version对应的字段修改成通用的0x03xx，wireshark方能解析。

 

故该规范定义的SSL协议被称为 国密SSL 1.1。具体参考规范6.3.2.1。

 

 

2：加密算法

 

国密SSL定义了多个加密套件，见6.4.4.1.1




 

 

 

 

实际上，若加密各个阶段（非对称、对称、摘要）都替换成国密标准的，较主流的是如下2个

 1：ECC_SM4_SM3 

 2：ECDHE_SM4_SM3 

 IBC并没有研究过，这里不再详细说明。

注意 ，ECDHE_SM4_SM3 必须要求双向认证。

 

 ECC 对应的是标准的RSA，切勿被ECC迷惑，这个ECC并不和标准的椭圆曲线密钥交换算法类似，而是和RSA类似。服务器发送ECC公钥（在证书中）到客户端，客户端拿ECC公钥加密随机数给服务器（client key exchange）。

 

3：PRF算法

PRF算法和TLS 1.2类似，唯一区别：TLS1.2下，PRF算法为SHA256，而GM SSL的算法为SM3。

 

4：server key exhcnage

标准的server key exchange计算方式是

DH_sign(client_random + server_random + hash_in)

 

而国密SSL下server key exchange计算方式略有不同

Sm2_sign(lient_random + server_random +hash_len + hash_in)

 

国密规范如下描述 server key exchange：

Case ECC:

    Digitally-signed struct

    {

        Opaque client_random[32];

        Opaque server_random[32];

        Opaque ASN.1Cert<1, 2^24-1>;

    }

Case ECDHE:

    ServerECDHEParams params

    Digitally-signed struct

    {

        Opaque client_random[32];

        Opaque server_random[32];

        ServerECDHEParams params

    }

 

总结一下，

(1)：

标准TLS server key exchange是椭圆曲线参数+对该报文的签名，签名时的hash是2个随机数加该报文本身。

(2)：

国密SSL的ECC的server key exchange只是签名，由于本身不包含任何参数，故签名时的hash是2个随机数加上加密证书（注意国密规范描述证书时采用的尖括号的描述，即证书前需要加上长度信息表示）。具体签名算法使用的是sm2。

(3)：

国密SSL的ECDHE证书和标准TLS就一样了，只是签名算法使用的是sm2。

Sm2签名需要一个ID，规范中建议为”1234567812345678”。实现上，都使用该值。

 

5：finished报文

标准TLS对finished使用标准的SHA1、SHA256、SHA384等进行hash运算，国密SSL中，hash运算为smx。Hash运算完成后，就使用上面描述的prf算法进行计算。

 

6：certificate报文

 国密规范定义发送证书时需要发送两个证书，签名证书和加密证书（双证书体系）。

与标准TLS报文格式一样，只是第一个证书是签名证书，第二个证书是加密证书。

我没有找到国密SSL规范定义怎么发送证书链的。


https://blog.csdn.net/xiaxiaorui2003/article/details/4464023