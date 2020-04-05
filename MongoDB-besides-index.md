### 索引该加的都加了，还不够快。怎么办？

(前文)[https://github.com/no7dw/no7dw.github.io/blob/master/source/_posts/MongoDB-Best-Proactices-About-Index.md] 末尾提及过这个 索引该加的都加了，还不够快。怎么办？

首先
不是建立了index 就完事了. 写query 要注意：
  - table size
  - growing trend 
  - cardinality (稀疏问题) 

然后决定index的选择。

另外不是DB 每次都会选择到最优的索引，要跟进自己的实际情况去挑选（如果DB的winning plan 非最佳），使用 hint 来指定索引.


再次
遇到过几种case：

 - 每次从DB拿部分数据，不断for loop step ,skip & limit. -- 为了避免每次拿太多数据
 - or 定时任务， 每次从DB拿部分数据进行处理 。（例子每1 或 5分钟，拿 {status: pending} 的order 与第三方进行check 状态，以保证协调状态的一致）

这些可以考虑是读DB 数据到cache (redis or mq), 数据推送到DB同时更新到cache。 -- 目的是为了减少DB的查询，减少DB 查询的量（次数&取的数据量）

注意large skip limit 对DB 是有害的。参考 其他文章google 为什么不做很大的分页查询。
[DB large skip bad to use](https://ios.develop-bugs.com/article/19535500/Why+is+MongoDB's+skip()+so+slow+and+bad+to+use%2C+while+MySQL's+LIMIT+is+so+fast%3F)
[large skip limit 取巧法](https://stackoverflow.com/questions/1243952/how-can-i-speed-up-a-mysql-query-with-a-large-offset-in-the-limit-clause)

还碰到过，为了显示类似阅读量，销售量，每次详情、摘要，都要过来查count ！！！ -- 真的十分大压力。
典型做法：
 - 另外保存一个计数器，而不在原始记录里面count
 - 存在cache中 $inc，而不再DB层进行count (原始记录写没问题)
 - 粗算法，很多业务其实不需要精确的给用户，销售量 12,345 跟 12,344 对买家是没有区别的。(但对卖家有区别, 但两者对应系统不一样了)。想想 微信阅读数 10万+ , 1.4万 的显示，而非直接显示阅读数。

