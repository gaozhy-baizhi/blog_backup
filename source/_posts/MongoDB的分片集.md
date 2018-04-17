---
title: MongoDB的分片集 (六)
date: 2018-01-19 16:28:19
tags: 
	- NOSQL
	- 数据库
categories: Big Data
toc: true
---

{% asset_img logo.jpg 分片集%}

> Sharding（分片） is a method for distributing data across multiple machines. MongoDB uses sharding to support deployments with very large data sets and high throughput operations.  
> 
> ##### 个人理解：  
> 分片是一种支持海量数据存储并进行高吞吐量操作的方式  
> 在大数据集和高吞吐量操作的情况下，对单一的服务器硬件要求较高（一般要求算力优异的CPU提供运算能力，RAM或者DISK也要足够大）。传统的方式就是对服务器硬件进行升级，而这样的做的成本往往很高（**垂直扩展**）。而MongoDB提供的分片，其实上就是使用多数的廉价服务器构建成集群，提供海量的数据存储以及并行计算的能力（**水平扩展**）

<!-- more -->

### 一、 原理图
![原理图](sharding.png)

### 二、 分片集群的组件
- **shard server**：用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组个一个replica set承担，防止主机单点故障
- **config server**：顾名思义为配置服务器，存储所有数据库元信息（路由、分片）的配置。
- **mongos server**：数据库集群请求的入口，所有的请求都通过mongos进行协调，不需要在应用程序添加一个路由选择器，mongos自己就是一个请求分发中心，它负责把对应的数据请求请求转发到对应的shard服务器上。在生产环境通常有多mongos作为请求的入口，防止其中一个挂掉所有的mongodb请求都没有办法操作。

### 三、 搭建步骤
> 注意：3.4版本后，config server需要搭建集群

###### 1. 准备5台服务器
  ```
  shard1 192.168.128.156:28001
  shard2 192.168.128.156:28002
  shard3 192.168.128.156:28003
  config1 192.168.128.156:28004
  config2 192.168.128.156:28005
  config3 192.168.128.156:28006
  mongos 192.168.128.156:28007
  ```
###### 2. 启动shard服务器
  ``` json
  mkdir -p /data/{shard1,shard2,shard3}
  mkdir -p /data/{config1,config2,config3}
  mkdir -p /data/mongos
  mongodb/bin/mongod --port 28001 --dbpath=/data/shard1/ --bind_ip 192.168.128.156 --shardsvr
  mongodb/bin/mongod --port 28002 --dbpath=/data/shard2/ --bind_ip 192.168.128.156 --shardsvr
  mongodb/bin/mongod --port 28003 --dbpath=/data/shard3/ --bind_ip 192.168.128.156 --shardsvr
  ```
