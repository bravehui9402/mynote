## 依赖引入

```xml
        <!-- 日志 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.25</version>
        </dependency>

        <!-- redis -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
            <version>1.8.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>

        <!--单元测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
```

## 配置文件

> redis-cluster.properties

```properties
redis.host1=****
redis.port1=7001
redis.host2=****
redis.port2=7002 
redis.host3=****
redis.port3=7003  
redis.host4=****
redis.port4=7004  
redis.host5=****
redis.port5=7005
redis.host6=****
redis.port6=7006
redis.expiration=3000  
#最大空闲数
redis.maxIdle=300 
#连接池的最大数据库连接数。设为0表示无限制,如果是jedis 2.4以后用redis.maxTotal
redis.maxActive=600  
#最大建立连接等待时间
redis.maxWait=1000  
#是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
redis.testOnBorrow=true
#客户端超时时间单位是毫秒 默认是2000
redis.timeout=10000  
#控制一个pool可分配多少个jedis实例,用来替换上面的redis.maxActive,如果是jedis 2.4以后用该属性
redis.maxTotal=1000  
#最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。
redis.maxWaitMillis=1000  
#连接的最小空闲时间 默认1800000毫秒(30分钟)
redis.minEvictableIdleTimeMillis=300000  
#每次释放连接的最大数目,默认3
redis.numTestsPerEvictionRun=1024  
#逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
redis.timeBetweenEvictionRunsMillis=30000  
```

> applocation.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <description>redis集群</description>

    <!-- 加载properties文件 -->
    <bean id="propertyPlaceholderConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath*:/redis-cluster.properties</value>
            </list>
        </property>
    </bean>

    <!-- 配置JedisPoolConfig实例 -->
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <!--最大空闲数-->
        <property name="maxIdle" value="${redis.maxIdle}" />
        <!--连接池的最大数据库连接数  -->
        <property name="maxTotal" value="${redis.maxActive}" />
        <!--最大建立连接等待时间-->
        <property name="maxWaitMillis" value="${redis.maxWait}" />
        <!--是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个-->
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
        <!--逐出连接的最小空闲时间 默认1800000毫秒(30分钟)-->
        <property name="minEvictableIdleTimeMillis" value="${redis.minEvictableIdleTimeMillis}" />
        <!--每次逐出检查时 逐出的最大数目 如果为负数就是 : 1/abs(n), 默认3-->
        <property name="numTestsPerEvictionRun" value="${redis.numTestsPerEvictionRun}" />
        <!--逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1-->
        <property name="timeBetweenEvictionRunsMillis" value="${redis.timeBetweenEvictionRunsMillis}" />
    </bean>

    <!-- Redis集群配置 -->
    <bean id="redisClusterConfig" class="org.springframework.data.redis.connection.RedisClusterConfiguration">
        <property name="maxRedirects" value="6"></property>
        <property name="clusterNodes">
            <set>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="${redis.host1}"></constructor-arg>
                    <constructor-arg name="port" value="${redis.port1}"></constructor-arg>
                </bean>

                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="${redis.host2}"></constructor-arg>
                    <constructor-arg name="port" value="${redis.port2}"></constructor-arg>
                </bean>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="${redis.host3}"></constructor-arg>
                    <constructor-arg name="port" value="${redis.port3}"></constructor-arg>
                </bean>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="${redis.host4}"></constructor-arg>
                    <constructor-arg name="port" value="${redis.port4}"></constructor-arg>
                </bean>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="${redis.host5}"></constructor-arg>
                    <constructor-arg name="port" value="${redis.port5}"></constructor-arg>
                </bean>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="${redis.host6}"></constructor-arg>
                    <constructor-arg name="port" value="${redis.port6}"></constructor-arg>
                </bean>
            </set>
        </property>
    </bean>


    <!-- 配置JedisConnectionFactory -->
    <bean id="jeidsConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <constructor-arg name="clusterConfig" ref="redisClusterConfig" />
        <property name="poolConfig" ref="poolConfig"/>
        <property name="timeout" value="${redis.timeout}" />
        <property name="password" value="123456"></property>
    </bean>


    <!-- 配置RedisTemplate -->
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="jeidsConnectionFactory" />
        <property name="keySerializer" >
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
        </property>
        <property name="valueSerializer" >
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
        </property>
        <property name="hashKeySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
        </property>
        <property name="hashValueSerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
        </property>
    </bean>
