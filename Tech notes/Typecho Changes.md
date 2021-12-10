Posted on 2019-02-28
> 记录 Typecho 或其主题有关改动。

### 评论显示 User Agent

使用插件实现，插件[地址](https://github.com/ennnnny/typecho)。

1. 下载后将其上传到网站根目录下的 `usr/plugins` 文件夹并解压，内含两个插件。
2. 这时可以登录网站管理后台配置并启用：<img src="https://tryme.wang/usr/images/sina/5cd95d52bcf52.jpg" alt="image" align="center">

3. 修改评论页面代码，配置 UA 显示位置，`Handsome` 主题修改主题根目录下的 `component/comments.php`，大致位置如下：

   <img src="https://tryme.wang/usr/images/sina/5cd95d54283c1.jpg" alt="image" align="center">

4. 查看评论效果：

   <img src="https://tryme.wang/usr/images/sina/5cd95d6593cb8.jpg" alt="image" align="center">