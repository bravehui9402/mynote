Redis事务提供了一种将多个命令打包，然后一次性、按顺序性的执行多个命令的机制。在事务执行期间，服务器不会中断事务而去执行其他客户端的命令请求，它会将事务中的所有命令执行完毕，然后才去处理其他客户端的命令请求。

**「Redis事务中没有像Mysql关系型数据库事务隔离级别的概念，不能保证原子性操作，也没有像Mysql那样执行事务失败会进行回滚操作」**。

> 事务的开始到结束会经历以下三个阶段

1、事务开始（MULTI）

2、命令入队

3、执行事务（EXEC）、撤销事务（DISCARD）

- MULTI **「事务开始的命令」**，执行该命令后，后面执行的对Redis数据类型的**「操作命令都会顺序的放进队列中」**，等待执行EXEC命令后队列中的命令才会被执行  

- DISCARD **「放弃执行队列中的命令」**，你可以理解为Mysql的回滚操作，**「并且将当前的状态从事务状态改为非事务状态」**。  

- EXEC 执行该命令后**「表示顺序执行队列中的命令」**，执行完后并将结果显示在客户端，**「将当前状态从事务状态改为非事务状态」**。若是执行该命令之前有key被执行WATCH命令并且又被其它客户端修改，那么就会放弃执行队列中的所有命令，在客户端显示报错信息，若是没有修改就会执行队列中的所有命令。 

-  WATCH key 表示指定监视某个key，**「该命令只能在MULTI命令之前执行」**，如果监视的key被其他客户端修改，**「EXEC将会放弃执行队列中的所有命令」**  

- UNWATCH **「取消监视之前通过WATCH 命令监视的key」**，通过执行EXEC 、DISCARD 两个命令之前监视的key也会被取消监视

## 事务的实现

### 事务开始

multi命令的执行标志着事务的开始。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210428220200.png)

multi命令执行完毕后，客户端状态的flags属性会打开REDIS_MULTI标识，表示该客户端从非事务状态切换到事务状态。

### 命令入队

当客户端处于非事务状态时，客户端发送的命令会立即被服务器执行。

当一个客户端处于事务状态时，服务器是否会执行客户端发出的命令，主要分为以下两种情况：

1、如果客户端发送的命令为**MULTI**、**EXEC**、**WATCH**、**DISCARD**四个命令中的其中一个，服务器会立即执行这个命令。

> multi:开启事务
>
> exec:执行事务
>
> discard：取消事务

2、如果客户端发送的命令为以上4个命令外的其他指令，服务器不会立即执行，而是将命令放入队列，然后向客户端返回入队（queue）回复

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210428220822.png)

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210428225627.png)

### 事务执行

当处于事务状态的客户端执行EXEC命令时，服务器会遍历客户端事务队列，按照先进先出顺序执行保存的所有命令，然后将执行命令的结果一次性返回客户端。

DISCARD命令取消一个事务的时候，就会将命令队列清空，并且将客户端的状态从事务状态修改为非事务的状态。

**「Redis的事务是不可重复的」**，当客户端处于事务状态的时候，再次向服务端发送MULTI命令时，直接就会向客户端返回错误。

## WATCH命令

`WATCH`命令是在MULTI命令之前执行的，表示监视任意数量的key，与它对应的命令就是`UNWATCH`命令，取消监视的key。

`WATCH`命令有点**「类似于乐观锁机制」**，在事务执行的时候，若是被监视的任意一个key被更改，则队列中的命令不会被执行，直接向客户端返回(nil)表示事务执行失败。

```shell
redis> WATCH num
OK
redis> MULTI
OK
redis> incrby num 10
QUEUED
redis> decrby num 1
QUEUED
redis> EXEC   // 执行成功
```

这个是`WATCH`命令的正常的操作流程，若是在其它的客户端，修改了被监视的任意key，就会放弃执行该事务，如下图所示：

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210428225840.png)

WATCH命令的底层实现中保存了`watched_keys` 字典，**「字典的键保存的是监视的key，值是一个链表，链表中的每个节点值保存的是监视该key的客户端」**。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210428225919.png)

若是某个客户端不再监视某个key，该客户端就会从链表中脱离。如client3，通过执行UNWATCH命令，不再监视key1：

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210428225939.png)





## 事务执行失败（错误处理）

Redis是没有回滚机制的，那么执行的过程，若是不小心敲错命令，Redis的命令发送到服务端没有被立即执行，所以是暂时发现不到该错误。

Redis中的错误处理主要分为两类：**「语法错误」**、**「运行错误」**。

**「（1）语法错误」**

比如执行命令的时候，命令的不存在或者错误的敲错命令、参数的个数不对等都会导致语法错误。

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set num 1
QUEUED
127.0.0.1:6379> set num
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379> ssset num 3
(error) ERR unknown command 'ssset'
127.0.0.1:6379> set num 2
QUEUED
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
```

语法错误是在Redis语法检测的时候就能发现的，所以当你执行错误命令的时候，也会即使的返回错误的提示。

最后，即使命令进入队列，只要存在语法错误，该队列中的命令都不会被执行，会直接向客户端返回事务执行失败的提示。

**「（2）运行错误」**

执行时使用不同类型的操作命令操作不同数据类型就会出现运行时错误，这种错误时Redis在不执行命令的情况下，是无法发现的。

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set num 3
QUEUED
127.0.0.1:6379> sadd num 4
QUEUED
127.0.0.1:6379> set num 6
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
3) OK
127.0.0.1:6379> get key
"6"
```

这样就会导致，正确的命令被执行，而错误的命令不会不执行，这也显示出Redis的事务并不能保证数据的一致性，因为中间出现了错误，有些语句还是被执行了。