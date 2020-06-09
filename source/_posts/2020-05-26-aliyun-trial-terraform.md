---
title: 阿里云体验之Terraform
date: 2020-05-26 16:58:25
tags:

 - tech
 - aliyun
 - terraform
---



<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_06_09_14_41_1591684917659.png" alt="aliyun-trial-terraform-1" style="zoom:50%;" />

### Terraform是什么

>Use Infrastructure as Code to provision and manage any cloud, infrastructure, or service

为什么需要使用Terraform,[这篇文章](https://blog.gruntwork.io/why-we-use-terraform-and-not-chef-puppet-ansible-saltstack-or-cloudformation-7989dad2865c)说得比较透彻.跟传统配置管理工具的区别可以看看下面这张图,Terraform主要是提供资源本身,而系统配置不是Terraform的重点.加之近些年docker已经基本成为了事实标准,配置管理工具的生存范围会逐渐被docker image替代.同样的逻辑,如果kubernetes成为部署应用的标准解决方案后,Terraform的使用范围也会被蚕食.因为kubernetes的抽象层次更高,而且天然具备跨云的能力.目前Terraform在海外客户的接受程度还比较高,国内倒没怎么听说哪家大公司在重度使用.我个人判断,国内的这些企业客户很有可能会跳过这个阶段,直接进入k8s时代,即使用也很可能会架在k8s下面.接下来简单介绍一下Terraform在aliyun中的使用.



> 跟其他配置软件工具的区别

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_06_09_14_42_1591684935103.png" alt="aliyun-trial-terraform-2" style="zoom:50%;" />

### 准备

先看一下Terraform的三种AK认证方式

#### 1.明文写在配置文件(不推荐)

```js
provider "alicloud" {
  access_key = "your_access_key"
  secret_key = "your_access_secret"
  region     = "cn-hangzhou"
}
```



#### 2.用环境变量暴露

```js
provider "alicloud" {
}
```

然后export对应的环境变量

- ALICLOUD_ACCESS_KEY
- ALICLOUD_SECRET_KEY
- ALICLOUD_REGION

#### 3.使用Aliyun CLI配置文件中的Profile

```js
provider "alicloud" {
  profile = "default"
}
```

CLI的使用方式看[这里](https://github.com/aliyun/aliyun-cli),初始化完成后,直接使用配置文件中的Profile就行,默认的配置文件在`~/.aliyun/config.json`,配置文件中默认的Profile是`default`

#### 基本概念

配置文件的语法可以看[官方文档](https://www.terraform.io/docs/configuration/index.html),我这里只简单介绍一下本文会用到的

- data: 数据源,可以被resource引用,比如region列表,可用的image列表都是数据源,能提高`可复用性`
- resource: 配置文件中最重要的概念,一台机器,一个IP,一个域名,一个绑定关系都是resource
- output: 可以用于信息的打印,可以作为一种调试手段

#### 小试牛刀

通过上面的介绍,对基本的概念有了一个大概的了解,下面写个最简单的配置测试一下.(推荐使用vscode+[terraform插件](https://marketplace.visualstudio.com/items?itemName=mauve.terraform))

> 提示: 把配置文件直接拷贝到vscode中,可能会报错,不用管.那是因为目前插件还不支持最新语法,这个插件已经被[Terraform官方接管](https://www.hashicorp.com/blog/supporting-the-hashicorp-terraform-extension-for-visual-studio-code/),相信很快就会推出新的版本

```js
# 这里使用的AK和region都是从profile里面来的
provider "alicloud" {
  profile = "default"
}

# 列出阿里云所有的region
data "alicloud_regions" "region_list" {
}

# 定义一个输出,方便调试
output "regions" {
  value = data.alicloud_regions.region_list
}
```

```bash
terraform init  # init:第一次或者新增module的时候需要执行
terraform plan  # 试运行,会有语法检查
terraform apply # 真实运行,试运行通过也不代表这一步没有问题,有些问题试运行是检查不出来的
# 执行结果,也可以用terraform show看
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

regions = {
  "id" = "2613364496"
  "ids" = [
    "cn-qingdao",
    "cn-beijing",
    "cn-zhangjiakou",
    "cn-huhehaote",
    "cn-hangzhou",
    ......
```

### 进入主题

#### 选题

上面已经完成最简单的`hello world` ,接下来我们开始动手配置一个比较真实的场景,直接看图

> 默认你已经对阿里云的产品有一定的了解

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_06_09_14_43_1591684996406.png" alt="image-20200609144316209" style="zoom:50%;" />

从这个图中可以看出,这是一个最简单的web应用,后端使用mysql.前面用slb做负载均衡.涉及到的云产品有:

- SLB
- VPC
- VSW
- EIP
- 安全组
- ECS (2)
- RDS

在开始创建ECS之前,需要先把VPC,VSW,安全组创建好

```js
# 创建VPC
resource "alicloud_vpc" "charles_vpc" {
  name       = "charles_vpc"
  cidr_block = "172.16.0.0/12"
}

# 创建交换机
resource "alicloud_vswitch" "charles_vsw" {
  name              = "charles_vsw"
  vpc_id            = alicloud_vpc.charles_vpc.id
  cidr_block        = "172.16.0.0/24"
  availability_zone = "cn-hangzhou-h"
}

# 创建安全组
resource "alicloud_security_group" "charles_security_group" {
  name   = "charles_security_group"
  vpc_id = alicloud_vpc.charles_vpc.id
}
```

还是一样,执行`terraform apply`就能创建好,之后可以用`terraform state list`查看创建好的资源

#### 创建ECS

```js
# 创建ECS-> web1 649849
resource "alicloud_instance" "charles-web1" {
  image_id             = data.alicloud_images.ubuntu.ids.0
  internet_charge_type = "PayByBandwidth"
  instance_type        = "ecs.s6-c1m1.small"
  system_disk_category = "cloud_efficiency"
  security_groups      = [alicloud_security_group.charles_security_group.id]
  instance_name        = "charles-web1"
  host_name            = "charlesweb1"
  password             = "pass_1234"
  private_ip           = "172.16.0.1"
  vswitch_id           = alicloud_vswitch.charles_vsw.id
}

# 创建ECS-> web2 124428
resource "alicloud_instance" "charles-web2" {
  image_id             = data.alicloud_images.ubuntu.ids.0
  internet_charge_type = "PayByBandwidth"
  instance_type        = "ecs.s6-c1m1.small"
  system_disk_category = "cloud_efficiency"
  security_groups      = [alicloud_security_group.charles_security_group.id]
  instance_name        = "charles-web2"
  host_name            = "charlesweb2"
  password             = "pass_1234"
  private_ip           = "172.16.0.2"
  vswitch_id           = alicloud_vswitch.charles_vsw.id
}
```

#### 配置负载均衡

这里的步骤稍微多一点

- 创建SLB,默认没有公网地址
- 创建EIP并与SLB绑定
- 启动SLB的HTTP的监听,对外80,对后8080
- 将上面的两台ECS加入负载均衡

```js
# 创建SLB
resource "alicloud_slb" "charles_slb" {
  name                 = "charles_slb"
  vswitch_id           = alicloud_vswitch.charles_vsw.id
  internet_charge_type = "PayByTraffic"
}

# 创建EIP
resource "alicloud_eip" "charles_eip" {}

# 将EIP绑定到SLB
resource "alicloud_eip_association" "charles_eip_slb_asso" {
  allocation_id = alicloud_eip.charles_eip.id
  instance_id   = alicloud_slb.charles_slb.id
}

# SLB的监听配置
resource "alicloud_slb_listener" "charles_listener_http" {
  load_balancer_id = alicloud_slb.charles_slb.id
  backend_port     = "8080"
  frontend_port    = "80"
  protocol         = "http"
  bandwidth        = "5"
}

# 配置负载均衡的后端服务器
resource "alicloud_slb_backend_server" "charles_slb_backend_server" {
  load_balancer_id = alicloud_slb.charles_slb.id
  backend_servers {
    server_id = alicloud_instance.charles-web1.id
    weight    = 100
  }
  backend_servers {
    server_id = alicloud_instance.charles-web2.id
    weight    = 100
  }
}
```

到这一步基本的框架已经成型了,可以写个代码测试一下,我这里用go起了一个http server

```go
// hello.go
package main

import (
	"log"
	"net/http"
)

func main() {
	log.Fatal(http.ListenAndServe(":8080", http.FileServer(http.Dir("/tmp"))))
}
```

```bash
# 启动服务
nohup go run hello.go &
# 用curl测试一下EIP是否已经正常使用
curl "http://[eip address]"
```

#### 配置数据库

步骤:

- 创建一个RDS Mysql实例
- 创建一个高权限用户
- 创建一个database实例

```js
# RDS
resource "alicloud_db_instance" "charles_db_instance" {
  engine           = "MySQL"
  engine_version   = "5.6"
  instance_type    = "rds.mysql.s1.small"
  instance_storage = "10"
  vswitch_id       = alicloud_vswitch.charles_vsw.id
  instance_name    = "charles_db_rds"
}

# RDS 账号
resource "alicloud_db_account" "charles_account" {
  instance_id = alicloud_db_instance.charles_db_instance.id
  type        = "Super"
  name        = "charles"
  password    = "pass_1234"
}

# RDS database
resource "alicloud_db_database" "charles_rds_db_test" {
  instance_id   = alicloud_db_instance.charles_db_instance.id
  name          = "mytest"
  character_set = "utf8mb4"
  description   = "just for test"
}
```

```sql
# 建好测试用的表
CREATE TABLE `accesslog` (
	`id` int NOT NULL AUTO_INCREMENT,
	`log` varchar(1000) NULL,
	PRIMARY KEY (`id`)
) ENGINE=InnoDB
DEFAULT CHARACTER SET=utf8
COMMENT='access log';
```

到这里,最上面架构图里面的东西都已经部署完毕,并且配置好.接下来写一个简单的测试代码.

#### 业务代码

这个web服务非常简单,就是每次用户访问都写一条log到mysql.还是用go来写,代码里用到了mysql的driver,部署的时候需要在机器上先下载依赖`go get -u github.com/go-sql-driver/mysql`.[完整代码在这里](https://gist.github.com/ichengchao/27e8d73e00d9f7d1f94e67a513884337)

```go
//hello.go
package main

import (
	"database/sql"
	"fmt"
	"net/http"
	"os"
	"strconv"
	"time"

	_ "github.com/go-sql-driver/mysql"
)

func main() {
	http.HandleFunc("/hello", indexHandler)
	http.HandleFunc("/", rootHandler)
	http.ListenAndServe(":8080", nil)

}

func rootHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "ok")
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
	result := dolog()
	fmt.Fprintf(w, result)
}

func dolog() string {
	db, err := sql.Open("mysql", "root:password@tcp(mysql_ip:3306)/mytest?charset=utf8")
	if err != nil {
		panic(err)
	}
	insert(db)
	return query(db)
}

// query data
func query(db *sql.DB) string {
	rows, err := db.Query("select  * from accesslog order by id desc")
	if err != nil {
		panic(err)

	}
	var result string
	for rows.Next() {
		var id int
		var log string
		err = rows.Scan(&id, &log)
		//fmt.Println("id: " + strconv.Itoa(id) + ", log: " + log)
		result += "id: " + strconv.Itoa(id) + ", log: " + log + "\n"
	}
	return result
}

func insert(db *sql.DB) {
	stmt, err := db.Prepare("insert accesslog set log=?")
	timeStr := time.Now().Format("2006-01-02 15:04:05")
	hostname, err := os.Hostname()
	stmt.Exec(timeStr + " @ " + hostname)
	if err != nil {
		panic(err)
		return
	}

}
```

curl测试几次,大概会拿到下面的结果

```
id: 593, log: 2020-05-28 21:57:34 @ charlesweb1
id: 592, log: 2020-05-28 21:57:33 @ charlesweb2
id: 591, log: 2020-05-28 21:57:32 @ charlesweb1
id: 590, log: 2020-05-28 21:56:59 @ charlesweb2
id: 589, log: 2020-05-28 18:20:02 @ charlesweb1
id: 588, log: 2020-05-28 18:20:01 @ charlesweb2
id: 587, log: 2020-05-28 18:19:59 @ charlesweb2
id: 586, log: 2020-05-28 18:19:56 @ charlesweb1
id: 585, log: 2020-05-28 18:19:52 @ charlesweb2
id: 584, log: 2020-05-28 18:13:20 @ charlesweb1
id: 583, log: 2020-05-28 18:13:16 @ charlesweb1
```

### 总结

近年来,不管是上层的应用(k8s),到系统(docker),再到基础设施(terraform)都是声明式的.这个最大的好处就是继承了代码管理中的一些成熟经验,比如:

- 版本管理
- 能分享,能重用
- 一次编写,多处运行

一旦习惯了这个就再也回不去了,真香! ^_^

Terraform在国内的普及率还非常低,希望没有尝试过的同学都可以试用一下.文章中涉及的[完整配置文件在这里](https://gist.github.com/ichengchao/27e8d73e00d9f7d1f94e67a513884337)

