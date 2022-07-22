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
