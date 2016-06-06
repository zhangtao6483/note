# Redis

@(Redis)[redis]

[TOC]

# 1. 持久化

**Redis提供了不同级别的持久化策略**

1. RDB 可以在指定的时间间隔内生成快照（point-in-time snapshot）
2. AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据。AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。Redis 还可以在后台对 AOF 文件进行重写（rewrite）。
3. 可以关闭持久化功能，让数据只在服务器运行时存在。
4. Redis 还可以同时使用 AOF 持久化和 RDB 持久化。在这种情况下，当 Redis 重启时，它会优先使用AOF 文件来还原数据，因为 AOF 文件保存的数据通常比 RDB 文件所保存的数据更完整。

## 1.1 RDB
- RDB是整个内存的快照（Snapshot）
- 通过配置触发快照生成条件，其中默认配置：
```
save 900 1
save 300 10
save 60 10000
# save  <seconds> <changes>
# 经过多少秒且多少个key有改变就进行，可以配置多个，只要有一个满足就进行保存数据快照到磁盘
```
- [RDB文件格式](http://blog.nosqlfan.com/html/3734.html)
- 存储执行流程：

	1. fork()一个子进程
	2. 子进程将数据写入临时的rdb文件
	3. 当子进程完成文件写入，替换旧的rdb文件

- 相关配置：

	> **rdbcompression yes**
	> 保存数据到rdb文件时是否进行压缩，如果不想可以配置成’no’，默认是’yes’，因为压缩可以减少I/O，当然，压缩需要消耗一些cpu资源。
	> **dbfilename dump.rdb**
	> 快照文件名
	> **dir ./**
	> 快照文件所在的目录，同时也是AOF文件所在的目录

##1.2 AOF

- AOF（append-only file） 以协议文本的方式，将有效的写操作增量记录到 AOF文件
通过修改配置打开AOF功能
	> appendonly yes
	
- AOF重写因为AOF是将记录追加到文件末尾，所以随着命令的不断写入会导致AOF文件体积逐渐增大。为了处理这种情况，Redis将会rebuild AOF文件
	可以执行BGREWRITEAOF命令，Redis重建AOF文件，生成当前内存中命令的最短序列
	Redis的2.4是能够触发自动记录改写（更多信息请参见2.4示例配置文件）
- 触发AOF可以配置Redis多久将数据fsync到磁盘

	1. 每次有新命令追加到 AOF 文件时就执行一次 fsync ，慢，安全。
	2. 每秒 fsync 一次：快（和使用 RDB 持久化差不多，2.4），并且在故障时只会丢失 1 秒钟的数据。
	3. 从不 fsync ：将数据交给操作系统来处理。快，不安全。
推荐（默认）的措施为每秒 fsync 一次，这种 fsync 策略可以兼顾速度和安全性。

- 执行流程：

	1. fork()一个子进程
	2. 子进程将数据写入临时的aof文件
	3. 对于所有新执行的写入命令，父进程一边将它们累积到一个内存缓存中，一边将这些改动追加到现有AOF 文件的末尾：这样即使在重写的中途发生停机，现有的 AOF 文件也还是安全的
	4. 当子进程完成重写工作时，它给父进程发送一个信号，父进程在接收到信号之后，将内存缓存中的所有数据追加到新 AOF 文件的末尾。
	5. 现在 Redis 原子地用新文件替换旧文件，之后所有命令都会直接追加到新 AOF 文件的末尾。

- 相关配置：
	> **appendfilename appendonly.aof**
	>	指定AOF日志文件名称，默认名称：
	>	**appendonly.aof**
	>	**appendfsync everysec**
	> 什么时候将数据写入disk，redis提供三种模式:
	> no : 不进行fsync，有OS决定数据刷盘的时间粒度，性能高
	> always : 每次写操作都做fsync，安全
	> everysec : 上一个fsync后至少1s，折中
	> **no-appendfsync-on-rewrite no**
	> 当Aof log进行重写时，是否写日志时fsync。如果系统遇到latency问题，建议设为yes（rewrite时不强制fsync）
	> **auto-aof-rewrite-percentage 100**
	> 当Aof log增长超过指定比例时，重写log file， 设置为0表示不自动重写Aof log
	> **auto-aof-rewrite-min-size 64mb**
	> 启动重写Aof log时，Aof log的最小大小

#2. Scale

##2.1 主从复制（垂直Scale）

- Redis 使用异步复制。从 Redis 2.8 开始，从服务器将周期性的确认主服务器报告复制流（replication stream）的处理进度。
- 复制功能不会阻塞主服务器：即使有一个或多个从服务器正在进行初次同步，主服务器也可以继续处理命令请求。复制功能也不会阻塞从服务器。
- 可以通过复制功能来让主服务器免于执行持久化操作：关闭主服务器的持久化功能，由从服务器去执行持久化操作。
- 如果主服务器是 Redis 2.8 或以上版本，那么从服务器使用PSYNC 命令来进行同步，Redis 2.8 之前版本，使用SYNC。
- 配置从服务器 slaveof [主服务器IP] [端口号]
执行 slaveof no one 关闭复制功能，变为主服务器
- SYNC流程
	1. 从服务器向主服务器发送SYNC命令
	2. 收到SYNC命令的主服务器执行BGSAVE命令，在后台生成一个RDB文件，并使用一个缓冲区记录从现在开始执行的所有命令
	3. 当服务器的BGSAVE命令执行完毕时，主服务器会将BGSAVE命令生成RDB文件发送给从服务器，从服务器接受并载入这个RDB文件，将自己的数据库状态更新至主服务器执行BGSAVE命令时的数据库状态
	4. 主服务器将记录在缓冲区里面的所有写数据命令发送给服务器，从服务器执行这些写命令，将自己的数据库状态更新至从服务器数据库当前所处的状态
- PSYNC命令具有完整重同步（full resynchronization）和部分重同步（partial resynchronization）两种模式：
	- 其中完整重同步用于处理初次复制的情况：完整重同步的执行步骤和SYNC命令的执行步骤基本一样，它们都是通过让主服务器创建并发送RDB文件，以及向从服务器发送保存在缓冲区里面的写命令来进行同步。
	- 而部分重同步则用于处理断线后复制情况：当从服务器在断线后重新连接主服务器时，如果条件允许，主服务器可以将从服务器连接断开期间执行的写命令发送给从服务器，从服务器只要接收并执行这些命令，就可以将数据库更新至主服务器当前所处的状态

- **相关配置：**

	> **slaveof <masterip> <masterport>**
	> 表示该redis服务作为slave，masterip和masterport分别为master 的ip和port
	> **masterauth <master-password>**
	> 如果master设置了安全密码，则此处设置为相应的密码
	> **slave-serve-stale-data yes**
	> 当slave丢失master或者同步正在进行时，如果发生对slave的服务请求：
	> slave-serve-stale-data设置为yes则slave依然正常提供服务
	> slave-serve-stale-data设置为no则slave返回client错误："SYNC with master in progress"
	> **repl-ping-slave-period 10**
	> slave发送PINGS到master的时间间隔
	> **repl-timeout 60**
	> IO超时时间

##2.2 水平Scale

**方案一：使用jedis中的ShardedJedisPool**

扩容问题：
ShardedJedisPool采用了一致性Hash算法，Jedis的hash算法使用服务器的IP计算节点位置，然后再使不同的key分布到不同的Redis-Server上，当我们需要扩容时，需要增加机器到分片列表中，这时候会使得同样的key算出来落到跟原来不同的机器上，这样如果要取某一个值，会出现取不到的情况，对于这种情况，Redis的作者提出了一种名为Pre-Sharding的方式。Pre-Sharding方法是将每一个台物理机上，运行多个不同断口的Redis实例，假如有三个物理机，每个物理机运行三个Redis实际，那么我们的分片列表中实际有9个Redis实例，当我们需要扩容时，增加一台物理机，步骤如下：

1. 在新的物理机上运行Redis-Server
2. 该Redis-Server从属于(slaveof)分片列表中的某一Redis-Server（假设叫RedisA）
3. 等主从复制(Replication)完成后，将客户端分片列表中RedisA的IP和端口改为新物理机上Redis-Server的IP和端口
4. 停止RedisA。
Pre-Sharding实际上是一种在线扩容的办法，但还是很依赖Redis本身的复制功能的，如果主库快照数据文件过大，这个复制的过程也会很久，同时会给主库带来压力。所以做这个拆分的过程最好选择为业务访问低峰时段进行。

单点故障问题：
还是用到Redis主从复制的功能，两台物理主机上分别都运行有Redis-Server，其中一个Redis-Server是另一个的从库，采用双机热备技术，客户端通过虚拟IP访问主库的物理IP，当主库宕机时，切换到从库的物理IP。只是事后修复主库时，应该将之前的从库改为主库（使用命令slaveof no one），主库变为其从库（使命令slaveof IP PORT），这样才能保证修复期间新增数据的一致性。
当节点发生故障后，ShardedJedisPool会抛异常信息，如果节点有从服务器，并且可以自动Fail-Over，ShardedJedisPool不会得到从服务器信息。如果使用sentinel做Fail-Over，可以使用发布/订阅做到发现、切换功能

ShardedJedisPool调用initialize方法shared
```java
private void initialize(List<S> shards) {
        nodes = new TreeMap<Long, S>();

        for (int i = 0; i != shards.size(); ++i) {
            final S shardInfo = shards.get(i);
            if (shardInfo.getName() == null)
                for (int n = 0; n < 160 * shardInfo.getWeight(); n++) {
                    nodes.put(this.algo.hash("SHARD-" + i + "-NODE-" + n), shardInfo);
                }
            else
                for (int n = 0; n < 160 * shardInfo.getWeight(); n++) {
                    nodes.put(this.algo.hash(shardInfo.getName() + "*" + shardInfo.getWeight() + n), shardInfo);
                }
            resources.put(shardInfo, shardInfo.createResource());
        }
    }

```
JedisShardedPool通过使用每个节点的名字和权重映射到hash换上
```java
 public S getShardInfo(byte[] key) {
        SortedMap<Long, S> tail = nodes.tailMap(algo.hash(key));
        if (tail.isEmpty()) {
            return nodes.get(nodes.firstKey());
        }
        return tail.get(tail.firstKey());
 }
 ```
获取值时，首先根据传入的key按照hash算法取得其value，然后用这个value到map中找key大于前面生成的value值的第一个键值对，这个键值对的value既是对应的shardedInfo
最后根据得到的shardedInfo从resources中取得对应的Jedis对象

**方案二：通过Proxy 分片**

使用twemproxy
Twemproxy是memcached和redis协议的代理服务器，并能有效减少大量连接对redis服务器的性能影响，性能损耗小于20%
	>不足：
	虽然可以动态移除节点，但该移除节点的数据就丢失了。
	redis集群动态增加节点的时候,twemproxy不会对已有数据做重分布，需要重启服务器。
Twemproxy适合作为缓存集群
Codis

**方案三：官方的Redis Cluster**

分片使用哈希槽算法，集群将整个数据库分为16384（2 的14 次方）个槽（slot）。集群中的每个节点负责处理一部分哈希槽。举个例子，一个集群可以有三个哈希槽，其中：

- 节点 A 负责处理 0 号至 5500 号哈希槽。
- 节点 B 负责处理 5501 号至 11000 号哈希槽。
- 节点 C 负责处理 11001 号至 16384 号哈希槽。
这种将哈希槽分布到不同节点的做法使得用户可以很容易地向集群中添加或者删除节点。比如说：

- 如果用户将新节点 D 添加到集群中，那么集群只需要将节点 A 、B 、C 中的某些槽移动到节点 D 就可以了。
- 与此类似，如果用户要从集群中移除节点 A ，那么集群只需要将节点 A 中的所有哈希槽移动到节点B 和节点 C ，然后再移除空白（不包含任何哈希槽）的节点 A 就可以了。
因为将一个哈希槽从一个节点移动到另一个节点不会造成节点阻塞，所以无论是添加新节点还是移除已存在节点，又或者改变某个节点包含的哈希槽数量，都不会造成集群下线。

执行命令的两种情况

1. 命令发送到了正确的节点：命令要处理的键所在的槽正好是由接收命令的节点负责，那么该节点执行命令，就像单机Redis 服务器一样。
2. 命令发送到了错误的节点：接收到命令的节点并非处理键所在槽的节点，那么节点将向客户端返回一个转向（redirection）错误，告知客户端应该到哪个节点去执行这个命令，客户端会根据错误提示的信息，重新向正确的节点发送命令。
Redis 集群不保证数据的强一致性（strong consistency）：在特定条件下，Redis 集群可能会丢失已经被执行过的写命令。

关于Redis集群详细内容：http://redis.readthedocs.org/en/latest/topic/cluster-tutorial.html

# 3. HA

**方案一：**

keepalived：通过keepalived的虚拟IP，提供主从的统一访问，在主出现问题时，通过keepalived运行脚本将从提升为主，待主恢复后先同步后自动变为主，该方案的好处是主从切换后，应用程序不需要知道(因为访问的虚拟IP不变)，坏处是引入keepalived增加部署复杂性；

**方案二：**

zookeeper：通过zookeeper来监控主从实例，维护最新有效的IP，应用通过zookeeper取得IP，对Redis进行访问；

**方案三：**

sentinel：通过Sentinel监控主从实例，自动进行故障恢复，该方案有个缺陷：因为主从实例地址是不同的，当故障发生进行主从切换后，应用程序无法知道新地址，故在Jedis2.2.2中新增了对Sentinel的支持，应用通过redis.clients.jedis.JedisSentinelPool.getResource()取得的Jedis实例会及时更新到新的主实例地址。

# 4. 方案设计

主流集群方案：

**Redis小型高可用架构**

**方案：Redis主从复制+Keepalived实现Failover**

![enter image description here](http://img226.poco.cn/mypoco/myphoto/20140408/11/54670915201404081145134027465749356_009.jpg)

服务器资源：两台PC Server
优点：架构简单，节省资源
缺点：主从切换有间隔，这期间客户端将收到错误
**方案：Redis Sentinel实现Failover**

![enter image description here](http://img226.poco.cn/mypoco/myphoto/20140408/11/54670915201404081145134027465749356_008.jpg)

服务器资源：

两台PC Server部署Redis，一台Redis Sentinel；
Redis可选择一主多从架构；
一台Redis Sentinel选择低配。
优点：Redis官方自带HA方案，Redis作者所编写，具备
缺点：发生Failover之后，客户端需要手动更正地址
Redis中型高可用架构

**方案：Redis主从+Haproxy负载均衡**

![enter image description here](http://img226.poco.cn/mypoco/myphoto/20140408/11/54670915201404081145134027465749356_007.jpg)

服务器资源：至少3台PC Server部署Redis主从，两台PC Server部署Haproxy
优点：读写分离，横向扩展Slave
缺点：Master为单点
Redis大型高可用架构

**方案：Twemproxy实现Redis存储分片**

![enter image description here](http://img226.poco.cn/mypoco/myphoto/20140408/11/54670915201404081145134027465749356_006.jpg)

服务器资源：至少6台PC Server部署Redis主从，至少3台PC Server部署Twemproxy，2台PC Server部署HAProxy
优点：分片，负载均衡，Redis和Twemproxy都可以横向扩展
缺点：Twemproxy所存在的缺点：

Twemproxy节点扩展，原来的数据需要重新处理分布，避免出现找不到key值；
扩展Redis节点，数据不会自动均匀分布，而需人工处理。

# 5. 参考

《Redis设计与实现》
http://redis.readthedocs.org/en/latest/
http://www.luocs.com/archives/849.html
http://blog.nosqlfan.com/topics/redis
https://github.com/springside/springside4/wiki/redis
http://redis.io/topics/partitioning
http://blog.csdn.net/mydreamongo/article/details/8951905
http://redis.readthedocs.org/en/latest/topic/cluster-tutorial.html


