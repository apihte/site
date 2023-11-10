+++
title = 'Java.md'
date = 2023-11-09T15:56:46+08:00
draft = true
+++

# Java

## Java 线上问题排查

先用 top 命令查看系统的运行状态，定位是 CPU 的问题还是内存的问题

### jmap

查看对内对象的统计信息、查看 ClassLoader，导出整个 JVM 的 dump

jmap -histo:live pid 查看内存使用量

jmap -heap pid 查看堆内存配置和使用

### jstack

生成线程堆栈调用，用来分析锁的问题

jstack pid 查看线程 CPU 占用

jstack -l pid 分析死锁

### jstat

查看 GC 统计

jstat -gc pid duration 查看 GC



## 数据结构

### BlockingQueue

四组 api

| 方式         | 抛出异常  | 有返回，无异常 | 阻塞等待 | 超时等待                                |
| ------------ | --------- | -------------- | -------- | --------------------------------------- |
| 添加         | add(E e)  | offer(E e)     | put(E e) | offer(E e, long timeout, TimeUnit unit) |
| 移除         | remove()  | poll()         | take()   | poll(long timeout, TimeUnit, unit)      |
| 检测队首元素 | element() | peek()         |          |                                         |

### SynchronousQueue

不存储元素，put 元素后必须取出来才能重新 put

