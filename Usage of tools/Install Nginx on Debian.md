
> 服务器每次安装 `Nginx`的时候都没有可供参考的详细文档，手写一份文档以供参考。

## 安装准备

一台联网的服务器：本机为纯净的 Debian 10.10(Kernel 4.19.0-17-amd64)

[Nginx](https://nginx.org/en/download.html) 最新稳定版：V1.20

## 安装

### 下载 Nginx

```sh
$ wget https://nginx.org/download/nginx-1.20.2.tar.gz && tar -zxvf nginx-1.20.2.tar.gz && cd nginx-1.20.2
```

### 配置

```sh
$ ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_v2_module --with-stream --with-stream=dynamic --with-stream_ssl_module
```

参数含义（[参考](https://nginx.org/en/docs/configure.html)）：

`--prefix`:安装目录

`--with-http_ssl_module`:HTTPS 支持模块

`--with-http_v2_module`:HTTP/2 支持模块

`--with-stream --with-stream=dynamic --with-stream_ssl_module`:TCP/UDP 代理和负载均衡及 SSL/TLS 支持

由于安装了额外模块，需要同步安装依赖，可能会出现如下错误信息：

#### 缺少 `pcre`依赖

  ![no-pcre](https://resource.tryme.wang/image/2021/11/nginx-no-pcre.png)

  执行命令安装`pcre`依赖即可：

  ```sh
  # 更新源
  $ sudo apt update
  $ sudo apt install libpcre3 libpcre3-dev
  ```

#### 缺少`openssl`依赖

  ![no-openssl](https://resource.tryme.wang/image/2021/11/nginx-no-openssl.png)

  安装依赖：

  ```sh
  $ sudo apt install openssl libssl-dev
  ```

#### 缺少`zlib`依赖

  ![no-zlib](https://resource.tryme.wang/image/2021/11/nginx-no-zlib.png)

  安装依赖：

  ```sh
  $ sudo apt install zlib1g-dev
  ```

编译成功的实例：

![all-done](https://resource.tryme.wang/image/2021/11/nginx-all-done.png)

### 编译

在 Nginx 目录下执行：

```sh
$ make && make install
```

此时，`/usr/local/nginx`目录已经有了安装好的 Nginx 可以使用了，为了使用方便，在 `/usr/sbin`目录下建立 Nginx 可执行文件软链接：

```sh
$ ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx
```

此时，直接执行 `nginx`，即可启动 Nginx。

