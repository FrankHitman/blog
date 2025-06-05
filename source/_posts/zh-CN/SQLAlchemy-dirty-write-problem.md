---
title: SQLAlchemy dirty write problem
date: 2025-06-05 20:56:38
categories:
    - Python
tags:
    - SQLAlchemy
    - Mysql
    - Isolation level
---

在sqlalchemy中使用 select_for_update 和 transaction 避免脏写，然而并没有效果。
打开 echo=True 看日志如sql.log显示。
[source/_posts/zh-CN/SQLAlchemy-dirty-write-problem/sql.log](/zh-CN/SQLAlchemy-dirty-write-problem/sql.log)
begin_nested() 产生的 sa_savepoint_1 会交叉
![source/_posts/zh-CN/SQLAlchemy-dirty-write-problem/95921740391336_.pic.jpg](/zh-CN/SQLAlchemy-dirty-write-problem/95921740391336_.pic.jpg)

<!--more-->

但是在mysql的命令行里面，事务加锁会互斥的
![source/_posts/zh-CN/SQLAlchemy-dirty-write-problem/95941740391452_.pic.jpg](/zh-CN/SQLAlchemy-dirty-write-problem/95941740391452_.pic.jpg)


阅读官方文档
https://docs.sqlalchemy.org/en/20/core/connections.html#dbapi-autocommit
https://docs.sqlalchemy.org/en/20/dialects/mysql.html#mysql-isolation-level

应该是事物隔离级别默认被设置为 “auto commit” 了，从而导致 transaction 并不生效。
![source/_posts/zh-CN/SQLAlchemy-dirty-write-problem/97181740464307_.pic.jpg](/zh-CN/SQLAlchemy-dirty-write-problem/97181740464307_.pic.jpg)

说来也奇怪，SQLAlchemy可以设置的隔离级别比MySQL多出一个“auto commit”
![source/_posts/zh-CN/SQLAlchemy-dirty-write-problem/97191740465384_.pic.jpg](/zh-CN/SQLAlchemy-dirty-write-problem/97191740465384_.pic.jpg)
￼

显式的声明隔离级别
![source/_posts/zh-CN/SQLAlchemy-dirty-write-problem/97281740467784_.pic.jpg](/zh-CN/SQLAlchemy-dirty-write-problem/97281740467784_.pic.jpg)

可以看到修改配置后的 save point 会依次执行，而不会交叉
[source/_posts/zh-CN/SQLAlchemy-dirty-write-problem/sql2.log](/zh-CN/SQLAlchemy-dirty-write-problem/sql2.log)
![source/_posts/zh-CN/SQLAlchemy-dirty-write-problem/97211740467661_.pic.jpg](/zh-CN/SQLAlchemy-dirty-write-problem/97211740467661_.pic.jpg)

