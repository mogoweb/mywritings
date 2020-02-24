HTTPS 通信的第一步是协商密码套件(CipherSuite)。

密码套件是一系列密码学算法的组合，主要包含多个密码学算法：

* 身份验证算法。
* 密码协商算法。
* 加密算法或者加密模式。
* HMAC算法的加密基元。
* PRF算法的加密基元，需要注意的是，不同的TLS/SSL协议版本、密码套件，PRF算法最终使用的加密基元和HMAC算法使用的加密基元是不一样的。

密码套件的构成如下图所示:

![密码套件结构](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_work/gmbrowser/images/tls_01.png)

使用GMSSL的命令查看所支持的密码套件：

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

而 GMSSL 代码中定义的密码套件如下(t1_trce.c):

```c++
    /* GM/T [SM2DHE|SM2|SM9DHE|SM9|RSA]-WITH-[SM1|SMS4]-[SM3|SHA1] */
    {0xE001, "GMTLS_SM2DHE_WITH_SM1_SM3"},
    {0xE003, "GMTLS_SM2_WITH_SM1_SM3"},
    {0xE005, "GMTLS_SM9DHE_WITH_SM1_SM3"},
    {0xE007, "GMTLS_SM9_WITH_SM1_SM3"},
    {0xE009, "GMTLS_RSA_WITH_SM1_SM3"},
    {0xE00A, "GMTLS_RSA_WITH_SM1_SHA1"},
    {0xE011, "GMTLS_SM2DHE_WITH_SMS4_SM3"},
    {0xE013, "GMTLS_SM2_WITH_SMS4_SM3"},
    {0xE015, "GMTLS_SM9DHE_WITH_SMS4_SM3"},
    {0xE017, "GMTLS_SM9_WITH_SMS4_SM3"},
    {0xE019, "GMTLS_RSA_WITH_SMS4_SM3"},
    {0xE01A, "GMTLS_RSA_WITH_SMS4_SHA1"},

    {0xE101, "GMTLS_ECDHE_SM2_WITH_SM1_SM3"},
    {0xE102, "GMTLS_ECDHE_SM2_WITH_SMS4_SM3"},
    {0xE103, "GMTLS_ECDHE_SM2_WITH_SSF33_SM3"},
    {0xE105, "GMTLS_ECDHE_SM2_WITH_SMS4_SHA256"},
    {0xE107, "GMTLS_ECDHE_SM2_WITH_SMS4_GCM_SM3"},
    {0xE108, "GMTLS_ECDHE_SM2_WITH_SMS4_CCM_SM3"},

    {0xE10D, "GMTLS_ECDHE_SM2_WITH_ZUC_SM3"},
    {0xE10E, "GMTLS_ECDHE_SM2_WITH_ZUC256_SM3"},
    {0xE201, "GMTLS_SM2DHE_SM2_WITH_SM1_SM3"},
    {0xE202, "GMTLS_SM2DHE_SM2_WITH_SMS4_SM3"},
    {0xE203, "GMTLS_SM2DHE_SM2_WITH_SSF33_SM3"},
    {0xE204, "GMTLS_SM2DHE_SM2_WITH_ZUC_SM3"},
    {0xE205, "GMTLS_SM2DHE_SM2_WITH_SMS4_GCM_SM3"},
    {0xE206, "GMTLS_SM2DHE_SM2_WITH_SMS4_CCM_SM3"},
    {0xE209, "GMTLS_SM2DHE_SM2_WITH_ZUC256_SM3"},
    {0xF101, "GMTLS_PSK_WITH_SMS4_CBC_SM3"},
    {0xF102, "GMTLS_PSK_WITH_SMS4_GCM_SM3"},
    {0xF103, "GMTLS_PSK_WITH_SMS4_CCM_SM3"},
    {0xF10B, "GMTLS_SM2DHE_PSK_WITH_SMS4_CBC_SM3"},
    {0xF10C, "GMTLS_SM2DHE_PSK_WITH_SMS4_GCM_SM3"},
    {0xF10D, "GMTLS_SM2DHE_PSK_WITH_SMS4_CCM_SM3"},
    {0xF10E, "GMTLS_PSK_WITH_SM1_CBC_SM3"},
    {0xF117, "GMTLS_PSK_WITH_SSF33_CBC_SM3"},
    {0xF120, "GMTLS_ECDHE_PSK_WITH_SMS4_CBC_SM3"},
    {0xF121, "GMTLS_ECDHE_PSK_WITH_SMS4_GCM_SM3"},
    {0xF122, "GMTLS_ECDHE_PSK_WITH_SMS4_CCM_SM3"},
    {0xF123, "GMTLS_PSK_WITH_ZUC_SM3"},
    {0xF124, "GMTLS_PSK_WITH_ZUC256_SM3"},
    {0xF125, "GMTLS_ECDHE_PSK_WITH_ZUC_SM3"},
    {0xF126, "GMTLS_ECDHE_PSK_WITH_ZUC256_SM3"},
    {0xF127, "GMTLS_SM2DHE_PSK_WITH_ZUC_SM3"},
    {0xF128, "GMTLS_SM2DHE_PSK_WITH_ZUC256_SM3"},
    {0xF201, "GMTLS_SRP_SM3_WITH_SMS4_CBC_SM3"},
    {0xF202, "GMTLS_SRP_SM3_WITH_SMS4_GCM_SM3"},
    {0xF203, "GMTLS_SRP_SM3_WITH_SMS4_CCM_SM3"},
    {0xFEFE, "SSL_RSA_FIPS_WITH_DES_CBC_SHA"},
    {0xFEFF, "SSL_RSA_FIPS_WITH_3DES_EDE_CBC_SHA"},

#ifndef OPENSSL_NO_GMTLS
    {0xF10E, "GMTLS_ECDHE_PSK_WITH_SMS4_CBC_SM3"},
#endif
```

