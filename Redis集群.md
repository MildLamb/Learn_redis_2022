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
