---
title: linux初始设置
date: 2017-11-01 07:40:34
tags:
  - linux
  - tech
  
---

> 以下操作都是基于ubuntu 16.04,别的linux发行版可能会有不同


### 修改主机名

临时修改直接用hostname newName就可以了

- 修改/etc/hostname,重启后生效
- 修改/etc/hosts的映射关系

### 创建新用户

```bash
# 用这个,这个用把用户组,设置密码等一步都完成
adduser admin

# 不要用这个
useradd admin
```

### 安装oh-my-zsh
[官方文档](https://github.com/robbyrussell/oh-my-zsh)
安装之前先要把zsh,git安装一下

```
sudo apt-get install zsh
sudo apt-get install git

修改默认主题,把全路径显示出来,这纯粹是个人习惯
/home/admin/.oh-my-zsh/themes/robbyrussell.zsh-theme

PROMPT='${ret_status} %{$fg[green]%}$USER@hz✭ %d%{$reset_color%} $(git_prompt_info)'
#PROMPT='${ret_status} %{$fg[cyan]%}%c%{$reset_color%} $(git_prompt_info)'

如果要区别不用的主机,也可以个性化一下,比如调个颜色[cyan] -> [yellow],或者加上用户名

```

具体颜色的设置参考[这里](https://gabri.me/blog/custom-colors-in-your-zsh-prompt/)

### 修改apt源
推荐用[阿里云的源: http://mirrors.aliyun.com/](http://mirrors.aliyun.com/)
如果购买的是阿里云ECS,还是把http://mirrors.aliyun.com/换成内网地址http://mirrors.aliyuncs.com/

修改完后,用apt-get update更新一下

### 修改ssh端口
配置文件:/etc/ssh/sshd_config
最好不要密码登录,而是使用密钥