测试服务器是否支持某个特定的密码套件:

```bash
openssl s_client -cipher "ECDHE-SM2-WITH-SMS4-SM3" -connect sm2test.ovssl.cn:443 -tls1_2 -servername sm2test.ovssl.cn
```

提示如下：

```
CONNECTED(00000003)
depth=2 C = CN, O = \E6\B2\83\E9\80\9A\E7\94\B5\E5\AD\90\E8\AE\A4\E8\AF\81\E6\9C\8D\E5\8A\A1\E6\9C\89\E9\99\90\E5\85\AC\E5\8F\B8, CN = \E5\9B\BD\E5\AF\86SM2\E6\A0\B9\E8\AF\81\E4\B9\A6
verify error:num=19:self signed certificate in certificate chain
139996653156096:error:14094438:SSL routines:ssl3_read_bytes:tlsv1 alert internal error:ssl/record/rec_layer_s3.c:1385:SSL alert number 80
---
Certificate chain
 0 s:/C=CN/ST=\xE5\xB9\xBF\xE4\xB8\x9C\xE7\x9C\x81/L=\xE6\xB7\xB1\xE5\x9C\xB3\xE5\xB8\x82/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=sm2test.ovssl.cn
   i:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6V3
 1 s:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6V3
   i:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6
 2 s:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6
   i:/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIDdzCCAx2gAwIBAgIQJUaFElLAqiYv5f9gEXi5tjAKBggqgRzPVQGDdTBkMQsw
CQYDVQQGEwJDTjEtMCsGA1UECgwk5rKD6YCa55S15a2Q6K6k6K+B5pyN5Yqh5pyJ
6ZmQ5YWs5Y+4MSYwJAYDVQQDDB3lm73lr4ZTTTLmnI3liqHlmajmoLnor4HkuaZW
MzAeFw0xOTA0MTExNDE1MjRaFw0yMTA0MTAxOTA5MTFaMH8xCzAJBgNVBAYTAkNO
MRIwEAYDVQQIDAnlub/kuJznnIExEjAQBgNVBAcMCea3seWcs+W4gjEtMCsGA1UE
Cgwk5rKD6YCa55S15a2Q6K6k6K+B5pyN5Yqh5pyJ6ZmQ5YWs5Y+4MRkwFwYDVQQD
DBBzbTJ0ZXN0Lm92c3NsLmNuMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAE+eyf
ggvpJ6OMdCHklaOPmdMToBhrbDkcQE0jlHs5IzL7Ud625vICsjaFcGMxnDpvC7vM
ceZfdwl2271ajjn+oKOCAZQwggGQMA4GA1UdDwEB/wQEAwIHgDAdBgNVHSUEFjAU
BggrBgEFBQcDAQYIKwYBBQUHAwIwCQYDVR0TBAIwADAdBgNVHQ4EFgQU7hM+52Zw
ldjO4s054zz8pvm0LfkwHwYDVR0jBBgwFoAUjoZouSzYrssgJvZftjtaXuqbmWYw
ZQYIKwYBBQUHAQEEWTBXMCIGCCsGAQUFBzABhhZodHRwOi8vb2NzcC53b3RydXMu
Y29tMDEGCCsGAQUFBzAChiVodHRwOi8vYWlhLndvdHJ1cy5jb20vd3Mtc20yLXNz
bDMuY2VyMDYGA1UdHwQvMC0wK6ApoCeGJWh0dHA6Ly9jcmwud290cnVzLmNvbS93
cy1zbTItc3NsMy5jcmwwGwYDVR0RBBQwEoIQc20ydGVzdC5vdnNzbC5jbjBYBgNV
HSAEUTBPMAgGBmeBDAECAjAJBgcqgRyJmCoNMDgGCSqBHImYKgMBAzArMCkGCCsG
AQUFBwIBFh1odHRwOi8vd3d3LndvdHJ1cy5jb20vcG9saWN5LzAKBggqgRzPVQGD
dQNIADBFAiBBfaLq7LJm2ntgodJDyt3m57Osw5VsttxYOv4wkZGd5AIhAMsIlcWu
323uXs4yRQ3Pa3jdxmoapeRYcM6WredGASeV
-----END CERTIFICATE-----
subject=/C=CN/ST=\xE5\xB9\xBF\xE4\xB8\x9C\xE7\x9C\x81/L=\xE6\xB7\xB1\xE5\x9C\xB3\xE5\xB8\x82/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=sm2test.ovssl.cn
issuer=/C=CN/O=\xE6\xB2\x83\xE9\x80\x9A\xE7\x94\xB5\xE5\xAD\x90\xE8\xAE\xA4\xE8\xAF\x81\xE6\x9C\x8D\xE5\x8A\xA1\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=\xE5\x9B\xBD\xE5\xAF\x86SM2\xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xA0\xB9\xE8\xAF\x81\xE4\xB9\xA6V3
---
No client certificate CA names sent
Peer signing digest: SM3
Server Temp Key: ECDH, SM2, 256 bits
---
SSL handshake has read 2472 bytes and written 319 bytes
Verification error: self signed certificate in certificate chain
---
New, TLSv1.2, Cipher is ECDHE-SM2-WITH-SMS4-SM3
Server public key is 256 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-SM2-WITH-SMS4-SM3
    Session-ID: 
    Session-ID-ctx: 
    Master-Key: CFFBDA58AA48F7FF9148F5F563667F3265420844D315735AC2206187946E33DE78B98B2F4DE7D4C33B335F6291DC3EBA
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1582372157
    Timeout   : 7200 (sec)
    Verify return code: 19 (self signed certificate in certificate chain)
    Extended master secret: yes
---
```

