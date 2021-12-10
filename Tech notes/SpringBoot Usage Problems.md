Posted on 2018-12-01
> Spring Boot 使用中遇到的问题总结，持续更新...

### IDEA 中读取配置文件问题
问题如图：
<img src="https://tryme.wang/usr/images/sina/5cd95cd4cf68f.jpg" align="center" alt="image">
解决方案：
引入如下依赖
```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
</dependency>
```
此时会出现如下图：
<img src="https://tryme.wang/usr/images/sina/5cd95cd5e0508.jpg" align="center" alt="image">
可以直接忽略。

---
### Tomcat 容器启动过慢的问题
启动项目时，明明一个很简单的项目，却启动很慢，Tomcat 日志中有这么一行：
```
WARNING: Creation of SecureRandom instance for session ID generation using [SHA1PRNG] took [20,617] milliseconds.
```
启动成功耗时：
```
INFO: Server startup in 33239 ms
```
这个警告的操作耗了大半的启动时间，查阅 Tomcat 的 [wiki](https://wiki.apache.org/tomcat/HowTo/FasterStartUp)发现这么一段话：
<img src="https://tryme.wang/usr/images/sina/5cd95cd729506.jpg" align="center" alt="image">
解释得很清楚，Tomcat 启动时熵源的生成默认是 JRE 的阻塞式熵源（/ dev / random），替换为非阻塞式熵源（/ dev /./ urandom）即可，但是会降低安全性，因为获得的随机数据较少。
修改方式：
直接修改 $JAVA_HOME/jre/lib/security/java.security 中
```
securerandom.source=file:/dev/./urandom
```
或者直接执行
```
java -jar app.jar -Djava.security.egd=file:/dev/./urandom
```
修改完再启动：
```
INFO: Server startup in 10847 ms
```
效果还是很显著的?。

---
### 静态变量的注入
日常开发中，一般常用的工具会抽取到工具类中，工具类中的方法一般都是静态调用的，方法中可能会注入其他成员变量，由于方法是 static 的，所以成员变量也必须是静态的。此时，使用 @Autowired 已经无法注入，此时就需要新的操作了?。
首先，将需要使用的定义为静态变量：
```
private static RestTemplate restTemplate;
```
然后生成 Setter 方法并在方法上加 @Autowired（读取属性文件的话使用 @Value）注解（注意不要加 static）：
```
@Autowired
 public void setRestTemplate(RestTemplate restTemplate) {
        AuthTokenUtil.restTemplate = restTemplate;
}
```
最后，类上加上 @Component注解即可。

---
### RestTemplate 的 GET 请求的 Headers 设置

#### RestTemplate 介绍

> **Spring's central class for synchronous client-side HTTP access.** It simplifies communication with HTTP servers, and enforces RESTful principles. It handles HTTP connections, leaving application code to provide URLs (with possible template variables) and extract results.
>
> Spring 的同步客户端 HTTP 访问的中心类。它简化了与 HTTP 服务器的通信，并实施了 RESTful 原则。它处理 HTTP 连接，使应用程序代码提供 URL（带有可能的模板变量）并提取结果。

#### 环境配置

- Spring Boot 版本 2.0.3.RELEASE
- spring-web-5.0.7.RELEASE.jar 的 RestTemplate
- RestTemplate 的配置

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.ClientHttpRequestFactory;
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

/**
 * @author tryme.wang
 * @create 2018-11-30 10:59:48
 * @description RestTemplate配置 
 **/
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
        RestTemplate restTemplate = new RestTemplate(factory);
        return restTemplate;
    }

    @Bean
    public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
        /**
         * 默认情况下，RestTemplate依赖于标准JDK工具来建立HTTP连接。
         * 可以通过InterceptingHttpAccessor.setRequestFactory（org.springframework.http.client.ClientHttpRequestFactory）
         * 属性切换到使用不同的HTTP库，例如Apache HttpComponents，Netty和OkHttp。
         */
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        // 单位：ms
        factory.setReadTimeout(5000);
        // 单位：ms
        factory.setConnectTimeout(15000);
        return factory;
    }
}
```

#### 方案

RestTemplate 使用时可能会有 Headers 中需要传入 token 之类参数的情况，使用 POST 请求时，RestTemplate 的 API 中有明显的方法postForEntity或postForObject：

<img src="https://tryme.wang/usr/images/sina/5cd95cd8b1721.jpg" align="center" alt="image">

然而 GET 请求并没有明显可以理解的 API 使用，查阅资料发现了 exchange 方法可以指定 HTTP 请求方式：

<img src="https://tryme.wang/usr/images/sina/5cd95cd9c78e3.jpg" align="center" alt="image">

OK，就它了，而且其他请求方式均可以使用这个来为请求添加 Headers。
