---
title: 用Java写一个最简单的MCP Server
date: 2025-04-02 20:00:00
tags:
 - tech
 - java
 - mcp
 - ai
---

# **引言**

最近 MCP（Model Context Protocol）在开发者社区中逐渐火了起来，越来越多的项目实现了MCP 。关于它的背景、设计理念、协议细节等内容，网上已经有不少优秀的文章进行了深入剖析。本文就不再赘述相关概念，咱们开门见山，直接用 Java 写一个最简单的 MCP Server。



# **效果演示**

## **配置**

这里我们使用 Cline + VSCode 进行演示。假设我们已经构建好了一个符合 MCP 标准的 Jar 包，接下来只需要在 Cline 中做一些简单配置，就可以验证它的接入效果。配置完成后，我们可以在 MCP Server 中看到，系统中已经多出了一个新的工具节点 "userGroup"。这意味着我们的模块已经成功注册，并可以被大模型调用。

```json
{
  "mcpServers": {
    "userGroup": {
      "command": "java",
      "args": [
        "-jar",
        "/Users/charles/Documents/git/mcpserverdemo/target/mcpserverdemo.jar"
      ],
      "env": {
        "ALIBABA_CLOUD_ACCESS_KEY_ID": "LTAI5tJHDnKc****",
        "ALIBABA_CLOUD_ACCESS_KEY_SECRET": "U48j3ul1****"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

![img](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_04_02_1743576534.png)

## **演示视频**

测试三个case

1. 列出RAM用户组
2. 创建RAM用户组
3. 删除RAM用户组

<video controls="" src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/video/demo-mcp-server.mp4" title="Window-2025-03-26-163040.mp4" data-size="13462170"></video>



# **实现**

1. 使用的是Spring AI，最大程度简化代码
2. 使用aliyun 的 java sdk 做一个RAM用户组的增删查的操作
3. 按照下面结构写好后，直接利用打成jar包就行("mvn clean package -Dmaven.test.skip=true")

## **工程结构**

```
├── pom.xml
└── src
    └── main
        ├── java
        │   └── mcpserverdemo
        │       ├── MCPDemoApplication.java
        │       └── UserGroupService.java
        └── resources
            └── application.properties
```

## **pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.4.4</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>cc.landingzone</groupId>
	<artifactId>mcpserverdemo</artifactId>
	<version>0.0.1</version>
	<name>mcpserverdemo</name>

	<properties>
		<java.version>21</java.version>
		<maven.build.timestamp.format>yyyy-MM-dd_HHmm</maven.build.timestamp.format>
		<spring-ai.version>1.0.0-M6</spring-ai.version>
	</properties>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.ai</groupId>
				<artifactId>spring-ai-bom</artifactId>
				<version>${spring-ai.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<dependencies>
		<dependency>
			<groupId>org.springframework.ai</groupId>
			<artifactId>spring-ai-mcp-server-spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>2.0.56</version>
		</dependency>
		<dependency>
			<groupId>com.aliyun</groupId>
			<artifactId>aliyun-java-sdk-core</artifactId>
			<version>4.6.0</version>
		</dependency>
		<dependency>
			<groupId>com.aliyun</groupId>
			<artifactId>aliyun-java-sdk-ram</artifactId>
			<version>3.3.2</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<executions>
					<execution>
						<id>repackage</id>
						<goals>
							<goal>repackage</goal>
						</goals>
						<configuration>
							<finalName>mcpserverdemo</finalName>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

</project>
```

## **MCPDemoApplication.java**

```java
package mcpserverdemo;

import org.springframework.ai.tool.ToolCallbackProvider;
import org.springframework.ai.tool.method.MethodToolCallbackProvider;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class MCPDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MCPDemoApplication.class, args);
    }

    @Bean
    public ToolCallbackProvider weatherTools(UserGroupService userGroupService) {
        return MethodToolCallbackProvider.builder().toolObjects(userGroupService).build();
    }

}
```

## **UserGroupService.java**

```java
package mcpserverdemo;

import org.springframework.ai.tool.annotation.Tool;
import org.springframework.ai.tool.annotation.ToolParam;
import org.springframework.stereotype.Service;

import com.alibaba.fastjson.JSON;
import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.ram.model.v20150501.CreateGroupRequest;
import com.aliyuncs.ram.model.v20150501.CreateGroupResponse;
import com.aliyuncs.ram.model.v20150501.DeleteGroupRequest;
import com.aliyuncs.ram.model.v20150501.DeleteGroupResponse;
import com.aliyuncs.ram.model.v20150501.ListGroupsRequest;
import com.aliyuncs.ram.model.v20150501.ListGroupsResponse;

@Service
public class UserGroupService {

    DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", System.getenv("ALIBABA_CLOUD_ACCESS_KEY_ID"),
        System.getenv("ALIBABA_CLOUD_ACCESS_KEY_SECRET"));
    IAcsClient client = new DefaultAcsClient(profile);

    @Tool(description = "List all user group")
    public String listGroups() {
        ListGroupsRequest request = new ListGroupsRequest();
        try {
            ListGroupsResponse response = client.getAcsResponse(request);
            return JSON.toJSONString(response.getGroups());
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }

    @Tool(description = "Create user group")
    public String createGroup(@ToolParam(description = "group name") String groupName) {
        CreateGroupRequest request = new CreateGroupRequest();
        request.setGroupName(groupName);
        try {
            CreateGroupResponse response = client.getAcsResponse(request);
            return "create success! group:" + JSON.toJSONString(response.getGroup());
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }

    @Tool(description = "Delete user group")
    public String deleteGroup(@ToolParam(description = "group name") String groupName) {
        DeleteGroupRequest request = new DeleteGroupRequest();
        request.setGroupName(groupName);
        try {
            DeleteGroupResponse response = client.getAcsResponse(request);
            return "delete success! requestID:" + response.getRequestId();
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }
}
```

## **application.properties**

```properties
spring.main.web-application-type=none
spring.main.banner-mode=off
logging.pattern.console=
logging.file.name=./log/mcp-aliyun-api-server.log

spring.ai.mcp.server.name=aliyun-api-server
spring.ai.mcp.server.version=0.0.1
```

# **总结**

可以看到，MCP 作为一种协议，巧妙地将现有的大模型能力与标准化的 API 接口连接了起来。通过大模型现成的tools参数，让原本的API几乎无需改动就能对接大模型，实现上下文驱动的智能调用。这种设计既优雅又实用，极大地降低了开发门槛，也为大模型在实际业务场景中的落地提供了路径。可以看到三个小趋势：

1. token消耗量会大量增加，function列表信息(tools)也会记录到token中，而且如果调用了MCP Server，至少是两次调用
2. tools参数会不够用(例如Deepseek最多不能超过128个function)，或者需要在tools之前还要再做一个类似RAG的东西？
3. API会变成更加重要，有API才有了跟大模型集成的基础
