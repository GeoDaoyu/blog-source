---
title: 使用Jenkins代码构建
date: 2020-07-06 21:25:19
tags:
- Jenkins
- CI&CD
categories:
- Jenkins
---
以前构建部署，都是手工操作。先在本机编译打包，然后拷贝到服务器上做部署。

频繁的构建部署，费时烦心，所以学习使用Jenkins来实现自动化。


> Jenkins是开源CI&CD软件领导者， 提供超过1000个插件来支持构建、部署、自动化， 满足任何项目的需要。

[官网](https://www.jenkins.io/)的介绍和文档写的也很详尽了，下面我简单介绍下流程。

本文针对是Windows。Linux的环境时，一般都是内网，无法通过Git拉取更新。
<!-- more -->
## 1 环境准备

+ Java

Jenkins 是基于 Java 的独立程序，所以需要Java环境，这里不细说。

+ Node

主要是前端项目，所以需要Node。

+ Git

使用Git做版本控制。

+ Gradle环境

后端如果使用Maven，则需要配置Gradle环境。

## 2 下载安装

官网提供了两种方式，一种是war包，一种是msi安装包。

安装包的实质也是war包启动。所以推荐使用war包。

启动的两种方式：

一是把jenkins.war放到Tomcat下，作为项目启动。

二是通过命令行`java -jar jenkins.war`来启动。

## 3 Jenkins配置

### 3.1 端口号

默认端口号是8080，为了避免冲突，一般是需要修改的。

如果是Tomcat启动，则修改Tomcat的端口号。

如果是`java -jar jenkins.war`启动，修改根目录jenkins.xml文件

### 3.2 初始化

浏览器输入`http://localhost:8080/jenkins`可以进入到Jenkins页面。

然后是引导式的配置，需要配置一些如管理员账号，安装常用插件等。

### 3.3 项目构建配置

初始化完成后可以进入到Jenkins主页面，新建任务。

#### 3.3.1 源码管理

设置源码git地址和git账号，默认分支是master。

#### 3.3.2 构建器触发

有多种触发方式，简单介绍两种：

+ Poll SCM

定时器，间隔时间拉取git，如果有新代码则触发构建。

+ GitLab Hook

通过钩子，每次有代码推送，就触发构建。

需要在GitLab开启Webhooks，在Jenkins中安装 GitLab Hook插件。

> 修改jenkins配置，在jenkins的 Config System 功能中，取消 Enable authentication for ‘/project’ end-point 的选中状

### 3.4 构建

#### 3.4.1 npm构建

流程：git拉取->npm打包->替换部署文件

示例Shell脚本代码

``` powershell
git pull
npm install
npm run build
Remove-Item C:\Projects\xxx -Recurse -Force -Confirm:$false
Copy-Item dist C:\Projects\xxx -Recurse
```

#### 3.4.2 gradle构建

流程：git拉取->gradel打jar包->停止jar包->替换文件->启动新的jar包

### 3.5 通知

构建结果可以通知，常见有邮箱通知和钉钉群通知。

> Jenkins需要安装钉钉插件

## 4 使用总结

配置一次项目自动构建大概需要大半天时间。

然后后面就不怎么用管它了。

我理解的自动构建的实质就是，在服务器上放了一个"金老头"，然后把帮你把你手动操作的构建部署做一遍。

如果构建失败，可以在Jenkins页面查看日志，排查问题。但是一般直接重新构建就能成功。

