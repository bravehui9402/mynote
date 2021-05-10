# 常用运维命令

### 查看指定 Topic 的详细信息，包括 Partition 个数，副本数，ISR 信息

```text
$ bin/kafka-topics.sh --zookeeper localhost:2181 --describe op_log
```

###  修改指定 Topic 的 Partition 个数。注意：该命令只能增加 Partition 的个数，不能减少 Partition 个数，当修改的 Partition 个数小于当前的个数，会报如下错误：`Error while executing topic command : The number of partitions for a topic can only be increased`

```text
$ bin/kafka-topics.sh --alter --zookeeper localhost:2181 --topic op_log1 --partition 4
```

###  删除名字为 "op_log" 的 Topic。注意如果直接执行该命令，而没有设置 delete.topic.enable 属性为 true 时，是不会立即执行删除操作的，而是仅仅将指定的 Topic 标记为删除状态，之后会启动后台删除线程来删除。

```text
$ bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic op_log
```

## **kafka-consumer-groups.sh** 脚本常用命令，主要用于操作消费组相关的。

###  查看消费端的所有消费组信息（ZK 维护消费信息）

```text
$ bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --list
```

###  查看消费端的所有消费组信息（Kafka 维护消费信息）

```text
$ bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --list
```

###  查看指定 group 的详细消费情况，包括当前消费的进度，消息积压量等等信息（ZK 维护消息信息）

```text
$ bin/kafka-consumer-groups.sh --zookeeper 22.187.29.169:2181 --group exchangeVerify3 --describe
```

###  查看指定 group 的详细消费情况，包括当前消费的进度，消息积压量等等信息（Kafka 维护消费信息）

```text
$ bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server 22.187.27.61:9092 --group exchangeVerify3 --describe
```

## **kafka-consumer-offset-checker.sh** 脚本常用命令，用于检查 OffSet 相关信息。（注意：该脚本在 0.9 以后可以使用 kafka-consumer-groups.sh 脚本代替，官方已经标注为 deprecated 了）

###  查看指定 group 的 OffSet 信息、所有者等信息

```text
$ bin/kafka-consumer-offset-checker.sh --zookeeper localhost:2181 --topic mytopic --group console-consumer-1291
```

###  查看指定 group 的 OffSet 信息，包括消费者所在的 IP 信息等等

```text
$ bin/kafka-consumer-offset-checker.sh --zookeeper localhost:2181 --topic mytopic --group console-consumer-1291 --broker-info
```

## **kafka-configs.sh** 脚本常用命令，该脚本主要用于增加/修改 Kafka 集群的配置信息。

###  对指定 entry 添加配置，其中 entry 可以是 topics、clients；如下，给指定的 topic 添加两个属性，分别为 max.message.byte、flush.message。

```text
$ bin/kafka-configs.sh --zookeeper localhost:2181 --alter --entity-type topics --entity-name mytopic --add-config 'max.message.bytes=50000000,flush.message=5'
```

###  对指定 entry 删除配置，其中 entry 可以是 topics、clients；如下。对指定的 topic 删除 flush.message 属性。

```text
$ bin/kafka-configs.sh --zookeeper localhost:2181 --alter --entity-type topics --entity-name mytopic --delete-config 'flush.message'
```

###  查看指定 topic 的配置项。

```text
$ bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name mytopic --describe
```

## **kafka-reassign-partitions.sh** 脚本相关常用命令，主要操作 Partition 的负载情况。

###  生成 partition 的移动计划，--broker-list 参数指定要挪动的 Broker 的范围，--topics-to-move-json-file 参数指定 Json 配置文件

```text
$ bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --broker-list "0" --topics-to-move-json-file ~/json/op_log_topic-to-move.json --generate
```

###  执行 Partition 的移动计划，--reassignment-json-file 参数指定 RePartition 后 Assigned Replica 的分布情况。

```text
$ bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file ~/json/op_log_reassignment.json --execute
```

###  检查当前 rePartition 的进度情况。

```text
$ bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file ~/json/op_log_reassignment.json --verify
```

## **常用配置及说明**

kafka 常见重要配置说明，分为四部分

