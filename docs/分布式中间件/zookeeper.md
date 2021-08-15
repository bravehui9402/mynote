zookeeper是一个开源的`分布式协调服务`框架。主要用来解决分布式集群中应用系统的`一致性问题`。

zookeeper本质上是一个分布式的小文件存储系统。提供基于`类似文件系统的目录方式`的数据存储，并且可以对树中的节点进行有效管理。从而维护和监控节点的状态以及数据变化。通过监控节点的状态和数据的变化，从而达到基于数据的集群管理。分布式应用程序可以基于 Zookeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

### zookeeper特性

1. **全局数据一致性**：每个节点保存相同的数据副本，客户端无论连接那个节点，展示的数据都是一致的。
2. **数据更新原子性**：数据更新要么成功，要么失败，不存在中间状态。
3. **实时性**：zookeeper保证客户端在一个时间间隔范围内获得服务器的更新信息。
4. **有序性：**有序性是 zookeeper 中非常重要的一个特性，所有的更新都是全局有序的，每个更新都有一个唯一的时间戳，这个时间戳称为 zxid（Zookeeper Transaction Id）。而读请求只会相对于更新有序，也就是读请求的返回结果中会带有这个zookeeper 最新的 zxid。

### zookeeper集群搭建

#### 搭建过程

环境前提：JDK

1. 上传zookeeper安装文件，并进行解压：

```shell
tar -zxvf zookeeper-3.4.10.tar.gz
```

2. 进入安装目录，在`conf`文件夹下，有zoo_sample.cfg文件，该文件为zookeeper配置文件示例文件，对此文件进行复制，并修改名称为：`zoo.cfg`

3. 修改zoo.cfg配置文件

   ```shell
   1.tickTime：Client-Server通信心跳时间
   Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime以毫秒为单位。
   tickTime=2000
   
   2.initLimit：Leader-Follower初始通信时限
   集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）。
   initLimit=5
   
   3.syncLimit：Leader-Follower同步通信时限
   集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。
   syncLimit=2
   
   4.dataDir：数据文件目录
   Zookeeper保存数据的目录，默认情况下，Zookeeper将写数据的日志文件也保存在这个目录里。
   dataDir=/home/michael/opt/zookeeper/data
   
   5.clientPort：客户端连接端口
   客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
   clientPort=2181
   
   6.服务器名称与地址：集群信息（服务器编号，服务器地址，LF通信端口，选举端口）
   这个配置项的书写格式比较特殊，规则如下：
   server.N=YYY:A:B
   server.N=ip1:2888:3888
   server.N=ip2:2888:3888
   server.N=ip3:2888:3888
   N：代表服务器编号（也就是myid里面的值）
   YYY：服务器地址
   A：表示 Flower 跟 Leader的通信端口，简称服务端内部通信的端口（默认2888）
   B：表示 是选举端口（默认是3888）
   
   7.maxClientCnxns：对于一个客户端的连接数限制，默认是60，这在大部分时候是足够了。但是在我们实际使用中发现，在测试环境经常超过这个数，经过调查发现有的团队将几十个应用全部部署到一台机器上，以方便测试，于是这个数字就超过了。
   
   8.autopurge.snapRetainCount、autopurge.purgeInterval -- 客户端在与zookeeper交互过程中会产生非常多的日志，而且zookeeper也会将内存中的数据作为snapshot保存下来，这些数据是不会被自动删除的，这样磁盘中这样的数据就会越来越多。不过可以通过这两个参数来设置，让zookeeper自动删除数据。autopurge.purgeInterval就是设置多少小时清理一次。而autopurge.snapRetainCount是设置保留多少个snapshot，之前的则删除。
   
   
   ```

   **修改参数**

   1. dataDir，修改为指定的地址
   2. server.N=YYY:A:B 根据集群进行配置

4. 集群操作：

   ```shell
   1.启动操作
   安装目录/bin/zkServer.sh start
   2.停止操作
   安装目录/bin/zkServer.sh stop
   3.状态查看
   安装目录/bin/zkServer.sh status
   4.客户端连接
   安装目录/bin/zkCli.sh -server host:port
   ```

5. shell基本操作

   1. 节点创建

   ```shell
   create [-s] [-e] path data acl
   -s 为顺序节点
   -e 为临时节点
   acl 权限控制
   默认为持久无顺序节点
   ```

   2. 读取节点

   ```shell
   ls path 查看节点
   get path 查看节点数据
   
   listquota path 列出指定节点quota
   ```

   3. 节点数据更新

   ```shell
   set path data [version]
   version 指定的是当前的数据版本，若更新时当前节点版本与version不一致将会更新失败（数据一致性问题）
   
   setquota -n|-b val path
   n:表示子节点最大个数
   b:表示数据值最大长度
   val:子节点最大个数或数据值最大长度
   ```

   4. 删除节点

   ```shell
   delete path [version]
   Rmr path 递归删除
   
   delquota [-n|-b] path 删除quota
   ```

### Zookeeper 特点

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815153939.png)

1. **集群**：Zookeeper是一个领导者（Leader），多个跟随者（Follower）组成的集群。

2. **高可用性**：集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。

3. **全局数据一致**：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。

4. **更新请求顺序进行**：来自同一个Client的更新请求按其发送顺序依次执行。

5. **数据更新原子性**：一次数据更新要么成功，要么失败。

6. **实时性**：在一定时间范围内，Client能读到最新数据。

7. 从设计模式角度来看，zk是一个基于**观察者设计模式**的框架，它负责管理跟存储大家都关心的数据，然后接受观察者的注册，数据反生变化zk会通知在zk上注册的观察者做出反应。

8. Zookeeper是一个分布式协调系统，满足CP性，跟SpringCloud中的Eureka满足AP不一样。



###  zookeeper数据模型

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815152657.png)

 **ZooKeeper** = **文件系统** + **监听通知机制**。

**Zookeeper**维护一个类似文件系统的树状数据结构，这种特性使得 **Zookeeper** 不能用于存放大量的数据，每个节点的存放数据上限为**1M**。每个子目录项如 NameService 都被称作为 **znode**(目录节点)。和文件系统一样，我们能够自由的增加、删除**znode**，在一个**znode**下增加、删除子**znode**，唯一的不同在于**znode**是可以存储数据的。默认有四种类型的**znode**：

1. **持久化目录节点 PERSISTENT**：客户端与zookeeper断开连接后，该节点依旧存在。
2. **持久化顺序编号目录节点 PERSISTENT_SEQUENTIAL**：客户端与zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号。
3. **临时目录节点 EPHEMERAL**：客户端与zookeeper断开连接后，该节点被删除。
4. **临时顺序编号目录节点 EPHEMERAL_SEQUENTIAL**：客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号。

每个节点都包含了一些列属性，通过get命令可以得到节点的属性。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815151519.png)

dataVersion：数据版本号，每次对节点进行 set 操作，dataVersion 的值都会增加 1（即使设置的是相同的数据），可有效避免了数据更新时出现的先后顺序问题。