出现: New, TLSv1.2, Cipher is ECDHE-SM2-WITH-SMS4-SM3 ，表明协商成功。

经过测试，只有 "ECDHE-SM2-WITH-SMS4-SM3" 协商成功。

使用NSS指令列出支持的密码套件：

```
listsuites | grep "Enabled"
```
输出为：

```
  0x1301 TLS 1.3 TLS 1.3 AES-GCM  128 AEAD   Enabled  FIPS Domestic            
  0x1303 TLS 1.3 TLS 1.3 CHACHA20POLY1305 256 AEAD   Enabled       Domestic            
  0x1302 TLS 1.3 TLS 1.3 AES-GCM  256 AEAD   Enabled       Domestic            
  0xc02b ECDHE ECDSA AES-GCM  128 AEAD   Enabled  FIPS Domestic            
  0xc02f ECDHE RSA   AES-GCM  128 AEAD   Enabled  FIPS Domestic            
  0xcca9 ECDHE ECDSA CHACHA20POLY1305 256 AEAD   Enabled       Domestic            
  0xcca8 ECDHE RSA   CHACHA20POLY1305 256 AEAD   Enabled       Domestic            
  0xc00a ECDHE ECDSA AES      256 SHA1   Enabled  FIPS Domestic            
  0xc009 ECDHE ECDSA AES      128 SHA1   Enabled  FIPS Domestic            
  0xc013 ECDHE RSA   AES      128 SHA1   Enabled  FIPS Domestic            
  0xc023 ECDHE ECDSA AES      128 SHA256 Enabled  FIPS Domestic            
  0xc027 ECDHE RSA   AES      128 SHA256 Enabled  FIPS Domestic            
  0xc014 ECDHE RSA   AES      256 SHA1   Enabled  FIPS Domestic            
  0x009e DHE   RSA   AES-GCM  128 AEAD   Enabled  FIPS Domestic            
  0xccaa DHE   RSA   CHACHA20POLY1305 256 AEAD   Enabled       Domestic            
  0x0033 DHE   RSA   AES      128 SHA1   Enabled  FIPS Domestic            
  0x0032 DHE   DSA   AES      128 SHA1   Enabled  FIPS Domestic            
  0x0067 DHE   RSA   AES      128 SHA256 Enabled  FIPS Domestic            
  0x0039 DHE   RSA   AES      256 SHA1   Enabled  FIPS Domestic            
  0x0038 DHE   DSA   AES      256 SHA1   Enabled  FIPS Domestic            
  0x006b DHE   RSA   AES      256 SHA256 Enabled  FIPS Domestic            
  0x0016 DHE   RSA   3DES     112 SHA1   Enabled  FIPS Domestic            
  0x0013 DHE   DSA   3DES     112 SHA1   Enabled  FIPS Domestic            
  0x009c RSA   RSA   AES-GCM  128 AEAD   Enabled  FIPS Domestic            
  0x002f RSA   RSA   AES      128 SHA1   Enabled  FIPS Domestic            
  0x003c RSA   RSA   AES      128 SHA256 Enabled  FIPS Domestic            
  0x0035 RSA   RSA   AES      256 SHA1   Enabled  FIPS Domestic            
  0x003d RSA   RSA   AES      256 SHA256 Enabled  FIPS Domestic            
  0x000a RSA   RSA   3DES     112 SHA1   Enabled  FIPS Domestic            
  0x0005 RSA   RSA   RC4      128 SHA1   Enabled       Domestic            
  0x0004 RSA   RSA   RC4      128 MD5    Enabled       Domestic
```

