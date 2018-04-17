---
title: 搭建MongoDB副本集 (四)
date: 2018-01-17 17:30:13
tags: 
	- NOSQL
	- 数据库
categories: Big Data
toc: true
---

{% asset_img logo.jpg logo%}

{% blockquote Mongo官网—— https://docs.mongodb.com/manual/replication/ Replication %}
  - 原文：A replica set in MongoDB is a group of mongod processes that maintain the same data set. Replica sets provide redundancy and high availability, and are the basis for all production deployments.
  - 译文：副本集是一组mongod进程，它维护了同样的数据集。副本集提供了信息冗余和高可用，是所有生产的基础
  
{% endblockquote %}

<!-- more -->

![副本集](replset0.png)

#### 一、 节点
##### 1. `Primary Node` 主节点，一个副本集只能有一个主节点，主要作用接受客户端所有写操作（默认情况下，也可以读取数据），并记录主节点操作日志，副节点复制主节点日志，用其同步数据。
![replset1](replset1.png)  

##### 2. `Secondary Node ` 副节点，复制主节点的操作，并同步其数据，实际上副节点是主节点数据的备份。如果主节点挂掉的话，剩余的副节点会触发选举算法，将其中的一个副节点，选举为主节点。 
![replset2](replset2.png) 

#### 二. 搭建步骤
##### 1. 准备至少三台服务器（注意：在这里我们在同一台虚拟机上，通过端口号区分不同的Mongo Server）
	```
	Node1: 192.168.128.156 28000
	Node2: 192.168.128.156 28001
	Node3: 192.168.128.156 28002
	```

##### 2. 准备数据存放目录
  ``` shell
  mkdir -p /data/node1
  mkdir -p /data/node2
  mkdir -p /data/node3
  ```
##### 3. 分别启动三台服务器
  ```
  # 启动主服务器
  mongodb/bin/mongod --port 28000 --dbpath=/data/node1/ --bind_ip 192.168.128.156 --replSet rs
  
  # 分别启动两个副服务器
  mongodb/bin/mongod --port 28001 --dbpath=/data/node2/ --bind_ip 192.168.128.156 --replSet rs
  mongodb/bin/mongod --port 28002 --dbpath=/data/node3/ --bind_ip 192.168.128.156 --replSet rs
  ```
  
  ![replset3](replset3.png) 

##### 4. 初始化副本集（使用客户端连接任意Mongo Server）
  ```
  # 连接Mongo Server
  /root/mongodb/bin/mongo 192.168.128.156:28000
  
  # 初始化副本集（注意：_id的名字应该和启动参数 `--replSet` value一致）
  rs.initiate( {
     _id : "rs",
     members: [
        { _id: 0, host: "192.168.128.156:28000" },
        { _id: 1, host: "192.168.128.156:28001" },
        { _id: 2, host: "192.168.128.156:28002" }
     ]
  })
  ```
  ![replset4](replset4.png) 

##### 5. 查看副本集状态  
  `rs.status();`  
  
  ![replset5](replset5.png) 
  
##### 6. 测试副本集数据同步
  ```
  # 亲们，我们先往主节点插入一条数据
  ```
  
  ![replset6](replset6.png) 
  
  ```
  # 好了，我们再连接副节点，执行查看命令
  /root/mongodb/bin/mongo 192.168.128.156:28001
  ```
  
  ![replset7](replset7.png) 
  
  ```
  # 咦，为什么出现错误了呢？ 
  # 原因是因为副本集中默认使用主节点读写数据，副节点只做数据备份，不参与读操作。
  # 哎呦！原来如此
  # 那如何解决这个问题呢？
  # 好了不卖关子了
  # 在客户端执行命令 db.getMongo().setSlaveOk();
  # 再试试看，是不是OK了！看到同步过来的数据没？ 啦啦啦~~
  ```
  
  ![replset8](replset8.png) 

##### 7. 测试集群容错  
  ![replset2](replset2.png) 
  
  ```
  # 副本集其余的小伙伴，要选举新的Leader啦，原谅我再次引用这张图
  # 说干就干
  # 关闭端口28000的服务
  # 结果如图，是不是很神奇？
  ```
  
  ![replset10](replset10.png) 
  
  ```
  # 执行命令rs.status()
  # 发现关闭28000主节点端口后，28002端口的副节点经过选举之后变成了主节点，而28001端口的副节点变成了28002节点的副节点
  ```