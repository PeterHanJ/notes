# SqlSessionFactoryBuilder

```java
SqlSessionFactoryBuilder
通过 XMLConfigBuilder 解析xml设定

build(Configuration config) 返回一个DefaultSqlSessionFactory
```

# DefaultSqlSessionFactory

```java
主要是 openSession方法
具体实现在 openSessionFromDataSource，主要做了如下：
    1. 通过env来得到一个 TransactionFactory
    2. TransactionFactory创建一个 transaction
    3. 开一个新的thread,传入transation
       Executor executor = configuration.newExecutor(tx, execType);
    4. 返回一个new DefaultSqlSession(configuration, executor, autoCommit);
    5. DefaultSqlSession提供
```

