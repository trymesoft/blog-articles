Posted on 2018-12-04
> 在使用 Dubbo  时，如果配置了 AOP，会导致 Dubbo 发布的服务无法暴露的问题，查阅资料后，解决思路如下。

### 配置方式
Dubbo 配置：
```
# dubbo配置
spring: 
  dubbo:
    application:
      id: im-provider
      name: im-provider
      owner: tryme.wang
    registry:
      protocol: zookeeper
      address: 192.168.31.203:3182
    protocol:
      threads: 200
      id: dubbo
      name: dubbo
      port: 20891
    scan:
      basePackages: im.*.service
```
接口的实现：
```
import com.alibaba.dubbo.config.annotation.Service;
import im.v1.service.SendMessageService;

@Component
@Service(timeout = 5000, version = "1.0.0")
public class SendMessageServiceImpl implements SendMessageService {
    @Override
    public String sendMessage(String message) {
        return "done"
    }
}
```
application：
```
@SpringBootApplication
@EnableDubboConfiguration
public class ImApplication {
    public static void main(String[] args) {
  SpringApplication.run(ImApplication.class, args);
    }
}

```
AOP 配置：
```
spring:
    aop:
        proxy-target-class: true
```

```
@Configuration
@Aspect
public class ExceptionAspect {
  @Pointcut("execution(*im.v1.service.*.*(..))")
    public void services() {
    }

    @AfterThrowing(value = "services()", throwing = "exception")
    public void afterThrowing(JoinPoint joinPoint, Exception exception) {
        // 处理异常逻辑
}
```
项目启动后，查看 dubbo-admin 页面发布的接口，发现并没有 SendMessageService
<img src="https://tryme.wang/usr/images/sina/5cd95cda7e627.jpg" align="center" alt="image">

---
### 解决
开始以为是因为指定了 AOP 的代理方式为 cglib（proxy-target-class: true），在增强 SendMessageService 的时候会生成它的子类，Dubbo 扫描的时候无法扫描到导致的，后来点进 Dubbo 的 @Service 注解源码，发现有 @Inherited 这个注解，即子类可以继承使用该元注解的注解，问题不在这里
<img src="https://tryme.wang/usr/images/sina/5cd95cdb98d02.jpg" align="center" alt="image">

√> 发现 @Service 注解中有 interfaceName 属性，尝试指定接口名 interfaceName="im.v1.service.SendMessageService" 后问题解决。