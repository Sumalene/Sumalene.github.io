---
layout: single
title: "[BigData]MongoDB学习"
categories: [DB]
last_modified_at: 2024-04-08
excerpt: " just a record."
header:
  teaser: https://static.runoob.com/images/mix/life-code-typography-hd-wallpaper-1920x1080-7168.jpg
---


## Mongodb学习

Mongodb是一个开源的、面向文档的、非关系型数据库，它是由C++语言编写的，旨在为开发者提供高性能、高可用性、易扩展的数据库。  
-- MongoDB 介于关系数据库和非关系数据库之间，是非关系数据库当中功能最丰富，最像关系数据库的。他支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。 

Mongo最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。  

### Mongo Shell

```JavaScript
 
//-- Mongodb的通用命令包括：show dbs、use dbname、db、show collections、
// db.collection.find()、db.collection.insert()、db.collection.update()、db.collection.remove()  
  
use test //-- 切换数据库,此时不需要存在数据库  
  
db.createCollection("student") //-- 创建集合  
  
db.student.insert({"name": "Kasumi", "age": 18}) //-- 插入文档 （其实不需要上面的创建集合，直接插入文档就会自动创建集合）  
  
//-- 循环插入文档  
for (var i = 0; i < 10; i++) {  
    db.student.insert({"name": "Kasumi" + i, "age": 18 + i})  
}  
  
db.student.find({"name": "Kasumi"}) -- 查询文档  
  
db.student.update({"name": "Kasumi"}, {"name": "Kasumi", "age": 19}) -- 更新文档  
  
//-- 多条件查询，与关系（或关系）查询，大于（小于）查询，降序（升序）查询，limit+skip分页  
//-- 查询计算机专业成绩在90分以上（包含），年龄为19-20岁或小于15岁的女生信息，按成绩降序排序，取第5-10条数据  
db.student.find(  
    {  
        "major": "computer",  
        "score": {"$gte": 90},  
        "$or": [  
            {"age": {"$gte": 19, "$lte": 20}},  
            {"age": {"$lt": 15},}  
        ]  
    }  
).sort({"score": -1}).limit(10).skip(5)  
  
  
db.student.find({"name": "Kasumi"}).count() //-- 查询文档数量  
  
db.student.remove({"name": "Kasumi"}) //-- 删除文档  
  
db.student.drop() //-- 删除集合  
  
db.dropDatabase() //-- 删除数据库  


```
  

  
>  mongodb的Shell命令本质上是JavaScript，所以可以直接使用JavaScript的语法，如for循环、if判断等。


---

### Java API编程


导入驱动JAR

```java

import com.mongodb.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

public class SimpleMongoDBExample {
    public static void main(String[] args) {
        // 创建MongoClient对象连接到本地的MongoDB服务器
        MongoClient mongoClient = new MongoClient("localhost", 27017);

        MongoDatabase database = mongoClient.getDatabase("test");

        // 使用MongoDatabase对象获取集合
        MongoCollection<Document> collection = database.getCollection("student");

        // 创建一个Document对象
        Document document = new Document("name", "Kasumi").append("age", 18);

        // insertOne方法插入文档
        collection.insertOne(document);

        System.out.println("Document inserted successfully");
    }
}

```

---

### JavaScript 直接编程


```JavaScript

// 导入 MongoDB、
var MongoClient = require('mongodb').MongoClient;

// 定义 MongoDB 连接字符串
var url = "mongodb://localhost:27017/";

// 连接
MongoClient.connect(url, function(err, db) {
  if (err) throw err;

  var dbo = db.db("test");//选择

  // 查询 "student" 集合
  dbo.collection("student").find({}).toArray(function(err, result) {
    if (err) throw err;

    // 打印查询结果
    console.log(result);

    db.close();
  });
});

```
