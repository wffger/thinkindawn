---
weight: 7
keywords:
- Linux named pipe
title: "SHELL-命名管道模拟线程池"
date: 2023-06-14
lastmod: 2023-06-14
draft: false
authors: ["wffger"]
description: ""

tags: ["SHELL","Linux"]
categories: ["Linux"]
lightgallery: true
---

<!--more-->
# SHELL-命名管道模拟线程池

## 代码
```sh
# ping-ip-by-pipe.sh
thread=5
tmp_fifo=/tmp/$$.fifo
start_time=$(date +%s)
mkfifo $tmp_fifo
exec {tmp_fd}<> $tmp_fifo #关联一个临时fd到管道
rm -rf $tmp_fifo

for i in $(seq $thread)
do
  echo $i >&${tmp_fd} #放入限定数量的空行
done

for i in {1..30}
do
  read -u ${tmp_fd} #拿走一行，进入下一步，如果无内容会等待
  {
    sleep 1
    ip=192.168.5.$i
    ping -c1 -W1 $ip &>/dev/null
    if [ $? -eq 0 ];then
      echo "$ip is up."
    else
      echo "$ip is down."
    fi
    echo >&${tmp_fd}  #放入一行
  }&
done

wait

stop_time=$(date +%s)
echo "TIME:  $(($stop_time-$start_time))"
exec {tmp_fd}<&-
exec {tmp_fd}>&-

```

## 讲解
### 为什么不直接用命名管道$tmp_fifo？
> 因为在脚本中使用echo写入命名管道时，会等待一个读取管道的进程，无法进入下一步。

### 关闭fd
其实用下面的其中一个就可以关闭
```sh
exec {tmp_fd}<&-
exec {tmp_fd}>&-
```

## 参考

[shell编程(二十一)文件描述符](https://blog.csdn.net/wzj_110/article/details/121167608)