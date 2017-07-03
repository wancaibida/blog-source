title: HPKP(HTTP Public Key Pinning)详解
author: Gang Chen
tags:
  - HPKP
categories:
  - Security
date: 2017-07-03 21:18:00
---
### HPKP是什么

HPKP(HTTP Public Key Pinning)又名公钥打孔,可以通过告知客户端将特定的加密公钥与特定服务器关联,以减少通过伪造证书进行中间人攻击(MITM)的风险.

### HPKP原理
HPKP是一种首次信任技术,当客户端第一次访问服务器的时候,服务器通过特定的HTTP头来告知客户端哪些公钥是属于它的,客户端会将该信息存储一段时间,当客户端第二次访问服务端的时候,它会期望当前证书链中至少有一个证书的公钥指纹与通过HPKP已知的公钥指纹相匹配,如果没有找到匹配的公钥指纹,客户端应该警告用户.

### 生成HPKP
公钥指纹可以通过openssl命令来成成:

```
#!/bin/sh
 
openssl s_client -connect [YOUR_DOMAIN]:443 -showcerts | awk '/-----BEGIN/{f="cert."(n++)} f{print>f} /-----END/{f=""}'
 
 for c in cert.*; do
   openssl x509 <$c -noout -pubkey | openssl rsa -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
 done
```
上面的命令会将当前证书链中的所有公钥指纹以base64格式打印出来.

公钥指纹也可以通过在线网站[REPORT URI](https://report-uri.io/home/pkp_hash)来生成 

### 备份密钥(Backup Key)
#### 什么是备份密钥
备份密钥是不属于当前证书链的公钥指纹

#### 为什么要备份密钥
假设你只对你的子证书(leaf certificate)做了公钥指纹,你发现你的证书私钥被泄露了,这时你不得不更换当前的证书,这时唯一能恢复的办法就是将当前证书切换到备份密钥指向的备份证书.

### 其他类似备份密钥的方法
备份密钥的缺点也很明显,你需要同时支付至少两份证书的钱.

另一个办法就是你可以拿另一家证书颁发机构(CA)的公钥指纹做为备份密钥,当你要更换证书的时候,你只需要在这家CA申请一份新的证书.当然这么做也有缺点,当这家CA被黑,攻击者可以给自己颁发一份你网站的证书同时又通过你的HPKP验证,应为这份恶意证书也在CA的证书链上.



MDN上建议网站要有一个备份密钥
> HPKP has the potential to lock out users for a long time if used incorrectly! The use of backup certificates and/or pinning the CA certificate is recommended.

但在Chrome(59.0.3071.115)下测试,如果HPKP头不包含备份密钥Chrome将忽略该头.

### 添加HPKP头
```
response.setHeader('Public-Key-Pins', 'pin-sha256="base64+primary=="; pin-sha256="base64+backup=="; max-age=5184000; includeSubDomains')
```

**sha256**
公钥的哈希值,base64编码

**max-age**
在浏览器的存储时间

**includeSubDomains**
对子域名是否有效

**report-uri**
验证失败后调用的URL(注:必须不同域名)

### 总结
用HPKP的网站比较少,就发现github在用.

HPKP缺点也很明显:出错的成本太高,当你不得不切换到备份密钥指向的证书时,如果备份密钥配错了而且你只有一个备份密钥,你的网站将会无法被访问,无法访问的时间取决于max-age的值,比如上面的例子max-age配了两个月,那你的用户将在两个月内无法访问你的网站,这后果是灾难性的.

个人认为HPKP带来的风险大于收益,不建议使用.

### 外部链接
[HTTP Public Key Pinning (HPKP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Public_Key_Pinning)

[HTTP PUBLIC-KEY-PINNING EXPLAINED
](https://timtaubert.de/blog/2014/10/http-public-key-pinning-explained/)

[Guidance on setting up HPKP
](https://scotthelme.co.uk/guidance-on-setting-up-hpkp/)

[Is HTTP Public Key Pinning Dead?](https://blog.qualys.com/ssllabs/2016/09/06/is-http-public-key-pinning-dead)



