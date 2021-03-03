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