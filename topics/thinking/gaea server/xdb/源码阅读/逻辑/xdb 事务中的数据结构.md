# xdb 事务中的数据结构

因为是在 xdb 事务中的数据的修改，所以涉及到 commit 和 rollback，commit 还好理解，而既然有 rollback 的话，就涉及到一定有什么数据结构在记录这个数据修改的过程，以便事务失败时能够回滚数据。

所以在 Transcation 类中有这么一个成员变量：

```java
private List<Savepoint> logs = new ArrayList<Savepoint>();
```

事务中数据的修改日志都记录在这个 logs 对象中。

当数据提交，就会正序遍历这个 logs 对象并执行 Savepoint.commit；当数据回滚时，则逆序遍历 logs 对象并执行 Savepoint.rollback 方法。

```java
public void perform(Procedure p) throws Throwable {
    try {
        // 总数 = .True(未统计此项) + .False + .Exception
        counter.increment(p.getClass().getName());
        totalCount.incrementAndGet();

        // flush lock . MEMORY类型的表本来不需要这个锁，为了不复杂化流程，不做特殊处理。
        Lock flushLock = Xdb.getInstance().getTables().flushReadLock();
        flushLock.lockInterruptibly();
        try {
            if (p.call()) {
                if (_real_commit_() > 0)
                    logNotify(p);
                // else : 没有修改，不需要logNotify。至此过程处理已经完成了。
                // .True
            } else {
                // 执行逻辑返回false统计
                counter.increment(p.getClass().getName() + ".False");
                totalFalse.incrementAndGet();
                _last_rollback_(); // 应用返回 false，回滚。
            }
        } 
        ... // 非必要代码省略
    }
}
```

事务的执行会通过 Procedure 中的 call、submit 等方法调用到 Transcation 的 perform 方法，perform 方法最终调用的则是 \_real_commit\_ 和 \_last_rollback\_ 方法

```java
private int _real_commit_() {
    if (0 != trancount) // 检查 begin 与 commit/rollback 是否匹配
        throw new XError("xdb: mismatch (begin,commit/rollback) trancount=" + trancount);
    int count = 0;
    for (Savepoint sp : logs) // 正序遍历
        count += sp.commit();
    logs.clear();
    return count;
}
```

```java
private void _last_rollback_() {
    try {
        // 逆序遍历
        for (int index = logs.size() - 1; index >= 0; --index)
            logs.get(index).rollback();
        logs.clear();
        trancount = 0;
    } catch (Throwable err) {
        // 如果发生错误，此时数据已经处于不正常状态.
        Trace.fatal("last rollback ", err);
        Runtime.getRuntime().halt(54321);
    }
}
```

再看下 Savepoint 中的 commit 和 rollback 两个方法

```java
int commit() {
    for (Log log : addOrder)
        log.commit();
    return addOrder.size();
}

int rollback() {
    // 按加入相反顺序回滚。
    // 当set或者map的key使用自定义xbean时，可以在加入容器前修改(hashCode会发生变化)，
    // 此时需要按造相反顺序回滚，先回滚容器，然后是xbean本身。
    for (int i = addOrder.size() - 1; i >= 0; --i)
        addOrder.get(i).rollback();
    return addOrder.size();
}
```

可以看到主要是在遍历一个叫 addOrder 的元素类型为 Log 的成员对象 List：

```java
List<Log> addOrder = new ArrayList<Log>();
```

看一下 Log 的定义：

```java
package xdb;

public interface Log {
	public void rollback();
	public void commit();
}
```

是一个接口，xdb 支持的所有数据类型都继承了这个接口，比如 LogInt、LogString、LogObject 等等，这些数据 