cversion ：子节点的版本号。当 znode 的子节点有变化时，cversion 的值就会增加 1。

aclVersion ：ACL 的版本号。

cZxid ：Znode 创建的事务 id。

mZxid ：Znode 被修改的事务 id，即每次对 znode 的修改都会更新 mZxid。对于 zk 来说，每次的变化都会产生一个唯一的事务 id，zxid（ZooKeeper Transaction Id）。通过 zxid，可以确定更新操作的先后顺序。例如，如果 zxid1小于 zxid2，说明 zxid1 操作先于 zxid2 发生，zxid 对于整个 zk 都是唯一的，即使操作的是不同的 znode。 

ctime：节点创建时的时间戳. 

mtime：节点最新一次更新发生时的时间戳.

ephemeralOwner:如果该节点为临时节点, ephemeralOwner 值表示与该节点绑定的 session id. 如果不是, ephemeralOwner 值为 0.

在 client 和 server 通信之前,首先需要建立连接,该连接称session。连接建立后,如果发生连接超时、授权失败,或者显式关闭连接,连接便处于 CLOSED状态, 此时 session 结束。

### 监听通知机制

**Watcher** 监听机制是 **Zookeeper** 中非常重要的特性，我们基于 **Zookeeper** 上创建的节点，可以对这些节点绑定**监听**事件，比如可以监听节点数据变更、节点删除、子节点状态变更等事件，通过这个事件机制，可以基于 **Zookeeper** 实现分布式锁、集群管理等功能。

> 当数据发生变化的时候， **Zookeeper** 会产生一个 **Watcher** 事件，并且会发送到客户端。但是客户端只会收到一次通知。如果后续这个节点再次发生变化，那么之前设置 **Watcher**的客户端不会再次收到消息。（**Watcher** 是一次性的操作）。可以通过循环监听去达到永久监听效果。

ZooKeeper 的 Watcher 机制，总的来说可以分为三个过程：

> 客户端注册 Watcher，注册 watcher 有 3 种方式，getData、exists、getChildren。
>
> 服务器处理 Watcher 。
>
> 客户端回调 Watcher 客户端。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815153751.png)

监听流程:

1. 在main线程中创建Zookeeper客户端，这时就会创建两个线程，一个负责网络连接通信（connet），一个负责监听（listener）。
2. 通过connect线程将注册的监听事件发送给Zookeeper。
3. 在Zookeeper的注册监听器列表中将注册的监听事件添加到列表中。
4. Zookeeper监听到有数据或路径变化，就会将这个消息发送给listener线程。
5. listener线程内部调用了process()方法。

### Zookeeper 提供的功能

通过对 Zookeeper 中丰富的数据节点进行交叉使用，配合 **Watcher** 事件通知机制，可以非常方便的构建一系列分布式应用中涉及的核心功能，比如 **数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列**等功能。

#### 1. 数据发布/订阅

当某些数据由几个机器共享，且这些信息经常变化数据量还小的时候，这些数据就适合存储到ZK中。

**数据存储**：将数据存储到 Zookeeper 上的一个数据节点。

**数据获取**：应用在启动初始化节点从 Zookeeper 数据节点读取数据，并在该节点上注册一个数据变更 **Watcher**

**数据变更**：当变更数据时会更新 Zookeeper 对应节点数据，Zookeeper会将数据变更**通知**发到各客户端，客户端接到通知后重新读取变更后的数据即可。

#### 2.分布式锁

基于ZooKeeper的分布式锁一般有如下两种。

1. 保持独占：在zk中有一个唯一的临时节点，只有拿到节点的才可以操作数据，没拿到的线程就需要等待。**缺点**：可能引发羊群效应，第一个用完后瞬间有999个同时并发的线程向zk请求获得锁。

2. 控制时序：主要是避免了羊群效应，临时节点已经预先存在，所有想要获得锁的线程在它下面创建临时顺序编号目录节点，编号最小的获得锁，用完删除，后面的依次排队获取。

   ![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815154845.png)

#### 3.负载均衡

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815155021.png)

多个相同的jar包在不同的服务器上开启相同的服务，可以通过nginx在服务端进行负载均衡的配置。也可以通过ZooKeeper在客户端进行负载均衡配置。

1. 多个服务注册
2. 客户端获取中间件地址集合
3. 从集合中随机选一个服务执行任务

> **ZooKeeper**不存在单点问题，zab机制保证单点故障可重新选举一个leader只负责服务的注册与发现，不负责转发，减少一次数据交换（消费方与服务方直接通信），需要自己实现相应的负载均衡算法。
>
> **Nginx**存在单点问题，单点负载高数据量大,需要通过 **KeepAlived** + **LVS** 备机实现高可用。每次负载，都充当一次中间人转发角色，增加网络负载量（消费方与服务方间接通信），自带负载均衡算法。

#### 4. 命名服务

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815161227.png)

命名服务是指通过指定的名字来获取资源或者服务的地址，利用 zk 创建一个全局唯一的路径，这个路径就可以作为一个名字，指向集群中的集群，提供的服务的地址，或者一个远程的对象等等。

#### 5. 分布式协调/通知

1. 对于系统调度来说，用户更改zk某个节点的value， ZooKeeper会将这些变化发送给注册了这个节点的 watcher 的所有客户端，进行通知。

2. 对于执行情况汇报来说，每个工作进程都在目录下创建一个携带工作进度的临时节点，那么汇总的进程可以监控目录子节点的变化获得工作进度的实时的全局情况。

#### 6. 集群管理

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815161344.png)

大数据体系下的大部分集群服务好像都通过**ZooKeeper**管理的，其实管理的时候主要关注的就是机器的动态上下线跟**Leader**选举。

> 比如在**zookeeper**服务器端有一个**znode**叫 **/Configuration**，那么集群中每一个机器启动的时候都去这个节点下创建一个**EPHEMERAL**类型的节点，比如**server1** 创建**/Configuration/Server1**，**server2**创建**/Configuration /Server1**，然后**Server1**和**Server2**都**watch** **/Configuration** 这个父节点，那么也就是这个父节点下数据或者子节点变化都会通知到该节点进行**watch**的客户端。
>
> 利用ZooKeeper的**强一致性**，能够保证在分布式高并发情况下节点创建的全局唯一性，即：同时有多个客户端请求创建 /Master 节点，最终一定只有一个客户端请求能够创建成功。利用这个特性，就能很轻易的在分布式环境中进行集群选举了。
>
> 就是动态Master选举。这就要用到 **EPHEMERAL_SEQUENTIAL**类型节点的特性了，这样每个节点会自动被编号。允许所有请求都能够创建成功，但是得有个创建顺序，每次选取序列号**最小**的那个机器作为**Master** 。

###  Zookeeper Leader选举

**节点四种状态**