###### 3. 配置启动config服务器
  ``` json
  # 启动
  mongodb/bin/mongod --port 28004 --dbpath=/data/config1/ --bind_ip 192.168.128.156  --configsvr --replSet rs
  mongodb/bin/mongod --port 28005 --dbpath=/data/config2/ --bind_ip 192.168.128.156  --configsvr --replSet rs
  mongodb/bin/mongod --port 28006 --dbpath=/data/config3/ --bind_ip 192.168.128.156  --configsvr --replSet rs
  
  # 配置副本集
  rs.initiate( {
   _id : "rs",
   members: [
      { _id: 0, host: "192.168.128.156:28004" },
      { _id: 1, host: "192.168.128.156:28005" },
      { _id: 2, host: "192.168.128.156:28006" }
   ]
  })
  
  # 查看配置服务器副本集状态
  rs:SECONDARY> rs.status();
  {
  	"set" : "rs",
  	"date" : ISODate("2018-01-19T01:42:20.535Z"),
  	"myState" : 1,
  	"term" : NumberLong(1),
  	"configsvr" : true,
  	"heartbeatIntervalMillis" : NumberLong(2000),
  	"optimes" : {
  		"lastCommittedOpTime" : {
  			"ts" : Timestamp(1516326131, 1),
  			"t" : NumberLong(1)
  		},
  		"readConcernMajorityOpTime" : {
  			"ts" : Timestamp(1516326131, 1),
  			"t" : NumberLong(1)
  		},
  		"appliedOpTime" : {
  			"ts" : Timestamp(1516326131, 1),
  			"t" : NumberLong(1)
  		},
  		"durableOpTime" : {
  			"ts" : Timestamp(1516326131, 1),
  			"t" : NumberLong(1)
  		}
  	},
  	"members" : [
  		{
  			"_id" : 0,
  			"name" : "192.168.128.156:28004",
  			"health" : 1,
  			"state" : 1,
  			"stateStr" : "PRIMARY",
  			"uptime" : 175,
  			"optime" : {
  				"ts" : Timestamp(1516326131, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDate" : ISODate("2018-01-19T01:42:11Z"),
  			"electionTime" : Timestamp(1516326018, 1),
  			"electionDate" : ISODate("2018-01-19T01:40:18Z"),
  			"configVersion" : 1,
  			"self" : true
  		},
  		{
  			"_id" : 1,
  			"name" : "192.168.128.156:28005",
  			"health" : 1,
  			"state" : 2,
  			"stateStr" : "SECONDARY",
  			"uptime" : 133,
  			"optime" : {
  				"ts" : Timestamp(1516326131, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDurable" : {
  				"ts" : Timestamp(1516326131, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDate" : ISODate("2018-01-19T01:42:11Z"),
  			"optimeDurableDate" : ISODate("2018-01-19T01:42:11Z"),
  			"lastHeartbeat" : ISODate("2018-01-19T01:42:18.777Z"),
  			"lastHeartbeatRecv" : ISODate("2018-01-19T01:42:19.933Z"),
  			"pingMs" : NumberLong(0),
  			"syncingTo" : "192.168.128.156:28004",
  			"configVersion" : 1
  		},
  		{
  			"_id" : 2,
  			"name" : "192.168.128.156:28006",
  			"health" : 1,
  			"state" : 2,
  			"stateStr" : "SECONDARY",
  			"uptime" : 133,
  			"optime" : {
  				"ts" : Timestamp(1516326131, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDurable" : {
  				"ts" : Timestamp(1516326131, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDate" : ISODate("2018-01-19T01:42:11Z"),
  			"optimeDurableDate" : ISODate("2018-01-19T01:42:11Z"),
  			"lastHeartbeat" : ISODate("2018-01-19T01:42:18.778Z"),
  			"lastHeartbeatRecv" : ISODate("2018-01-19T01:42:20.030Z"),
  			"pingMs" : NumberLong(0),
  			"syncingTo" : "192.168.128.156:28004",
  			"configVersion" : 1
  		}
  	],
  	"ok" : 1,
  	"operationTime" : Timestamp(1516326131, 1),
  	"$gleStats" : {
  		"lastOpTime" : Timestamp(1516326007, 1),
  		"electionId" : ObjectId("7fffffff0000000000000001")
  	},
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1516326131, 1),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  }
  ```
  
  ![sharding1](sharding1.png) 
  
###### 4. 启动mongos服务器
  ```
  mongodb/bin/mongos --port 28007 --bind_ip 192.168.128.156 --configdb rs/192.168.128.156:28004,192.168.128.156:28005,192.168.128.156:28006
  ```
  