- Broker Config：kafka 服务端的配置
- Producer Config：生产端的配置
- Consumer Config：消费端的配置
- Kafka Connect Config：kafka 连接相关的配置

### **Broker Config**

1. **zookeeper.connect**

连接 zookeeper 集群的地址，用于将 kafka 集群相关的元数据信息存储到指定的 zookeeper 集群中

**2. advertised.port**

注册到 zookeeper 中的地址端口信息，在 IaaS 环境中，默认注册到 zookeeper 中的是内网地址，通过该配置指定外网访问的地址及端口，advertised.host.name 和 advertised.port 作用和 advertised.port 差不多，在 0.10.x 之后，直接配置 advertised.port 即可，前两个参数被废弃掉了。

**3. auto.create.topics.enable**

自动创建 topic，默认为 true。其作用是当向一个还没有创建的 topic 发送消息时，此时会自动创建一个 topic，并同时设置 -num.partition 1 (partition 个数) 和 default.replication.factor (副本个数，默认为 1) 参数。

一般该参数需要手动关闭，因为自动创建会影响 topic 的管理，我们可以通过 kafka-topic.sh 脚本手动创建 topic，通常也是建议使用这种方式创建 topic。在 0.10.x 之后提供了 kafka-admin 包，可以使用其来创建 topic。

**4. auto.leader.rebalance.enable**

自动 rebalance，默认为 true。其作用是通过后台线程定期扫描检查，在一定条件下触发重新 leader 选举；在生产环境中，不建议开启，因为替换 leader 在性能上没有什么提升。

**5. background.threads**

后台线程数，默认为 10。用于后台操作的线程，可以不用改动。

**6. broker.id**

Broker 的唯一标识，用于区分不同的 Broker。kafka 的检查就是基于此 id 是否在 zookeeper 中/brokers/ids 目录下是否有相应的 id 目录来判定 Broker 是否健康。

**7. compression.type**

压缩类型。此配置可以接受的压缩类型有 gzip、snappy、lz4。另外可以不设置，即保持和生产端相同的压缩格式。

**8. delete.topic.enable**

启用删除 topic。如果关闭，则无法使用 admin 工具进行 topic 的删除操作。

**9. leader.imbalance.check.interval.seconds**

partition 检查重新 rebalance 的周期时间

**10. leader.imbalance.per.broker.percentage**

标识每个 Broker 失去平衡的比率，如果超过改比率，则执行重新选举 Broker 的 leader

**11. log.dir / log.dirs**

保存 kafka 日志数据的位置。如果 log.dirs 没有设置，则使用 log.dir 指定的目录进行日志数据存储。

**12. log.flush.interval.messages**

partition 分区的数据量达到指定大小时，对数据进行一次刷盘操作。比如设置了 1024k 大小，当 partition 积累的数据量达到这个数值时则将数据写入到磁盘上。

**13. log.flush.interval.ms**

数据写入磁盘时间间隔，即内存中的数据保留多久就持久化一次，如果没有设置，则使用 log.flush.scheduler.interval.ms 参数指定的值。

**14. log.retention.bytes**

表示 topic 的容量达到指定大小时，则对其数据进行清除操作，默认为-1，永远不删除。

**15. log.retention.hours**

标示 topic 的数据最长保留多久，单位是小时

**16. log.retention.minutes**

表示 topic 的数据最长保留多久，单位是分钟，如果没有设置该参数，则使用 log.retention.hours 参数

**17. log.retention.ms**

表示 topic 的数据最长保留多久，单位是毫秒，如果没有设置该参数，则使用 log.retention.minutes 参数

**18. log.roll.hours**

新的 segment 创建周期，单位小时。kafka 数据是以 segment 存储的，当周期时间到达时，就创建一个新的 segment 来存储数据。

**19. log.segment.bytes**

segment 的大小。当 segment 大小达到指定值时，就新创建一个 segment。

**20. message.max.bytes**

topic 能够接收的最大文件大小。需要注意的是 producer 和 consumer 端设置的大小需要一致。

**21. min.insync.replicas**

