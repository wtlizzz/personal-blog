title: 使用https
author: Wtli
tags: []
categories: []
date: 2021-07-19 16:13:00
---
博客之前是挂在github上的，速度感人，现在改挂在国内服务器上，配置https。

1. 证书申请
2. 证书绑定域名
3. 证书配置
4. 配置nginx的https

<!--more-->

#### 证书申请

首选免费。

在阿里云上申请一年有效期的免费证书（证书基本都要每年申请）。

地址：https://common-buy.aliyun.com/?spm=5176.13785142.commonbuy2container.9.2cd6778bWUDkX0&commodityCode=cas_dv_public_cn&request=%7B%22ord_time%22:%221:Year%22,%22order_num%22:1,%22product%22:%22free_product%22,%22certCount%22:%2220%22%7D

<font color='grey'>
附：每个实名主体个人/企业，一个自然年内可以领取一次数量为20的免费证书资源包。免费资源包到自然年结束时，会自动清除未签发的数量（每个自然年12月31日24:00）</font>

点立即购买，即获得证书。

进入管理界面：

![upload successful](/images/pasted-96.png)

需要手动创建证书，然后点击右方的证书申请。

在这里可以看到整个流程：

1. 购买证书
2. 申请证书
3. 域名所有权验证
4. 签发证书
5. 下载证书

#### 证书绑定域名

我的域名是在阿里云上的，直接在阿里云上配置申请。

![upload successful](/images/pasted-97.png)

在申请的过程中绑定域名，然后提交。

#### 证书配置

在域名处配置，我的是阿里云的域名，就直接自动生成了域名解析配置。其中记录值是证书申请的时候生成的。

![upload successful](/images/pasted-98.png)


配置完后，下载证书，然后配置nginx服务器。

证书下载后会有一个压缩包，里面两个文件，分别是

![upload successful](/images/pasted-99.png)

这两个文件都需要在服务器中的nginx配置。

#### 配置nginx

检查服务器中的nginx有没有编译https。

```
[root@VM-8-14-centos sbin]# ./nginx -V
nginx version: nginx/1.9.9
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

如果configure arguments后面没有，则需要重新变异nginx。

找到下载的nginx的安装包。如果找不到或者需要重新配置nginx可以参考[链接](https://wtlizzz.com/%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%83%A8%E7%BD%B2/#%E9%85%8D%E7%BD%AEnginx)安装。

否则会报错：

```
[root@VM-8-14-centos sbin]# ./nginx
nginx: [emerg] the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf:99
```

安装完nginx，之后配置nginx的配置文件：

所在位置：/usr/local/nginx/conf/nginx.conf

我在服务器中配置了两个web，一个是80端口，一个是443端口，分别对应的是http连接，一个https连接，配置如下。

配置https时，需要注意在配置文件中配置证书的密钥的地址。

```
    server {
        listen       443 ssl;
        server_name  localhost;

        ssl_certificate      /aircas/ssl/5977718_www.wtlizzz.com.pem;
        ssl_certificate_key  /aircas/ssl/5977718_www.wtlizzz.com.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            root   /aircas/blog/public;
            index  index.html index.htm;
        }
    }
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /aircas/dist;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html; 
        }
        #error_page  404              /404.html;
    }
```

到此https就配置完了，如果有时间会专门分析https证书不同，加密方式不同，价钱为什么差距这么大。

