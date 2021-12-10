Posted on 2019-02-28
> 国内很多网盘已死，就剩一家作恶的还存在，不开会员速度限制的还死死的。试过 `Nextcloud`，但是会占用服务器的硬盘空间，后来改用 `OneIndex`，下载后直接上传到 `OneDrive`，不过还是不方便，而且巨硬的东西有问题都没地诉苦。
>
> 突发奇想，可不可以用家里的电脑当云端存储服务器，emm…，帝都联通光纤，关键是这玩意没个公网 IP，NAT 算上自己的路由器都有 3 层，没公网 IP 可不好搞了，还好 “GayHub” 神人多?。

### 准备工作

首先，多层 NAT 下弱小、可怜、无助的我的美帝良心想牌电脑，需要借助内网穿透工具，[frp](https://github.com/fatedier/frp) 刚好解决内网穿透的问题，当然，前提还是得有一台具有公网 IP 的机器，看下项目官方描述：

> frp is a fast reverse proxy to help you expose a local server behind a NAT or firewall to the internet. As of now, it supports tcp & udp, as well as http and https protocols, where requests can be forwarded to internal services by domain name.
>
> Now it also try to support p2p connect.
>
> (frp 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp 协议，为 http 和 https 应用协议提供了额外的能力，且尝试性支持了点对点穿透。)

这项目还有中文文档，很是贴心?，不过开发人员还是尽量阅读英文文档比较好。

内网穿透搞定了，下面还得有一个网盘客户端（立个 flag，等我??了，我自己写一个 WEB 端的），这种东西开源的很多，先用 `File Browser` 玩玩，项目[地址](https://github.com/filebrowser/filebrowser)，准备的差不多了，下面开搞！

### 配置 File Browser

配置可以参考项目[文档](https://docs.filebrowser.xyz/)，我就配置了个端口号和用户目录（用户目录还可以登录进去配置?）。

设置端口号（无所谓=_=）：

```shell
filebrowser config set port 8888
```

然后，Windows 下直接用命令启动即可（嗯，对，就是进去那个黑乎乎的窗口里）。

### 配置 frp 内网穿透

frp [文档](https://github.com/fatedier/frp/blob/master/README_zh.md)很详细，这里在内网的 Windows 下只需使用 `frpc.ini` 运行 `frpc.exe` 即可，上一个简单的配置：

```ini
[common]
server_addr = xx.xx.xx.xx  // 具有公网 IP 的服务器地址
server_port = 7000         // 公网服务器的端口

[web]
type = http  // HTTP 协议
local_port = 8888  // 本地服务端口号
custom_domains = www.yourdomain.com  // 自定义域名
```

然后，在具有公网 IP 的机器上配置 `frps`，`frps.ini` 配置如下：

```ini
[common]
bind_port = 7000  // 绑定的端口
vhost_http_port = 80  // 公网访问的端口
```

最后，记得将域名解析到该 IP 上，由于映射的 `80` 端口，然后直接访问 `http://www.yourdomain.com` 即可看到 File Browser 的登录页面，嗯，凑合着玩玩先。