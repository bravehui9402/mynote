## 依赖

```xml
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>
        <!--redis 客户端-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
        <!--自定义序列化-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <!--属性配置支持-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
```

## application.yml

```properties
spring:
  application:
    name: spring-boot-redis-cluster
  redis:
    cluster:
      nodes: *:7001,*:7002,*:7003,*:7004,*:7005,*:7006
      max-redirects: 6
redis:
  timeout: 10000 #客户端超时时间单位是毫秒 默认是2000
  maxIdle: 300 #最大空闲数
  maxTotal: 1000 #控制一个pool可分配多少个jedis实例,用来替换上面的redis.maxActive,如果是jedis 2.4以后用该属性
  maxWaitMillis: 1000 #最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。
  minEvictableIdleTimeMillis: 300000 #连接的最小空闲时间 默认1800000毫秒(30分钟)
  numTestsPerEvictionRun: 1024 #每次释放连接的最大数目,默认3
  timeBetweenEvictionRunsMillis: 30000 #逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
  testOnBorrow: true #是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
  testWhileIdle: true #在空闲时检查有效性, 默认false
  password: 123456 #密码
```

## RedisCluster配置类

```java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Data;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisNode;
import org.springframework.data.redis.connection.RedisPassword;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import redis.clients.jedis.JedisPoolConfig;
import java.util.HashSet;
import java.util.Set;

/**
 * @author liuhui
 * @title: RedisConfig
 * @projectName study
 * @description: redis 配置类
 * @date 2021/4/2811:29
 */
@Configuration
@Data
public class RedisClusterConfig {
    @Value("${spring.redis.cluster.nodes}")
    private String clusterNodes;
    @Value("${spring.redis.cluster.max-redirects}")
    private int maxRedirects;
    @Value("${redis.password}")
    private String password;
    @Value("${redis.timeout}")
    private int timeout;
    @Value("${redis.maxIdle}")
    private int maxIdle;
    @Value("${redis.maxTotal}")
    private int maxTotal;
    @Value("${redis.maxWaitMillis}")
    private int maxWaitMillis;
    @Value("${redis.minEvictableIdleTimeMillis}")
    private int minEvictableIdleTimeMillis;
    @Value("${redis.numTestsPerEvictionRun}")
    private int numTestsPerEvictionRun;
    @Value("${redis.timeBetweenEvictionRunsMillis}")
    private int timeBetweenEvictionRunsMillis;
    @Value("${redis.testOnBorrow}")
    private boolean testOnBorrow;
    @Value("${redis.testWhileIdle}")
    private boolean testWhileIdle;


    /**
     * Redis连接池的配置
     *
     * @return JedisPoolConfig
     */
    @Bean
    public JedisPoolConfig getJedisPoolConfig() {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        // 最大空闲数
        jedisPoolConfig.setMaxIdle(maxIdle);
        // 连接池的最大数据库连接数
        jedisPoolConfig.setMaxTotal(maxTotal);
        // 最大建立连接等待时间
        jedisPoolConfig.setMaxWaitMillis(maxWaitMillis);
        // 逐出连接的最小空闲时间 默认1800000毫秒(30分钟)
        jedisPoolConfig.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
        // 每次逐出检查时 逐出的最大数目 如果为负数就是 : 1/abs(n), 默认3
        jedisPoolConfig.setNumTestsPerEvictionRun(numTestsPerEvictionRun);
        // 逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
        jedisPoolConfig.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
        // 是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
        jedisPoolConfig.setTestOnBorrow(testOnBorrow);
        // 在空闲时检查有效性, 默认false
        jedisPoolConfig.setTestWhileIdle(testWhileIdle);
        return jedisPoolConfig;
    }


    /**
     * Redis集群的配置
     *
     * @return RedisClusterConfiguration
     */
    @Bean
    public RedisClusterConfiguration redisClusterConfiguration() {
        RedisClusterConfiguration redisClusterConfiguration = new RedisClusterConfiguration();
        // Set<RedisNode> clusterNodes
        System.out.println(clusterNodes);
        String[] serverArray = clusterNodes.split(",");
        Set<RedisNode> nodes = new HashSet<RedisNode>();
        for (String ipPort : serverArray) {
            String[] ipAndPort = ipPort.split(":");
            nodes.add(new RedisNode(ipAndPort[0].trim(), Integer.valueOf(ipAndPort[1])));
        }
        redisClusterConfiguration.setClusterNodes(nodes);
        redisClusterConfiguration.setMaxRedirects(maxRedirects);
        redisClusterConfiguration.setPassword(RedisPassword.of(password));
        return redisClusterConfiguration;
    }

    /**
     * redis连接工厂类
     *
     * @return JedisConnectionFactory
     */
    @Bean
    public JedisConnectionFactory jedisConnectionFactory() {
        // 集群模式
        JedisConnectionFactory factory = new JedisConnectionFactory(redisClusterConfiguration(), getJedisPoolConfig());
        return factory;
    }


    /**
     * 实例化 RedisTemplate 对象
     *
     * @return RedisTemplate<String, Object>
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // Template初始化
        initDomainRedisTemplate(redisTemplate);
        return redisTemplate;
    }

    /**
     * 设置数据存入 redis 的序列化方式 使用默认的序列化会导致key乱码
     */
    private void initDomainRedisTemplate(RedisTemplate<String, Object> redisTemplate) {
        // 开启redis数据库事务的支持
        redisTemplate.setEnableTransactionSupport(true);
        redisTemplate.setConnectionFactory(jedisConnectionFactory());

        // 如果不配置Serializer，那么存储的时候缺省使用String，如果用User类型存储，那么会提示错误User can't cast to
        // String！
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        redisTemplate.setHashKeySerializer(stringRedisSerializer);

        // jackson序列化对象设置
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(
                Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        // value序列化方式采用jackson
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

        redisTemplate.afterPropertiesSet();
    }
}
```

