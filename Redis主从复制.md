# 主从复制
## 是什么
- 主机数据更新后根据配置和策略，自动同步到从机的 master/slaver 机制，Master以写为主，Slave以读为主

## 能干嘛
- 读写分离，性能扩展
- 容灾快速恢复

![image](https://user-images.githubusercontent.com/92672384/180104240-15eff975-1649-4a8e-a80e-be22b02d04ec.png)

## 配置主从
1. 创建文件夹
2. 复制redis.conf配置文件
3. 配置一主两从，创建三个配置文件

- 新建redis6379/6380/6381.conf,修改端口号和日志文件名称和RDB文件名称
```text
pidfile /var/run/redis_6379.pid 
port 6379
dbfilename dump6379.rdb
masterauth yourredispwd  # 主机密码，如果不设置，即使说明成为从机，主机也不会显示
```

4. 启动三个redis服务
5. 使用info replication查看主从信息
6. 通过命令让某个成为主机的从机
```bash
# slaveof <ip> <host> : 成为指定主机的从机
slaveof 127.0.0.1 6379  
```

## 复制原理

![image](https://user-images.githubusercontent.com/92672384/180111436-474f03f0-d12e-4508-ae15-f0f8f34f6d45.png)

1）slave服务器连接到master服务器，便开始进行数据同步，发送psync命令（Redis2.8之前是sync命令）
2）master服务器收到psync命令之后，开始执行bgsave命令生成RDB快照文件并使用缓存区记录此后执行的所有写命令
3）master服务器bgsave执行完之后，就会向所有Slava服务器发送快照文件，并在发送期间继续在缓冲区内记录被执行的写命令
4）slave服务器收到RDB快照文件后，会将接收到的数据写入磁盘，然后清空所有旧数据，在从本地磁盘载入收到的快照到内存中，同时基于旧的数据版本对外提供服务。
5）master服务器发送完RDB快照文件之后，便开始向slave服务器发送缓冲区中的写命令
6）slave服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令
7）如果slave node开启了AOF，那么会立即执行BGREWRITEAOF，重写AOF


(以下均为命令行操作，实际操作应配置文件中进行)
### 一主两从
1. 切入点问题?slave1,slave2是从头开始复制还是从切入点开始复制?比如从k4进来，那么之前的k1，k2，k3是否也可以复制?
- 从服务器挂了，重新启动后会进行一次全量复制
2. 从机是否可以写?set可否?
- 从机不可以写，不可以set
3. 主机shutdown后情况如何?从机是上位还是待机?
- 从机待机

### 薪火相传
- 上一个SLAVE可以是下一个SLAVE的Master，slave同样可以接收其他slave的连接和同步请求，那么该slave作为了链条中下一个
master，可以有效减轻master的写压力

### 反客为主
```bash
slaveof no one   # 把从机变成主机
```