> 1. **LOOKING**：寻 找 Leader 状态。当服务器处于该状态时会认为当前集群中没有 Leader，因此需要进入 Leader 选举状态。
>
> 2. **FOLLOWING**：跟随者状态。处理客户端的非事务请求，转发事务请求给 Leader 服务器，参与事务请求 Proposal(提议) 的投票，参与 Leader 选举投票。
>
> 3. **LEADING**：领导者状态。事务请求的唯一调度和处理者，保证集群事务处理的顺序性，集群内部个服务器的调度者(管理follower,数据同步)。
>
> 4. **OBSERVING**：观察者状态。3.0 版本以后引入的一个服务器角色，在不影响集群事务处理能力的基础上提升集群的非事务处理能力，处理客户端的非事务请求，转发事务请求给 Leader 服务器，不参与任何形式的投票。

**Leader**的选举一般分为**启动时选举**跟Leader挂掉后的**运行时选举**。

#### 启动时leader选举

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815161824.png)

1. 服务器1启动，发起一次选举。

   > 服务器1投自己一票。此时服务器1票数一票，不够半数以上（3票），选举无法完成，服务器1状态保持为**LOOKING**。

2. 服务器2启动，再发起一次选举。

   > 服务器1和2分别投自己一票，此时服务器1发现服务器2的id比自己大，更改选票投给服务器2。此时服务器1票数0票，服务器2票数2票，不够半数以上（3票），选举无法完成。服务器1，2状态保持**LOOKING**。

3. 服务器3启动，发起一次选举。

   >与上面过程一样，服务器1和2先投自己一票，然后因为服务器3id最大，两者更改选票投给为服务器3。此次投票结果：服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数已经超过半数（3票），服务器3当选**Leader**。服务器1，2更改状态为**FOLLOWING**，服务器3更改状态为**LEADING**；

4. 服务器4启动，发起一次选举。

   > 此时服务器1、2、3已经不是**LOOKING**状态，不会更改选票信息，交换选票信息结果。服务器3为3票，服务器4为1票。此时服务器4服从多数，更改选票信息为服务器3，服务器4并更改状态为**FOLLOWING**。

5. 服务器5启动，发起一次选举

   > 同4一样投票给3，此时服务器3一共5票，服务器5为0票。服务器5并更改状态为**FOLLOWING**；

最终：**Leader**是服务器3，状态为**LEADING**。其余服务器是**Follower**，状态为**FOLLOWING**。

#### 运行时leader选举

运行时候如果Master节点崩溃了会走恢复模式，新Leader选出前会暂停对外服务，大致可以分为四个阶段 选举、发现、同步、广播。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815162157.png)



>zxid:
>
>​	**ZooKeeper** 采用全局递增的事务 Id 来标识，所有 proposal(提议)在被提出的时候加上了**ZooKeeper Transaction Id** ，zxid是64位的Long类型，**这是保证事务的顺序一致性的关键**。zxid中高32位表示纪元**epoch**，低32位表示事务标识**xid**。你可以认为zxid越大说明存储数据越新。
>
>1. 每个leader都会具有不同的**epoch**值，表示一个纪元/朝代，用来标识 **leader** 周期。每个新的选举开启时都会生成一个新的**epoch**，新的leader产生的话**epoch**会自增，会将该值更新到所有的zkServer的**zxid**和**epoch**，
>2. **xid**是一个依次递增的事务编号。数值越大说明数据越新，所有 proposal（提议）在被提出的时候加上了**zxid**，然后会依据数据库的两阶段过程，首先会向其他的 server 发出事务执行请求，如果超过半数的机器都能执行并且能够成功，那么就会开始执行。



1. 每个Server会发出一个投票，第一次都是投自己，其中投票信息 = (myid，ZXID)

2. 收集来自各个服务器的投票

3. 处理投票并重新投票，处理逻辑：**优先比较ZXID，然后比较myid**。

4. 统计投票，只要超过半数的机器接收到同样的投票信息，就可以确定leader，注意epoch的增加跟同步。

5. 改变服务器状态Looking变为Following或Leading。

6. 当 Follower 链接上 Leader 之后，Leader 服务器会根据自己服务器上最后被提交的 ZXID 和 Follower 上的 ZXID 进行比对，比对结果要么回滚，要么和 Leader 同步，保证集群中各个节点的事务一致。

7. 集群恢复到广播模式，开始接受客户端的写请求。

### Curator

**Apache Curator**是一个比较完善的`ZooKeeper`客户端框架，通过封装的一套高级`API` 简化了`ZooKeeper`的操作。通过查看官方文档，可以发现`Curator`主要解决了三类问题：

1. 封装`ZooKeeper client`与`ZooKeeper server`之间的连接处理。
2. 提供了一套`Fluent`风格的操作`API`。
3. 提供`ZooKeeper`各种应用场景(`recipe`， 比如：分布式锁服务、集群领导选举、共享计数器、缓存机制、分布式队列等)的抽象封装。

目前 Curator 有 2.x.x 和 3.x.x 两个系列的版本，支持不同版本的 Zookeeper。其中Curator 2.x.x 兼容 Zookeeper的 3.4.x 和 3.5.x

**Curator的组件**

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815163308.png)

**Maven的依赖**

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.10.0</version>
</dependency>
```

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815163320.png)

在实际的应用时，最常用的是`curator-recipes`，它可以实现：

- **锁**：包括共享锁、共享可重入锁、读写锁等。
- **选举**：`Leader`选举算法。
- **Barrier**：阻止分布式计算直至某个条件被满足的“栅栏”，可以看做`JDK Concurrent`包中`Barrier`的分布式实现。
- **缓存**：三种`Cache`及监听机制。
- **持久化结点**：连接或`Session`终止后仍然在`ZooKeeper`中存在的结点。
- **队列**：分布式队列、分布式优先级队列等。

#### Curator 客户端的初始化和初始化时机

在实际的工程中，Zookeeper 客户端的初始化会在程序启动期间完成。

##### 初始化时机

在 Spring 或者 SpringBoot 工程中最常见的就是绑定到容器启动的生命周期或者应用启动的生命周期中：

- 监听 ContextRefreshedEvent 事件，在容器刷新完成之后初始化 Zookeeper
- 监听 ApplicationReadyEvent/ApplicationStartedEvent 事件，初始化 Zookeeper 客户端
- 实现 InitializingBean 接口 ，在 afterPropertiesSet 中完成 Zookeeper 客户端初始化

##### Curator 初始化

```java
public class ZookeeperCuratorClient implements InitializingBean {
    private CuratorFramework curatorClient;
    @Value("${zookeeper.address:localhost:2181}")
    private String           connectString; //要连接的服务器列表
    @Value("${zookeeper.baseSleepTimeMs:1000}")
    private int              baseSleepTimeMs; //重试之间等待的初始时间
    @Value("${zookeeper.maxRetries:3}")
    private int              maxRetries; //最大重试次数
    @Value("${zookeeper.sessionTimeoutMs:6000}")
    private int              sessionTimeoutMs; //session 超时时间
    @Value("${zookeeper.connectionTimeoutMs:6000}")
    private int              connectionTimeoutMs; //连接超时时间
  
