---
title: 内网Linux安装OpenJDK
date: 2023-11-14 17:11:40
tags:
- OpenJDK
- Linux
categories:
- Java
---

## 背景

某项目，在内网的统信（Linux）系统下安装OpenJDK。

<!-- more -->

## 过程

### 下载

官网下载部署包

https://jdk.java.net/archive/

示例地址：`https://download.java.net/java/ga/jdk11/openjdk-11_linux-x64_bin.tar.gz`

### 解压

拷贝到服务器/opt下，进行解压，然后删除压缩包

``` shell
cd /opt
tar -zxvf openjdk-11_linux-x64_bin.tar.gz
rm -f openjdk-11_linux-x64_bin.tar.gz
```

解压后，文件夹内容为：

``` text
- opt
  - jdk11
```

### 环境变量

``` shell
vim /root/.bash_profile

# 最后添加
export PATH=/opt/jdk11/bin:$PATH

# 配置生效
source /root/.bash_profile
```

### 验证

``` shell
java --version
```

输出OpenJDK的信息。
