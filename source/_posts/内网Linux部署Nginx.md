---
title: 内网Linux部署Nginx
date: 2023-11-14 16:56:29
tags:
- Nginx
- Linux
categories:
- Nginx
---

## 背景

某项目，在内网的统信（Linux）系统下通过编译安装的方式部署Nginx。

<!-- more -->

## 过程

### 下载

官网下载部署包

https://nginx.org/en/download.html

示例地址：`https://nginx.org/download/nginx-1.20.1.tar.gz`

### 解压

拷贝到服务器/opt下，进行解压，然后删除压缩包

``` shell
cd /opt
tar -zxvf nginx-1.20.1.tar.gz 
rm -f nginx-1.20.1.tar.gz
```

解压后，文件夹内容为：

``` text
- opt
  - nginx-1.20.1
```

### 编译

``` shell
cd nginx-1.20.1
./configure
make
make install
```

编译后，文件夹内容为：

``` text
- opt
  - nginx-1.20.1 (源码)
  - nginx (安装目录)
```

### 启动

通过`whereis nginx`指令可以查看装到哪儿了。

因为没有修改任何配置，所以默认情况下：

配置在/etc/nginx
启动在/usr/sbin/nginx,启动指令: ./nginx start,加载配置文件: ./nginx -s reload
