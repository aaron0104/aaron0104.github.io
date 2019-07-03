---
layout: post
title:  "搭建LVS/DR集群负载均衡"
categories: 集群
tags: LVS 集群
author: frank
---

* content
{:toc}


### DR模式特点
director与realserver必须处于同一个网络。  
director负责处理客户端请求，realserver直接响应客户端请求。  
无法像NAT模式一样使用端口映射    
具体介绍请求转到这篇文章：http://www.linuxvirtualserver.org/zh/lvs3.html

下面是体系结构图，原图来源于上面网站地址  
![图片1](/assets/images/2019-07-02/3.png)

### 实验准备
director: ens34:192.168.8.120 , VIP ens34:0 192.168.121 , gatway:192.168.8.1  
realserver节点1： 192.168.8.123 , gatway:192.168.8.1  
realserver节点2： 192.168.8.124 , gatway:192.168.8.1  
三台实验机均为ubuntu18.04LTS 64位，vm虚拟机，网络模式为host-only

### 配置节点机
设置节点机的环回地址，并忽略arp请求（因为分发机与realserver群共享vip），只让分发机响应用户请求  

![图片1](/assets/images/2019-07-02/1.png)

安装nginx并新建测试站点，具体不在详述  
![图片2](/assets/images/2019-07-02/2.png)


### Director设置
安装lvs管理工具
{% highlight shell  %}
$ sudo apt install ipvsadm
{% endhighlight %}

{% highlight shell  %}
$ sudo ipvsadm -C      #清空ipvsadm规则
$ sudo ipvsadm -A -t 192.168.8.121:80 -s wlc
$ sudo ipvsadm -a -t 192.168.8.121:80 -r 192.168.8.123 -g  -w 1   #-w设置权重
$ sudo ipvsadm -a -t 192.168.8.121:80 -r 192.168.8.124 -g  -w 1   
{% endhighlight %}

设置虚拟IP地址并配置静态路由
{% highlight shell  %}
$ sudo ifconfig ens34:0 192.168.8.121 broadcast 192.168.8.121 netmask 255.255.255.0 up
$ sudo route add -host 192.168.8.121 dev ens34:0
{% endhighlight %}

### 测试
在浏览器中输入http://192.168.8.121

![图片3](/assets/images/2019-07-02/4.png)
![图片4](/assets/images/2019-07-02/5.png)

