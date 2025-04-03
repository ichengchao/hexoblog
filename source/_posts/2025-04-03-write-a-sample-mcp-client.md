---
title: 用Java写一个最简单的MCP Client
date: 2025-04-03 20:00:00
tags:
 - tech
 - java
 - mcp
 - ai
---



# 引言

之前用 [Java 实现了一个最简单的 MCP Server](./2025/04/02/write-a-sample-mcp-server/)。为了更全面地体验 MCP 协议的完整链路，继续用 Java 编写一个最简单的 MCP Client，搭配Cloudflare的AI Gateway能看到整个MCP的交互过程。



# **效果演示**

<video controls="" src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/video/demo-mcp-client-v1.mp4" title="Screen-2025-03-29-145547.mp4" data-size="13212340"></video>



# **交互过程**

## **交互时序**

![img](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_04_03_1743643105.png)

1. MCP Client会向LLM发送用户问题，同时在tools附上MCP Server提供的所有function以及说明
2. LLM根据用户问题自行判断需要tools中的function，返回需要调用的function以及对应的参数
3. MCP Client根据指示完成function的调用，并将调用结果和用户原始问题发送给LLM
4. LLM根据用户问题以及function的结果整理最终答案返回给MCP Client

## **交互日志**

### **1. Client第一次请求**

```json
{
  "messages": [
    {
      "content": "列出所有的用户组",
      "role": "user"
    }
  ],
  "model": "deepseek-chat",
  "stream": false,
  "temperature": 0.7,
  "tools": [
    {
      "type": "function",
      "function": {
        "description": "Creat user group",
        "name": "createGroup",
        "parameters": {
          "additionalProperties": false,
          "type": "object",
          "properties": {
            "groupName": {
              "type": "string",
              "description": "group name"
            }
          },
          "required": [
            "groupName"
          ]
        }
      }
    },
    {
      "type": "function",
      "function": {
        "description": "List all user group",
        "name": "listGroups",
        "parameters": {
          "additionalProperties": false,
          "type": "object",
          "properties": {},
          "required": []
        }
      }
    },
    {
      "type": "function",
      "function": {
        "description": "Delete user group",
        "name": "deleteGroup",
        "parameters": {
          "additionalProperties": false,
          "type": "object",
          "properties": {
            "groupName": {
              "type": "string",
              "description": "group name"
            }
          },
          "required": [
            "groupName"
          ]
        }
      }
    }
  ]
}
```

### 2. LLM第一次响应

```json
{
  "id": "00e396d6-eddb-4771-b1bd-cc481be4cc11",
  "object": "chat.completion",
  "created": 1743230174,
  "model": "deepseek-chat",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "",
        "tool_calls": [
          {
            "index": 0,
            "id": "call_0_e12efc33-081b-4e58-ab60-01e7ef95f6b5",
            "type": "function",
            "function": {
              "name": "listGroups",
              "arguments": "{}"
            }
          }
        ]
      },
      "logprobs": null,
      "finish_reason": "tool_calls"
    }
  ],
  "usage": {
    "prompt_tokens": 271,
    "completion_tokens": 14,
    "total_tokens": 285,
    "prompt_tokens_details": {
      "cached_tokens": 256
    },
    "prompt_cache_hit_tokens": 256,
    "prompt_cache_miss_tokens": 15
  },
  "system_fingerprint": "fp_3d5141a69a_prod0225"
}
```

### 3. Client第二次请求

```json
{
  "messages": [
    {
      "content": "列出所有的用户组",
      "role": "user"
    },
    {
      "content": "",
      "role": "assistant",
      "tool_calls": [
        {
          "id": "call_0_e12efc33-081b-4e58-ab60-01e7ef95f6b5",
          "type": "function",
          "function": {
            "name": "listGroups",
            "arguments": "{}"
          }
        }
      ]
    },
    {
      "content": "[{\"type\":\"text\",\"text\":\"\\\"[{\\\\\\\"comments\\\\\\\":\\\\\\\"\\\\\\\",\\\\\\\"createDate\\\\\\\":\\\\\\\"2021-11-29T09:51:24Z\\\\\\\",\\\\\\\"groupId\\\\\\\":\\\\\\\"g-zfpYxjHnqOwbN3MJ\\\\\\\",\\\\\\\"groupName\\\\\\\":\\\\\\\"mygroup\\\\\\\",\\\\\\\"updateDate\\\\\\\":\\\\\\\"2021-11-29T09:51:24Z\\\\\\\"},{\\\\\\\"comments\\\\\\\":\\\\\\\"测试备注\\\\\\\",\\\\\\\"createDate\\\\\\\":\\\\\\\"2022-05-25T07:53:20Z\\\\\\\",\\\\\\\"groupId\\\\\\\":\\\\\\\"g-lEiWMkA83dbdytz3\\\\\\\",\\\\\\\"groupName\\\\\\\":\\\\\\\"testgroup2\\\\\\\",\\\\\\\"updateDate\\\\\\\":\\\\\\\"2022-07-04T09:53:23Z\\\\\\\"},{\\\\\\\"comments\\\\\\\":\\\\\\\"\\\\\\\",\\\\\\\"createDate\\\\\\\":\\\\\\\"2024-11-20T07:30:44Z\\\\\\\",\\\\\\\"groupId\\\\\\\":\\\\\\\"g-AgYxkaRwtAKrMs5z\\\\\\\",\\\\\\\"groupName\\\\\\\":\\\\\\\"zhangsangroup\\\\\\\",\\\\\\\"updateDate\\\\\\\":\\\\\\\"2024-11-20T07:30:44Z\\\\\\\"}]\\\"\"}]",
      "role": "tool",
      "name": "listGroups",
      "tool_call_id": "call_0_e12efc33-081b-4e58-ab60-01e7ef95f6b5"
    }
  ],
  "model": "deepseek-chat",
  "stream": false,
  "temperature": 0.7,
  "tools": [
    {
      "type": "function",
      "function": {
        "description": "Creat user group",
        "name": "createGroup",
        "parameters": {
          "additionalProperties": false,
          "type": "object",
          "properties": {
            "groupName": {
              "type": "string",
              "description": "group name"
            }
          },
          "required": [
            "groupName"
          ]
        }
      }
    },
    {
      "type": "function",
      "function": {
        "description": "List all user group",
        "name": "listGroups",
        "parameters": {
          "additionalProperties": false,
          "type": "object",
          "properties": {},
          "required": []
        }
      }
    },
    {
      "type": "function",
      "function": {
        "description": "Delete user group",
        "name": "deleteGroup",
        "parameters": {
          "additionalProperties": false,
          "type": "object",
          "properties": {
            "groupName": {
              "type": "string",
              "description": "group name"
            }
          },
          "required": [
            "groupName"
          ]
        }
      }
    }
  ]
}
```

