# MyBatis执行流程

先说MyBatis在没有Spring框架的时候是如何执行的，整个过程如下

```java
// 加载配置文件
InputStream in = Resources.getResourceAsStream("mybatis-config.xml");

// 通过配置文件创建sqlSessionFactory
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);

// 创建sqlSession
SqlSession sqlSession = sqlSessionFactory.openSession();

// 拿到对应的代理类
MessageMapper mapper = sqlSession.getMapper(MessageMapper.class);

// 执行方法，得到结果
List<MessageEntity> messageEntities = mapper.selectList();
```

## 1、创建sqlSessionFactory

> 非常简单，传入配置文件，然后将配置文件解析出来，创建sqlSessionFactory，同时将配置信息传递给sqlSessionFactory


实际的代码如下：

```java
public SqlSessionFactory build(InputStream inputStream) {
  return build(inputStream, null, null);
}

public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
  try {
    XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
    return build(parser.parse());
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error building SqlSession.", e);
  } finally {
    ErrorContext.instance().reset();
    try {
      inputStream.close();
    } catch (IOException e) {
      // Intentionally ignore. Prefer previous error.
    }
  }
}

public SqlSessionFactory build(Configuration config) {
  return new DefaultSqlSessionFactory(config);
}
```

## 2、获取sqlSession

> 这一步同样也不难，一眼就能看出是怎么回事，如下：


```java
public SqlSession openSession() {
  return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
}

private SqlSession openSessionFromConnection(ExecutorType execType, Connection connection) {
  try {
    boolean autoCommit;
    try {
      autoCommit = connection.getAutoCommit();
    } catch (SQLException e) {
      // Failover to true, as most poor drivers
      // or databases won't support transactions
      autoCommit = true;
    }
    final Environment environment = configuration.getEnvironment();
    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    final Transaction tx = transactionFactory.newTransaction(connection);
    final Executor executor = configuration.newExecutor(tx, execType);
    return new DefaultSqlSession(configuration, executor, autoCommit);
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}

private TransactionFactory getTransactionFactoryFromEnvironment(Environment environment) {
  if (environment == null || environment.getTransactionFactory() == null) {
    return new ManagedTransactionFactory();
  }
  return environment.getTransactionFactory();
}
```

## 3、拿到对应的代理类

## 4、执行方法
