---
title:  "阿里RDS数据库连接异常 table_definition_cache"
date:   2018-3-15 10:14:54
layout: post
comments: true
categories: MySQL
tags: MYSQL 阿里云 异常
---

* content
{:toc}


# 异常现象

公司使用了阿里云的 RDS 数据库，里面存放了多个项目的数据库。

今天在发布一个新的项目后，同时将对应的数据库放进去了，可是代码在运行时，出现了如下错误：

```php
ERROR 1615 (HY000): Prepared statement needs to be re-prepared
```





# 解决

查阅相关文档，问题应该出在数据库的表缓存数量限制上，查询当前打开表的数量如下：

```php
show global status like 'open%tables%';
```

变量 | 值
---|---
Open_tables	| 751
Opened_tables|77765

- `Open_tables`：当前正打开的表数量
- `Opened_tables`：打开表的历史累计数量，若数值非常大，说明cache太小，导致要频繁地open table


再查询表缓存数量配置：

```php
show variables like 'table%cache';
```

变量 | 值
---|---
table_definition_cache|500
table_open_cache|2000

- `table_definition_cache` ：存放表的定义信息，是frm文件在内存中的映射。MySQL需要打开frm文件，并将其内容初始化为Table Share 对象
- `table_open_cache`：缓存当前已经打开的表的文件句柄，与表创建时指定的存储引擎相关。


由此，可能的原因是 `table_definition_cache` 较小，可以通过增大对应值  来尝试（这里，我将值调为2048），如果是在本地数据库可以 `root` 用户直接执行：

```php
set global table_definition_cache=2048;
```

但是RDS数据库给到的账号并非超级管理员，执行时，报错：

```php
[Err] 1227 - Access denied; you need (at least one of) the SUPER privilege(s) for this operation
```

需要去到阿里后台进行参数设置，具体步骤如下：

- 进入：云数据库管理
- 点击 “管理” 进入指定示例的详情
- 点击侧边菜单：参数设置
- 找到 `table_definition_cache` 参数项，并修改参数值为：2048
- 点击按钮“提交参数”

设置教程：

https://help.aliyun.com/document_detail/26179.html?spm=5176.2020520165.120.d26179.5d257029OciRfx


