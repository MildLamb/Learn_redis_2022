# Springboot整合Redis
- 导入依赖
```xml
<!-- redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- spring2.x 集成redis 所需common-pool2 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.11.1</version>
</dependency>

<!-- ObjectMapper -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

- 配置Redis参数
```yml
spring:
  redis:
    # 服务器地址
    host: 43.142.97.59
    # 服务器连接端口
    port: 6379
    # 选择的数据库索引
    database: 0
    # 链接超时时间(毫秒)
    connect-timeout: 1800000
    lettuce:
      pool:
        # 连接池最大连接数(使用负数表示没有限制)
        max-active: 8
        # 最大阻塞等待时间(负数表示没有限制)
        max-wait: -1
        # 连接池中的最大空闲连接
        max-idle: 6
        # 连接池中的最小空闲连接
        min-idle: 0
    password: xxx
```
- Redis配置类
```java
@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {
    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String,Object> template = new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        jsonRedisSerializer.setObjectMapper(om);
        template.setConnectionFactory(redisConnectionFactory);

        // key 的序列化方式
        template.setKeySerializer(redisSerializer);
        // value 的序列化方式
        template.setValueSerializer(jsonRedisSerializer);
        // value hashmap 序列化
        template.setHashValueSerializer(jsonRedisSerializer);
        return template;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory){
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        // 解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL,JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        jsonRedisSerializer.setObjectMapper(om);
        // 配置序列化(解决乱码问题),过期时间600秒
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(600))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jsonRedisSerializer))
                .disableCachingNullValues();
        RedisCacheManager cacheManager = RedisCacheManager.builder(redisConnectionFactory)
                .cacheDefaults(redisCacheConfiguration)
                .build();
        return cacheManager;
    }
}
```

- 测试
```java
@RestController
@RequestMapping("/redis")
public class RedisTestController {

    @Autowired
    private RedisTemplate redisTemplate;

    @GetMapping("/setget")
    public String testRedis(){
        // 设置字符串的值
        redisTemplate.opsForValue().set("name","mildlamb");
        // 从redis中获取值
        String name = (String) redisTemplate.opsForValue().get("name");
        return name;
    }
}
```
