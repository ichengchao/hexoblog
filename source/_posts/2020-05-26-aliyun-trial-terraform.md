---
title: 阿里云体验之Terraform
date: 2020-05-26 16:58:25
tags:

 - tech
 - aliyun
 - terraform
---

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/image/aliyun-trial-terraform-1.png" alt="Dynamic Infrastructure" style="zoom:50%;" />



### Terraform是什么

>Use Infrastructure as Code to provision and manage any cloud, infrastructure, or service

为什么需要使用Terraform,[这篇文章](https://blog.gruntwork.io/why-we-use-terraform-and-not-chef-puppet-ansible-saltstack-or-cloudformation-7989dad2865c)说得比较透彻.跟传统配置管理工具的区别可以看看下面这张图,Terraform主要是提供资源本身,而系统配置不是Terraform的重点.加之近些年docker已经基本成为了事实标准,配置管理工具的生存范围会逐渐被docker image替代.同样的逻辑,如果kubernetes成为部署应用的标准解决方案后,Terraform的使用范围也会被蚕食.因为kubernetes的抽象层次更高,而且天然具备跨云的能力.目前Terraform在海外客户的接受程度还比较高,国内倒没怎么听说哪家大公司在重度使用.我个人判断,国内的这些企业客户很有可能会跳过这个阶段,直接进入k8s时代,即使用也很可能会架在k8s下面.接下来简单介绍一下Terraform在aliyun中的使用.



> 跟其他配置软件工具的区别

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/image/aliyun-trial-terraform-2.png" alt="image-20200526174425413" style="zoom:50%;" />

### 场景

设定要使用Terraform来初始化下面这个简单架构图

- SLB
- VPC
- VSW
- EIP
- 安全组
- ECS (2)
- RDS

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/upic/2020_05_28_image-20200528214208661.png" alt="image-20200528214208661" style="zoom:50%;" />

### 环境准备

先看一下Terraform的认证方式,有三种:

- 直接明文写在配置文件中
- 使用环境变量
- 使用CLI中的profile (推荐)
  - 使用方式看[这里](https://github.com/aliyun/aliyun-cli)
  - 在`~/.aliyun/config.json`中生成profile,默认名称是`default`

先写一个最简单的配置文件来测试一下,推荐使用vscode+terraform插件

```json
# 这里使用的AK和region都是从profile里面来的
provider "alicloud" {
  profile = "default"
}

data "alicloud_regions" "current_region_ds" {
  current = true
}

# 定义一个输出,方便调试
output "current_region" {
  value = data.alicloud_regions.current_region_ds.regions.0
}
```

```bash
terraform init  # init:第一次或者新增module的时候需要执行
terraform plan  # 试运行,会有语法检查
terraform apply # 真实运行,试运行通过也不代表这一步没有问题,有些问题试运行是检查不出来的
# 执行结果,也可以用terraform show看
Outputs:
current_region = {
  "id" = "cn-hangzhou"
  "local_name" = "华东 1"
  "region_id" = "cn-hangzhou"
}
```

###前置资源

在创建ECS之前,需要先把VPC,VSW,安全组创建好

```json
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

###创建ECS

```json
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

### 配置负载均衡

这里的步骤稍微多一点

- 创建SLB,默认没有公网地址
- 创建EIP并与SLB绑定
- 启动SLB的HTTP的监听,对外80,对后8080
- 将上面的两台ECS加入负载均衡

```json
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

### 配置数据库

步骤:

- 创建一个RDS Mysql实例
- 创建一个高权限用户
- 创建一个database实例

```json
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

```mysql
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

### 业务代码

这个web服务非常简单,就是每次用户访问都写一条log到mysql.还是用go来写,代码里用到了mysql的driver,部署的时候需要在机器上先下载依赖`go get -u github.com/go-sql-driver/mysql`.

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

近年来,不管是上层的应用(k8s),到系统(docker),再到基础设施(terraform)都面向配置文件的.这个最大的好处就是继承了代码管理中的所有好处.

- 版本管理
- 能分享,能重用
- 一次编写,多处运行

一旦习惯了这个就再也回不去了,真香!

Terraform在国内的普及率还非常低,希望没有尝试过的同学都可以试用一下.文章中涉及的完整配置文件在[这里](https://gist.github.com/ichengchao/27e8d73e00d9f7d1f94e67a513884337)