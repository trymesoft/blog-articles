Posted on 2018-05-05
> 网上下载很多文件，都会提供 MD5 或者 SHA256 校验，防止文件内容被篡改，文件下载到本地后，
我们可以对文件进行校验。

---
## Windows 环境下

在 Windows 操作系统环境下，计算这几个常用的校验值需要下载一款软件

[MD5 & SHA Checksum Utility](https://raylin.wordpress.com/downloads/md5-sha-1-checksum-utility/),下载后即可使用。

---
## Linux 环境下
Linux 下有 md5sum 命令，sha1sum 命令和 sha256sum 命令来计算文件相对应的值，具体使用如下：
```
[root@iz2ze0ibk1pvak9ckuwb5yz /]# md5sum test.txt 
2f6c38be914b756fde482fff83064d37 test.txt

[root@iz2ze0ibk1pvak9ckuwb5yz /]# sha1sum test.txt 
228dfdb10e9ad6fdf5ca97f402355df1952112fe test.txt

[root@iz2ze0ibk1pvak9ckuwb5yz /]# sha256sum test.txt 
d8a0785f3ce124ee8c79c172eefdc6989141aafaf9deab2dac8437ed5e60f5c4 test.txt
```

---
## macOS环境下
mac 下有 md5 命令和 shasum 命令，具体使用如下：
```
wangMBP:~ wangyx$ md5 test.txt
MD5 (test.txt) = af7bbf6c8b822e9e89ccc2b8552ca294

wangMBP:~ wangyx$ shasum -a 1 test.txt
1367a1ab1da100436c5ea6b0dc0d737ada6aede4 test.txt

wangMBP:~ wangyx$ shasum -a 256 test.txt
5913f04a2a92cbd0346a4a7cf7856f8174e12657ca267764b57da90b27c4c51f test.txt
```

**我们可以使用计算出的校验值和网站提供的校验值对比，可以看出文件是否被篡改。**