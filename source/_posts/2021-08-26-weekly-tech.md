---
title: weekly tech
date: 2021-08-26 19:03:48
tags:

 - tech
 - weekly

---

### AskGit
官网: https://askgit.com/  
用SQL统计分析git的提交记录,特别好用,举几个例子.官方还有更加复杂的例子

```sql
-- 按月统计提交次数
SELECT Count (*),
       Strftime('%Y-%m', committer_when) AS month
FROM   commits
GROUP  BY month
ORDER  BY month DESC

-- 提交次数大于等于10次的人员统计
SELECT author_email,
       Count(*)
FROM   commits
GROUP  BY author_email
HAVING Count(*) >= 10
ORDER  BY Count(*) DESC 

```

### Linux 发行版
除了我们常见的ubuntu之外,推荐两个比较有名的发行版

- [Linux Mint](https://linuxmint.com/) : 基于ubuntu的轻量级发行版
- [Manjaro](https://manjaro.org/) : 比较漂亮

### github.dev
官网: https://github.dev  
一个github官方推出的基于vscode的web编辑器,在已有的github项目中只要按一下 `.` 就能快速进入.不同于官方推出的[codespace](https://github.com/features/codespaces),这个web版本的编辑器不是运行在一个单独的容器内,所以不是所有插件都能安装,虽然不能和完整的IDE相比,但个人觉得已经非常好用了.最关键的是能和github无缝集成,修改完后可以直接提交,爽!这篇文章就是用这个写的^_^  
顺便推荐一个github的镜像站: https://fastgit.org/


### keychron键盘
官网: https://www.keychron.com/  
性价比挺高的一个品牌,关键还挺好看,很多YouTuber大佬都在用,我最近入手了k3,到目前为止感受还不错.