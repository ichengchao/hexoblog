---
title: 写一个简单的terraform provider
date: 2023-08-08 13:20:48
tags:

 - tech
 - terraform
 - go

---



# 背景

在云计算不断发展的今天, 基础设施即代码(Infrastructure as Code)逐渐成为趋势. 它将基础设施代码化,直接继承了代码管理的所有优点,比如: 版本管理, 跟CICD集成, 确保测试生产环境的一致性等等. 能极大的提高效率和减少人为错误. 而HashiCorp的Terraform无疑是这个领域的领头羊. 至于Terraform是什么,我就不多介绍了. 本文假设你已经有了一定Terraform基础, 将介绍如何写一个简单的Terraform Provider. 

#  准备

![](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/neweditor/3521cbe5-fca7-4299-bbce-de5921d361ab.png)
Terraform其实由Terraform Core和Terraform Plugins (各种Provider)组成:

- Terraform Core: 负责读取配置文件并构建资源关系图谱
- Terraform Provider: Provider通过基本 CRUD API 与第三方服务进行通信

比如创建阿里云的一台ECS就可以通过以下的配置文件实现:

```terraform
provider "alicloud" {
  region = "cn-hangzhou"
}

resource "alicloud_instance" "instance_demo_1" {
  vswitch_id          = "vsw-bp13emwb6r********"
  instance_type       = "ecs.n2.small"
  instance_name       = "instance_demo_1"
  count               = 1
  image_id            = "ubuntu_18_04_64_20G_alibase_20190624.vhd"
  security_groups     = ["sg-bp15hf3akh7*******"]
}
```

其背后就是调用了阿里云的OpenAPI实现的, 下面我用一个简单的例子来模拟一下这个过程.

# 第一步: 定义User对象,并构建出对应的API

我们先定义一个简单的类User

```java
public class User {
	String id;
	String name;
	String comment;
}
```

接着构建出对应的CRUD API
**Create**

```bash
curl --request POST 'http://localhost:8080/rest/user' \
--header 'Content-Type: application/json' \
--data-raw '{"name":"zhangsan","comment":"test"}'

# response:
# {"id":"4126ef31-4f4e-453d-9290-6360b77c5a6c","name":"zhangsan","comment":"test"}
```

**Read**

```bash
curl --request GET 'http://localhost:8080/rest/user/4126ef31-4f4e-453d-9290-6360b77c5a6c'
# response:
# {"id":"4126ef31-4f4e-453d-9290-6360b77c5a6c","name":"zhangsan","comment":"test"}
```

**Update**

```bash
curl --request PUT 'http://localhost:8080/rest/user/4126ef31-4f4e-453d-9290-6360b77c5a6c' \
--header 'Content-Type: application/json' \
--data-raw '{"name": "zhangsan-v1","comment": "test-v1"}'
# response:
# {"id":"4126ef31-4f4e-453d-9290-6360b77c5a6c","name":"zhangsan-v1","comment":"test-v1"}
```

**Delete**

```bash
curl --request DELETE 'http://localhost:8080/rest/user/4126ef31-4f4e-453d-9290-6360b77c5a6c'
# response:
# {"id":"4126ef31-4f4e-453d-9290-6360b77c5a6c","name":"zhangsan-v1","comment":"test-v1"}
```

**List**

> 这个接口用于测试的时候方便校验后台数据,Provider不使用

```bash
curl --request GET 'http://localhost:8080/rest/users'
# response 
# [
# {"id":"4126ef31-4f4e-453d-9290-6360b77c5a6c","name":"zhangsan","comment":"test"},
# {"id":"1c7dbee9-3992-41b4-a08c-1e653a5ca30a","name":"wangwu","comment":"test-wang"}
# ]
```

