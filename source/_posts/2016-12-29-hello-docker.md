---
title: Hello Docker
date: 2016-12-29 21:44:18
tags:
  - tech
  - docker

---


![](data:image/png;base64,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)

### 什么是Docker
>What is Docker?
Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and ship it all out as one package. By doing so, thanks to the container, the developer can rest assured that the application will run on any other Linux machine regardless of any customized settings that machine might have that could differ from the machine used for writing and testing the code.

上面是[opensource.com](https://opensource.com/resources/what-docker)对docker的解释.总结一下:docker是一个完整的应用运行容器,而且用起来很方便.


### 安装
以mac os为例,直接到[docker官网](https://www.docker.com)下载安装包即可.对于初学者我不太推荐Kitematic(一个管理docker的可视化工具).就好比学习git,再强大的可视化工具也干不过原生的命令行.所以最好就是直接使用docker提供的原生命令行工具.  
安装完成后,最重要的事情就是设置一下镜像,否则速度真是没法忍受.我自己使用的是[阿里云的docker加速器](https://dev.aliyun.com).注册完成后,会有一个属于个人的加速器地址`https://xxx.mirror.aliyuncs.com`,把这个地址加入到docker的mirror列表中就行了.

### 试试看
我们先装个nginx试试

```
# 安装nginx
docker pull nginx

# 查看已经安装的image
docker images

# 启动,名称是hello-nginx,将80端口映射到宿主机的8080
docker run --name hello-nginx -d -p 8000:80 nginx

# 进入命令行, 02697b7bf225为container id,可以用docker ps查看
docker exec -it 02697b7bf225 bash


# 如果是ubuntu的image
docker run --name myubuntu -it -d ubuntu

# 和宿主机的文件拷贝
docker cp  <CONTAINER_ID>:SRC_PATH DEST_PATH
docker cp  SRC_PATH <CONTAINER_ID>:DEST_PATH

# 挂载宿主机的目录到容器中
docker run -itd --name myubuntu -v /Users/charles/Desktop/mydata:/mydata  ubuntu
```

启动后就能在宿主机的浏览器中直接访问`http://localhost:8000`就能看到nginx的欢迎页了.接着进入容器的命令行之后就跟普通的linux没什么区别了,可以看到系统是debian的8.0版本.

```bash
cat /etc/issue

Debian GNU/Linux 8 \n \l
```

接着我们在容器里装个vim试试,把apt的源换成阿里云的

```bash
# 更换成aliyun的源
echo -e "\
deb http://mirrors.aliyun.com/debian/ jessie main non-free contrib\n\
deb http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib\n\
deb-src http://mirrors.aliyun.com/debian/ jessie main non-free contrib\n\
deb-src http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib\
" >/etc/apt/sources.list

# 当然也可以直接从宿主机cp过来
docker cp sources.list [containerID]:/etc/apt/sources.list

# 安装vim
apt-get update
apt-get install vim

```

是不是很爽,一切命令都不用sudo,直接root执行,而且不怕搞出问题,因为随时都可以再run一个起来.

### Dockerfile
Dockerfile用来描述一个image,写过java的人可以把Dockerfile类比成maven工程的pom.xml文件.我们写个最简单的Dockerfile试试

```
FROM nginx
MAINTAINER charles
RUN echo "\
deb http://mirrors.aliyun.com/debian/ jessie main non-free contrib\n\
deb http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib\n\
deb-src http://mirrors.aliyun.com/debian/ jessie main non-free contrib\n\
deb-src http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib\
" >/etc/apt/sources.list
RUN apt-get update
RUN apt-get install -y vim
RUN echo "alias ll='ls -lh'" >> /root/.bashrc
```

写法很简单,基本上跟上面的例子差不多,有点小区别就是在Dockerfile里面的echo不需要加-e了.接着用docker build编译一个image出来

```
docker build -t "charles/mynginx:v1" ./
```

接着就能用docker images查看到刚刚打出来的镜像了.还能把刚才打出来的镜像直接push到阿里云的私人仓库中.基本的使用差不多就这样了,更多的玩法就你自己去探索吧!  
ps: 我现在的电脑里是不装mysql,mongo这些东西了,直接用docker代替了,挺好.还有你看到这个blog也是运行在docker中的,😄

