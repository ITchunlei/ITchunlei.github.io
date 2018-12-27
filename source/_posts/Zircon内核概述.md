---
title: Zircon内核概述
date: 2018/12/27
categories: 
- 操作系统
tags:
- 内核
- fuchsia
- zircon
---


### 简介

Zircon内核管理着许多不同类型的对象。众多实现了Dispatcher接口的C++ 类可以通过系统调用直接访问这些对象，这些C++的实现类都在kernel/object目录中。这些内核对象大部分是内核内部的高级别对象，此外也包括一些低级的lk使用的基础对象。

<!-- more -->

### 系统调用
用户空间代码是通过系统调用与内核对象进行交互的，
在用户空间中，句柄(Handle)是一个32位的整型数值(type zx_handle_t)。当执行系统调用时，内核将会检查被句柄(Handle)参数所引用的调用进程的handle table 中的实际handle。内核还将进一步检查Handle的正确类型，（）以及Handle是否具有相应请求权限。

从访问的角度来看，系统调用可以分为三大类：
1. 没有任何限制的调用，这些调用数量不多，例如：zx_clock_get()、zx_nanosleep(),这些是可以由任意线程中调用的。
