---
title: 'Nginx基础-1:安装部署'
date: 2020-06-03 20:25:34
tags:
- Nginx
categories: 
- Nginx
---

Nginx作为高性能的Web服务器，小巧、稳定、香。

官网文档链接：<http://nginx.org/en/docs/>

下面介绍两种操作系统下的安装部署。
<!-- more -->
## Windows

在<http://nginx.org/en/download.html>下载压缩包

![1568250404696](https://gitee.com/GeoDaoyu/PicGo/raw/master/blog/1568250404696.png)

解压后即可使用

配置在`nginx\conf`目录修改`nginx.conf`

**常用命令**：

~~~ shell
cd nginx         -- 进入到nginx解压文件的目录
start nginx      -- 启动
nginx -s reload  -- 重新装载配置文件并重启
nginx -s stop    -- 快速停止
nginx -s quit    -- 正常停止
~~~

## Linux

官网安装说明：<http://nginx.org/en/linux_packages.html#RHEL-CentOS>

执行下面两个命令进行安装

~~~ shell
sudo yum install yum-utils
sudo yum install nginx
~~~

安装后，cd到` /usr/sbin/`目录下启动nginx

~~~ shell
cd /usr/sbin
./nginx
~~~

配置在` /etc/nginx`目录下修改`nginx.conf`

**常用命令**：

```shell
cd nginx         -- 进入到nginx解压文件的目录
./nginx          -- 启动
nginx -s reload  -- 重新装载配置文件并重启
nginx -s stop    -- 快速停止
nginx -s quit    -- 正常停止
```

> 注：linux下如果启动失败，报错缺失日志文件，则在对应路径下新建日志文件。