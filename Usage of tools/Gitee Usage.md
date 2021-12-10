Posted on 2018-03-27
## 概要

使用 GitHub 时，经常遇到速度慢甚至无法连接的情况👀，而且 GitHub 创建私有仓库是需要收费的，有以上需求的可以考虑使用国内的 Git 托管服务 [码云](https://gitee.com/)。

> 相比于 GitHub ，码云有以下优势：
> - 首先，在语言的使用上，当然中文交流更畅快、更有效率；
> - 其次，国内 IT 行业有自己的特点，天然决定了对开源软件的需求也有自己的特点，比如小程序这个东西，只有国内有，做个开源的小程序托管在码云比 GitHub 明显更有优势，因为关注着都是国内的开发者；
> - 然后，开源不仅仅是大项目，像 OpenStack、Tensorflow 这样的大厂大作，确实有 GitHub 就够了，但是这样的大型项目毕竟不是普遍情况。现在编程越来越普及，每个人都可以参与和贡献开源项目，去做一些有意思的东西分享出来，那么从受众、交流便利度、访问速度等方面，码云都有优势

---
## 配置公钥
macOS下运行
`$ ssh-keygen -t rsa -C "XXXXX"`

<img src="https://tryme.wang/usr/images/sina/5cd95c7f56d47.jpg" align="center" alt="image">

将生成的公钥内容按照如下方式配置到个人码云上：

<img src="https://tryme.wang/usr/images/sina/5cd95c802b424.jpg" align="center" alt="image">

<img src="https://tryme.wang/usr/images/sina/5cd95c815a83e.jpg" align="center" alt="image"> 

---
#### 新建项目

<img src="https://tryme.wang/usr/images/sina/5cd95c81ab557.jpg" align="center" alt="image">  

---
## 配置远程库关联

首先执行：

`$ git remote -v`

查看本地关联的远程库，如果没有任何关联，可以执行
`$ git remote add gitee git@gitee.com:trymesoft/myblog.git`
添加远程库。  

如果存在说明当前目录之前关联过Git远程库，可以在一个新的目录执行 
`$ git init`
初始化一个新的本地仓库，也可以执行
`$ git remove rm "XXX"`
删除，还可以再添加一个不重名的远程库。  

提交的时候指定push，例：
`$ git push gitee master`
这样一来，我们的本地库就可以同时与多个远程库互相同步：

<img src="https://tryme.wang/usr/images/sina/5cd95c82405ca.jpg" align="center" alt="image"> 

**参考：**[地址1](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00150154460073692d151e784de4d718c67ce836f72c7c4000) [地址2](https://www.zhihu.com/question/50212423)

---
**2018-05-06更新**

有的时候，我们按上述过程配置完成后，执行
`$ ssh -T v git@gitee.com`
时，会出现以下错误：

```
OpenSSH_7.4p1, LibreSSL 2.5.0

debug1: Reading configuration data /etc/ssh/ssh_config

debug1: Connecting to gitee.com [120.55.226.24] port 22.

debug1: Connection established.

debug1: key_load_public: No such file or directory

debug1: identity file /Users/wangyx/.ssh/id_rsa type -1

debug1: key_load_public: No such file or directory

debug1: identity file /Users/wangyx/.ssh/id_rsa-cert type -1

debug1: key_load_public: No such file or directory

debug1: identity file /Users/wangyx/.ssh/id_dsa type -1

debug1: key_load_public: No such file or directory

debug1: identity file /Users/wangyx/.ssh/id_dsa-cert type -1

debug1: key_load_public: No such file or directory

debug1: identity file /Users/wangyx/.ssh/id_ecdsa type -1

debug1: key_load_public: No such file or directory

debug1: identity file /Users/wangyx/.ssh/id_ecdsa-cert type -1

debug1: key_load_public: No such file or directory

debug1: identity file /Users/wangyx/.ssh/id_ed25519 type -1

debug1: key_load_public: No such file or directory

debug1: identity file /Users/wangyx/.ssh/id_ed25519-cert type -1

debug1: Enabling compatibility mode for protocol 2.0

debug1: Local version string SSH-2.0-OpenSSH_7.4

debug1: Remote protocol version 2.0, remote software version OSChina.NET

debug1: no match: OSChina.NET

debug1: Authenticating to gitee.com:22 as 'git'

debug1: SSH2_MSG_KEXINIT sent

debug1: SSH2_MSG_KEXINIT received

debug1: kex: algorithm: curve25519-sha256@libssh.org

debug1: kex: host key algorithm: ecdsa-sha2-nistp256

debug1: kex: server->client cipher: aes128-ctr MAC: hmac-sha2-256 compression: none

debug1: kex: client->server cipher: aes128-ctr MAC: hmac-sha2-256 compression: none

debug1: expecting SSH2_MSG_KEX_ECDH_REPLY

debug1: Server host key: ecdsa-sha2-nistp256 SHA256:FQGC9Kn/eye1W8icdBgrQp+KkGYoFgbVr17bmjey0Wc

debug1: Host 'gitee.com' is known and matches the ECDSA host key.

debug1: Found key in /Users/wangyx/.ssh/known_hosts:1

debug1: rekey after 4294967296 blocks

debug1: SSH2_MSG_NEWKEYS sent

debug1: expecting SSH2_MSG_NEWKEYS

debug1: SSH2_MSG_NEWKEYS received

debug1: rekey after 4294967296 blocks

debug1: SSH2_MSG_SERVICE_ACCEPT received

debug1: Authentications that can continue: publickey

debug1: Next authentication method: publickey

debug1: Trying private key: /Users/wangyx/.ssh/id_rsa

debug1: Trying private key: /Users/wangyx/.ssh/id_dsa

debug1: Trying private key: /Users/wangyx/.ssh/id_ecdsa

debug1: Trying private key: /Users/wangyx/.ssh/id_ed25519

debug1: No more authentication methods to try.

Permission denied (publickey).
```
**解决方法** 执行
`ssh-add 文件绝对路径`
即可。