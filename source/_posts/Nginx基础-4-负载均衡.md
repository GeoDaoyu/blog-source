---
title: 'Nginx基础-4:负载均衡'
date: 2020-06-03 20:51:00
tags:
- Nginx
categories:
- Nginx
---

负载均衡是为减轻单个服务器负担，将请求按照某种策略分发给多个服务器。

官方文档参考：<http://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream>

以2台Tomcat集群为例
<!-- more -->
## Tomcat配置

Tomcat下载地址：<https://tomcat.apache.org/download-80.cgi>

![1568258710917](https://gitee.com/GeoDaoyu/PicGo/raw/master/blog/1568258710917.png)

解压后修改`\tomcat\conf\`目录下的server.xml

修改**所有的**端口号，避免冲突

~~~ xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" 
/>
~~~

本例中，Tomcat A部署在`http://localhost:8080`，Tomcat B部署在`http://localhost:8082`

分别在两个Tomcat下部署`tomcat\webapps\demo\load-balance.html`

~~~ html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>

<body>
  <h1 style="color: pink">8080</h1>
</body>

</html>
~~~

用颜色和文字区分两个Tomcat

## Nginx配置

~~~ conf
upstream tomcat {
    server localhost:8080;
    server localhost:8082;
}

server {
    listen       8088;
    server_name  localhost;

    charset utf-8;

    location / {
        proxy_pass   http://tomcat;
        proxy_redirect default;

        include proxy_params;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
~~~

进入浏览器，用`http://localhost:8088/demo/load-balance.html`进行访问

F5刷新页面，会依次地访问到8080和8082上的Tomcat

## 分发策略

### 1.轮询（默认）

~~~ conf
upstream tomcat {
    server localhost:8080;
    server localhost:8082;
}
~~~

按时间顺序，依次分发

### 2.加权轮询

```conf
upstream tomcat {
    server localhost:8080 weight=1;
    server localhost:8082 weight=10;
}
```

weight值与访问几率成正比

### 3.ip_hash

```conf
upstream tomcat {
	ip_hash;
    server localhost:8080;
    server localhost:8082;
}
```

考虑缓存的情况，保持会话

来自同一ip的请求会固定到一个server

### 4.最小连接

```conf
upstream tomcat {
	least_conn；
    server localhost:8080;
    server localhost:8082;
}
```

根据连接数来分发，起到均分的作用

### 5.url_hash

```conf
upstream tomcat {
	hash $request_url;
    server localhost:8080;
    server localhost:8082;
}
```

来自同一url的请求会固定到一个server

## 状态参数

### 1.down

```conf
upstream tomcat {
    server localhost:8080 down;
    server localhost:8082;
}
```

当前server不参与负载均衡

### 2.backup

```conf
upstream tomcat {
    server localhost:8080 down;
    server localhost:8082;
}
```

预留的server，当有其他的server挂掉或者忙的时候，才会参与负载

### 3.max_fails和fail_timeout

```conf
upstream tomcat {
    server localhost:8080;
    server localhost:8082 max_fails=3 fail_timeout=30s;
}
```

最大连接错误次数3次，每次失败后，间隔30s再重新发起连接

### 4.max_conns

```conf
upstream tomcat {
    server localhost:8080;
    server localhost:8082 max_conns=500;
}
```

超过最大连接数后，就不参与负载