以上就是一个User类的定义,以及使用curl操作API的代码,是不是非常简单,对应的服务端的代码我不就展示了,([代码在这里](https://github.com/ichengchao/terraformProviderServer))可以用自己喜欢的技术栈实现一个就可以了,下面我们开始来构建对应的Terraform Provider

# 第二步: 构建Terraform provider

在开始构建之前,我们先定义好一些meta信息如下 (这个关系到provider文件的存放地址):

```
provider path: ~/.terraform.d/plugins/${host_name}/${namespace}/${type}/${version}/${target}
hostname: landingzone.cc
namespace: landingzone
type: mydemo
version: 1.0.0
target: darwin_amd64
```

创建如下目录结构和文件, 我们会定义一个resource(User)和一个data(Country), 这两种是我们最常用的类型

```
demo_provider
├── main.go
├── provider.go
├── data_country.go
└── resource_user.go
```

在demo_provider目录下执行编译 (对应的源代码附在最后面)

```bash
go mod init
go mod tidy
go build -o terraform-provider-mydemo

# 拷贝到缓存目录, 最后面是平台,比如mac_arm:darwin_arm64, linux: linux_amd64
 mv terraform-provider-mydemo ~/.terraform.d/plugins/landingzone.cc/landingzone/mydemo/1.0.0/darwin_arm64
```

别忘记需要安装Terraform, 要不然后面测试会跑不起来

# 第三部: 测试

```
terraformtest
├── main.tf
└── versions.tf
```

**main.tf**

```terraform
# resource test
resource "mydemo_user" "userA" {
  name    = "name_A"
  comment = "comment_A"
}
resource "mydemo_user" "userB" {
  name    = "name_B"
  comment = "comment_B"
}

# data test
data "mydemo_country" "countryList" {
}
output "mycounty" {
  value = data.mydemo_country.countryList.items[0].country_name
}
output "countryList" {
  value = data.mydemo_country.countryList
}
```

**versions.tf**

```terraform
terraform {
  required_providers {
    mydemo = {
      version = "~> 1.0.0"
      source  = "landingzone.cc/landingzone/mydemo"
    }
  }
}
```

**运行测试**

```bash
# 在terraformtest目录下执行
terraform init
terraform plan
terraform apply

# 如果运行正常应该就能看到类似的结果,可以使用List接口看看后台数据是否已经正确更新
mydemo_user.userA: Creating...
mydemo_user.userB: Creating...
mydemo_user.userB: Creation complete after 0s [id=78e8394f-5bdf-4264-bc54-10ae35a556b4]
mydemo_user.userA: Creation complete after 0s [id=d85d0224-5c45-4f31-bc95-832fbcdedc01]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

countryList = {
  "id" = "countID-1234"
  "items" = tolist([
    {
      "country_id" = 1
      "country_name" = "CHINA"
    },
    {
      "country_id" = 2
      "country_name" = "JAPAN"
    },
  ])
}
mycounty = "CHINA"
```

到此,一个简单的Terraform Provider已经构建完成了,我们可以继续修改main.tf来测试修改和删除的case. 真实的Provider的构建会比这个复杂很多,也会有更多的特殊情况需要处理,这里只是用最简单的例子而已.

# 附录: Provider源代码

### main.go

```go
package main

import (
	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
	"github.com/hashicorp/terraform-plugin-sdk/v2/plugin"
)

func main() {
	plugin.Serve(&plugin.ServeOpts{
		ProviderFunc: func() *schema.Provider {
			return Provider()
		},
	})
}

```

### provider.go

```go
package main

import (
	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
)

func Provider() *schema.Provider {
	return &schema.Provider{
		ResourcesMap: map[string]*schema.Resource{
			"mydemo_user": resourceUser(),
		},
		DataSourcesMap: map[string]*schema.Resource{
			"mydemo_country": dataSourceCountry(),
		},
	}
}

```

### data_country.go

```go
package main

import (
	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
)

func dataSourceCountry() *schema.Resource {
	return &schema.Resource{
		Read: dataSourceCountryRead,
		Schema: map[string]*schema.Schema{
			"id": {
				Type:     schema.TypeString,
				Optional: true,
			},
			"items": {
				Type:     schema.TypeList,
				Computed: true,
				Elem: &schema.Resource{
					Schema: map[string]*schema.Schema{
						"country_id": {
							Type:     schema.TypeInt,
							Computed: true,
						},
						"country_name": {
							Type:     schema.TypeString,
							Computed: true,
						},
					},
				},
			},
		},
	}
}

func dataSourceCountryRead(d *schema.ResourceData, m interface{}) error {
	//这里简化了一下,直接hard code了,也可以从server端获取
	d.SetId("countID-1234")
	countryList := make([]interface{}, 3)
	country_0 := make(map[string]interface{})
	country_0["country_id"] = 1
	country_0["country_name"] = "China"
	country_1 := make(map[string]interface{})
	country_1["country_id"] = 2
	country_1["country_name"] = "Singapore"
	country_2 := make(map[string]interface{})
	country_2["country_id"] = 3
	country_2["country_name"] = "Japan"
	countryList[0] = country_0
	countryList[1] = country_1
	countryList[2] = country_2
	d.Set("items", countryList)
	return nil
}
```



### resource_user.go

```go
package main

import (
	"encoding/json"
	"io"
	"log"
	"net/http"
	"strings"

	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
)

func resourceUser() *schema.Resource {
	return &schema.Resource{
		Create: resourceUserCreate,
		Read:   resourceUserRead,
		Update: resourceUserUpdate,
		Delete: resourceUserDelete,

		Schema: map[string]*schema.Schema{
			"name": {
				Type:     schema.TypeString,
				Required: true,
			},
			"comment": {
				Type:     schema.TypeString,
				Required: true,
			},
		},
	}
}

func resourceUserCreate(d *schema.ResourceData, m interface{}) error {
	var name string = d.Get("name").(string)
	var comment string = d.Get("comment").(string)

	postBody := strings.NewReader("{\"name\":\"" + name + "\",\"comment\":\"" + comment + "\"}")
	resp, err := http.Post("http://localhost:8080/rest/user", "application/json", postBody)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	log.Printf("[INFO][Create] body: %s", body)
	var result map[string]interface{}
	json.Unmarshal([]byte(string(body)), &result)
	idstring := result["id"].(string)
	d.SetId(idstring)
	return resourceUserRead(d, m)
}

func resourceUserRead(d *schema.ResourceData, m interface{}) error {
	resp, err := http.Get("http://localhost:8080/rest/user/" + d.Id())
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	var result map[string]interface{}
	json.Unmarshal([]byte(string(body)), &result)
	comment := result["comment"].(string)
	name := result["name"].(string)
	log.Printf("[INFO][Read] name: %s,comment: %s", name, comment)
	d.Set("name", name)
	d.Set("comment", comment)
	return nil
}

func resourceUserUpdate(d *schema.ResourceData, m interface{}) error {
	var name string = d.Get("name").(string)
	var comment string = d.Get("comment").(string)
	var id string = d.Id()

	postBody := strings.NewReader("{\"name\":\"" + name + "\",\"comment\":\"" + comment + "\"}")
	req, err := http.NewRequest(http.MethodPut, "http://localhost:8080/rest/user/"+id, postBody)
	req.Header.Set("Content-Type", "application/json")
	if err != nil {
		log.Fatal(err)
	}
	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	log.Printf("[INFO][Update] body: %s", body)
	return resourceUserRead(d, m)
}

func resourceUserDelete(d *schema.ResourceData, m interface{}) error {
	var id string = d.Id()
	req, err := http.NewRequest(http.MethodDelete, "http://localhost:8080/rest/user/"+id, nil)
	req.Header.Set("Content-Type", "application/json")
	if err != nil {
		log.Fatal(err)
	}
	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	log.Printf("[INFO][Delete] body: %s", body)
	return nil
}

```
