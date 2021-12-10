Posted on 2018-09-20
> 在 MySQL 数据库中，我们常用的 utf-8 格式编码最多支持 3 个字节，而 emoji 表情是 4 个字节的符号，所以  emoji 表情的长度就超出了我们常用 utf-8 的字符集范围而无法存储。从 MySQL5.5 开始，可以更改数据库和数据表的编码为 utf8mb4 使其支持存储 emoji，utf8mb4 兼容 utf8，且比 utf8 能表示更多的字符，是 utf8 字符集的超集。
修改之前一定要备份数据！修改之前一定要备份数据！修改之前一定要备份数据！

## 1. 更改数据库的排序规则
![order-rule](https://tryme.wang/usr/images/sina/5cd95ca8c3bf5.jpg)

## 2. 点击 SQL 按钮，执行以下 SQL 语句(将 Typecho 修改为自己对应的前缀)
```sql
alter table typecho_comments convert to character set utf8mb4 collate utf8mb4_unicode_ci;
alter table typecho_contents convert to character set utf8mb4 collate utf8mb4_unicode_ci;
alter table typecho_fields convert to character set utf8mb4 collate utf8mb4_unicode_ci;
alter table typecho_metas convert to character set utf8mb4 collate utf8mb4_unicode_ci;
alter table typecho_options convert to character set utf8mb4 collate utf8mb4_unicode_ci;
alter table typecho_relationships convert to character set utf8mb4 collate utf8mb4_unicode_ci;
alter table typecho_users convert to character set utf8mb4 collate utf8mb4_unicode_ci;
```
修改完之后：
![after-modified](https://tryme.wang/usr/images/sina/5cd95ca978416.jpg)

## 3. 编辑网站根目录的 config.inc.php 文件，文件末尾
```php
/** 定义数据库参数 */
$db = new Typecho_Db('Pdo_Mysql', 'typecho_');
$db->addServer(array (
  'host' => 'localhost',
  'user' => 'username',
  'password' => 'password',
  'charset' => 'utf8mb4',// 将utf-8修改为utf8mb4
  'port' => '3306',
  'database' => 'database-name',
), Typecho_Db::READ | Typecho_Db::WRITE);
Typecho_Db::set($db);
```

按照以上步骤修改完之后，文章即可支持emoji表情。

[参考地址](https://www.moidea.info/archives/how-to-set-up-typecho-blog-to-support-emoji-expression.html)