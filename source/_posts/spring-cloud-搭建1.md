---
title: spring cloud 搭建-1
date: 2017-05-19 10:20:30
tags: 
	- spring
	- spirng cloud
	- spring boot
	- mircro-service
toc: true
---
# Spring Cloud 基础

## 创建基础结构包

在 spring 官网提供了一套初始化程序，可以用于组件的初步加载和 maven (gradle) 的创建过程[链接地址](https://start.spring.io/)

创建之后，应该可以得到如下的 pom.xml 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>edu.uestc.msstudio.supermt.cloud</groupId>
	<artifactId>spring-cloud-study-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>

	<name>spring-cloud-study-parent</name>
	<description>Learning Project for Spring Cloud System</description>

	<modules>
		<module>discovery-eureka</module>
	</modules>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.3.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>


	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Dalston.RELEASE</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.kafka</groupId>
			<artifactId>spring-kafka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>

```

在上述步骤之后，得到的内容包中会含有一个启动文件，需要注意的是，parent 包只是一个用于个其他子模块提供内容约束服务的包，其中不应该包含逻辑代码或者是启动文件。所以，需要将启动文件删除。

此外，还要将

```xml
	<groupId>edu.uestc.msstudio.supermt.cloud</groupId>
	<artifactId>spring-cloud-study-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>
```

改为

```xml
	<groupId>edu.uestc.msstudio.supermt.cloud</groupId>
	<artifactId>spring-cloud-study-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>
```

这是因为该包作为 parent 包，不应该被打成任何形式的部署包。

## 添加子模块

在父模块定义完成后，可以开始子模块的创建，我们需要创建的第一个子模块就是 Spring Cloud 中的服务发现模块，即 Eureka 服务器模块。

子模块的创建形式可以根据实际情况做改变

- 如果项目文件完全公开，可以考虑直接将所有内容放置在一个文件夹和版本控制仓库内，这样 Maven 在做部署的时候可以直接从内含文件中提取最新的更新内容。
- 如果项目文件需要保密，同时您有足够的能力搭建自己的本地 Maven 库（如果能放置到 Maven Repository 上也是一样的） ， 那么您可以将所有文件放置在不同的文件夹和版本控制仓库内。需要注意的是，如果您不能保证您的每一个子模块在内容更新后能够尽快在构建仓库中更新的话，很容易对并行的开发过程造成一系列的困扰。

简单期间，[本次示例](https://github.com/supermt/spring-cloud-example)仅使用第二种方式

初始化的方式既可以使用[ maven 的 generate 命令]()http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html，也可以继续使用 [Spring Start](https://start.spring.io/)

为了尽量兼容 Eclipse 编程环境中的 STS (Spinrg Tool Suites) ，我们使用第二种方式作为创建方法。由于大部分内容都已经在 parent 中描述了，我们仅仅需要下载并解压就行，因为剩余内容应该由我们自行配制。

1. 需要修改的第一步是将原本的 parent 包从 spring boot 设置为刚才的 parent 包

```xml

<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.5.3.RELEASE</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>

```

改为

```xml

<parent>
	<groupId>edu.uestc.msstudio.supermt.cloud</groupId>
	<artifactId>spring-cloud-study-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
</parent>

```

2. 将包名、制品名等替换成 parent 中规定的内容

```xml
<groupId>com.example</groupId>
<artifactId>demo</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>jar</packaging>
```

改为

```xml
<artifactId>discovery-eureka</artifactId>
<packaging>jar</packaging>
```

为了简单期间，我们选择所有的子组件都以 jar 的形式独立部署，这样在后期接入 docker 之后，我们不需要预安装过多的组件如 tomcat 等 java web 容器。如果希望使用独立的容器进行管理可以选择打包成 war 包

3. 依赖

由于在 parent 中已经提供了大量的基础依赖，在子项目中只需要添加独有依赖或者需要被重载的内容便可，在这个子项目中，添加 Eureka 作为依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

4. 编译测试

之后在 parent 下面使用 `mvn clean packge` 命令进行测试，应该可以得到成功执行的内容

> 如果出现一些测试错误导致无法编译，请删除所有测试用例，因为目前没有添加启动文件，无法正常启动和测试。
>> 测试用例往往统一放置在 `test` 文件夹下,递归删除即可

## 修改启动文件和配置文件

创建一个启动，如本次项目中创建的就是 `edu.uestc.msstudio.supermt.cloud.context.EurekaServer`。在类定义上添加 @SpringBootApplication 和 @EnableEurekaServer。之后在本地启动并测试，就能看到 Eureka 的监视器界面了。

```java
package edu.uestc.msstudio.supermt.cloud.context;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServer
{

    public static void main(String[] args)
    {
        SpringApplication.run(EurekaServer.class, args);
    }
    
}

```

但是，为了能够保证各个服务能够正常注册，我们需要规定一个统一的服务发现与注册接口，所以，我们需要修改一些配置文件

```yaml
# Spring Service Configuration
server:
  port: 8000
spring:
  application:
    name: data-service

# Spring MVC Configuration
spring:
  datasource:
    username: root
    password: MysqlRoot
    url: jdbc:mysql://localhost:3306/db_sxf?createDatabaseIfNotExist=true&useUnicode=true&characterEncoding=UTF-8    
spring:
  jpa: 
    hibernate:
      ddl-auto: create-drop

# Eureka Client Configuration

eureka: 
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```