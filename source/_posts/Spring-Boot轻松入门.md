---
title: Spring Boot轻松入门
date: 2018-01-07 11:10:59
tags: 
	- 微服务
	- 框架
categories: JAVA
---

{% asset_img  logo.jpg 学习笔记%}

## 一、 SpringBoot概述
### 1. 概述
#####   Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置(**习惯优于配置**)，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot致力于在蓬勃发展的快速应用开发领域（rapid application development）成为领导者。
<!-- more -->
### 2. 特点
- 创建可以独立运行的 Spring 应用。
- 直接嵌入 *Tomcat* 或 *Jetty* 服务器，不需要部署war文件。
- 提供推荐的基础 *POM* 文件来简化 *Apache Maven* 配置。
- 尽可能的根据项目依赖来自动配置 Spring 框架。
- 提供可以直接在生产环境中使用的功能，如性能指标、应用信息和应用健康检查。
- 没有代码生成，也没有XML配置文件（基于注解）。

### 3. 优点
- 快速构建项目
- 对主流开发框架的无配置集成
- 项目可独立运行，无需外部依赖servlet容器
- 提供运行时的应用监控
- 极大提高了开发、部署效率
- 与云计算的天然集成

## 二、开发第一个SpringBoot工程
### 1. 开发环境
- *Spring Boot 1.5.7.RELEASE*
- *JDK1.7+*
- *Spring Framework 4.3.11.RELEASE+*
- *maven3.2.0+*

### 2. 创建SpringBoot工程
- 手动构建
	- 创建空的Maven Project
	- 修改pom.xml
		```
		<!-- 添加spring boot项目的父依赖 -->
		<parent>
			<!-- 提供相关的maven默认依赖 -->
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>1.5.7.RELEASE</version>
		</parent>
		```
		```
		<!-- spring boot Web支持 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
		```
	- 创建入口类
		```
		/**
		 * Spring Boot项目入口类
		 * @author gaozhy
		 */
		// Spring Boot的核心注解 目的开启自动配置
		@SpringBootApplication
		public class Application {
			
			// main方法为项目入口
			public static void main(String[] args) {
				// 启动Spring Boot
				SpringApplication.run(Application.class, args);
			}
		}
		```
- 自动构建
	- 使用sts创建Spring Starter Project
	- 配置项目信息
		![](7.jpg)
	- 创建项目
	- 项目目录
		![](8.jpg)

### 3. 第一个SpringBoot Demo
- 新建RestController
	```
	/**
	 * Created by gaozhy on 2017/10/4.
	 */
	@RestController // 组合注解
	@RequestMapping
	public class HelloController {
	
	    @RequestMapping("/")
	    public String index(){
	        return "Greetings from Spring Boot";
	    }
	}
	```
- 运行启动类|入口类
- 访问：http://localhost:8080/

### 4. 单元测试
	```
	// 通过@RunWith() + @SpringBootTest开启注解
	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class HelloTest {
		
		@Autowired
		private HelloService helloService;
		
		@Test
		public void testHello(){
			System.out.println(helloService.getHello());
		}
	}
	```

> #### Note:
> 1. 无需做任何的web.xml配置
> 2. 无需做任何的sping mvc的配置; springboot为你做了
> 3. 无需配置tomcat，springboot内嵌tomcat
> 4. 启动项目（maven命令spring-boot:run、jar -jar xxx.jar、运行入口类方法）

## 三、Spring Boot配置文件详解
> Spring Boot采纳了建立生产就绪Spring应用程序的观点。 Spring Boot优先于配置的惯例，旨在让您尽快启动和运行。在一般情况下，我们不需要做太多的配置就能够让Spring Boot正常运行。在一些特殊的情况下，我们需要做修改一些配置，或者需要有自己的配置属性。

### 1. Spring Boot的配置文件application.properties或application.yml
- application.properties	
	```
	server.port=8989
	server.context-path=/helloboot
	```
- application.yml
	```
	server:
	  port: 8989
	  context-path: /helloboot
	```

### 2. 常规属性配置
- 在application.properties文件中定义一组属性：
	```
	id=1
	name=\u5C0F\u7EA2
	```
