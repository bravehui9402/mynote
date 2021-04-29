用到分布式锁说明遇到了多个进程共同访问同一个资源的问题。引入分布式锁势必要引入一个第三方的基础设施，比如MySQL，Redis，Zookeeper等。

锁的本质就是**互斥**，保证任何时候能有一个客户端持有同一个锁，如果考虑使用redis来实现一个分布式锁，最简单的方案就是在实例里面创建一个键值，释放锁的时候，将键值删除。

## idea1：分布式锁 SETNX

直接基于 redis 的 setNX (SET if Not eXists)命令，实现一个简单的锁。

```java
SETNX key value
命令执行时，如果 key 不存在，则设置 key 值为 value（同set）；如果 key 已经存在，则不执行赋值操作。并使用不同的返回值标识。 
```

还可以通过 SET 命令的 NX 选项使用：

```
SET key value [expiration EX seconds|PX milliseconds] [NX|XX]
NX - 仅在 key 不存在时执行赋值操作。可同时使用其它选项，如EX/PX设置超时时间
```

```java
setnx lock_a random_value
// do sth
delete lock_a
```

此实现方式的问题在于：一旦服务获取锁之后，因某种原因挂掉，则锁一直无法自动释放。从而导致死锁。

## idea2：分布式锁 SETNX + 过期时间

```
setnx lock_a random_value
setex lock_a 10 random_value // 10s超时
// do sth
delete lock_a
```

按需设置超时时间。此方案解决了方案1死锁的问题，但同时引入了新的死锁问题： 如果setnx之后，setex 之前服务挂掉，会陷入死锁。 根本原因为 setnx/setex 分为了两个步骤，非原子操作。

## idea3：分布式锁 SET NX  PX

```
SET lock_a random_value NX PX 10000 // 10s超时
// do sth
delete lock_a
```

此方案通过 set 的 NX/PX 选项，将加锁、设置超时两个步骤合并为一个原子操作，从而解决方案1、2的问题。(PX与EX选项的语义相同，差异仅在单位。) 此方案目前大多数 sdk、redis 部署方案都支持，因此是推荐使用的方式。 但此方案也有如下问题：

如果锁被错误的释放（如因为执行时间过长，key超时），或被错误的抢占，或因redis问题等导致锁丢失，无法很快的感知到。

## idea4：SET key randomvalue NX PX

```
SET lock_a random_value NX PX 10000
// do sth
eval "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end" 1 lock_a random_value
```

方案4在3的基础上，增加对 value 的检查，只解除自己加的锁。 类似于 CAS，不过是 compare-and-delete。 此方案 redis 原生命令不支持，为保证原子性，需要通过lua脚本实现:。

超时时间不能太短，否则在任务执行完成前就自动释放了锁，导致资源暴露在锁保护之外。 超时时间不能太长，否则会导致意外死锁后长时间的等待。除非人为接入处理。 因此建议是根据任务内容，合理衡量超时时间，将超时时间设置为任务内容的几倍即可。 如果实在无法确定而又要求比较严格，可以采用定期 setex/expire 更新超时时间实现。

## 完美解决方案：Redlock算法 与 Redisson 实现

假设有5个独立的Redis节点（**注意这里的节点可以是5个Redis单master实例，也可以是5个Redis Cluster集群，但并不是有5个主节点的cluster集群**）：

- 获取当前Unix时间，以毫秒为单位
- 依次尝试从5个实例，使用相同的key和具有唯一性的value(例如UUID)获取锁，当向Redis请求获取锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应用小于锁的失效时间，例如你的锁自动失效时间为10s，则超时时间应该在5~50毫秒之间，这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务端没有在规定时间内响应，客户端应该尽快尝试去另外一个Redis实例请求获取锁
- 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间，当且仅当从大多数(N/2+1，这里是3个节点)的Redis节点都取到锁，并且使用的时间小于锁失败时间时，锁才算获取成功。
- 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）
- 如果某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的Redis实例上进行解锁（即便某些Redis实例根本就没有加锁成功，防止某些节点获取到锁但是客户端没有得到响应而导致接下来的一段时间不能被重新获取锁）

### Redisson实现简单分布式锁

Redisson底层使用Netty可以实现非阻塞I/O，该客户端封装了锁的，继承了J.U.C的Lock接口，所以我们可以像使用ReentrantLock一样使用Redisson，具体使用过程如下。

1. 首先加入POM依赖

   ```
   <dependency>
       <groupId>org.redisson</groupId>
       <artifactId>redisson</artifactId>
       <version>3.10.6</version>
   </dependency>
   ```

2. 配置类

   ```java
   @Configuration
    public class RedissonConfig {
    
        @Bean
        public RedissonClient redissonClient() {
            Config config = new Config();
            config.useClusterServers()
                    .setScanInterval(2000)
                    .addNodeAddress("redis://*:6379", "redis://*:6379")
                    .addNodeAddress("redis://*:6379");
            RedissonClient redisson = Redisson.create(config);
            return redisson;
        }
    
    }
   ```

   

3. 使用Redisson，代码如下(与使用ReentrantLock类似）

```java
RLock lock = redissonClient.getLock("redlock");
lock.lock();
try {
    System.out.println("获取锁成功，实现业务逻辑");
    Thread.sleep(10000);
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    lock.unlock();
}
```

