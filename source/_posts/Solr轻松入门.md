---
title: Solr轻松入门
date: 2018-01-07 11:24:18
tags: 
	- 全文检索
	- 引擎
categories: Big Data
toc: true
---

{% asset_img  logo.jpg 学习笔记%}

## 1. Solr概述
- Solr是Apache下的顶级项目，采用java开发，它是基于Lucene的全文检索服务器。Solr提供了比Lucene更为丰富的查询语言，同时实现了可配置、可扩展，并对索引、搜索性能进行了优化。
- Solr可以运行在Jetty、Tomcat等Web服务器。

<!-- more -->
## 2. Solr架构（体系架构）
- 系统架构图  	
	![](1.jpg)
- 组件
	- **请求处理程序**  - 发送到**Apache Solr**的请求由这些请求处理程序处理。请求可以是查询请求或索引更新请求。根据这些请示的要求来选择请求处理程序。为了将请求传递给Solr，通常将处理器映射到某个URI端点，并且它将为指定的请求提供服务。
	- **搜索组件**  -  搜索组件是**Apache Solr**中提供的搜索类型(功能)。它可能是拼写检查，查询，构面，命中突出显示等。这些搜索组件被注册为搜索处理程序。多个组件可以注册到搜索处理程序。
	- **查询解析器**− **Apache Solr**查询解析器解析传递给Solr的查询，并验证查询的语法是否有错误。解析查询后，将它们转换为Lucene理解的格式。
	- **响应写入器** - **Apache Solr**中的响应写入器是为用户查询生成格式化输出的组件。 Solr支持XML，JSON，CSV等响应格式。对每种类型的响应都有不同的响应写入。
	- **分析器/分词器** - Lucene以令牌的形式识别数据。 Apache Solr分析内容，将其分成令牌，并将这些令牌传递给Lucene。 Apache Solr中的分析器检查字段的文本并生成令牌流。分词器将分析器准备的令牌流分解成令牌。
	- **更新请求处理器**- 每当向Apache Solr发送更新请求时，请求都通过一组称为更新请求处理器的插件(签名，日志记录，索引)运行。这个处理器负责修改，例如删除字段，添加字段等。
	
## 3. Solr安装和环境搭建
- 下载（windows：版本号：solr-5.5.0.zip）
- Solr目录介绍
	1. bin： 运行脚本目录
	2. contrib： 插件目录
	3. dist： solr相关jar包
	4. doc： 文档目录
	5. example： solr示例项目
	6. server： 服务相关数据
- Solr所需环境
	1. jdk1.7+、Web服务器Tomcat8.0
