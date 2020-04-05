### MongoDB主库的 Global Lock 异常高

事件：主库的 Global lock 异常高， 代码没做啥变动。
时间：2019-12-21 3：00AM左右。
DB 配置：16核64G， 500G  数据，三节点 , V4.0 . IOPS 1.6W 

现状见图：
![2020-04-05-15-45-45](https://imgs.no1token.com/2020-04-05-15-45-45.png)

serverStatus

![2020-04-05-15-46-50](https://imgs.no1token.com/2020-04-05-15-46-50.png)
![2020-04-05-15-47-15](https://imgs.no1token.com/2020-04-05-15-47-15.png)

currentOp 里面 "waitingForLock" : true, 的语句
https://gist.github.com/no7dw/3a7303eb2776063ee7017c5074719163 
- 少量multiple update
- 4个COLLSCAN , 其中一个collection 近百万，但非最近增长特别快
    - 之前没出事

当时Primary & Secondary CPU、mem % 都 不高。

处理：
- 尝试kill currentOp (getMore, query , agg) https://github.com/no7dw/mongodb-slow-query/blob/master/killslow.js 
    - 没有对update 操作进行kill  —— 问题没解决
    - 对update 也kill的 —  问题没解决  
- 没信心快速恢复—》重启了DB 实例，临时恢复了。


PS：阿里云的审计日志出问题，暂时拿不到当时的审计日志。待跟踪

参考了：
https://yq.aliyun.com/articles/703395?spm=a2c4e.11155435.0.0.2ee55554IcwIkS 
https://yq.aliyun.com/articles/201983?spm=a2c4e.11155435.0.0.2cd05554ai0AHB
https://docs.mongodb.com/manual/faq/concurrency/
https://stackify.com/mongodb-performance-tuning/

几个点：
![2020-04-05-15-49-10](https://imgs.no1token.com/2020-04-05-15-49-10.png)
![2020-04-05-15-49-31](https://imgs.no1token.com/2020-04-05-15-49-31.png)

张友东的文章看完，对应我们的case总结下：
- 当时没有对涉及上面截图的操作的主动调用处理。
- 对COLLSCAN 的进行了有优化
- 对慢的查询进行优化
- TODO : sort 的优化 
- 其他硬件资源的优化暂时没做。因为mem, cpu,iops 都不算高

目前：
- 对currentOp 里面的COLLSCAN 的进行了index 优化
- 等待排查对应的审计日志看对应的时间点
- global lock 加了监控

**last but not least**
发现有个账号，大量在做全量同步，从主库读到hive，停了之后，竟然再没出现。


几个后续待跟进的问题：
- MetaData 的 “W” 锁是什么意思？（官方文档很少对这个的描述）
    - 看了src code ，这个MetaData关键字貌似比较通用，但没有头绪
    — 对写入 db.system.profile  ？ 需要关注吗？
- 谁、哪个语句引起这个这个globallock 的飙升，是查数值最小的opid的语句？ (因为currentOp 没有该语句的运行时间戳 )
- kill掉所有操作(包括DB的update ) 会有助于解决这个问题？
