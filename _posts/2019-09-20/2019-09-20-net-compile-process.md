---
layout: post
title:  ".NET编译过程"
categories: .NET
tags: .NET
author: frank
---

* content
{:toc}


开始正文前先提两个问题  
1：asp.net从编写完成到运行，经历了哪些过程？  
2：为什么在初次运行时速度很慢？

### 前言
大家应该都知道，CPU只能直接执行机器指令，而机器指令是由二进制代码组成的；但是.NET应用通常被编译器编译为IL，微软称为Intermediate Languag（中间语言），IL被封装在程序集（exe/dll）中，很显然CPU并不能直接执行IL语言，所以这就需要借助JIT编译器。

### JIT编译流程
JIT（Just In Time），全称即时编译器，负责在方法首次执行前将IL翻译为本机代码（native code），这个过程发生在runtime中；[这里](https://github.com/dotnet/coreclr/blob/master/Documentation/botr/ryujit-overview.md)有一篇介绍JIT编译器的文档，有兴趣的可以去看一看，假如你比较懒，又不想翻译，[这里](https://www.52pojie.cn/thread-1005018-1-1.html)还有一篇JIT脱壳文章，里面大致讲了JIT编译流程。

### 总结
在运行时，JIT编译器将IL翻译为本机代码（native code），再由CPU执行本机码，本机代码会被缓存，这也是应用在初次运行时速度很慢的原因，因为又经历了一次编译。

所以，.NET应用从编码到运行，实际上一共经历了2次编译；一次为编译器（如csc.exe)编译为IL，一次在运行时由JIT编译为native code。