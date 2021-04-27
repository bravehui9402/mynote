## String（字符串）

String 是Redis最基本数据类型。一个Redis中字符串最大不超过512M。
String类型是二进制安全的。可以包含任何数据。比如jpg图片或者序列化的对象。

### 常用命令

> 1、set key value ：设置值
>
>   	 mset key1 value1 key2 value2 ... 同时设置1-多个
>
> 2、get key ：获取值
>
>   	 mget key1 key2 key3 ...同时获取1-多个value
>
> 3、append key str： 追加
>
> 4、strlen key：获取值长度
>
> 5、setnx key value ：当key不存在设置
>
>    	msetnx key1 value1 key2 value2 ...同时设置1-多个key-value 当且仅当给定key不存在；原子性的，当有一个设置失败时，其他全部失败。
>
> 6、incr key ：储存数字值+1；原子性
>
> 7、decr key : 储存数字值-1；原子性
>
> 8、incrby key 步长 ：储存数字值+步长；原子性
>
> 9、decrby key 步长: 储存数字值-步长；原子性
>
> 10、getrange key start end : 获取值的范围，前包，后包。
>
> 11、setrange key start str : 替换值，前包
>
> 12、setex key time  value : 在设置值的时候设置过期时间
>
> 13、getset key value ：值的替换，返回旧值



### 数据结构

String的数据结构为简单动态字符串，可以修改的字符串。内部结构类似ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210426182507.png)

 当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。

## List（列表）

单key多value，字符串列表。可以再头部或尾部进行操作。底层双向链表

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210426185414.png)

### 常用命令

> lpush/rpush key value1 value2 value3... 从左边或右边插入一个或多个值
>
> lpop/rpop key count 从左/右 弹出count(可不加 默认1)值。值在健在，值光键亡。
>
> rpoplpush key1 key2 从key1列表右侧弹出值插到key2列表左边。
>
> lrange key start end 按照索引下标获得元素 0 -1 取出所有值
>
> lindex key index 按照索引下标获得元素（从左到右）
>
> llen key 获得列表长度
>
> linsert key before/after value newvalue 在value后插入newvalue值
>
> lrem key count value 从左边删除count个value
>
> lset key index value 将列表下标为index的值替换成value

### 数据结构

list数据结构 quickList ;在列表元素较少的情况下会使用一块连续的内存存储，这个结构叫zipList（压缩链表）；它将所有元素紧挨着存储，分配的是一块连续的内存。当数据量多的时候会转变成quickList,由多个zipList组成

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210426185535.png)

Redis将链表和ziplist结合起来 组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

## Set（集合）

Set是无序集合。底层是一个value为null的hash表。添加、删除、查找的复杂度为O(1)

### 常用命令

> sadd key value1 value2... 将一个或多个member元素加入到集合key中，已存在的member元素将被忽略。
>
> smembers key 取出该集合所有值
>
> sismember key value 判断key集合是否含有该value值。有1否0；
>
> scard key 返回该集合元素个数
>
> srem key value1 value2... 删除集合中某个元素
>
> spop key 随机从该集合中吐出一个值。
>
> srandmember key n 随机从集合中取出n个值。不会从集合中删除。
>
> smove source target value  将value从source移动到target集合
>
> sinter key1 key2 返回两个集合的交集
>
> sunion key1 key2 返回两个集合的并集
>
> sdiff key1 key2 返回两个集合的差集 key1-key2

### 数据结构

dict字典，通过hash表实现

## Hash（哈希）

Hash 是一个键值对集合。是一个string类型的field和value的映射表，hash适合存储对象。类似Java中的Map。例如：用户ID为key 。存储value包括 姓名、年龄、生日等信息。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210426190741.png)

### 常用命令

> hset key field value ..赋值
>
> hget key fieled 从key集合中取出field的值value
>
> hmset key field1 value1 field2 value2... 批量设置hash的值
>
> hexists key field 查看哈希表key中，给定域filed是否存在
>
> hkeys key 取出该hash集合的所有field 
>
> hvals key 列出该hash集合所有value
>
> hincrby key field increment 为哈希表key中的域field的值加上增量 1 -1 
>
> hsetnx key field value 将哈希表key中的域field的值设置为value，当且仅当域field不存在。

### 数据结构

Hash类型对应数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时。使用ziplist，否则使用hashtable。

## Zset（有序集合）

Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。不同之处是有序集合的每个成员都关联了一个***\*评分（score）\****,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复的 。

因为元素是有序的, 所以可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

### 常用命令

> zadd key socre1 value1 score2 value2   将一个或者多个member元素及其score值加入到有序集中
>
> zrange key startScore stopScore 【withscores】  获取选组在分数之间的元素
>
> zrangebyscore key minmax  [withscores] [limit offset count]  返回的有序集key中，所有score值介于min和max之间（包括等于min或max）的成员，有序集成员按score值递增（从小到大）次序排列
>
> zrevrangebyscore key maxmin [withscores] [limit offext count] 同上从大到小排列
>
> zincrby key increment value  为元素的score加上增量increment
>
> zrem key value 删除该集合指定元素
>
> zcount key min max 统计该集合分数区间的元素个数
>
> zrank key value 返回该值在集合中的排名，从0开始。

### 数据结构

类似于Java的Map<String,Double> 可以给每个元素value赋予一个权重score。另一方面又类似于TreeSet，内部元素按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围获取元素的列表。

zset底层使用了两个数据结构：

1、hash

2、跳跃表