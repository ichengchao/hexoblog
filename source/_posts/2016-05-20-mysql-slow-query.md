---
title: 慢SQL问题排查
date: 2016-05-20 21:03:53
tags:
	- tech
	- mysql
	
---

今天碰到一个慢sql问题.记录一下.  
有两张表ta和tb,简化后的表结构如下

```sql
# ta 有14605条数据
CREATE TABLE `ta` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

# tb 有2790条数据
CREATE TABLE `tb` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

慢sql如下:

```sql
select count(*) from ta left join tb on ta.name = tb.name;

+----------+
| count(*) |
+----------+
|    14605 |
+----------+
1 row in set (2.96 sec)
```

刚开始以为是没有建索引.检查了一下索引是没有问题的.于是就纳闷了.后来看了很久才发现原来是ta表中的name的类型弄错了.ta表中的name的类型是int,而t2表中name的类型是varchar.类型不一样就用不上索引.把ta表中的类型修改回varchar后就马上正常了.

```sql
select count(*) from ta left join tb on ta.name = tb.name;

+----------+
| count(*) |
+----------+
|    14605 |
+----------+
1 row in set (0.02 sec)
```

