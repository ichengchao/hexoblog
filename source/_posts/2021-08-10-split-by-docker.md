---
title: 用docker切分网站模块
date: 2021-08-10 19:03:48
tags:

 - tech
 - docker
 - website

---

> 更新时间: 2025年01月22日18:15:02



# 主机nginx配置

安装完nginx后,使用certbot安装https证书

**init_nginx.sh**

```bash
#!/bin/bash

# 更新系统软件包列表并安装Nginx
echo "更新软件包列表并安装Nginx..."
sudo apt update && sudo apt install -y nginx

# 配置Nginx站点
SITE_CONFIG="/etc/nginx/sites-available/chengchao.name"
SITE_CONTENT="
server {
    root /var/www/html;

    index index.html index.htm index.nginx-debian.html;

    server_name chengchao.name www.chengchao.name;

    # 配置完https再打开
    #location / {
    #    proxy_pass http://[ossname].oss-cn-hangzhou.aliyuncs.com/;
    #}
    location /springrun/ {
        proxy_pass http://172.17.0.3:8080/springrun/;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
    location /myspringrun/ {
        proxy_pass http://127.0.0.1:8080/springrun/;
    }
}
"

echo "创建Nginx站点配置文件..."
echo "$SITE_CONTENT" | sudo tee $SITE_CONFIG > /dev/null

# 创建符号链接以启用站点
echo "启用Nginx站点..."
sudo ln -s $SITE_CONFIG /etc/nginx/sites-enabled/

# 测试Nginx配置
echo "测试Nginx配置..."
sudo nginx -t

# 重新加载Nginx以应用更改
echo "重新加载Nginx服务..."
sudo systemctl reload nginx

echo "Nginx配置完成！"
```





# Docker 配置

## Docker安装配置

```
# 使用snap安装,所有配置和启动都不太一样
sudo snap install docker

# 配置文件添加mirror
/var/snap/docker/current/config/daemon.json

# 启动
sudo snap start docker
sudo snap stop docker
sudo snap restart docker
```

**daemon.json**

```
{
    "log-level":        "error",
    "registry-mirrors": ["https://XXXXXXX.mirror.aliyuncs.com"]
}
```





## 安装mysql

mysqldocker下面需要三个文件:

**Dockerfile**

```dockerfile
FROM mysql:5.7.35

# 设置环境变量
ENV MYSQL_DATABASE=springrun

# 复制 MySQL 配置文件到容器
COPY my.cnf /etc/mysql/conf.d/

# 复制 SQL 文件到容器的 /docker-entrypoint-initdb.d 目录（MySQL 会自动执行）
COPY dump.sql /docker-entrypoint-initdb.d/

# 设定容器启动时的字符集（可选）
CMD ["mysqld", "--character-set-server=utf8mb4", "--collation-server=utf8mb4_unicode_ci"]
```

**my.cnf**

```
[mysqld]
character-set-server = utf8mb4

[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4
```

**dump.sql**

```
这个文件就是mysql的dump文件
mysqldump --default-character-set=utf8mb4 -uroot -h[host] --port=3306 -p[password] springrun>./dump.sql
```



### mysql启动

```
docker run --name [名称] -e MYSQL_ROOT_PASSWORD="[密码]" -d [镜像名称:版本]
```





## 部署应用

```shell
# 如果是在ARM的Mac构建的话需增加--platform=linux/amd64的参数
docker build --platform=linux/amd64 -t "charles/springrun:v2" ./

# 根据dockerfile build image
docker build -t "charles/springrun:v2" ./
# 这里需要加-it
docker run --name myspring -it -d charles/springrun:v2
git clone https://github.com/tldr-pages/tldr.git

```



**Dockerfile**

```dockerfile
FROM ubuntu:24.04
LABEL maintainer="charles <me@chengchao.name>"
# 备份原始源配置文件
RUN mv /etc/apt/sources.list.d/ubuntu.sources /etc/apt/sources.list.d/ubuntu.sources.bak

# 使用新的 DEB822 格式创建阿里云的源
RUN echo 'Types: deb\n\
URIs: http://mirrors.aliyun.com/ubuntu/\n\
Suites: noble noble-updates noble-security noble-backports\n\
Components: main restricted universe multiverse\n\
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg\n' > /etc/apt/sources.list.d/aliyun.sources

RUN apt-get update \
&& apt install -y curl \
&& apt install -y openssh-server \
&& apt install -y procps \
&& apt install -y net-tools \
&& apt install -y vim \
&& apt install -y git \
&& apt install -y openjdk-21-jdk \
&& apt install -y maven

RUN echo "alias ll='ls -lh'" >> /root/.bashrc
RUN wget https://ichengchao.oss-cn-hangzhou.aliyuncs.com/aliyunconfig/maven/settings.xml -O /etc/maven/settings.xml
RUN wget https://ichengchao.oss-cn-hangzhou.aliyuncs.com/aliyunconfig/ossutil -O /usr/local/bin/ossutil
RUN chmod +x /usr/local/bin/ossutil
WORKDIR /root
RUN [ -d .ssh ] || mkdir .ssh
RUN ssh-keygen -q -t rsa -N '' -f .ssh/id_rsa
RUN touch .ssh/authorized_keys && chmod 664 .ssh/authorized_keys
RUN echo "alias oss='ossutil'"
RUN echo "\
service ssh start\n\
tail\n\
" >/root/init.sh
EXPOSE 8080
CMD ["/bin/bash","/root/init.sh"]
```