    @Override
    public void afterPropertiesSet() throws Exception {
        // custom policy
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(baseSleepTimeMs, maxRetries);
        // to build curatorClient
        curatorClient = CuratorFrameworkFactory.builder().connectString(connectString)
                .sessionTimeoutMs(sessionTimeoutMs).connectionTimeoutMs(connectionTimeoutMs)
                .retryPolicy(retryPolicy).build();
        curatorClient.start();
    }

    public CuratorFramework getCuratorClient() {
        return curatorClient;
    }
}
```

在连接 Zookeeper 时，Curator 提供了多种重试策略以满足各种需求，所有重试策略均继承自 `RetryPolicy` 接口

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815163858.png)



重试策略类主要分为以下两类：

- **RetryForever** ：代表一直重试，直到连接成功；
- **SleepingRetry** ： 基于一定间隔时间的重试。
  - RetryOneTime：只重连一次
  - RetryNTime：指定重连的次数N
  - RetryUtilElapsed：指定最大重连超时时间和重连时间间隔，间歇性重连直到超时或者链接成功
  - ExponentialBackoffRetry：基于 "backoff"方式重连，和 RetryUtilElapsed 的区别是重连的时间间隔是动态的。
  - BoundedExponentialBackoffRetry： 同 ExponentialBackoffRetry的区别是增加了最大重试次数的控制

> 在一些场景中，需要对不同的业务进行隔离，这种情况下，可以通过设置 namespace 来解决，namespace 实际上就是指定zookeeper的根路径，设置之后，后面的所有操作都会基于该根目录。

#### 节点增删改查

1. 创建节点

   > 在递归调用中，如果不指定 CreateMode，则默认`PERSISTENT`，如果指定为临时节点，则最终节点会是临时节点，父节点仍旧是`PERSISTENT`

```java
client.create().creatingParentsIfNeeded()        //递归创建
            .withMode(CreateMode.PERSISTENT)      //节点类型
            .forPath(nodePath, data);
递归方式创建节点有两个方法，creatingParentsIfNeeded 和 creatingParentContainersIfNeeded。在新版本的 zookeeper 这两个递归创建方法会有区别； creatingParentContainersIfNeeded() 以容器模式递归创建节点，如果旧版本 zookeeper，此方法等于creatingParentsIfNeeded()。


创建时可以指定节点类型，这里的节点类型和 Zookeeper 原生的一致，全部类型定义在枚举类 CreateMode 中：
    public enum CreateMode {
    // 永久节点
    PERSISTENT (0, false, false),
    //永久有序节点
    PERSISTENT_SEQUENTIAL (2, false, true),
    // 临时节点
    EPHEMERAL (1, true, false),
    // 临时有序节点
    EPHEMERAL_SEQUENTIAL (3, true, true);
    ....
}
```

2. 获取节点信息

```java
   根据配置的压缩提供程序对数据进行解压缩处理
   byte[] data = curatorClient.getData().decompressed().forPath("/glmapper/test");
  

	Stat stat = new Stat();
    byte[] data = client.getData().storingStatIn(stat).forPath(nodePath);
	
节点信息被封装在 Stat 类中，其主要属性如下：
public class Stat implements Record {
    private long czxid;
    private long mzxid;
    private long ctime;
    private long mtime;
    private int version;
    private int cversion;
    private int aversion;
    private long ephemeralOwner;
    private int dataLength;
    private int numChildren;
    private long pzxid;
    ...
}

```

| **状态属性**   | **说明**                                                     |
| -------------- | ------------------------------------------------------------ |
| czxid          | 数据节点创建时的事务 ID                                      |
| ctime          | 数据节点创建时的时间                                         |
| mzxid          | 数据节点最后一次更新时的事务 ID                              |
| mtime          | 数据节点最后一次更新时的时间                                 |
| pzxid          | 数据节点的子节点最后一次被修改时的事务 ID                    |
| cversion       | 子节点的更改次数                                             |
| version        | 节点数据的更改次数                                           |
| aversion       | 节点的 ACL 的更改次数                                        |
| ephemeralOwner | 如果节点是临时节点，则表示创建该节点的会话的 SessionID；如果节点是持久节点，则该属性值为 0 |
| dataLength     | 数据内容的长度                                               |
| numChildren    | 数据节点当前的子节点个数                                     |

3. 获取子节点列表

```java
    List<String> childNodes = client.getChildren().forPath("/hadoop");
