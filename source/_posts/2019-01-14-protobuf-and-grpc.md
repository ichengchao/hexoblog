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

option java_package = "name.chengchao.myprotobuf.proto";
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
import java.io.FileOutputStream;

import name.chengchao.myprotobuf.proto.StudentProtos.Student;
import name.chengchao.myprotobuf.proto.StudentProtos.Student.Gender;

public class Test {

    public static void testSerialize() throws Exception {

        Student student = Student.newBuilder().setAge(10).setName("nina").setGendar(Gender.GIRL).build();

        student.toBuilder().build().writeTo(new FileOutputStream(new File("/Users/charles/Desktop/protobuf_student")));

        System.out.println(student);

        System.out.println("==========copy============");
        Student studentCopy =
            Student.parseFrom(new FileInputStream(new File("/Users/charles/Desktop/protobuf_student")));
        System.out.println(studentCopy);

    }

    public static void main(String[] args) throws Exception {
        testSerialize();
    }

}

```



### hello grpc

> 编辑一下student.proto的内容,增加service

```proto
syntax = "proto3";
package chengchao.prototest;

option java_package = "name.chengchao.myprotobuf.proto";
option java_outer_classname = "StudentProtos";


service HelloStudent {
    rpc SayHello (Student) returns (Student) {}
  }

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

>修改一下pom.xml

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>name.chengchao</groupId>
	<artifactId>myprotobuf</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<java.version>1.8</java.version>
		<grpc.version>1.17.0</grpc.version>
		<protobuf.version>3.5.1</protobuf.version>
		<protoc.version>3.5.1-1</protoc.version>
	</properties>

	<dependencies>

		<!--grpc -->
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-netty-shaded</artifactId>
			<version>${grpc.version}</version>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-protobuf</artifactId>
			<version>${grpc.version}</version>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-stub</artifactId>
			<version>${grpc.version}</version>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-alts</artifactId>
			<version>${grpc.version}</version>
		</dependency>
		<dependency>
			<groupId>javax.annotation</groupId>
			<artifactId>javax.annotation-api</artifactId>
			<version>1.2</version>
			<scope>provided</scope> <!-- not needed at runtime -->
		</dependency>

		<!-- <dependency> <groupId>com.google.protobuf</groupId> <artifactId>protobuf-java</artifactId> 
			<version>3.6.1</version> </dependency> -->


	</dependencies>
	<build>

		<extensions>
			<extension>
				<groupId>kr.motd.maven</groupId>
				<artifactId>os-maven-plugin</artifactId>
				<version>1.5.0.Final</version>
			</extension>
		</extensions>

		<finalName>myprotobuf</finalName>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.3.2</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>utf-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-release-plugin</artifactId>
				<version>2.0</version>
			</plugin>


			<plugin>
				<groupId>org.xolstice.maven.plugins</groupId>
				<artifactId>protobuf-maven-plugin</artifactId>
				<version>0.5.1</version>
				<configuration>
					<protocArtifact>com.google.protobuf:protoc:${protoc.version}:exe:${os.detected.classifier}</protocArtifact>
					<pluginId>grpc-java</pluginId>
					<pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}</pluginArtifact>
				</configuration>
				<executions>
					<execution>
						<goals>
							<goal>compile</goal>
							<goal>compile-custom</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>

```

```
# 生成对应的java代码,分别是HelloStudentGrpc.java和StudentProtos.java
mvn clean compile
```
参照官方示例写一个Server和Client

```java
package name.chengchao.myprotobuf;

import java.io.IOException;
import java.util.logging.Logger;

import io.grpc.Server;
import io.grpc.ServerBuilder;
import io.grpc.stub.StreamObserver;
import name.chengchao.myprotobuf.proto.HelloStudentGrpc.HelloStudentImplBase;
import name.chengchao.myprotobuf.proto.StudentProtos.Student;

public class HelloServer {

	private static final Logger logger = Logger.getLogger(HelloServer.class.getName());

	private Server server;

	private void start() throws IOException {
		/* The port on which the server should run */
		int port = 50051;
		server = ServerBuilder.forPort(port).addService(new HelloStudentImpl()).build().start();
		logger.info("Server started, listening on " + port);
		Runtime.getRuntime().addShutdownHook(new Thread() {
			@Override
			public void run() {
				// Use stderr here since the logger may have been reset by its JVM shutdown
				// hook.
				System.err.println("*** shutting down gRPC server since JVM is shutting down");
				HelloServer.this.stop();
				System.err.println("*** server shut down");
			}
		});
	}

	private void stop() {
		if (server != null) {
			server.shutdown();
		}
	}

	/**
	 * Await termination on the main thread since the grpc library uses daemon
	 * threads.
	 */
	private void blockUntilShutdown() throws InterruptedException {
		if (server != null) {
			server.awaitTermination();
		}
	}

	/**
	 * Main launches the server from the command line.
	 */
	public static void main(String[] args) throws IOException, InterruptedException {
		final HelloServer server = new HelloServer();
		server.start();
		server.blockUntilShutdown();
	}

	static class HelloStudentImpl extends HelloStudentImplBase {

		@Override
		public void sayHello(Student student, StreamObserver<Student> responseObserver) {

			Student reply = student.toBuilder().setAge(20).build();
			responseObserver.onNext(reply);
			responseObserver.onCompleted();
		}

	}

}

```

```java

package name.chengchao.myprotobuf;

import java.util.concurrent.TimeUnit;
import java.util.logging.Level;
import java.util.logging.Logger;

import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;
import io.grpc.StatusRuntimeException;
import name.chengchao.myprotobuf.proto.HelloStudentGrpc;
import name.chengchao.myprotobuf.proto.StudentProtos.Student;
import name.chengchao.myprotobuf.proto.StudentProtos.Student.Gender;

public class HelloClient {

	private static final Logger logger = Logger.getLogger(HelloClient.class.getName());

	private final ManagedChannel channel;
	private final HelloStudentGrpc.HelloStudentBlockingStub blockingStub;

	/** Construct client connecting to HelloWorld server at {@code host:port}. */
	public HelloClient(String host, int port) {
		this(ManagedChannelBuilder.forAddress(host, port).usePlaintext().build());
	}

	/**
	 * Construct client for accessing HelloWorld server using the existing channel.
	 */
	HelloClient(ManagedChannel channel) {
		this.channel = channel;
		blockingStub = HelloStudentGrpc.newBlockingStub(channel);
	}

	public void shutdown() throws InterruptedException {
		channel.shutdown().awaitTermination(5, TimeUnit.SECONDS);
	}

	/** Say hello to server. */
	public void sayHello(String name) {
		logger.info("Will try to hello " + name + " ...");
		Student request = Student.newBuilder().setName(name).setAge(10).setGendar(Gender.GIRL).build();
		Student response;
		try {
			response = blockingStub.sayHello(request);
		} catch (StatusRuntimeException e) {
			logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
			return;
		}
		logger.info("==============");
		logger.info("hello response: " + response);
	}

	/**
	 * Greet server. If provided, the first element of {@code args} is the name to
	 * use in the greeting.
	 */
	public static void main(String[] args) throws Exception {
		HelloClient client = new HelloClient("localhost", 50051);
		try {
			/* Access a service running on the local machine on port 50051 */
			client.sayHello("nina");
		} finally {
			client.shutdown();
		}
	}
}

```

先运行server,在运行client就能测试一下rpc的基本用法了.