最小副本同步个数。当 producer 设置了 request.required.acks 为-1 时，则 topic 的副本数要同步至该参数指定的个数，如果达不到，则 producer 端会产生异常。

**22. num.io.threads**

指定 io 操作的线程数

**23. num.network.threads**

执行网络操作的线程数

**24. num.recovery.threads.per.data.dir**

每个数据目录用于恢复数据的线程数

**25. num.replica.fetchers**

从 leader 备份数据的线程数

**26. offset.metadata.max.bytes**

允许消费者端保存 offset 的最大个数

**27. offsets.commit.timeout.ms**

offset 提交的延迟时间

**28. offsets.topic.replication.factor**

topic 的 offset 的备份数量。该参数建议设置更高保证系统更高的可用性

**29. port**

端口号，Broker 对外提供访问的端口号。

**30. request.timeout.ms**

Broker 接收到请求后的最长等待时间，如果超过设定时间，则会给客户端发送错误信息

**31. zookeeper.connection.timeout.ms**

客户端和 zookeeper 建立连接的超时时间，如果没有设置该参数，则使用 zookeeper.session.timeout.ms 值

**32. connections.max.idle.ms**

空连接的超时时间。即空闲连接超过该时间时则自动销毁连接。

### **Producer Config**

1. **bootstrap.servers**

服务端列表。即接收生产消息的服务端列表

**2. key.serializer**

消息键的序列化方式。指定 key 的序列化类型

\3. **value.serializer**

消息内容的序列化方式。指定 value 的序列化类型

\4. **acks**

消息写入 Partition 的个数。通常可以设置为 0，1，all；当设置为 0 时，只需要将消息发送完成后就完成消息发送功能；当设置为 1 时，即 Leader Partition 接收到数据并完成落盘；当设置为 all 时，即主从 Partition 都接收到数据并落盘。

\5. **buffer.memory**

客户端缓存大小。即 Producer 生产的数据量缓存达到指定值时，将缓存数据一次发送的 Broker 上。

\6. **compression.type**

压缩类型。指定消息发送前的压缩类型，通常有 none, gzip, snappy, or, lz4 四种。不指定时消息默认不压缩。

\7. **retries**

消息发送失败时重试次数。当该值设置大于 0 时，消息因为网络异常等因素导致消息发送失败进行重新发送的次数。

### **Consumer Config**

1. **bootstrap.servers**

服务端列表。即消费端从指定的服务端列表中拉取消息进行消费。

\2. **key.deserializer**

消息键的反序列化方式。指定 key 的反序列化类型，与序列化时指定的类型相对应。

\3. **value.deserializer**

消息内容的反序列化方式。指定 value 的反序列化类型，与序列化时指定的类型相对应。

\4. **fetch.min.bytes**

抓取消息的最小内容。指定每次向服务端拉取的最小消息量。

\5. **group.id**

消费组中每个消费者的唯一表示。

\6. **heartbeat.interval.ms**

心跳检查周期。即在周期性的向 group coordinator 发送心跳，当服务端发生 rebalance 时，会将消息发送给各个消费者。该参数值需要小于 session.timeout.ms，通常为后者的 1/3。

\7. **max.partition.fetch.bytes**

Partition 每次返回的最大数据量大小。

**8. session.timeout.ms**

consumer 失效的时间。即 consumer 在指定的时间后，还没有响应则认为该 consumer 已经发生故障了。

**9. auto.offset.reset**

当 kafka 中没有初始偏移量或服务器上不存在偏移量时，指定从哪个位置开始消息消息。earliest：指定从头开始；latest：从最新的数据开始消费。

### **Kafka Connect Config**

1. **group.id**

消费者在消费组中的唯一标识

**2. internal.key.converter**

内部 key 的转换类型。

**3. internal.value.converter**

内部 value 的转换类型。

**4. key.converter**

服务端接收到 key 时指定的转换类型。

\5. **value.converter**

服务端接收到 value 时指定的转换类型。

**6. bootstrap.servers**

服务端列表。

**7. heartbeat.interval.ms**

心跳检测，与 consumer 中指定的配置含义相同。

**8. session.timeout.ms**

session 有效时间，与 consumer 中指定的配置含义相同。