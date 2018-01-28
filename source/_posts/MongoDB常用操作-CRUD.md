---
title: MongoDB常用操作命令 (二)
date: 2018-01-16 17:39:44
tags: 
	- NOSQL
	- 数据库
categories: Big Data
---
{% asset_img  logo.jpg 图片%}

### 一、插入操作
##### 1. 单个文档插入
<!-- more -->
  `db.COLLECTION_NAME.insert(document)`
  ![](8.png) 
##### 2. 批量文档插入
  `db.COLLECTION_NAME.insert([document,document1.....])`
  ![](9.png)  
  > 注：在高版本mongodb中又新增了 db.collection.insertOne()和db.collection.insertMany() 这两个方法，分别用来插入单个文档和多个文档

### 二、查询操作
##### 1. 查集合中所有文档   
  `db.collection.find( {} )` 等同于SQL查询中的 `SELECT * FROM collection`
  ![](10.png) 

##### 2. 指定条件查询 `{ <field1>: <value1>}`  
  例: `db.users.find({"name":"zs"});` 类似于 `select * from t_user where name = 'zs' `
  > 注：在这里我们先学习下最基本的语法，更复杂的查询在后续介绍


### 三、修改文档
##### 1. `db.collection.update(<filter>, <update>);` 
  ![update](update.png) 
  
##### 2. `db.collection.updateOne(<filter>, {<update operator>: { <field1>: <value1>, ... })` 只修改符合条件的第一条文档
  ![update1](update1.png) 
  > 注：`$set` 操作符 使用指定的value替换旧值,如果修改的属性不存在，则创建
  
##### 3. `db.collection.updateMany(<filter>, {<update operator>: { <field1>: <value1>, ... })`  将符合条件的所有文档都进行修改
  ![update2](update2.png) 
  
##### 4. `db.collection.replaceOne()` 替换除_id外的所有属性
  ![update3](update3.png) 

### 四、删除文档
##### 1. `db.collection.deleteOne({<field1>: <value1>,...})`
  ![](delete.png)
  
##### 2. `db.collection.deleteMany({<field1>: <value1>,...})`
  ![delete1](delete1.png)

### 五、 复杂查询  
##### 1. 使用查询操作符指定条件  
  语法： `{ <field1>: { <operator1>: <value1> }, ... }`
     
  - **$in**   
    `db.collection.find( { status: { $in: [ "A", "D" ] } } )` 等同于 `SELECT * FROM collection WHERE status in ("A", "D")`
  - **$and** | **$lt**  
    `db.collection.find( { status: "A", qty: { $lt: 30 } } )` 等同于 `SELECT * FROM collection WHERE status = "A" AND qty < 30`
  - **$or**  
    `db.collection.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )` 等同于 `SELECT * FROM collection WHERE status = "A" OR qty < 30`
  - **$and** 和 **$or**  
    `db.collection.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
     } )
     ` 等同于 `SELECT * FROM collection WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")`
  - **$lt | $gt | $lte | $gte**
  
##### 2. 查询结果排序  
  语法： `db.collection.find().sort({_id:1})` **1 升序 -1 降序**
    
##### 3. 分页查询  
  语法： `db.collection.find().sort().skip(起始条数).limit(结束条数)`
    
##### 4. 总条数  
  语法： `db.collection.find().count()` 
  
##### 5. 模糊查询
  语法： `db.collection.find({"name":/zs/});`