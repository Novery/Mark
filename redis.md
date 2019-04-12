# Redis
+ 存在内存中（数据持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用,），速度快（传统数据库，硬盘）
+ Redis只是缓存，虽然做了持久化 但是还是背后要有db的支撑，开了aof也做不到一个数据都不丢失
### 高性能、高并发
+ 高性能
  + 数据存在缓存上，操作缓存就是直接操作内存，如果数据中数据改变了，同步改变缓存中的数据
+ 高并发
  + 缓存上承受的请求大于直接访问数据库
### 和map/guava的区别
  + java自带的map或guava实现的是本地缓存，生命周期随jvm销毁而结束，在多实例的情况下，每个实例都需要各自保存缓存，缓存不具有一致性
  + Redis，分布式缓存，多实例共用，缺点是需要保持Redis高可用，架构上较为复杂
###  Redis设置过期时间
  + 对过期数据批量删除，比如token或者登陆信息
  + 定时删除，随机抽取，过期则删除（随机而不是遍历，性能考虑）
  + 惰性删除，定时删除漏掉的，只有查询到这个key，才会删除
### Redis事务
  + 通过multi、exec、watch等命令实现事务，事务提供了一种机制，将多个请求打包，一次性按顺序执行所有命令，期间不会处理其他客户端请求命令

### Redis持久化
+ rdb
  + 快照是默认的持久化方式。这种方式是就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为dump.rdb。可以通过配置设置自动做快照持久化的方式。我们可以配置redis在n秒内如果超过m个key被修改就自动做快照
+ aof
  + redis会将每一个收到的写命令都通过write函数追加到文件中(默认是appendonly.aof)。当redis重启时会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容
  + os会在内核中缓存write做的修改，所以可能不是立即写到磁盘上。这样aof方式的持久化也还是有可能会丢失部分修改
  + 可以通过配置文件告诉redis我们想要通过fsync函数强制os写入到磁盘的时机
### 缓存穿透和缓存雪崩
  + 雪崩，缓存同一时间大面积失效，后面请求都落到数据库，造成数据库短时间承受大量请求而崩溃
  + 解决方案：
    + 事前：尽量保证整个Redis集群的高可用性，发现机器宕机尽快补上。选择合适的内存淘汰策略。
    + 事中：本地ehcache缓存 + hystrix限流&降级，避免MySQL崩掉
    + 事后：利用 Redis 持久化机制保存的数据尽快恢复缓存
  + 缓存穿透,黑客故意去请求缓存中不存在的数据，导致所有请求落到数据库，造成数据库短期承受大量请求而崩塌
  + 解决方案，采用布隆过滤器，将所有可能存在的数据hash到一个足够大的bitmap上，一个一定不存在的数据会被这个bitmap拦截掉
### 淘汰策略（6种）
  + volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
  + volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
  + volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
  + allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key（这个是最常用的）
### 分布式锁
  + 并发竞争key，多个系统同时对key操作，执行顺序与我们期望不同
  + Redis，zookeeper都可以实现分布式锁，会影响性能
### 保证缓存数据库一致性
  + 读请求和写请求串行化，串到一个内存队列里去，这样就可以保证一定不会出现不一致的情况
  + 串行化之后，就会导致系统的吞吐量会大幅度的降低，用比正常情况下多几倍的机器去支撑线上的一个请求。