###### 5. 初始化分片集群
  ```
  # 使用客户端命令连接分片集群
  mongodb/bin/mongo 192.168.128.156:28007
  
  # 设置chunk大小
  use config
  db.settings.save( { _id:"chunksize", value: 1 } )
  
  # 添加分片节点
  db.runCommand({addShard:"192.168.128.156:28001"});
  db.runCommand({addShard:"192.168.128.156:28002"});
  db.runCommand({addShard:"192.168.128.156:28003"});
  ```

  ``` 
  mongodb/bin/mongo 192.168.128.156:28007
  
  # MongoDB分片是针对集合的，要想使集合支持分片，首先需要使其数据库支持分片，为数据库testdb启动分片
  sh.enableSharding("testdb");
  
  # 为分片字段建立索引，同时为集合指定片键
  use testdb
  db.users.ensureIndex({name:1});
  
  # 启用集合分片，为其指定片键
  sh.shardCollection("testdb.users",{name:1});
  ```
  ![Sharding2](sharding2.png) 
  
  ![分片状态](sharding3.png) 

###### 6. 测试
  ``` JAVASCRIPT
  // 连接mongos，插入50W数据测试下分片
  for(var i = 0;i<500000;i++){
  	db.users.insert({"name":"zs"+i,"age":i});
  }
  ```
  
  ``` JSON
  mongos> sh.status();
  --- Sharding Status --- 
    sharding version: {
    	"_id" : 1,
    	"minCompatibleVersion" : 5,
    	"currentVersion" : 6,
    	"clusterId" : ObjectId("5a6163a5130c3601a3a20db4")
    }
    shards:
          {  "_id" : "shard0000",  "host" : "192.168.128.156:28001",  "state" : 1 }
          {  "_id" : "shard0001",  "host" : "192.168.128.156:28002",  "state" : 1 }
          {  "_id" : "shard0002",  "host" : "192.168.128.156:28003",  "state" : 1 }
    active mongoses:
          "3.6.2" : 1
    autosplit:
          Currently enabled: yes
    balancer:
          Currently enabled:  yes
          Currently running:  yes
          Collections with active migrations: 
                  testdb.users started at Fri Jan 19 2018 11:25:25 GMT+0800 (CST)
          Failed balancer rounds in last 5 attempts:  0
          Migration Results for the last 24 hours: 
                  5 : Success
    databases:
          {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                  config.system.sessions
                          shard key: { "_id" : 1 }
                          unique: false
                          balancing: true
                          chunks:
                                  shard0000	1
                          { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : shard0000 Timestamp(1, 0) 
          {  "_id" : "testdb",  "primary" : "shard0000",  "partitioned" : true }
                  testdb.users
                          shard key: { "name" : 1 }
                          unique: false
                          balancing: true
                          chunks:
                                  shard0000	3
                                  shard0001	3
                                  shard0002	5
                          { "name" : { "$minKey" : 1 } } -->> { "name" : "zs1" } on : shard0002 Timestamp(5, 0) 
                          { "name" : "zs1" } -->> { "name" : "zs108900" } on : shard0002 Timestamp(6, 2) 
                          { "name" : "zs108900" } -->> { "name" : "zs17318" } on : shard0002 Timestamp(6, 3) 
                          { "name" : "zs17318" } -->> { "name" : "zs19072" } on : shard0002 Timestamp(6, 4) 
                          { "name" : "zs19072" } -->> { "name" : "zs28146" } on : shard0001 Timestamp(5, 1) 
                          { "name" : "zs28146" } -->> { "name" : "zs42" } on : shard0001 Timestamp(4, 3) 
                          { "name" : "zs42" } -->> { "name" : "zs5163" } on : shard0001 Timestamp(4, 4) 
                          { "name" : "zs5163" } -->> { "name" : "zs60703" } on : shard0002 Timestamp(6, 0) 
                          { "name" : "zs60703" } -->> { "name" : "zs6978" } on : shard0000 Timestamp(6, 1) 
                          { "name" : "zs6978" } -->> { "name" : "zs8724" } on : shard0000 Timestamp(5, 4) 
                          { "name" : "zs8724" } -->> { "name" : { "$maxKey" : 1 } } on : shard0000 Timestamp(1, 3) 
  
  ```
  `分片结果`
  ![sharding4](sharding4.png) 
  
  ![sharding5](sharding5.png) 