第一步： 实现 ECDHE-SM2-WITH-SMS4-SM3 密码套件

0xE1,0x02  -  ECDHE-SM2-WITH-SMS4-SM3        TLSv1.2    Kx=ECDH      Au=SM2    Enc=SMS4(128)               Mac=SM3

在NSS中，系统所支持的密码套件定义在 sslenum.c 文件中的 SSL_ImplementedCiphers[]数组中，这里面值是密码套件的编号。

密码套件具体的定义列表位于 ssl3conf.c 文件的 cipher_suite_defs[] 数组中，其数据结构 为 ssl3CipherSuiteDef, 字段有加密算法、MAC算法、密钥交换算法、哈希算法。

系统实现了的密码套件配置列表定义在 ssl3con.c 文件的 cipherSuites[ssl_V3_SUITES_IMPLEMENTED] 数组中， 其数据结构为 ssl3CipherSuiteCfg, 字段有策略、是否启用等。

具体的密码套件信息列表定义在 sslinfo.c 文件的 suiteInfo[] 数组中，其数据结构为 SSLCipherSuiteInfo，包含了很多信息：

```c++
typedef struct SSLCipherSuiteInfoStr {
    /* On return, SSL_GetCipherSuitelInfo sets |length| to the smaller of
     * the |len| argument and the length of the struct used by NSS.
     * Callers must ensure the application uses a version of NSS that
     * isn't older than the version used at compile time. */
    PRUint16 length;
    PRUint16 cipherSuite;

    /* Cipher Suite Name */
    const char* cipherSuiteName;

    /* server authentication info */
    const char* authAlgorithmName;
    SSLAuthType authAlgorithm; /* deprecated, use |authType| */

    /* key exchange algorithm info */
    const char* keaTypeName;
    SSLKEAType keaType;

    /* symmetric encryption info */
    const char* symCipherName;
    SSLCipherAlgorithm symCipher;
    PRUint16 symKeyBits;
    PRUint16 symKeySpace;
    PRUint16 effectiveKeyBits;

    /* MAC info */
    /* AEAD ciphers don't have a MAC. For an AEAD cipher, macAlgorithmName
     * is "AEAD", macAlgorithm is ssl_mac_aead, and macBits is the length in
     * bits of the authentication tag. */
    const char* macAlgorithmName;
    SSLMACAlgorithm macAlgorithm;
    PRUint16 macBits;

    PRUintn isFIPS : 1;
    PRUintn isExportable : 1; /* deprecated, don't use */
    PRUintn nonStandard : 1;
    PRUintn reservedBits : 29;

    /* The following fields were added in NSS 3.24. */
    /* This reports the correct authentication type for the cipher suite, use
     * this instead of |authAlgorithm|. */
    SSLAuthType authType;

    /* When adding new fields to this structure, please document the
     * NSS version in which they were added. */
} SSLCipherSuiteInfo;
```

