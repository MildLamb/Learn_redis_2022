# 分布式锁
## 问题描述
随着业务发展的需要，原单体单机部署的系统被演化成分布式集群系统后，由于分布式系统多线程，多线程并且分布在不同的机器上，  
这将使原单机部署情况下的并发控制锁策略失效，单纯的Java API并不能提供分布式锁的能力。为了解决这个问题就需要一种跨JVM  
的互斥机制来控制共享资源的访问，这就是分布式锁要解决的问题！  

分布式锁主流的解决方案：  
1. 基于数据库实现分布式锁
2. 基于缓存(Redis等)
3. 基于Zookeeper

## 使用redis实现分布式锁
redis命令  
# set key "ok" NX PX 10000
- EX seconds：设置键的过期时间为seconds秒。SET key value EX second 效果等同于 SETEX key seconds value
- PX milliseconds ：设置见的过期时间为 millisecond 毫秒。SET key value PX millisecond效果等同于 PSETEX key millisecond value
- NX ：只有键不存在时，才对键进行设置操作。SET key value NX 效果等同于 SETNX key value
- XX ：只有键已经存在时，才对键进行设置操作

![image](https://user-images.githubusercontent.com/92672384/180674862-d0ee53db-d1a1-4341-901c-afb63ae5bf9f.png)


## 分布式锁实现
```java
    /**
     * Redis分布式锁
     */
    @GetMapping("/testLock")
    public void testLock(){

        // 使用UUID防止误删锁
        String uuid = UUID.randomUUID().toString();

        // 1. 上锁，setnx
        Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock",uuid,10, TimeUnit.SECONDS);
        // 2. 获取锁成功，查询num的值
        if (lock){
            Object value = redisTemplate.opsForValue().get("num");
            // 2.1 判断num为空则返回return
            if (StringUtils.isEmpty(value)){
                return;
            }
            // 2.2 有值就转为int
            int num = Integer.parseInt(value+"");
            // 2.3 redis的num加1
            redisTemplate.opsForValue().set("num",++num);
            // 2.4 释放锁 del
            // 比较UUID值是否一样,防止误删锁
            String checkUUid = (String) redisTemplate.opsForValue().get("lock");
            if (uuid.equals(checkUUid)){
                redisTemplate.delete("lock");
            }
        } else {
            // 3. 如果获取锁失败，每个1秒再获取一次
            try {
                Thread.sleep(100);
                testLock();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
```

### 分布式锁可能出现的一些问题
- 可能出现误删锁的情况，使用uuid解决
- 删除操作缺乏原子性
