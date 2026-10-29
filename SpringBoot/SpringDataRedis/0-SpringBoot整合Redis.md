## 1  添加项目依赖

```xml
<!-- Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- 连接池依赖 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

- 版本号由父项目进行管理

## 2  配置 Redis 连接信息

```yaml
spring:
  data:
    redis:
      # 基础连接信息
      host: localhost               # Redis 服务器地址
      port: 6379                    # 端口
      password:                     # 密码（若无留空）
      database: 0                   # 使用的数据库索引（默认0）
      timeout: 2000ms               # 连接超时时间（毫秒）
      client-type: lettuce          # 客户端类型（默认 lettuce，也可选 jedis）
      
      # Lettuce 连接池配置（需引入 commons-pool2）
      lettuce:
        pool:
          max-active: 8             # 最大活跃连接数
          max-idle: 8               # 最大空闲连接
          min-idle: 0               # 最小空闲连接
          max-wait: -1ms            # 获取连接的最大等待时间（-1表示无限等待）
        shutdown-timeout: 100ms     # 关闭超时
      
      # Sentinel 模式（集群高可用）配置示例（注释）
      # sentinel:
      #   master: mymaster
      #   nodes: 192.168.1.1:26379,192.168.1.2:26379
      
      # Cluster 模式配置示例（注释）
      # cluster:
      #   nodes: 192.168.1.1:6379,192.168.1.2:6379
      #   max-redirects: 3
```

## 3  配置 Redis 序列化方式

创建一个配置类，来定义一个自定义的 `RedisTemplate` Bean。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;

@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 1. 设置连接工厂
        template.setConnectionFactory(factory);
        
        // 2. 设置Key的序列化器为 StringRedisSerializer
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        
        // 3. 设置Value的序列化器为 Jackson2JsonRedisSerializer
        //    这样可以直接存储 Java 对象为 JSON 格式
        template.setValueSerializer(RedisSerializer.json());
        template.setHashValueSerializer(RedisSerializer.json());
        
        // 4. 初始化设置
        template.afterPropertiesSet();
        return template;
    }
}
```
