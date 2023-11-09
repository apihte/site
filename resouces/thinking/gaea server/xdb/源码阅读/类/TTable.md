+++
title = 'TTable.md'
date = 2023-11-09T15:56:46+08:00
draft = true
+++

# TTable

```java
public abstract class TTable<K, V> extends Table implements TTableMBean {}
```

xbean 的管理器，支持并发访问、自动持久化、管理缓存、统计性能数据。

只能在 Procedure 中操作，否则抛出异常。

管理器使用 Map 实现，也就是说，任何 table 都有唯一关键字，关键字必须是可以比较的，需要实现 Comparable，刚好目前只允许简单类型做关键字。

从 table 得到的 xbean 都是内部实例中的引用，直接对引用进行读取和修改，没有单独的保存操作，只在事务中有效，可以传递给嵌套过程或函数，但不能保存。

工具最终转出的数据表都是继承自 TTable。

例如在 xml 中配置：

```xml
<table name="properties" key="long" value="Properties" cacheCapacity="ROLE_CACHE_CAPACITY" lock="rolelock"/>
```

则是定义了一个 TTable<Long, Properties> 类型的表。

