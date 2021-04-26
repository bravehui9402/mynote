## String（字符串）

String 是Redis最基本数据类型。一个Redis中字符串最大不超过512M。
String类型是二进制安全的。可以包含任何数据。比如jpg图片或者序列化的对象。

### 常用命令

```
1、set key value ：设置值
  mset key1 value1 key2 value2 ... 同时设置1-多个
2、get key ：获取值
  mget key1 key2 key3 ...同时获取1-多个value
3、append key str： 追加
4、strlen key：获取值长度
5、setnx key value ：当key不存在设置
   msetnx key1 value1 key2 value2 ...同时设置1-多个key-value 当且仅当给定key不存在；原子性的，当有一个设置失败时，其他全部失败。
6、incr key ：储存数字值+1；原子性
7、decr key : 储存数字值-1；原子性
8、incrby key 步长 ：储存数字值+步长；原子性
9、decrby key 步长: 储存数字值-步长；原子性
10、getrange key start end : 获取值的范围，前包，后包。
11、setrange key start str : 替换值，前包
12、setex key time  value : 在设置值的时候设置过期时间
13、getset key value ：值的替换，返回旧值
```



### 数据结构

String的数据结构为简单动态字符串，可以修改的字符串。内部结构类似ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210426182507.png)

 当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。

11