```

4. 更新节点

   > 更新时可以传入版本号也可以不传入，如果传入则类似于乐观锁机制，只有在版本号正确的时候才会被更新。

   ```java
       byte[] newData = "defg".getBytes();
       client.setData().withVersion(0)     // 传入版本号，如果版本号错误则拒绝更新操作,并抛出 BadVersion 异常
               .forPath(nodePath, newData);
   	//设置数据并使用配置的压缩提供程序压缩数据
   	curatorClient.setData().compressed().forPath("/glmapper/test","newData".getBytes());
   
   ```

5. 删除节点

   > 使用 guaranteed 方式删除，guaranteed 会保证在session有效的情况下，后台持续进行该节点的删除操作，直到删除掉

   ```java
   client.delete()
               .guaranteed()                // 如果删除失败，那么在会继续执行，直到成功
               .deletingChildrenIfNeeded()  // 如果有子节点，则递归删除
               .withVersion(0)              // 传入版本号，如果版本号错误则拒绝删除操作,并抛出 BadVersion 异常
               .forPath(nodePath);
   ```

6. 判断节点是否存在

   ```java
       // 如果节点存在则返回其状态信息如果不存在则为 null
       Stat stat = client.checkExists().forPath(nodePath + "aa/bb/cc");
   ```

#### 监听事件

1. 创建一次性监听

   和 Zookeeper 原生监听一样，使用 `usingWatcher` 注册的监听是一次性的，即监听只会触发一次，触发后就销毁。

   ```java
       client.getData().usingWatcher(new CuratorWatcher() {
           public void process(WatchedEvent event) {
               System.out.println("节点" + event.getPath() + "发生了事件:" + event.getType());
           }
       }).forPath(nodePath);
   ```

2. CuratorListener 方式

   > CuratorListener 监听，此监听主要针对 background 通知和错误通知。使用此监听器之后，调用inBackground 方法会异步获得监听，对于节点的创建或修改则不会触发监听事件。

   ```java
   CuratorListener listener = new CuratorListener(){
       @Override
       public void eventReceived(CuratorFramework client, CuratorEvent event) throws Exception {
         System.out.println("event : " + event);
       }
    };
   // 绑定监听器
   curatorClient.getCuratorListenable().addListener(listener);
   // 异步获取节点数据
   curatorClient.getData().inBackground().forPath("/glmapper/test");
   // 更新节点数据
   curatorClient.setData().forPath("/glmapper/test","newData".getBytes());
   ```

   这里只触发了一次监听回调，就是 getData 。

3. 创建永久监听

   三种`Cache`及监听机制，值得是`Curator`提供了三种`Watcher`(`Cache`)来监听结点的变化：

   - **Path Cache**：监视一个路径下1）孩子结点的创建、2）删除，3）以及结点数据的更新。产生的事件会传递给注册的`PathChildrenCacheListener`。
   - **Node Cache**：监视一个结点的创建、更新、删除，并将结点的数据缓存在本地。
   - **Tree Cache**：`Path Cache`和`Node Cache`的“合体”，监视路径下的创建、更新、删除事件，并缓存路径下所有孩子结点的数据。

   > nodeCache.start()，NodeCache 在先后多次修改监听节点的内容时，出现了丢失事件现象，在用例执行的5次中，仅一次监听到了全部事件；如果 nodeCache.start(true)，NodeCache 在先后多次修改监听节点的内容时，不会出现丢失现象。
   >
   > NodeCache不仅可以监听节点内容变化，还可以监听指定节点是否存在。如果原本节点不存在，那么Cache就会在节点被创建时触发监听事件，如果该节点被删除，就无法再触发监听事件。

   1. NodeCache

      > 监听数据节点本身的变化。对节点的监听需要配合回调函数来进行处理接收到监听事件之后的业务处理。NodeCache 通过 NodeCacheListener 来完成后续处理。

   ```java
      // 使用 NodeCache 包装节点，对其注册的监听作用于节点，且是永久性的
       NodeCache nodeCache = new NodeCache(client, nodePath);
       // 通常设置为 true, 代表创建 nodeCache 时,就去获取对应节点的值并缓存
       nodeCache.start(true);
       nodeCache.getListenable().addListener(new NodeCacheListener() {
           public void nodeChanged() {
               ChildData currentData = nodeCache.getCurrentData();
               if (currentData != null) {
                   System.out.println("节点路径：" + currentData.getPath() +
                           "数据：" + new String(currentData.getData()));
               }
           }
       });
   ```

   2. PathChildrenCache节点监听

      > PathChildrenCache 不会对二级子节点进行监听，只会对子节点进行监听。

   ```java
     PathChildrenCache pathChildrenCache = new PathChildrenCache(curatorClient,path,true);
   // 如果设置为true则在首次启动时就会缓存节点内容到Cache中。 nodeCache.start(true);
   pathChildrenCache.start(PathChildrenCache.StartMode.POST_INITIALIZED_EVENT);
   pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
     @Override
     public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent event) throws Exception {
       System.out.println("-----------------------------");
       System.out.println("event:"  + event.getType());
       if (event.getData()!=null){
         System.out.println("path:" + event.getData().getPath());
       }
       System.out.println("-----------------------------");
     }
   });
   ```

   3. TreeCache

      > TreeCache 使用一个内部类`TreeNode`来维护这个一个树结构。并将这个树结构与ZK节点进行了映射。所以TreeCache 可以监听当前节点下所有节点的事件。

      ```java
      String path = "/glmapper";
      TreeCache treeCache = new TreeCache(curatorClient,path);
      treeCache.getListenable().addListener((client,event)-> {
          System.out.println("-----------------------------");
          System.out.println("event:"  + event.getType());
          if (event.getData()!=null){
            System.out.println("path:" + event.getData().getPath());
          }
          System.out.println("-----------------------------");
      });
      treeCache.start();
      
      ```

#### 事务操作

>  CuratorFramework 的实例包含 inTransaction( ) 接口方法，调用此方法开启一个 ZooKeeper 事务。 可以复合create、 setData、 check、and/or delete 等操作然后调用 commit() 作为一个原子操作提交。

```java
// 开启事务  
CuratorTransaction curatorTransaction = curatorClient.inTransaction();
Collection<CuratorTransactionResult> commit = 
  // 操作1 
curatorTransaction.create().withMode(CreateMode.EPHEMERAL).forPath("/glmapper/transaction")
  .and()
  // 操作2 
  .delete().forPath("/glmapper/test")
  .and()
  // 操作3
  .setData().forPath("/glmapper/transaction", "data".getBytes())
  .and()
  // 提交事务
  .commit();
Iterator<CuratorTransactionResult> iterator = commit.iterator();
while (iterator.hasNext()){
  CuratorTransactionResult next = iterator.next();
  System.out.println(next.getForPath());
  System.out.println(next.getResultPath());
  System.out.println(next.getType());
}
```

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210815171728.png)

#### 异步操作

> 前面提到的增删改查都是同步的，但是 Curator 也提供了异步接口，引入了 BackgroundCallback 接口用于处理异步接口调用之后服务端返回的结果信息。BackgroundCallback 接口中一个重要的回调值为 CuratorEvent，里面包含事件类型、响应吗和节点的详细信息。

在使用上也是非常简单的，只需要带上 inBackground() 就行

```java
 curatorClient.getData().inBackground().forPath("/glmapper/test");
public T inBackground(BackgroundCallback callback, Executor executor);

```



#### Curator实现分布式锁

##### 1. 可重入锁Shared Reentrant Lock

> Shared意味着锁是全局可见的， 客户端都可以请求锁。 Reentrant和JDK的ReentrantLock类似， 意味着同一个客户端在拥有锁的同时，可以多次获取，不会被阻塞。 它是由类InterProcessMutex来实现。

```java
public InterProcessMutex(CuratorFramework client, String path)
    
通过acquire获得锁，并提供超时机制：
public void acquire();
public boolean acquire(long time, TimeUnit unit);

通过release()方法释放锁。
release()
```

**Revoking** ZooKeeper recipes wiki定义了可协商的撤销机制。 为了撤销mutex, 调用下面的方法：

```java
public void makeRevocable(RevocationListener<T> listener)
/**
将锁设为可撤销的. 当别的进程或线程想让你释放锁时Listener会被调用。
Parameters:
listener - the listener
**/
```

如果你请求撤销当前的锁， 调用`attemptRevoke()`方法,注意锁释放时`RevocationListener`将会回调。

```java
public static void attemptRevoke(CuratorFramework client,String path) throws Exception
```

首先创建一个模拟的共享资源， 这个资源期望只能单线程的访问，否则会有并发问题。

```java
public class FakeLimitedResource {
	private final AtomicBoolean inUse = new AtomicBoolean(false);

	public void use() throws InterruptedException {
		// 真实环境中我们会在这里访问/维护一个共享的资源
		//这个例子在使用锁的情况下不会非法并发异常IllegalStateException
		//但是在无锁的情况由于sleep了一段时间，很容易抛出异常
		if (!inUse.compareAndSet(false, true)) {
			throw new IllegalStateException("Needs to be used by one client at a time");
		}
		try {
			Thread.sleep((long) (3 * Math.random()));
		} finally {
			inUse.set(false);
		}
	}
}
```

然后创建一个`InterProcessMutexDemo`类， 它负责请求锁， 使用资源，释放锁这样一个完整的访问过程。

```java
public class InterProcessMutexDemo {

	private InterProcessMutex lock;
	private final FakeLimitedResource resource;
	private final String clientName;