</beans>
```

## String操作

```java
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:application.xml");
        RedisTemplate redisTemplate = context.getBean("redisTemplate", RedisTemplate.class);

        //set值
        redisTemplate.opsForValue().set("person:1","tom");

        //set值同时设置过期时间
        redisTemplate.opsForValue().set("person:2","bob",10, TimeUnit.SECONDS);

        //set void set(K key, V value, long offset);
        //覆写(overwrite)给定 key 所储存的字符串值，从偏移量 offset 开始
        redisTemplate.opsForValue().set("person:1","baby",0);

        //getAndSet V getAndSet(K key, V value);
        //设置键的字符串值并返回其旧值
        Object bob = redisTemplate.opsForValue().getAndSet("person:1", "bob");

        //Integer append(K key, String value);
        //如果key已存在并且是一个字符串，则该命令将该值追加到字符串末尾。如果key不存在。则被创建 并设置为空字符串
        redisTemplate.opsForValue().append("newKey","hello");


        //size Long size(K key);
        //返回key所对应的value值得长度
        redisTemplate.opsForValue().size("newKey");
```

## List操作

```java
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:application.xml");
        RedisTemplate redisTemplate = context.getBean("redisTemplate", RedisTemplate.class);
        redisTemplate.delete("listdemo1");
        //Long size(K key);
        //返回存储在键中的列表的长度。如果键不存在，则将其解释为空列表，并返回0。当key存储的值不是列表时返回错误。
        Long listdemo1Size = redisTemplate.opsForList().size("listdemo1");


        //Long leftPush(K key, V value);
        //将所有指定的值插入存储在键的列表的头部。如果键不存在，则在执行推送操作之前将其创建为空列表。（左侧插值）
        redisTemplate.opsForList().leftPush("listdemo1","value0");

        //Long leftPushAll(K key, V... values);
        //批量把一个数组插入到列表中(批量左侧插值)
        redisTemplate.opsForList().leftPushAll("listdemo1", Arrays.asList("value1","value2","value3"));

        //Long rightPush(K key, V value);
        //将所有指定的值插入存储在键的列表的头部。如果键不存在，则在执行推送操作之前将其创建为空列表。（右侧插值）
        redisTemplate.opsForList().rightPush("listdemo1","value4");

        //Long rightPushAll(K key, V... values); (批量右侧插值)
        redisTemplate.opsForList().rightPushAll("listdemo1","value5","value6","value7");

        //void set(K key, long index, V value);
        //在列表中index的位置设置value值
        redisTemplate.opsForList().set("listdemo1",0,"value8");

        //Long remove(K key, long count, Object value);
        //从存储在键中的列表中删除等于value的元素的第一个计数事件。
        //计数参数以下列方式影响操作：
        //count > 0：删除等于value从头到尾移动的值的元素。
        //count <0：删除等于value从尾到头移动的值的元素。
        //count = 0：删除等于value的所有元素。
        redisTemplate.opsForList().remove("listdemo1",1,"value0"); ////将删除列表中存储的列表中第一次次出现的“value0”。

        //V index(K key, long index);
        //根据下表获取列表中的值，下标从0开始
        Object listdemo1Index = redisTemplate.opsForList().index("listdemo1", 0);

        //V leftPop(K key);
        //弹出最左边的元素，弹出之后该值在列表中将不复存在
        redisTemplate.opsForList().leftPop("listdemo1");

        //V rightPop(K key);
        //弹出最右边的元素，弹出之后该值在列表中将不复存在
        redisTemplate.opsForList().rightPop("listdemo1");



        System.out.println("list contains ： " );
        redisTemplate.opsForList().range("listdemo1",0,-1).stream().forEach(obj -> System.out.println(obj));
   
