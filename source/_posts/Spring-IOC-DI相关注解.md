---
title: Spring4.0常用注解——IOC/DI相关注解
date: 2018-04-10 09:13:38
tags:
---

- **`@Configuration`**

该注解等价于applicationContext.xml配置
``` java
@Configuration
public class ApplicationConfig {
  //...
}
```
使用程序实现加载工厂
``` java
AnnotationConfigApplicationContext ctx= new AnnotationConfigApplicationContext(ApplicationConfig.class);
```
<!-- more -->
老版本编写
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```
使用程序实现加载工厂
``` java
ApplicationContext ctx=new ClassPathXmlApplicationContext("applicationContext.xml");
```

- **`@Bean`**
该注解等价于 <bean />标签
``` java
@Bean(name="date")
public Date getDate() {
		return new Date();
}
```
等价
``` xml
<bean id="date" class="java.util.Date"></bean>
```
@Bean属性介绍

|  属性名   	  | 类型		  | 是否必须  | 说明 	 									  |
| --------------- | --------------| --------  |---------------------------------------------  |
| name       	  | String 		  | 否		  |等价于<bean /> 的id属性，不可以和value同时出现 |
| value           | String        | 否		  |等价于<bean /> 的id属性，不可以和name同时出现  |
| autowire        | Bean   	   	  | 否		  |自动注入方式Autowire.NO/BY_NAME/BY_TYPE		  |
| destroyMethod   | String        | 否	      |等价于<bean /> 的destroy-method属性			  |
| init-method     | String 		  | 否  	  |等价于<bean /> 的init-method属性				  |

- **`@Component`、`@Service`、`@Repository`、`@Controller`**
以上注解主要面向分层开发时候需要对不同层次的Bean分类管理。
``` java
@Service("userService")
public class UserService implements IUserService{

}
```

- **`@ComponentScan`**
该注解用于扫描指定spring容器的bean，并且将扫描的bean放置到Spring工厂中
``` java
@Configuration
@ComponentScan(basePackages="com.baizhi",
  excludeFilters= {
		@Filter(type=FilterType.ANNOTATION,value=Controller.class)
  },
  includeFilters= {
		  @Filter(Service.class),
		  @Filter(Repository.class)
  }
)
public class ApplicationConfig {
	//...
}
```
等价写法
``` xml
<context:component-scan base-package="com.baizhi">
    <context:include-filter 
               type="annotation"
                expression="org.springframework.stereotype.Repository,
                org.springframework.stereotype.Service"/>
    <context:exclude-filter 
         		type="annotation"
                expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

- **Bean生命周期控制`@PreDestroy`、`@PostConstruct`、`@Scope`**
``` java
@Component("otherComponent")
@Scope(value=BeanDefinition.SCOPE_PROTOTYPE|SCOPE_SINGLETON)
public class OtherComponent {
	public OtherComponent() {
		// TODO Auto-generated constructor stub
	}
	@PostConstruct
	public void init() {
		System.out.println("init...");
	}
	@PreDestroy
	public void destory() {
		System.out.println("destory...");
	}
}
```
> 默认Spring工厂管理的Bean都是单例的，同时Spring会在工厂初始化的时候就创建配置的组件bean。这个时候如果想在工厂创建或者工厂消亡的时候执行某些方法时候，可以考虑添加@PostConstruct、@PreDestroy注解即可。

等价写法
``` xml
<bean id="otherComponent" class="com.baizhi.other.OtherComponent" 
          scope="prototype|singleton"
     	  init-method="init" 
     	  destroy-method="destory"/>
```

- **`@Lazy`**
表示延迟spring工厂初始化Bean等价  beans标签的default-lazy-init="true"属性
``` java
@Configuration
@Lazy
public class ApplicationConfig {
  
}
```

- **`@Autowired`、`@Resource`、`@Inject`、`@Qualifier`、`@Primary`**
``` java
@Bean(name="date1")
public Date getDate1() {
		return new Date();
}
@Bean(name="date2")	
public Date getDate2() {
	return new Date();
}
@Bean("date3")
public Date date(@Qualifier("date1")Date date) {
	System.out.println("000000000");
	return date;
}
```
> @Bean管理的Bean的参数默认是自动注入的，此时如果发现有两个相同类型的Bean出现就会导致注入失败，解决之道是使用`@Qualifier`注解指定注入的参数当然也可以使用`@Primary`指定默认参与注入的类型。

``` java
@Bean(name="date1") 
@Primary
public Date getDate1() { 
  return new Date(); 
} 
@Bean(name="date2") 
public Date getDate2() { 
  return new Date(); 
} 
@Bean("date3") 
public Date date(Date date) { 
    System.out.println("000000000"); 
    return date; 
}
```
> @Autowired和@Resource都是解决Spring的值注入的不同的是@Autowired按照类型注入，此时如果容器存在两个相同类型的Bean组件的时候此时注入则是失败的。这个时候需要和@Qulifier或者@Primary注解联合使用解决类型冲突问题；@Resource注解是JDK1.6主持的注解该注解一旦指定name之后只会按照name注入如果没有指定name属性，先按照name注入如果查找不到再按照类型注入。@Inject和@Autowired用法一致但是使用@Inject需要导入额外的Maven依赖，而且如果存在类型冲突需要使用@Named注解指明需要注入的组件名字

``` xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

- **`@Required`**
该属性用于作用在set方法上表明在初始化该Bean的时候，该set个方法的属性必须设置值。一般和配置文件联合使用
``` java
public class UserService implements IUserService{
	private Date date;
	public Date getDate() {
		return date;
	}
	@Required
	public void setDate(Date date) {
		this.date = date;
	}
}
```
配置文件
```
<context:annotation-config/>
<bean id="userService" class="com.baizhi.service.impl.UserService">
		<property name="date">
			<bean class="java.util.Date"></bean>
		</property>
</bean>	
```
> **注意**:必须配置`<context:annotation-config/>`属性，否则@Required不起作用


- **`@Import`(引入配置类)**
``` java
@Configuration
public class ConfigA {
     @Bean
    public A a() {
        return new A();
    }
}
@Configuration
@Import(ConfigA.class)
public class ConfigB {
    @Bean
    public B b() {
        return new B();
    }
}
```
``` java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);
// now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

- **`@Profile`**
``` java
@Configuration
@Import(value= {DevConfig.class,ProConfig.class})
public class ApplicationConfig {
	
}
-----------------------
@Configuration
@ImportResource(value= {"classpath:applicationContext.xml"})
@Profile("dev")
public class DevConfig {
	
}
-----------------------
@Configuration
@Profile("pro")
public class ProConfig {	
}
```
测试环境
``` java
AnnotationConfigApplicationContext ctx= new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("pro");
ctx.register(ApplicationConfig.class);
ctx.refresh();
		
System.out.println(ctx.getBean("userService"));
ctx.close();
```

- **`@ImportResource`(引入外部配置资源)**
``` java
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;
    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```
老版本配置文件导入properties-config.xml
``` xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```
jdbc.properties
```
jdbc.url=jdbc:mysql://localhost:3306/test
jdbc.username=root
jdbc.password=root
```