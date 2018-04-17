---
title: Hadoop概述及环境搭建 (一)
date: 2018-03-13 22:45:50
tags:
	- HDFS
	- MapReduce
categories: Big Data
toc: true
---
{% asset_img  logo.jpg 图片%}
### 一、概述
`Hadoop`衍生自`Nutch`（搜索引擎和web爬虫），面临的问题：海量数据存储和计算  

`Big Data`大数据，谈的不仅仅是数据量，其实包含了数据量（Volume）、时效性（Velocity）、多样性（Variety）、可疑性（Veracity）

`Hadoop`是一个开源存储和计算框架，`HDFS`大规模数据存储服务，`MapReduce`实现了对海量数据的并行处理和分析。

使用领域：电商推荐、论坛（精装营销）、交通（实时路况）、医疗、电信、金融
<!-- more -->
### 二、Hadoop生态系统
`HBase`：面向列存储的NOSQL数据库，使用`HDFS`作为底层存储服务  
`Hive`: 一款工具，将用户的`SQL`翻译成`MapReduce`任务  
`Zookeeper`: 分布式协调服务框架  
`Mahout`: 一个可以扩展的及其学习以及数据挖掘库  
`Spark`：一个快速的通用的计算引擎用于计算Hadoop上的数据。基于内存  
`flume`: 分布式日志采集，实现对数据收集、转移以及聚合  
`Kafaka`: 分布式消息队列，实现对海量数据的缓冲  

### 三、大数据的计算形式
#### 离线计算
不适用实时性要求较高的计算  
1. `Hadoop`的离线计算（基于磁盘的迭代计算）   如：火车普快
2. `Spark`离线计算（基于内存的迭代计算）      如：火车高铁

#### 在线 | 流计算
实时在线计算（在线推荐）
1. `Strom`流计算框架 任务——>流程 处理——>结果  如：来一件衣服洗一件
2. `kafaka stream`流计算框架 任务——>流程 处理——>结果

### 四、环境搭建（单机伪分布式）
搭建步骤： 
- CentOS6.5
- 安装JDK环境
- 配置主机名
	``` shell
	vim /etc/sysconfig/network
	# 将主机名修改为Hadoop
	HOSTNAME=Hadoop
	# 保存退出 并重启
	reboot
	```
- 配置主机名和IP的映射关系
  ``` shell
  vi /etc/hosts
  # 在文件的末尾新增IP和主机映射
  192.168.128.137 Hadoop
  # 测试
  ping Hadoop
  ```
  ![映射关系测试](映射关系测试.png) 
- 配置SSH  
  > 摘自官网：ssh must be installed and sshd must be running to use the Hadoop scripts that manage remote Hadoop daemons.  
  > 主要目的：实现`Hadoop`服务器间的免密访问，加密方式`非对称加密`
  
  ``` shell
  # 测试 是否可以免密访问
  # 如无需输入密码 跳过
  ssh 主机名
  
  # 生成公钥、私钥
  ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
  
  # 将生成的公钥添加到需要免密访问的主机的授权文件中
  cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
  ```
- 解压并安装Hadoop
  ``` shell
  # 解压
  tar -zxvf hadoop-2.6.0.tar.gz
  mkdir /usr/hadoop
  mv hadoop-2.6.0 /usr/hadoop/
  
  vi /etc/profile
  # 配置环境变量  在/etc/profile文件目录末尾添加以下内容
  export CLASSPATH=.
  export JAVA_HOME=/usr/java/jdk1.7.0_71/
  export HADOOP_HOME=/usr/hadoop/hadoop-2.6.0
  export PATH=$PATH:$HADOOP_HOME/bin:$JAVA_HOME/bin:$HAPOOP_HOME/sbin
  
  # 保存退出 生效配置
  source /etc/profile
  ```
  
  ``` shell
  # 修改配置文件
  vi /usr/hadoop/hadoop-2.6.0/etc/hadoop/hadoop-env.sh 
  export JAVA_HOME=/usr/java/jdk1.7.0_71/
  ```
  
  ``` xml
  vi /usr/hadoop/hadoop-2.6.0/etc/hadoop/core-site.xml
  <configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/hadoop/hadoop-${user.name}</value>
    </property>
  </configuration>
  ```
  
  ``` xml
  vi /usr/hadoop/hadoop-2.6.0/etc/hadoop/hdfs-site.xml
  <configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
  </configuration>
  ```
- 初始化
  ``` shell
  # 初始化namenode
  hdfs namenode -format
  ```
  ![初始化成功](初始化成功.png) 
- 启动
  ``` shell
  start-dfs.sh
  ```
- 测试  
方案一：使用浏览器访问 http://服务器ip:50070  
![浏览器结果访问成功](浏览器结果访问成功.png) 
方案二：`jps`指令  
![jps测试成功效果](jps测试成功效果.png) 
