Posted on 2018-12-17
### Tomcat 容器启动问题

> #### 前提
>
> 1. Spring Boot 版本 2.0.3.RELEASE
> 2. druid-spring-boot-starter 版本 1.1.9
> 3. mysql-connector-java 版本 5.1.46
> 4. MySQL 数据库版本 5.7
> 5. Tomcat 版本 7.0.67（最终升级了版本到 Tomcat 8）
> 6. JDK 版本 1.7（最终升级到1.8）

---
**问题：** 测试、生产配置文件属性不同步，生产环境很多属性未配置，导致错误注入。
**解决：** 将所有注入的 API 地址同步。

---
**问题：** 启动时 Dubbo 报错，找不到服务。

**解决：** 提供者所在机器 Dubbo 端口未开放，开放对应端口即可。



------

**问题：** Tomcat 无法启动，每次启动报不同的错误。

**解决：** JDK版本升级到 1.8→Tomcat 版本升级到 7.0.92



------

**问题：** java.lang.NoClassDefFoundError: javax/el/ELManager

**解决：** [post cid="29" /]



------

**问题：**

配置：

```yaml
# MySQL数据源配置
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    druid:
      url: jdbc\:mysql\://rds.xxx.com\:3306/xxx?useUnicode\=true&characterEncoding\=UTF-8&zeroDateTimeBehavior\=convertToNull
      username: xxx
      password: password
```

异常信息：

```java
Caused by: java.sql.SQLException: connect error, url jdbc\:mysql\://rds.xxx.com\:3306/xxx?useUnicode\=true&characterEncoding\=UTF-8&zeroDateTimeBehavior\=convertToNull, driverClass com.mysql.jdbc.Driver
	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1582)
	at com.alibaba.druid.pool.DruidDataSource.init(DruidDataSource.java:859)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeCustomInitMethod(AbstractAutowireCapableBeanFactory.java:1833)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1776)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1704)
	... 116 more
```

**解决：** 异常信息里非常明确的提及是数据库连接 url 的问题，在 Spring Boot  的 yml 配置文件中，格式为：<code>property: value</code> 属性名称加「:」之后有一个空格，项目之前的配置文件中 MySQL 数据库的连接地址转义过，这里直接复制过来使用，是行不通的，将转义字符「\」去掉即可。



------

### Tomcat 重启之后异常

**问题：**

异常信息：

```java
org.apache.catalina.session.StandardManager doLoad
SEVERE: IOException while loading persisted sessions: java.io.EOFException
java.io.EOFException
    at java.io.ObjectInputStream$PeekInputStream.readFully(ObjectInputStream.java:2298)
    at java.io.ObjectInputStream$BlockDataInputStream.readShort(ObjectInputStream.java:2767)
    at java.io.ObjectInputStream.readStreamHeader(ObjectInputStream.java:798)
    at java.io.ObjectInputStream.<init>(ObjectInputStream.java:298)
   ...

org.apache.catalina.session.StandardManager startInternal
SEVERE: Exception loading sessions from persistent storage
java.io.EOFException
   ...
    
```

**解决：** 删除 <code> ${catalina.home}/work/Catalina/localhost/${APP-NAME}/SESSION.ser</code> 即可



------

**问题：** Dubbo 报错

报错信息：

```java
2018-12-15 20:26:52,387 [DubboSaveRegistryCache-thread-1] WARN  [com.alibaba.dubbo.registry.zookeeper.ZookeeperRegistry] -  [DUBBO] Failed to save registry store file, cause: Can not lock the registry cache file /root/.dubbo/dubbo-registry-10.11.12.13.cache, ignore and retry later, maybe multi java process use the file, please config: dubbo.registry.file=xxx.properties, dubbo version: 2.5.3, current host: 10.20.30.40
java.io.IOException: Can not lock the registry cache file /root/.dubbo/dubbo-registry-10.11.12.13.cache, ignore and retry later, maybe multi java process use the file, please config: dubbo.registry.file=xxx.properties
        at com.alibaba.dubbo.registry.support.AbstractRegistry.doSaveProperties(AbstractRegistry.java:193)
        at com.alibaba.dubbo.registry.support.AbstractRegistry$SaveProperties.run(AbstractRegistry.java:150)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at java.lang.Thread.run(Thread.java:745)
```

Dubbo 会使用文件缓存注册中心地址列表及服务提供者列表，默认路径在 <code>/${user.home}/.dubbo/dubbo-registry-10.20.30.40.cache</code>，应用重启时将基于此文件恢复，一台服务器有多个应用使用这个文件恢复时，会出现这个警告。

