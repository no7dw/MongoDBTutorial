title: mongodb-repl-url
date: 2016-03-16 23:53:22
tags:
---
# mongodb 集群的orm之waterline 稳定用法
---

ORM waterline 在sails.js 中的配置见下：

    prodMongodb: {
    module: 'sails-mongo',
    
    host: process.env.MONGO_HOST  || 'localhost',
    port: process.env.MONGO_PORT  || 27017,
    user: 'username',
    password: 'password',
    database: 'DBName',

    // Replica Set (optional)
    replSet: {
      servers: [
        {   
          host: 'host1',
          port: 27017 // Will override port from default config (optional)
        },  
        {   
          host: 'host2',
          port: 27017
        },  
        {   
          host: 'host3',
          port: 27017
        }   
      ],  
      // See http://mongodb.github.io/node-mongodb-native/api-generated/replset.html (optional)
      options: {
        poolSize: 1,
        reconnectWait: 5 * 60 * 1000,
        retries: 10, 
        socketOptions: {
          keepAlive: 0,
          connectTimeoutMS: 5 * 60 * 1000,
          socketTimeoutMS: 5 * 60 * 1000
        }
      }
    }
  
  
  
  但是实际这算是指定了集群的host1作为 primary， 当这些节点因为其他某些问题重新选举时，会造成连接不上的问题，程序会抛出错误。
  
###  正确的做法直接使用url 属性：

     prodMongodb: {
    module: 'sails-mongo',
    url:"mongodb://username:password@host1:27017,host2:27017,host3:27017/dbName?readPreference=secondaryPreferred"
    ...
    
 
测试后，集群primary 节点变化 ，不会crash 程序。
 
