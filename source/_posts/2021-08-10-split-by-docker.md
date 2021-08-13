---
title: 用docker切分网站模块
date: 2021-08-10 19:03:48
tags:

 - tech
 - docker
 - website

---

### 安装mysql

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

### Dockerfile

```dockerfile
FROM ubuntu:20.04
LABEL maintainer="charles <me@chengchao.name>"
RUN echo "\
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse\n\
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse\n\
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse\n\
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse\n\
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse\n\
" >/etc/apt/sources.list
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt-get update \
&& apt-get install -y curl \
&& apt-get install -y openssh-server \
&& apt-get install -y procps \
&& apt-get install -y net-tools \
&& apt-get install -y vim \
&& apt-get install -y git \
&& apt-get install -y openjdk-11-jdk \
&& apt-get install -y maven
RUN echo "alias ll='ls -lh'" >> /root/.bashrc
RUN wget https://ichengchao.oss-cn-hangzhou.aliyuncs.com/aliyunconfig/maven/settings.xml -O /etc/maven/settings.xml
RUN wget https://ichengchao.oss-cn-hangzhou.aliyuncs.com/aliyunconfig/ossutil -O /usr/local/bin/ossutil
RUN chmod +x /usr/local/bin/ossutil
WORKDIR /root
RUN mkdir .ssh && ssh-keygen -q -t rsa -N '' -f .ssh/id_rsa
RUN touch .ssh/authorized_keys && chmod 664 .ssh/authorized_keys
RUN echo "alias oss='ossutil'"
RUN echo "\
service ssh start\n\
tail\n\
" >/root/init.sh
EXPOSE 8080
CMD ["/bin/bash","/root/init.sh"]
```

