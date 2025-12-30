---
title: HPCToolKit基础使用教程
copyright: true
donate: true
toc:
  enable: true
  number: true
  auto_open: true
comments: true
sticky: 0
warning:
  enable: false
  days: 180
date: 2022-12-30 11:02:27
updated: 2025-12-30 11:02:27
permalink:
keywords:
description:
summary:
categories:
tags:
cover:
top_img:
password:
---


## 安装

```bash
$ spack install hpctoolkit
```

## 性能数据采样

>基础教程推荐看 HPCToolKit 手册 第三章 Quick Start

主要有以下几个流程
1. *measurement* of context-sensitive performance metrics while an application executes;
2. *binary analysis* to recover program structure from CPU and GPU binaries;
3. *attribution* of performance metrics by correlating dynamic performance metrics with static program structure;
4. *presentation* of performance metrics and associated source code.

具体示意图如下：
![HPCToolKit基础使用教程-1.png](https://s2.loli.net/2023/02/09/2TrxnOcZ8RSsuiz.png)

### 1. Compiling an Application

在使用HPCToolKit之前，需要对于源代码编译加入调试信息，即使用 `-g -O3` 选项。但这并不是必须的，不加入 `-g` 选项的话，只是无法定位到相关接口所在的代码段。如果需要使用静态链接 (参考手册第6章：Monitoring Statically Linked Applications with hpclink) ，需要编译时加入如下代码：

```bash
$ hpclink mpicc -o app -static file.o ... -l ...
```

对于大部分用户说动态分析已经能够抓到性能瓶颈

### 2. Measuring Application Performance

* 对于动态链接的应用：

```bash
$ [mpi-launcher] hpcrun [hpcrun-options] app [app-arguments]
```

* 参数说明
	* `mpi-launcher` 即为你 mpi 的运行命令，例如 `mpirun -np 2`
	* `hpcrun-options` 即为 HPCToolKit 参数，具体形式如下
		* `-t -e event@howoften -e event@howoften ... app arg ...`
		* `-t` 记录 `tracing` ，在最终可视化可以看出每个进程不同时间下运行轨迹图
		* `-e event@howoften`
			* `event` 为设置要采样的指标，所有能采样的指标使用 `hpcrun -L`，或者查看手册第五章 (Monitoring Dynamically-linked Applications with hpcrun) 介绍常用参数
			* `howoften` 为采样频率，有两种设置方式：一种为 `f+number`，例如 `f100` 即为 100 samples/second，每秒采样100次；另一种直接为 `number` ，例如 `CPUTIME@5000` 即为 5,000 microseconds 采样一次，即 200 times/second
		* `app` 为运行的二进制程序
		* `arg` 为 `app` 运行的参数
	* Measuring GPU Computations：参考手册中第8章 (Measurement and Analysis of GPU-accelerated Applications)
 
* 对于静态链接的应用

唯一不同点在于使用环境变量设置采样指标

```bash
$ export HPCRUN_EVENT_LIST="event@howoften event@howoften"
$ [mpi-launcher] app [app-arguments]
```

运行完后会生成如下的文件夹：

```bash
hpctoolkit-app-measurements[-<jobid>]
```

jobid对应于作业提交id，需要在 Slurm, Cobalt, or IBM’s Job Manager 等作业调度系统中才会出现。在此文件夹内部又会存在对应于进程或者线程的文件

```bash
app-<mpi-rank>-<thread-id>-<host-id>-<process-id>.<generation-id>.hpcrun
app-<mpi-rank>-<thread-id>-<host-id>-<process-id>.<generation-id>.hpctrace
```

### 3. Recovering Program Structure

分析可执行程序的架构，包含 source les, procedures, inlined code, loop nests, and statements 等信息，自动使用多线程进行分析。对于一个可执行程序 b ，得到程序架构的命令如下：

```bash
$ hpcstruct b
```

会在当前工作目录生成 `b.hpcstruct`

### 4. Analyzing Measurements & Attributing Them to Source Code

将性能测试数据与代码结合：

```bash
$ hpcprof -I <app-src>/+ -S app.hpcstruct hpctoolkit-app-measurements1 [hpctoolkit-app-measurements2 ...]
# or
$ <mpi-launcher> hpcprof-mpi -I <app-src>/+ -S app.hpcstruct hpctoolkit-app-measurements1 [hpctoolkit-app-measurements2 ...]
```

* 参数说明
	* `hpcprof` 和 `hpcprof-mpi` 都可以灵活使用，后者对于大规模并行程序的性能数据提出的运行方式，会将所有性能指标对所有进程求和，前者有更多分析方式，后者能显示线程级的指标 (参考手册 10.6 Thread-level Metric Values) 。
	*  `app-src` 表示源码的文件夹， `-I/--include <app-src>/+` 表示递归的在  `app-src` 搜索
	* `app.hpcstruct` 表示刚刚生成的可执行程序的结构文件
* 最终会生成 `hpctoolkit-app-database` 文件夹，主要数据文件存在 `xml` 格式的文件中

### 5. Presenting Performance Measurements for Interactive Analysis

查看可视化的数据，Linux下有图形化界面的也可以使用如下命令：

```bash
hpcviewer hpctoolkit-app-database
```

具体如下界面数据展示：

![HPCToolKit基础使用教程-2.png](https://s2.loli.net/2023/02/09/VQfF74ixs5GCvKR.png)
有三种呈现方式，一种是 Top-down view，从顶部接口层层查看数据，另一种是 Bottom-up view，从底层接口看顶层调用，还有一种是从链接库看，这三种方式默认按指标大小排序呈现。对于图表呈现方式还有很多，包括派生新的性能指标，具体可以参考手册第10章 (Analyzing Performance Data with hpcviewer)。