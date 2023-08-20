---
weight: 7
keywords:
- linux ntp chrony
title: "chrony NTP配置"
date: 2023-08-20
lastmod: 2023-08-28
draft: false
authors: ["wffger"]
description: ""

tags: ["NTP"]
categories: ["Linux"]
lightgallery: true
---

<!--more-->
# chrony NTP配置
[How to configure chrony as an NTP client or server in Linux](https://www.redhat.com/sysadmin/chrony-time-services-linux)

精确计时确保可靠的网络通信，同时也保证系统组件的正常运行，例如systemd timer和cronjobs。计算机可以使用网络时间协议NTP来同步时间，精确的时间源可以是上游的时间服务器或者服务器池。<br />RHEL中，chronyd守护进程是默认的NTP客户端。同样，chronyd也可以配置为服务端用来作为时间源。<br />下面我们来介绍如何进行客户端和服务端的配置。
<a name="IxdhS"></a>
## 配置为客户端
默认配置使用NTP.org作为时间源。对于大多数家庭用户来说，这是可靠的时间源。但是在一些企业环境中，内部网络不允许NTP出站，所以这时候需要配置一个内部的时间源。<br />我们可以修改文件/etc/chrony.conf。我们有三种方式配置时间源，但通常我们使用server或者pool配置项。pool可以指向多个NTP服务器。<br />下面是一个例子，其中NTP服务器地址后面使用了iburst选项，可以使得前四次请求间隔缩短。具体用法可以参考[chrony – chrony.conf(5)](https://chrony-project.org/doc/3.4/chrony.conf.html)
```bash
server ntp.lab.int iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony
```
配置更新后，需要重启chronyd服务。最后检查时间源和系统时钟是否同步成功。
```bash
systemctl restart chronyd.service
systemctl is-active chronyd.service
chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* 3dhomejoe.org                 2   6    17     8    -65us[  +80us] +/-   27ms
^+ li1210-167.members.linod>     2   6    17     9    +11us[ +156us] +/-   48ms
^- 155.248.196.28                2   6    17     8   -384us[ -384us] +/-   19ms
^- static.36.62.78.5.client>     4   6    17     8    +51us[  +51us] +/-   31ms
timedatectl
               Local time: Sun 2023-08-20 12:00:17 CST
           Universal time: Sun 2023-08-20 04:00:17 UTC
                 RTC time: Sun 2023-08-20 04:00:18
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

<a name="LOZ2K"></a>
## 配置为服务端
我们使用allow指令指定一个子网，使得NTP客户端能通过子网访问NTP服务器。默认没有配置allow，说明客户端不能通过子网访问服务器，chronyd是单纯的客户端。<br />下面是一个例子，允许连接子网。
```bash
server ntp.lab.int iburst
allow 192.168.0.0/24
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony
```
更新配置后，重启chronyd服务，然后配置防火墙。
```bash
systemctl restart chronyd.service
firewall-cmd --add-service=ntp --permanent
firewall-cmd --reload
```
