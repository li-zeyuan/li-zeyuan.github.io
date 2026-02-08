---
title: goroutine泄露排查
date: 2020-01-32 00:00:00 +0800
categories: [language, golang]
tags: [golang, troubleshooting]
author: ahern
---

## 定性goroutine泄漏

现象1: goroutine数量持续增长（本文例子使用了goroutine pool，没有持续增长现象）

现象2: 内存持续增长

`go tool pprof http://0.0.0.0:6060/debug/pprof/goroutine`查看top，runtime.gopark接近100%

  - ```
    root@ip:~/pprof# go tool pprof http://0.0.0.0:6060/debug/pprof/goroutine
    File: my-tracing
    Type: goroutine
    Time: Jul 4, 2023 at 11:00am (+08)
    Entering interactive mode (type "help" for commands, "o" for options)
    (pprof) top
    Showing nodes accounting for 2571, 99.30% of 2589 total
    Dropped 117 nodes (cum <= 12)
    Showing top 10 nodes out of 61
          flat  flat%   sum%        cum   cum%
          2571 99.30% 99.30%       2571 99.30%  runtime.gopark
             0     0% 99.30%         24  0.93%  bufio.(*Reader).Read
             0     0% 99.30%        421 16.26%  git.***.ConsumeTraces
             0     0% 99.30%        421 16.26%  git.***.ConsumeTraces
             0     0% 99.30%       1779 68.71%  git.**.func1
             0     0% 99.30%       1779 68.71%  git.**.processTraces
             0     0% 99.30%         90  3.48%  git.**.AddToCurrentBatch
             0     0% 99.30%         16  0.62%  github.com/panjf2000/ants.(*Pool).Submit
             0     0% 99.30%         16  0.62%  github.com/panjf2000/ants.(*Pool).retrieveWorker
             0     0% 99.30%         27  1.04%  github.com/panjf2000/ants.(*Pool).revertWorker
    (pprof)
    ```

## 命令行

`go tool pprof http://0.0.0.0:6060/debug/pprof/goroutine`

top: 查看函数调用占比

traces: 查看所有goroutine的调用栈

  ```
  (pprof) traces
  File: my-tracing
  Type: goroutine
  Time: Jul 4, 2023 at 6:01pm (+08)
  -----------+-------------------------------------------------------
        1875   runtime.gopark
               runtime.goparkunlock (inline)
               runtime.semacquire1
               sync.runtime_SemacquireMutex
               sync.(*Mutex).lockSlow
               sync.(*Mutex).Lock (inline)
               sync.(*Map).LoadOrStore
               git.**.processTraces
               git.**.ConsumeTraces.func1
               github.com/panjf2000/ants.(*goWorker).run.func1
  -----------+-------------------------------------------------------
  .....
  ```

  - 1875个goroutine执行了该调用栈，然后runtime.gopark阻塞挂起

- list:  查看函数代码，远程机器不能支持

## Debug=1，定位函数调用栈

`wget http://ip:port/debug/pprof/goroutine?debug=1 -O 1.gor`下载pprof

`wim 'goroutine?debug=1'`打开文件，搜索阻塞函数名:samplingPolicyOnTickPerShards

  ```
  1 @ 0x439bb6 0x44aa2c 0x44aa06 0x466fa5 0x483471 0xd1d53d 0xd16cb2 0xd16ca4 0x46b1e1
  #       0x466fa4        sync.runtime_Semacquire+0x24                                                                                                                    /usr/local/go/src/runtime/sema.go:56
  #       0x483470        sync.(*WaitGroup).Wait+0x70                                                                                                                     /usr/local/go/src/sync/waitgroup.go:130
  #       0xd1d53c        git.**.samplingPolicyOnTickPerShards+0xfc    /apps/vendor/git.**/processor.go:245
  #       0xd16cb1        git.**/timeutils.(*PolicyTicker).OnTick+0x31                              /apps/vendor/git.**/ticker_helper.go:56
  #       0xd16ca3        git.**/timeutils.(*PolicyTicker).Start.func1+0x23                         /apps/vendor/git.**/ticker_helper.go:47
  
  1 @ 0x439bb6 0x44aa2c 0x44aa06 0x466fa5 0x483471 0xd1e1af 0xcc048f 0x262e86d 0x25eb5f3 0xcb3eef 0xcb20f8 0xb00dac 0xacf39b 0xaf777c 0xacf3c3 0xacf2eb 0xcb1fb8 0xad0112 0xad3e4a 0xacdc38 0x46b1e1
  ```

  

## Debug=2，查看goroutine阻塞时间

`wget http://ip:port/debug/pprof/goroutine?debug=2 -O 2.gor`下载pprof

`vim 'goroutine?debug=2'`打开文件,搜索阻塞函数名:samplingPolicyOnTickPerShards

  ```
  goroutine 279 [semacquire, 892 minutes]:
  sync.runtime_Semacquire(0xc001298f30)
          /usr/local/go/src/runtime/sema.go:56 +0x25
  sync.(*WaitGroup).Wait(0xc00179c1e0)
          /usr/local/go/src/sync/waitgroup.go:130 +0x71
  git.**.samplingPolicyOnTickPerShards(0xc0009d65a0)
          /apps/vendor/git.**/processor.go:245 +0xfd
  git.**/timeutils.(*PolicyTicker).OnTick(...)
          /apps/vendor/git.**/ticker_helper.go:56
  git.**/timeutils.(*PolicyTicker).Start.func1()
          /apps/vendor/git.**/ticker_helper.go:47 +0x32
  created by git.**/timeutils.(*PolicyTicker).Start
          /apps/vendor/git.**/ticker_helper.go:43 +0xb2
  ```

  - goroutine id: 279
  - 当前状态：semacquire
  - 阻塞了：892 minutes

## 总结

- 第一步：定性是否为goroutine泄漏
- 第二步：利用debug=1和debug=2定位代码函数
- 第三步：梳理代码逻辑，修复问题

## 参考

- 实战Go内存泄露：https://lessisbetter.site/2019/05/18/go-goroutine-leak/





