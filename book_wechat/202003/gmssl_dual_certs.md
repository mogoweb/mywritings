# 啥？双证书？

国密标准对于SSL通信定义得不是很清楚，所能依仗的标准只有《GMT 0024-2014 SSL VPN 技术规范》。在文档中提到，国密TLS需要有签名证书和加密证书。开发伊始并没有注意到这个细节，以至于在后面的联调中吃了很多苦头。现将双证书的概念以及配置方法总结一下，希望对大家有所帮助。

### 何为单证书和双证书？

通常情况下，服务器会部署一张证书，用于签名和加密，这就是所谓的**单证书**:

* 签名时，服务器使用自己的私钥加密信息的摘要（签名），客户端使用服务器的公钥（包含在证书中）进行解密，对比该摘要是否正确，若正确，则客户端就确定了服务器的身份，即验签成功。

* 加密时，服务器和客户端协商出会话密钥（一般为对称密钥），会话密钥的产生根据密钥协商算法的不同，过程有所不同，但都会用到证书的公钥和私钥，也就是说证书也用在加密场景中。

在单证书配置下，服务器端的公钥和私钥由服务器负责保存。私钥需要特别保存，如果泄漏出去就会有很大的安全风险。客户端的公钥和私钥一般在通信过程中动态产生，客户端也不会存储。如果客户端也要配置证书，这种情形不常见，不在讨论之列。

而**双证书**则包括签名证书和加密证书：

* 签名证书在签名时使用，仅仅用来验证身份使用，其公钥和私钥均由服务器自己产生，并且由自己保管，CA不负责其保管任务。

* 加密证书在密钥协商时使用，其私钥和公钥由CA产生，并由CA保管（存根）。

既然单证书可以搞定一切，为何要使用双证书？

从道理上来说两个密钥具有不同的属性，逻辑上应该分开处理。其实最主要的原因是国家要保证必要的时候有能力对某些通讯进行监控，如果采用单证书，除了自己谁也无法解密（理论上如此），不利于国家安全。因此某些国家法律规定使用双证书。如果拥有加密证书的私钥，可以进行实时监控。使用过wireshark抓HTTPS包的朋友应该知道，如果配置了RSA密钥，可以解密出HTTPS通信中的加密信息。

关于安全性话题，我并非这方面的专家，也许理解有误。下面说说如何配置双证书。

### 配置双证书测试环境

#### TASSL

在《[搭建国密SSL开发测试环境](https://mp.weixin.qq.com/s/rUeOtFjB3QTPJW3RKccKag)》一文中，我介绍了GmSSL这个开源项目。在开发过程中，我也不断在搜寻资料，后来又发现另一个实现国密的开源项目TASSL。TASSL也是基于OpenSSL开发，由北京江南天安科技有限公司维护，这是中国领先的密码技术与信息安全综合服务商。TASSL基于OpenSSL 1.0.2o版本开发，其项目地址为：

> https://github.com/jntass/TASSL/

他们还有一个基于OpenSSL 1.1.1b版本开发的版本，其项目地址为：

> https://github.com/jntass/TASSL-1.1.1b

在这个项目里面，我找到了配置双证书的方法。下面就以TASSL开源项目为例，说说双证书测试环境的搭建。

#### 编译TASSL

因为都是基于OpenSSL开发，其编译过程和GmSSL差不多，详细的步骤请参考我之前写的《[搭建国密SSL开发测试环境](https://mp.weixin.qq.com/s/rUeOtFjB3QTPJW3RKccKag)》。

下面直接给出步骤：

```
$ git clone https://github.com/jntass/TASSL/
$ cd TASSL
$ chmod a+x config
$ ./config --prefix=/home/alex/work/gmbrowser/usr/local/tassl
$ make
$ make install
```

如果出现如下错误：

```
/bin/sh: 1: ./pod2mantest: Permission denied
installing man1/CA.pl.1
sh: 1: --section=1: not found
Makefile:646: recipe for target 'install_docs' failed
make: *** [install_docs] Error 127
```

你可以选择忽略，因为这一步是安装文档，一般来说用不着。如果要解决，也很简单。其原因在于克隆下来的文件，某些可执行脚本和文件的可执行属性整没了。执行如下的命令，修改util下脚本和可执行文件的可执行属性，然后再执行一遍make install。

```
$ chmod a+x ./util/*
```

设置环境变量：

```
PATH=$HOME/work/gmbrowser/usr/local/tassl/bin:$PATH
LD_LIBRARY_PATH=$HOME/work/gmbrowser/usr/local/tassl/lib:$LD_LIBRARY_PATH
```

查看一下版本：

```
$ openssl version
OpenSSL/TaSSL 1.0.2o  27 Mar 2018
```

#### 生成签名证书和加密证书

TASSL提供了一个脚本，可用来生成CA证书、Server证书和Client证书。脚本位于 *Tassl_demo/mk_tls_cert/SM2certgen.sh*，执行如下命令可以生成全套的测试证书：

```
$ cd Tassl_demo/mk_tls_cert
$ sh SM2certgen.sh
```

证书生成在其下的sm2Certs子目录里，其中：

* CA.key.pem和CA.cert.pem分别是CA私钥和CA证书。
* CE.cert.pem和CE.key.pem分别是客户端的加密证书和对应的私钥。
* CS.cert.pem和CS.key.pem分别是客户端的签名证书和对应的私钥。
* SE.cert.pem和SE.key.pem分别是服务器的加密证书和对应的私钥。
* SS.cert.pem和SS.key.pem分别是服务器的签名证书和对应的私钥。

有人可能注意到，生成的文件还有CA.pem、CE.pem、CS.pem、SE.pem和SS.pem文件，他们是证书和对应的私钥合并在一起形成的，主要是为了使用方便。

#### 配置双证书通信

服务器端：

```
openssl s_server -accept 44330 -CAfile sm2Certs/CA.pem -cert sm2Certs/SS.pem -enc_cert sm2Certs/SE.pem
```

客户端：

```
openssl s_client -connect 127.0.0.1:44330 -cntls
```

它们之间通信会使用 ECC-SM4-SM3 密码套件。

### 小结

本文介绍了双证书的来头，随后介绍了一个新的国密开源实现TASSL，最后介绍了双证书的生成和测试环境配置。

如果大家有什么问题，欢迎交流。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)