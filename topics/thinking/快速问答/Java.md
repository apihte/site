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