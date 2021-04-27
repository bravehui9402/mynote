## Redis单机安装

> 安装C语言的编译环境

```shell
yum install centos-release-scl scl-utils-build
yum install -y devtoolset-8-toolchain
scl enable devtoolset-8 bash
--测试 gcc版本
gcc --version

```

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210426205731.png)

```shell
--上传redis安装包到optmulu
--解压
tar -zxvf redis-6.2.1.tar.gz
--解压完成后进入解压后的目录，执行make命令,这里make只是编译
cd redis-6.2.1
make
make install 
--安装目录：/usr/local/bin
```

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210426210103.png)

- redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何

- redis-check-aof：修复有问题的AOF文件，rdb和aof后面讲

- redis-check-dump：修复有问题的dump.rdb文件

- redis-sentinel：Redis集群使用

- redis-server：Redis服务器启动命令

- redis-cli：客户端，操作入口

> 启动

后台启动：

```shell
--copy redis.conf 到其他目录
mkdir redis6379
mv redis.conf /usr/local/soft/redis6379/

cd /usr/local/bin
redis-server /usr/local/soft/redis6379/redis.conf
ps -ef | grep redis
```

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210427084124.png)

```shell
--客户端放完
redis-cli
--多端口访问
redis-cli -p 6379

```

## Redis集群

