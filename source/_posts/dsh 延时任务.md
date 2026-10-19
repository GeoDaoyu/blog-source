---
title: dsh 延时任务
date: 2026-10-19 12:00:00
tags:
- dsh
- deepseek
- harness
- 定时任务
categories:
- 工具
---
DeepSeek 的 token 计费分峰时和谷时，谷时价格只有峰时的一半。

我的工作大多不紧急，所以一直在把执行时间往谷时挪。

最后用的是 dsh 的自动化任务。
<!-- more -->
## 背景

需求是不紧急的活都放到谷时跑。

这些活不着急，晚几个小时没有影响。

官方[价目表](https://api-docs.deepseek.com/zh-cn/quick_start/pricing/)里，工作日（不含法定节假日）9 点到 12 点、14 点到 18 点是峰时。

其余时段是谷时。

12 点和下午 6 点刚好是谷时的起点。

## 手动

最早的方案是手动。

先把提示词写好，到 12 点或者 6 点，自己执行。

执行完去吃饭，回来再看任务进度，决定要不要操作。

这个方案能省钱，但到点人得在。

不在，任务就往后拖。

## 插件

后来在 GitHub 上找过几个 dsh 的定时插件，比如 [dsh-timer](https://github.com/life1996cou/dsh-timer)、[dsh-timer-scheduler](https://github.com/GMH13552/dsh-timer-scheduler)、[dsh-timer-agent](https://github.com/LouisHaoL/dsh-timer-agent)。

star 数最多的是 13 星，另外两个各 1 星，且都一两个月没提交。

star 数和活跃度都不值得信任，所以没装。

## 自动化任务

10 月 17 日，[dsh](https://deepseek-harness.github.io/deepseek-harness/guide/quickstart) 升级到 0.2.1-rc.2，新增了自动化任务。

它允许用自然语言描述一个自动化任务或定时任务，比如：

> 12 点执行某某任务。

现在的方案就是直接在 dsh 里写延时任务。

不用装插件，用一句话描述就行。
