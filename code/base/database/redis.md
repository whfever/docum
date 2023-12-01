# Redis

## 缓存读写策略
### CacheAside Pattern
1. 更新DB
2. 删除缓存
缺陷：
1. 缓存和DB不一致
2. 缓存和DB不一致时，缓存还在使用，可能导致缓存击穿
3. 缓存和DB不一致时，缓存还在使用，可能导致缓存雪崩

分布式锁解决方案：
1. 基于Redis的SETNX命令，如果SETNX命令返回1，则说明key不存在，则执行更新操作，并返回1；如果SETNX命令返回0，则说明key已经存在，则不执行更新操作，并返回0。
2. 基于Redis的INCR命令，如果key不存在，则执行更新操作，并返回1；如果key存在，则不执行更新操作，并返回0。
### Cache-Aside + Write-Through Pattern
Write-Through Pattern
1. 更新DB
2. 更新缓存

Write-Behind Pattern
1. 更新DB
2. 异步更新缓存

## 缓存淘汰策略
LRU（Least Recently Used）
LFU（Least Frequently Used）
FIFO（First In First Out）
LRU + LFU + FIFO

## 缓存一致性
缓存一致性问题：缓存数据与数据库数据不一致问题。

解决方案：
1. 缓存穿透：查询一个不存在的数据，导致缓存中没有，从数据库中查询，然后将结果放入缓存。
2. 缓存击穿：查询一个存在的数据，导致缓存中有，但是失效时间未到，从数据库中查询，然后将结果放入缓存。