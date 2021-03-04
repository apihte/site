# TRecord

记录实现，内部实现类。管理记录的状态、锁。

```java
/**
 * @param <K> key type
 * @param <V> value type
 */
public final class TRecord<K, V> extends XBean {}
```

## 内部类

### State

```java
public static enum State {
    INDB_GET,    // 查询装载进入
    INDB_REMOVE, // 删除，在数据库中存在该记录。
    INDB_ADD,    // 新增记录，在数据库中存在该记录。这种情况原因：删除操作还没有flush，又执行了增加。
    ADD,         // 新增的记录
    REMOVE,      // 新增操作还没有flush，又执行了删除。此状态记录只存在于事务过程中，事务结束就会从cache中删除。
}
```

记录的状态的枚举

### LogAddRemove

[TRecord.LogAddRemove]: ./TRecord.LogAddRemove.md	"TRecord.LogAddRemove"

记录添加或删除时，会在 savepoint 中通过 LogAddRemove 保存记录当前的值和状态

## 成员

### value

```java
private V value;
```

记录的值

### lockey

```java
private final Lockey lockey;
```

持有的锁

### state

```java
private State state;
```

记录的状态，新增、修改、删除等

### recordLockey

```java
private Lockey recordLockey;
```

持有的记录锁

[^lockey 和 recordLockey 的区别]: 

### lastAccessTime

```java
private volatile long lastAccessTime = System.nanoTime();
```

## 成员方法

### getTable

```java
final TTable<K, V> getTable() {
    @SuppressWarnings("unchecked")
    TTable<K, V> table = (TTable<K, V>)xdbParent();
    return table;
}
```

获取表实例

### getKey

```java
public final K getKey() {
    @SuppressWarnings("unchecked")
    K key = (K)lockey.getKey();
    return key;
}
```

获取 key