---
title: 给网站配置ssl证书
date: 2020-01-18 11:42:12
tags:
- 建站
comments: true
description: 之前Godaddy的域名居然过期了，重新去阿里云买了一个，顺便把ssl证书配置好了，记录一下
---
寒假里本来想写点博客，惊觉Godaddy的域名过期了，赎回来的流程一场复杂且昂贵，索性去阿里云重新买一个域名，意外发现只需50rmb，不到godaddy的一半，只是备案大约需要十几天，但还是可以接受，顺便把ssl证书也搞好了，毕竟之前的`Not Secure`字样令人非常不愉悦。

## HTTPS
众所周知，HTTPS就是HTTP over SSL/TLS。可以简单了解一下SSL/TLS的机制原理。SSL/TLS是为了防止第三方能够获取通信的内容并加以篡改等问题。早期国内的通信商就会搞这样的操作。包括之前小米路由器劫持404页面也是这么个情况。当然，HTTPS实现的细节非常多，里面涉及到的每个相关加密算法也并不简单，所以只是简述一下大体的过程。

首先client会发出请求，也就`Client Hello`，它包含了client支持的协议，加密方法等内容，以及一个28 bytes的随机数。随后server会进行回应，也就是`Server Hello`，它会包含确认使用的加密通信协议版本，加密方法，方便之后连接的session id等内容，同样也会生成一个28 bytes的随机数，以及Certificate也就是证书。之后client会根据证书内容进行判断，证书是否是可信任机构颁布，证书中域名与实际域名是否相符合。没有问题的话，就会从证书中提取出公钥（这里涉及到非对称加密的相关内容，不具体展开）。同时，还会生成一个48bytes的pre-master key，使用公钥加密后发给server。现在server和client各自都拥有三个随机key了，利用这3个key生成真正的`会话密钥`后，后续的所有会话都是使用这一密钥来进行加密，因为对称加密的效率要远远高于非对称加密。至于为什么要三个随机数而不是客户端直接生成一个来进行加密，是因为SSL协议并不信任每个主机都能产生完全随机的随机数，所以三个伪随机数就会非常接近随机了。据说直接产生的随机数在1996年时大概25秒就能被破解。

## 获取证书
如果像我一样是直接在阿里云买的域名，可以直接去阿里云那边[免费购买](https://common-buy.aliyun.com/?spm=5176.7968328.1266638.1.6d431232cyeS5X&commodityCode=cas#/buy)一个证书就好，选个人那个选项就行。

购买完成后在控制台申请验证，大概几分钟后就会通过，然后就能够下载下来了。如果你有多个子域名就多买几个。

## 配置证书
验证成功后下载下来即可。之后需要去云服务器那边配置一下。如果像我一样使用的nginx代理，可以放到`/etc/nginx/cert`目录下。随后去修改下nginx的配置文件如下：
```
server {
        listen 443 ssl;                                    
        server_name your_domain_name.com;
        root /your_websites_dir;
        index index.html index.htm index.nginx-debian.html;
        location / {
                try_files $uri $uri/ =404;
        }
        ssl_certificate   /etc/nginx/cert/xxx.pem;
        ssl_certificate_key  /etc/nginx/cert/xxx.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;                                                                                    
        ssl_prefer_server_ciphers on;      
}
server {
        listen 80;
        server_name your_domain_name.com;
        return      301 https://$server_name$request_uri; 
}
```
可以看到我这边有两个server项，一个监听443的加密端口，并在其中制定了证书的位置以及相关的加密算法和协议。而底下那个80端口则是重定向到443端口，这样即使输入的是http也会被重定向到https。

配置完成后，重新加载nginx就好啦：
```
$ nginx -s reload
```

## 参考
* 阮一峰, [SSL/TLS协议运行机制的概述](https://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)
* Juff Moser, [The First Few Milliseconds of an HTTPS Connection](http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html)
* [阿里云文档](https://help.aliyun.com/document_detail/28535.html?spm=5176.7968328.1120764.1.6d431232drKpoL)