- Solr和Tomcat整合
	1. 在tomcat的webapps目录新建文件夹solr，并将solr-5.5.0\server\solr-webapp\webapp目录下的所有资源复制过来
	2. 将solr-5.5.0\server\lib\ext下的所有jar包复制到tomcat\webapps\solr\WEB-INF\lib
	3. 将solr-5.5.0\server\resources\log4j.properties文件复制到tomcat\webapps\solr\WEB-INF\classes目录（*如果没有classes目录请新建*）
	4. 在tomcat目录下新建solr-home文件夹 
	5. 将solr-5.5.0\server\solr目录下的所有文件复制到tomcat\solr-home目录
	6. 修改tomcat\\webapps\solr\WEB-INF\web.xml文件,指定solr-home目录
	```
	<!--取消注释，并配置solr-home-->
	<env-entry>
       <env-entry-name>solr/home</env-entry-name>
       <env-entry-value>F:\solr\tomcat\solr-home</env-entry-value>
       <env-entry-type>java.lang.String</env-entry-type>
    </env-entry>
	```
	7. 启动tomcat，输入地址[http://localhost:8080/solr/admin.html](http://localhost:8080/solr/admin.html "http://localhost:8080/solr/admin.html")
	8. 在tomcat\solr-home\目录下新建core
	9. 将tomcat\solr-home\configsets\basic_configs\conf文件夹复制到tomcat\solr-home\core目录下
	10. 重启tomcat，打开浏览器输入网址[http://localhost:8080/solr/admin.html](http://localhost:8080/solr/admin.html "http://localhost:8080/solr/admin.html")
	11. 添加core
	![](2.jpg)

## 4. solr的相关配置文件
- managed-schema配置文件
	1. type部分
		> fieldType 是一些常见的可重用定义，定义了 Solr（和 Lucene）如何处理 Field。也就是添加到索引中的xml文件属性中的类型，如int、text、date等。	
		```
		<fieldType name="tint" class="solr.TrieIntField" precisionStep="8" positionIncrementGap="0"/>
	    <fieldType name="tfloat" class="solr.TrieFloatField" precisionStep="8" positionIncrementGap="0"/>
	    <fieldType name="tlong" class="solr.TrieLongField" precisionStep="8" positionIncrementGap="0"/>
	    <fieldType name="tdouble" class="solr.TrieDoubleField" precisionStep="8" positionIncrementGap="0"/>
		```
	2. fields部分
		> filed是你添加到索引文件中出现的属性名称
		```
		<field name="_root_" type="string" indexed="true" stored="false"/>
		<dynamicField name="*_i"  type="int"    indexed="true"  stored="true"/>
		<!-- 复制field 从source到dest 分词前复制 经过不同的分词器可以得到不同的分词结果 -->
		<copyField source="title" dest="text"/>
		```
	3. 其它配置
		a. uniqueKey：唯一键，这里配置的是上面出现的fileds，一般是id、url等不重复的。在更新、删除的时候可以用到
		b. defaultSearchField：默认搜索属性，如q=solr就是默认的搜索那个字段
		c. solrQueryParser：查询转换模式，是并且还是或者（AND/OR必须大写）  
- solrconfig.xml
	> solr的核心配置文件,该配置文件主要定义了solr的一些处理规则，包括索引数据的存放位置，更新，删除，查询的一些规则配置

	1. dataDir 
		```
		<!--定义了索引数据和日志文件的存放位置-->
		<dataDir>${solr.data.dir:d:/Server/Solr/data}</dataDir>
		```
	2. luceneMatchVersion 
		```
		<!--solr底层使用的是lucene4.8-->
		<luceneMatchVersion>4.8</luceneMatchVersion>	
		```
	3. directoryFactory 索引存储方案 默认为NRTCachingDirectoryFactory
	4. codecFactory 编解码工厂  定义了索引格式工具
	5. indexconfig  索引配置
	6. updateHandler 更新器
	7. query
	8. requestDispatcher 请求转发器
	9. requestHandler 请求处理器
		> 输入的请求会通过请求中的路径被转发到特定的处理器
	
		```
		<!--SearchHandler基本的请求处理器是SearchHandler,它提供一系列SearchComponents,通过multiple shards支持分布式-->
		<requestHandler name="/select" class="solr.SearchHandler">
	    <!-- default values for query parameters can be specified, these
	         will be overridden by parameters in the request
	      -->
	     <lst name="defaults">
	       <str name="echoParams">explicit</str>
	       <int name="rows">10</int>
	     </lst>
	
	    </requestHandler>
		```

## 5. solr整合中文分词器（IK、mmseg4j、smartcn）
- IK中文分词器
	1. 将ikanalyzer-solr5\ik-analyzer-solr5-5.x.jar复制到tomcat\webapps\solr\WEB-INF\lib目录
	2. 修改solr-home\baizhi\conf\managed-schema，新增
		```
		<!--添加域 并使用ik分词-->
		<field name="name" type="text_ik" indexed="true" stored="true"/>
		
		<!-- 添加的IK分词 -->
	    <fieldType name="text_ik" class="solr.TextField">   
	      <analyzer type="index" isMaxWordLength="false" class="org.wltea.analyzer.lucene.IKAnalyzer"/>   
	      <analyzer type="query" isMaxWordLength="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/>   
	    </fieldType>	
		```
	3. ikanalyzer-solr5\目录下的IKAnalyzer.cfg.xml、ext.dic、stopword.dic文件拷贝到tomcat\webapps\solr\WEB-INF\classes
	4. 重启tomcat
	5. 测试中文分词
		![ik中文分词测试](4.jpg)
- mmseg4j中文分词器
	1. 将mmseg4j-core-1.10.0.jar、mmseg4j-solr-2.3.0.jar复制到tomcat\webapps\solr\WEB-INF\lib目录
	2. 修改solr-home\baizhi\conf\managed-schema，新增
		```
		<!--测试mmseg4j-->
    	<field name="mmseg4j_name" type="textComplex" indexed="true" stored="true"/>
		
		<!--mmseg4j中文分词器-->
	    <fieldtype name="textComplex" class="solr.TextField" positionIncrementGap="100">
	      <analyzer>
			<!--
				dicPath 参数 － 设置自定义的扩展词库，支持相对路径(相对于 solr_home)
				mode 参数 － 分词模式
			-->
	        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="dic"/>
	      </analyzer>
	    </fieldtype>
	    <fieldtype name="textMaxWord" class="solr.TextField" positionIncrementGap="100">
	      <analyzer>
	        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" />
	      </analyzer>
	    </fieldtype>
	    <fieldtype name="textSimple" class="solr.TextField" positionIncrementGap="100">
	      <analyzer>
	        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="dic" />
	      </analyzer>
	    </fieldtype>
		```
	3. 自定义扩展词,名字必须为**wordsXXX.dic**
		```
		<tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="F:\solr\apache-tomcat-8.5.15\webapps\solr\WEB-INF\classes\dic" />
		```
	4. 重启tomcat
	5. 测试中文分词
		![](3.jpg)
- smartcn中文分词（同理）

## 6. 使用solr的dataimport导入数据库数据
- full-import（全部导入）
	1. 首先找到solr-5.5.0\dist\solr-dataimporthandler-5.5.0.jar、solr-dataimporthandler-extras-5.5.0.jar，把这个文件复制到tomcat\webapps\solr\WEB-INF/lib/下，并且找到相应数据库的驱动包，也同样放到该目录。这里用的是mysql的驱动包
	2. 将solr-5.5.0\example\example-DIH\solr\db\conf\db-data-config.xml复制到复制到solr-home/baizhi/conf/下，并改名为data-config.xml
	3. 修改tomcat\solr-home\baizhi\conf\solrconfig.xml,添加内容
		```
		<!--配置数据库导入-->
	  	<requestHandler name="/dataimport" class="solr.DataImportHandler">
		    <lst name="defaults">
		      <str name="config">data-config.xml</str>
		    </lst>
	  	</requestHandler>
		```
	4. 打开并编辑data-config.xml
		```
		<dataConfig>
			<!-- 这是mysql的配置，学会jdbc的都应该看得懂 -->
		    <dataSource driver="com.mysql.jdbc.Driver" url="jdbc:mysql://localhost:3306/lucene" user="root" password="root" />
		    <document>
				<!-- name属性，就代表着一个文档，可以随便命名 -->
        		<!-- query是一条sql，代表在数据库查找出来的数据 -->
		        <entity name="user" query="select * from t_user">
				    <!-- 每一个field映射着数据库中列与文档中的域，column是数据库列，name是solr的域(必须是在managed-schema文件中配置过的域才行) -->
		            <field column="id" name="id" />
		            <field column="name" name="mmseg4j_name"/>
		            <field column="last_update_time" name="last_update_time_dt"/>
		        </entity>
		    </document>
		</dataConfig>
		```
	5. 重启tomcat
	6. 测试
		![](5.jpg)
- delta-import（增量导入）
	> 全量导入在数据量大的时候代价非常大，一般来说都会使用增量的方式来导入数据（将最新的数据进行导入）
	
	1. 修改配置文件data-config.xml
		```
		<dataConfig>
		    <dataSource driver="com.mysql.jdbc.Driver" url="jdbc:mysql://localhost:3306/lucene" user="root" password="root" />
		    <document>
		        <entity name="user"
		                query="select * from t_user"
						<!--查询增量数据id-->
		                deltaQuery="select id from t_user where last_update_time &gt; '${dataimporter.last_index_time}'"
						<!--根据增量数据id，查询增量数据信息-->
		                deltaImportQuery="select * from t_user where id = '${dataimporter.delta.id}'"
		            >
		            
		            <field column="id" name="id" />
		            <field column="name" name="mmseg4j_name"/>
		            <field column="last_update_time" name="last_update_time_dt"/>
		        </entity>
		    </document>
		</dataConfig>
		```
		> 注意：
		> 1. deltaImportQuery与deltaQuery, deltaImportQuery使用deltaQuery返回的文章id作为查询条件，然后进行增量导入。
		> 2. dataimporter.last_index_time  ：存储在文件import.properties 中
		> 3. dataimporter.delta.id：deltaImportQuery 返回的id
	2. 重启tomcat
	3. 测试
		![](6.jpg)	
- 复杂导入
	1. 一对多导入（列：唐代诗人和其诗作,一对多关系）
	2. 数据库表如下
		```
		CREATE TABLE `poets` (
		  `id` int(11) NOT NULL AUTO_INCREMENT,
		  `name` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
		  `created_at` datetime DEFAULT NULL,
		  `updated_at` datetime DEFAULT NULL,
		  PRIMARY KEY (`id`)
		) ENGINE=InnoDB AUTO_INCREMENT=2529 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
			
		CREATE TABLE `poetries` (
		  `id` int(11) NOT NULL AUTO_INCREMENT,
		  `poet_id` int(11) DEFAULT NULL,
		  `content` text COLLATE utf8_unicode_ci,
		  `title` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
		  `created_at` datetime DEFAULT NULL,
		  `updated_at` datetime DEFAULT NULL,
		  PRIMARY KEY (`id`)
		) ENGINE=InnoDB AUTO_INCREMENT=13220 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
		```
	3. solr配置如下
		data-config.xml
		```
		<dataConfig>
		    <dataSource driver="com.mysql.jdbc.Driver" url="jdbc:mysql://localhost:3306/lucene" user="root" password="root" />
		    <document>
		        <entity name="poetries" 
		                query="select * from poetries"
		                deltaQuery="select id from poetries where updated_at &gt; '${dataimporter.last_index_time}'"
		                deltaImportQuery="select * from poetries where id = '${dataimporter.delta.id}'"
		         >
		            <field column="id" name="id"/>
		            <field column="content" name="poetries_content"/>
		            <field column="title" name="poetries_title"/>
		            <field column="created_at" name="poetries_created_at"/>
		            <field column="updated_at" name="poetries_updated_at"/>
		            
		            <entity name="poet"
		                query="select * from poets where id=${poetries.poet_id}" 
		                deltaQuery="select id from poets where updated_at &gt; '${dataimporter.last_index_time}'"
		                deltaImportQuery="select * from poets where id = '${dataimporter.delta.id}'"
		            >
		            
		                <field column="id" name="poet_id" />
		                <field column="name" name="poet_name"/>
		                <field column="created_at" name="poet_created_at"/>
		                <field column="updated_at" name="poet_updated_at"/>
		            </entity>
		        
		         </entity>
		    </document>
		</dataConfig>
		```
		managed-schema
		```
		<!--诗人-->
		<field name="poet_id" type="string" indexed="true" stored="true" multiValued="false"/>
		
		<field name="poet_name" type="textComplex" indexed="true" stored="true"/>
		
		<field name="poet_created_at" type="date" indexed="true" stored="false"/>
		
		<field name="poet_updated_at" type="date" indexed="true" stored="true"/>
		
		<!--诗集-->
		
		<field name="poetries_content" type="textComplex" indexed="true" stored="true"/> 
		
		<field name="poetries_title" type="textComplex" indexed="true" stored="true"/>
		
		<field name="poetries_created_at" type="date" indexed="true" stored="false"/>
		
		<field name="poetries_updated_at" type="date" indexed="true" stored="true"/>
		```
	4. 导入数据
		![](7.jpg)
	5. 测试
		![](8.jpg)

## 7. 查询参数
- 常用查询参数
	1. q - 查询字符串，如果查询所有\*:\* (id:1)
	2. fq - (filter query)使用Filter Query可以充分利用FilterQuery Cache，提高检索性能。作用：在q查询符合结果中同时是fq查询符合的，例如：q=mm&fq=date_time:[20081001TO 20091031]，找关键字mm，并且date_time是20081001到20091031之间的。（fq查询字段后面的冒号和关键字必须有）
	3. sort - 排序，格式：sort=<field name>+<desc|asc>[,<field name>+<desc|asc>]… 。示例：（score desc, price asc）表示先 “score” 降序, 再 “price” 升序，默认是相关性降序
	4. start - 用于分页定义结果起始记录数，默认为0，从第1条记录开始。
	5. rows - 用于分页定义结果每页返回记录数，默认为10。
	6. fl - field list，指定返回结果字段。以空格“ ”或逗号“,”分隔。
	7. df - 默认的查询字段，一般默认指定。
	8. wt - writer type。指定查询输出结构格式，输出格式：xml、json、Python、ruby、PHP、phps、custom
	9. indent - 返回的结果是否缩进，默认关闭，用indent=true|on 开启，一般调试json,php,phps,ruby输出才有必要用这个参数。
	10. debugQuery - 设置返回结果是否显示Debug信息
	11. q.op - 表示q中查询语句的各条件的逻辑操作 AND(与)OR(或)
	12. hl - 是否高亮 ,如hl=true
	13. hl.fl - 高亮field ,hl.fl=Name,SKU
	14. hl.snippets - 默认是1,这里设置为3个片段
	15. hl.simple.pre - 高亮前面的格式
	16. hl.simple.post - 高亮后面的格式
	17. facet - 是否启动统计
	18. facet.field - 统计field
- 常用查询语法
	1. “:” 指定字段查指定值，如返回所有值*:*
	2. “?” 表示单个任意字符的通配
	3. “\*” 表示多个任意字符的通配（不能在检索的项开始使用*或者?符号）
	4. “~” 表示模糊检索，如检索拼写类似于”roam”的项这样写：roam~将找到形如foam和roams的单词；roam~0.8，检索返回相似度在0.8以上的记录。
	5. 邻近检索，如检索相隔10个单词的”apache”和”jakarta”，”jakarta apache”~10
	6.  “^” 控制相关度检索，如检索jakarta apache，同时希望去让”jakarta”的相关度更加好，那么在其后加上”^”符号和增量值，即jakarta^4 apache
	7. 布尔操作符AND、&&
	8. 布尔操作符OR、||
	9. 布尔操作符NOT、!、- （排除操作符不能单独与项使用构成查询）
	10. “+” 存在操作符，要求符号”+”后的项必须在文档相应的域中存在
	11. ( ) 用于构成子查询
	12. [] 包含范围检索，如检索某时间段记录，包含头尾，date:[200707 TO 200710]

## 8. SolrJ的使用
- 引入MAVEN的依赖，solrj的版本号要对应solr的版本号
	```
	<!-- https://mvnrepository.com/artifact/org.apache.solr/solr-solrj -->
    <dependency>
        <groupId>org.apache.solr</groupId>
        <artifactId>solr-solrj</artifactId>
        <version>5.5.0</version>
    </dependency>
	```
- 新增和修改索引
	```
	private static String serverUrl = "http://localhost:8080/solr/baizhi";

    /**
     * 新增和修改索引
     *   修改索引同理：索引存在修改，不存在新增
     */
    @Test
    public void addOrUpdateIndex() {

        try {
            SolrClient solrClient = new HttpSolrClient(serverUrl);

            /*
            SolrInputDocument solrInputDocument = new SolrInputDocument();

            solrInputDocument.addField("id","1");
            solrInputDocument.addField("poetries_title","李白");

            solrClient.add(solrInputDocument);
            */

            ArrayList<Poetry> poetries = new ArrayList<Poetry>();
            for (int i = 0; i < 10; i++) {
                Poetry poetry = new Poetry(i+1,"上课头疼的故事","头好痛",new Date(),new Date());
                poetries.add(poetry);
            }

            // 注意：bean需要使用@Field注解,绑定属性和其索引域Field
            solrClient.addBeans(poetries);

            solrClient.commit();
            solrClient.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
	```
- 删除索引
	```
	/**
     * 删除索引
     */
    @Test
    public void deleteAllIndex(){
        try {
            SolrClient solrClient = new HttpSolrClient(serverUrl);
            //1. 删除一个
            //solrClient.deleteById("1");

            //2. 删除多个
            solrClient.deleteById(Arrays.asList("1","2"));

            //3. 根据查询条件删除
            solrClient.deleteByQuery("id:10");

            //4. 删除所有索引
            //solrClient.deleteByQuery("*:*");
            solrClient.commit();
            solrClient.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
	```
- 查询索引
	```
	/**
     * 查询索引
     */
    @Test
    public void searchIndex(){
        try {
            HttpSolrClient httpSolrClient = new HttpSolrClient(serverUrl);
            /*
            SolrDocument solrDocument = httpSolrClient.getById("3");

            Object poetries_title = solrDocument.get("poetries_title");

            System.out.println(poetries_title);
            */
            // 设置查询参数
            SolrQuery solrQuery = new SolrQuery();
            solrQuery.set("q","poetries_content:上课");
            solrQuery.set("start ",0);
            solrQuery.set("rows",20);

            // 过滤查询 id范围在3到6之间
            solrQuery.set("fq","id:[3 TO 6]");
            // 查询结果 根据id倒序排列 多个排序条件用逗号隔开
            solrQuery.set("sort","id desc");

            // 默认搜索域
            solrQuery.set("df","poetries_title");

            // 设置高亮
            solrQuery.setHighlight(true);
            // 设置高亮域
            solrQuery.addHighlightField("poetries_title");
            solrQuery.addHighlightField("poetries_content");

            solrQuery.setHighlightSimplePre("<font style='color:red'>");
            solrQuery.setHighlightSimplePost("</font>");

            // 查询
            QueryResponse queryResponse = httpSolrClient.query(solrQuery);

            // 获取查询结果
            SolrDocumentList results = queryResponse.getResults();

            System.out.println("查询出来的结果数："+results.size());
            System.out.println("共查询到记录："+results.getNumFound());

            // 获取高亮结果
            Map<String, Map<String, List<String>>> highlighting = queryResponse.getHighlighting();

            // 处理高亮结果集
            for (String s : highlighting.keySet()) {
                Map<String, List<String>> map = highlighting.get(s);
                System.out.println(map);
            }

            // 处理普通结果集
            for (SolrDocument result : results) {
                System.out.println(result.get("id")+"  "+result.get("poetries_title")+" "+result.get("poetries_content"));
            }

            httpSolrClient.close();
        } catch (SolrServerException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
	```
> SolrCloud未完待续
