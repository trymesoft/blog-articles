Posted on 2019-03-06
> 传输层安全性协议（英语：Transport Layer Security，缩写作 TLS），及其前身安全套接层（Secure Sockets Layer，缩写作 SSL）是一种安全协议，目的是为互联网通信提供安全及数据完整性保障。---维基百科

## HTTP 的安全问题

1. 使用明文进行通信，内容可能会被截取窃听；

2. 不验证通信双方的身份，通信双方的身份有可能遭遇伪装；
3. 无法证明报文的完整性，报文有可能遭篡改。

## HTTPS 概念

> 超文本传输安全协议（英语：Hypertext Transfer Protocol Secure，缩写：HTTPS，常称为 HTTP over TLS，HTTP over SSL 或 HTTP Secure）是一种通过计算机网络进行安全通信的传输协议。HTTPS 经由 HTTP 进行通信，但利用 SSL/TLS 来加密数据包。HTTPS 开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。---维基百科

通俗来说，HTTPS 并不是新协议，而是让 HTTP 先和 SSL/TLS通信，再由 SSL/TLS 和 TCP 通信，也就是说 HTTPS 使用了隧道进行通信。

<img src="https://tryme.wang/usr/images/sina/5cd95d6678253.jpg" alt="image" align="center"/>

## HTTPS 解决 HTTP 安全问题的方式

### 加密防止窃听

加密方式有两种，`对称密钥加密`和`非对称密钥加密`。

#### 对称密钥加密

对称密钥加密（Symmetric-Key Encryption），加密和解密使用同一密钥。

- 优点：运算速度快；

- 缺点：无法将密钥安全的传输给通信双方。

  <img src="https://tryme.wang/usr/images/sina/5cd95d68c3635.jpg" alt="image" align="center"/>

#### 非对称密钥加密

非对称密钥加密（Public-key cryptography），是密码学的一种算法，它需要`两个`密钥，一个是`公开`密钥，另一个是`私有`密钥；一个用作加密，另一个则用作解密。

公开密钥所有人都可以获得，通信发送方获得接收方的公开密钥之后，就可以使用公开密钥进行加密，接收方收到通信内容后使用私有密钥解密。

非对称密钥除了用来加密，还可以用来进`·签名`。因为私有密钥无法被其他人获取，因此通信`发送方`使用其`私有密钥`进行`签名`，通信`接收方`使用发送方的`公开密钥`对签名进行`解密`，就能判断这个签名是否正确。

- 优点：可以更安全地将公开密钥传输给通信发送方；

- 缺点：运算速度慢。

  <img src="https://tryme.wang/usr/images/sina/5cd95d6a56709.jpg" alt="image" align="center"/>

#### HTTPS 加密方式

HTTPS 采用混合的加密机制，使用`非对称密钥加密`用于传输`对称密钥`来保证传输过程的`安全性`，之后使用`对称密钥加密`进行通信来保证通信过程的`效率`。（下图中的 Session Key 就是对称密钥）

<img src="https://tryme.wang/usr/images/sina/5cd95d6b58773.jpg" alt="image" align="center"/>

### 证书进行认证

证书由数字证书认证机构（CA，Certificate Authority），客户端与服务器双方都可信赖的第三方机构颁发。

服务器的运营人员向 CA 提出公开密钥的申请，CA 在判明提出申请者的身份之后，会对已申请的公开密钥做数字签名（私钥加密），然后分配这个已签名的公开密钥，并将该`公开密钥`放入`公开密钥证书后绑定`在一起。

进行 HTTPS 通信时，服务器会把证书发送给客户端。客户端取得其中的公开密钥之后，先使用数字签名进行验证（公钥解密），如果验证通过，就可以开始通信了。

<img src="https://tryme.wang/usr/images/sina/5cd95d6cb5ed2.jpg" alt="image" align="center"/>

### SSL 提供报文摘要功能进行完整性保护

HTTP 也提供了 MD5 报文摘要功能，但不是安全的。例如报文内容被篡改之后，同时重新计算 MD5 的值，通信接收方是无法意识到发生了篡改。

HTTPS 的报文摘要功能之所以安全，是因为它结合了加密和认证这两个操作。加密之后的报文，遭到篡改之后，也很难重新计算报文摘要，因为无法轻易获取明文。

## HTTPS 的缺点

- 因为需要进行加密解密等过程，因此速度会更慢；
- 需要支付证书授权的高额费用（可以使用 Let's Encrypt Authority 免费申请）。

## 中间人攻击 MIMT

大致原理是在`公钥`的传输中，中间人截取到并替换为自己的公钥来进行双方通信，中间人用自己的公钥加解密并可随意篡改内容，HTTPS 中，通过 CA 来进行公钥的验证，通常浏览器内部也会内置合法流行的 CA 公钥，避免了此类情况。


参考：[CyC2018 大神](https://cyc2018.github.io/CS-Notes/#/notes/HTTP?id=%E5%85%AD%E3%80%81https)