	public InterProcessMutexDemo(CuratorFramework client, String lockPath, FakeLimitedResource resource, String clientName) {
		this.resource = resource;
		this.clientName = clientName;
		this.lock = new InterProcessMutex(client, lockPath);
	}

	public void doWork(long time, TimeUnit unit) throws Exception {
		if (!lock.acquire(time, unit)) {
			throw new IllegalStateException(clientName + " could not acquire the lock");
		}
		try {
			System.out.println(clientName + " get the lock");
			resource.use(); //access resource exclusively
		} finally {
			System.out.println(clientName + " releasing the lock");
			lock.release(); // always release the lock in a finally block
		}
	}

	private static final int QTY = 5;
	private static final int REPETITIONS = QTY * 10;
	private static final String PATH = "/examples/locks";

	public static void main(String[] args) throws Exception {
		final FakeLimitedResource resource = new FakeLimitedResource();
		ExecutorService service = Executors.newFixedThreadPool(QTY);
		final TestingServer server = new TestingServer();
		try {
			for (int i = 0; i < QTY; ++i) {
				final int index = i;
				Callable<Void> task = new Callable<Void>() {
					@Override
					public Void call() throws Exception {
						CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
						try {
							client.start();
							final InterProcessMutexDemo example = new InterProcessMutexDemo(client, PATH, resource, "Client " + index);
							for (int j = 0; j < REPETITIONS; ++j) {
								example.doWork(10, TimeUnit.SECONDS);
							}
						} catch (Throwable e) {
							e.printStackTrace();
						} finally {
							CloseableUtils.closeQuietly(client);
						}
						return null;
					}
				};
				service.submit(task);
			}
			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);
		} finally {
			CloseableUtils.closeQuietly(server);
		}
	}
}
```

代码也很简单，生成10个client， 每个client重复执行10次 请求锁–访问资源–释放锁的过程。每个client都在独立的线程中。 结果可以看到，锁是随机的被每个实例排他性的使用。既然是可重用的，你可以在一个线程中多次调用`acquire()`,在线程拥有锁时它总是返回true。**你不应该在多个线程中用同一个InterProcessMutex**， 你可以在每个线程中都生成一个新的InterProcessMutex实例，它们的path都一样，这样它们可以共享同一个锁。

##### 2.不可重入共享锁—Shared Lock

> 这个锁和`InterProcessMutex`相比，就是少了Reentrant的功能，也就意味着它不能在同一个线程中重入。这个类是`InterProcessSemaphoreMutex`,使用方法和`InterProcessMutex`类似

```java
public class InterProcessSemaphoreMutexDemo {

	private InterProcessSemaphoreMutex lock;
	private final FakeLimitedResource resource;
	private final String clientName;

	public InterProcessSemaphoreMutexDemo(CuratorFramework client, String lockPath, FakeLimitedResource resource, String clientName) {
		this.resource = resource;
		this.clientName = clientName;
		this.lock = new InterProcessSemaphoreMutex(client, lockPath);
	}

	public void doWork(long time, TimeUnit unit) throws Exception {
		if (!lock.acquire(time, unit))
		{
			throw new IllegalStateException(clientName + " 不能得到互斥锁");
		}
		System.out.println(clientName + " 已获取到互斥锁");
		if (!lock.acquire(time, unit))
		{
			throw new IllegalStateException(clientName + " 不能得到互斥锁");
		}
		System.out.println(clientName + " 再次获取到互斥锁");
		try {
			System.out.println(clientName + " get the lock");
			resource.use(); //access resource exclusively
		} finally {
			System.out.println(clientName + " releasing the lock");
			lock.release(); // always release the lock in a finally block
			lock.release(); // 获取锁几次 释放锁也要几次
		}
	}

	private static final int QTY = 5;
	private static final int REPETITIONS = QTY * 10;
	private static final String PATH = "/examples/locks";

	public static void main(String[] args) throws Exception {
		final FakeLimitedResource resource = new FakeLimitedResource();
		ExecutorService service = Executors.newFixedThreadPool(QTY);
		final TestingServer server = new TestingServer();
		try {
			for (int i = 0; i < QTY; ++i) {
				final int index = i;
				Callable<Void> task = new Callable<Void>() {
					@Override
					public Void call() throws Exception {
						CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
						try {
							client.start();
							final InterProcessSemaphoreMutexDemo example = new InterProcessSemaphoreMutexDemo(client, PATH, resource, "Client " + index);
							for (int j = 0; j < REPETITIONS; ++j) {
								example.doWork(10, TimeUnit.SECONDS);
							}
						} catch (Throwable e) {
							e.printStackTrace();
						} finally {
							CloseableUtils.closeQuietly(client);
						}
						return null;
					}
				};
				service.submit(task);
			}
			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);
		} finally {
			CloseableUtils.closeQuietly(server);
		}
		Thread.sleep(Integer.MAX_VALUE);
	}
}
```

运行后发现，有且只有一个client成功获取第一个锁(第一个`acquire()`方法返回true)，然后它自己阻塞在第二个`acquire()`方法，获取第二个锁超时；其他所有的客户端都阻塞在第一个`acquire()`方法超时并且抛出异常。

这样也就验证了`InterProcessSemaphoreMutex`实现的锁是不可重入的。

##### 3.可重入读写锁—Shared Reentrant Read Write Lock

>  类似JDK的**ReentrantReadWriteLock**。一个读写锁管理一对相关的锁。一个负责读操作，另外一个负责写操作。读操作在写锁没被使用时可同时由多个进程使用，而写锁在使用时不允许读(阻塞)。
>
> 此锁是可重入的。**一个拥有写锁的线程可重入读锁，但是读锁却不能进入写锁**。这也意味着**写锁可以降级成读锁， 比如请求写锁 —>请求读锁—>释放读锁 ---->释放写锁**。从读锁升级成写锁是不行的。

可重入读写锁主要由两个类实现：`InterProcessReadWriteLock`、`InterProcessMutex`。使用时首先创建一个`InterProcessReadWriteLock`实例，然后再根据你的需求得到读锁或者写锁，读写锁的类型是`InterProcessMutex`。

```java
public class ReentrantReadWriteLockDemo {

	private final InterProcessReadWriteLock lock;
	private final InterProcessMutex readLock;
	private final InterProcessMutex writeLock;
	private final FakeLimitedResource resource;
	private final String clientName;

	public ReentrantReadWriteLockDemo(CuratorFramework client, String lockPath, FakeLimitedResource resource, String clientName) {
		this.resource = resource;
		this.clientName = clientName;
		lock = new InterProcessReadWriteLock(client, lockPath);
		readLock = lock.readLock();
		writeLock = lock.writeLock();
	}

