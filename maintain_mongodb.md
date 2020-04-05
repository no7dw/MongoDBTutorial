title: 将已有的集合转化为固定集合碰到的问题 
---
将已有的集合转化为固定集合
    db.runCommand({convertToCapped:"test",size:10000});
    
    //check
    db.test.state()
    //isCap:true
    
转的时候size 是collection的大小，
另外创建的时候可以制定，最大的documents size （max 属性）

    db.createCollection("collect",{capped:true, size:10000, max:20});

坑：创建后，手动create >20 条纪录时，会出现先进先出的队列效果。

    > db.createCollection("testC",{capped:true, size:100, max:2});
    { "ok" : 1 }
    > db.testC.insert({'name':'wade', 'time':new Date().toLocaleTimeString()})
    WriteResult({ "nInserted" : 1 })
    > db.testC.find({})
    { "_id" : ObjectId("55d5fdf2c49e250bbf9e4770"), "name" : "wade", "time" : "00:18:58" }
    > db.testC.insert({'name':'wade', 'time':new Date().toLocaleTimeString()})
    WriteResult({ "nInserted" : 1 })
    > db.testC.find({})
    { "_id" : ObjectId("55d5fdf2c49e250bbf9e4770"), "name" : "wade", "time" : "00:18:58" }
    { "_id" : ObjectId("55d5fe02c49e250bbf9e4771"), "name" : "wade", "time" : "00:19:14" }
    > db.testC.insert({'name':'wade', 'time':new Date().toLocaleTimeString()})
    WriteResult({ "nInserted" : 1 })
    > db.testC.find({})
    { "_id" : ObjectId("55d5fe02c49e250bbf9e4771"), "name" : "wade", "time" : "00:19:14" }
    { "_id" : ObjectId("55d5fe08c49e250bbf9e4772"), "name" : "wade", "time" : "00:19:20" }
    > 


----------
offline simulating


    > db.article.find({}).count()
    6
    > db.runCommand({convertToCapped:"article",size:1000});
    { "ok" : 1 }
    > db.article.insert({ 'title':'article title','name':'wade', 'time':new Date().toLocaleTimeString()})
    WriteResult({ "nInserted" : 1 })
    > db.article.find({}).count()
    7


----------
reach it's limit

   

    > while(i<100){db.article.insert({ 
    'title':'article title','name':'wade', 'time':new Date().toLocaleTimeString()}) ; i++}
    > db.article.find({}).count()
    37
    > while(i<1000){db.article.insert({ 'title':'article title','name':'wade', 'time':new Date().toLocaleTimeString()}) ; i++}    
    999
    > db.article.find({}).count()
    37
    > db.article.insert({ 'title':'article title','name':'wade123', 'time':new Date().toLocaleTimeString()})
    WriteResult({ "nInserted" : 1 })
    > db.article.find({}).sort({time:-1})
    { "_id" : ObjectId("55d6092af311196af3d3c3b2"), "title" : "article title", "name" : "wade123", "time" : "01:06:50" }
    { "_id" : ObjectId("55d60911f311196af3d3c3b1"), "title" : "article title", "name" : "wade123", "time" : "01:06:25" }
    { "_id" : ObjectId("55d608d5f311196af3d3c395"), "title" : "article title", "name" : "wade", "time" : "01:05:25" }


----------
结果看起来work

    > db.article.find({}).count()
    37

around 16:10 - 16:40
   - 线下运行ok，但在线上发现，超过了大小，再也新增不了。报错：


    > { [MongoError: exception: failing update: objects in a capped ns cannot grow] ...}

   - 尝试将原的size增加，看看能否插新数据，还是不行。
   - 尝试 undo  capped , stackoverflow 后发现只有rename ,and recreate data 的方法。由于创建大量数据会锁库很严重。于是紧紧rename 了这个数据库（没有旧数据关系不大，所以这么做）


at last, 做完db 操作，要跑测试代码.


