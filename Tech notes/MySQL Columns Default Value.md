Posted on 2018-12-15
### 前提

使用 SQL 脚本创建表的时候，遇到了一个问题，SQL 脚本如下：

```mysql
create table test
(
   ...
   create_time          datetime not null default now() comment '创建时间',
   ...
)
```

在为”create_time“字段设置默认值为当前时间时，SQL 执行报了一个错误如下：

```
[42000][1067] Invalid default value for 'create_time'
```

字段默认值无效的原因，查看了下当前的 MySQL 版本：

```
mysql> select version();
+------------+
| version()  |
+------------+
| 5.5.35-log |
+------------+
1 row in set (0.00 sec)
```

版本是 5.5 的。



### 解决思路

查看了 [MySQL 5.5](https://dev.mysql.com/doc/refman/5.5/en/data-type-defaults.html) 的文档，在 Data Type Default Values 这一段中有这么个描述：

```wiki
With one exception, the default value specified in a DEFAULT clause must be a literal constant; it cannot be a function or an expression. This means, for example, that you cannot set the default for a date column to be the value of a function such as NOW() or CURRENT_DATE. The exception is that, for a TIMESTAMP column, you can specify CURRENT_TIMESTAMP as the default. See Section 11.3.5, “Automatic Initialization and Updating for TIMESTAMP”. 
（除了一个例外，DEFAULT子句中指定的默认值必须是文字常量;它不能是一个功能或表达。这意味着，例如，您不能将日期列的默认值设置为函数的值，例如NOW（）或CURRENT_DATE。例外情况是，对于TIMESTAMP列，您可以将CURRENT_TIMESTAMP指定为默认值。请参见第11.3.5节“TIMESTAMP的自动初始化和更新”。）
```

原因很明显了，DEFAULT 指定的默认值不能是 NOW() ，可以将 ”create_time“字段的类型改为 TIMESTAMP，再将默认值设为 CURRENT_TIMESTAMP 来使用。

**PS：**MySQL 5.6/5.7 版本中，对于默认值的描述有所变化：

```wiki
With one exception, the default value specified in a DEFAULT clause must be a literal constant; it cannot be a function or an expression. This means, for example, that you cannot set the default for a date column to be the value of a function such as NOW() or CURRENT_DATE. The exception is that, for TIMESTAMP and DATETIME columns, you can specify CURRENT_TIMESTAMP as the default. See Section 11.3.5, “Automatic Initialization and Updating for TIMESTAMP and DATETIME”. 
（除了一个例外，DEFAULT子句中指定的默认值必须是文字常量;它不能是一个功能或表达。这意味着，例如，您不能将日期列的默认值设置为函数的值，例如NOW（）或CURRENT_DATE。例外情况是，对于TIMESTAMP和DATETIME列，您可以将CURRENT_TIMESTAMP指定为默认值。请参见第11.3.5节“TIMESTAMP和DATETIME的自动初始化和更新”。）
```

可以看到，后续版本中 DATETIME 类型的字段也可以将 CURRENT_TIMESTAMP 设为默认值。

