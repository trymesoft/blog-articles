Posted on 2018-09-26
> RSS 已经逐渐没落，被如今大数据信息流，推荐算法所取代。如今虽然用的人比较少，但是好在没那么多烦人的“专属广告”及宣称“千人千面”的??算法推荐。虽然有很多 RSS 提供商，例如 Feedly、Inoreader 等，也各有免费的方案，但是好的服务终究是走向了付费时代。本文主要介绍 Docker 下安装 Tiny Tiny RSS 的方法及开启 HTTPS  支持，Docker 可以理解为一种宿主机上的完全隔离的容器，安装卸载容器内的东西完全不影响宿主机。

## 开始安装

### 安装 Docker 环境
执行命令：
```bash
$ curl https://get.docker.io/ | sh
```
如果主机位于国内，可以切换为国内镜像安装脚本：
```bash
$ curl -sSL https://get.daocloud.io/docker | sh
```

### 开始部署
参考 [HenryQW](https://github.com/HenryQW/docker-ttrss-plugins) 的方法通过 docker-compose 部署，需要安装 docker-compose 命令，一步步执行以下命令：
```bash
$ yum -y install epel-release
$ yum -y install python-pip
$ pip install docker-compose
```
docker-compose.yml 内容：
```yml
version: "3"
services:
  database.postgres:
    image: sameersbn/postgresql:latest
    container_name: postgres
    environment:
      - PG_PASSWORD=tt-rss # please change the password
      - DB_EXTENSION=pg_trgm
    restart: always

  rss:
    image: wangqiru/ttrss:latest
    container_name: ttrss
    ports:
      - 181:80
    environment:
      - SELF_URL_PATH=https://rss.tryme.wang/ # please change to your own domain, keep https there and no port for https support later
      - DB_HOST=database.postgres
      - DB_PORT=5432
      - DB_NAME=ttrss
      - DB_USER=postgres
      - DB_PASS=tt-rss # please change the password
    stdin_open: true
    tty: true
    restart: always
    command: sh -c 'sh /wait-for database.postgres:5432 -- php /configure-db.php && exec s6-svscan /etc/s6/'
```
在 docker-compose.yml 目录下执行：
```bash
$ docker-compose up -d
```
等待部署完成。

### Nginx https 支持
添加 Nginx yum 资源库；
```bash
$ rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

使用 yum 命令安装 Nginx：
```bash
$ yum install -y nginx
```

启动 Nginx：
```bash
$ service nginx start
```
这时访问即可看到 "Welcome to Nginx" 欢迎页。

> 使用 [Let'sEncrypt](https://letsencrypt.org/) 提供的免费 HTTPS  证书
```bash
$ wget https://dl.eff.org/certbot-auto
$ chmod a+x ./certbot-auto
$ certbot-auto
```
根据提示生成即可。证书自动续期执行：
`$ ./certbot-auto renew --dry-run`

编辑 <code>/etc/nginx/conf.d/ttrss.conf</code>（没有的话手动创建），保存如下内容：
```bash
upstream ttrssdev {
  server 127.0.0.1:181;
}

server {
    listen 80;
    server_name  rss.tryme.wang;
    #return 301 https://rss.tryme.wang$request_uri;
    rewrite ^(.*)$ https://$host$1 permanent;
}

server {
    listen 443 ssl;
    gzip on;
    server_name  rss.tryme.wang;

    ssl_certificate /etc/letsencrypt/live/rss.tryme.wang/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/rss.tryme.wang/privkey.pem;

    access_log /var/log/nginx/ttrssdev_access.log combined;
    error_log  /var/log/nginx/ttrssdev_error.log;

    location / {
        proxy_redirect off;
        proxy_pass http://ttrssdev;

        proxy_set_header  Host                $http_host;
        proxy_set_header  X-Real-IP           $remote_addr;
        proxy_set_header  X-Forwarded-Ssl     on;
        proxy_set_header  X-Forwarded-For     $proxy_add_x_forwarded_for;
        proxy_set_header  X-Forwarded-Proto   $scheme;
        proxy_set_header  X-Frame-Options     SAMEORIGIN;

        client_max_body_size        100m;
        client_body_buffer_size     128k;

        proxy_buffer_size           4k;
        proxy_buffers               4 32k;
        proxy_busy_buffers_size     64k;
        proxy_temp_file_write_size  64k;
    }
}
```
> 确保 <code>/etc/nginx/nginx.conf</code> 主配置文件 http 块里有 "include /etc/nginx/conf.d/*.conf;"

验证并更新 Nginx 配置
```bash
$ nginx -t
$ nginx -s reload
```

这是直接访问 https://rss.tryme.wang 即可看到登录页面，默认用户名 admin，密码 password，建议立即修改默认密码！

### 可能出现的问题
GCP 的机器直接访问域名的时候 Nginx 的错误日志可能会出现下面问题：
```
*33 connect() to 127.0.0.1:181 failed (13: Permission denied) while connecting to upstream
```

> 解决方案参考  [地址](https://my.oschina.net/zk875/blog/823710)
