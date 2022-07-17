# Jedis操作Redis
- 导入依赖
```xml
<dependency>
<groupId>redis.clients</groupId>
<artifactId>jedis</artifactId>
<version>3.7.0</version>
<exclusions>
    <exclusion>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
    </exclusion>
</exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-simple</artifactId>
  <version>1.7.30</version>
  <scope>compile</scope>
</dependency>
```
- 基本操作
```java
public class JedisDemo1 {
    public static void main(String[] args) {
        // 创建Jedis对象
        Jedis jedis = new Jedis("43.142.97.59",6379);
        // 测试
        String value = jedis.ping();
        System.out.println(value);
    }


    // 操作key string
    @Test
    public void test1(){
        // 创建Jedis对象
        Jedis jedis = new Jedis("43.142.97.59",6379);

        // 添加/获取键值对
        jedis.set("name","mildlamb");
        System.out.println(jedis.get("name"));
        System.out.println("===============================");

        // 添加多个键值对
        jedis.mset("role1","kindred","role2","gnar","role3","neeko");
        List<String> list = jedis.mget("role1", "role2", "role3");
        for (String s : list) {
            System.out.println(s);
        }
        System.out.println("===============================");

        // 获取所有的key
        Set<String> keys = jedis.keys("*");
        for (String key : keys) {
            System.out.println(key);
        }
    }

    // 操作key List
    @Test
    public void test2(){
        // 创建Jedis对象
        Jedis jedis = new Jedis("43.142.97.59",6379);

        jedis.lpush("roles","千珏","纳尔","妮蔻");
        List<String> roles = jedis.lrange("roles", 0, -1);
        for (String role : roles) {
            System.out.println(role);
        }
    }

    // 操作 set
    @Test
    public void test3(){
        // 创建Jedis对象
        Jedis jedis = new Jedis("43.142.97.59",6379);

        jedis.sadd("names","我无语啊干什么","爱上冰雪的小萌兽","Engulf迷失","MildLamb","千青灵花王玉瓷间","我无语啊干什么");
        Set<String> names = jedis.smembers("names");
        for (String name : names) {
            System.out.println(name);
        }
    }

    // 操作hash
    @Test
    public void test4(){
        // 创建Jedis对象
        Jedis jedis = new Jedis("43.142.97.59",6379);

        Map map = new HashMap<String, String>();
        map.put("age","1500");
        jedis.hset("user:1", map);
        System.out.println(jedis.hget("user:1","age"));
    }


    // 操作 zset
    @Test
    public void test5(){
        // 创建Jedis对象
        Jedis jedis = new Jedis("43.142.97.59",6379);

        jedis.zadd("level",214,"kindred");
        jedis.zadd("level",251,"gnar");
        jedis.zadd("level",127,"neeko");

        Set<String> level = jedis.zrevrange("level", 0, -1);
        for (String s : level) {
            System.out.println(s);
        }
    }
}

```
