[toc]
## 客户端开发

步骤：
* 配置生产者参数
* 构建待发送的消息
* 发送消息
* 关闭生产者实例

```java
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"localhost:9092");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class);

        KafkaProducer producer = new KafkaProducer(properties);
        ProducerRecord producerRecord = new ProducerRecord("topic1","key","value");

        try {
            producer.send(producerRecord);
        }catch (Exception e){
            e.printStackTrace();
        }
```
### 必要的配置参数
* bootstrap.servers kafka集群地址，不需要所有的集群地址，生产者会从给定的broker中查找到其他的broker信息。建议至少设置两个以上的broker地址，在初始连接中，如果其中一个宕机可以选择另一个进行连接。
* key.serializer and value.serializer broker端接受消息必须以字节数组的形式存在（网络传输）。这里表示的序列化方式，除了示例的字符串型的序列化器，还有其他如integer、object类型的序列化器，但是字符串型的序列化器能够满足大部分的需求需要。
* client.id 设定producer对应的客户端id，一般不用配置。

### 消息的构建
ProducerRecord是要发送的消息的构建类，除了上面指出的topic和key、value，还有其他属性供我们去选择。
```java
    private final String topic;
    private final Integer partition;
    private final Headers headers;
    private final K key;
    private final V value;
    private final Long timestamp;
```
* topic 主题
* partition 分区号
* key 指定消息的键，不仅是消息的附加信息，还可以用来计算分区号进而可以让信息发送到指定的分区，也就是说，可以通过key值得设置指定消息向同一个分区发送。
* value 消息体，也就是要发送的消息。
* timestamp 消息时间戳。

也就是说除了示例的构建方法外，还有其他的重载构建方法，只不过需要提供的参数不同罢了。
```java
     public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value, Iterable<Header> headers) {
        if (topic == null) {
            throw new IllegalArgumentException("Topic cannot be null.");
        } else if (timestamp != null && timestamp < 0L) {
            throw new IllegalArgumentException(String.format("Invalid timestamp: %d. Timestamp should always be non-negative or null.", timestamp));
        } else if (partition != null && partition < 0) {
            throw new IllegalArgumentException(String.format("Invalid partition: %d. Partition number should always be non-negative or null.", partition));
        } else {
            this.topic = topic;
            this.partition = partition;
            this.key = key;
            this.value = value;
            this.timestamp = timestamp;
            this.headers = new RecordHeaders(headers);
        }
    }
    public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value) {
        this(topic, partition, timestamp, key, value, (Iterable)null);
    }

    public ProducerRecord(String topic, Integer partition, K key, V value, Iterable<Header> headers) {
        this(topic, partition, (Long)null, key, value, headers);
    }

    public ProducerRecord(String topic, Integer partition, K key, V value) {
        this(topic, partition, (Long)null, key, value, (Iterable)null);
    }

    public ProducerRecord(String topic, K key, V value) {
        this(topic, (Integer)null, (Long)null, key, value, (Iterable)null);
    }

    public ProducerRecord(String topic, V value) {
        this(topic, (Integer)null, (Long)null, (Object)null, value, (Iterable)null);
    }
```
### 消息的发送
消息的发送方式有三种模式：
* 发送即忘
* 同步发送
* 异步发送

示例的发送方式就是发送即忘的发送方式，在发送消息后并不关心消息是否正确到达。这种方法方式的性能最高，可靠性最差。

send（）方法的返回值并不是void类型，而是Future<RecordMetadata>类型，，send（）方法有两个重载方法：
```java
    public Future<RecordMetadata> send(ProducerRecord<K, V> record) {
        return this.send(record, (Callback)null);
    }

    public Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback) {
        ProducerRecord<K, V> interceptedRecord = this.interceptors == null ? record : this.interceptors.onSend(record);
        return this.doSend(interceptedRecord, callback);
    }
```
#### 同步发送
而要实现同步的发送方式，可以利用返回的Fature对象实现。
```java
        try {
            Future<RecordMetadata> future = (Future<RecordMetadata>) producer.send(producerRecord).get();
            RecordMetadata recordMetadata = future.get();
            recordMetadata.topic();
            recordMetadata.partition();
            recordMetadata.offset();
            recordMetadata.timestamp();
            recordMetadata.serializedKeySize();
            recordMetadata.serializedValueSize();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
```
send()方法本身就是异步的，send()方法返回的Future对象可以使调用方稍后获得发送的结果。也就是通过调用get方法阻塞等待kafka的响应，知道消息发送成功，或者发生异常，如果发生异常，就可以捕获异常进行逻辑处理。

RecordMetadata对象包好了消息的 元数据信息，比如当前消息的主题、分区、偏移量等。

Fature表示一个任务的生命周期，并提供了相应的方法来判断任务是否已经完成或取消，以及获取任务的结果或取消任务。Fature的get（long timeout，TimeUnit unit）方法实现可超时的阻塞。
    
KafkaProducer中一般会发生两种类型的异常，可重试异常和不可重试异常。

常见的可重试异常有：NetworkException（网络异常，可能由于网络的瞬时故障导致的异常，可通过重试解决）、LeaderNotAvailableException（Leader不存在异常，通常发生在leader副本下线而没有选出新的leader之前）、UnknownTopicOrPartitionException、NotEnoughReplicasException、NotCoordinatorException等。

不可重试异常：RecordTooLargeException异常，发送消息太大，此时生产者不会进行任何重试，直接抛出异常。

对于可重试的异常，如果配置了retries参数，那么只要在规定的重试次数内自行恢复，就不会抛出异常。

同步发送可靠性高，消息发送要么成功，要么发生异常。如果发生异常，可以捕获相应异常进行相应处理。但是性能会差很多，需要阻塞等待一条消息发送完之后，才能发送下一条。

#### 异步发送
异步发送一般是在sned（）方法里指定一个CallBack的回调函数，Kafka在返回响应时调用该函数来实现异步的发送确认。
```java
         producer.send(producerRecord, new Callback() {
             public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                 if(e == null){
                     System.out.println("send success");
                     recordMetadata.topic().....
                 }else{
                     System.out.println("send fail"+e.getMessage());
                 }
             }
         });
```
通过添加回调函数，重写方法，对异常进行判空处理，就可以对发生异常和发送成功进行不同的处理，这里只做了简单的输出处理，在日常开发中，可以将异常信息记录，亦可以进行消息重发。
#### close
close（）方法会等待之前所有的发送请求完成后再关闭KakfaProducer，与此同时，Kafka还提供了一个带超时时间的close（）方法：
```java
 public void close(long timeout,TimeUnit timeUnit)
```
如果调用了带超时时间的timeout的close方法，在规定时间内没有完成发送，就会强行退出，在实际应用中，一般使用无参的close方法。