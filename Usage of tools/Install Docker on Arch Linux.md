## 系统信息

```bash
[root@archlinux ~]# uname -a
Linux archlinux 5.7.12-arch1-1 #1 SMP PREEMPT Fri, 31 Jul 2020 17:38:22 +0000 x86_64 GNU/Linux
```

## 切换国内源

1. 编辑 `/etc/pacman.d/mirrorlist` 文件，加入 `Server = https:``//mirrors``.tuna.tsinghua.edu.cn``/archlinuxcn/``$arch` 到开头，保存并退出。
2. 更新源，执行 `sudo pacman -Sy`

## 安装 Docker

直接执行

```bash
$ pacman -S docker 
```

如果使用 AUR 包，执行

```bash
$ yaourt -S docker-git
```

二者区别是第一种是安装最新版本的 Docker，后者则是由当前 master 分支构建的包。

## 启动

执行 `systemctl start docker`，但出现报错信息：

```bash
[root@archlinux ~]# systemctl start docker
Job for docker.service failed because the control process exited with error code.
See "systemctl status docker.service" and "journalctl -xe" for details.
```

按提示执行 `systemctl status docker.service`，信息如下：

```bash
[root@archlinux ~]# systemctl status docker.service
* docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
     Active: failed (Result: exit-code) since Mon 2020-09-14 15:10:59 CST; 2min 9s ago
TriggeredBy: * docker.socket
       Docs: https://docs.docker.com
    Process: 5973 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
   Main PID: 5973 (code=exited, status=1/FAILURE)

Sep 14 15:10:59 archlinux systemd[1]: docker.service: Scheduled restart job, restart counter is at 3.
Sep 14 15:10:59 archlinux systemd[1]: Stopped Docker Application Container Engine.
Sep 14 15:10:59 archlinux systemd[1]: docker.service: Start request repeated too quickly.
Sep 14 15:10:59 archlinux systemd[1]: docker.service: Failed with result 'exit-code'.
Sep 14 15:10:59 archlinux systemd[1]: Failed to start Docker Application Container Engine.
```

好像也没看出来啥有用的信息，继续执行 `journalctl -xe`，信息如下；

```bash
[root@archlinux ~]# journalctl -xe
Sep 14 15:10:59 archlinux systemd[1]: Stopping Docker Socket for the API.
-- Subject: A stop job for unit docker.socket has begun execution
-- Defined-By: systemd
-- Support: https://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- A stop job for unit docker.socket has begun execution.
--
-- The job identifier is 2359.
Sep 14 15:10:59 archlinux systemd[1]: Starting Docker Socket for the API.
-- Subject: A start job for unit docker.socket has begun execution
-- Defined-By: systemd
-- Support: https://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- A start job for unit docker.socket has begun execution.
--
-- The job identifier is 2359.
Sep 14 15:10:59 archlinux systemd[1]: Listening on Docker Socket for the API.
-- Subject: A start job for unit docker.socket has finished successfully
-- Defined-By: systemd
-- Support: https://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- A start job for unit docker.socket has finished successfully.
--
-- The job identifier is 2359.
Sep 14 15:10:59 archlinux systemd[1]: docker.service: Start request repeated too quickly.
Sep 14 15:10:59 archlinux systemd[1]: docker.service: Failed with result 'exit-code'.
-- Subject: Unit failed
-- Defined-By: systemd
-- Support: https://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- The unit docker.service has entered the 'failed' state with result 'exit-code'.
Sep 14 15:10:59 archlinux systemd[1]: Failed to start Docker Application Container Engine.
-- Subject: A start job for unit docker.service has failed
-- Defined-By: systemd
-- Support: https://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- A start job for unit docker.service has finished with a failure.
--
-- The job identifier is 2287 and the job result is failed.
Sep 14 15:10:59 archlinux systemd[1]: docker.socket: Failed with result 'service-start-limit-hit'.
-- Subject: Unit failed
-- Defined-By: systemd
-- Support: https://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- The unit docker.socket has entered the 'failed' state with result 'service-start-limit-hit'.
```

搜索相关错误，在 https://github.com/docker/for-linux/issues/820 发现了疑似解决方法，执行 `journalctl -fu docker`，出现如下信息：

```bash
[root@archlinux ~]# journalctl -fu docker
-- Logs begin at Wed 2020-08-05 16:20:46 CST. --
Sep 14 15:10:57 archlinux dockerd[5973]: Perhaps iptables or your kernel needs to be upgraded.
Sep 14 15:10:57 archlinux dockerd[5973]:  (exit status 3)
Sep 14 15:10:57 archlinux systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
Sep 14 15:10:57 archlinux systemd[1]: docker.service: Failed with result 'exit-code'.
Sep 14 15:10:57 archlinux systemd[1]: Failed to start Docker Application Container Engine.
Sep 14 15:10:59 archlinux systemd[1]: docker.service: Scheduled restart job, restart counter is at 3.
Sep 14 15:10:59 archlinux systemd[1]: Stopped Docker Application Container Engine.
Sep 14 15:10:59 archlinux systemd[1]: docker.service: Start request repeated too quickly.
Sep 14 15:10:59 archlinux systemd[1]: docker.service: Failed with result 'exit-code'.
Sep 14 15:10:59 archlinux systemd[1]: Failed to start Docker Application Container Engine.
```

看起来是要升级 `iptables` 或 `kernel`，可是在安装 Docker 之前我已经执行过 `pacman -Syu`：

```bash
[root@archlinux ~]# pacman -Syu
:: Synchronizing package databases...
 core is up to date
 extra is up to date
 community is up to date
:: Starting full system upgrade...
 there is nothing to do
```

而且报错信息里并没有明显的 `iptables` 或 `kernel` 的错，这时候当我执行伟大的 `reboot` 命令后，竟然不报错了。。。。。。

执行 `systemctl enable docker` 设置开机启动。

## 总结

Arch 装东西真是简单，遇到问题还是重启好使?