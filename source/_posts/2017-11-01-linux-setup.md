---
title: linux初始设置
date: 2017-11-01 07:40:34
tags:
  - linux
  - tech
  
---

> 以下linux操作都是基于ubuntu 16.04,别的linux发行版可能会有不同

## Linux 设置

### 修改主机名

临时修改直接用hostname newName就可以了

- 修改/etc/hostname,重启后生效
- 修改/etc/hosts的映射关系

### 创建新用户

```bash
# 新增admin用户,平时不使用root帐号
adduser admin

```

### 安装oh-my-zsh
[官方文档](https://github.com/robbyrussell/oh-my-zsh)
安装之前先要把zsh,git安装一下

```
sudo apt-get install zsh
sudo apt-get install git

#robbyrussell是默认主题,把全路径显示出来,这纯粹是个人习惯
/home/admin/.oh-my-zsh/themes/robbyrussell.zsh-theme

PROMPT='${ret_status} %{$fg[green]%}$USER@hz✭ %d%{$reset_color%} $(git_prompt_info)'
#PROMPT='${ret_status} %{$fg[cyan]%}%c%{$reset_color%} $(git_prompt_info)'

如果要区别不用的主机,也可以个性化一下,比如调个颜色[cyan] -> [yellow],或者加上用户名

```

具体颜色的设置参考[这里](https://gabri.me/blog/custom-colors-in-your-zsh-prompt/)

### autojump
目录跳转神器,`brew install autojump`,然后在.zshrc中的plugins中增加,比如plugins=(git autojump)  
>现在有更简单的办法了,oh-my-zsh中已经默认包含了一个`z`的插件,直接开启就行了plugins=(git z),如果不习惯的话,可以把`alias j='z'`
```
自动补全的插件也好用: https://github.com/zsh-users/zsh-autosuggestions
plugins=(git z zsh-autosuggestions)
source ~/.zshrc
```



### 修改apt源
推荐用[阿里云的源](http://mirrors.aliyun.com/)
如果购买的是阿里云ECS,还是把`http://mirrors.aliyun.com/`换成内网地址`http://mirrors.aliyuncs.com/`
修改完后,用apt-get update更新一下

>测试了一下新买的阿里云的ECS已经默认用的是aliyun自己的源了

### 修改ssh端口
>最好不要密码登录,而是使用密钥

配置文件路径

```
/etc/ssh/sshd_config
# 修改完成后重启ssh服务
sudo service ssh restart

```


### 安装Docker

参照[官方文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/#set-up-the-repository) ,就是添加一个官方的源,然后直接apt-get install就可以了.就是速度慢了点  
安装完之后,把当前用户加入到docker用户组中,否则所有命令都需要用`sudo`

```
sudo usermod -a -G docker $USER
```

## Mac 设置

### 系统设置
- 下载chrome并登陆，自动同步书签
- 下载Alfred，神器，不必多说
- iterm2,又一个神器
- 下载百度输入法,相比原生的还是略胜一筹,特别是原生的不支持`,.`选词
- 将fn键设置成默认
- 将触摸板设置成轻触表示点击,启动三指拖动(在设置的`辅助功能`中)
- 调节键盘设置中的按键重复(调成最快),延迟调短,删代码的时候会爽很多.

### Dock 设置 
- 删除一切不必要的图标,把启动交给神器Alfred
- 按住control点击下载图标,设置成自己想要的方式(我喜欢用文件夹+列表)
- 在dock设置中点击`将窗口最小化成应用程序图标`减少dock的占用空间
- 我个人希望把dock放在屏幕的左边,充分利用宽屏
- 在dock设置中启动缩放效果,这是mac的经典特性,不用浪费.