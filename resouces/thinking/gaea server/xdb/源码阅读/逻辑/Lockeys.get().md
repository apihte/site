+++
title = 'Lockeys.get().md'
date = 2023-11-09T15:56:46+08:00
draft = true
+++

# Lockeys.get()

```java
public static Lockey get(TTable<?, ?> ttable, Object key) {
    return get(ttable.getLockName(), ttable.getLockId(), key);
}
```

返回表格单个记录锁。

调用了一个静态方法：

```java
static Lockey get(String name, int index, Object key) {
    return instance.get(new Lockey(name, index, key));
}
```

进而调用了

```java
private final Lockey get(Lockey lockey) {
    // 优化，先看当前事务是否持有，若持有，直接返回
    Transaction curTransaction = Transaction.current();
    if (curTransaction != null) {
        Lockey lockey1 = curTransaction.get(lockey);
        if (lockey1 != null) 
            return lockey1;
    }
    return locks.get(lockey);
}
```

先去查看了当前事务中是否已经持有这把锁，已持有则直接返回，否则调用 Locks 中的 get 方法

```java
public Lockey get(Lockey lockey) {
    return this.segmentFor(lockey).get(lockey);
}
```

调用 Locks 的内部类 Segment 的 get 方法

```java
Lockey get(Lockey key) {			
    sync.lock();
    try {
        Lockey weakey = locks.add(key);
        if (key == weakey)
            key.alloc(); // new key
        return weakey;
    } finally {
        sync.unlock();
    }
}
```

