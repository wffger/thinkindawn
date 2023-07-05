---
weight: 7
keywords:
- goroutine channel
title: "golang限制并发数"
date: 2023-07-05
lastmod: 2023-07-05
draft: false
authors: ["wffger"]
description: ""

tags: ["multithreading", "concurrence"]
categories: ["Golang"]
lightgallery: true
---

<!--more-->
# golang限制并发数

## 情景
在多个服务器上执行部署作业，每个作业有多个任务。  
限制并发的作业数量。  
每个作业串行完成全部任务，然后启动下一个作业。  

## 代码
```go
package main

import (
	"flag"
	"fmt"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

type Server struct {
	Name string
}

func deploy(serverChan chan Server, tasks []string, stop <-chan bool) {
	for {
		select {
		case <-stop:
			fmt.Println("退出运行")
			return
		case server := <-serverChan:
			for _, task := range tasks {
				fmt.Printf("%v: %v\n", server.Name, task)
				time.Sleep(1 * time.Second)
			}
			return
		default:
			fmt.Println("正在运行……")
		}
	}

}

func waitForSignal() {
	sigs := make(chan os.Signal)
	signal.Notify(sigs, os.Interrupt)
	signal.Notify(sigs, syscall.SIGTERM)
	<-sigs
}

func main() {
	maxRoutines := flag.Int("maxRoutines", 2, "并发数")
	servers := []Server{
		{Name: "NIFI01"},
		{Name: "NIFI02"},
		{Name: "NIFI03"},
		{Name: "NIFI04"},
	}
	totalJobs := flag.Int("totalJobs", len(servers), "作业总数")
	tasks := []string{"01-[deploy app]", "02-[update config]", "03-[run app]", "04-[clean backup]"}

	flag.Parse()
	concurrentChan := make(chan struct{}, *maxRoutines)
	serverChan := make(chan Server)
	stop := make(chan bool)
	var wg sync.WaitGroup

	for i := 1; i <= *totalJobs; i++ {
		// for i, server := range servers {
		wg.Add(1)
		go func(i int, stop <-chan bool) {
			defer wg.Done()
			concurrentChan <- struct{}{}
			fmt.Printf("作业%v正在运行\n", i)
			deploy(serverChan, tasks, stop)
			fmt.Printf("作业%v已经完成\n", i)
			<-concurrentChan
		}(i, stop)
	}

	for _, server := range servers {
		serverChan <- server
	}
	// waitForSignal()
	// close(serverChan)
	// close(stop)
	// fmt.Println("停止所有任务！")
	wg.Wait()
}
```

## 待续
需要接收中断信号，停止goroutine。  
虽然目前不调用waitForSignal也可以使用Ctrl+C中断进程，但是我认为会存在问题。 

## 参考
深入golang之---goroutine并发控制与通信  
[Limit the maximum number of goroutines running at the same time](https://gist.github.com/AntoineAugusti/80e99edfe205baf7a094)  
[Select waits on a group of channels](https://yourbasic.org/golang/select-explained/)  