- 在需要读取配置文件值的属性上使用@Value("${名字}")
	```
	@Value("${id}")
	private Integer id;
	@Value("${name}")
	private String name;
	```
- 测试
	
### 3. 将配置文件中的属性赋给实体类
- 通过@ConfigurationProperties将properties属性和一个Bean及其属性关联
- 在application.properties文件中定义对象的一组属性
	```
	my.id=100
	my.name=xiaohei
	my.salary=1000
	my.uuid=${random.uuid}
	# 10-20的随机数
	my.number=${random.int[10,20]}
	```
- 实体ConfigBean
	```
	@ConfigurationProperties(prefix="my")
	public class ConfigBean {
		private Integer id;
		private String name;
		private Double salary;
		private String uuid;
		private Integer number;
	```
- 启动测试
	```
	//入口类添加注解
	@EnableConfigurationProperties(ConfigBean.class)
	```
### 4. 多个环境配置文件
> 在现实的开发环境中，我们需要不同的配置环境；格式为application-{profile}.properties，其中{profile}对应你的环境标识，比如：
> - application-test.properties：测试环境
> - application-dev.properties：开发环境
> - application-prod.properties：生产环境

- 在application.yml中添加配置
	```
	spring.profiles.active=dev
	```
- 准备application-dev.yml
	```
	server:
		port: 8085
	```
- 启动测试

## 四、SpringBoot整合JPA
> **JPA**全称*Java Persistence API*；JPA通过JDK 5.0注解或XML描述对象和关系表的映射关系，并将运行期的实体对象持久化到数据库中
> **JPA**是需要Provider来实现其功能的，Hibernate就是JPA Provider

### 1. pom.xml中添加jpa依赖
	```
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-jpa</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>

	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<scope>runtime</scope>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
	<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>druid</artifactId>
		<version>1.1.2</version>
	</dependency>
	```
### 2. application.yml中配置数据源
	```
	spring:
	  datasource:
	    username: root
	    password: root
	    url: jdbc:mysql://localhost:3306/springboot?useUnicode=true&characterEncoding=utf-8
	    driver-class-name: com.mysql.jdbc.Driver
	    type: com.alibaba.druid.pool.DruidDataSource
	  jpa:
	    hibernate:
	      ddl-auto: update
	    show-sql: true
	```
### 3. 创建实体类
	```
	@Table(name="t_user")
	@Entity
	public class User {
	
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name="u_id")
	    private Integer id;
	    private String name;
	    private Double salary;
	
	    @Temporal(value=TemporalType.DATE)
	    private Date birthday;
	
	    public Integer getId() {
	        return id;
	    }
	
	    public void setId(Integer id) {
	        this.id = id;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public Double getSalary() {
	        return salary;
	    }
	
	    public void setSalary(Double salary) {
	        this.salary = salary;
	    }
	
	    public Date getBirthday() {
	        return birthday;
	    }
	
	    public void setBirthday(Date birthday) {
	        this.birthday = birthday;
	    }
	
	    @Override
	    public String toString() {
	        return "User{" +
	                "id=" + id +
	                ", name='" + name + '\'' +
	                ", salary=" + salary +
	                ", birthday=" + birthday +
	                '}';
	    }
	}
	```

> #### Note:
> @Table 表信息
> @Entity 表明类为实体或表
> @Id 代表标示属性 为数据库主键
> @GeneratedValue 指定如何标识属性可以被初始化，例如自动，手动，或从序列表中获得的值。
> @Column 指定持久属性栏属性
> @Temporal 日期类型

### 4. 创建DAO
```
public interface UserRepository extends JpaRepository<User,Integer> {

}
```

> **JpaRepository**接口封装了常用的增删改成方法，如果不满足需求，可以在DAO接口声明自己的方法

### 5. 创建Service

```
@Service
public class UserServiceImpl implements UserService{

    @Autowired
    private UserRepository userRepository;

    @Override
    public User queryById(Integer id) {
        User user = userRepository.findOne(id);
        System.out.println(user);
        return user;
    }

    @Override
    public List<User> queryAll() {
        return userRepository.findAll();
    }

    @Override
    @Transactional
    public void modifyById(User user) {
        userRepository.save(user);
    }

    @Override
    @Transactional
    public void removeById(Integer id) {
        userRepository.delete(id);
    }

    @Override
    @Transactional
    public void addUser(User user) {
        userRepository.save(user);
        //int i = 1/0;
    }
}
```