# Redis的使用
### 环境准备
+ [Redis官网](https://Redis.io)
+ 官网提供的是linux版，github有提供window版
+ [安装Ubuntu，提供windows的Linux子系统](https://baijiahao.baidu.com/s?id=1608773578329225793&wfr=spider&for=pc)

### 安装
+ 安装
  + 安装分为apt-get和一般的安装方式
+ apt-get
```
apt-get update

apt-get install Redis-server
```
+ 一般方式
  + Redis是c开发的，需要c语言编译环境
```
gcc -v
```
+ 如何没有gcc,需要在线安装
+ wget默认安装路径在执行wget命令的当前目录
```
yum install gcc-c++
$ wget http://download.redis.io/releases/redis-5.0.4.tar.gz
```
+ 解压
```
$ tar xzf redis-5.0.4.tar.gz
```
+ cd到对应目录
```
$ cd redis-5.0.4
```
+ 编译
+ linux里大部分软件都是开源的，都是让你把c代码拉下来，自己make）
+ make完后 redis-5.0.4目录下会出现编译后的redis服务程序redis-server,还有用于测试的客户端程序redis-cli,两个程序位于安装目录 src 目录下
+ 编译通过会出现It's a good idea to run 'make test' ;)
```
$ make
```
+ 安装并指定安装目录
```
make install PREFIX=/usr/local/redis
```

### 使用
+ 启动redis服务.（makefile方式需要手工启动，apt-get不需要）
+ cd到usr/local/redis/bin
```
$ ./redis-server
```

+ 查看是否启动
```
ps -aux|grep Redis
```
+ 查看版本
```
Redis-server -v
```
+ 通过命令行客户端访问Redis（apt）
```
Redis-cli
```
+ 通过命令行客户端访问Redis（make）
+ cd到usr/local/redis/bin
```
./redis-cli
```
+ 修改配置文件
```
vi 文件名
sudo nano /etc/Redis/Redis.conf
sudo命令以系统管理者的身份执行指
```
+ 重启重启Redis服务器。
```
sudo /etc/init.d/Redis-server restart
```
+ 增加一条字符串记录
```
set key1 "hello"
```
+ 查看字符串记录
```
get key1
```
### 集群搭建

****Redis Cluster（Redis集群）简介****

+ redis3.0版本之前只支持单例模式，在3.0版本及以后才支持集群
+ redis集群采用P2P模式，是完全去中心化的，不存在中心节点或者代理节点
+ reids集群无统一入口，客户端连接任意节点即可，节点是相互通信的（PING-PONG机制），每个节点是一个redis实例
+ 健康检查，当超过半数节点认为某节点挂了，则该节点被判断为挂了
+ 当任意一个节点挂了，而且该节点没有从节点（备份节点），那么这个集群就挂了

****集群搭建需要的环境****
+ 需要三个节点，因为容错机制需要半数以上的节点通过判断节点是否健康
+ 要保证高可用，每个节点需要从节点(备份)
+ 安装ruby

****步骤****
+ usr/local下新建文件夹，用于存放集群节点
```
sudo mkdir redis-cluster
```
+ 把wget下载下的源码目录下的redis.conf复制到/usr/local/redis-cluster/redis01目录下
```
cp -r redis.conf ../../../usr/local/redis-cluster/redis01
```
+ 删除redis01目录下的快照文件dump.rdb，并且修改该目录下的redis.cnf文件，具体修改两处地方：一是端口号修改为7001，二是开启集群创建模式
```
rm -rf dump.rdb
vi命令分命令行模式和输入模式，按i进入输入模式，按ESC键退回命令行模式
命令行模式  ‘/’可查询内容
```
+ redis.cnf修改port，将cluster-enabled yes的注释打开，将cluster-config-file
```
:wq 保存退出
:q! 退出不保存
```
+ 制作启动shell脚本
+ 赋权限 sudo chmod +x redis-start.sh
+ touch 文件名/新建文件
```
sudo ./redis/bin/redis-server redis-cluster/redis01/redis.conf &
sudo ./redis/bin/redis-server redis-cluster/redis02/redis.conf &
sudo ./redis/bin/redis-server redis-cluster/redis03/redis.conf &
sudo ./redis/bin/redis-server redis-cluster/redis04/redis.conf &
sudo ./redis/bin/redis-server redis-cluster/redis05/redis.conf &
sudo ./redis/bin/redis-server redis-cluster/redis06/redis.conf
```
+ 查看服务启动
```
ps aux|grep redis
```
+ 安装ruby环境
```
sudo apt-get install ruby
```
+ 将ruby脚本工具复制到usr/local/redis-cluster目录下
+ cd到redis源码src目录下
```
cp -r redis-trib.rb ../../../../usr/local/redis-cluster
```
+ 执行ruby脚本
```
./redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006
```
+ redis5.0不需要使用ruby了，功能移到redis-cli
+ 最后一段文字，记录了每个节点分配的slots（哈希槽）
```
./redis/bin/redis-cli --cluster create --cluster-replicas 1  127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 
```
+ 


+ 关闭服务
```
sudo  pkill -9 redis
```

****代办问题****
+ reids同个接口可以实例化多次
+ sudo  pkill -9 redis含义

### 卸载
+ apt-get方式安装的
```
sudo apt-get purge --auto-remove redis-server
```
+ make方式安装的
```
关闭已经启动的 Redis 服务，注意，你可能启动了多个实例，所以可能要逐个关闭，我这里的情况只有 redis_6379 在运行：

sudo service redis_6379 stop

删除 usr/local/bin/ 中所有 redis 相关的文件

sudo rm /usr/local/bin/redis-*

删除配置目录和内容

sudo rm -r /etc/redis/

删除日志

sudo rm /var/log/redis_*

删除数据目录和内容

sudo rm -r /var/lib/redis/

删除初始化脚本

sudo rm /etc/init.d/redis_*

删除现有的Redis PID文件(仅当存在时)

sudo rm /var/run/redis_*

重启服务器
```
### 客户端安装(Redisdesktopmanager)
+ 除了Redis自带的命令行客户端还可以使用其他客户端
+ [Redisdesktopmanager官网](https://Redisdesktop.com/download)
+ [Redisdesktopmanager快速入门](http://docs.Redisdesktop.com/en/latest/quick-start/)
+ 建立连接
+ 使用过滤器
+ 使用控制台

### java中的调用
+ 引入jedis.jar驱动
```xml
<!--版本号可根据实际情况填写-->
<dependency>
  <groupId>Redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.7.1</version>
</dependency>
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.3.2</version>
</dependency>
```
+ application启动类加入连接代码
  + 控制台输出：连接成功服务正在运行: PONG
```java
//连接本地的 Redis 服务
Jedis jedis = new Jedis("localhost");
System.out.println("连接成功");
//查看服务是否运行
System.out.println("服务正在运行: "+jedis.ping());
```
+ Redis Java String(字符串) 实例
```java
//设置 Redis 字符串数据
jedis.set("chinakey", "www.china.com");
// 获取存储的数据并输出
System.out.println("Redis 存储的字符串为: "+ jedis.get("chinakey"));
```
+ Redis Java List(列表) 实例
```java
jedis.lpush("site-list", "China");
jedis.lpush("site-list", "Google");
jedis.lpush("site-list", "Taobao");
// 获取存储的数据并输出
List<String> list = jedis.lrange("site-list", 0 ,2);
System.out.println(list);
```

# Redis开发规范
+ [Redis开发规范](https://mp.weixin.qq.com/s/lkQsOsqY3m4V_SCGksI_BQ)

# 目前项目中的情况
+ 虽然生产上配置了双机，但主要作用是备灾，保证服务的可用，在缓存的使用上没选择Redis、memcache等分布式架构下的缓存系统，而是采用了公司封装的技术框架，solar中的Cache包，其底层实现为map,其本质为一个传统的本地内存，在多实例的情况下会维护多个内存，应用场景主要是读多写少的静态数据上，当服务启动时，一次性的将db的数据写入a机、b机的缓存中，此时两边的缓存数据是一致的，当用户发起写入请求时，sds公共组件会发起一次广播，通过xnet消息中间件多播的方式发布消息，a机、b机通过订阅消息，同步的更新自己的缓存，保证两个服务内缓存的一致性，在写入请求较少的情况下，此方案有其实用性

