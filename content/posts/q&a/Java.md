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

## 问题查询

*   top 看进程资源占用
*   top -H -p pid 看指定进程中的线程的资源占用
*   jmap 堆 dump
*   jstack 栈 dump

## jvm

### 内存模型

1.  堆
2.  方法区
3.  虚拟机栈
4.  本地方法栈
5.  程序计数器

### 双亲委派

#### 什么是双亲委派

当一个类加载器收到类加载请求时，不是直接加载类，而是向上委托给自己的父加载器加载，父加载器无法加载时才会自己尝试加载。

#### 为什么要有双亲委派

1.  避免类的重复加载
2.  分工加载，保证核心 api 不会被篡改，保证安全性

#### 双亲委派是继承关系吗

是组合关系，父加载器作为子加载器的一个成员

### 破坏双亲委派

自定义类加载器，重写 loadClass 方法，不使用双亲委派及时即可

### CMS (Concurrent Mark Sweep) 收集器

CMS 作于于老年代，基于标记-清除算法，回收步骤：

1.  初始标记
2.  并发标记
3.  重新标记
4.  并发清除
5.  重置

优缺点：

*   优点：并发收集、低停顿
*   缺点：
    1.  对 CPU 资源敏感
    2.  无法处理浮动垃圾
    3.  收集时容易产生大量空间碎片

### G1 收集器

引入分区思路，弱化分代概念，合理利用各个周期的资源，解决了其他收集器甚至 CMS 的众多缺陷

*   但 G1 仍然分代，它将堆内存分成了很多个 Region，这些 Region 可以属于新生代也可以属于老年代，回收的时候不会按照代来回收，而是哪块区域回收的收益高就回收哪块

回收步骤：

1.  初始标记
2.  并发标记
3.  最终标记
4.  筛选清除

#### 内存模型

1.  分区，G1 采用了分区（Region）的思路，将整个堆空间分成若干个大小相等的内存区域，每次分配对象空间将逐段地使用内存。

2.  卡片，每个分区内部又被分成了若干个大小为 512 Byte 的卡片，表示堆内存最小可用粒度，所有的分区的卡片将会记录在全局卡片表中

3.  堆

MaxCGPauseMillis 最大停顿时间，默认200

### Shenandoah

1.  初始标记
2.  并发标记
3.  最终标记
4.  并发清理
5.  并发回收
6.  初始引用更新
7.  并发引用更新
8.  最终引用更新
9.  并发清理

## 调优

### 参数

*   +AlwaysPreTouch

jvm 在启动时原本只会先分配虚拟内存，只有内存真正需要被使用的时候才会分配物理内存。

使用这个参数可以使 jvm 在启动的时候直接分配物理，不需要使用的时候再去申请，所以会加快程序运行的速度，但这个分配的过程是在main 函数执行之前，会减慢程序的启动速度

*   AutoBoxCacheMax=30000

jvm 对 Integer 的缓存默认范围是 -128\~127，这个参数的作用就是修改这个默认值，最小不能小于127。

*   -OmitStackTraceInFastThrow

该参数默认开启，当 jvm 检测到大量重复的异常后，会抛出一个已经分配好的异常对象，日志中将不会再打印这个异常，这样会节省一些性能，但是如果前面的日志没有保存，后面会找不到异常堆栈

*   -UseBiasedLocking

是否开启偏向锁

*   -UseCounterDecay

计数器衰减参数，默认情况下，每次 GC 都会对方法的调用计数器进行减半，会使得一些方法一直无法达到 C2 编译器的阈值，无法触发C2 编译器对热点方法的优化。

*   +EliminateAllocations

是否开启标量替换（默认开启），通过逃逸分析来决定是否进行标量替换，可以减少变量在堆内存上的分配，减轻 gc 压力。类似的还有EliminateLocks（锁消除），它们都是基于 DoEscapeAnalysis（逃逸分析）来工作的，默认开启。

*   -UseAdaptiveSizePolicy

自适应大小策略，jvm 每次 minor gc 时根据 gc 过程统计的一些数据重新计算 eden 区和 survivor 区的大小。

*   +HeapDumpOnOutOfMemoryError

jvm 发生内存溢出时自动生成 dump 文件

*   -UseCompressedClassPointers

是否使用使用压缩类指针，堆内存超过 32G 后会失效（自动关闭压缩），在我们的项目为啥关了？

*   MetaspaceSize=384m

元空间内存大小

*   MaxMetaspaceSize=384m

元空间最大内存大小

*   ReservedCodeCacheSize=261m

jit 编译代码的最大代码缓存大小，默认大小 240m，禁用 TieredCompilation 后默认 48m，最多不允许超过 2GB，最小不允许小于 InitialCodeCacheSize（初始代码缓存大小）。

