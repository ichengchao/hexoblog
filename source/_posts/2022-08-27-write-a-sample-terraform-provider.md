---
title: write a sample terraform provider
date: 2022-08-27 19:03:48
tags:

 - tech
 - terraform
 - go

---



#  准备

在开始写terraform provider之前,需要准备好几个东西:

- 一个服务端,包含简单的CRUD逻辑
- Go的运行环境,因为provider是用golang写的
- 安装好terraform,写完provider后用于测试



# 服务端CRUD

有一个简单的Demo类,还有对应的CRUD操作

```java
public class Demo {
	private String id;
	private String name;
	private String comment;
}
```

### Create

```
POST http://localhost:8080/demo/addDemo.do
formString: {"name":"zhangsan","comment":"testcomment"}
```

### Read

```
GET http://localhost:8080/demo/getDemoById.do?id=0bd0ced3-809a-46f3-919a-933264e73777
```

### Update

```
POST http://localhost:8080/demo/addDemo.do
formString: {"id":"0bd0ced3-809a-46f3-919a-933264e73777","name":"zhangsan11","comment":"testcomment11"}
```

### Delete

```
http://localhost:8080/demo/deleteDemo.do?id=0bd0ced3-809a-46f3-919a-933264e73777
```



# demo provider

创建如下目录结构和文件, 我们会定义一个resource和一个data

```
demo_provider
├── main.go
├── provider.go
├── data_demo_data.go
└── resource_demo_server.go
```

在demo_provider目录下执行编译

```
go mod init
go mod tidy
go build -o terraform-provider-demo

# 拷贝到缓存目录, 最后面是平台,比如mac_arm:darwin_arm64, linux: linux_amd64
cp terraform-provider-demo ~/.terraform.d/plugins/chengchao.name/demoprovider/demo/1.0.0/darwin_amd64/
```

# 测试

```
terraformtest
├── main.tf
└── versions.tf
```

### main.tf

```
resource "demo_server" "my-server1" {
	name = "name_1111"
	comment = "comment_11111"
}
resource "demo_server" "my-server2" {
	name = "name_22"
	comment = "comment_2233"
}
data "demo_data" "mydata1"{
}
output "myoutput" {
	value = data.demo_data.mydata1
}
```

### versions.tf

```
terraform {
  required_providers {
    demo = {
      version = "~> 1.0.0"
      source  = "chengchao.name/demoprovider/demo"
    }
  }
}
```

### do test

```
terraform init
terraform plan
terraform apply
```



# Provider源代码

### main.go

```go
// main.go
package main

import (
	"github.com/hashicorp/terraform-plugin-sdk/plugin"
	"github.com/hashicorp/terraform-plugin-sdk/terraform"
)

func main() {
	plugin.Serve(&plugin.ServeOpts{
		ProviderFunc: func() terraform.ResourceProvider {
			return Provider()
		},
	})
}
```

### provider.go

```go
// provider.go
package main

import (
	"github.com/hashicorp/terraform-plugin-sdk/helper/schema"
)

func Provider() *schema.Provider {
	return &schema.Provider{
		ResourcesMap: map[string]*schema.Resource{
			"demo_server": resourceDemoServer(),
		},
		DataSourcesMap: map[string]*schema.Resource{
			"demo_data": dataSourceDemoData(),
		},
	}
}
```

### data_demo_data.go

```go
// data_demo_data.go
package main

import "github.com/hashicorp/terraform-plugin-sdk/helper/schema"

func dataSourceDemoData() *schema.Resource {
	return &schema.Resource{
		Read: dataSourceDemoDataRead,
		Schema: map[string]*schema.Schema{
			"address": {
				Type:     schema.TypeString,
				Computed: true,
			},
			"comment": {
				Type:     schema.TypeString,
				Computed: true,
			},
		},
	}
}

func dataSourceDemoDataRead(d *schema.ResourceData, meta interface{}) error {
	d.SetId("testdemodata11")
	d.Set("address", "data_address_11")
	d.Set("comment", "data_comment_11")
	return nil
}
```



### resource_demo_server.go

```go
// resource_demo_server.go
package main

import (
	"io"
	"log"
	"net/http"
	"net/url"

	"github.com/hashicorp/terraform-plugin-sdk/helper/schema"
	"github.com/tidwall/gjson"
)

func resourceDemoServer() *schema.Resource {
	return &schema.Resource{
		Create: resourceDemoServerCreate,
		Read:   resourceDemoServerRead,
		Update: resourceDemoServerUpdate,
		Delete: resourceDemoServerDelete,

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

func resourceDemoServerCreate(d *schema.ResourceData, m interface{}) error {
	var name string = d.Get("name").(string)
	var comment string = d.Get("comment").(string)

	resp, err := http.PostForm("http://localhost:8080/demo/addDemo.do",
		url.Values{"formString": {"{'name':'" + name + "','comment':'" + comment + "'}"}})
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	//fmt.Println(string(body))
	jsonstring := string(body)
	id := gjson.Get(jsonstring, "data")
	d.SetId(id.String())

	return resourceDemoServerRead(d, m)
}

func resourceDemoServerRead(d *schema.ResourceData, m interface{}) error {
	resp, err := http.Get("http://localhost:8080/demo/getDemoById.do?id=" + d.Id())
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	jsonstring := string(body)
	name := gjson.Get(jsonstring, "data.name")
	comment := gjson.Get(jsonstring, "data.comment")
	d.Set("name", name.String())
	d.Set("comment", comment.String())
	return nil
}

func resourceDemoServerUpdate(d *schema.ResourceData, m interface{}) error {
	var name string = d.Get("name").(string)
	var comment string = d.Get("comment").(string)
	var id string = d.Id()

	resp, err := http.PostForm("http://localhost:8080/demo/updateDemo.do",
		url.Values{"formString": {"{'id':'" + id + "','name':'" + name + "','comment':'" + comment + "'}"}})
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	return resourceDemoServerRead(d, m)
}

func resourceDemoServerDelete(d *schema.ResourceData, m interface{}) error {
	resp, err := http.Get("http://localhost:8080/demo/deleteDemo.do?id=" + d.Id())
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	return nil
}
```
