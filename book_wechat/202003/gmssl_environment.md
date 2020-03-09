# 搭建国密SSL开发测试环境

国密算法包含了一系列的加密算法，用途广泛，可以用于软硬件加密、签名等地方。我主要研究国密算法在SSL/TLS/HTTPS通信中的应用，这会涉及到客户端和服务器端，最典型的用例就是浏览器访问Web服务器。它要求客户端和服务器都支持国密算法，才能进行通信。如果我们在开发客户端产品，就需要有一个支持国密的服务器配合测试，反之亦然。我们可以使用一些现成的产品，如果开发服务器端，可以使用密信浏览器，如果开发客户端，可以使用一个在线网站：

> https://sm2test.ovssl.cn/

使用第三方产品有一些不可控，一种方案是自己搭建测试客户端或服务器端。

OpenSSL不仅仅是一个SSL库，还是一个SSL工具箱，可以用来进行加解密、制作证书、签名、等等，甚至还可以配置一个简单的SSL客户端和服务器端。而GmSSL基于OpenSSL开发，保持了接口兼容，SSL工具箱的命令行及参数也基本相同。

下面以GmSSL为蓝本，谈谈如何搭建国密SSL测试环境。

## 代码编译与运行

### 1. 代码下载

可以直接下载源代码包(https://github.com/guanzhi/GmSSL/archive/master.zip)。

也可以通过git命令克隆代码库：

```
$ git clone https://github.com/guanzhi/GmSSL gmssl
```

### 2. 编译代码

为了避免和系统的openssl库冲突，请务必指定 prefix 参数，最好指定一个 $HOME 下有可写权限的路径：

```
$ cd gmssl
$ ./config --prefix=/home/alex/work/gmbrowser/usr/local/gmssl
$ make
$ make install
```

经过上面的步骤，可执行文件会安装到 ${prefix} 的bin子目录下，而库文件则会复制到lib子目录下。

### 3. 运行gmssl程序

因为第2步将可执行文件和库安装到了 ${prefix} 目录下，所以运行之前请指定路径：

```
PATH=$HOME/work/gmbrowser/usr/local/gmssl/bin:$PATH
LD_LIBRARY_PATH=$HOME/work/gmbrowser/usr/local/gmssl/lib:$LD_LIBRARY_PATH
```

然后再运行命令行工具：

```
$ gmssl version
GmSSL 2.5.4 - OpenSSL 1.1.0d  3 Sep 2019
```

表明安装成功，注意gmssl有个指向openssl的软链接，所以使用openssl命令也是可以的。gmssl是在openssl源码的基础上修改，命令行工具也保持兼容，所以网上查找到的openssl指令和参数，在gmssl下同样适用。

## 创建证书

### 1. 生成SM2私钥：

```
$ gmssl ecparam -genkey -name sm2p256v1 -text -out user.key
```

私钥保存在user.key中，请妥善保存。

### 2. 创建证书请求：

```
$ gmssl req -new -key user.key -out user.req

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [CN]:
State or Province Name (full name) [Some-State]:Hubei
Locality Name (eg, city) []:Wuhan
Organization Name (eg, company) [Internet Widgits Pty Ltd]:mogoweb
Organizational Unit Name (eg, section) []:mogoweb
Common Name (e.g. server FQDN or YOUR name) []:mogoweb.com
Email Address []:mogoweb@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

上面的命令会要求提供一些信息，因为证书是自签名证书，只用于开发测试，所以填写什么内容无关紧要。

#### 3.创建一个 certext.ext 文本文件，内容为：

```
subjectAltName=DNS:www.example.com
```

如果客户端要校验服务器的证书，可能会检查证书的DNS是否和服务器的名称一致。假如你使用的服务器有DNS名称，请填上，如果是本地测试，可以先暂时填一个名称，以后再修改。

#### 4. 生成证书：

```
$ gmssl x509 -req -days 365 -in user.req -signkey user.key -out user_cert.pem -extfile certext.ext
Signature ok
subject=C = CN, ST = Hubei, L = Wuhan, O = mogoweb, OU = mogoweb, CN = mogoweb.com, emailAddress = mogoweb@gmail.com
Getting Private key
```

参数中的user.req和user.key分别为第2步和第1步中生成的证书请求和SM2私钥。

生成的证书为user_cert.pem，我们可以通过命令查看证书的内容：

```
$ gmssl x509 -text -in user_cert.pem -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            d8:ae:cf:5e:6e:14:56:66
    Signature Algorithm: sm2sign-with-sm3
        Issuer: C = CN, ST = Hubei, L = Wuhan, O = mogoweb, OU = mogoweb, CN = mogoweb.com, emailAddress = mogoweb@gmail.com
        Validity
            Not Before: Mar  2 09:00:52 2020 GMT
            Not After : Mar  2 09:00:52 2021 GMT
        Subject: C = CN, ST = Hubei, L = Wuhan, O = mogoweb, OU = mogoweb, CN = mogoweb.com, emailAddress = mogoweb@gmail.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:44:c4:49:e7:02:f4:fe:e2:18:12:44:ce:cc:d9:
                    f2:f2:36:1c:41:85:59:21:7c:43:9c:47:f5:f8:a5:
                    81:19:66:c9:3e:89:9a:d0:ed:32:9e:de:43:b7:dc:
                    d1:af:cd:40:31:9a:76:e5:ea:24:47:06:9f:22:14:
                    24:33:4f:fe:d1
                ASN1 OID: sm2p256v1
                NIST CURVE: SM2
        X509v3 extensions:
            X509v3 Subject Alternative Name: 
                DNS:www.example.com, DNS:www.example.com
    Signature Algorithm: sm2sign-with-sm3
         30:44:02:20:09:f7:ca:c6:5b:76:7b:4d:12:4e:62:42:a7:ae:
         1f:2c:f7:f2:3b:8b:97:b2:4a:39:dd:60:6e:d0:4c:61:5a:da:
         02:20:4e:cc:d1:80:34:4c:0d:42:d9:cd:85:39:12:c5:4f:3a:
         0d:04:25:bd:75:68:8e:60:b5:6a:86:60:65:84:5a:68
```

## 启动服务端和客户端

### 1. 启动服务器端

启动GmSSL s_server:

```
$ gmssl s_server -key user.key -cert user_cert.pem -accept 44330 -www
```

其中:

-key 参数指定私钥，-cert 参数指定证书，这里使用前面生成的自签名证书。-accept 参数指定端口号。-www 参数在客户端连接时将状态消息发送回客户端。这包括使用的密码套件和各种会话参数的信息。

如果通过chrome浏览器访问 https://localhost:44330，会出现如下提示：

![chrome浏览器访问出错](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_environment_01.png)

因为chrome浏览器并没有实现国密相关的密码套件，所以SSL通信不成功。不用担心，我们还是可以使用GmSSL的客户端程序进行连接测试。

如果你希望查看双方的通信信息，还可以加上 -debug 参数，将会输出更多的信息：

```
$ gmssl s_server -key user.key -cert user_cert.pem -accept 44330 -www -debug
Using default temp DH parameters
[GMTLS_DEBUG] set sm2 signing certificate
[GMTLS_DEBUG] set sm2 signing private key
ACCEPT
read from 0x2539a40 [0x253f4a3] (5 bytes => 5 (0x5))
0000 - 16 03 01 00 bf                                    .....
read from 0x2539a40 [0x253f4a8] (191 bytes => 191 (0xBF))
0000 - 01 00 00 bb 03 03 cc 41-c5 d1 e0 9e 32 4f 51 8d   .......A....2OQ.
0010 - 76 4b 5d 91 95 fa c5 ae-32 9f 4d ce 86 f7 b6 05   vK].....2.M.....
0020 - 9a ce 8a db 62 6b 00 00-48 c0 2c c0 30 00 9f cc   ....bk..H.,.0...
0030 - a9 cc a8 cc aa c0 2b c0-2f 00 9e e1 07 c0 24 c0   ......+./.....$.
0040 - 28 00 6b c0 23 c0 27 00-67 e1 02 c0 0a c0 14 00   (.k.#.'.g.......
0050 - 39 c0 09 c0 13 00 33 00-9d 00 9c 00 3d 00 3c 00   9.....3.....=.<.
0060 - 35 e0 17 e0 15 e0 13 e0-11 00 2f e0 1a e0 19 00   5........./.....
0070 - ff 01 00 00 4a 00 0b 00-04 03 00 01 02 00 0a 00   ....J...........
0080 - 0c 00 0a 00 1e 00 1d 00-17 00 19 00 18 00 23 00   ..............#.
0090 - 00 00 0d 00 22 00 20 06-01 06 02 06 03 05 01 05   ....". .........
00a0 - 02 05 03 04 01 04 02 04-03 03 01 03 02 03 03 02   ................
00b0 - 01 02 02 02 03 07 07 00-16 00 00 00 17            .............
00bf - <SPACES/NULS>
write to 0x2539a40 [0x254dc20] (821 bytes => 821 (0x335))
0000 - 16 03 03 00 41 02 00 00-3d 03 03 40 63 b8 ea 75   ....A...=..@c..u
0010 - e0 98 a9 d3 10 e7 ed 2f-13 fe e3 fd 7d b5 bb 7b   ......./....}..{
0020 - 1e f3 58 fd c9 ac 36 53-8f b9 7c 00 e1 07 00 00   ..X...6S..|.....
0030 - 15 ff 01 00 01 00 00 0b-00 04 03 00 01 02 00 23   ...............#
0040 - 00 00 00 17 00 00 16 03-03 02 47 0b 00 02 43 00   ..........G...C.
0050 - 02 40 00 02 3d 30 82 02-39 30 82 01 e0 a0 03 02   .@..=0..90......
0060 - 01 02 02 09 00 d8 ae cf-5e 6e 14 56 66 30 0a 06   ........^n.Vf0..
0070 - 08 2a 81 1c cf 55 01 83-75 30 81 89 31 0b 30 09   .*...U..u0..1.0.
0080 - 06 03 55 04 06 13 02 43-4e 31 0e 30 0c 06 03 55   ..U....CN1.0...U
0090 - 04 08 0c 05 48 75 62 65-69 31 0e 30 0c 06 03 55   ....Hubei1.0...U
00a0 - 04 07 0c 05 57 75 68 61-6e 31 10 30 0e 06 03 55   ....Wuhan1.0...U
00b0 - 04 0a 0c 07 6d 6f 67 6f-77 65 62 31 10 30 0e 06   ....mogoweb1.0..
00c0 - 03 55 04 0b 0c 07 6d 6f-67 6f 77 65 62 31 14 30   .U....mogoweb1.0
00d0 - 12 06 03 55 04 03 0c 0b-6d 6f 67 6f 77 65 62 2e   ...U....mogoweb.
00e0 - 63 6f 6d 31 20 30 1e 06-09 2a 86 48 86 f7 0d 01   com1 0...*.H....
00f0 - 09 01 16 11 6d 6f 67 6f-77 65 62 40 67 6d 61 69   ....mogoweb@gmai
0100 - 6c 2e 63 6f 6d 30 1e 17-0d 32 30 30 33 30 32 30   l.com0...2003020
0110 - 39 30 30 35 32 5a 17 0d-32 31 30 33 30 32 30 39   90052Z..21030209
0120 - 30 30 35 32 5a 30 81 89-31 0b 30 09 06 03 55 04   0052Z0..1.0...U.
0130 - 06 13 02 43 4e 31 0e 30-0c 06 03 55 04 08 0c 05   ...CN1.0...U....
0140 - 48 75 62 65 69 31 0e 30-0c 06 03 55 04 07 0c 05   Hubei1.0...U....
0150 - 57 75 68 61 6e 31 10 30-0e 06 03 55 04 0a 0c 07   Wuhan1.0...U....
0160 - 6d 6f 67 6f 77 65 62 31-10 30 0e 06 03 55 04 0b   mogoweb1.0...U..
0170 - 0c 07 6d 6f 67 6f 77 65-62 31 14 30 12 06 03 55   ..mogoweb1.0...U
0180 - 04 03 0c 0b 6d 6f 67 6f-77 65 62 2e 63 6f 6d 31   ....mogoweb.com1
0190 - 20 30 1e 06 09 2a 86 48-86 f7 0d 01 09 01 16 11    0...*.H........
01a0 - 6d 6f 67 6f 77 65 62 40-67 6d 61 69 6c 2e 63 6f   mogoweb@gmail.co
01b0 - 6d 30 59 30 13 06 07 2a-86 48 ce 3d 02 01 06 08   m0Y0...*.H.=....
01c0 - 2a 81 1c cf 55 01 82 2d-03 42 00 04 44 c4 49 e7   *...U..-.B..D.I.
01d0 - 02 f4 fe e2 18 12 44 ce-cc d9 f2 f2 36 1c 41 85   ......D.....6.A.
01e0 - 59 21 7c 43 9c 47 f5 f8-a5 81 19 66 c9 3e 89 9a   Y!|C.G.....f.>..
01f0 - d0 ed 32 9e de 43 b7 dc-d1 af cd 40 31 9a 76 e5   ..2..C.....@1.v.
0200 - ea 24 47 06 9f 22 14 24-33 4f fe d1 a3 2f 30 2d   .$G..".$3O.../0-
0210 - 30 2b 06 03 55 1d 11 04-24 30 22 82 0f 77 77 77   0+..U...$0"..www
0220 - 2e 65 78 61 6d 70 6c 65-2e 63 6f 6d 82 0f 77 77   .example.com..ww
0230 - 77 2e 65 78 61 6d 70 6c-65 2e 63 6f 6d 30 0a 06   w.example.com0..
0240 - 08 2a 81 1c cf 55 01 83-75 03 47 00 30 44 02 20   .*...U..u.G.0D. 
0250 - 09 f7 ca c6 5b 76 7b 4d-12 4e 62 42 a7 ae 1f 2c   ....[v{M.NbB...,
0260 - f7 f2 3b 8b 97 b2 4a 39-dd 60 6e d0 4c 61 5a da   ..;...J9.`n.LaZ.
0270 - 02 20 4e cc d1 80 34 4c-0d 42 d9 cd 85 39 12 c5   . N...4L.B...9..
0280 - 4f 3a 0d 04 25 bd 75 68-8e 60 b5 6a 86 60 65 84   O:..%.uh.`.j.`e.
0290 - 5a 68 16 03 03 00 95 0c-00 00 91 03 00 1e 41 04   Zh............A.
02a0 - 3c c4 84 28 59 aa 55 c8-2a eb 29 c6 90 cf b2 de   <..(Y.U.*.).....
02b0 - 96 43 28 71 09 39 37 0e-df 5e 64 92 c1 2d 42 44   .C(q.97..^d..-BD
02c0 - d9 f2 c5 4e 3f 51 4f 25-d0 5c d3 be 6a 88 c7 92   ...N?QO%.\..j...
02d0 - 92 ef 0c e3 ee 88 52 f7-86 f6 af 93 03 00 c9 3c   ......R........<
02e0 - 07 07 00 48 30 46 02 21-00 a5 10 2b f1 0a 02 e4   ...H0F.!...+....
02f0 - 6c 0c ad 0b aa 6a 1c 50-db c3 3f 74 4e 90 31 8f   l....j.P..?tN.1.
0300 - 07 d8 e5 ad 60 be f5 5c-ba 02 21 00 96 fd ae 5f   ....`..\..!...._
0310 - f2 36 d5 41 62 82 54 6b-18 e9 9f 8b f8 4a ed 24   .6.Ab.Tk.....J.$
0320 - 10 86 55 ab 40 5d d3 e4-1f 61 21 72 16 03 03 00   ..U.@]...a!r....
0330 - 04 0e                                             ..
0335 - <SPACES/NULS>
read from 0x2539a40 [0x253f4a3] (5 bytes => 5 (0x5))
0000 - 16 03 03 00 46                                    ....F
read from 0x2539a40 [0x253f4a8] (70 bytes => 70 (0x46))
0000 - 10 00 00 42 41 04 e1 62-08 b7 2f 94 e7 17 f9 cd   ...BA..b../.....
0010 - 69 2c a2 c7 ac b6 7f 42-7b bc d2 b6 14 8d 60 c9   i,.....B{.....`.
0020 - 3f ab b7 bc 0c c0 e7 88-83 5f af 24 5b f5 2b 07   ?........_.$[.+.
0030 - 71 57 98 c0 71 4f 8b 88-32 2f 0d e7 59 d9 b5 df   qW..qO..2/..Y...
0040 - 83 ec bc 90 75 64                                 ....ud
read from 0x2539a40 [0x253f4a3] (5 bytes => 5 (0x5))
0000 - 14 03 03 00 01                                    .....
read from 0x2539a40 [0x253f4a8] (1 bytes => 1 (0x1))
0000 - 01                                                .
read from 0x2539a40 [0x253f4a3] (5 bytes => 5 (0x5))
0000 - 16 03 03 00 28                                    ....(
read from 0x2539a40 [0x253f4a8] (40 bytes => 40 (0x28))
0000 - 7e 80 3e 0e f3 df 1f 62-f9 bd 08 90 1b 89 80 7e   ~.>....b.......~
0010 - 4a 5a 1d 3c 45 81 d5 d4-7b 14 79 0d 1b 94 df 20   JZ.<E...{.y.... 
0020 - 3b b3 82 0e e8 1f f4 0f-                          ;.......
write to 0x2539a40 [0x254dc20] (226 bytes => 226 (0xE2))
0000 - 16 03 03 00 aa 04 00 00-a6 00 00 1c 20 00 a0 f1   ............ ...
0010 - 9d 01 66 30 e9 b9 91 bf-ef d8 10 73 2a 52 00 5f   ..f0.......s*R._
0020 - 61 3a 86 ec 5f 96 04 0d-29 ac 51 4f d0 9c 4d 29   a:.._...).QO..M)
0030 - 4c 4b 8d 64 7c 80 f5 3d-4c 56 66 83 69 03 cb 1f   LK.d|..=LVf.i...
0040 - ce 39 4d b4 51 1c ba be-da 16 6a 34 f3 1b da 60   .9M.Q.....j4...`
0050 - 60 a3 1b 66 2a 7f 27 b3-82 8e 0c 88 6b 63 a0 72   `..f*.'.....kc.r
0060 - e7 be 0d 3f 8e 4a 46 cb-93 9a d6 97 2a 50 f7 17   ...?.JF.....*P..
0070 - f4 9d ae 89 85 35 0f 5c-d6 d0 ca e3 b5 72 40 96   .....5.\.....r@.
0080 - e8 80 dd 58 cf 68 5b ee-bb fc fb 0b 5b d6 27 a0   ...X.h[.....[.'.
0090 - fc 26 9b aa 45 b7 ec 34-19 7e a8 61 22 a8 16 a5   .&..E..4.~.a"...
00a0 - 3a 9d 10 d3 19 df 52 7e-83 41 c5 a1 88 ae 25 14   :.....R~.A....%.
00b0 - 03 03 00 01 01 16 03 03-00 28 59 c8 88 70 8e 48   .........(Y..p.H
00c0 - 9f ec b1 2d c6 5c 8c 72-cf 14 cd e5 ef 76 84 36   ...-.\.r.....v.6
00d0 - 2d 63 6d 63 0a 1a fe 17-51 91 af 96 9f 70 1e f3   -cmc....Q....p..
00e0 - 2b 5d                                             +]
```

### 2. 启动客户端

启动GmSSL s_client:

```
$ gmssl s_client -connect localhost:44330
```

输出如下：

```
CONNECTED(00000003)
depth=0 C = CN, ST = Hubei, L = Wuhan, O = mogoweb, OU = mogoweb, CN = mogoweb.com, emailAddress = mogoweb@gmail.com
verify error:num=18:self signed certificate
verify return:1
depth=0 C = CN, ST = Hubei, L = Wuhan, O = mogoweb, OU = mogoweb, CN = mogoweb.com, emailAddress = mogoweb@gmail.com
verify return:1
---
Certificate chain
 0 s:/C=CN/ST=Hubei/L=Wuhan/O=mogoweb/OU=mogoweb/CN=mogoweb.com/emailAddress=mogoweb@gmail.com
   i:/C=CN/ST=Hubei/L=Wuhan/O=mogoweb/OU=mogoweb/CN=mogoweb.com/emailAddress=mogoweb@gmail.com
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICOTCCAeCgAwIBAgIJANiuz15uFFZmMAoGCCqBHM9VAYN1MIGJMQswCQYDVQQG
EwJDTjEOMAwGA1UECAwFSHViZWkxDjAMBgNVBAcMBVd1aGFuMRAwDgYDVQQKDAdt
b2dvd2ViMRAwDgYDVQQLDAdtb2dvd2ViMRQwEgYDVQQDDAttb2dvd2ViLmNvbTEg
MB4GCSqGSIb3DQEJARYRbW9nb3dlYkBnbWFpbC5jb20wHhcNMjAwMzAyMDkwMDUy
WhcNMjEwMzAyMDkwMDUyWjCBiTELMAkGA1UEBhMCQ04xDjAMBgNVBAgMBUh1YmVp
MQ4wDAYDVQQHDAVXdWhhbjEQMA4GA1UECgwHbW9nb3dlYjEQMA4GA1UECwwHbW9n
b3dlYjEUMBIGA1UEAwwLbW9nb3dlYi5jb20xIDAeBgkqhkiG9w0BCQEWEW1vZ293
ZWJAZ21haWwuY29tMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAERMRJ5wL0/uIY
EkTOzNny8jYcQYVZIXxDnEf1+KWBGWbJPoma0O0ynt5Dt9zRr81AMZp25eokRwaf
IhQkM0/+0aMvMC0wKwYDVR0RBCQwIoIPd3d3LmV4YW1wbGUuY29tgg93d3cuZXhh
bXBsZS5jb20wCgYIKoEcz1UBg3UDRwAwRAIgCffKxlt2e00STmJCp64fLPfyO4uX
sko53WBu0ExhWtoCIE7M0YA0TA1C2c2FORLFTzoNBCW9dWiOYLVqhmBlhFpo
-----END CERTIFICATE-----
subject=/C=CN/ST=Hubei/L=Wuhan/O=mogoweb/OU=mogoweb/CN=mogoweb.com/emailAddress=mogoweb@gmail.com
issuer=/C=CN/ST=Hubei/L=Wuhan/O=mogoweb/OU=mogoweb/CN=mogoweb.com/emailAddress=mogoweb@gmail.com
---
No client certificate CA names sent
Peer signing digest: SM3
Server Temp Key: ECDH, SM2, 256 bits
---
SSL handshake has read 1047 bytes and written 322 bytes
Verification error: self signed certificate
---
New, TLSv1.2, Cipher is ECDHE-SM2-WITH-SMS4-GCM-SM3
Server public key is 256 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-SM2-WITH-SMS4-GCM-SM3
    Session-ID: C3B33D1A42F985178D56FAD7CABF28D80DFCBA556040CB22D8506B3806BD1779
    Session-ID-ctx: 
    Master-Key: 273C6E857837C94D979BEDDEC696357D5F5FB7F64ADD389B0D34BF29F8EE242881041C3E27E5B8E475B872D8CE3808FA
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 3a 32 4f 50 45 20 17 60-c1 22 13 eb a6 eb a7 63   :2OPE .`.".....c
    0010 - b0 fe d5 93 ac 51 6a 70-51 47 ef 4f 3d b4 ae 12   .....QjpQG.O=...
    0020 - 77 25 90 7f 94 0d 2b cf-88 15 06 2a 4e c6 71 27   w%....+....*N.q'
    0030 - f5 32 13 d9 16 fb 47 8e-9b 7c 1e 38 c1 31 45 8f   .2....G..|.8.1E.
    0040 - 3f ef 3c 12 95 3c 34 9d-06 fd 9c 55 79 24 21 ce   ?.<..<4....Uy$!.
    0050 - 57 8d bf d2 8c ec 4e 1a-61 b1 5a 17 6b 6d 05 12   W.....N.a.Z.km..
    0060 - 6f e3 e9 a2 40 0b ec 1c-87 59 3c 6a ff 4b 0d 09   o...@....Y<j.K..
    0070 - 5b f7 03 01 ff db 02 3d-42 ea ee c0 d3 28 17 f1   [......=B....(..
    0080 - a2 b2 37 13 0b dd a6 87-18 ad ae 15 21 23 3e 15   ..7.........!#>.
    0090 - ca 0a 17 d3 f8 e1 2b f0-6b 48 87 8b 84 de 40 76   ......+.kH....@v

    Start Time: 1583140976
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
    Extended master secret: yes
---
```

至此，客户端和服务器端的通信成功。因为 gmssl 命令有着超多的参数，一下子很难介绍完全，一般在使用的时候，有什么新的需求，再去翻手册。另外网上关于OpenSSL的资料也很多，通常情况下，OpenSSL的用法都可以照搬到GmSSL上。

如果大家有什么问题，欢迎交流。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)

