---
layout: post
title:  "ubuntu18.04安装nginx并配置多站点"
categories: nginx
tags: nginx linux
author: frank
---

* content
{:toc}

### 安装nginx
环境为vm虚拟机，命令行下输入：  
{% highlight shell  %}
$ sudo apt install nginx
{% endhighlight %}

### 新建网站
在/var/www/目录下新建个2站点目录并创建首页index.html
{% highlight shell  %}
$ sudo mkdir /var/www/web1.com
$ sudo vim /var/www/web1.com/index.html
$ sudo cp -r /var/www/web1.com /var/www/web2.com
{% endhighlight %}

### 配置网站
ubuntu下网站配置文件在/etc/nginx/sites-available下，新建两个配置，名字最好与你的站点一样
{% highlight shell  %}
$ sudo cp -r /etc/nginx/sites-available/default /etc/nginx/sites-available/web1.com
$ sudo cp -r /etc/nginx/sites-available/default /etc/nginx/sites-available/web2.com
{% endhighlight %}
然后修改下配置内容，类似下面这样：  
　　![选择文件](/assets/images/2019-06-28/1.png)

### 建立软链接
{% highlight shell  %}
$ sudo rm /etc/nginx/sites-enabled/default      #删除原有的配置
$ sudo ln -s /etc/nginx/sites-available/web1.com /etc/nginx/sites-enabled/
$ sudo ln -s /etc/nginx/sites-available/web2.com /etc/nginx/sites-enabled/
{% endhighlight %}

测试配置是否正确，没有问题重启nginx服务
{% highlight shell  %}
$ sudo nginx -t
$ sudo service nginx restart
{% endhighlight %}

本机绑定hosts然后浏览器中访问，192.168.8.120为虚拟机IP
{% highlight text  %}
192.168.8.120   web1.com
192.168.8.120   web2.com
{% endhighlight %}