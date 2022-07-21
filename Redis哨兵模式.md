# 哨兵模式
## 是什么
- 反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票机制自动将从库转为主库

## 哨兵配置
1. 创建哨兵的配置文件 sentinel.conf 名字不能错
2. 配置哨兵，sentinel.conf 填写内容
```bash
# mymaster 为监控对象起的服务器名称 ，1为至少有多少哨兵同意迁移的数量
sentinel monitor mymaster 127.0.0.1 6379 1
# 如果redis有密码
sentinel auth-pass mymaster yourredispwd
```
3. 启动哨兵
```bash
[root@VM-4-16-centos bin]# redis-sentinel myredis/sentinel.conf
```

## 复制延时
- 由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器
  有一定的延迟，当系统很繁忙的时候，延迟会更加严重，Slave数量的增加也会使这个问题更加严重

## 选举规则

![image](https://user-images.githubusercontent.com/92672384/180120182-a174630b-20c7-43f4-8bc3-a397e8fddc28.png)

- 优先级在 redis.conf 中默认：slave-priority 100 ，值越小优先级越高
- 偏移量是指获得原主机数据最全的
- 每个redis实例启动后都会随机生成一个40位的runid


## Java代码中使用哨兵
```java
public class JedisSentinelPoolUtil {
    private static JedisSentinelPool jedisSentinelPool = null;

    public static Jedis getJedisFormSentinel(){
        if (jedisSentinelPool==null){
            Set<String> sentinelSet = new HashSet<>();
            // 提供远程哨兵服务的地址和端口
            sentinelSet.add("xxx.xxx.xxx.xxx:26379");
            JedisPoolConfig poolConfig = new JedisPoolConfig();
            poolConfig.setMaxTotal(200);  // 最大可用连接数
            poolConfig.setMaxIdle(32);   // 最大闲置连接数
            poolConfig.setMaxWaitMillis(100*1000);  // 最大等待时间
            poolConfig.setBlockWhenExhausted(true);   // 连接耗尽时是否等待
            poolConfig.setTestOnBorrow(true);    // 获取连接时进行一下测试 ping
            
            // myredis 是哨兵监控的名称
            jedisSentinelPool = new JedisSentinelPool("myredis",sentinelSet,poolConfig);
            return jedisSentinelPool.getResource();
        } else {
            return null;
        }
    }
}
```
