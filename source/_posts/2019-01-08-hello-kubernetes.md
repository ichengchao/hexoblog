---
title: kubernetes集群搭建
date: 2019-01-08 18:15:31
tags:

 - tech
 - kubernetes
 
---



### 环境

操作系统: Ubuntu 18.04  
机器: 三台Aliyun ECS

- Master节点: 4c8g  
- work节点1: 2c4g
- work节点2: 2c4g  

### 关闭swap(每台机器都需要)

```
swapoff -a
```

###  安装docker(每台机器都需要)

安装最新版的docker-ce,把源切换成[Aliyun的源](https://opsx.alibaba.com/)


```
# 安装系统依赖
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common

# 添加aliyun的docker源,root执行
add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add -

# 安装docker
sudo apt-get update
sudo apt-get -y install docker-ce

# 启动docker
systemctl enable docker
systemctl start docker

# 在admin的用户下提示没有权限的问题,把当前用户加到docker用户组里就可以了
sudo groupadd docker
sudo usermod -aG docker $USER
```

安装完成后可以用`docker ps`测试一下

### 安装kubelet kubeadm kubectl (root执行,每台机器都需要)
>[阿里云kubernetes帮助](https://opsx.alibaba.com/mirror?lang=zh-CN)

```
apt-get update 
apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

### 关键一步 (每台机器都需要)

> 这里为了方便就不区分master和work节点了.一把都把这个docker image下载下来

本来只要完全按照[官方文档](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)就可以了,由于对应的google的源被墙了.所以需要先手动下载 先使用`kubeadm config images list` 看看需要下载什么镜像,网络用flannel也需要提前把docker镜像下载好,已经把flannel的镜像上传到我个人的阿里云仓库.脚本:


```
#!/bin/bash
images=(
    kube-apiserver:v1.13.1
    kube-controller-manager:v1.13.1
    kube-scheduler:v1.13.1
    kube-proxy:v1.13.1
    pause:3.1
    etcd:3.2.24
    coredns:1.2.6
)

for imageName in ${images[@]} ; do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done


docker pull registry.cn-hangzhou.aliyuncs.com/ichengchao/flannel:v0.10.0-amd64
docker tag f0fad859c909 quay.io/coreos/flannel:v0.10.0-amd64
docker rmi registry.cn-hangzhou.aliyuncs.com/ichengchao/flannel:v0.10.0-amd64

```


### 启动Master

```
kubeadm init --pod-network-cidr=10.244.0.0/16
```

启动完成后按照提示,在指定用户中执行(admin中执行)

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

启动flannel

```
# 根据官网的提示
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

启动后最下面会出现类似这样的语句,能在别的node上执行下面的语句加到集群中

```
kubeadm join 172.16.71.232:6443 --token 111q9t.111110uf711111 --discovery-token-ca-cert-hash sha256:11111111cccf12636af67cee2d3902d847b6600cb33b6114abbac41111111

```





### 排查命令

> 安装过程中出现的问题一般就是docker镜像下载失败,或者网络插件没有安装


```
# 很好用
journalctl -xe

# 启停服务,查看服务状态
systemctl status docker.service
systemctl start docker.service
systemctl stop docker.service

# 查看具体pod的详细信息
kubectl get pods --all-namespaces
kubectl describe pod kube-flannel-ds-amd64-7dvwb --namespace=kube-system

```

### 部署测试应用

[部署hello world](https://github.com/kubernetes/examples/blob/master/staging/simple-nginx.md)

```
kubectl expose deployment my-nginx --port=80 --type=LoadBalancer 改成NodePort
```


测试自己写的测试应用,我写了一个非常简单的springboot的应用,并且已经打成docker镜像并且上传到了阿里云的docker仓库中

```
# 启动
kubectl create deployment --image registry.cn-hangzhou.aliyuncs.com/ichengchao/hellojava:0.6 my-test

# 扩容
kubectl scale deployment --replicas 4 my-test

# 暴露服务
kubectl expose deployment my-test --port=8080 --type=NodePort
kubectl get services 查看映射端口

# 下线
kubectl delete deployment my-test
```


这样最基本的环境搭建和应用部署就完成了.