## 什么是集群?
- Redis集群实现了对Redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在N个节点中，每个节点存储总数据的1/N
- Redis集群通过分区(partition)来提供一定程度的可用性(availability);即使集群中有一部分节点失效或者无法通讯，  
  集群也可以继续处理命令请求
  
## 模拟Redis集群搭建
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
redis-cli --cluster create --cluster-replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6389 127.0.0.1:6390 127.0.0.1:6391
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

- 成功的样子
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
