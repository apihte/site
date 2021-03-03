# Procedure

## 概述

Procedure 类是 xdb **存储过程**的实现基类，**支持嵌套**，使用者通过继承 Procedure 类并重载 process() 方法来访问 xbean、xtable。

## 代码

### 成员变量

```java
// 内部状态和配置，根据编程需要添加
private volatile ProcedureConf conf;
// 执行结果
private volatile boolean success = false;
private volatile Throwable exception;
// 连接信息
private static volatile IOnlines onlines = null;

/**
 * 任务集合
 * 一般来说，一个存储过程里面不会提交太多任务，这里使用  ArrayList。
 */
private static ThreadLocal<ArrayList<Task>> ptasks = new ThreadLocal<ArrayList<Task>>() {
    protected java.util.ArrayList<Task> initialValue() {
        return new ArrayList<Task>();
    }
};

private ArrayList<Task> tasks = null;

private static ThreadLocal<TreeMap<Bean, xdb.util.BeanPool<?>>> pbeans = new ThreadLocal<TreeMap<Bean, xdb.util.BeanPool<?>>>() {
    protected TreeMap<Bean, xdb.util.BeanPool<?>> initialValue() {
        return new TreeMap<Bean, xdb.util.BeanPool<?>>(BeanComparator.instance);
    }
};
	
private static ThreadLocal<TreeSet<Bean>> pdatas = new ThreadLocal<TreeSet<Bean>>() {
    protected TreeSet<Bean> initialValue() {
        return new TreeSet<Bean>(BeanComparator2.instance);
    }
};

// 存储过程队列名
private String procedureQueueName = null;
```

