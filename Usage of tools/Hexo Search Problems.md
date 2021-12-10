Posted on 2019-01-17
> 整理 Hexo 博客时，发现页面搜索功能不好用，之前一直是没有问题的，点击“搜索”后，页面中间总是显示加载，如图：
>
> <img src="https://tryme.wang/usr/images/sina/5cd95d49dc9ba.jpg" alt="image" align="center">

### 解决
首先尝试重装插件，搜索插件如下：

```json
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.6.0"
  },
  "dependencies": {
    ...
    "hexo-generator-search": "^2.2.1",
    "hexo-generator-searchdb": "^1.0.8",
    ...
  }
}
```

将搜索的插件重装之后貌似还是不行，失败！
之后怀疑是配置的问题，查了下插件[开发者](https://github.com/wzpan/hexo-generator-search)使用方法：

<img src="https://tryme.wang/usr/images/sina/5cd95d4a7f99d.jpg" alt="image" align="center">

对照自己的配置，也无问题，失败！

后来，联想到搜索将内容输出到了根目录的 `search.xml` 文件中，直接本地调试访问 http://localhost:4000/search.xml，发现了报错：

<img src="https://tryme.wang/usr/images/sina/5cd95d4ce3240.jpg" alt="image" align="center">

果然有问题，去查看 `157行 335列`：

<img src="https://tryme.wang/usr/images/sina/5cd95d4e1b3c6.jpg" alt="image" align="center">

卧槽，多了个红点，估计是复制的时候多了什么奇怪的东西，将原文复制到 `Sublime Text` 中查看，发现果然是有东西：

<img src="https://tryme.wang/usr/images/sina/5cd95d51d983b.jpg" alt="image" align="center">

不知道哪里来的?，删除后重新调试，一切正常?！

参考：[IT范儿](https://www.itfanr.cc/2017/11/24/resolve-hexo-blog-search-exception/)

