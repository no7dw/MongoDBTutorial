title: watch new modify in mongodb
date: 2016-04-08 12:26:58
tags:
---
### 需求：系统update 数据时，要记录原始数据&改动的数据。
类似jira 一样，知道改动前的原数据，改动后的数据

 - orm's hook: beforeupdate, afterupdate, 但是native的更新无法捕获
 - new function: 写一个公共的function，每次更新上面的表的时候调用，需要找到所有更新这些表的代码，并且以后新写代码都要记得加上．
 - oplog: 使用[mongo-oplog][1]模块，可以监听某张表的插入更新操作，无需每个地方都改动。

第三个方法入侵性最少，效果最好，对性能基本无影响。

  [1]: https://github.com/cayasso/mongo-oplog
