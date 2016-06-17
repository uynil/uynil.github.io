---
layout: post
title: Python 垃圾回收机制
tags: [Python]
---
#### 1. 

#### 2. GIL 锁机制

ACQUITE, RELEASE

IO
操作完，释放锁

CPU
tick 机制，到达阀值（100）检查一次。 sys.setcheckinterval() 设置阀值。


单CPU，调度良好。
多CPU，调度打架。另外一个线程不停争夺资源，导致允许缓慢。

#### 3. 多线程之间调度
有两块一起控制（pthread 和）
pthread 先进先出 FIFO
OS层  线程具有优先级的队列， priority queue。 优先级高的先出列。


#### 4. 新能
单核性能不错
多核性能糟糕
CPU操作 线程打架
IO操作 不会释放锁，但有IO震颤问题，单一线程执行时会频繁释放信号协调，增重负担。

#### 5. python3.2 引入新GIL机制
    - 性能有提升
    - 解决IO 震颤问题
机制
    - 增加 gil_drop_request = 0, 到1时释放线程
    - 到1(或l)时信号，以及ACK信号保证线程切换
问题
    - 新版GIL 影响IO性能，增加响应时间
    - 新机制 忽略 优先级检查

### 参考文章
1. GIL的讲解 [David Beazley 2010年在PyCon的演讲ppt](http://www.dabeaz.com/python/UnderstandingGIL.pdf "PyCon2010GIL")
