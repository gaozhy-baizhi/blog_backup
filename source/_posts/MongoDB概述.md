---
title: MongoDB概述 (一)
date: 2018-01-16 17:25:13
tags: 
	- NOSQL
	- 数据库
categories: Big Data
---
{% asset_img  logo.jpg 图片%}
### 一、简介
##### *[MongoDB](https://www.mongodb.com/what-is-mongodb)* 是一款免费开源的NOSQL文档型数据库，旨在为WEB应用提供可护展的高性能数据存储解决方案。
> **Note:**
> 1. *NOSQL(not only sql)：*指的是非关系型数据库,没有固定的存储格式，一般适用于超大规模数据的存储。
> 2. *NOSQL*优点：高可用、可扩展、低成本、数据结构灵活
> 3. *NOSQL*缺点：弱化事务

<!-- more -->
### 二、 特点
1. 数据存储格式为 *BSON*（一种二进制形式的存储格式，类似于 *JSON*）
![](00.png)
2. 丰富的查询语言（CRUD、数据聚合、全文检索、地理位置查询）
3. 高可用（副本集）
4. 水平扩展,支持海量数据存储（分片）
5. MongoDB支持各种编程语言:RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言。
6. 支持完全索引

### 三、 MongoDB中概念解析
| SQL术语     | MongoDB术语 | 说明 |
| ----------- | ----------- | --------- |
| database    | database    | 数据库|
| table       | collection  | 集合 |
| row         | document    | 文档，一条记录 |
| index       | index       | 索引 |
| primary key | primary key | 主键,MongoDB自动将_id字段设置为主键 |
| foreign key            |  无           | 无|

### 四、环境搭建（注：最新版本3.6,只能在64位系统安装）
##### 1. 安装  
  ```
  wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.6.2.tgz
  tar -zxvf mongodb-linux-x86_64-3.6.2.tgz
  mv mongodb-linux-x86_64-3.6.2 mongodb
  ```
##### 2. 启动mongodb服务  
  ![](0.png)
  ```
  # 启动mongo server命令
  ./mongod --port 27017 --dbpath=/data/db
  ```

  \# 启动成功，如下图所示：
  ![](1.png)

### 五、MongoDB的客户端基本操作
##### 1. 打开客户端交互窗口
  ```
  /root/mongodb/bin/mongo 192.168.138.156:27017
  ```

  >注：
  > a. mongodb 客户端是一个javascript交互窗口，可以直接写js代码
  > b. mongodb和mysql数据库有点类似，有数据库的概念，在使用时需要先选中数据库，再执行操作
  
##### 2. 数据库相关操作
  ``show dbs`` 展示所有数据库
  ![](3.png)
  `` db `` 展示当前使用的数据库
  ![](4.png)
  `` use 数据库名 `` 切换到指定数据库
  ![](5.png)
  `` db.dropDatabase(); `` 删除数据库
  ![](6.png)
  ``db.help(); `` 帮助命令
  ![](7.png)