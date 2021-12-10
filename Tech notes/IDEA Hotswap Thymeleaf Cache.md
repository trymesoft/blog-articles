Posted on 2019-01-12
> 使用 IDEA 热部署项目时，之前的项目配置完成后，页面文件改动或者 Java 文件改动都能即时热部署，但是在公司一个项目中，热部署突然失效，折腾了半天。

### 起因

因为要接手一个项目，在部署调试的过程中，发现页面的热部署在 IDEA 中不好用，当页面中的代码更改时，无论是 IDEA 手动 update 还是失去焦点，虽然出发了热部署，但是页面文件分明有了改动，但是总是提示 `Loaded classes are up to date.Nothing to reload`，很是奇怪?，后来改动 Java 文件，热部署没问题，真是哔了?了。

------

### 解决

开始以为是自己配置配置的问题，可之前的项目也是这么配置的，有问题早就暴露出来了，不应该是配置的问题。

在 `Dont't be evil --- Google` 的网站上遨游了一番，找了好多答案都不是想要的，终于在下班前夕被我找到了?，答案在[这里](https://stackoverflow.com/questions/50395971/unable-to-hotswap-html-file-in-intellij-with-tomcat-9)。

<img src="https://tryme.wang/usr/images/sina/5cd95d46d1bf7.jpg" alt="image" align="center">

原来是 `Thymeleaf` 的缓存?，赶紧去查看配置文件，发现了 `Thymeleaf` 的配置：

```xml
...
<bean id="templateResolver" class="org.thymeleaf.templateresolver.ServletContextTemplateResolver">
    ...
    <property name="cacheable" value="true"/>
    <property name="cacheTTLMs" value="#{60*60*1000}"/>
    ...
</bean>
...
```

都是这个 `cacheable` 的锅，改为 `false` ，重启后一切正常，又能愉快的玩耍了?。