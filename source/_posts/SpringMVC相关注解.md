---
title: Spring4.0常用注解——MVC相关注解
date: 2018-04-09 16:34:08
tags:
---
- **`@EnableWebMvc`**

该注解用于开启基于注解的MVC支持
``` java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "com.baizhi")
public class MvcConfig {

}
```
<!-- more -->
老版本写法
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven></mvc:annotation-driven>
</beans>
```

启动引导类
``` java
/**
 * @author gaozhy
 * @date 2018/4/8.11:18
 */
public class WebInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(MvcConfig.class);
        ctx.setServletContext(servletContext);

        ServletRegistration.Dynamic servlet = servletContext.addServlet("dispatcher", new DispatcherServlet(ctx));
        servlet.addMapping("/");
        servlet.setLoadOnStartup(1);
    }
}
```

- **`@Controller`**

该注解指示特定的类充当控制器的角色
``` java
@Controller
public class HelloController {
    
}
```

- **`@RestController`**
该注解为组合注解@Controller和@ResponseBody（控制器实现REST API，响应JSON、XML或自定义的MediaType内容。无需用@ResponseBody注释所有的@RequestMapping方法。），用于构建Restful风格的控制器类

``` java
@RestController
public class RestfulController {

}
```

- **`@RequestMapping`**
``` java
@RequestMapping(value = "/",method = RequestMethod.GET)
public String index(){
    return "Hello World";
}
```

> **以下注解用于简化HTTP方法的映射，并更好地表达带注释的处理程序方法的语义。例如，@GetMapping 等同于@RequestMapping(method = RequestMethod.GET)**
> - @GetMapping
> - @PostMapping
> - @PutMapping
> - @DeleteMapping
> - @PatchMapping

- **`@PathVariable`**

该注解放在方法参数前，用于绑定URL中携带的Value
```
/**
 * 例如：http://localhost:8080/owners/1
 */
@GetMapping("/owners/{ownerId}")
@ResponseBody
public String findOwner(@PathVariable String ownerId) {
    System.out.println(ownerId);
    return ownerId;
}
```

- **`@ModelAttribute`**
	- 用法一：应用在普通方法上
	被 @ModelAttribute 注解的方法会在Controller每个目标方法执行之前都执行（注：对于一个Controller中包含多个URL的时候，要谨慎使用），并且会将返回值自动存储到Model中（key默认为首字母小写的返回值名，value为方法返回值）
	``` java
	/**
	 * 例如：http://localhost:8080/owners/1
	 */
	@GetMapping("/owners/{ownerId}")
	@ResponseBody
	public String findOwner(@PathVariable String ownerId) {
	    System.out.println(ownerId);
	    return ownerId;
	}

	@ModelAttribute
	public User getUser(){
	    System.out.println("----------进入到 getUser method----------");
	    return new User(1,"张三");
	}

	@ModelAttribute
	public Person getPerson(){
	    System.out.println("----------进入到 getPerson method----------");
	    return new Person(1,"李四","北京");
	}
	```
	> 在请求http://localhost:8080/owners/1，会依次访问getUser、getPerson，然后再去访问目标方法findOwner，被@ModelAttribute注解的方法的返回值会自动添加到Model中保存起来。
	
	类似于
	``` java
	@GetMapping("/owners/{ownerId}")
	@ResponseBody
	public String findOwner(@PathVariable String ownerId,Model model) {
	    model.addAttribute("user",new User(1,"张三"));
	    model.addAttribute("person",new Person(1,"李四","北京"));
	    System.out.println(ownerId);
	    return ownerId;
	}
	```

	- 用法二：应用在方法的参数上
	将Model中Key对应的Value绑定给指定参数
	``` java
	@ModelAttribute
	public User getUser(){
	    System.out.println("----------进入到 getUser method----------");
	    return new User(1,"张三");
	}

	@ModelAttribute
	public Person getPerson(){
	    System.out.println("----------进入到 getPerson method----------");
	    return new Person(1,"李四","北京");
	}

	@GetMapping("/test")
	@ResponseBody
	public String test(@ModelAttribute("user")User user,@ModelAttribute("person")Person person) {

	    System.out.println(user);
	    System.out.println(person);

	    return "test ok";
	}
	```


- **`@SessionAttributes`**
该注解使用在类上，用于将Model中的数据保存到HttpSession中（跨请求传递数据）

| 属性名    | 说明 				 										    	   |
| --------------- | --------------	 						   							   |
| value      | 指定存放到Session域中Model的key的键名   												   |
| type      | 指定存放到Session域中Model的value的类型  												   |
``` java
@Controller
@SessionAttributes(value={"name"},types = Integer.class)
public class HelloController {

