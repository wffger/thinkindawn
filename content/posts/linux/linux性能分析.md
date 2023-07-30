---
weight: 7
keywords:
- linux performance analysis
title: "linux性能分析"
date: 2023-07-28
lastmod: 2023-07-28
draft: false
authors: ["wffger"]
description: ""

tags: ["performance"]
categories: ["Linux"]
lightgallery: true
---

<!--more-->
# linux性能分析

## 原文

[linux performance analysis in 60s](https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55)

## 概览
```
uptime
dmesg -wT
vmstat 1
mpstat -P ALL 1
pidstat 1
iostat -xz 1
free -m
sar -n DEV 1
sar -n TCP,ETCP 1
top
```

### uptime
```
 12:21:36 up 2 days,  1:16,  2 users,  load average: 1.90, 1.97, 1.72

```
每列内容为：
1. 当前时间
2. 运行时长
3. 活跃的用户会话数
4. 最近1分钟平均负载
5. 最近5分钟平均负载
6. 最近15分钟平均负载

### [dmesg](https://man7.org/linux/man-pages/man1/dmesg.1.html)
```
[五 7月 28 12:00:40 2023] PM: resume devices took 0.869 seconds
[五 7月 28 12:00:40 2023] OOM killer enabled.
[五 7月 28 12:00:40 2023] Restarting tasks ... 
[五 7月 28 12:00:40 2023] Bluetooth: hci0: RTL: examining hci_ver=0a hci_rev=000c lmp_ver=0a lmp_subver=8822
[五 7月 28 12:00:40 2023] Bluetooth: hci0: RTL: rom_version status=0 version=3
[五 7月 28 12:00:40 2023] Bluetooth: hci0: RTL: loading rtl_bt/rtl8822cu_fw.bin
[五 7月 28 12:00:40 2023] Bluetooth: hci0: RTL: loading rtl_bt/rtl8822cu_config.bin
[五 7月 28 12:00:40 2023] Bluetooth: hci0: RTL: cfg_sz 6, total sz 35990
[五 7月 28 12:00:40 2023] done.
[五 7月 28 12:00:40 2023] random: crng reseeded on system resumption
[五 7月 28 12:00:40 2023] usb 1-4.1: USB disconnect, device number 7
[五 7月 28 12:00:40 2023] PM: suspend exit
```

我们可以通过过滤关键词来排查故障，例如：
1. Out of memory
2. Possible SYN flooding

## [vmstat](https://man7.org/linux/man-pages/man8/vmstat.8.html)
查看虚拟内存统计信息
```
 vmstat  1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0   1096 452348     64 9127268    0    0    77   380  477    6 15  6 79  0  0
 0  0   1096 452424     64 9127604    0    0   816 23704 3682 4651  2  2 96  0  0
 0  0   1096 452172     64 9127652    0    0     0     4 3135 4306  2  1 97  0  0
 0  0   1096 451916     64 9127652    0    0     0     0 2982 4220  2  1 97  0  0
 0  0   1096 451920     64 9127684    0    0     0     8 3072 4489  3  1 96  0  0
 0  0   1096 451444     64 9127700    0    0     0     0 2720 3675  2  1 97  0  0
 0  0   1096 450968     64 9127788    0    0    16   544 2853 4009  2  1 96  0  0

```
###字段解析
  **Procs**
  r大于CPU总数说明CPU工作饱和。
  - r: The number of runnable processes (running or waiting for run time).
  - b: The number of processes blocked waiting for I/O to complete.

  **Memory**
    These are affected by the --unit option.
  - swpd: the amount of swap memory used.
  - free: the amount of idle memory.
  - buff: the amount of memory used as buffers.
  - cache: the amount of memory used as cache.
  - inact: the amount of inactive memory.  (-a option)
  - active: the amount of active memory.  (-a option)

  **Swap**
    These are affected by the --unit option.
    大于零说明内存不够用。
  - si: Amount of memory swapped in from disk (/s).
  - so: Amount of memory swapped to disk (/s).
  

  **IO**
  - bi: Kibibyte received from a block device (KiB/s).
  - bo: Kibibyte sent to a block device (KiB/s).

  **System**
  - in: The number of interrupts per second, including the clock.
  - cs: The number of context switches per second.

  **CPU**
    These are percentages of total CPU time.
  - us: Time spent running non-kernel code.  (user time, including nice time)
  - sy: Time spent running kernel code.  (system time)
  - id: Time spent idle.  Prior to Linux 2.5.41, this includes IO-wait time.
  - wa: Time spent waiting for IO.  Prior to Linux 2.5.41, included in idle.
  - st: Time stolen from a virtual machine.  Prior to Linux 2.6.11, unknown.
  - gu: Time spent running KVM guest code (guest time, including guest nice).