### 6. 创建Controller，并构建restful api测试数据访问
```
@RestController
@RequestMapping
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(value="/add",method = RequestMethod.PUT)
    public void add(User user){
        userService.addUser(user);
    }

    @RequestMapping(value="/update",method = RequestMethod.POST)
    public void update(User user){
        userService.modifyById(user);
    }

    @RequestMapping(value="/delete/{id}",method = RequestMethod.DELETE)
    public void delete(@PathVariable("id") Integer id){
        userService.removeById(id);
    }

    @RequestMapping(value="/findOne/{id}",method = RequestMethod.GET)
    public User findOne(@PathVariable("id") Integer id){
        return userService.queryById(id);
    }

    @RequestMapping(value="/findAll",method = RequestMethod.GET)
    public List<User> findAll(){
        return userService.queryAll();
    }
}
```
### 7. 利用postman请求测试

## 五、SpringBoot整合MyBaits
### 1. pom.xml中添加mybatis依赖
```
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.2</version>
</dependency>
```

### 2. application.yml中配置数据源
```
# ===============数据源相关配置=====================
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/springboot?useUnicode=true&characterEncoding=utf-8
    # 连接池
    type: com.alibaba.druid.pool.DruidDataSource

# ===============MyBatis相关配置===================
mybatis:
  type-aliases-package: com.baizhi.entity
  mapper-locations: classpath:com/baizhi/mapper/*Mapper.xml
```

### 3. 创建DAO接口
>DAO接口需要使用注解@Mapper

```
// @Mapper 表示 映射器的标记接口
@Mapper
public interface UserDAO {

    public List<User> findAll();

    public User findOne(Integer id);

    public void insert(User user);

    public void delete(Integer id);

    public void update(User user);
}
```
### 4. 创建Mapper映射文件
> 无变化

### 5. 创建Service
> 无变化

### 6. 创建Controller，并构建restful api测试数据访问
> 无变化

## 六、SpringBoot整合Redis
### 1. pom.xml中添加redis依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
### 2. application.yml中配置数据源
```
spring:
  redis:
    host: 192.168.128.130
    port: 6379
	# redis server 有访问密码需配置此项
    # password:
    database: 0
    pool:
      max-active: 8
      max-wait: -1
      max-idle: 500
      min-idle: 0
    timeout: 3000
```
### 3. 创建RedisDAO
```
@Repository
public class RedisDAOImpl implements RedisDAO{

    @Autowired
    private RedisTemplate redisTemplate;

    public void setKeyValue(String key,String value){
        redisTemplate.opsForValue().set(key,value);
    }

    public String getValue(String key){
        return (String) redisTemplate.opsForValue().get(key);
    }

    public void setObject(String key,Object obj){
        redisTemplate.opsForValue().set(key,obj);
    }

    public Object getObject(String key){
        return redisTemplate.opsForValue().get(key);
    }

    public void delete(String key) {
        redisTemplate.delete(key);
    }

}
```
### 4. 单元测试
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootRedisApplicationTests {

	@Autowired
	private RedisDAO redisDAO;

	@Test
	public void testRedis() {
//		redisDAO.setKeyValue("username","王自健");
//		System.out.println(redisDAO.getValue("username"));
		redisDAO.delete("username");
	}

}
```

## 七、SpringBoot整合Fastjson
### 1. pom.xml中添加Fastjson依赖
```
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>fastjson</artifactId>
	<version>1.2.31</version>
</dependency>
```

### 2. 第一种方案
```
package com.baizhi.config;

import java.util.ArrayList;
import java.util.List;

import org.springframework.context.annotation.Configuration;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

import com.alibaba.fastjson.serializer.SerializerFeature;
import com.alibaba.fastjson.support.config.FastJsonConfig;
import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;

/**
 * 
 * @author gaozhy
 * @description 使用fastjson替换Spring Boot默认的jackson
 */

