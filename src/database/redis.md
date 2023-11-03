# Redis

## 缓存读写策略
CacheAside Pattern
1. 更新DB
2. 删除缓存

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