### 4. LLM第二次响应

```json
{
  "id": "6056aae5-7623-48af-a4e3-45672d732f31",
  "object": "chat.completion",
  "created": 1743230179,
  "model": "deepseek-chat",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "当前系统中的用户组如下：\n\n1. **mygroup**\n   - 创建日期: 2021-11-29\n   - 备注: 无\n\n2. **testgroup2**\n   - 创建日期: 2022-05-25\n   - 备注: 测试备注\n   - 最后更新日期: 2022-07-04\n\n3. **zhangsangroup**\n   - 创建日期: 2024-11-20\n   - 备注: 无\n\n如果需要进一步操作（如创建或删除用户组），请告诉我！"
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 580,
    "completion_tokens": 123,
    "total_tokens": 703,
    "prompt_tokens_details": {
      "cached_tokens": 256
    },
    "prompt_cache_hit_tokens": 256,
    "prompt_cache_miss_tokens": 324
  },
  "system_fingerprint": "fp_3d5141a69a_prod0225"
}
```

# **实现**

1. 使用Spring AI，实现代码最简化
2. 使用Cloudflare的AI Gateway（后端是Deepseek官方API），方便查看日志

## **工程结构**

```
├── pom.xml
└── src
    └── main
        ├── java
        │   └── mcpdemo
        │       ├── MCPClientDemoApplication.java
        │       └── controller
        │           └── ChatController.java
        └── resources
            ├── application.properties
            └── mcp-servers.json
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
		<version>3.4.5-SNAPSHOT</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>cc.landingzone</groupId>
	<artifactId>mcpdemo</artifactId>
	<version>0.0.1-</version>
	<name>mcpdemo</name>
	<description>web</description>

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
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.ai</groupId>
			<artifactId>spring-ai-openai-spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.ai</groupId>
			<artifactId>spring-ai-mcp-client-spring-boot-starter</artifactId>
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
							<finalName>mcpdemo</finalName>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

</project>
```

## **MCPClientDemoApplication.java**

```java
package mcpdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@SpringBootApplication
@ServletComponentScan("mcpdemo")

public class MCPClientDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MCPClientDemoApplication.class, args);
    }

}
```

## **ChatController.java**

```java
package mcpdemo.controller;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.client.advisor.SimpleLoggerAdvisor;
import org.springframework.ai.tool.ToolCallbackProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ChatController {

    private final ChatClient mcpChatClient;

    @Autowired
    public ChatController(ChatClient.Builder mcpChatClientBuilder,
        ToolCallbackProvider tools) {
        this.mcpChatClient = mcpChatClientBuilder
            .defaultTools(tools)
            .build();
    }

    @GetMapping("/mcpGenerate")
    public String mcpGenerate(@RequestParam(value = "message") String message) {
        String response = mcpChatClient.prompt(message).call().content();
        // 简单处理一下markdown的语法
        String result = "question: " + message + " <hr>"
            + response
                .replaceAll("\\*\\*(.*?)\\*\\*", "<strong>$1</strong>")
                .replaceAll("\n", "<br/>");
        return result;
    }
}
```

## **application.properties**

```properties
spring.application.name=mcp
# 这里可以用任何LLM的配置，用Cloudflare代理只是为了更加方便的查看交互日志
spring.ai.openai.api-key=sk-53dda19ed6ce42********
spring.ai.openai.base-url=https://gateway.ai.cloudflare.com/v1/ed2609b2e7b724226*****/mydeepseek/deepseek
spring.ai.openai.chat.options.model=deepseek-chat
spring.ai.mcp.client.stdio.servers-configuration=classpath:mcp-servers.json
```

## **mcp-servers.json**

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
        "ALIBABA_CLOUD_ACCESS_KEY_ID": "LTAI5tJHDnKc*****",
        "ALIBABA_CLOUD_ACCESS_KEY_SECRET": "U48j3ul1Hio*******"
      }
    }
  }
}
```

# **小彩蛋**

前面的页面有点丑，而且不是流式的。当然可以让AI帮忙修改一下。Deepseek还是一如既往的给力，一句话生成这个页面。

<video controls="" src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/video/demo-mcp-client-v2.mp4" title="Screen-2025-03-30-220734(1).mp4" data-size="12720644"></video>
