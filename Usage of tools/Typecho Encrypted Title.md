Posted on 2018-12-08
> Typecho 的文章可以加密，但是加密的文章标题默认显示的是“此内容被密码保护”，什么文章都不知道?，不过还好可以更改配置。

### 解决方案

在 $SITE_HOME/var/Widget/Abstract/Contents.php 文件的如图位置： <img src="https://tryme.wang/usr/images/sina/5cd95cdca9006.jpg" align="center" alt="image">

将 <code>$value['title'] = _t('此内容被密码保护')</code> 注释掉即可，当然，也可以修改上面的 <code>请输入密码以访问</code> 和 <code>提交</code> 为自定义内容。