	public void doWork(long time, TimeUnit unit) throws Exception {
		// 注意只能先得到写锁再得到读锁，不能反过来！！！
		if (!writeLock.acquire(time, unit)) {
			throw new IllegalStateException(clientName + " 不能得到写锁");
		}
		System.out.println(clientName + " 已得到写锁");
		if (!readLock.acquire(time, unit)) {
			throw new IllegalStateException(clientName + " 不能得到读锁");
		}
		System.out.println(clientName + " 已得到读锁");
		try {
			resource.use(); // 使用资源
			Thread.sleep(1000);
		} finally {
			System.out.println(clientName + " 释放读写锁");
			readLock.release();
			writeLock.release();
		}
	}

	private static final int QTY = 5;
	private static final int REPETITIONS = QTY ;
	private static final String PATH = "/examples/locks";

	public static void main(String[] args) throws Exception {
		final FakeLimitedResource resource = new FakeLimitedResource();
		ExecutorService service = Executors.newFixedThreadPool(QTY);
		final TestingServer server = new TestingServer();
		try {
			for (int i = 0; i < QTY; ++i) {
				final int index = i;
				Callable<Void> task = new Callable<Void>() {
					@Override
					public Void call() throws Exception {
						CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
						try {
							client.start();
							final ReentrantReadWriteLockDemo example = new ReentrantReadWriteLockDemo(client, PATH, resource, "Client " + index);
							for (int j = 0; j < REPETITIONS; ++j) {
								example.doWork(10, TimeUnit.SECONDS);
							}
						} catch (Throwable e) {
							e.printStackTrace();
						} finally {
							CloseableUtils.closeQuietly(client);
						}
						return null;
					}
				};
				service.submit(task);
			}
			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);
		} finally {
			CloseableUtils.closeQuietly(server);
		}
	}
}

```

##### 4.信号量—Shared Semaphore

> 一个计数的信号量类似JDK的Semaphore。 JDK中Semaphore维护的一组许可(**permits**)，而Curator中称之为租约(**Lease**)。 有两种方式可以决定semaphore的最大租约数。第一种方式是用户给定path并且指定最大LeaseSize。第二种方式用户给定path并且使用`SharedCountReader`类。**如果不使用SharedCountReader, 必须保证所有实例在多进程中使用相同的(最大)租约数量,否则有可能出现A进程中的实例持有最大租约数量为10，但是在B进程中持有的最大租约数量为20，此时租约的意义就失效了。**

这次调用`acquire()`会返回一个租约对象。 客户端必须在finally中close这些租约对象，否则这些租约会丢失掉。 但是， 但是，如果客户端session由于某种原因比如crash丢掉， 那么这些客户端持有的租约会自动close， 这样其它客户端可以继续使用这些租约。 租约还可以通过下面的方式返还：

```java
public void returnAll(Collection<Lease> leases)
public void returnLease(Lease lease)
```

注意你可以一次性请求多个租约，如果Semaphore当前的租约不够，则请求线程会被阻塞。 同时还提供了超时的重载方法。

```java
public Lease acquire()
public Collection<Lease> acquire(int qty)
public Lease acquire(long time, TimeUnit unit)
public Collection<Lease> acquire(int qty, long time, TimeUnit unit)
```

Shared Semaphore使用的主要类包括下面几个：

- `InterProcessSemaphoreV2`
- `Lease`
- `SharedCountReader`

```java
public class InterProcessSemaphoreDemo {

	private static final int MAX_LEASE = 10;
	private static final String PATH = "/examples/locks";

	public static void main(String[] args) throws Exception {
		FakeLimitedResource resource = new FakeLimitedResource();
		try (TestingServer server = new TestingServer()) {

			CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();

			InterProcessSemaphoreV2 semaphore = new InterProcessSemaphoreV2(client, PATH, MAX_LEASE);
			Collection<Lease> leases = semaphore.acquire(5);
			System.out.println("get " + leases.size() + " leases");
			Lease lease = semaphore.acquire();
			System.out.println("get another lease");

			resource.use();

			Collection<Lease> leases2 = semaphore.acquire(5, 10, TimeUnit.SECONDS);
			System.out.println("Should timeout and acquire return " + leases2);

			System.out.println("return one lease");
			semaphore.returnLease(lease);
			System.out.println("return another 5 leases");
			semaphore.returnAll(leases);
		}
	}
}

```

首先我们先获得了5个租约， 最后我们把它还给了semaphore。 接着请求了一个租约，因为semaphore还有5个租约，所以请求可以满足，返回一个租约，还剩4个租约。 然后再请求一个租约，因为租约不够，**阻塞到超时，还是没能满足，返回结果为null(租约不足会阻塞到超时，然后返回null，不会主动抛出异常；如果不设置超时时间，会一致阻塞)。**

上面说讲的锁都是公平锁(fair)。 总ZooKeeper的角度看， 每个客户端都按照请求的顺序获得锁，不存在非公平的抢占的情况。

#### 分布式计数器

> 顾名思义，计数器是用来计数的, 利用ZooKeeper可以实现一个集群共享的计数器。 只要使用相同的path就可以得到最新的计数器值， 这是由ZooKeeper的一致性保证的。Curator有两个计数器， 一个是用int来计数(`SharedCount`)，一个用long来计数(`DistributedAtomicLong`)。

##### 分布式int计数器—SharedCount

这个类使用int类型来计数。 主要涉及三个类。

- SharedCount
- SharedCountReader
- SharedCountListener

`SharedCount`代表计数器， 可以为它增加一个`SharedCountListener`，当计数器改变时此Listener可以监听到改变的事件，而`SharedCountReader`可以读取到最新的值， 包括字面值和带版本信息的值VersionedValue。

```java
public class SharedCounterDemo implements SharedCountListener {

	private static final int QTY = 5;
	private static final String PATH = "/examples/counter";

	public static void main(String[] args) throws IOException, Exception {
		final Random rand = new Random();
		SharedCounterDemo example = new SharedCounterDemo();
		try (TestingServer server = new TestingServer()) {
			CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();

			SharedCount baseCount = new SharedCount(client, PATH, 0);
			baseCount.addListener(example);
			baseCount.start();

			List<SharedCount> examples = Lists.newArrayList();
			ExecutorService service = Executors.newFixedThreadPool(QTY);
			for (int i = 0; i < QTY; ++i) {
				final SharedCount count = new SharedCount(client, PATH, 0);
				examples.add(count);
				Callable<Void> task = () -> {
					count.start();
					Thread.sleep(rand.nextInt(10000));
					System.out.println("Increment:" + count.trySetCount(count.getVersionedValue(), count.getCount() + rand.nextInt(10)));
					return null;
				};
				service.submit(task);
			}

			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);

			for (int i = 0; i < QTY; ++i) {
				examples.get(i).close();
			}
			baseCount.close();
		}
		Thread.sleep(Integer.MAX_VALUE);
	}

	@Override
	public void stateChanged(CuratorFramework arg0, ConnectionState arg1) {
		System.out.println("State changed: " + arg1.toString());
	}

	@Override
	public void countHasChanged(SharedCountReader sharedCount, int newCount) throws Exception {
		System.out.println("Counter's value is changed to " + newCount);
	}
}

