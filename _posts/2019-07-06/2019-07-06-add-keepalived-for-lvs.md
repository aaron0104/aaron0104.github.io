---
layout: post
title:  "为LVS/DR集群负载增加keepalived"
categories: 集群 keepalived
tags: 集群 keepalived
author: frank
---

* content
{:toc}
 
### 前言
上一篇文章中搭建了lvs集群负载，通过内容请求分发技术提升整体的性能；但是假如其中一台realserver挂掉了，就会有部分用户访问到坏掉的服务器而影响正常的使用，那有没有办法自动移除坏掉的服务器呢？

当然有，那就是keepalived！它不但能自动移除坏掉的服务器，当服务器恢复后，还能自动再加到集群中继续向用户提供服务！  
keepalived是集群管理中保证集群高可用的一个服务软件，用来防止单点故障。

本篇实验环境  
ubuntu18.04LTS/keepalived1.3.9/ipvsadm1.28/ipvs1.2.1/nginx1.14  
前面已经介绍过lvs集群负载，本篇不再说明，只介绍keepalived的配置

### 安装并配置keepalived

在director机安装keepalived
{% highlight shell  %}
aaron@aaron:~$ sudo apt install keepalived
{% endhighlight %}

创建配置文件
{% highlight shell  %}
aaron@aaron:~$ sudo vi /etc/keepalived/keepalived.conf
{% endhighlight %}


编辑配置文件
{% highlight shell  %}
global_defs {                       
    router_id LVS_MASTER
    vrrp_skip_check_adv_addr
    vrrp_gna_interval 0
    vrrp_garp_interval 0
}
vrrp_instance VI_1 {            
        state MASTER          #LVS主节点 文章后面会再建一台备机，假如主机挂掉，自动切换到备机提供分发服务        
        interface ens34         #提供服务的网卡，来发送vrrp通告      
        virtual_router_id 52   #vrid
        priority 100                #节点优先级，优先级高的为master，备机请低于这个值      
        advert_int 1               #检查间隔，单位秒
        authentication {        
                auth_type PASS
                auth_pass 1111
        }
        virtual_ipaddress {         
                192.168.8.121   #VIP
        }
}
virtual_server 192.168.8.121 80 {
        delay_loop 6           
        lb_algo wrr         #权重轮询法
        lb_kind DR         #LVS/DR模式                 
        nat_mask 255.255.255.0   
        persistence_timeout 10    
        protocol TCP                          
        real_server 192.168.8.123 80 {    #realserver IP 
                weight 1        #权重             
                TCP_CHECK {                     
                        connect_timeout 10   
                        retry 3
                        delay_before_retry 3
                        connect_port 80
                }
        }
        real_server 192.168.8.124 80 {
                weight 1
                TCP_CHECK {
                        connect_timeout 10
                        retry 3
                        delay_before_retry 3
                        connect_port 80
                }
        }
}
{% endhighlight %}


重启keepalived服务
{% highlight shell  %}
aaron@aaron:~$ sudo service keepalived stop
aaron@aaron:~$ sudo service keepalived start
{% endhighlight %}

查看keepalived日志
{% highlight shell  %}
aaron@aaron:~$ cat /var/log/syslog | grep Keepalived
{% endhighlight %}
<img src="/assets/images/2019-07-06/2.png" alt="keepalived启动日志" style="cursor:zoom-in">

停掉其中一台realserver服务模拟故障，然后再查看keepalived日志
{% highlight shell  %}
aaron@aaron:~$ sudo service nginx stop
{% endhighlight %}

<img src="/assets/images/2019-07-06/2.png" alt="keepalived自动移除故障机" style="cursor:zoom-in">

keepalived检测到故障服务器并自动移除了故障机器

恢复服务，然后再查看keepalived日志
{% highlight shell  %}
aaron@aaron:~$ sudo service nginx start
{% endhighlight %}
<img src="/assets/images/2019-07-06/3.png" alt="keepalived自动加入" style="cursor:zoom-in">

### LVS备份机配置
因为是vm虚拟机，直接clone了director机并修改DIP为192.168.8.122  
同样安装keepalived并创建配置文件，内容如下
{% highlight shell  %}
global_defs {                       
    router_id LVS_DEVEL
    vrrp_skip_check_adv_addr
    vrrp_gna_interval 0
    vrrp_garp_interval 0
}
vrrp_instance VI_1 {            
        state BACKUP          #配置为备份机
        interface ens34         
        virtual_router_id 52  
        priority 100                
        advert_int 1              
        authentication {        
                auth_type PASS
                auth_pass 1111
        }
        virtual_ipaddress {         
                192.168.8.121   #VIP
        }
}
virtual_server 192.168.8.121 80 {
        delay_loop 6           
        lb_algo wrr         
        lb_kind DR        
        nat_mask 255.255.255.0   
        persistence_timeout 10    
        protocol TCP                          
        real_server 192.168.8.123 80 {   
                weight 1       
                TCP_CHECK {                     
                        connect_timeout 10   
                        retry 3
                        delay_before_retry 3
                        connect_port 80
                }
        }
        real_server 192.168.8.124 80 {
                weight 1
                TCP_CHECK {
                        connect_timeout 10
                        retry 3
                        delay_before_retry 3
                        connect_port 80
                }
        }
}
{% endhighlight %}

然后重启服务
{% highlight shell  %}
aaron@aaron:~$ sudo service keepalived restart
{% endhighlight %}
停掉主机keepalived服务
{% highlight shell  %}
aaron@aaron:~$ sudo service keepalived stop
{% endhighlight %}

查看备机日志
<img src="/assets/images/2019-07-06/4.png" alt="keepalived自动加入" style="cursor:zoom-in">

再启动主机keepalived服务
{% highlight shell  %}
aaron@aaron:~$ sudo service keepalived start
{% endhighlight %}
查看备机日志
<img src="/assets/images/2019-07-06/5.png" alt="keepalived自动加入" style="cursor:zoom-in">

### 小结
通过keepalived自动移除或添加故障节点，实现服务高可用，并且主备自动化管理，省心又省力。