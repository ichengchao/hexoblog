---
title: VPN中继
date: 2019-03-23 10:04:43
tags:

 - tech
 - vpn
 
---


> OS: Ubuntu 16.04.4 LTS


最近直连VPN经常出问题,所有想用aliyun的ECS去中继一下.网上看了一下,方案是现成的:利用`Haproxy`做代理.
数据流向示意: `client -> aliyun -> vpn`

#### 安装

```
sudo apt update
sudo apt install -y haproxy
```

#### 配置
配置文件在`/etc/haproxy/haproxy.cfg`

```
global
        ulimit-n  51200

defaults
        log global
        mode    tcp
        option  dontlognull
        contimeout 1000
        clitimeout 150000
        srvtimeout 150000

frontend ss-in-node1
        bind *:[port]
        default_backend ss-out-node1

backend ss-out-node1
        server server1 [ip]:[port] maxconn 20480

frontend ss-in-node2
        bind *:[port]
        default_backend ss-out-node2

backend ss-out-node2
        server server1 [ip]:[port] maxconn 20480
```

把配置文件中的`bind *:[port]`替换成你想暴露的端口,`[ip]:[port]`替换成你的vpn提供的ip和端口,重启一下服务

```
sudo service haproxy restart
```


#### 客户端

- ios: Potatso Lite (美区)
- mac: ShadowsocksX-NG (github)

配置的时候只要把ip和port替换成aliyun的就可以了.