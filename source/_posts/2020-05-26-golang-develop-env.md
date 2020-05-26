---
title: 搭建Go开发环境
date: 2020-05-26 14:53:19
tags:

 - tech
 - go
---

### 安装

直接[官网](https://golang.org/)下载安装,mac使用brew更方便

```bash
brew install go
```

安装完成后设置一下GOPATH,一般是放在用户目录下

```
export GOPATH="/Users/charles/go"
```



### IDE设置

比较知名的有[jetbrain 的GoLand](https://www.jetbrains.com/go/),现在比较推荐vscode,免费而且插件丰富,关键是性能更好.

安装一下Microsoft官方出品的[Go插件](https://marketplace.visualstudio.com/items?itemName=ms-vscode.Go),然后运行一下命令行中的`Go:Install/Update tools`全选就行了.

写段测试代码试试

```go
package main

import "fmt"

func main() {
	fmt.Println("hello,world!")
}
```

