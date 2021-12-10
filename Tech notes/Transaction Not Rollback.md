Posted on 2019-01-08
### 当前配置

由于项目是 SSM 框架整合而成，MySQL 的事务配置如下：

```xml
<!--数据源配置-->
<bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
		init-method="init" destroy-method="close">
		<property name="url" value="${jdbc_url}" />
		<property name="username" value="${jdbc_username}" />
		<property name="password" value="${jdbc_password}" />
</bean>

<!--事务管理配置-->
<bean id="transactionManager"
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>

<!--切面配置-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="get*" read-only="true" />
        <tx:method name="find*" read-only="true" />
    </tx:attributes>
</tx:advice>

<!--AOP 配置-->
<aop:config>
    <!--把事务控制在 Service 层-->
    <aop:pointcut id="allServiceMethod"
                  expression="execution(* com.test.*.service..*.*(..))" />
    <aop:advisor advice-ref="txAdvice" pointcut-ref="allServiceMethod" />
</aop:config>
<!-- 开启注解方式配置事物 -->
<tx:annotation-driven transaction-manager="transactionManager" />
```



------

### 问题分析

在一个事务管理的接口，方法抛出了 `NullPointerException` 异常，但是有关数据库表的操作却没有回滚，伪代码片段如下：

```java
...
try {
   noticeMapper.insert(notice);
   noticeLogMapper.insert(noticeLog);
} catch (Exception e) {
    LOGGER.error("数据插入异常！！！");
    e.printStackTrace();
}
...
```

在 `noticeLog` 的插入过程中代码抛出了异常，但是 `notice` 却成功的插入到了数据库，相当于这个明明被事务管理的方法在遇到程序异常却没有回滚事务?。 

查阅文档及资料，在 Spring [官方文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#transaction-declarative-rolling-back)中有解释：

<img src="https://tryme.wang/usr/images/sina/5cd95d454a613.jpg" alt="image" align="center">

注意中间红框段落，翻译一下：

```
In its default configuration, the Spring Framework’s transaction infrastructure code marks a transaction for rollback only in the case of runtime, unchecked exceptions. That is, when the thrown exception is an instance or subclass of RuntimeException. ( Error instances also, by default, result in a rollback). Checked exceptions that are thrown from a transactional method do not result in rollback in the default configuration. 

在其默认配置中，Spring Framework的事务基础结构代码仅在运行时未经检查的异常情况下标记回滚事务。也就是说，抛出的异常是RuntimeException的实例或子类。 （默认情况下，错误实例也会导致回滚）。从事务方法抛出的已检查异常不会导致在默认配置中回滚。
```

可以看到代码抛出 RuntimeException 的实例、子类或在默认情况下的 Error 才会导致回滚，`但是，从事务方法抛出的已检查异常不会导致在默认配置中回滚。`

所以代码中 `try catch` 捕获了异常，导致了事务无法回滚(⊙x⊙)。



------

### 解决方案

#### 方案一

既然代码的异常被捕获后事务无法回滚，那么可以手动抛出 `RuntimeException` 的实例或子类：

```java
...
try {
   noticeMapper.insert(notice);
   noticeLogMapper.insert(noticeLog);
} catch (Exception e) {
    LOGGER.error("数据插入异常！！！");
    e.printStackTrace();
    throw new RuntimeException();
}
...
```

#### 方案二

既然事务没有自动回滚，我们可以手动设置事务的回滚：

```java
...
try {
   noticeMapper.insert(notice);
   noticeLogMapper.insert(noticeLog);
} catch (Exception e) {
    LOGGER.error("数据插入异常！！！");
    e.printStackTrace();
    TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
}
...
```



再次调试运行后，事务正常回滚?。