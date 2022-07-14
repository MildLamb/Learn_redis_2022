# key操作
- keys * ：keys*查看当前库所有key
- exists key ：判断某个key是否存在
- type key ：查看key是什么类型
- del key ：删除指定的key数据
- expire key 10 ：给key设置过期时间
- ttl key ：查看key还有多少秒过期，-1表示永不过期，-2表示已经过期

```bash
127.0.0.1:6379> set name mildlamb
OK
127.0.0.1:6379> get name
"mildlamb"
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> exists asd
(integer) 0
127.0.0.1:6379> exists name
(integer) 1
127.0.0.1:6379> del name
(integer) 1
127.0.0.1:6379> keys *
(empty array)
127.0.0.1:6379> set name wildwolf
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> expire name 30
(integer) 1
127.0.0.1:6379> ttl name
(integer) 24
127.0.0.1:6379> ttl name
(integer) -2
```

# 数据库操作
- select num ：选择redis数据库，0-15
- dbsize ：查看当前数据库的key的数量
- flushdb ：清空当前库
- flushall ：清空所有库