```

在这个例子中，我们使用`baseCount`来监听计数值(`addListener`方法来添加SharedCountListener )。 任意的SharedCount， 只要使用相同的path，都可以得到这个计数值。 然后我们使用5个线程为计数值增加一个10以内的随机数。相同的path的SharedCount对计数值进行更改，将会回调给`baseCount`的SharedCountListener。

```java
count.trySetCount(count.getVersionedValue(), count.getCount() + rand.nextInt(10))
```

这里我们使用`trySetCount`去设置计数器。 **第一个参数提供当前的VersionedValue,如果期间其它client更新了此计数值， 你的更新可能不成功， 但是这时你的client更新了最新的值，所以失败了你可以尝试再更新一次。 而setCount是强制更新计数器的值**。

注意计数器必须`start`,使用完之后必须调用`close`关闭它。

强烈推荐使用`ConnectionStateListener`。 在本例中`SharedCountListener`扩展`ConnectionStateListener`。

##### 分布式long计数器—DistributedAtomicLong

> Long类型的计数器。 除了计数的范围比`SharedCount`大了之外， 它首先尝试使用乐观锁的方式设置计数器， 如果不成功(比如期间计数器已经被其它client更新了)， 它使用`InterProcessMutex`方式来更新计数值。

此计数器有一系列的操作：

- get(): 获取当前值
- increment()： 加一
- decrement(): 减一
- add()： 增加特定的值
- subtract(): 减去特定的值
- trySet(): 尝试设置计数值
- forceSet(): 强制设置计数值

你**必须**检查返回结果的`succeeded()`， 它代表此操作是否成功。 如果操作成功， `preValue()`代表操作前的值， `postValue()`代表操作后的值。

```java
public class DistributedAtomicLongDemo {

	private static final int QTY = 5;
	private static final String PATH = "/examples/counter";

	public static void main(String[] args) throws IOException, Exception {
		List<DistributedAtomicLong> examples = Lists.newArrayList();
		try (TestingServer server = new TestingServer()) {
			CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			ExecutorService service = Executors.newFixedThreadPool(QTY);
			for (int i = 0; i < QTY; ++i) {
				final DistributedAtomicLong count = new DistributedAtomicLong(client, PATH, new RetryNTimes(10, 10));

				examples.add(count);
				Callable<Void> task = () -> {
					try {
						AtomicValue<Long> value = count.increment();
						System.out.println("succeed: " + value.succeeded());
						if (value.succeeded())
							System.out.println("Increment: from " + value.preValue() + " to " + value.postValue());
					} catch (Exception e) {
						e.printStackTrace();
					}
					return null;
				};
				service.submit(task);
			}

			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);
			Thread.sleep(Integer.MAX_VALUE);
		}
	}
}

```

#### 分布式屏障—Barrier

> 分布式Barrier是这样一个类： 它会阻塞所有节点上的等待进程，直到某一个被满足， 然后所有的节点继续进行。比如赛马比赛中， 等赛马陆续来到起跑线前。 一声令下，所有的赛马都飞奔而出。

##### DistributedBarrier

`DistributedBarrier`类实现了栅栏的功能。 它的构造函数如下：

```java
public DistributedBarrier(CuratorFramework client, String barrierPath)
Parameters:
client - client
barrierPath - path to use as the barrier
```

首先你需要设置栅栏，它将阻塞在它上面等待的线程:

```java
setBarrier();
```

然后需要阻塞的线程调用方法等待放行条件:

```java
public void waitOnBarrier()
```

当条件满足时，移除栅栏，所有等待的线程将继续执行：

```java
removeBarrier();
```

**异常处理** DistributedBarrier 会监控连接状态，当连接断掉时`waitOnBarrier()`方法会抛出异常。

```java
public class DistributedBarrierDemo {

	private static final int QTY = 5;
	private static final String PATH = "/examples/barrier";

	public static void main(String[] args) throws Exception {
		try (TestingServer server = new TestingServer()) {
			CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			ExecutorService service = Executors.newFixedThreadPool(QTY);
			DistributedBarrier controlBarrier = new DistributedBarrier(client, PATH);
			controlBarrier.setBarrier();

			for (int i = 0; i < QTY; ++i) {
				final DistributedBarrier barrier = new DistributedBarrier(client, PATH);
				final int index = i;
				Callable<Void> task = () -> {
					Thread.sleep((long) (3 * Math.random()));
					System.out.println("Client #" + index + " waits on Barrier");
					barrier.waitOnBarrier();
					System.out.println("Client #" + index + " begins");
					return null;
				};
				service.submit(task);
			}
			Thread.sleep(10000);
			System.out.println("all Barrier instances should wait the condition");
			controlBarrier.removeBarrier();
			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);

			Thread.sleep(20000);
		}
	}
}


```

这个例子创建了`controlBarrier`来设置栅栏和移除栅栏。 我们创建了5个线程，在此Barrier上等待。 最后移除栅栏后所有的线程才继续执行。

如果你开始不设置栅栏，所有的线程就不会阻塞住。

##### 双栅栏—DistributedDoubleBarrier

双栅栏允许客户端在计算的开始和结束时同步。当足够的进程加入到双栅栏时，进程开始计算， 当计算完成时，离开栅栏。 双栅栏类是`DistributedDoubleBarrier`。 构造函数为:

```java
public DistributedDoubleBarrier(CuratorFramework client,
                                String barrierPath,
                                int memberQty)
Creates the barrier abstraction. memberQty is the number of members in the barrier. When enter() is called, it blocks until
all members have entered. When leave() is called, it blocks until all members have left.

Parameters:
client - the client
barrierPath - path to use
memberQty - the number of members in the barrier


```

`memberQty`是成员数量，当`enter()`方法被调用时，成员被阻塞，直到所有的成员都调用了`enter()`。 当`leave()`方法被调用时，它也阻塞调用线程，直到所有的成员都调用了`leave()`。 就像百米赛跑比赛， 发令枪响， 所有的运动员开始跑，等所有的运动员跑过终点线，比赛才结束。

DistributedDoubleBarrier会监控连接状态，当连接断掉时`enter()`和`leave()`方法会抛出异常。

```java
public class DistributedDoubleBarrierDemo {

	private static final int QTY = 5;
	private static final String PATH = "/examples/barrier";

	public static void main(String[] args) throws Exception {
		try (TestingServer server = new TestingServer()) {
			CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			ExecutorService service = Executors.newFixedThreadPool(QTY);
			for (int i = 0; i < QTY; ++i) {
				final DistributedDoubleBarrier barrier = new DistributedDoubleBarrier(client, PATH, QTY);
				final int index = i;
				Callable<Void> task = () -> {

					Thread.sleep((long) (3 * Math.random()));
					System.out.println("Client #" + index + " enters");
					barrier.enter();
					System.out.println("Client #" + index + " begins");
					Thread.sleep((long) (3000 * Math.random()));
					barrier.leave();
					System.out.println("Client #" + index + " left");
					return null;
				};
				service.submit(task);
			}

			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);
			Thread.sleep(Integer.MAX_VALUE);
		}
	}

}
```

