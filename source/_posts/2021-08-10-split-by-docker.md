---
title: 用docker切分网站模块
date: 2021-08-10 19:03:48
tags:

 - tech
 - docker
 - website

---

###安装mysql

docker版的[mysql](https://hub.docker.com/_/mysql),使用的版本是`5.7.35`

```shell
#拉取docker镜像,并启动
docker pull mysql:5.7.35
docker run --name [名称] -e MYSQL_ROOT_PASSWORD=[密码] -d mysql:5.7.35

#修改默认编码为utf8mb4
vi /etc/mysql/conf.d/mysql.cnf

[mysqld]
character-set-server = utf8mb4

[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4


#命令行显示中文
show variables like 'char%';
set character_set_results=utf8mb4;

#创建utf8mb4的database
create database [库名称] default character set utf8mb4 collate utf8mb4_unicode_ci;

#导出原有库的数据,指定编码
mysqldump --default-character-set=utf8mb4 -u[用户名] -p [库名称] > dump.sql
mysql> use [库名称]
mysql> source /path/dump.sql
```



### 部署应用

```shell
# 根据dockerfile build image
docker build -t "charles/springrun:v2" ./
# 这里需要加-it
docker run --name myspring -it -d charles/springrun:v2
git clone https://github.com/tldr-pages/tldr.git

```

