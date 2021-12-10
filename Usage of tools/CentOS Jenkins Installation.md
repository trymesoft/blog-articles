Posted on 2018-02-26
## 下载
下载Jenkins的war包，[地址](https://jenkins.io/download/) 新的Jenkins的war包需要jdk1.8的支持，如果想在jdk1.7的环境下使用，可以下载Jenkins2.19.3的包  
![download](http://p6arrbd5c.bkt.clouddn.com/centos/jenkins/download.png)  
 点击如上链接可以进入选择旧版本的war包进行下载。
 
---
## 启动
可将Jenkins直接放到tomcat容器中，启动tomcat容器即可自动部署，启动时会生成一个默认密码如下  
![start](http://p6arrbd5c.bkt.clouddn.com/centos/jenkins/start.png)  
此时可以直接访问了，第一次访问Jenkins会自动下载一些东西，等待片刻即可  
![visit](http://p6arrbd5c.bkt.clouddn.com/centos/jenkins/visit.png)   
此时可以访问Jenkins页面，输入启动时控制台打印的随机密码即可  
![go](http://p6arrbd5c.bkt.clouddn.com/centos/jenkins/go.png)

---
## 开始使用
输入完密码后出现如下页面  
![in](http://p6arrbd5c.bkt.clouddn.com/centos/jenkins/in.png)  
可根据个人使用习惯选择安装建议的插件或者自定义安装插件，后续仍可在插件管理中管理插件，这里我选择第一个。  
自动安装一些插件，安装完成后页面：  
![down](http://p6arrbd5c.bkt.clouddn.com/centos/jenkins/done.png)  
进入后，选择性的安装插件即可使用！