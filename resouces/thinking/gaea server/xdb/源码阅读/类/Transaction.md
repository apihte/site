+++
title = 'Transaction.md'
date = 2023-11-09T15:56:46+08:00
draft = true
+++

# Transcation

## 成员变量

### locks

```java
private HashMap<Lockey, Lockey> locks
```

记录事务中的锁

### logNotifyTTables

```java
private HashMap<String, TTable<?, ?>> logNotifyTTables
```

记录改动过的表

## 方法

### add

```java
public void add(Lockey lockey)
```

添加事务锁

### addSeqLock

```java
public void addSeqLock(Lockey lockey, boolean isSelect)
```

添加锁到锁检测工具中

### addLogNotifyTTable

```java
public void addLogNotifyTTable(TTable<?, ?> ttable) {
    logNotifyTTables.put(ttable.getName(), ttable);
}
```

添加修改过的脏表

### addCachedTRecord

```java
<K, V> void addCachedTRecord(TTable<K, V> ttable, TRecord<K, V> r) {
    HashMap<Object, Object> cachedForTable = cachedTRecord.get(ttable.getName());
    if (cachedForTable == null) {
        cachedForTable = new HashMap<Object, Object>();
        cachedTRecord.put(ttable.getName(), cachedForTable);
    }
    cachedForTable.put(r.getKey(), r);
}
```

添加 TRecord 的缓存

### rmvCachedTRecord

```java
<K, V> void rmvCachedTRecord(TTable<K, V> ttable, K k) {
    HashMap<Object, Object> cachedForTable = cachedTRecord.get(ttable.getName());
    if (cachedForTable != null) {
        cachedForTable.remove(k);
    }
}
```

删除 TRecord 的缓存

### finish

```java
private void finish() {
    wrappers.clear();

    clearOriginKeys();

    // 没有按照lock的顺序unlock。
    for (Lockey lockey : locks.values()) {
        // Trace.debug("unlock " + lockey);
        try {
            lockey.setOwner(null);
            lockey.unlock();
        } catch (Throwable e) {
            Trace.fatal("unlock " + lockey, e);
        }
    }
    locks.clear();
    locksChecker.clear();
    cachedTRecord.clear();
    selectCached.clear();
}
```

事务结束，释放所有锁并清除数据