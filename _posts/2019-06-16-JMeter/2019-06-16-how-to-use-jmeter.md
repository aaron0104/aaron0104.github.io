---
layout: post
title:  "简单介绍JMeter使用"
categories: Test
tags: JMeter
author: frank
---

* content
{:toc}

本文简单介绍如何使用JMeter工具进行负载与性能测试。

环境安装与搭建不再介绍，本文所用到的环境如下：

jdk-8u211-windows-x64位  
jmeter5.1.1  
OS:windows 10 Enterprise  

### 添加线程组
右键TestPlan->Threads(Users)->Thread Group
![添加线程组](/assets/images/2019-06-17-jmeter/1.png)

解释：  
Number of Threads(users):虚拟用户数，此处设置线程数  
Ramp-Up Period(in seconds):准备时长；设置的线程数需要多久全部启动  
Loop Count:循环次数；假如设置了10个线程，循环次数为10，则每个线程发送请求10次，总request=10*10=100;选择Forever则直到取消或停止  
Scheduler:设置调度器 

![线程组详情](/assets/images/2019-06-17-jmeter/2.png)

### 添加HTTP Request
右键Thread Group->Add->Sampler->HTTP Request
![HTTP请求](/assets/images/2019-06-17-jmeter/3.png)

### 添加用户自定义变量
右键HTTP Request->Add->Config Element->User Defined Variables
![自定义变量](/assets/images/2019-06-17-jmeter/4.png)

### 添加结果树
右键HTTP Request->Add->Listener->View Results Tree
![结果树](/assets/images/2019-06-17-jmeter/5.png)

### 添加响应断言
右键HTTP Request->Add->Assertions->Response Assertion
![响应断言](/assets/images/2019-06-17-jmeter/6.png)

### 添加断言结果
右键HTTP Request->Add->Listener->Assertion Results
![断言结果](/assets/images/2019-06-17-jmeter/7.png)

### 添加聚合报告
右键HTTP Request->Add->Listener->Aggregate Report
![聚合报告](/assets/images/2019-06-17-jmeter/8.png)


报告参数解释  
Samples:请求数；值=模拟用户数*LoopCount  
Average:平均响应时间  
Median:50%用户响应时间  
90%Line:90%用户响应时间  
95%Line:同上类似  
99%Line:同上类似  
Min:最小响应时间  
Maximum:最大响应时间  
Error%:错误率=错误请求数/请求总数  
Throughput:吞吐量（request per second）
Received KB/sec:每秒接收  
Send KB/sec:每秒发送  
