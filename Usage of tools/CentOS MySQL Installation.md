Posted on 2017-09-01
> Centos下安装MySQL比较麻烦，每一步都有坑，还是记下来比较好 O(∩_∩)O哈哈~

## 1. 下载安装包
[官网](https://dev.mysql.com/downloads/mysql/)下载文件

- 选择系统版本

![下载](http://p6arrbd5c.bkt.clouddn.com/centos/mysql/%E4%B8%8B%E8%BD%BD.png)

- 勾选如下选项下载：

![勾选](http://p6arrbd5c.bkt.clouddn.com/centos/mysql/%E5%8B%BE%E9%80%89.png)

---
## 2. 上传服务器并解压

**注：** CentOS7预装MariaDB，可能有冲突，先卸载  

```
$ rpm -qa |grep -i mariadb  
$ rpm -e --nodeps mariadb-libs-5.5.52-1.el7.x86_64`
```
再执行解压：  
`$ tar -xvf mysql-5.7.19-1.el7.x86_64.rpm-bundle.tar`
![解压](http://p6arrbd5c.bkt.clouddn.com/centos/mysql/%E8%A7%A3%E5%8E%8B.png)

---
## 3. 开始安装
执行以下命令：  
```
$ rpm -ivh mysql-community-common-5.7.19-1.el7.x86_64.rpm

$ rpm -ivh mysql-community-libs-5.7.19-1.el7.x86_64.rpm

$ rpm -ivh mysql-community-libs-compat-5.7.19-1.el7.x86_64.rpm 

$ rpm -ivh mysql-community-client-5.7.19-1.el7.x86_64.rpm

$ rpm -ivh mysql-community-server-5.7.19-1.el7.x86_64.rpm

$ rpm -ivh mysql-community-devel-5.7.19-1.el7.x86_64.rpm
```
倒数第二步可能会报如下**错误**：  
![错误](http://p6arrbd5c.bkt.clouddn.com/centos/mysql/%E9%94%99%E8%AF%AF.png)

**原因：缺少libaio库**  
**解决方案：安装libaio库 [点我](http://mirror.centos.org/centos/6/os/x86_64/Packages/libaio-0.3.107-10.el6.x86_64.rpm)**  
```
$ rpm -ivh libaio-0.3.107-10.el6.x86_64.rpm
```

---
## 4. 启动mysql服务并设置开机启动
```
$ systemctl start mysqld

$ systemctl enable mysqld

$ systemctl daemon-reload
```

---
## 5. 根据临时密码登录MySQL
`$ vi /var/log/mysqld.log`

![密码](http://p6arrbd5c.bkt.clouddn.com/centos/mysql/%E5%AF%86%E7%A0%81.png)  

---
## 6. 修改密码
登录后必须修改密码，不然无法执行sql语句，会报如下错误：  
`ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.`  
当执行  
`ALTER USER 'root'@'localhost' IDENTIFIED BY 'wang123';`  
修改密码时又会报一下错误：  
`ERROR 1819 (HY000): Your password does not satisfy the current policy requirements`  
这个错误与validate_password_policy密码策略有关  
执行以下代码修改策略，此时只会基于长度判断：  
`$ set global validate_password_policy=0;`  
**太复杂了，