title: mongodb-notice
date: 2015-10-27 23:13:23
tags:
---
## 使用mongodb注意事项  ##  

 - mongodb 会lock db
 - multi write: 注意速度，一般而言，以下是速度正常参考值：
     - query 0-10 ms  
     - write 0-100ms
 - transaction : 
   - two phase commit （官方推荐，－－ **没回滚，然并卵**）; 
   - mongoose-transaction