@Configuration // 此类为配置类
public class WebMvcConfig extends WebMvcConfigurerAdapter{
	
	@Override
	public void configureMessageConverters(
			List<HttpMessageConverter<?>> converters) {
		// 1. 创建json转换器
		FastJsonHttpMessageConverter messageConverter = new FastJsonHttpMessageConverter();
		// 2. 处理中文乱码
		ArrayList<MediaType> mediaTypes = new ArrayList<MediaType>();
		mediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
		messageConverter.setSupportedMediaTypes(mediaTypes);
		
		// 3. 配置转换器
		FastJsonConfig config = new FastJsonConfig();
		config.setSerializerFeatures(SerializerFeature.PrettyFormat); // 优雅的json格式
		messageConverter.setFastJsonConfig(config);
		
		// 4. 添加转换器
		converters.add(messageConverter);
	}
}
```

### 2. 第二种方案
```
/**
 * @description 通过@Bean注解声明，创建转换器，为其注册fastjson转换器，用以替换jackson
 * @return 表示返回值为一个Bean对象
 */
@Bean
public HttpMessageConverters messageConverters(){
	
	// 1. 创建json转换器
	FastJsonHttpMessageConverter messageConverter = new FastJsonHttpMessageConverter();
	// 2. 处理中文乱码
	List<MediaType> mediaTypes = new ArrayList<MediaType>();
	
	mediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
	
	messageConverter.setSupportedMediaTypes(mediaTypes);
	// 3. 配置转换器
	FastJsonConfig config = new FastJsonConfig();
	// 对响应的json进行格式化处理
	config.setSerializerFeatures(SerializerFeature.PrettyFormat);
	
	messageConverter.setFastJsonConfig(config);
	// 4. 添加转换器
	return new HttpMessageConverters(messageConverter);
}
```

### 3. 测试
- 实体属性加fastjson注解
	```
	@JSONField(format="yyyy-MM-dd HH:mm:ss")
	private Date birthday;
	```
- 测试结果
	![](9.jpg)

## 八、SpringBoot配置devtools实现热部署
### 1. pom.xml中添加devtools依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```
### 2. 测试

> #### Note:
> 由于热部署是监听 Class 文件的变化，它自身不会主动去编译 Java 文件，所以我们得在 Java 文件改动时，自动编译成 Class 文件，然后热部署工具创造的新的类加载器才会加载改变后的 Class文件。所以，如果你使用 IDEA 开发工具的话，记得要把自动编译打开

## 九、SpringBoot中整合JSP
### 1. pom.xml中添加依赖
```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <!--<scope>provided</scope>-->
</dependency>
```

### 2. application.yml中配置
```
spring.mvc.view.prefix= /WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

### 3. 创建jsp文件存放目录src/main/webApp/WEB-INF/views/
![](2017-10-05_230438.jpg)

### 4. 创建Controller
```
/**
 * Created by gaozhy on 2017/10/5.
 */
@Controller
public class HelloController {

    @RequestMapping("/")
    public String index(){
        return "index";
    }

    @RequestMapping("/hello")
    public ModelAndView hello(ModelAndView modelAndView){
        modelAndView.setViewName("index1");
        modelAndView.addObject("username","zhangsan");
        return modelAndView;
    }
}
```

### 5. 使用maven命令启动测试
```
mvn springboot:run
```
![](2017-10-05_230902.jpg)

> **NOTE:**
> ###### SpringBoot不推荐使用JSP作为View，而是推荐我们使用模板（如：thymeleaf、freemarker等模板引擎），原因如下：
> 1. JSP性能较差
> 2. 绝对的前后端分离思想，JSP并不利于页面调试（*运行依赖于web容器*）
> 3. SpringBoot对内嵌web容器的支持默认也是用tomcat。但tomcat对web资源的处理上写死了要使用文件目录，对于打包成jar包的SpringBoot应用来说，显然不行，也有的人打包成war，然后还是部署到tomcat上，这样违反了SpringBoot的初衷，这样一来，等于否定了嵌入式容器，而且程序员还要处理嵌入式环境和外部tomcat环境的不同带来的问题。
