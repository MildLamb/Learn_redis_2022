# 哨兵模式
## 是什么
- 反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票机制自动将从库转为主库

## 哨兵配置
1. 创建哨兵的配置文件 sentinel.conf 名字不能错
2. 配置哨兵，sentinel.conf 填写内容
```bash
# mymaster 为监控对象起的服务器名称 ，1为至少有多少哨兵同意迁移的数量
sentinel monitor mymaster 127.0.0.1 6379 1
```
3. 启动哨兵
```bash
[root@VM-4-16-centos bin]# redis-sentinel myredis/sentinel.conf
```
