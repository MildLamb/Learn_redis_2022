## 什么是集群?
- Redis集群实现了对Redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在N个节点中，每个节点存储总数据的1/N
- Redis集群通过分区(partition)来提供一定程度的可用性(availability);即使集群中有一部分节点失效或者无法通讯，  
  集群也可以继续处理命令请求
  
## 模拟Redis集群搭建 （配置集群前，先不要设置密码，配置完成后再设置）
- 编写6个实例 redis6379.conf/... 等6个redis配置文件
  - 开启daemonize yes
  - pid 文件名字
  - 指定端口
  - Log 文件名称

- 集群相关配置
  - cluster-enable yes ：开启集群模式
  - cluster-config-file nodes-6379.conf : 设定节点集群配置文件名称
  - cluster-node-timeout 15000 ：设定节点失联时间，超过该时间(毫秒)，集群自动进行主从切换

- 编写好配置文件后，启动这6个服务
- 将六个节点合成一个集群
  - 需要在redis安装目录的src目录下执行，下列命令 
```bash
redis-cli --cluster create --cluster-replicas 1 43.142.97.59:6379 43.142.97.59:6380 43.142.97.59:6381 43.142.97.59:6389 43.142.97.59:6390 43.142.97.59:6391
```
## 集群密码设置
1. 如果是使用redis-trib.rb工具构建集群，集群构建完成前不要配置密码，集群构建完毕再通过config set + config rewrite命令逐个机器设置密码
2. 如果对集群设置密码，那么requirepass和masterauth都需要设置，否则发生主从切换时，就会遇到授权问题，可以模拟并观察日志
3. 各个节点的密码都必须一致，否则Redirected就会失败

方式一：修改所有Redis集群中的redis.conf文件  
```txt
masterauth 1234
requirepass 1234
```
方式二：进入各个实例通过config set设置
```bash
[root@iZj6c7eeosj2t5vjw8rf4xZ redis_cluster]# redis-cli -c -p 7000
127.0.0.1:7000> config set masterauth mildlambW2kindredwildwolfW2snowgnar
OK 
127.0.0.1:7000> config set requirepass mildlambW2kindredwildwolfW2snowgnar
OK 
127.0.0.1:7000> auth mildlambW2kindredwildwolfW2snowgnar
OK 
127.0.0.1:7000> config rewrite 
OK
```

## 成功的话就是这样
```bash
[root@VM-4-16-centos src]# redis-cli --cluster create --cluster-replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6389 127.0.0.1:6390 127.0.0.1:6391
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:6390 to 127.0.0.1:6379
Adding replica 127.0.0.1:6391 to 127.0.0.1:6380
Adding replica 127.0.0.1:6389 to 127.0.0.1:6381
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 0d86a7f9ac52d0a60729b9dcfbf31c53301005d6 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
M: e2b07205de9f13ce5cf040052389521d217a7d1b 127.0.0.1:6380
   slots:[5461-10922] (5462 slots) master
M: 799a62f6550f9460572f9f69ce2c7c65081d3598 127.0.0.1:6381
   slots:[10923-16383] (5461 slots) master
S: 227c40bbe61e836c63612485dba8f45e1fb6f391 127.0.0.1:6389
   replicates 0d86a7f9ac52d0a60729b9dcfbf31c53301005d6
S: 1ad085c71e0e8e2662ddf0cd864eb4d4c4a44365 127.0.0.1:6390
   replicates e2b07205de9f13ce5cf040052389521d217a7d1b
S: c4164dd02421d33f5c1f2a2270508914d3cf66fa 127.0.0.1:6391
   replicates 799a62f6550f9460572f9f69ce2c7c65081d3598
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: 0d86a7f9ac52d0a60729b9dcfbf31c53301005d6 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: c4164dd02421d33f5c1f2a2270508914d3cf66fa 127.0.0.1:6391
   slots: (0 slots) slave
   replicates 799a62f6550f9460572f9f69ce2c7c65081d3598
M: e2b07205de9f13ce5cf040052389521d217a7d1b 127.0.0.1:6380
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 1ad085c71e0e8e2662ddf0cd864eb4d4c4a44365 127.0.0.1:6390
   slots: (0 slots) slave
   replicates e2b07205de9f13ce5cf040052389521d217a7d1b
S: 227c40bbe61e836c63612485dba8f45e1fb6f391 127.0.0.1:6389
   slots: (0 slots) slave
   replicates 0d86a7f9ac52d0a60729b9dcfbf31c53301005d6
M: 799a62f6550f9460572f9f69ce2c7c65081d3598 127.0.0.1:6381
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
[root@VM-4-16-centos src]# 
```

