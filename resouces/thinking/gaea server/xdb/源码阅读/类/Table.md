+++
title = 'Table.md'
date = 2023-11-09T15:56:46+08:00
draft = true
+++

# Table 

```java
public abstract class Table extends XBean {}
```

表内部接口定义，因充当 TRecord 的 parent，所以继承了 XBean。

## 内部类

### Persistence

```java
public static enum Persistence {
    MEMORY, DB
}
```

持久化枚举

- MEMORY - 内存
- DB - 数据库

## 成员方法

### open

```java
abstract Storage open(XdbConf xconf, Logger logger);
```



### close

```java
abstract void close();
```

