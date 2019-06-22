---
layout: post
title:  ".NET多线程"
categories: NET
tags: .NET 多线程
author: frank
---

* content
{:toc}

### 为什么需要多线程
设想以下情景：读取一个大的文件（1GB），然后对该文件进行后续处理，可能会像下图这样
![选择文件](/assets/images/2019-06-22-Thread/1.png)

为什么出现这样的情况呢？因为通常我们开发的windows程序都是单线程的，代码都是从上往下一行一行执行的，当遇到一个耗时操作时（比如加载大文件），后面的任务就要等待前面的操作完成才能继续，此时的状态为阻塞，程序因为在执行其他的操作而无法响应用户的另外请求。

### 如何解决
那如何解决呢？答案就是多线程；一个线程执行耗时操作，另一个线程继续响应用户的其他操作请求，这样也能得到更好的用户体验。

为了更好的合理利用计算机资源，提升程序性能；操作系统引入了线程这个概念。  
线程包含于进程，一个进程至少有一个线程（windows下），同一进程中的线程共享该进程的资源。

上面只是片面的讲了多线程的优点，其他优点就不在说明，下面给出例子如何使用多线程。

首先，用代码模拟一个计算机读取文件的任务  
{% highlight C#  %}
static void ReadFile()
{
    Console.WriteLine("开始读取文件...");
    for (int i = 0; i < 10; i++)
    {
        Thread.Sleep(500);
        Console.WriteLine("正在读取第" + (i + 1) + "行");
    }
    Console.WriteLine("读取文件完毕...");
}
{% endhighlight %}
再模拟一个播放音乐的任务
{% highlight C#  %}
static void Music()
{
    Console.WriteLine("开始播放音乐...");
    for (int i = 0; i < 10; i++)
    {
        Thread.Sleep(500);
        Console.WriteLine("正在播放第" + (i + 1) + "首音乐");
    }
    Console.WriteLine("音乐播放完毕...");
}
{% endhighlight %}

然后在Main中创建2个ThreadStart委托，并启动2个线程
{% highlight C#  %}
static void Main(string[] args)
{
    ThreadStart t1 = new ThreadStart(ReadFile);
    Thread thread1 = new Thread(t1);
    
    ThreadStart t2 = new ThreadStart(Music);
    Thread thread2 = new Thread(t2);

    //启动任务
    thread1.Start();
    thread2.Start();

    Console.Read();
}
{% endhighlight %}

这样，就完成了一个最简单的多线程例子；然后运行看下效果：  
　　![2](/assets/images/2019-06-22-Thread/2.png)

### 总结
虽然线程提高了性能与资源利用，但是并不是线程越多越好，过多的线程反而会启到反效果，因为线程的创建与销毁也是需要时间与空间的，在分时系统中，一个CPU核心同一时间只会执行一个线程，当该线程达到时间片后，系统就会给该CPU核心分配另外的线程，这样将导致线程切换，线程切换带来很严重的性能影响；所以要避免过多的线程和过于频繁的线程切换，合理的利用计算机硬件创建线程。