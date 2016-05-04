---
layout: post
title: Git入门
date: 2015-11-07 10:54
tags:
  - tech
  - git

---

![](http://www.chengchao.name/resource-container/image/git.jpg)

###环境准备
想要学习git,就必须先有个git环境.最有名的当然就是Github.不过访问速度时好时坏,所以建议用一个国内的git仓库.我推荐的是oschina: [http://git.oschina.net/](http://git.oschina.net/) 使用方式跟Github几乎一样,速度快了很多.btw:他家的maven仓库的速度也不错.[http://maven.oschina.net/home.html](http://maven.oschina.net/home.html)
申请好之后,导入ssh公钥就可以开始测试了.


###配置

```sh
git config --global user.name "charles"
git config --global user.email "charles@chengchao.name"
git config --global push.default simple
git config --global http.postBuffer 10485760 #上传最大10MB

#别名
git config --global alias.s status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.b branch
```

常用命令:

```sh
#查看配置,一般在~/.gitconfig
git config -l
#删除配置
git config --global --unset #key#

#获取远程分支,最好名称是一样的
git fetch
git co 分支名称

#直接切换
git co -b 新建分支名称

#查看所有分支,如果没有看到远程分支,需要git fetch一下
git b -a
```

###使用
基本的使用命令,很多地方都有,这边就不做介绍了.下面我通过一个测试项目来模拟一个真实的开发过程.
首先在git仓库新建一个项目`git-test`.  
第一步就是先导出到本地.命令很简单,就是git clone,可以选择http和ssh两种方式,我使用的ssh.由于之前已经把公钥导入到git仓库中.所以后续的操作都不用再输入用户名和密码了.  
`git clone git@git.oschina.net:ichengchao/git-test.git`  
有个比较特殊的文件需要简单介绍一下`.gitignore`,该文件用于忽略一些文件或者文件夹,这个文件或者文件夹就不会提交到仓库中.简单的语法如下:

```sh
#忽略所有.class结尾的文件
*.class

#不包含vip.class
!vip.class

#忽略根目录下的build文件夹,会忽略/build,但不会忽略/tmp/build目录
/build/

#忽略所有target目录,会同时忽略/target和/tmp/target目录
target/
```
先新建一个文件master.txt,再推送仓库,当做是初始化master分支

```sh
echo "master content">>master.txt
git add master.txt
git ci -m "init master"
git push
```
现在需要开发新功能叫featureA,那就需要新建一个分支.

```sh
#新建并切换到分支featureA
git co -b featureA

#提交分支到远程仓库
git push origin featureA

#模拟开发分支,增加一个featureA.txt文件,同时修改一下master.txt的内容
echo "feature a">>featureA.txt
echo "modify by featureA">>master.txt

#提交
git add *
git ci -m "featureA done"
#第一次提交需要set-upstream
git push --set-upstream origin featureA
```
到此,featureA的功能已经全部开发完成,等待测试完成后合并到master分支并发布.这时线上发现了一个bug,需要紧急修复.暂且把这个修复叫做hotfixA,同样的,这种操作也不能直接在master分支上操作.需要新建一个分支:hotfixA,但这个分支不用提交到远程仓库,直接本地测试完后就可以合并到master分支中.操作如下:

```sh
#先切换回master分支
git co master
#新建并切换到hotfixA分支
git co -b hotfixA

#该hotfix需要增加一个hotfixA.txt文件,同时也要修改一下master.txt中的内容
echo "hotfix a">>hotfixA.txt
echo "modify by hotfixA">>master.txt

git add * && git ci -m "hotfixA done"
```

hotfixA开发完成,假设测试也已经通过,需要合并到master分支中

```sh
#切换回master分支
git co master

#合并hotfixA分支
git merge hotfixA

#提交
git push

#删除本地的hotfixA分支
git b -d hotfixA
```

hotfixA上线后,featureA的功能也已经通过测试,需要合并到master,操作如下:

```sh
git merge featureA

#由于hotfixA中的修改和featureA中的修改有冲突,提示如下:

Auto-merging master.txt
CONFLICT (content): Merge conflict in master.txt
Automatic merge failed; fix conflicts and then commit the result.
```
手工解决冲突的方式有很多种,最简单的就是直接编辑冲突的文件.修改完成后重新执行`git add fileName`就可以了.不过如果文件内容冲突的很厉害,而且文件内容比较多的情况下,这种合并方式的难度就比较大了.这里推荐使用`git mergetool`命令.我用的是[SourceGear | DiffMerge](https://sourcegear.com/diffmerge/)工具.下载mac版本的installer方式安装.同时指定`git config --global merge.tool diffmerge`即可.不管使用那种方式merge,完成后执行如下操作:

```sh
git add *
git ci -m "merge featureA branch"
git push
```
这样master分支就可以发布了,一般我们发布完成后会打一个release的tag

```sh
#打一个release tag并推送到仓库
git tag release-featureA
git push --tag
```
下图为整个过程:  
![git-tree](http://www.chengchao.name/resource-container/image/git-test-tree.png)  

至此,一个完整的开发流程已经模拟完.基本的git使用应该没有问题了.  
作业题: 把上面的`git merge`操作用`git rebase`替换.

终于可以安心的睡觉去了,刚刚躺下就接到了电话说featureA线上有缺陷,需要马上回滚.

```sh
git log

#选择提交hotfix的id
git reset --hard 103f501e31add9cee2aabcc4f7e1756f8bbefdcd

#强制覆盖线上版本
git push --force
```
搞定.当然也可以选择使用`git revert`来回滚,当使用`git revert`时,原来的featureA分支将不能再次合并.更多关于git reset和git revert的区别参见[该文章](http://yijiebuyi.com/blog/8f985d539566d0bf3b804df6be4e0c90.html)

###使用技巧
好用的命令

```sh
#格式化查看log
git log --pretty=format:"%h - %an, %ar : %s"

git log --pretty=format:"%h %s" --graph
```
