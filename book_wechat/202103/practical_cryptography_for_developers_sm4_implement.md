
# 写给开发人员的实用密码学 - 国密对称加密算法SM4实现要点

在上一篇文章《[写给开发人员的实用密码学 - 对称加密算法](https://mp.weixin.qq.com/s/hsQOA79Bk0oUDlBHVucvng)》中，介绍了现代密码学中非常重要的加密解密算法，国密标准中 SM4/SMS4 就是一种对称加密算法。本文将介绍密码库 libtomcrypt 库中增加 SM4 算法的一些关键点。

论名气，libtomcrypt 远远不及 OpenSSL、NSS、Bouncy Castle 等加密库，不过 libtomcrypt 也有一些优点：

* 代码采用 C 语言实现，可移植性好
* 模块化设计，易于裁剪，可以编译出很精简的库，比较适用于嵌入式设备、IoT设备
* 实现完备，支持目前几乎所有公开的加密算法
* 代码规模适中，没有使用过多的语法糖，代码直观易懂，比较容易上手

不过需要注意的是 libtomcrypt 并没有实现 SSL/TLS、SSH 等协议，仅仅是一个加密解密库。如果项目中需要 SSL/TLS 支持，这个库就不太适用。

#### 国密 SM4 的实现

国密SM4算法在文档《GM/T 0002-2012》中有详细的说明，只有几页纸，算法并不复杂。更简单的方法是参考 GmSSL 项目的实现代码，代码位于 GmSSL/crypto/sms4 下。主要算法实现文件是：

* sms4_common.c
* sms4_enc.c
* sms4_setkey.c

其它的实现，与分组模式有关，还有的针对特定 CPU 指令集进行优化。

算法实现时，一定要注意不同 CPU 上字节序的问题，比如 X86 芯片采用 little-endian，而 ARM 芯片采用 big-endian。如果需要优化加密解密速度，可使用汇编，针对特定 CPU 指令集进行优化。

#### libtomcrypt 增加 SM4 算法

libtomcrypt 并没有使用过多的设计技巧，对于加密算法的支持，都定义在 tomcrypt_cipher.h 头文件中。有关加密算法的结构为：

```c
struct ltc_cipher_descriptor {
   /** 密码算法名称 */
   const char *name;
   /** 算法内部ID */
   unsigned char ID;
   /** 密钥长度最小值（字节数） */
   int  min_key_length,
   /** 密钥长度最大值（字节数） */
        max_key_length,
   /** 分组大小（字节数） */
        block_length,
   /** 默认的轮次 */
        default_rounds;
   /** Setup the cipher
      @param key         The input symmetric key
      @param keylen      The length of the input key (octets)
      @param num_rounds  The requested number of rounds (0==default)
      @param skey        [out] The destination of the scheduled key
      @return CRYPT_OK if successful
   */
   int  (*setup)(const unsigned char *key, int keylen, int num_rounds, symmetric_key *skey);
   /** Encrypt a block
      @param pt      The plaintext
      @param ct      [out] The ciphertext
      @param skey    The scheduled key
      @return CRYPT_OK if successful
   */
   int (*ecb_encrypt)(const unsigned char *pt, unsigned char *ct, symmetric_key *skey);
   /** Decrypt a block
      @param ct      The ciphertext
      @param pt      [out] The plaintext
      @param skey    The scheduled key
      @return CRYPT_OK if successful
   */
   int (*ecb_decrypt)(const unsigned char *ct, unsigned char *pt, symmetric_key *skey);
   /** Test the block cipher
       @return CRYPT_OK if successful, CRYPT_NOP if self-testing has been disabled
   */
   int (*test)(void);

   /** Terminate the context
      @param skey    The scheduled key
   */
   void (*done)(symmetric_key *skey);

   /** Determine a key size
       @param keysize    [in/out] The size of the key desired and the suggested size
       @return CRYPT_OK if successful
   */
   int  (*keysize)(int *keysize);
   ...
};
```

添加一个算法进来，我们需要获得密钥长度、分组长度、并实现初始化、加密、解密、结束几个函数。其它的以 accel_ 开头的函数，在优化某种分组模式实现时有用，如果暂时不做这方面的工作，在填入结构数据时，给一个 NULL 值即可。

SM4 使用的是 128 位固定长度密钥，所以 min_key_length、max_key_length 的值都给16。分组长度为 128 位，所以 block_length 的值也是 16。default_rounds 指的是默认加密的轮次（可以想象为一个轮盘，转的次数不同，加密结果就不同）。因为 SM4 的加密算法于密钥扩展算法都采用 32 轮，所以这里的缺省轮次和 setup 函数中的 num_rounds 都不会用到。

SM4 的数据加密和数据解密，算法相同，只是轮密钥的使用顺序相反，解密轮密钥是加密轮密钥的逆序。所以 libtomcrypt 还有一个重要的数据结构 symmetric_key:

```c
typedef union Symmetric_key {
#ifdef LTC_DES
   struct des_key des;
   struct des3_key des3;
#endif
#ifdef LTC_RC2
   struct rc2_key rc2;
#endif
   ...

   void   *data;
} symmetric_key;
```

这是一个联合体，不同的加密算法，其内部密钥结构并不相同。注意，这个结构并不是持有的私钥（一串随机数），而是对用户私钥进行处理，然后用于加密算法的一种结构定义。下面是我为 SM4 算法定义的密钥结构：

```c
struct sm4_key {
   /* 加密和解密时会对密钥做不同的处理，分别保存处理结果 */
   ulong32 rk[SMS4_NUM_ROUNDS];    /* 用于加密 */
   ulong32 rk_d[SMS4_NUM_ROUNDS];  /* 用于解密 */
};
```

读过上一篇文章《[写给开发人员的实用密码学 - 对称加密算法](https://mp.weixin.qq.com/s/hsQOA79Bk0oUDlBHVucvng)》，我们就会知道，所谓分组密码算法，一次只加密一个分组的数据。SM4 的分组长度是 128 位，也就是一次只处理 16 个字节。在实际应用中，肯定不会只加密 16 个字节的数据，这就涉及到如何分组，如果数据字节数不是 16 的整数倍，还涉及到填充问题。

libtomcrypt 实现了多种分组模式，基本上涵盖了现有的主流分组模式，在 src/modes 下有许多子目录，分别对应着 CBC、CFB、CTR 等分组模式。

在定义了 SM4 算法结构后，还需要调用 register_cipher 注册，一个简单的方法是添加到 register_all_ciphers 函数中，程序在初始化过程中调用 register_all_ciphers，就能得到所有加密算法的支持。

#### 应用程序调用 SM4 实现

在实际应用中，我们需要调用 modes 下的分组函数，比如 ctr_start、ctr_encrypt、ctr_decrypt 等，然后由这些函数再去调用相应的加密算法。如果我们不需要关心什么分组模式、填充模式，那就需要在此基础上做进一步封装。

关于如何调用 libtomcrypt 进行数据加密，我们可以参考 demos/ltcrypt.c 的代码。

#### 小结

国密 SM4 是一种分组加密算法，有国家标准文档详细描述了算法，另外也可以参考开源实现，移植起来并不复杂。真正考验水平的是算法优化，特别是针对 CPU 指令进行优化，此外算法结合分组模式进行优化也是一个方向。

有兴趣的同学也可以参考我的移植代码：https://gitee.com/mogoweb/libtomcrypt-gm.git

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)
