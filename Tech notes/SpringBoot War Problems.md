Posted on 2018-11-30
> Spring Boot 支持 JAR 包和 war 包两种打包方式。JAR 包方式运行简单，对于轻量的项目，从构造到部署极大的提升了效率；war 包方式对于启动参数、集群配置来说可能更方便一些。在使用 war 包方式的时候遇到了一个问题，记录下来。

## 前提概要
- Spring Boot 版本 <code>2.0.3.RELEASE</code>
- 打包方式  ```<packaging>war</packaging>```
- JDK1.8 编码 UTF-8
- 依赖只引入了 ``` spring-boot-starter-web、spring-boot-starter-test```
- Tomcat 版本 7.0.82

## 问题
配置好项目及 Tomcat 部署后启动失败，有这么一行异常信息：
```
Caused by: java.lang.NoClassDefFoundError: javax/el/ELManager
```

## 解决路径
### 缺少 JAR 包
第一反应，缺少 JAR 包，在 pom 文件引入：
```
<dependency>
    <groupId>javax.el</groupId>
    <artifactId>javax.el-api</artifactId>
    <version>3.0.0</version>
</dependency>
```
重新启动后问题仍然存在，加入 ` <scope>provided</scope>`  后仍然不行，<code>失败</code>。

### hibernate-validator 版本过高
搜索后有文章说是 <code>当前版本的 Spring Boot，依赖的验证 hibernate-validator 版本太高，导致找不到 ELManager。</code>查看项目依赖树，发现是 Web 依赖中的
<img src="https://tryme.wang/usr/images/sina/5cd95cd384355.jpg" align="center" alt="image">
查看其他项目中同样写法并不报错，排除这种原因。

### Tomcat 版本
有人说 Tomcat7 提供的 el-api.jar 版本是 2.2，Tomcat8 的版本是 3.0，未尝试切换 Tomcat 版本。

### 最终解决
将下载的 el-api3.0 的 JAR 包拷贝到 $TOMCAT_HOME/lib 下替换之前的 el-api.jar 即可。

参考地址：[Stack Overflow](https://stackoverflow.com/questions/45841464/java-lang-noclassdeffounderror-javax-el-elmanager)
