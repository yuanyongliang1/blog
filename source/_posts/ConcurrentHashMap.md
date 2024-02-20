---
title: 一文读懂 ConcurrentHashMap 原理
date: 2024-02-18 12:54:28
tags: 
 - 文章
categories: 
 - java基础
---



**出现背景**

#### 1、线程不安全的HashMap

- 非原子操作：HashMap的操作不是原子性的，例如put()方法涉及到了多个步骤，包括计算hash值，查找或插入元素等，如果多个线程同时执行这些操作，就有可能导致数据不一致的情况。
- 容量扩容，HashMap在扩容时，需要重新计算元素的hash值并分配存储位置，这个过程涉及到对原数组进行复制和重新插入元素的操作。如果在扩容期间有其他线程对HashMap进行并发修改，就可能导致数据丢失或出现异常

#### 2、效率低下的HashTable

HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低，因为当一个线程正在访问HashTable的同步方法时，这时，另外一个线程也来访问HashTable的同步方法， 可能会进入阻塞和轮询状态。如线程A使用put进行添加元素的时候，线程B不但不能使用put方法添加元素，而且不能使用get方法获取元素，所以竞争越激烈效率越低，也就是说对于HashTable而言，synchronized是针对Hash整张表的，即每次锁住整张表让线程独占，相当于所有线程进行读写的时候都去竞争一把锁。

#### ConcurrentHashMap的锁分段技术

编写

