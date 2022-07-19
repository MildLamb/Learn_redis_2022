# Redis事务
- Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化，按顺序执行，事务在执行的过程中，不会被其他客户端发送的命令请求所打断。
- Redis事务的主要作用就是串联多个命令防止别的命令插队

![image](https://user-images.githubusercontent.com/92672384/179639596-1904feb7-4120-4b8a-b150-05eb660ec5b3.png)

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> flushdb
QUEUED
127.0.0.1:6379> set name mildlamb
QUEUED
127.0.0.1:6379> get name
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) "mildlamb"
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name kindred
QUEUED
127.0.0.1:6379> get name
QUEUED
127.0.0.1:6379> discard
OK
127.0.0.1:6379> 
```

## 事务的错误处理
- 组队中某个命令出现了语法错误，执行时整个命令队列都会取消
- 组队中某个命令执行出现了运行错误，则只有报错的命令不执行

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name kindred
QUEUED
127.0.0.1:6379> set age 1500
QUEUED
127.0.0.1:6379> set sex
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
```
```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name kindred
QUEUED
127.0.0.1:6379> incr name
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) ERR value is not an integer or out of range
127.0.0.1:6379> get name
"kindred"
```

# 事务锁
## 悲观锁
悲观锁(Pessimistic Lock),顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候  
都会上锁，这样别人想拿这个数据就会block直到拿到锁。传统的关系型数据库里面就用到了很多这种锁机制，比如行锁，  
表锁等，读锁，写锁等。都是在做操作之前上锁。

## 乐观锁
乐观锁(Optimistic Lock),顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但在  
更新的时候会判断一下期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读类型的应用，这样可以  
提高吞吐量。Redis就是利用这种check-and-set机制实现事务的。

## WATCH key [key...]
- 在执行multi之前，先执行watch key1 [key2...]，可以监视一个(或多个key)，如果在事务执行之前这个(或这些)中的  
  key被其他命令所改动，那么事务将被打断
  
# Redis事务三大特性
1. 单独的隔离操作
  - 事务中的所有命令都会序列化，按顺序的执行。事务在执行过程中，不会被其他客户端发送来的命令请求所打断
2. 没有隔离级别的概念
  - 队列中的命令没有提交之前都不会实际执行，因为事务提交前任何指令都不会实际执行
3. 不保证原子性
  - 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚
