# 啊，原来申请数字证书这么简单

数字证书在公开密码算法中扮演着重要的角色，可用于数字签名、密钥协商、身份认证等等。如果要部署 HTTPS 网站，首先就要申请数字证书。

提到数字证书，人们可能会想到高大上的 CA中心、繁琐的申请流程、高昂的价格。虽然很多安全专家都警告 HTTP 的安全问题，呼吁全站采用 HTTPS，谷歌甚至在 Chrome 浏览器中将所有未部署 HTTPS 的网站都标记为不安全，但目前还有很多网站没有采用 HTTPS。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/let_us_encrypt_01.png)

这周在阿里云上申请了一个云主机，尝试着部署 HTTPS，发现其实申请证书并部署 HTTPS 超级简单。

首先需要澄清的是，数字证书并不是都需要花钱。Let's Encrypt是一个完全免费的CA机构，是个非赢利的组织。为了推动HTTPS网站的部署，截至2020年底，Let's Encrypt就已经为2.4亿网站签发了证书，影响力越来越大。Let's Encrypt 签发的证书兼容性非常好，大部分浏览器和操作系统都支持。

而且，Let's Encrypt设计了一套证书管理的标准协议，使得整个申请过程不需要人工干预，自动化完成。

下面就说说具体如何操作，云主机采用的操作系统是 Ubuntu 20.04。

1. 安装Certbot。

```
$ sudo snap install --classic certbot
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

2. 申请证书

```
$ sudo certbot --nginx
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): mogoweb@126.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
Account registered.
No names were found in your configuration files. Please enter in your domain
name(s) (comma and/or space separated)  (Enter 'c' to cancel): www.ilego.club
Requesting a certificate for www.ilego.club
Performing the following challenges:
http-01 challenge for www.ilego.club
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/default
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/default

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://www.ilego.club
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/www.ilego.club/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/www.ilego.club/privkey.pem
   Your certificate will expire on 2021-06-30. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again with the "certonly" option. To non-interactively
   renew *all* of your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

不敢相信，两个步骤就搞定。我试着访问我部署的网站 https://www.ilego.club ，成功了！

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/let_us_encrypt_02.png)

因为我的 WEB 服务器使用的是 nginx，所以 certbot 后面带的是 --nginx 参数。certbot 会自动找到 nginx 的配置文件，并对配置文件进行修改。查看了一下，配置文件主要增加了如下内容：

```
server {

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;
    server_name www.ilego.club; # managed by Certbot


        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        # pass PHP scripts to FastCGI server
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
        #       fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}


    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/www.ilego.club/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/www.ilego.club/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = www.ilego.club) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80 ;
        listen [::]:80 ;
    server_name www.ilego.club;
    return 404; # managed by Certbot


}
```

可以看到，不仅配置了 HTTPS，还将 HTTP 请求重定向到 HTTPS。而且Let's Encrypt证书默认有效期是90天，但certbot自动为系统增加了一个自动化任务，到期自动给原有证书续期（renew）。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/let_us_encrypt_03.png)

可能会有人疑惑，这么简单，安全性如何？

根据审核的宽松程度，数字证书可以分为三种类型，分别是DV证书、OV证书、EV证书。

* DV（Domain Validated）证书是最常见的一种证书类型，比如Let's Encrypt只会签发DV证书，申请证书的CSR请求会包含域名信息，CA机构获取CSR请求后，从中取出域名，校验域名的所有权，如果域名所有者就是证书申请者，代表身份审核通过，申请者有权申请该域名（包含子域名）对应的证书。

* OV（Organization Validated）证书，CA机构会对申请者的身份进行严格的审核，从而给用户（浏览器）提供更安全的信任。CA根据严格的标准会审核申请者身份，比如说审核申请者的企业资质、企业地址等消息，确保申请者的身份是真实的。

* EV（Extended Validation）证书，CA机构会对申请者的身份进行更严格的审核，对于CA机构来说，CA机构会严格根据CA/Browser论坛制定的标准审核申请者的身份，该标准称为Baseline Requirement标准，是由浏览器厂商、CA等机构创建的。

一般来说，DV证书更适合于个人、中小企业网站，企业和政府机构一般会申请OV证书，而银行、电商企业通常会申请EV证书。

好了，关于申请数字证书并为网站配置 HTTPS 就聊到这，有没有考虑也将自己个人或公司的网站也升级到 HTTPS ？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)