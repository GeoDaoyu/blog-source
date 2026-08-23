---
title: 内网Linux安装neo4j
date: 2023-11-14 17:27:16
tags: 
- neo4j
- Linux
categories:
- Java
---
## 背景

某项目，在内网的统信（Linux）系统下安装neo4j。

neo4j是一个图数据库。安装需要Java环境（安装OpenJDK）。

<!-- more -->

## 过程

### 下载

官网下载社区版的部署包

https://neo4j.com/deployment-center/

### 解压

拷贝到服务器/opt下，进行解压，然后删除压缩包

``` shell
cd /opt
tar -zxvf neo4j-community-5.5.0-unix.tar.gz
rm -f neo4j-community-5.5.0-unix.tar.gz
```

解压后，文件夹内容为：

``` text
- opt
  - neo4j-community-5.5.0
```

### 环境变量

``` shell
vim /root/.bash_profile

# 最后添加
export PATH=/opt/neo4j-community-5.5.0/bin:$PATH

# 配置生效
source /root/.bash_profile
```

### 启动

``` shell
cd neo4j-community-5.5.0/bin/
neo4j start
```

访问http://localhost:7474，默认账号密码：neo4j/neo4j

第一次进入，需要修改默认密码。

## 坑

### ip监听

neo4j默认不开启ip地址监听，所以别的服务器访问不了neo4j的页面。

``` shell
vim neo4j.conf
```

在配置中，把`dbms.connectorss.default_listen_address=0.0.0.0`解除注释即开启监听。记得重启。

