---
layout: post
title: windows7问题排查技巧
date: 2014-03-09 21:21
tags:
  - windows
  - tech
  
---

u盘或者移动硬盘不能弹出
---
用lockhunter查看被什么进程hold了.   网址: <http://lockhunter.com/>  

蓝屏
---

最近这段时间,我的电脑开机的时候经常直接蓝屏,今天决定好好看看是什么原因引起的.
查了下资料,大概分为以下几个步骤.
<ol>
	<li>下载windebug并安装,<a href="http://msdn.microsoft.com/zh-cn/windows/hardware/hh852365.aspx">下载地址</a></li>
	<li>查看目录C:\Windows\Minidump 是否有生成.dmp文件</li>
	<li>用管理员权限打开windbg,在File->symbol file path设置SRV*c:\temp*http://msdl.microsoft.com/download/symbols</li>
	<li>File->Open crash dump,找到dmp文件并打开</li>
	<li>因为语法文件需要根据需要下载,所以打开的时候有点慢,不要着急,等一等.</li>
	<li>看看Probably caused by : ***,后面的原因可以google一下,基本能判断出什么问题.</li>
</ol>
我的是Probably caused by : ProtectorA.sys ( ProtectorA+1ab0 )  
google一下,应该是中行的网银插件引起的,直接删除 ProtectorA.sys.问题解决.
