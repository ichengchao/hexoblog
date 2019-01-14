---
title: protobuf and grpc
date: 2019-01-14 10:32:19
tags:

 - tech
 - google
 - grpc
 - protobuf
 - java
 
---



### 准备

官方文档

- [protobuf文档](https://developers.google.com/protocol-buffers/docs/overview)
- [protobuf 编译器](https://github.com/protocolbuffers/protobuf/releases),下载下来后把protoc放到/usr/local/bin中
- [grpc文档(java)](https://grpc.io/docs/quickstart/java.html)

下载一个proto的编辑器,这里推荐vs code中的[proto3插件](https://marketplace.visualstudio.com/items?itemName=zxh404.vscode-proto3)


### hello protobuf

> student.proto
```protobuf
syntax = "proto3";
package chengchao.prototest;

option java_package = "name.chengchao.myprotobuf";
option java_outer_classname = "StudentProtos";


message Student {
    string name = 1;
    int32 age = 2;
    Gender gendar = 3;

    enum Gender {
        BOY = 0;
        GIRL = 1;
    }
}
```

```bash
# 编译
protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto
```

在java工程中添加protobuf的依赖

```xml
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.6.1</version>
</dependency>

```

测试一下序列化和反序列化

```java
package name.chengchao.myprotobuf;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

import name.chengchao.myprotobuf.StudentProtos.Student;
import name.chengchao.myprotobuf.StudentProtos.Student.Gender;

public class Test {

	public static void main(String[] args) throws FileNotFoundException, IOException {

		Student student = Student.newBuilder().setAge(10).setName("nina").setGendar(Gender.GIRL).build();

		student.toBuilder().build().writeTo(new FileOutputStream(new File("/Users/charles/Desktop/protobuf_student")));

		System.out.println(student);
		
		
		System.out.println("==========copy============");
		Student studentCopy = Student.parseFrom(new FileInputStream(new File("/Users/charles/Desktop/protobuf_student")));
		System.out.println(studentCopy);

	}

}

```



### hello grpc
