---
title: hello,etcd
date: 2019-12-27 12:29:41
tags:

 - cncf
 - tech
 - k8s
---

### etcd 是什么?

>etcd is a distributed reliable key-value store for the most critical data of a distributed system, with a focus on being:
>
>- Simple*: well-defined, user-facing API (gRPC)*
>- Secure: automatic TLS with optional client cert authentication
>- Fast*: benchmarked 10,000 writes/sec*
>- *Reliable*: properly distributed using Raft

### 搭建

搭建一个测试环境测试一下,当然最方便的就是用docker.

```bash
docker pull busybox
# --rm 参数指的是在stop后会自动删除容器
docker run -it --rm busybox

# 从官网下载最新版本并且解压,我下载的是etcd-v3.3.18-linux-amd64.tar.gz

# 在root目录创建data目录
mkdir -p /root/data

nohup ./etcd --config-file /root/config.yml &
```

```yaml
# 把ip地址替换成对应docker的ip,集群模式
# docker1: 172.17.0.2
data-dir: /root/data
listen-client-urls: http://172.17.0.2:2379,http://localhost:2379
advertise-client-urls: http://172.17.0.2:2379,http://localhost:2379
name: 'etcd1'

listen-peer-urls: http://172.17.0.2:2380
initial-advertise-peer-urls: http://172.17.0.2:2380
initial-cluster: 'etcd1=http://172.17.0.2:2380,etcd2=http://172.17.0.3:2380'
initial-cluster-token: 'etcd-cluster'
initial-cluster-state: 'new'


# docker1: 172.17.0.3
data-dir: /root/data
listen-client-urls: http://172.17.0.3:2379,http://localhost:2379
advertise-client-urls: http://172.17.0.3:2379,http://localhost:2379
name: 'etcd2'

listen-peer-urls: http://172.17.0.3:2380
initial-advertise-peer-urls: http://172.17.0.3:2380
initial-cluster: 'etcd1=http://172.17.0.2:2380,etcd2=http://172.17.0.3:2380'
initial-cluster-token: 'etcd-cluster'
initial-cluster-state: 'new'
```



### 测试

```bash
# 可以当做一个文件系统使用
# 创建一个文件夹
./etcdctl mkdir testdir
# 创建一个文件,而且内容是"testvalue"
./etcdctl set testdir/testkey testvalue
# 查看testdir下面的文件列表
./etcdctl ls testdir/
# 查看testkey的文件内容
./etcdctl get testdir/testkey
```

