---
title: 'Jmeter 体验之旅'
date: 2019-11-12 19:23:40
tags: [Jmeter]
published: true
hideInList: false
feature: /post-images/jmeter-ti-yan-zhi-lu.png
---
Jmeter 是一款测试工具，主要用来进行压力测试，今天和测试的同事在聊这个话题，顺便就体验了一下Jmeter进行压力测试，下面我就简单描述一下今天的体验吧。
<!-- more -->
## Jmeter 介绍
Apache JMeter 是一款开源的Java开发的用于进行软件性能测试的软件。
## 安装和运行
下载地址：http://mirrors.tuna.tsinghua.edu.cn/apache//jmeter/binaries/apache-jmeter-5.2.zip
运行：windows环境： “bin/jmeter.bat”   linux环境： “bin/jmeter.sh”   
切换语言： 启动Jmeter 后， 点击 Options -> Choose Language -> Chinese[Simplified]  来选择简体中文
截图如下：
![Jmeter](https://bofengsun.github.io//post-images/1573559262706.png)
## 案例
### 添加默认设置
1、添加 HTTP Cookie管理器
![HTTP Cookie管理器](https://bofengsun.github.io//post-images/1573563560952.png)
2、添加 默认请求头
![默认请求头](https://bofengsun.github.io//post-images/1573563633762.png)
3、添加 默认请求
![默认请求](https://bofengsun.github.io//post-images/1573563677239.png)
### 添加线程组
参数说明：
    线程数：表示需要启动的线程数量
    Ramp-Up时间：多少秒内启动这些线程，也就是说会在这个时间内启动这么多线程，如果设置为0，则表示同时启动
    循环次数：表示重复运行多少次，永远表示一直重复直到时间结束
    调度器配置：在配置调度器时生效，持续时间：就是执行多久
![添加线程组](https://bofengsun.github.io//post-images/1573563731549.png)
### 添加定时器
![添加定时器](https://bofengsun.github.io//post-images/1573609185253.png)
### 添加事务控制器
![事务控制器](https://bofengsun.github.io//post-images/1573563817861.png)
### 添加请求
![添加请求](https://bofengsun.github.io//post-images/1573609251077.png)
### 添加断言
![添加断言](https://bofengsun.github.io//post-images/1573609303540.png)
### 添加查看结果树
![添加查看结果树](https://bofengsun.github.io//post-images/1573609390929.png)
### 添加聚合报告
![添加聚合报告](https://bofengsun.github.io//post-images/1573609486454.png)
