<!--
 * @Author: 陈正勇
 * @Date: 2021-02-22 10:54:30
 * @LastEditTime: 2021-02-22 11:17:38
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: /mywritings/book_wechat/202102/practical_cryptography_for_developers_asymmetric.md
-->
# 写给开发人员的实用密码学 - 非对称加密算法

经过一个漫长的年假后，恢复了一些元气，接着写一写加密解密相关的内容。

在上一篇文章《[写给开发人员的实用密码学 - 对称加密算法]()》中，介绍了现代密码学中非常重要的加密解密算法。从工程学的角度，选取密钥足够长的加密算法（比如AES 256、AES 512），是无法破解的。但在对称加密算法中，存在明显的薄弱环节，那就是密钥的存储与分发。因为算法是公开的，那么决定加密系统是否安全的因素就是密钥。