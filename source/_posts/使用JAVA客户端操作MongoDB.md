---
title: 使用JAVA客户端操作MongoDB (三)
date: 2018-01-17 16:41:08
tags: 
	- NOSQL
	- 数据库
categories: Big Data
---
{% asset_img  logo.jpg 图片%}

> ##### NOTE
> 1. 3.6版本的MongoDB，只允许localhost连接，如果您要使用非本地的客户端连接，需要绑定IP才可以，在这里我们关闭mongodb服务端，添加参数 `--bind_ip_all`，重新启动服务。继续测试~~~

#### maven坐标
```
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver</artifactId>
    <version>3.6.1</version>
</dependency>
```
<!-- more -->

#### 1. 插入
  ``` JAVA
  @Test
    public void testInsert(){

        MongoClient mongoClient = new MongoClient("192.168.128.156", 27017);

        // 获取指定数据库
        MongoDatabase database = mongoClient.getDatabase("test");

        // 获取指定集合
        MongoCollection<Document> collection = database.getCollection("users");

        // 创建需要插入的单个文档
        Document doc = new Document("name", "MongoDB")
                .append("type", "database")
                .append("count", 1)
                .append("versions", Arrays.asList("v3.2", "v3.0", "v2.6"))
                .append("info", new Document("x", 203).append("y", 102));

        // 测试插入
        collection.insertOne(doc);

        // 测试插入多个文档
        ArrayList<Document> documents = new ArrayList<Document>();
        for (int i = 0; i < 100; i++) {
             documents.add(new Document("i",i));
        }

        collection.insertMany(documents);
        mongoClient.close();
    }
  ```
  ![add](add.png) 

#### 2. 修改
  ``` JAVA
   @Test
    public void test3(){
        MongoClient mongoClient = new MongoClient("192.168.128.156", 27017);

        // 获取指定数据库
        MongoDatabase database = mongoClient.getDatabase("test");

        // 获取指定集合
        MongoCollection<Document> collection = database.getCollection("users");

        // update t_user set i = 110 where i = 10
        //UpdateResult updateResult = collection.updateOne(new Document("i", 10), new Document("$set", new Document("i", 110)));

        // update t_user set i =10 where i < 20
        // lt(key,value)  静态导入  import static com.mongodb.client.model.Filters.lt;
        collection.updateMany(lt("i",20),new Document("$set",new Document("i",10).append("test","test value")));

        mongoClient.close();
    }
  ```
  ![update4](update4.png)

#### 3. 删除
  ``` JAVA
  @Test
    public void test4(){
        MongoClient mongoClient = new MongoClient("192.168.128.156", 27017);

        // 获取指定数据库
        MongoDatabase database = mongoClient.getDatabase("test");

        // 获取指定集合
        MongoCollection<Document> collection = database.getCollection("users");

        DeleteResult deleteResult = null;

        //deleteResult =collection.deleteOne(eq("i", 110));

        deleteResult = collection.deleteMany(lt("i", 20));

        System.out.println("删除的文档数："+deleteResult.getDeletedCount());

        mongoClient.close();
    }
  ```
  ![delete2](delete2.png)

#### 4. 查询
  ``` JAVA
   @Test
    public void test5(){
        MongoClient mongoClient = new MongoClient("192.168.128.156", 27017);

        // 获取指定数据库
        MongoDatabase database = mongoClient.getDatabase("test");

        // 获取指定集合
        MongoCollection<Document> collection = database.getCollection("users");

        // 查所有
        FindIterable<Document> documents = null;

        documents = collection.find();

        // 条件查询 select * fromt t_user where i = 20
        documents = collection.find(eq("i",20));

        // select * from t_user where i <= 20
        documents = collection.find(lte("i",30));

        // select * from t_user where i = 30 or i = 40 or i = 50
        documents = collection.find(in("i",30,40,50));

        // SELECT * FROM t_user WHERE i = 50 OR i < 25
        documents = collection.find(or(new Document("i",50),lt("i",25)));

        // 模糊查询 参数二：模糊条件
        documents = collection.find(regex("test","test"));

        // 排序 SELECT * FROM t_user WHERE i >= 90 order by i desc
        documents = collection.find(gt("i",90)).sort(Sorts.descending("i"));

        // 分页查询
        documents = collection.find().skip(50).limit(10);

        for (Document document : documents) {
            System.out.println(document);
        }

        mongoClient.close();
    }
  ```