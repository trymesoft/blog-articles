> GitLab 是使用 Docker 安装，由于版本太低，导致某些功能使用受限，需要升级。记录下升级全过程。

## 准备工作

### 当前环境

- 服务器：Ubuntu 16.04.3 LTS
- Docker 版本：1.13.1
- GitLab 版本：CE 10.5.3

### 修改 Docker pull 镜像源

由于 Docker Hub 对镜像拉取限速，经过对阿里、中科大、清华、网易等镜像源测试，网易最快。

编辑 `/etc/docker/daemon.json`，加入网易镜像地址：

```json
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

使配置生效：

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl docker restart
```

执行`docker info`查看是否生效：

```bash
$ docker info
...
Registry Mirrors:
 http://hub-mirror.c.163.com
...
```

## 备份 GitLab 数据

此部分直接参考官方文档：https://docs.gitlab.com/ce/raketasks/backup_restore.html，由于版本低于 12.2，所以使用 `gitlab-rake`。

```bash
$ docker exec -t gitlab-container gitlab-rake gitlab:backup:create
```

备份成功的文件位于`/var/opt/gitlab/backups`文件夹下。

## 恢复数据

备份文件恢复时，GitLab 的版本必须和备份时的版本一致，先恢复出数据，然后才可升级。由于备份时所用的版本是`10.5.3`,所以必须在同版本的 GitLab 下恢复数据。

拉取镜像：

```bash
$ docker pull gitlab/gitlab-ce:10.5.3-ce.0
```

启动容器：

```bash
$ sudo docker run --detach  --hostname gitlab.xxx.com  --publish 9443:443 --publish 9080:80 --publish 9022:22   --name gitlab  --restart always   --volume /srv/gitlab/config:/etc/gitlab   --volume /srv/gitlab/logs:/var/log/gitlab   --volume /srv/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:10.5.3-ce.0
```

复制备份文件到指定目录：

```bash
$ sudo cp 1616412017_2021_03_22_10.5.3_gitlab_backup.tar /srv/gitlab/data/backups/
```

进入容器，关闭与数据库有关的服务（也可直接不进入容器直接运行命令）：

```bash
$ docker exec -it gitlab /bin/sh
$ gitlab-ctl stop unicorn
$ gitlab-ctl stop puma
$ gitlab-ctl stop sidekiq
# Verify
$ gitlab-ctl status
```

恢复数据（注意文件名）：

```bash
$ gitlab-rake gitlab:backup:restore 1616412017_2021_03_22_10.5.3
```

重启并检查：

```bash
$ docker restart gitlab
$ docker exec -it gitlab gitlab-rake gitlab:check SANITIZE=true
```

## 升级 Docker 中的 GitLab 镜像

GitLab 不可直接升级到最新版，需要多个中间版本过渡，否则会升级失败（各个大版本的数据表结构甚至都不相同）。此处可参考官方的升级路线：https://docs.gitlab.com/ce/update/#upgrade-paths。

<img src="https://resource.tryme.wang/image/gitlab-upgrade-paths.png" alt="gitlab-upgrade-paths" alith="center" />

所以很明显，从 10.5.3 升级到最新版本路线是：`10.5.3`->`10.8.7`->`11.11.8`->`12.0.12`->`12.1.17`->`12.10.14`->`13.0.14`->`13.1.11`->`13.10.0`。

接下来就是无脑的`停止并删除容器->拉取指定版本镜像->启动容器`的循环了：

```bash
$ docker stop gitlab
$ docker rm gitlab
$ docker pull gitlab/gitlab-ce:X.X.X
$ sudo docker run --detach  --hostname gitlab.xxx.com  --publish 9443:443 --publish 9080:80 --publish 9022:22   --name gitlab  --restart always   --volume /srv/gitlab/config:/etc/gitlab   --volume /srv/gitlab/logs:/var/log/gitlab   --volume /srv/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:X.X.X
```

最终，升级完毕：

<img src="https://resource.tryme.wang/image/gitlab-version.png" style="zoom:50%;" />