在 secoidt.h 文件中添加一个 SECOidTag 枚举值后，需要在 secoid.c 文件中的 oids 数组加入一条数据，通常用 OD 宏定义。

TLS 测试客户端, 使用 TLS_ECDHE_SM2_WITH_SMS4_SM3 密码套件连接国密测试网站:

```
tstclnt -h sm2test.ovssl.cn -p 443 -D -o -v -c a
```

提示：

```
tstclnt: read from socket failed: SSL_ERROR_NO_CYPHER_OVERLAP: Cannot communicate securely with peer: no common encryption algorithm(s).
```

密码套件协商不成功。使用wireshark分析网络包，握手协议 客户端发送的信息如下：

![NSS发送的Client Hello](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_work/gmbrowser/images/tls_02.png)

而使用 GMSSL ，客户端发送的信息如下：

![GMSSL发送的Client Hello](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_work/gmbrowser/images/tls_03.png)

NSS更换密码套件 TLS_RSA_WITH_AES_256_CBC_SHA 协商成功：

![NSS发送的Client Hello，协商成功](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_work/gmbrowser/images/tls_04.png)

其原因可能在于 TLS_ECDHE_SM2_WITH_SMS4_SM3 密码套件未发送所需的 extension 数据。

```
{TLSEXT_TYPE_ec_point_formats, "ec_point_formats"},
```

在GMSSL中定义了 ssl_print_extension 方法，输出扩展信息。


=================== 2020.2.24 补充 =================

要在Client Hello 数据报文中附加 ec_point_formats 扩展，首先需要在 ssl3ecc.c 文件的 ssl_all_ec_suites 数组中添加 TLS_ECDHE_SM2_WITH_SMS4_SM3 项。

发送 ec_point_formats 扩展的函数为： ssl3_SendSupportedPointFormatsXtn。

其中 ec_point_formats 参数定义为：
```c++
     static const PRUint8 ecPtFmt[6] = {
        0, 11, /* Extension type */
        0, 2,  /* octets that follow */
        1,     /* octets that follow */
        0      /* uncompressed type only */
    };
```

这个和GMSSL 发送的 ec_point_formats 参数不同：
