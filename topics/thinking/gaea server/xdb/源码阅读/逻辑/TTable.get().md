# TTable.get()

查找表中的数据，返回 value 的引用

```java
public final V get(K key, boolean holdNull) {...}
```

1. 首先去事务中查找缓存，如果缓存中存在，则直接返回，如果不存在，则继续后面的逻辑。
2. 获取并持有锁。
3. 