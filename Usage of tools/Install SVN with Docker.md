### docker 安装 svn

> 记录一下使用 docker 镜像安装 svn 的流程

#### 查询 docker 镜像

```sh
$ docker search svn
```

 `garethflowers/svn-server` 是只支持 svn 协议的， `elleflorio/svn-server` 是内置了 apache，支持 HTTP 和 svn 双协议，因为需要 Nginx 转发，所以这里安装  `elleflorio/svn-server`

```sh
$ docker pull elleflorio/svn-server
```

#### 启动及配置

启动一下子看看：

```sh
$ docker run --restart always --name svn -d -v /usr/local/svn/repo:/home/svn -p 3680:80 -p 3690:3690 elleflorio/svn-server
```

进入容器：

```sh
$ docker exec -it svn sh
```

建立一个仓库：

```sh
$ svnadmin create /home/svn/rep
```

权限配置，编辑 `/etc/apache2/conf.d/dav_svn.conf`：

```xml
LoadModule dav_svn_module /usr/lib/apache2/mod_dav_svn.so
LoadModule authz_svn_module /usr/lib/apache2/mod_authz_svn.so

<Location /svn>
     DAV svn
     SVNParentPath /home/svn
     SVNListParentPath On
     AuthType Basic
     AuthName "Subversion Repository"
     AuthUserFile /home/svn/passwd.conf
     AuthzSVNAccessFile /home/svn/auth.conf
     Require valid-user
  </Location>
```

建立用户：

```sh
$ htpasswd -b /home/svn/passwd.conf admin 123456
```

配置访问权限，修改上面权限配置中的授权文件，即 `/home/svn/auth.conf`：

```shell
[repo:/]
admin=rw #即 admin 用户拥有 repo 仓库根目录下的 read 和 write 权限
```

以上用户权限以最简单为例，并未涉及到较复杂的用户分组之类的。

最后可以配置 Nginx 转发到主机的 3680 端口，即镜像内的 80 端口，就可使用 HTTP 访问 svn，server 块配置如下：

```sh
server {
    listen       80;
    server_name svn.xxx.com;

    location / {
        proxy_pass     http://127.0.0.1:3680;
    }
}
```

### 可能遇到的一些问题

#### Nginx 配置完毕后，无法通过 HTTP 请求访问域名

原因：容器 80 端口未被占用，即 apache 启动失败

解决方法：查看 /var/log/apache2/error.log 错误日志，可能会发现：

```shell
[:emerg] [pid 10619] AH00020: Configuration Failed, exiting
[core:emerg] [pid 10625] (28)No space left on device: AH00023: Couldn't create the authdigest-client mutex 
[auth_digest:error] [pid 10625] (28)No space left on device: AH01760: failed to create lock (client_lock) - all nonce-count checking 
and one-time noncesdisabled
```

创建 /run/apache2 目录：`mkdir -p /run/apache2/`

也可能会发现：

```shell
[:emerg] [pid 9595] AH00020: Configuration Failed, exiting
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.18.0.2. Set the 'ServerName' directive globally to suppress this messa
ge
```

顺手编辑 `/etc/apache2/httpd.conf`，将‘ServerName’修改为 172.18.0.2 即可。

#### Windows 客户端访问报错：`can't open file db/txn-current-lock:permission`

原因：权限问题

解决方法：

```sh
$ chmod 777 db
$ chmod -R o+rw /home/svn/repo/
```

