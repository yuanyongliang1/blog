---
title: threadPool
date: 2024-02-21 14:14:52
tags: 
 - 线程池
categories: 
 - 多线程
---

#### 线程池的核心参数以及具体场景下核心参数怎么设置合适

- 核心线程数（Core Pool Size）：线程池中最小的线程数，即在线程池中一直保持的线程数量，不受空间时间的影响

- 最大线程数（MaximumPoolSize）：线程池维护线程的最大数量

- 空闲线程存活时间（Keep Alive Time）：当线程池中的线程数超过核心线程数时，多余的线程会被回收，此参数即为非核心线程的空闲时间，超过此时间将会被回收

- 空闲线程存货单位（unit ）

- 工作队列（Work Queue）：用于存储等待执行的任务队列，当线程池中的线程数达到核心线程数时，新的任务将被加入工作队列。

- 拒绝策略（Reject Execution Handler）：当线程池和工作队列都已经达到最大容量，无法再接收新的任务时，拒绝策略将被触发，常见的拒绝策略有抛出异常，直接丢弃任务，丢弃队列中最老的任务等

- 线程工厂（Thread Factory）：用于创建新的线程，可定制线程名字，线程组，优先级等

  

在不同的场景下设置核心参数，首先需要判断程序是CPU密集型还是I/O密集型，CPU密集型是指CPU使用频率较高，也就是CPU经常计算一些复杂的运算，逻辑处理等，这时候一般吧核心线程数设置为CPU核心数就行，如果是I/O密集型，核心线程数是CPU密集型的两倍