---
title: Spring4.0常用注解——@AspectJ相关注解（基于注解的AOP）
date: 2018-04-04 17:30:33
tags:
---

> @AspectJ是一种aspects 的风格，它是用注释注释的常规Java类。

- **`@EnableAspectJAutoProxy`**

该注解用于开启@AspactJ支持
``` java
@Configuration
@EnableAspectJAutoProxy
@ComponentScan(basePackages = "com.baizhi")
public class AopConfig {

}
```
<!-- more -->
等价写法
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
        
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

- **`@Aspect`**

该注解用于声明切面类
``` java
@Aspect
@Component
public class AspectModule {
    
}
```

- **`@Pointcut`**

该注解用于声明切入点
``` java
@Pointcut("execution(* com.baizhi..*.*(..))") // 切入点表达式
public void pc(){}
```
常用的切入点标志符

| 切入点标志符    | 说明 				 										    	   |
| --------------- | --------------	 						   |
| execution       | 匹配执行方法的连接点   												   |
| within          | 限定匹配特定类型的连接点           									   |
| this            | 用于指定AOP代理必须是指定类型的实例，用于匹配该对象的所有连接点   	   | 
| target          | 用于限定目标对象必须是指定类型的实例，用于匹配该对象的所有连接点       |
| args            | 用于对连接点的参数类型进行限制，要求参数的类型时指定类型的实例 		   |
| @annotation     |  用于对连接点的注解类型进行限制，要求注解的类型为指定类型的实例        | 

- **`@Before`**

该注解用于声明前置通知
``` java
@Aspect
public class AspectModule {
    @Pointcut("execution(* com.baizhi..*.*(..))")
    public void pc(){}
    
    // 参数为切入点方法名（好处，定义一个切入点实现复用）
    @Before("pc()")
    // JoinPoint 用于获取连接点信息（如获取方法和类信息）
    public void before(JoinPoint joinPoint){ 
        System.out.println("-------------before advice-----------");
    }
}
```

老版本写法
``` java
public class MyBeforeAdvice implements MethodBeforeAdvice{

    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println("-------------before advice-----------");
    }
}
```

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="userService" class="com.baizhi.service.UserServiceImpl"></bean>
    
    <bean id="beforeAdvice" class="MyBeforeAdvice"></bean>

    <aop:config>
        <aop:pointcut id="pc" expression="execution(* com.baizhi..*.*(..))"></aop:pointcut>
        <aop:advisor advice-ref="beforeAdvice" pointcut-ref="pc"></aop:advisor>
    </aop:config>
</beans>
```

- **`@AfterReturning`**

该注解用于声明匹配方法正常返回的后置通知
``` java
@AfterReturning("pc()")
public void afterReturning(){
    System.out.println("-------------after returning advice-----------");
}
```

正常返回  
![](2018-04-04_155455.jpg)  

非正常返回  
![](2018-04-04_155557.jpg)


- **`@AfterThrowing`**

该注解用于声明异常通知（目标方法产生异常执行的代码片段）
``` java
@AfterThrowing("pc()")
public void afterThrowing(){
    System.out.println("-------------after throwing advice-----------");
}
```

切入方法抛出异常
![](2018-04-04_160154.jpg)

- **`@After`**

该注解用于声明后置通知（无论目标方法是否产生异常都会执行的代码片段）
``` java
@After("pc()")
public void after(){
    System.out.println("-------------after advice-----------");
}
```
![](2018-04-04_160728.jpg)

- **`@Around`**

该注解用于声明环绕通知（在调用目标方法前后执行）
``` java
@Around("pc()")
// 注意：参数ProceedingJoinPoint只能适用于环绕通知，用于获取连接点信息
public void aroud(ProceedingJoinPoint pjp){
    System.out.println("-------------aroud advice start-----------");
    try {
        // 调用目标方法 并获取目标方法返回值
        Object proceed = pjp.proceed();
    } catch (Throwable throwable) {
        throwable.printStackTrace();
    }
    System.out.println("-------------aroud advice end-----------");
}
```

- **`@Order`**

该注解用于指定多切面执行顺序（@Order值越小优先级越高）
``` java
@Aspect
@Component
@Order(1)
public class AspectModule {

    @Pointcut("execution(* com.baizhi..*.*(..))")
    public void pc(){}

    @Before("pc()")
    public void before1(JoinPoint joinPoint){
        System.out.println("------before1 advice first execute-----");
    }
}
```

``` java
@Aspect
@Component
@Order(5)
public class AspectModule1 {

    @Pointcut("execution(* com.baizhi..*.*(..))")
    public void pc(){}
    
    @Before("pc()")
    public void before2(JoinPoint joinPoint){
        System.out.println("------before2 advice second execute-----");
    }
}
```

老版本写法
``` xml
<aop:config>
    <aop:pointcut id="pc" expression="execution(* com.baizhi..*.*(..))"></aop:pointcut>
    <aop:advisor advice-ref="beforeAdvice1" pointcut-ref="pc" order="1"></aop:advisor>
    <aop:advisor advice-ref="beforeAdvice2" pointcut-ref="pc" order="5"></aop:advisor>
</aop:config>
```
![](2018-04-04_180425.jpg)