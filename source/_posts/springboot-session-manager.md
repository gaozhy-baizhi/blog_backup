---
title: SpringBoot系列——如何解决SpringBoot应用分布式会话问题
date: 2018-04-03 14:54:58
tags: 
	- 微服务
	- 框架
categories: JAVA
toc: true
---

*SpringBoot快速上手教程请参阅博主之前的教程：[Spring-Boot轻松入门](http://www.gaozhy.cn/blog/2018/01/07/Spring-Boot%E8%BD%BB%E6%9D%BE%E5%85%A5%E9%97%A8/)*

**在JavaWeb开发中，我们知道服务器使用HttpSession记录客户端信息。而HttpSession由容器创建、销毁（隔离），如果我们将Web应用水平扩展搭建成分布式的集群，然后利用LVS或Nginx做负载均衡，那么来自同一用户的Http请求将有可能被负载分发到两个不同的实例中去，如何保证不同实例间Session共享成为一个不得不解决的问题**

{% asset_img  分布式应用.png 分布式应用%}   
**解决方案**：
<!-- more -->
- 利用Servlet容器提供的插件功能，自定义HttpSession的创建和管理策略，并通过配置的方式替换掉默认的策略。不过这种方式有个缺点，就是需要耦合Tomcat/Jetty等Servlet容器的代码。这方面其实早就有开源项目了，例如memcached-session-manager，以及tomcat-redis-session-manager。暂时都只支持Tomcat6/Tomcat7。
- 配置Nginx的负载均衡算法为ip_hash，这样每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的Session共享问题
- 如果你使用Shiro管理Session，可以用Redis来实现Shiro 的SessionDao接口，这样Session便归Redis保管。
- 设计一个Filter，利用HttpServletRequestWrapper，实现自己的 getSession()方法，接管创建和管理Session数据的工作。Spring-Session就是通过这样的思路实现的。 

> **Note**：
> 1. SpringBoot应用打包方式为jar包，使用内嵌Tomcat无法集成`*-session-manager`等插件
> 2. ip_hash 固定IP的这种策略，需要考虑故障转移

### SpringBoot应用分布式会话解决方案:
参考资料：https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-redis.html

##### 测试步骤
- 创建两个SpringBoot工程  
![创建工程](创建工程.png) 

- 更新Maven依赖
  ``` xml
  <parent>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-parent</artifactId>
  	<version>1.5.10.RELEASE</version>
  	<relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
  	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  	<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  	<java.version>1.8</java.version>
  </properties>

  <dependencies>
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-web</artifactId>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework.session</groupId>
  		<artifactId>spring-session</artifactId>
  	</dependency>

  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-test</artifactId>
  		<scope>test</scope>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-data-redis</artifactId>
  	</dependency>
  </dependencies>
  ```
- 修改配置文件`application.properties`
  ```
  spring.session.store-type=redis
  server.session.timeout=1500
  # 立即保存
  spring.session.redis.flush-mode=immediate
  spring.session.redis.namespace=baizhi

  spring.redis.host=192.168.128.129
  spring.redis.port=6379
  server.port=8081
  ```
- 测试代码
  - `springboot_tomcat1`项目
    ``` java
    package com.baizhi;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    import javax.servlet.http.HttpSession;

    @SpringBootApplication
    @RestController
    public class SpringbootTomcat1Application {
    	private static final Logger logger = LoggerFactory.getLogger(SpringbootTomcat1Application.class);
      
    	@RequestMapping("/")
    	public String index(HttpSession session){
    		logger.info("tomcat1 session {}: ",session);
    		logger.info("tomcat1 session id is {}: ",session.getId());

    		session.setAttribute("name","张三");

    		return "this is tomcat1";
    	}

    	public static void main(String[] args) {
    		SpringApplication.run(SpringbootTomcat1Application.class, args);
    	}
    }

    ```
  - `springboot_tomcat2`项目
    ``` java
    package com.baizhi;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    import javax.servlet.http.HttpSession;

    @SpringBootApplication
    @RestController
    public class SpringbootTomcat2Application {

    	private static final Logger logger = LoggerFactory.getLogger(SpringbootTomcat2Application.class);

    	@RequestMapping("/")
    	public String index(HttpSession session){
    		logger.info("tomcat2 session: {}",session);
    		logger.info("tomcat2 session is is : {}",session.getId());
            String name = (String) session.getAttribute("name");
            logger.info("name is {}",name);
            return "this is tomcat2";
    	}

    	public static void main(String[] args) {
    		SpringApplication.run(SpringbootTomcat2Application.class, args);
    	}
    }
    ```
- 运行测试
    - 先访问 http://localhost:8081/
    - 再访问 http://localhost:8082/
- 测试结果
  - 日志
  ![测试结果](测试结果.png) 
  
  - Redis
  ![测试结果1](测试结果1.png) 

  
