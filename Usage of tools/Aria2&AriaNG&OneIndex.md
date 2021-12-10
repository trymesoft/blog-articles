Posted on 2018-11-14
## Aria2 介绍
> aria2 is a lightweight multi-protocol & multi-source command-line download utility. It supports HTTP/HTTPS, FTP, SFTP, BitTorrent and Metalink. aria2 can be manipulated via built-in JSON-RPC and XML-RPC interfaces.    (from [https://aria2.github.io](https://aria2.github.io))

## 安装 Aria2
下载脚本并运行
```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/aria2.sh && chmod +x aria2.sh && bash aria2.sh
```
运行脚本后会出现以下菜单选项：
```
 Aria2 一键安装管理脚本 [v1.1.9]
  -- Toyo | doub.io/shell-jc4 --
  
  0. 升级脚本
————————————
  1. 安装 Aria2
  2. 更新 Aria2
  3. 卸载 Aria2
————————————
  4. 启动 Aria2
  5. 停止 Aria2
  6. 重启 Aria2
————————————
  7. 修改 配置文件
  8. 查看 配置信息
  9. 查看 日志信息
 10. 配置 自动更新 BT-Tracker服务器
————————————

 当前状态: 已安装 并 已启动

 请输入数字 [0-10]:
```
按提示操作即可。
PS:
**启动：** /etc/init.d/aria2 start
**停止：** /etc/init.d/aria2 stop
**当前状态：** /etc/init.d/aria2 status
**配置文件路径：** /root/.aria2/aria2.conf
**令牌密钥：**随机生成，可以更改
**下载目录：**/usr/local/caddy/www/aria2/Download
来源：<code>[逗比教程](https://doub.io/shell-jc4/)</code>

---
## 安装 AriaNG
> 由于 Aria2 是一个命令行下载工具，每次使用敲命令下载文件不方便，有大佬开发出的开源 Web 面板来管理下载任务。其中之一便是 [AriaNG](https://github.com/mayswind/AriaNg)
AriaNG 只是一个 Web 端的管理面板，可以远程连接 Aria2，二者可以分处不同服务器。

直接从 [GitHub](https://github.com/mayswind/AriaNg/releases) 项目主页下载解压到服务器即可，此时即可通过 IP 或配置好的域名进行访问。
具体效果可访问 <code>http://aria2.tryme.wang</code>查看

---
## 安装 OneIndex
可以使用宝塔面板新建网站，全程 Web 面板操作，简单粗暴。安装参考：[宝塔](https://bt.cn)
OneIndex 项目主页：[https://github.com/donwa/oneindex](https://github.com/donwa/oneindex)
<img src="https://tryme.wang/usr/images/sina/5cd95cd1836b6.jpg" align="center" alt="image">
下载：
```bash
git clone https://github.com/donwa/oneindex.git
```
移动到网站根目录后要分配权限：
```bash
chmod 777 ./config && chmod 777 ./cache
```
此时根据配置的域名，访问 <code> http://xxx.com/?/admin</code>，默认的密码是 <code>oneindex</code>，按照提示配置即可。
PS:可以配置 Nginx 伪静态去掉管理页面 URL 中的 "?"，伪静态配置如下：
```bash
 if (!-e $request_filename) {
     rewrite / /?/ last;
 } 
```

---
## 配置 Aria2 下载后自动上传
新建一个脚本文件：
```bash
vi /root/.aria2/oneindexup.sh
```
复制以下内容：
```bash
#!/bin/bash
path=$3 #取原始路径，我的环境下如果是单文件则为/data/demo.png,如果是文件夹则该值为文件夹内某个文件比如/data/a/b/c/d.jpg
downloadpath='/usr/local/caddy/www/aria2/Download'  #修改成Aria2下载文件夹
domain='tryme.wang'  #修改成自己域名

if [ $2 -eq 0 ]
        then
                exit 0
fi
while true; do  #提取下载文件根路径，如把/data/a/b/c/d.jpg变成/data/a
filepath=$path
path=${path%/*}; 
if [ "$path" = "$downloadpath" ] && [ $2 -eq 1 ]  #如果下载的是单个文件
    then
    php /www/wwwroot/$domain/one.php upload:file $filepath /$folder/
    rm -rf $filepath
    php /www/wwwroot/$domain/one.php cache:refresh
    exit 0
elif [ "$path" = "$downloadpath" ]
    then
    php /www/wwwroot/$domain/one.php upload:folder $filepath /$folder/
    rm -rf "$filepath/"
    php /www/wwwroot/$domain/one.php cache:refresh
    exit 0
fi
done
```
给脚本分配可执行权限：
```bash
chmod +x /root/.aria2/oneindexup.sh
```
最后写入配置文件，即上面安装 Aria2 时的配置文件<code>/root/.aria2/aria2.conf</code>
编辑配置文件或者直接命令：
```bash
echo "on-download-complete=/root/.aria2/oneindexup.sh" >>/root/.aria2/aria2.conf
```
<code>参考自：[Rat‘s大佬](https://www.moerats.com/archives/700/)</code>
