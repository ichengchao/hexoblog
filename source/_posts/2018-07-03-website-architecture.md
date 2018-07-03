---
title: 个人网站架构升级
date: 2018-07-03 12:06:48
tags:

 - tech

---

![](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/image/website.png)

### 目的

这次改造的最主要的目的就是为了**方便迁移**.因为阿里云的售卖策略是老客户续费很难拿到优惠,所以有时候重新买一台的成本会比续费的成本便宜很多.


### 实现

上面的图基本上把原理都说清楚了.我的用到的组件罗列一下:

- Aliyun OSS
- Aliyun ECS
- Aliyun Code
- Aliyun Docker Hub
- Github
- Docker
- Nginx
- Tomcat
- Mysql


原来就是一坨东西全部堆在ECS中,想要迁移的话非常的麻烦.所以最核心的思路就是按模块拆分,然后让每个模块都容易迁移.

##### 静态首页

![](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/image/oss_site_config.png)

之前是代码托管在github中,然后直接放在nginx的静态目录中,更新的时候git pull一下就可以.现在是直接把所有内容直接放在oss中,只要简单设置一下就行了.好处是彻底不用考虑迁移的事情,坏处是没有了版本管理.


##### Blog

这个很早的时候就已经跑在Docker中了.这次更新的了一下Dockerfile.还有就是我做了一个好玩的功能:**自动部署**.听起来很高大上,但实现起来非常的简单,就是在Github中加了一个webhook,当有新的commit的时候向我的java后台发送一个请求,然后java后台收到请求后检查一下commit的消息中是否包含"@deploy"的关键字,如果包含就调用发布blog的脚本.非常简单,但用起来真是爽.

```docker
FROM nginx:1.15.0
LABEL maintainer="charles <me@chengchao.name>"
# RUN echo "\
#deb http://mirrors.aliyun.com/debian/ stretch main non-free contrib\n\
#deb http://mirrors.aliyun.com/debian-security stretch/updates main\n\
#deb http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib\n\
#deb http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib\n\
# " >/etc/apt/sources.list
RUN apt-get update \
&& apt-get install -y curl \
&& apt-get install -y openssh-server \
&& apt-get install -y procps \
&& apt-get install -y net-tools \
&& apt-get install -y vim \
&& apt-get install -y gnupg2 \
&& apt-get install -y git \
&& curl -sL https://deb.nodesource.com/setup_10.x | /bin/bash - \
&& apt-get install -y nodejs
RUN npm install hexo-cli -g
RUN echo "alias ll='ls -lh'" >> /root/.bashrc
WORKDIR /root
RUN mkdir .ssh && ssh-keygen -q -t rsa -N '' -f .ssh/id_rsa
RUN touch .ssh/authorized_keys && chmod 664 .ssh/authorized_keys
RUN git clone https://github.com/ichengchao/hexoblog.git
RUN hexo init test \
&& cp -r /root/test/node_modules /root/hexoblog/ \
&& rm -rf /root/test
RUN echo "\
cd ~/hexoblog\n\
git pull\n\
hexo clean\n\
hexo generate\n\
cd /usr/share/nginx/html\n\
rm -rf *\n\
cp -r /root/hexoblog/public/* ./\n\
" >/root/deploy.sh
RUN chmod +x /root/deploy.sh
RUN echo "\
service ssh start\n\
service nginx start\n\
node\n\
" >/root/init.sh
EXPOSE 22 80
CMD ["/bin/bash","/root/init.sh"]

```

##### Java后台

这次主要改的就是这个模块,直接把mysql(省钱没上RDS),tomcat装进一个docker.原来机器上起的git server迁移到Aliyun的code服务中.还有个要做的就是定时的把mysql dump出来上传到oss做备份.



做完这些后,迁移就变得so easy了.

