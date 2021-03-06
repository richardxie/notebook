# MongoDB

Document Oriented DB

## 基础

### 特点

- 灵活性（Flexibility）
- 灵活的查询模型（Flexible Query Model）
- 本机聚合（Native Aggregation）
- 无模式模型（Schemaless Model）

### 使用场景

- 文档存储： BI（上限集合）
- 物联网：第业务价值数据
- 移动应用: 基于地理位置服务（LBS）
- 个性化
- 目录管理
- 内容管理 （分布，复制）

### 最佳实践

- 启动Journaling日志

### 建模

- 一对一（很少）且不需要单独访问内嵌内容的情况下，使用嵌入式建模
- 一对多，通过数组方式引用多端
- 一对非常多，将一端的引用嵌入多端对象

嵌入

```json
db.Person.findOne()
// 一对一
{
	"_id": ObjectID("590..."),
	"name": "alex",
	"address": {
		"city": "shanghai",
		"location": "GLXJ29"
	}
}
// 一对多
{
	"_id": ObjectID("590..."),
	"name": "alex",
	"address": [{
		"city": "shanghai",
         "desc": "home",
         "postcode": 200233,
		"location": "GLXJ29"
		},
         {
           "city": "shanghai",
         	"desc": "work",
         	"postcode": 202200,
			"location": "GHL55"         
         }
   ]
}
```

引用

```json
//一对多
person = db.Person.findOne({"name":"alix"});
addresses = db.Addresses.find({"_id":{'$in':person.addresses}})
{
	"_id": ObjectID("5900..."),
	"name": "alex",
	"addresses": [
        ObjectID("59011..."),
        ObjectID("59012..."),
    ]
}

{
	"_id": ObjectID("59011..."),
	"city": "shanghai",
   	"desc": "work",
   	"postcode": 202200,
	"location": "GHL55" 
}

person = db.Person.findOne({"name":"alix"});
addresses = db.Addresses.find({"person": person._id})
//多对多
{
	"_id": ObjectID("59011..."),
    "person": ObjectID("5900..."),
	"city": "shanghai",
   	"desc": "work",
   	"postcode": 202200,
	"location": "GHL55" 
}


```

## 索引

B数

- 单字段索引（Single Field Index）

  db.person.createInex({city:1}) //city升序索引

  db.books.createIndex({"meta_data.page_count":1}) //嵌入字段索引

- 复合索引 （Compound Index）

  db.books.createIndex({"name":1, "isbn":1})

  - 排序

    db.books.find().sort("name":1, "isbn":1) //使用索引

    db.books.find().sort("name":1, "isbn":-1) //不使用索引

  - 复用

    db.books.find({"name":1});  //使用索引

    db.books.find({"isbn":1});  //不要索引

  - 前缀索引

    套娃模式

- 多键索引

  索引数组

- 文本索引

  用于支持文本搜索，最多一个文本索引

- TTL索引

- 哈希索引

## 复制

主从逻辑（Master-Slave）复制

- 配置

  1Master 1Slave 1Arbiter

  - mongo.config

  ```
  processManagement:
     fork: true
  net:
     bindIp: 127.0.0.1,192.168.10.110,10.9.0.1,14.17.97.226 #绑定IP
     port: 27017 #端口
  storage:
     dbPath: /mnt/raid5/server/data/mongodb #数据文件地址
     journal:
        enabled: true
  systemLog:
     destination: file
     path: "/mnt/raid5/server/logs/mongodb/mongodb.log" #日志文件地址
     logAppend: true
     logRotate: rename
  replication:
     replSetName: rs01
     oplogSizeMB: 1000
  ```
  - 启动3个mongo实例

    mongod -f mongo.config

  - 配置副本集

    use admin

    cfg={ _id:"rs01", members:[ {_id:0,host:'192.168.10.110:27017',priority:2}, {_id:1,host:'192.168.10.111:27017',priority:1},   
    {_id:2,host:'192.168.10.112:27017',arbiterOnly:true}] }

    rs.initiate(cfg)

  - 查看状态

    rs.status(）



## 分片

单台服务器无法分片

- 分片集群
  - 两个或多个分片， 且分片是副本集
  - 一个或多个查询路由器（mongos）
  - 配置服务器（Config Server），副本集存储整个集群的元数据和配置信息
- 选择正确的分布键
  - 高基数
  - 低频率
  - 非单调变化
- 配置

## Spring mongo

```java
@Configuration
public class AppConfig {   
    /*
    *Use the standard Mongo driver API to create a com.mongodb.client.MongoClient instance.   
    */
    public @Bean MongoClient mongoClient() {       
        return MongoClients.create("mongodb://localhost:27017");   
    } 
    
	public @Bean MongoTemplate mongoTemplate() {
    	return new MongoTemplate(mongoClient(), "mydatabase");
  	}
}
```

```java
public class MongoApp {

private static final Log log = LogFactory.getLog(MongoApp.class);

  public static void main(String[] args) {

    MongoOperations mongoOps = new MongoTemplate(new SimpleMongoClientDbFactory(MongoClients.create(), "database"));
    
    Person p = new Person("Joe", 34);
    
    // Insert is used to initially store the object into the database.
    mongoOps.insert(p);
    log.info("Insert: " + p);
    
    // Find
    p = mongoOps.findById(p.getId(), Person.class);
    log.info("Found: " + p);
    
    // Update
    mongoOps.updateFirst(query(where("name").is("Joe")), update("age", 35), Person.class);
    p = mongoOps.findOne(query(where("name").is("Joe")), Person.class);
    log.info("Updated: " + p);
    
    // Delete
    mongoOps.remove(p);
    
    // Check that deletion worked
    List<Person> people =  mongoOps.findAll(Person.class);
    log.info("Number of people = : " + people.size());


    mongoOps.dropCollection(Person.class);

  }
}
```

