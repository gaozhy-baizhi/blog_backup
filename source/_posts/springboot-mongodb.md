---
title: SpringBoot系列——集成MongoDB
date: 2018-04-12 15:25:06
tags: 
	- 微服务
	- 框架
categories: JAVA
---
{% asset_img  logo.jpg mongodb%} 

> 注：示例代码使用的Spring Boot版本为：2.0.1.RELEASE

- **创建Spring Boot工程并导入Maven依赖**
``` xml
<dependencies>
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-mongodb</artifactId>
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
```
<!-- more -->

- **修改配置文件 application.yml**
``` yml
spring:
  data:
    mongodb:
      host: 192.168.128.156
      port: 27017
      database: test
```

- **创建User实体**
``` java
@Document(collection = "user") // 集合名
public class User {

    @Id
    private String id;

    private String name;

    private Double salary;

    private Date birthday;
    
    // 省略 getter/setter
}
```
> **Note:**
> User映射Document

- **创建MongoRepository**
``` java
package com.baizhi.dao;

import com.baizhi.entity.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * @author gaozhy
 * @date 2018/4/11.16:42
 */
@Repository
public class MongoRepository {

    @Autowired
    private MongoTemplate mongoTemplate;

    public void insertOne(User user){
        mongoTemplate.insert(user);
    }

    public void deleteOne(User user){
        mongoTemplate.remove(user);
    }

    public User findOne(String id){
        return mongoTemplate.findById(id,User.class);
    }

    public List<User> findAll(){
        return mongoTemplate.findAll(User.class);
    }

    public List<User> findByName(String name){
        return mongoTemplate.find(Query.query(Criteria.where("name").is(name)),User.class);
    }
}
```

- **测试类**
``` java
package com.baizhi;

import com.baizhi.dao.MongoRepository;
import com.baizhi.entity.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Date;
import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootMongodbApplicationTests {

   @Autowired
   private MongoRepository mongoRepository;

   @Test
   public void contextLoads() {
        mongoRepository.insertOne(new User("zs1",2000.0D,new Date()));
        List<User> users = mongoRepository.findAll();
        users.forEach(user -> System.out.println(user));
    }
}
```
测试结果
![](2018-04-12_154535.jpg)