*   NonNMethodCodeHeapSize=5m

设置包含非方法代码的代码段的字节大小，只有在 SegmentedCodeCache  参数生效时有效。

*   NonProfiledCodeHeapSize=128m
*   ProfiledCodeHeapSize=128m
*   -UseCodeCacheFlushing

是否在code cache满的时候先尝试清理一下，如果还是不够用再关闭编译，默认开启

*   +PrintCodeCache

jvm 关闭时输出 code cache 的使用情况

*   +PrintCodeCacheOnCompilation

用于在方法每次被编译时输出 code cache 的使用情况

*   -TieredCompilation

是否开启分层编译（默认开启）

*   SoftRefLRUPolicyMSPerMB=0

设置每占用 1 MB 的软引用的对象在最后一次被引用后可以存活的时长（millis），默认是 1 秒钟。一般客户端 vm 倾向于刷新软引用而不是增大堆，而服务器 vm 倾向于增大堆而不是刷新软引用。在 gc 时受最大堆内存 Xmx 参数的影响比较大。

*   +UseTransparentHugePages

开启透明大页，只在 linux 下生效。会对 redis 性能产生影响，使 redis 在 fork 时产生延迟，在其他数据库环境下也不推荐使用。

Transparent Huge Pages，简称 THP，是一种在 Linux 中简化大页面使用和启用的方法。启用后，Linux 内核将尝试使用大页面来保留足够大且有资格使用 THP。可以在三个不同级别配置 THP 支持：

· always - 任何应用程序都会自动使用透明大页面。

· madvise- 透明大页面仅在应用程序使用madvise()标志MADV\_HUGEPAGE来标记某些内存段应由大页面支持时才使用。

· never - 从不使用透明大页面。

配置存储在/sys/kernel/mm/transparent\_hugepage/enabled并且可以像这样轻松更改：\$ echo "madvise" > /sys/kernel/mm/transparent\_hugepage/enabled

JVM 支持在madvisemode 中配置时使用 THP ，但需要使用-XX:+UseTransparentHugePages. 完成此操作后，Java 堆以及其他内部 JVM 数据结构将由透明大页面支持。

为了使内核能够满足使用透明大页面的请求，需要有足够的连续物理内存可用。如果没有内核将尝试对内存进行碎片整理以满足请求。碎片整理可以通过几种不同的方式进行配置，当前策略存储在/sys/kernel/mm/transparent\_hugepage/defrag.

## 压力测试

### 类生命周期

*   加载
*   验证
*   准备
*   解析
*   初始化
*   使用
*   卸载

### ThreadLocal

每个 Thread 中都有一个 ThreadLocalMap，ThreadLocalMap 中存储数据的结构又是一个 key 为 ThreadLocal、value 为 Object 的 Entry
查看 ThreadLocal 中 set 和 get 方法可以更好的理解底层逻辑

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

所以 ThreadLocal 的原理就是每个 Thread 自己存自己的数据，和使用锁或者 sychronized 时的共享同一份数据的理念正好相反

线程结束时，要记得调用 remove 移除掉这个线程对这个 ThreadLocal 内容的引用，因为 ThreadLocal 对象不是垃圾，不会被回收，所以相对的，它所关联的对象也不会被回收

## 线程池 ThreadPoolExecutor

## 编译器

1.  前端编译器：JDK 中的 javac、Eclipse JDT 中的 ECJ
2.  即时编译器：HotSpot 中的 C1、C2 编译器，Graal 编译器
3.  提前编译器：JDK 的 jaotc、GNU Compiler for the Java（GCJ）、Excelsior JET

## 锁

### synchronized

是一个关键字，基于会锁住被其修饰的代码块
实现原理是修改对象头的 monitor
不能精确控制

#### CAS

compare and swap
三个值

*   内存值 V：要比较时从内存刚获取到的值
*   预期旧值 A：比较之前从内存获取到的值
*   目标值 B：通过和预期旧值进行计算之后得到的新值
    只有当 A 等于 V 的时候，才会用 B 赋值

ABA 问题，加版本号

### ReentrantLock 重入锁

### wait、notify、notifyAll

wait 立即释放锁，并使当前线程进入到等待状态
notify 不立即释放锁，从处于等待队列的线程中选择一个进行通知，通知其进入同步队列，当同步块执行结束后将锁释放
notifyAll 同 notify，不同的是不是选择一个等待线程，而是使所有等待线程都进入同步队列

> 这三个方法都必须在同步块中执行

### Semaphore 共享锁

### Queue

*   AbstractQueue

    *   ConcurrentLinkendQueue（线程安全，CAS）
*   DeQue
*   BlockingQueue（线程安全，ReentrantLock）