    @RequestMapping("/test1")
    public String test1(Model model){
        System.out.println("--------------test1-----------------");
        model.addAttribute("name","zs");
        model.addAttribute("age",18);
        model.addAttribute("sex","male");
        return "redirect:test2";
    }

    @RequestMapping("/test2")
    @ResponseBody
    public String test2(ModelMap modelMap, SessionStatus sessionStatus){
        System.out.println("--------------test2-----------------");
        System.out.println(modelMap.get("name") + " | "+ modelMap.get("age")+ " | "+modelMap.get("sex"));
        // 清除Session中保存的数据
        sessionStatus.setComplete();
        return "test2 ok";
    }
}
```

- **`@SessionAttribute`**
该注解用于将Session中key对应的value绑定给指定参数
``` java
@RequestMapping("/test3")
@ResponseBody
public String test3(@SessionAttribute("name") String name,@SessionAttribute("age") Integer age){
    System.out.println("--------------test3-----------------");
    System.out.println(name + " | " + age);
    return "test3 ok";
}
```

- **`@RequestAttribute`**
该注解用于将Request中key对应的value绑定给指定参数
``` java
@RequestMapping("/test4")
@ResponseBody
public String test4(@RequestAttribute("sex")String sex){
    System.out.println("--------------test4-----------------");
    System.out.println(sex);
    return "test4 ok";
}
```

- **`@RequestParam`**
该注解用于绑定请求参数给控制器方法指定参数
``` java
// http://localhost:8080/test5?id=1&name=zs
@RequestMapping("/test5")
@ResponseBody
public String test5(@RequestParam("id") Integer id,@RequestParam("name") String username){
    System.out.println("--------------test5-----------------");
    System.out.println(id + " | " + username);
    return "test5 ok";
}
```

- **`@RequestHeader`**
该注解用于将请求头中参数绑定给控制器指定方法参数
``` java
@RequestMapping("/test6")
@ResponseBody
public String test6(@RequestHeader Map requestHeaderMap){
    System.out.println("--------------test6-----------------");
    requestHeaderMap.forEach((k,v) -> System.out.println(k + " | " +v));
    return "test6 ok";
}
```
> **注**：在上面的案列中，请求头中所有数据会以key-value的方式存放到Map集合中，也可以使用 @RequestHeader("Accept-Encoding") 这样的方式，获取特定信息。

- **`@CookieValue`**
该注解用于将Cookie中的value绑定给控制器指定方法参数
``` java
@RequestMapping("/test7")
@ResponseBody
public String test7(@CookieValue("JSESSIONID") String jessionId){
    System.out.println("--------------test7-----------------");
    System.out.println(jessionId);
    return "test7 ok";
}
```

- **`@RequestBody`**
该注解用于将请求主体通过HttpMessageConverter读取和反序列化为对象
``` java
@PostMapping("/test8")
@ResponseBody
public String test8(@RequestBody User user){
    System.out.println(user);
    return "test8 ok";
}
```

- **`@ResponseBody`**
该注解用于将方法返回值通过HttpMessageConverter序列化到响应体

- **`@CrossOrigin`**
该注解用于支持跨越请求处理
``` java
@PostMapping("/test10")
@ResponseBody
@CrossOrigin(origins = "http://192.168.31.187:8080",maxAge = 1800)
public String test10(String name,Integer id){
    System.out.println(name + " | " + id);
    return "test10 ok";
}
```
> 浏览器从一个域名的网页去请求另一个域名的资源时，域名、端口、协议任一不同，都是跨域 