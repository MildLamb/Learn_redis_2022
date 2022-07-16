# Learn_redis_2022
学习redis2022

# NoSQL 数据库
## NoSQL数据库概述
NoSQL（Not Only SQL），意为"不仅仅是SQL"，泛指非关系型的数据库。  
NoSQL不依赖业务逻辑方式存储，而以简单的key-value模式存储。因此大大的增加了数据库的扩展能力。  
- 不遵循SQL标准
- 不支持ACID
- 远超于SQL的性能

## NoSQL适用场景
- 对数据高并发的读写
- 海量数据的读写
- 对数据高可扩展性


# 安装redis跳转 Learn_Redis

# 相关知识介绍
## Redis是单线程+多路IO复用技术
多路复用是指使用一个线程来检查多个文件描述符(Socket)的就绪状态，比如调用select和poll函数，传入多个文件描述符，  
如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作，可以在同一个线程里执行，也可以启动  
线程执行(比如使用线程池)

# Redis过期键删除策略
- LRU是最近最少使用页面置换算法(Least Recently Used),也就是首先淘汰最长时间未被使用的页面!
- LFU是最近最不常用页面置换算法(Least Frequently Used),也就是淘汰一定时期内被访问次数最少的页!
```bash
# volatile-lru -> Evict using approximated LRU, only keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key having an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
```