**解决：** 可以在每个项目 Dubbo 配置文件中的 <code><dubbo:registry></code> 标签中的 file 指定不同的文件路径。  [官方文档](http://dubbo.apache.org/zh-cn/docs/user/references/xml/dubbo-registry.html) [参考地址](https://github.com/apache/incubator-dubbo/issues/81)


---
**问题：** Tomcat 容器无日志输出
logback-spring.xml配置：
```xml
...
<!-- 测试环境+开发环境. -->
    <springProfile name="test,dev">
        <logger name="com.xxx.im" level="INFO"/>
    </springProfile>
    <!-- 生产环境 -->
    <springProfile name="prod">
        <logger name="com.xxx.im" level="ERROR"/>
    </springProfile>
...
```
**解决：** 生产环境日志输出级别过高，<code>INFO</code> 级别的无法输出，将  <code>ERROR</code> 改为 <code>INFO</code> 即可。

---
[scode type="yellow"]转战华为云之后，所有的服务器重新配置???[/scode]

### RabbitMQ 连接


[scode type="lblue"]Spring Boot 版本 2.0.3.RELEASE[/scode]

**问题：** RabbitMQ 直接使用 yaml 配置如下：

```yaml
spring:
	rabbitmq:
    host: rabbitmq.xxx.com
    username: user
    password: password123456
    port: 5672
```

 项目启动时报错信息：

```java
...
    ...ForgivingExceptionHandler - An unexpected connection driver error occured (Exception message: Connection closed)
...
```

**解决：** 从异常日志中可以很明显的看到，是 <code>Connection closed</code>，解决思路如下：

1. 初步判断当前机器无法连通 RabbitMQ 所在机器，使用  `ping rabbitmq.xxx.com` 命令可以 ping 通 RabbitMQ 所在机器，并且  `telnet rabbitmq.xxx.com 5672` 也是可以连通的，排除此原因；

2. 确认 <code>username、password</code> 准确性，发现 RabbitMQ 是新装的，使用命令查看其用户：

   ```sh
   [root@rabbit ~]# rabbitmqctl list_users
   Listing users ...
   admin   [administrator]
   ```

   发现并没有 <code>user</code> 这个用户，创建用户：

   ```sh
   [root@rabbit ~]# rabbitmqctl add_user user password123456
   Creating user "user" ...
   ```

   设置为管理员：

   ```sh
   [root@rabbit ~]# rabbitmqctl set_user_tags user administrator
   Setting tags for user "user" to [administrator] ...
   ```

   此时，使用该用户登录 RabbitMQ WEB 端管理页面后查看该用户信息：
   <img src="https://tryme.wang/usr/images/sina/5cd95cde1e971.jpg" alt="image" align="center">
   点击 <code>Set permission</code> 配置权限，重启项目后问题解决。

---
### Tomcat 问题

**问题：** 启动 Tomcat 容器报错信息如下：

```java
org.apache.tomcat.jni.Error: 70023: This function has not been implemented on this platform
        at org.apache.tomcat.jni.SSL.initialize(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
```

**解决：** 修改 <code>$TOMCAT_HOME/conf/server.xml</code> 中的 `<Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="off" />` 即可解决。

### 线上问题处理

**问题：** 

[scode type="share"] MongoDB 版本：TokuMX 2.0.2 | MongoDB 2.4
 spring-data-mongodb 版本：1.4.1.RELEASE
 spring-data-commons 版本：1.7.2.RELEASE
 [/scode]

```java
java.lang.IllegalArgumentException: You have to provide at least one property to sort by!
        at org.springframework.data.domain.Sort.<init>(Sort.java:91)
        at org.springframework.data.domain.Sort.<init>(Sort.java:79)
    	...
        at sun.reflect.GeneratedMethodAccessor415.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
    	...
```

**解决：** 异常信息很明确，未提供排序的属性，检查代码，异常代码片段如下：

```java
	...
    @Autowired
    private MongoTemplate mongoTemplate;
	...
    Query query = new Query();
	List<String> sortStrList = new ArrayList<>();
	if (...) {
    	sortStrList.add("property");
    }
	query.with(new Sort(Sort.Direction.DESC, sortStrList.toArray(new String[sortStrList.size()])))
    mongoTemplate.find(query, Xxx.class);
	...
```

很明显，排序属性集合 <code>sortStrList</code> 中有可能没有任何属性，再查看 <code>org.springframework.data.domain.Sort</code> 中的构造方法：

```java
    public Sort(Sort.Direction direction, String... properties) {
        this(direction, (List)(properties == null ? new ArrayList() : Arrays.asList(properties)));
    }

    public Sort(Sort.Direction direction, List<String> properties) {
        if (properties != null && !properties.isEmpty()) {
            this.orders = new ArrayList(properties.size());
            Iterator i$ = properties.iterator();

            while(i$.hasNext()) {
                String property = (String)i$.next();
                this.orders.add(new Sort.Order(direction, property));
            }

        } else {
            throw new IllegalArgumentException("You have to provide at least one property to sort by!");
        }
    }
```

可以看出，<code>sortStrList</code> 完全无需转换为数组，但这不是主要原因，从下面的构造方法中可以看出，传入的 List 集合不可为空对象并且必须集合大小必须大于 0，所以解决方案可以有多种，只需控制 <code>sortStrList</code> 集合不为空且 size 大于 0，或者根据判断查询不加排序条件也可以解决。



------

**问题：** 

```java
org.springframework.data.mongodb.UncategorizedMongoDbException: Lock not granted. Try restarting the transaction.; nested exception is com.mongodb.MongoExcep
tion: Lock not granted. Try restarting the transaction.
        at org.springframework.data.mongodb.core.MongoExceptionTranslator.translateExceptionIfPossible(MongoExceptionTranslator.java:83)
        at org.springframework.data.mongodb.core.MongoTemplate.potentiallyConvertRuntimeException(MongoTemplate.java:1828)
        at org.springframework.data.mongodb.core.MongoTemplate.execute(MongoTemplate.java:409)
        at org.springframework.data.mongodb.core.MongoTemplate.doUpdate(MongoTemplate.java:995)
        at org.springframework.data.mongodb.core.MongoTemplate.updateFirst(MongoTemplate.java:969)
        ...
        at sun.reflect.GeneratedMethodAccessor378.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
        at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:317)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:183)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:150)
        at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:80)
        ...
        at sun.reflect.GeneratedMethodAccessor200.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
```

**解决：** 手动配置的 MongoDB 事务问题，<code>尚未彻底解决</code>。



------

**问题：** 

[scode type="share"] Spring 整合 MongoDB配置文件如下 [/scode]
 ```xml
 <bean id="mongoOptions" class="com.mongodb.MongoOptions">
 ...
 <!-- 连接超时时间(毫秒)，默认为4000 -->
 		<property name="connectTimeout" value="4000" />
 		<!-- socket读写时超时时间(毫秒)，默认为0，不超时 -->
 		<property name="socketTimeout" value="0" />
 		<!-- 是socket连接在防火墙上保持活动的特性，默认为false -->
 		<property name="socketKeepAlive" value="true" />
     	...
 </bean>
 ```


```java
org.springframework.dao.DataAccessResourceFailureException: can't say something; nested exception is com.mongodb.MongoException$Network: can't say something
        at org.springframework.data.mongodb.core.MongoExceptionTranslator.translateExceptionIfPossible(MongoExceptionTranslator.java:56)
        at org.springframework.data.mongodb.core.MongoTemplate.potentiallyConvertRuntimeException(MongoTemplate.java:1828)
        at org.springframework.data.mongodb.core.MongoTemplate.execute(MongoTemplate.java:409)
        at org.springframework.data.mongodb.core.MongoTemplate.insertDBObject(MongoTemplate.java:893)
        at org.springframework.data.mongodb.core.MongoTemplate.doInsert(MongoTemplate.java:713)
        at org.springframework.data.mongodb.core.MongoTemplate.insert(MongoTemplate.java:668)
        at org.springframework.data.mongodb.core.MongoTemplate.insert(MongoTemplate.java:659)
        at net.okdi.o2o.core.helper.ExceptionHelper.afterThrow(ExceptionHelper.java:71)
        at sun.reflect.GeneratedMethodAccessor438.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
        ...
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at java.lang.Thread.run(Thread.java:745)
Caused by: com.mongodb.MongoException$Network: can't say something
        at com.mongodb.DBTCPConnector.say(DBTCPConnector.java:194)
        at com.mongodb.DBTCPConnector.say(DBTCPConnector.java:155)
        at com.mongodb.DBApiLayer$MyCollection.insert(DBApiLayer.java:249)
        at com.mongodb.DBApiLayer$MyCollection.insert(DBApiLayer.java:205)
        at com.mongodb.DBCollection.insert(DBCollection.java:57)
        at com.mongodb.DBCollection.insert(DBCollection.java:100)
        at org.springframework.data.mongodb.core.MongoTemplate$8.doInCollection(MongoTemplate.java:898)
        at org.springframework.data.mongodb.core.MongoTemplate.execute(MongoTemplate.java:407)
        ... 44 more
Caused by: java.io.IOException: couldn't connect to [mongodb.xxx.com/192.168.1.22:27017] bc:java.net.SocketTimeoutException: connect timed out
        at com.mongodb.DBPort._open(DBPort.java:214)
        at com.mongodb.DBPort.go(DBPort.java:107)
        at com.mongodb.DBPort.go(DBPort.java:88)
        at com.mongodb.DBPort.findOne(DBPort.java:143)
        at com.mongodb.DBPort.runCommand(DBPort.java:148)
        at com.mongodb.DBPort.checkAuth(DBPort.java:307)
        at com.mongodb.DBTCPConnector.say(DBTCPConnector.java:180)
        ... 51 more
```

**解决：** 异常最后很明确的提示了 <code>connect timed out</code> 连接超时的问题，我们可以将 <code>connectTimeout</code> 连接超时属性适当扩大，如果是提示 <code>read timed out</code>，原因是在进行数据操作时过长时间没有返回结果，此时要修改 <code>socketTimeout</code> 属性了。

[参考文章](https://zyjustin9.iteye.com/blog/2017986)