- 使用集群方式连接
```bash
[root@VM-4-16-centos bin]# redis-cli -c -p 6380
127.0.0.1:6380> 
```

- 删除集群
```bash
#直接杀死所有进程（直接暴力）
kill -9 $(pidof redis-server)

#停止所有节点（这个关闭也行，不暴力）
redis-cli -h 172.17.97.2 -p 7001 shutdown
redis-cli -h 172.17.97.2 -p 7002 shutdown
redis-cli -h 172.17.97.2 -p 7003 shutdown
redis-cli -h 172.17.97.2 -p 7004 shutdown
redis-cli -h 172.17.97.2 -p 7005 shutdown
redis-cli -h 172.17.97.2 -p 7006 shutdown

#删除节点下面的所有相关文件（这个是批量删除的操作）
rm -f ./*/nodes-*.conf ./*/appendonly.aof ./*/dump.rdb
```

# 搭建Redis集群遇到的问题：Waiting for the cluster to join~~~  (使用云服务器搭建)
- 使用云服务器的话，要开启云服务器对应端口和linux防火墙对应端口
- 集群总线

```
每个Redis集群中的节点都需要打开两个TCP连接。一个连接用于正常的给Client提供服务，比如6379，
还有一个额外的端口（通过在这个端口号上加10000）作为数据端口，例如：redis的端口为6379，那么
另外一个需要开通的端口是：6379 + 10000， 即需要开启 16379。16379端口用于集群总线，这是一个
用二进制协议的点对点通信信道。这个集群总线（Cluster bus）用于节点的失败侦测、配置更新、故障转移
授权，等等。
```
- 解决方案
```
只需要将开启Redis端口对应的 集群总线端口即可。例如： 6379 + 10000 = 16379。
所以开放每个集群节点的客户端端口和集群总线端口才能成功创建集群！
```

## 什么是slots
- 一个Redis集群包含16384个插槽(hash slot)，数据库中的每个键都属于16384个插槽中的其中一个
- 集群使用公式CRC16(key)%16384来计算键key属于哪个插槽，其中CRC16(key)语句用于计算键key的CRC16校验和
- 集群中的每个节点负责处理一部分插槽。


## 集群操作
- 添加单个键时，会计算插槽值，然后需要根据插槽值到对应的集群中设置值
- 不在一个插槽下的键值，是不能使用mset，mget等多键操作的
```bash
43.142.97.59:6380> mset k1 v1 name kindred
(error) CROSSSLOT Keys in request don't hash to the same slot
```
- 可以通过{}来定义组的概念，从而使key中{}内相同内容的键值放到一个slot中
```bash
43.142.97.59:6380> mset name1{role} kindred name2{role} gnar name3{role} neeko
OK
```
- 查询集群中的值  CLUSTER GETKEYSINSLOT <slot> <count> 返回count个slot槽中的键
```bash
# 查询键对应的插槽值
43.142.97.59:6380> cluster keyslot name
(integer) 5798
# 查询插槽内 键的数量 只能查主机自己插槽范围内的
43.142.97.59:6380> cluster countkeysinslot 5798
(integer) 1
# 查询role组的插槽值
43.142.97.59:6380> cluster keyslot role
(integer) 10755
# 返回 5 个 指定插槽值上的键
43.142.97.59:6380> cluster getkeysinslot 10755 5
1) "name1{role}"
2) "name2{role}"
3) "name3{role}"
```
  
## 如果某一段插槽的主从节点都宕机了，redis服务能否继续?
- 如果某一段插槽主从都挂了，而cluster-require-full-coverage 设置为 yes ，那么整个集群都挂了，如果设置为 no ，则是对应的插槽段无法提供服务
- redis.conf 中的参数 cluster-require-full-coverage