```

## Hash操作

```java
       ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:application.xml");
        RedisTemplate redisTemplate = context.getBean("redisTemplate", RedisTemplate.class);
        redisTemplate.delete("hashdemo");
        redisTemplate.opsForHash().put("hashdemo","name","xiaoming");
        redisTemplate.opsForHash().put("hashdemo","age","18");
        redisTemplate.opsForHash().put("hashdemo","gender","1");

        //Boolean hasKey(H key, Object hashKey);
        //确定哈希hashKey是否存在
        redisTemplate.opsForHash().hasKey("hashdemo","name");

        //HV get(H key, Object hashKey);
        //从键中的哈希获取给定hashKey的值
        redisTemplate.opsForHash().get("hashdemo","name");

        //Set<HK> keys(H key);
        //获取key所对应的散列表的key
        redisTemplate.opsForHash().keys("hashdemo");

        //Long size(H key);
        //获取key所对应的散列表的大小个数
        redisTemplate.opsForHash().size("hashdemo");

        // void putAll(H key, Map<? extends HK, ? extends HV> m);
        //使用m中提供的多个散列字段设置到key对应的散列表中
        Map<String,Object> testMap = new HashMap();
        testMap.put("name","666");
        testMap.put("age","27");
        testMap.put("class","1");
        redisTemplate.opsForHash().putAll("hashdemo",testMap);

        // void put(H key, HK hashKey, HV value);
        //设置散列hashKey的值
        redisTemplate.opsForHash().put("hashdemo","name","lisi");

        // List<HV> values(H key);
        //获取整个哈希存储的值
        redisTemplate.opsForHash().values("hashdemo");

        //Map<HK, HV> entries(H key);
        //获取整个哈希存储的entry
        redisTemplate.opsForHash().entries("hashdemo");


        redisTemplate.opsForHash().entries("hashdemo").entrySet().stream().forEach(entry -> System.out.println(entry));
```

## Set操作

```java
       ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:application.xml");
        RedisTemplate redisTemplate = context.getBean("redisTemplate", RedisTemplate.class);
        redisTemplate.delete(Arrays.asList("setdemo1","setdemo2"));


        //Long add(K key, V... values);
        //无序集合中添加元素，返回添加个数
        //也可以直接在add里面添加多个值 如：template.opsForSet().add("setTest","aaa","bbb")
        redisTemplate.opsForSet().add("setdemo1","value0","value1","value2","value4","value5");
        redisTemplate.opsForSet().add("setdemo2","value1","value2","value3");

        //Long remove(K key, Object... values);
        //移除集合中一个或多个成员
        redisTemplate.opsForSet().remove("setdemo1","value5");

        //V pop(K key);
        //移除并返回集合中的一个随机元素
        redisTemplate.opsForSet().pop("setdemo1");

        // Boolean move(K source, V value, K destination);
        //将 member 元素从 source 集合移动到 destination 集合
        redisTemplate.opsForSet().move("setdemo1","value1","setdemo2");

        //Long size(K key);
        //无序集合的大小长度
        redisTemplate.opsForSet().size("setdemo1");

        //Set<V> members(K key);
        //返回集合中的所有成员
        redisTemplate.opsForSet().members("setdemo1");



        System.out.println("setdemo1");
        redisTemplate.opsForSet().members("setdemo1").stream().forEach(entry -> System.out.print(entry+";"));
        System.out.println();
        System.out.println("setdemo2");
        redisTemplate.opsForSet().members("setdemo2").stream().forEach(entry -> System.out.print(entry+";"));
        System.out.println();

```

## Zset操作

```Java
       ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:application.xml");
        RedisTemplate redisTemplate = context.getBean("redisTemplate", RedisTemplate.class);

        //Boolean add(K key, V value, double score);
        //新增一个有序集合，存在的话为false，不存在的话为true
        redisTemplate.opsForZSet().add("zsetdemo","value1",100);
        redisTemplate.opsForZSet().add("zsetdemo","value2",200);
        redisTemplate.opsForZSet().add("zsetdemo","value3",300);
        redisTemplate.opsForZSet().add("zsetdemo","value3ForRemove",300);

        //Long remove(K key, Object... values);
        //从有序集合中移除一个或者多个元素
        redisTemplate.opsForZSet().remove("zsetdemo","value3ForRemove");

        // Long rank(K key, Object o);
        //返回有序集中指定成员的排名，其中有序集成员按分数值递增(从小到大)顺序排列
        redisTemplate.opsForZSet().rank("zsetdemo","value2");

        //Set<V> range(K key, long start, long end);
        //通过索引区间返回有序集合成指定区间内的成员，其中有序集成员按分数值递增(从小到大)顺序排列
        redisTemplate.opsForZSet().range("zsetdemo",0,-1);

        //Long count(K key, double min, double max);
        //通过分数返回有序集合指定区间内的成员个数
        Set zsetdemo = redisTemplate.opsForZSet().rangeByScore("zsetdemo", 0, 300);

        //Double score(K key, Object o);
        //获取指定成员的score值
        Double score = redisTemplate.opsForZSet().score("zsetdemo", "value2");

        //Long removeRange(K key, long start, long end);
        //移除指定索引位置的成员，其中有序集成员按分数值递增(从小到大)顺序排列
        redisTemplate.opsForZSet().remove("zsetdemo",0,1);
```

