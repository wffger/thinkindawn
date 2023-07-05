---
weight: 7
keywords:
- python multithreading
title: "python并发与并行"
date: 2023-06-16
lastmod: 2023-06-16
draft: false
authors: ["wffger"]
description: ""

tags: ["multithreading"]
categories: ["Python"]
lightgallery: true
---

<!--more-->
# python并发与并行
## 概念理解
* 进程（process）：操作系统资源分配的基本单位。
* 线程（thread）：任务调度和执行的基本单位。
* 并发（concurrency）：把任务在不同的时间点交给处理器进行处理。在同一时间点，任务并不会同时运行。
* 并行（parallelism）：把每一个任务分配给每一个处理器独立完成。在同一时间点，任务一定是同时运行。

注意，[python官方文档](https://docs.python.org/zh-cn/3.11/library/concurrency.html)，目录结构中“并发”下面包含了“并行”。
> IO密集型任务使用多线程编程。  
> CPU密集型任务使用多进程编程。

{{< image src="https://cdn.hashnode.com/res/hashnode/image/upload/v1605981951100/fkhGPsoMP.png?auto=compress,format&format=webp" caption="process and thread" >}}
{{< image src="https://miro.medium.com/v2/resize:fit:640/format:webp/1*_nSpk2ZTATQpps0XOHJHGA.png" caption="Processes run in parallel. Threads execute concurrently.">}}


---

## GIL
[全局解释器锁（global interpreter lock）](https://docs.python.org/zh-cn/3.11/glossary.html#term-global-interpreter-lock)是CPython采用的一种互斥锁机制，目的是为了线程安全。
GIL限制了线程使用多个CPU的能力，你使用threading模块创建多线程时，实际上多线程是并发执行而不是并行执行的。

解决并行能力的相关提案：
* [PEP 684](https://peps.python.org/pep-0684/)
* [PEP 703](https://peps.python.org/pep-0703/)

---

## 使用情景
线程占用CPU，CPU是用来计算的。
CPU执行任务时，我们尽量让它进行计算操作，如果任务中有IO操作，我们让它去执行别的任务中的计算操作。
IO瓶颈一般为存储设备或网络等待导致。

### 测试计算密集情景
```python
# comp_cpu.py
import time
from threading import Thread
from multiprocessing import Pool

COUNT = 600000

def count_cpu(n):
    while n>0:
        n -= 1

start = time.time()
count_cpu(COUNT)
end = time.time()
print("Single threads:\n  Time taken in seconds -", end - start)

t1 = Thread(target=count_cpu, args=(COUNT//2,))
t2 = Thread(target=count_cpu, args=(COUNT//2,))
# t1 = Thread(target=count_cpu, args=(COUNT,))
# t2 = Thread(target=count_cpu, args=(COUNT,))
start = time.time()
t1.start()
t2.start()
t2.join()
end = time.time()
print("Two threads:\n  Time taken in seconds -", end - start)

pool = Pool(processes=2)
start = time.time()
r1 = pool.apply_async(count_cpu, [COUNT//2])
r2 = pool.apply_async(count_cpu, [COUNT//2])
pool.close()
pool.join()
end = time.time()
print("Two processes:\n  Time taken in seconds -", end - start)

```

**结果**
```
Single threads:
  Time taken in seconds - 0.030620098114013672
Two threads:
  Time taken in seconds - 0.03354072570800781
Two processes:
  Time taken in seconds - 0.019550323486328125
```
两个线程执行count_io，效率无提升。  
两个进程执行count_io，耗时有时长，有时短，效率不稳定。

### 测试IO密集情景
```python
# comp_io.py
import time
from threading import Thread
from multiprocessing import Pool


COUNT = 6000

def count_io(n):
    while n>0:
        time.sleep(0.001)
        n -= 1

start = time.time()
count_io(COUNT)
end = time.time()
print("Single threads:\n  Time taken in seconds -", end - start)

t1 = Thread(target=count_io, args=(COUNT//2,))
t2 = Thread(target=count_io, args=(COUNT//2,))
# t1 = Thread(target=count_io, args=(COUNT,))
# t2 = Thread(target=count_io, args=(COUNT,))
start = time.time()
t1.start()
t2.start()
t2.join()
end = time.time()
print("Two threads:\n  Time taken in seconds -", end - start)

pool = Pool(processes=2)
start = time.time()
r1 = pool.apply_async(count_io, [COUNT//2])
r2 = pool.apply_async(count_io, [COUNT//2])
pool.close()
pool.join()
end = time.time()
print("Two processes:\n  Time taken in seconds -", end - start)

```

**结果**
```
Single threads:
  Time taken in seconds - 6.363478899002075
Two threads:
  Time taken in seconds - 3.1789000034332275
Two processes:
  Time taken in seconds - 3.1876704692840576
```
两个线程或两个进程执行count_io，效率都提升了。


---

## 参考
[Understanding Processes, Threads and CPU Cores](https://kishoreconnect.com/understanding-processes-threads-and-cpu-cores)
[Multithreading vs. Multiprocessing in Python](https://towardsdatascience.com/multithreading-vs-multiprocessing-in-python-3afeb73e105f)
[Real Multithreading is Coming to Python - Learn How You Can Use It Now](https://martinheinz.dev/blog/97)