## [mpstate](https://man7.org/linux/man-pages/man1/mpstat.1.html)
查看每个处理器的统计信息。可以检查计算负载是否均衡。如果单个处理器繁忙，可证明单线程应用有问题。

```
10时56分36秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
10时56分41秒  all    1.80    0.00    0.65    0.40    0.20    0.18    0.00    0.00    0.00   96.77
10时56分41秒    0    2.40    0.00    1.00    1.00    0.60    0.40    0.00    0.00    0.00   94.59
10时56分41秒    1    0.00    0.00    0.00    1.00    0.20    0.20    0.00    0.00    0.00   98.60
10时56分41秒    2    2.81    0.00    1.00    0.20    0.20    0.20    0.00    0.00    0.00   95.59
10时56分41秒    3    1.20    0.00    0.40    0.00    0.20    0.00    0.00    0.00    0.00   98.19
10时56分41秒    4    2.21    0.00    0.80    0.00    0.20    0.20    0.00    0.00    0.00   96.58
10时56分41秒    5    3.61    0.00    0.80    0.00    0.00    0.20    0.00    0.00    0.00   95.39
10时56分41秒    6    1.61    0.00    0.60    1.00    0.20    0.20    0.00    0.00    0.00   96.39
10时56分41秒    7    0.60    0.00    0.60    0.00    0.00    0.00    0.00    0.00    0.00   98.80
```

## [pidstat](https://man7.org/linux/man-pages/man1/pidstat.1.html)
报告任务统计信息。
可以记录一段时间内的信息汇总。通过对比不同时间点的数据，定位问题。

```
11时02分25秒   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
11时02分30秒     0        72    0.00    0.20    0.00    0.00    0.20     2  khugepaged
11时02分30秒   998       825    0.20    0.00    0.00    0.00    0.20     6  systemd-oomd
11时02分30秒  1000      1161    0.00    0.20    0.00    0.00    0.20     4  appimagelaunche
11时02分30秒  1000      1271    0.59    0.79    0.00    0.00    1.39     4  gnome-shell
11时02分30秒     0      1404    0.20    0.00    0.00    0.00    0.20     1  filebeat
11时02分30秒     0      1439    0.20    0.00    0.00    0.00    0.20     1  metricbeat
11时02分30秒     0      1451    1.98    0.00    0.00    0.00    1.98     1  filebeat
11时02分30秒  1000      2528    1.19    0.20    0.00    0.00    1.39     5  chrome
11时02分30秒  1000      2535    0.00    0.20    0.00    0.00    0.20     4  cgroupify
11时02分30秒  1000      2580    1.58    0.79    0.00    0.00    2.38     1  chrome
11时02分30秒  1000      2581    0.40    0.00    0.00    0.00    0.40     4  chrome
11时02分30秒  1000      3369    0.20    0.00    0.00    0.00    0.20     5  chrome
11时02分30秒  1000      8560    0.00    0.20    0.00    0.00    0.20     1  chrome
11时02分30秒  1000     10667    0.00    0.20    0.00    0.00    0.20     2  wpscloudsvr
11时02分30秒  1000     16152    0.20    0.00    0.00    0.00    0.20     0  chrome
11时02分30秒  1000     18185    3.76    0.59    0.00    0.00    4.36     6  chrome
11时02分30秒  1000     19039    0.00    0.20    0.00    0.00    0.20     6  terminator
11时02分30秒  1000     19733    1.39    0.59    0.00    0.00    1.98     1  code
11时02分30秒  1000     19780    0.79    0.59    0.00    0.00    1.39     0  code
11时02分30秒  1000     21019    0.20    0.00    0.00    0.00    0.20     6  code
11时02分30秒     0     42910    0.00    0.20    0.00    0.00    0.20     4  kworker/u32:7-btrfs-endio-write
11时02分30秒     0     43109    0.00    0.20    0.00    0.00    0.20     0  kworker/u32:35-btrfs-endio-write
11时02分30秒     0     43113    0.00    0.20    0.00    0.00    0.20     4  kworker/u32:39-events_unbound
11时02分30秒     0     43205    0.99    0.20    0.00    0.00    1.19     3  sssd_kcm
11时02分30秒  1000     44092    0.20    0.00    0.00    0.00    0.20     1  chrome
11时02分30秒  1000     44127    3.96    0.59    0.00    0.00    4.55     4  chrome
11时02分30秒  1000     44202    0.20    0.00    0.00    0.00    0.20     3  chrome
11时02分30秒  1000     44665    0.20    0.00    0.00    0.00    0.20     7  chrome
11时02分30秒  1000     44930    0.00    0.20    0.00    0.00    0.20     2  pidstat
```

