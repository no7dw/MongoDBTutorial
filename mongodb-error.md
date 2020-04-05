title: mongodb_error
date: 2016-01-15 12:33:10
tags:
---

查询用户条件:  {}
err:  MongoError: Executor error: Overflow sort stage buffered data usage of 33554872 bytes exceeds internal limit of 33554432 byt
es

http://stackoverflow.com/questions/27023622/overflow-sort-stage-buffered-data-usage-exceeds-internal-limit

there is a limit of sort in memory:32MB

### solution:index the sort field


Indexing the sort field allows MongoDB to stream documents to you in sorted order, rather than attempting to load them all into memory on the server and sort them in memory before sending them to the client.


https://docs.mongodb.org/manual/reference/limits/#Sorted-Documents

If MongoDB cannot use an index to get documents in the requested sort order, the combined size of all documents in the sort operation, plus a small overhead, must be less than 32 megabytes.
