Posted on 2019-01-16
> 某个项目启动时超慢，启动到一半进程自动停掉，有的时候十分钟才能启动，重复测试后发现是数据库连接的问题。

## 环境

- Spring Framework 整合 Druid 配置 DataSource

- ```xml
  <bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
        init-method="init" destroy-method="close">
      ...
      <property name="validationQuery" value="SELECT 1" />
      <property name="testOnBorrow" value="false" />
      <property name="testOnReturn" value="false" />
      <property name="testWhileIdle" value="true" />
      <property name="timeBetweenEvictionRunsMillis" value="60000" />
      <property name="minEvictableIdleTimeMillis" value="25200000" />
      ...
  </bean>
  ```

- MySQL 5.7

------

### 问题

异常日志：

```java
The last packet successfully received from the server was 540,992 milliseconds ago.  The last packet sent successfully to the server was 540,993 milliseconds ago.
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:411)
	at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:1117)
	at com.mysql.jdbc.MysqlIO.nextRowFast(MysqlIO.java:2144)
	at com.mysql.jdbc.MysqlIO.nextRow(MysqlIO.java:1900)
	at com.mysql.jdbc.MysqlIO.readSingleRowSet(MysqlIO.java:3390)
	at com.mysql.jdbc.MysqlIO.getResultSet(MysqlIO.java:483)
	at com.mysql.jdbc.MysqlIO.readResultsForQueryOrUpdate(MysqlIO.java:3096)
	at com.mysql.jdbc.MysqlIO.readAllResults(MysqlIO.java:2266)
	at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2687)
	at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2719)
	at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:2155)
	at com.mysql.jdbc.PreparedStatement.execute(PreparedStatement.java:1379)
	at com.alibaba.druid.filter.FilterChainImpl.preparedStatement_execute(FilterChainImpl.java:2931)
	at com.alibaba.druid.filter.FilterEventAdapter.preparedStatement_execute(FilterEventAdapter.java:440)
	at com.alibaba.druid.filter.FilterChainImpl.preparedStatement_execute(FilterChainImpl.java:2929)
	at com.alibaba.druid.filter.FilterEventAdapter.preparedStatement_execute(FilterEventAdapter.java:440)
	at com.alibaba.druid.filter.FilterChainImpl.preparedStatement_execute(FilterChainImpl.java:2929)
	at com.alibaba.druid.proxy.jdbc.PreparedStatementProxyImpl.execute(PreparedStatementProxyImpl.java:118)
	at com.alibaba.druid.pool.DruidPooledPreparedStatement.execute(DruidPooledPreparedStatement.java:493)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.ibatis.logging.jdbc.PreparedStatementLogger.invoke(PreparedStatementLogger.java:62)
	at com.sun.proxy.$Proxy32.execute(Unknown Source)
	at org.apache.ibatis.executor.statement.PreparedStatementHandler.query(PreparedStatementHandler.java:59)
	at org.apache.ibatis.executor.statement.RoutingStatementHandler.query(RoutingStatementHandler.java:73)
	at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:60)
	at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:267)
	at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:137)
	at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:96)
	at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:77)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:108)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:102)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:358)
	... 84 more
Caused by: java.io.EOFException: Can not read response from server. Expected to read 4 bytes, read 0 bytes before connection was unexpectedly lost.
	at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:3017)
	at com.mysql.jdbc.MysqlIO.nextRowFast(MysqlIO.java:1985)
	... 120 more
2019-01-16 14:40:03,649 ERROR [com.alibaba.druid.pool.DruidDataSource] - create connection error
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:411)
	at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:1117)
	at com.mysql.jdbc.MysqlIO.<init>(MysqlIO.java:350)
	at com.mysql.jdbc.ConnectionImpl.coreConnect(ConnectionImpl.java:2393)
	at com.mysql.jdbc.ConnectionImpl.connectOneTryOnly(ConnectionImpl.java:2430)
	at com.mysql.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:2215)
	at com.mysql.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:813)
	at com.mysql.jdbc.JDBC4Connection.<init>(JDBC4Connection.java:47)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:411)
	at com.mysql.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:399)
	at com.mysql.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:334)
	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:148)
	at com.alibaba.druid.filter.FilterAdapter.connection_connect(FilterAdapter.java:786)
	at com.alibaba.druid.filter.FilterEventAdapter.connection_connect(FilterEventAdapter.java:38)
	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:142)
	at com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:211)
	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:142)
	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1360)
	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1414)
	at com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:1649)
Caused by: java.net.ConnectException: Connection refused (Connection refused)
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:589)
	at java.net.Socket.connect(Socket.java:538)
	at java.net.Socket.<init>(Socket.java:434)
	at java.net.Socket.<init>(Socket.java:244)
	at com.mysql.jdbc.StandardSocketFactory.connect(StandardSocketFactory.java:257)
	at com.mysql.jdbc.MysqlIO.<init>(MysqlIO.java:300)
	... 21 more
2019-01-16 14:40:03,666 ERROR [com.alibaba.druid.pool.DruidDataSource] - create connection error
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
```

日志中可以看出是数据库连接的问题

------

### 解决

#### 数据库层面

查阅资料发现是应用打开数据库连接后，操作数据库时，连接超时被 MySQL 关闭了，查看 MySQL [官方文档](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_wait_timeout)，查看超时配置：

<img src="https://tryme.wang/usr/images/sina/5cd95d48c2d05.jpg" alt="image" align="center">

可以更改 MySQL 配置文件将改属性值扩大，或者直接使用 `set` 命令而无需重启数据库。

#### 应用配置层面

将 `DruidDataSource` 中的属性 `testOnBorrow(申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。)`、`testOnReturn(归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。)`改为 `true`，重启应用即可。

将数据库连接 URL 后加上 `autoReconnect=true`，当连接断开时自动重连。