## [iostat](https://man7.org/linux/man-pages/man1/iostat.1.html)
报告中央处理器的信息，以及设备和分区的输入输出信息。  
如果设备是一个逻辑盘，100%的利用率只是说明一直有IO活动，并不表明物理盘饱和。  
IO的性能饱和不一定表明应用有性能问题，因为很多应用设计都会考虑异步IO处理，应用并不会因IO性能低下直接导致严重的延迟问题（例如，预读，缓存写入）。

```
iostat -xz 1
Linux 6.4.4-200.fc38.x86_64 (ydx) 	2023年07月30日 	_x86_64_	(8 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          14.26    0.05    6.24    0.08    0.00   79.37

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          1.36     27.03     0.01   0.90    0.43    19.89    8.43    211.88     2.09  19.87    2.60    25.15    0.54    732.67     0.00   0.00    0.62  1359.96    0.18    0.41    0.02   0.66
zram0            0.00      0.00     0.00   0.00    0.00    21.53    0.00      0.00     0.00   0.00    0.00     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00

```

## [free](https://man7.org/linux/man-pages/man1/free.1.html)


**buffers**: For the buffer cache, used for block device I/O.
**cached**: For the page cache, used by file systems.
```
free -m
               total        used        free      shared  buff/cache   available
Mem:           14907        5477         861         270        8568        8827
Swap:           8191           0        8191
```

## [sar](https://man7.org/linux/man-pages/man1/sar.1.html)
收集、报告和保存系统活动信息。

### sar -n DEV 1
下面命令报告网络设备信息。
-n选项指的是network，关键词DEV指的是device。该命令可以查看网卡的流量信息。
```
sar -n DEV 1
Linux 6.4.4-200.fc38.x86_64 (ydx) 	2023年07月30日 	_x86_64_	(8 CPU)

11时21分54秒     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
11时21分55秒        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11时21分55秒    wlp2s0     16.00     17.00      6.65      3.67      0.00      0.00      0.00      0.00
11时21分55秒    virbr0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

11时21分55秒     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
11时21分56秒        lo      2.00      2.00      0.10      0.10      0.00      0.00      0.00      0.00
11时21分56秒    wlp2s0      7.00      6.00      0.53      0.55      0.00      0.00      0.00      0.00
11时21分56秒    virbr0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

11时21分56秒     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
11时21分57秒        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11时21分57秒    wlp2s0      7.00      7.00      1.03      0.77      0.00      0.00      0.00      0.00
11时21分57秒    virbr0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
^C


平均时间:     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
平均时间:        lo      0.67      0.67      0.03      0.03      0.00      0.00      0.00      0.00
平均时间:    wlp2s0     10.00     10.00      2.74      1.66      0.00      0.00      0.00      0.00
平均时间:    virbr0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

### sar -n TCP,ETCP 1
命令解析：
**TCP**：TCPv4网络流量
**ETCP**：TCPv4网络错误
结果解析：
**active/s**: 本地发起的每秒TCP连接数。Number of locally-initiated TCP connections per second (e.g., via connect()).
**passive/s**: 远程发起的每秒TCP连接数。Number of remotely-initiated TCP connections per second (e.g., via accept()).
**retrans/s**: 每秒TCP重传数。Number of TCP retransmits per second.
我们可以把主动数和被动数近似看作出站数和入站数。这个不严谨，因为存在一些本地与本地的连接。

重传数可以作为信号，表明网络或服务器的问题，一般由不可靠的网络引起（例如公网波动）。或者服务器过载导致丢包。


```
sar -n TCP,ETCP 1
Linux 6.4.4-200.fc38.x86_64 (ydx) 	2023年07月30日 	_x86_64_	(8 CPU)

11时38分09秒  active/s passive/s    iseg/s    oseg/s
11时38分10秒      0.00      0.00      0.00      0.00

11时38分09秒  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
11时38分10秒      0.00      0.00      0.00      0.00      0.00

11时38分10秒  active/s passive/s    iseg/s    oseg/s
11时38分11秒      0.00      0.00      2.00      2.00

11时38分10秒  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
11时38分11秒      0.00      0.00      0.00      0.00      0.00

11时38分11秒  active/s passive/s    iseg/s    oseg/s
11时38分12秒      0.00      0.00      1.00      1.00

11时38分11秒  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
11时38分12秒      0.00      0.00      0.00      0.00      0.00
^C


平均时间:  active/s passive/s    iseg/s    oseg/s
平均时间:      0.00      0.00      1.00      1.00

平均时间:  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
平均时间:      0.00      0.00      0.00      0.00      0.00
```
## [top](https://man7.org/linux/man-pages/man1/top.1.html)
如果top的输出与前面的命令输出有很明显的不同，表明引起问题的负载一直在变化。我们需要用vmstat或者pidstat保存更具体的信息以便定位问题。


## 延伸
<iframe src="//player.bilibili.com/player.html?aid=828931031&bvid=BV1Nu4y12759&cid=1211316586&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>