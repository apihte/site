# xdb 源码阅读

## xdb 存储引擎

### 概述

xdb 存储引擎的大致工作流程：

- 插入数据
  1. 根据数据大小分配数据页
  2. 根据 key 加入到 B+ Tree 中，在这个过过程中所有被修改过的页标记为脏页
  3. 登倒存储引擎 checkpoint 执行，先将脏页写入到 LoggerFile 中，再将 LoggerFile 写入到 DataFile


### 代码

#### Xdb 类

##### 成员
- XdbConf conf `xdb 的配置，从 xml 读取`
- Tables tables
- Angel angel
- Checkpoint checkpoint
- boolean isOpen
- int dumpCheckPoint

##### xdb 启动过程
1. 检查 Dbx 是否已经启动，如果已经启动则抛出异常
2. 检查 xdb 是否已经打开，如果已打开则返回 false
3. 检查是否有 XdbConf
4. 创建 dbdata 和 dblogs 文件夹
5. 创建 metadata.xml 文件
6. 静态创建 Lockeys 实例
7. 反射创建 \_Tables_ 类，\_Tables_ 类继承自 Tables 类，在创建时在构造函数里会 add 所有的数据表到 Tables 类中的 tables Map 中（之后把这个 \_Tables 对象赋值给 Xdb 类中的 tables，则完成了对 Xdb.tables 的初始化）
8. 创建线程池，线程池包含 scheduled、procedure、default 三种，scheduled 负责定时器任务，procedure 负责 xdb 事务任务，defualt 一般负责协议任务
9. 将 \_Table_ 对象赋值给 Xdb.tables，然后使用 XdbConf 初始化 Xdb.tables（这里为所有的表创建了 Storage 对象，然后为表分配锁 id）
10. 创建 CheckPoint 和 Angle 实例
11. 设置 Xdb.isOpen 标志位为 true，表示已经开启
12. 初始化 UniqName 和 xio 网络
13. 注册 MBean
14. 启动 CheckPoint 和 Angel 线程

