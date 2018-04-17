---
title: SpringBoot系列——集成Dubbo
date: 2018-04-12 11:22:49
tags: 
	- 微服务
	- 框架
categories: JAVA
---
**更新**：注册中心使用 `Zookeeper`（示例代码使用的Spring Boot版本为：1.5.8.RELEASE）

参考文档：[https://github.com/apache/incubator-dubbo-spring-boot-project](https://github.com/apache/incubator-dubbo-spring-boot-project)

{% asset_img  logo.jpg dubbo%}  
Apache Dubbo(incubating) Spring Boot Project makes it easy to create Spring Boot application using Dubbo as RPC Framework. What's more, it aslo provides

- auto-configure features (e.g., annotation-driven, auto configuration, externalized configuration).
- production-ready features (e.g., security, health checks, externalized configuration).
> Apache Dubbo(incubating) is a high-performance, java based RPC framework open-sourced by Alibaba. As in many RPC systems, dubbo is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. On the server side, the server implements this interface and runs a dubbo server to handle client calls. On the client side, the client has a stub that provides the same methods as the server.

<!-- more -->

- **创建工程并导入Maven依赖（springboot_dubbo_provider、springboot_dubbo_consumer）**
``` xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>

	<dependency>
		<groupId>com.alibaba.boot</groupId>
		<artifactId>dubbo-spring-boot-starter</artifactId>
		<version>0.1.0</version>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-actuator</artifactId>
	</dependency>

	<dependency>
		<groupId>com.101tec</groupId>
		<artifactId>zkclient</artifactId>
		<version>0.8</version>
	</dependency>
</dependencies>
```
> 注：dubbo-spring-boot-starter-0.1.0 只支持1.x.x的spring boot版本

- **springboot_dubbo_provider(服务提供方)**
提供服务接口和服务实现，并将服务注册到注册中心
	- 服务接口
	``` java
	package com.baizhi.service;

	/**
	 * @author gaozhy
	 * @date 2018/4/11.17:49
	 */
	public interface DemoService {

	    String sayHello(String name);

	}
	```
	- 服务实现
	``` java
	package com.baizhi.service;

	import com.alibaba.dubbo.config.annotation.Service;

	/**
	 * @author gaozhy
	 * @date 2018/4/11.17:49
	 */
	@Service(
	        version = "1.0.0",
	        application = "${dubbo.application.id}",
	        protocol = "${dubbo.protocol.id}",
	        registry = "${dubbo.registry.id}"
	)
	public class DemoServiceImpl implements DemoService{

	    @Override
	    public String sayHello(String name) {
	        return "Hello, " + name + " (from Spring Boot)";
	    }
	}
	```
	> 注：@Service 是 `<dubbo:service>` 的替代注解，用于服务提供方 Dubbo 服务暴露

	- 配置文件
	```
	# Spring boot application
	spring.application.name = dubbo-provider-demo
	server.port = 9090
	management.port = 9091

	# Base packages to scan Dubbo Components (e.g., @Service, @Reference)
	dubbo.scan.basePackages  = com.baizhi.service

	# Dubbo Config properties
	## ApplicationConfig Bean
	dubbo.application.id = dubbo-provider-demo
	dubbo.application.name = dubbo-provider-demo

	## ProtocolConfig Bean
	dubbo.protocol.id = dubbo
	dubbo.protocol.name = dubbo
	dubbo.protocol.port = 20880

	## RegistryConfig Bean
	# dubbo.registry.id = my-registry
	# dubbo.registry.address = N/A

	#================ zk registry====================
	dubbo.registry.id = zkRegistry
	dubbo.registry.address = 192.168.128.137
	dubbo.registry.port = 2181
	dubbo.registry.protocol = zookeeper
	```

- **springboot_dubbo_consumer**
服务消费方 调用消费服务提供方的服务
	- 服务接口 （相同）
	- 服务消费
	``` java
	package com.baizhi.controller;

	import com.alibaba.dubbo.config.annotation.Reference;
	import com.baizhi.service.DemoService;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.RestController;

	/**
	 * @author gaozhy
	 * @date 2018/4/12.10:03
	 */
	@RestController
	public class DemoConsumerController {

	    @Reference(version = "1.0.0",
	            application = "${dubbo.application.id}",
	            registry = "${dubbo.registry.id}")
	    private DemoService demoService;

	    @RequestMapping("/sayHello")
	    public String sayHello(@RequestParam String name) {
	        return demoService.sayHello(name);
	    }
	}
	```
	> 注意：@Reference 是`<dubbo:reference>`的替代注解，用于服务的消费引用
	
	- 启动入口类
	``` java
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;

	@SpringBootApplication(scanBasePackages = "com.baizhi.controller")
	public class SpringbootDubboConsumerApplication {

	   public static void main(String[] args) {
	      SpringApplication.run(SpringbootDubboConsumerApplication.class, args);
	   }
	}
	```

	- 配置文件
	```
	# Spring boot application
	spring.application.name = dubbo-consumer-demo
	server.port = 8080
	management.port = 8081


	# Dubbo Config properties
	## ApplicationConfig Bean
	dubbo.application.id = dubbo-consumer-demo
	dubbo.application.name = dubbo-consumer-demo

	## ProtocolConfig Bean
	dubbo.protocol.id = dubbo
	dubbo.protocol.name = dubbo
	dubbo.protocol.port = 20880

	#================ zk registry====================
	dubbo.registry.id = zkRegistry
	dubbo.registry.address = 192.168.128.137
	dubbo.registry.port = 2181
	dubbo.registry.protocol = zookeeper
	```

- 分别启动 测试
	- Provider Log
	![](2018-04-12_110326.jpg)

	- Consumer Log
	![](2018-04-12_110759.jpg)

	- 测试：
	Consumer Service：[http://localhost:8080/sayHello?name=zs](http://localhost:8080/sayHello?name=zs)
	![](2018-04-12_110949.jpg)
	Health Checks: http://localhost:8081/health
	Dubbo Endpoint: http://localhost:8081/dubbo
	 