---
title: 'Nginx基础-5:SSL配置'
date: 2020-06-03 20:59:33
tags:
- Nginx
categories:
- Nginx
---

为了安全性，需要配置SSL。配置后就可以使用https来访问。

## 生成密钥

下载安装OpenSSL

然后进入到`OpenSSL-Win64\bin`目录下，执行exe文件。

看到命令行窗口。
<!-- more -->
1. 使用命令生成一个key文件
   需要输入一个密码：123456（以后会用到）

~~~ shell
genrsa -idea -out geodaoyu.key 1024
~~~

2. 使用命令导出一个csr文件
   需要输入个人信息

   ~~~ shell
   req -new -key geodaoyu.key -out geodaoyu.csr
   ~~~

3. 使用命令把key和csr打包成crt文件
   需要输入到第一步给的密码

   ~~~ shell
   x509 -req -days 3650 -in geodaoyu.csr -signkey geodaoyu.key -out geodaoyu.crt
   ~~~



发现这个key在window系统不能用，只好换个指令

4. 使用命令，一次性生成key和crt
   同样需要输入个人信息

   ~~~ shell
   req -days 3650 -x509 -sha256 -nodes -newkey rsa:1024 -keyout geodaoyu.key -out geodaoyu.crt
   ~~~

## 修改配置

修改`nginx.conf`中的server

```conf
server {
    listen       80;
    listen       443 ssl;
    server_name  geodaoyu.arcgisonline.cn;

    ssl_certificate      C:/Application/nginx-1.17.6/ssl/geodaoyu.crt;
    ssl_certificate_key  C:/Application/nginx-1.17.6/ssl/geodaoyu.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        root   html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

