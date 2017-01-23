---
layout:     post
title:      "关于 MySQL 的默认字符集"
subtitle:   ""
date:       2017-01-18 12:00:00
author:     "GeniusV"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - MySQL
---

MySQL 默认字符集为 `latin1 `，可以通过运行命令：

``` sql
SHOW  variables like '%char%';
```
查看。

运行以下命令来修改默认字符集：

``` sql
SET character_set_client = utf8 ;
SET character_set_connection = utf8 ;
SET character_set_database = utf8 ;
SET character_set_results = utf8 ;
SET character_set_server = utf8 ;
SET collation_connection = utf8 ;
SET collation_database = utf8 ;
SET collation_server = utf8 ;
```

如果错误依然存在，运行命令：

``` sql
ALTER TABLE <TableName>..
  CONVERT TO CHARACTER SET utf8;
```

批量修改表字符集为utf8.

另，创建数据库和表时最好指定字符集，防止抽风。


