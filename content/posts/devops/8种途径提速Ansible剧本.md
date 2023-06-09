---
weight: 7
title: "8种途径提速Ansible剧本"
date: 2023-06-09
lastmod: 2023-06-09
draft: false
authors: ["wffger"]
description: ""

tags: ["Ansible","翻译"]
categories: ["DevOps"]

lightgallery: true
---

<!--more-->
# 8种途径提速Ansible剧本

> 原文 [8 ways to speed up your Ansible playbooks](https://www.redhat.com/sysadmin/faster-ansible-playbook-execution)

下面介绍一些提高剧本执行速度的优化方法。

## 1.使用回调插件定位慢任务

一个简单的任务也可能导致剧本执行缓慢。 
你可以启用回调插件，例如timer, profile_task, 和profile_roles，得到任务耗时以确定哪个作业拖慢剧本。

修改ansible.cfg以启用插件：  

```
[defaults]
inventory = ./hosts
callbacks_enabled = timer, profile_tasks, profile_roles
```

## 2.禁用事实收集功能
剧本每次执行都会运行一个隐藏的任务，叫作“收集事实”（gathering facts）,用到setup模块。
收集的信息与远程节点有关，详细信息存储在变量ansible_facts。
如果你不需要用到这些信息，可以禁用这个操作。`gather_facts: False`

## 3.配置并行
Ansible按批次执行任务，由变量forks控制。
默认值为5，也就是说每次在5台主机执行一个任务。
当前批次执行完毕再在另外5台主机执行相同任务。
当在所有主机完成当前任务，Ansible启动新的批次执行下一个任务。

修改ansible.cfg的并行数：

```
[defaults]
inventory = ./hosts
forks=50
```
你也可以使用--forks（-f）选项动态地修改并行数：

```
ansible-playbook site.yaml --forks 50

```

## 4.配置SSH优化项
建立SSH连接是一个相对缓慢的后台动作。
当你的剧本有大量任务和节点时，执行耗时会大幅提升。
你可以用ControlMaster和ControlPersist特性（ssh_connection节）减少影响。

* **ControlMaster** 允许在一个网络连接中使用多个同步的SSH会话。这会节省很多时间，因为后续SSH会话使用第一个SSH连接。
* **ControlMaster** 设置后台SSH空闲连接保活时长。

例子：

```
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

## 5. 在变化环境中禁用主机密钥检查
Ansible默认会检查验证SSH主机密钥，确保没有服务器欺诈和中间人攻击。
如果你的环境包含一些不可变节点（虚拟机或容器），这些主机重装或重建时密钥会变化。
你可以使用host_key_checking禁用主机密钥检查。

```
[defaults]
host_key_checking = False
```
不建议在不受控环境中使用这个配置。

## 6.使用管道技术
Ansible使用SSH时，后台会执行很多SSH操作，例如复制文件，复制脚本或者其他命令。
你可以在ansible.cfg中使用管道参数减少SSH连接数：

```
# ansible.cfg 
pipelining = True
```

## 7.使用执行策略
Ansible默认会等到所有主机都完成一个任务时，再执行下一个任务，这是线性策略。
如果任务之间，或者节点之间没有依赖关系。你可修改策略为free，这样Ansible会在一个主机上执行全部剧本任务，而不用等待别的主机完成任务。

```
- hosts: production servers
  strategy: free
  tasks:
```
你也可以用其他策略插件，例如[Mitogen](https://mitogen.networkgenomics.com/ansible_detailed.html)。

## 8.使用异步任务
执行一个任务时，Ansible在关闭和主机的连接之前会一直等待任务完成。
如果任务执行耗时较长（例如磁盘备份，安装软件等），这会成为一个瓶颈，因为它增加了全局的执行时间。
如果后续任务并不依赖当前的耗时任务，你可以使用async模式，搭配一个合适的poll间隔，告知Ansible无需等待，可以直接执行下一任务：

```
​​​​---
- name: Async Demo
  hosts: nodes
  tasks:
    
    - name: Initiate custom snapshot
      shell:
        "/opt/diskutils/snapshot.sh init"
      async: 120 # Maximum allowed time in Seconds
      poll: 05 # Polling Interval in Seconds
```

## 优化永不停步
Ansible剧本的全局执行时间由许多配置控制。
你可以根据你的基础设施特性找到满足你情景的配置参数最佳搭配。

这是一个不完整的优化列表。你可以使用许多其他参数来控制和优化剧本执行。
例如，serial, throttle, run_once等。
可以参考[playbooks_strategies](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_strategies.html)学习更多知识。


















































