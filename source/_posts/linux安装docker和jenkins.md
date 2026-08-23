---
title: linux安装docker和jenkins
date: 2024-01-30 11:33:27
tags:
  - Linux
  - Docker
  - Jenkins
categories:
  - Linux
---

## 背景

给一台新服务器安装Docker和Jenkins，做前端应用部署使用。

服务器是CentOS7，已经安装了Nginx，可以联网。

<!-- more -->

## 安装Docker

1.设置存储库

```shell
 sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

2.安装Docker

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

3.启动Docker

```shell
sudo systemctl start docker
```

4.验证Docker

```shell
sudo docker run hello-world
```

## 安装Jenkins

1.在Docker中安装Jenkins镜像

```shell
docker pull jenkins/jenkins
```

2.启动Jenkins

```shell
docker run -d -p 8080:8080 -p 50000:50000 -v /opt/apps/jenkins_home:/var/jenkins_home -v /opt/webs:/webs -e JENKINS_OPTS="--prefix=/jenkins" -e JENKINS_ARGS="--prefix=/jenkins" --name jenkins jenkins/jenkins
```

-d: 后台运行

-p: 端口映射，Jenkins使用8080和50000两个端口

--name: 别名

-v: 挂载文件，分别挂载jenkins_home和nginx部署文件夹

-e: 环境变量，jenkins的启动参数，给nginx代理用

jenkins/jenkins是镜像名称

3.Nginx代理Jenkins

在nginx配置中增加：

```conf
        # jenkins转发
        location /jenkins {
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header X-Forwarded-Host ps.geoscene.cn;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://192.168.190.50:8080/jenkins;
        }
```

4.验证Jenkins

访问页面：https://domain.com/jenkins

## 配置jenkins

1.管理员账号密码

admin/xxxxx

2.安装插件

插件中安装node、gitlab

全局配置中安装node多版本

3.新建流水线

## 运维

启动：

```shell
docker start jenkins
```

设置jenkins开机自启动：

1.docker自启动

```shell
systemctl enable docker.service
```

2.jenkins自启动

```shell
docker update --restart=always jenkins
```

设置nginx自启动

1.创建服务文件

```shell
sudo vim /etc/systemd/system/nginx.service
```

粘贴文本：

```text
[Unit]
Description=The Nginx HTTP and reverse proxy server
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

2.设置自启动

```shell
sudo systemctl enable nginx
```
