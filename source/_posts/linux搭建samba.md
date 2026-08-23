---
title: linux搭建samba
date: 2024-01-30 11:27:24
tags: 
- Linux 
categories: 
- Linux
---
## 背景

Samba是在Linux系统上实现SMB协议的一个免费软件。SMB（Server Messages Block，信息服务块）是一种在局域网上共享文件和打印机的一种通信协议，它为局域网内的不同计算机之间提供文件及打印机等资源的共享服务。

需求是给linux服务器安装samba，共享文件夹给windows桌面机。
<!-- more -->
## 安装samba

1.安装

``` shell
yum -y install samba-*
```

2.创建用户

``` shell
# 添加系统用户
useradd zhanggy
# 添加系统用户为samba用户，并设置密码
smbpasswd -a zhanggy
```

3.修改配置文件

``` shell
vim /etc/samba/smb.conf
```

在最后增加：

``` text
[zhanggyshare]
   # 我们要分享的文件夹路径
   path = /opt/projects/webs
   # 是否允许浏览
   browseable = yes
   # 是否可写
   writable = yes
   # 是否允许匿名(guest)访问,等同于public
   guest ok = yes
   # 客户端上传文件的默认权限
   create mask = 0777
   # 客户端创建目录的默认权限
   # 注意共享文件在系统本地的权限不能低于以上设置的共享权限。
   directory mask = 0777
```

记得修改共享文件夹的权限

``` shell
chmod 777 -R /opt/projects/webs
```

4.启动服务

``` shell
# 启动
sudo systemctl start smb nmb
# 重启
sudo systemctl restart smb nmb
# 停止
sudo systemctl stop smb nmb
```

## 验证

在windows上，打开文件管理器，访问`\\192.168.190.85\zhanggyshare`，输入之前注册的账号密码。

能浏览、拷出、写入、删除。