luttuce和redis

# NoSQL数据库简介

## 技术发展

1.解决功能性问题：Java，jsp，html，linux

2.解决扩展性问题：Spring，mybatis

3.解决性能问题：nosql，Java线程，nginx，mq，es

分布式session共享问题

1.存在cookie每次带上，但是不安全，网络效率

2.session复制，每台服务器都复制session，但是空间浪费

3.把session存储到nosql中，不用io，数据存在内存中

解决io问题

缓存数据库，减少io的读操作

## nosql数据库

### 概述

nosql：not only sql，泛指非关系型数据库，不依赖业务逻辑方式存储，而以简单的key-value模式存储，增加了数据库扩展能力。

## 适用场景

高并发

海量数据

## 不适用场景

需要事务

基于sql的结构话查询处处，处理复杂的关系

## nosql数据库种类

memcache（用的比较少了）、redis、mongodb

图关系数据库

neo4j

## redis常见知识

默认端口6379，alessia merz演员名字，merz在九键输入法就是6379

有16个数据库，从0开始，默认是用的0

串行 vs 多线程+锁（memcache） vs 单线程 + 多路io复用（redis）

什么事单线程+多路io复用

相当于所有人去火车站向黄牛买票，黄牛去站台买票，黄牛去买票时单线程，但是面多顾客时多路io

# 常用五大数据类型

## redis键

```shell
keys * # 查看当前数据库所有key
exists key # 判断某个key是否存在
type key # 查看你的key是什么类型
del key # 删除指定的key数据
unlink key # 根据value选择非阻塞删除，仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作
expire key 10 # 10秒钟：为给定的key设置过期时间
ttl key # 查看还有多少秒过期，-1表示用不过期，-2表示已过期

select db # 切换数据库
dbsize # 查看当前数据库的key的数量
flushdb # 清空当前库
flushall # 清空所有库
```

## Redis字符串

###  简介

String是Redis最基本的类型，你可以理解成Memcached一模一样类型，一个key对应一个value

String类型是二进制安全的，意味着Redis的String可以包含任何数据，比如jpg图片或者序列化的对象。

String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M。

### 常用命令

```shell
set key value # 设置一个值，设置相同的key会覆盖
get key # 获取值
append key value # 在key后面追加值
strlen key # 获取值的长度
setnx key value # 只有key不存在时，才设置key的值，不会出现覆盖的情况


incr key # 将key的数字值加1，如果为空新值就是1
decr key # 把key的数字值减1，如果为空新值就是-1
incrby key 步长 # 每次增加步长
decrby key 步长 # 每次减少步长
```

incr key的原子性，在单线程中，能够在单条指令中完成的操作都可以认为是原子操作，因为中断只能发生于指令之间。在多线程中，不能被其他进程打断的操作就叫原子操作。

redis单命令的原子性，主要是由于redis是单线程。

```shell
mset key1 value1 key2 value2 ... # 同时设置多个键值对
mget key1 key2 ... # 同时获取多个键
msetnx key1 value1 key2 value2 ... # 同时设置一个或多个键值对，当且仅当所给定的key不存在，具备原子性当一个key失败都失败
```

```shell
getrange key 起始位置 结束位置 # 获取值的范围
setrange key 起始位置 value # 用value覆盖从起始位置的值
setex key 过期时间 value # 设置值的同时就可以设置过期时间
getset key value # 用新值换旧值，并获取旧值
```

### 数据结构

String的数据结构为简单动态字符串（simple dynamic string），是可以修改的字符串，内部结构实现上类似于java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配。

内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len，当字符串长度小于1M，扩容都是加倍现有空间，如果超过1M，空阔一次就会加1M，当时最大长度不会超过512。

## Redis列表（List）

### 简介

单键多值，Redis列表是简单的字符串列表，按照插入顺序排序，你可以添加一个元素到列表的头部（左边）或者尾部（右边）。它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会比较差。

### 常用命令

```shell
lpush/rpush key value1...valuen # 从左边/右边插入一个或多个值，先进后出，链表头插法
lpop/rpop key #从左边/右边取出一个值，值在键在
rpoplpush key1 key2 #从列表key1右侧取一个值，插入key2左边
lrange key start stop # 按照索引下标获得元素（从左到右）0到-1表示所有的
```

```shell
lindex key index # 按照索引下标获得元素（从左到右）
llen key # 获得列表长度
linsert key before value newvalue #在value的后面插入newvalue新值，也可以after
lrem key n value # 从左边删除n个value
lset key index value #将列表key下标为index的值替换成value
```

### 数据结构

List的数据结构为快速练标quickList。首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构式ziplist，也即是压缩列表。

它将所有的元素紧挨着一起存储，分配的是一块连续的内存。

当数据量很多的时候才会改成quickList。

因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。

Redis将链表和ziplist结合起来组成了quickList。也就是将多个zipList使用双向指针串起来使用，这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

## Redis集合（Set）

### 简介

Redis Set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以自动排重的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

Redis的set时String类型的无序集合，它底层其实是一个value为null的hash表，所以添加，删除，查找的复查度都是O（1）。

一个算法，随着数据的增加，执行时间的长短，如果是O（1），数据增加，查找数据的时间不变。

### 常用命令

```shell
sadd key value1 ... valuen # 将一个活多个元素加入到集合key中，已经存在的value元素将被忽略
smember key # 取出该集合的所有值
sismember key value # 判断集合key是否含有该value值，有1，没有0
scard key # 返回集合的元素个数
srem key value1 value2 # 删除集合中的某个元素
spop key # 随机从该集合中取出一个值
sranmember key n # 随机从该集合中取出n个值，不会从集合中删除
smove source destination value # 把集合中一个值从一个集合移动到另一个集合
sinter key1 key2 # 返回两个集合的交集元素
sunion key1 key2 # 返回两个集合的并集元素
sdiff key1 key2 # 返回两个集合的差集元素（key1的，不包含key2的）
```

### 数据结构

Set数据结构式dict字典，字典是用哈希表实现的。

Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也适用hash结构，所有的value都是香同一个内部值。

## Redis哈希（Hash）

### 简介

Redis hash是一个键值对集合。

Redis hash是一个string类型的field类型和value的映射表，hash特别适合用于存储对象。

类似于Java里面的Map<String, Object>

用户ID为查找的key，存储的value用户对象包含姓名、年龄、生日等信息，如果用普通的key/value结构来存储

主要有以下两种存储方式：

用户ID -> 序列化的value对象：每次修改用户的某个属性，先反序列化改好后再序列化回去，开销较大

用户id+用户属性1 ->值  用户id+用户属性2 -> 值：用户ID数据冗余

### 常用命令

```shell
hset key field value # 给key集合中的field赋值value
hget key field # 从key集合field取出value
hmset key field1 value1 field2 value2 # 批量设置hash的值
hexists key field # 查看哈希表key中，给定域field是否存在
hkeys key # 列出该hash集合的所有field
hvals key # 列出该hash集合的所有value
hincrby key field increment # 为哈希表key中的域field的值加上增量increment
hsetnx key field value # 将哈希表key中的域field的值设置为value，当且仅当域field不存在
```

### 数据结构

hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable

## Redis有序集合（Zset）

### 简介

Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符集合。不同之处是有序集合的每个成员都关联了一个评分（score），这个评分（score）被用来按照从低分到高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复了。

因为元素是有序的，所以你也可以很快的根据评分或者次序来获取一个范围的元素。

访问有序集合的中间元素也是非常快的，因此你能够使用有序集合作为一个没有重复成员的智能列表。

### 常用命令

```shell
zdd key score1 value1 score2 value2 # 将一个活多个memeber元素及其score值加入到有序集合key当中
zrange key start stop [withscores] # 返回有序集合中，下标在start stop之间的元素，带withscores，可以让分数一起和值返回到结果集
zrangebyscore key minmax [withscores] [limit offset count] # 返回有序集key中，所有score值介于min和max之间（包括等于min和max）的成员，有序集成员按score值递增（从小到大）次序排列
zrevrangebyscore key maxmin [withscores] [limit offset count] # 同上，改为从大到小排列
zincrby key increment value # 为元素的score加上增量
zrem key value # 删除该集合下，指定值的元素
zcount key min max # 统计该集合，分数区间内的元素个数
zrank key value # 返回该值在集合中的排名，从0开始
```

### 数据结构

SortedSet（zset）是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构

hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

**跳跃表**，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。



# 事务

数据库：ACID

原子性：要么同时成功，要么同时失败

Redis事务本质：一组命令的集合。一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行。

一次性，顺序性，排他性，执行一些列的命令

```shell
----- 队列 set set set -----
```

redis事务没有隔离级别的概念，所有的命令在事务中，并没有直接被执行，只有发起执行命令的时候才会执行。

Redis单条命令时保证原子性的，但是事务不保证原子性

Redis的事务：

- 开启事务（multi）
- 命令入队（...）
- 执行事务（exec）

> 正常执行事务

> 放弃事务，所有的队列都不会执行
>
> multi
>
> set k1 v1
>
> set k2 v2
>
> Discard

> 编译型异常（代码有问题），事务中所有的命令都不会被执行，错误的命令

> 运行时异常（1/0），如果队列中存在语法错误，那么执行命令的时候，其他命令是可以正常执行的

锁：redis可以实现乐观锁

悲观锁：

- 很悲观，认为什么时候都会出问题，无论做什么都会加锁

乐观锁：

- 很乐观，认为什么时候都不会出问题，所以不会上锁，更新数据的时候去判断一下，在此期间是否有人修改过这个数据。
- 获取version
- 更新的时候比较version

> redis监视测试

正常执行成功

```shell
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> decrby money 20
QUEUED
127.0.0.1:6379(TX)> incrby out 20
QUEUED
127.0.0.1:6379(TX)> exec
1) (integer) 80
2) (integer) 20
```

错误执行

线程A

```shell
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money 
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> decrby money 20
QUEUED
127.0.0.1:6379(TX)> incrby out 20
QUEUED
```

线程B

```shell
127.0.0.1:6379> set money 100
OK
```

线程A

```shell
127.0.0.1:6379(TX)> exec
(nil) # 事务执行异常
```

测试多线程修改值，使用watch可以当作redis的乐观锁操作。

如果事务执行失败就执行unwatch命令。需要再研究？？？

# Jedis

使用Java操作redis

> 什么是jedis，是redis官方推荐的Java链接开发工具，使用Java操作redis中间件，如果你要使用Java操作redis，那么要对jedis十分熟悉

导入对应的依赖

```groovy
 <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.2.0</version>
        </dependency>
```

测试链接

```java
 Jedis jedis = new Jedis("127.0.0.1", 6379);
 System.out.println(jedis.ping());
```

关闭redis连接

```java
jedis.close()
```

> jedis操作事务

```java
 Jedis jedis = new Jedis("127.0.0.1", 6379);
        Transaction transaction = jedis.multi();

       try{
           transaction.set("user1", "zhangsan");
           transaction.exec();
       } catch (Exception e) {
           transaction.discard();
       } finally {
           System.out.println(jedis.get("user1"));
           jedis.close();
       }
```

# Springboot整合

## 入门

springboot操作数据：spring-data  jpg jdbc  mongodb  redis

springData也是和springboot齐名的项目

说明：在springboot2.x之后，原来使用的jedis被替换为lettuce

jedis：采用的直连，多个线程操作的话，是不安全的，如果想要避免不安全，使用jedis pool连接池，bio

lettuce：采用netty，实例可以在多个线程中进行共享，不存在线程不安全的情况，可以减少线程数据了，更像nio模式

源码分析: RedisAutoConfiguration

```java
@Bean
//我们可以自己定义RedisTemplate来替换这个默认的
@ConditionalOnMissingBean(name = "redisTemplate")
public RedisTemplate<Object, Object> redisTemplate(
    RedisConnectionFactory redisConnectionFactory)
    // RedisConnectionFactory 有两个实现类jedis和lettuce
    throws UnknownHostException {
    // 默认的redisTemplate没有过多的设置，redis的对象是需要序列化的
    // 两个泛型都是Object类型，我们后面使用都需要强制转换
    RedisTemplate<Object, Object> template = new RedisTemplate<Object, Object>();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}

@Bean
// 由于String是redis最常用的类型，所以有一个单独的bean
@ConditionalOnMissingBean(StringRedisTemplate.class)
public StringRedisTemplate stringRedisTemplate(
    RedisConnectionFactory redisConnectionFactory)
    throws UnknownHostException {
    StringRedisTemplate template = new StringRedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
```



```properties
# springboot 所有配置类，都有一个自动配置类 RedisAutoConfiguration
# 自动配置类都会绑定一个properties配置文件 RedisProperties
```

> 整合测试

1.导入依赖

```xml
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-redis</artifactId>
     <version>1.4.1.RELEASE</version>
</dependency>
```

2.配置连接

```properties
spring.redis.host="127.0.0.1"
spring.redis.port="6379"
```

3.测试



## 自定义RedisTemplate

```java
RedisSerializer 有七个实现类
ByteArrayRedisSerializer
GenericJackson2JsonRedisSerializer
GenericToStringSerializer
Jackson2JsonRedisSerializer
JdkSerializationRedisSerializer
OxmSerializer
StringRedisSerializer
```

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);

        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        template.setKeySerializer(stringRedisSerializer);
        template.setValueSerializer(jackson2JsonRedisSerializer);

        template.afterPropertiesSet();
        return template;
    }
}
```

# Redis.conf详解

## Units

配置大小单位,开头定义了一些基本的度量单位，只支持bytes，不支持bit。大小写不敏感

```shell
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
```

## INCLUDES

引入其他的内容，当有多个服务需要公用的配置时，可以使用include。

```shell
# include /path/to/local.conf
# include /path/to/other.conf
```

## NETWORK

### bind

默认情况bind=127.0.0.1只能接受本机的访问请求

不写的情况下，无限制接受任何ip地址的访问

生产环境肯定要写你应用服务器的地址；服务器是需要远程访问的，所以需要将其注释掉

如果开启了protected-mode，那么在没有设定bind ip且没有设密码的情况下，Redis只允许接受本机的响应

### protected-mode

将本机访问保护模式设置

### port

端口号，默认6379

### tcp-backlog

设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列+ 已经完成三次握手队列。

在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。

注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值（128），所以需要确认增大/proc/sys/net/core/somaxconn和/proc/sys/net/ipv4/tcp_max_syn_backlog（128）两个值来达到想要的效果

### timeout

一个空闲的客户端维持多少秒会关闭，0表示关闭该功能。即永不关闭。

### tcp-keepalive

对访问客户端的一种心跳检测，每个n秒检测一次。

单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置成60 

## GENERAL

### daemonize

是否为后台进程，设置为yes

守护进程，后台启动

### pidfile

存放pid文件的位置，每个实例会产生一个不同的pid文件

### loglevel

指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为**notice**

四个级别根据使用阶段来选择，生产环境选择notice 或者warning

### logfile

日志文件名称

### databases

设定库的数量默认16，默认数据库为0，可以使用SELECT dbid 命令在连接上指定数据库id

## SECURITY

访问密码的查看、设置和取消

在命令中设置密码，只是临时的。重启redis服务器，密码就还原了。

永久设置，需要再配置文件中进行设置。

```shell
# requirepass foobared
```

## CLIENTS

```shell
# maxclients 10000
```

 设置redis同时可以与多少个客户端进行连接。

 默认情况下为10000个客户端。

如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应。

## MEMORY MANAGEMENT

### maxmemory

```shell
# maxmemory <bytes>
```

建议**必须设置**，否则，将内存占满，造成服务器宕机

设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。

如果redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。

但是对于无内存申请的指令，仍然会正常响应，比如GET等。如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素。

### maxmemory-policy

volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）

allkeys-lru：在所有集合key中，使用LRU算法移除key

volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键

allkeys-random：在所有集合key中，移除随机的key

volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key

noeviction：不进行移除。针对写操作，只是返回错误信息

### maxmemory-samples

设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个。

一般设置3到7的数字，数值越小样本越不准确，但性能消耗越小。

# Redis持久化

## RDB

除了自动快照，还可以手动发送SAVE或BGSAVE命令让Redis执行快照，两个命令的区别在于，前者是由主进程进行快照操作，会阻塞住其他请求，后者会通过fork子进程进行快照操作

> 触发机制

1.save的规则满足的情况下，会自动触发rdb规则

2.执行flushall命令，也会出发我们的rbd规则

3.退出redis，也会产生rdb文件

> 恢复rbd

rbd还原直接将rbd.dump放到redis安装目录（也可以执行命令：config  get  dir），重启即可，

aof还原也是一样的

优点

1.适合大规模的数据恢复

2.如果对数据的完整性要求不高，save的规则导致有一部分无法备份

缺点

1.需要一定的间隔时间间隔进程操作

2.fork进程的时候，需要一定的内存空间

## AOF

将我们所有命令都记录下来，回复的时候就把这文件全都执行一遍。

# Redis发布/订阅

## 什么是Redis发布/订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。发布者生产消息放到队列里，多个监听队列的消费者都会收到同一份消息。
Redis客户端可以订阅任意数量的频道。

## 消息发布/订阅图

![](./pic/message_push_read.png)

## 消息发布/订阅命令

```shell
subscribe <channel> <channel> # 订阅一个或多个频道
publish <channel> message # 发布消息
UNSUBSCRIBE channel [channel ...] # 退订给定的一个或多个频道的信息
PSUBSCRIBE pattern [pattern ...] # 订阅一个或多个符合给定模式的频道
PUNSUBSCRIBE [pattern [pattern ...]] # 退订所有给定模式的频道
```

> redis-cli 1 订阅

```shell
127.0.0.1:6379> subscribe cha1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "cha1"
3) (integer) 1
1) "message"
2) "cha1"
3) "hello"

```

> redis-cli 2 发布

```shell
127.0.0.1:6379> publish cha1 hello
(integer) 2
127.0.0.1:6379> 
```

## 发布/订阅原理

Redis是使用C实现的，通过分析 Redis 源码里的pubsub.c文件，了解发布和订阅机制的底层实现，籍此加深对 Redis 的理解。
Redis 通过 PUBLISH 、SUBSCRIBE 和 PSUBSCRIBE 等命令实现发布和订阅功能。

通过SUBSCRIBE命令订阅某频道后，redis-server 里维护了一个字典，字典的键就是一个个 频道。， 而字典的值则是一个链表，链表中保存了所有订阅这个 channel 的客户端。SUBSCRIBE 命令的关键， 就是将客户端添加到给定 channel 的订阅链表中。

通过PUBLISH命令向订阅者发送消息，redis-server会使用给定的频道作为键，在它所维护的 channel 字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。

Pub/Sub 从字面上理解就是发布（Publish）与订阅（Subscribe），在Redis中，你可以设定对某一个 key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会收到相应 的消息。这一功能明显的用法就是用作实时消息系统，比如普通的即时聊天，群聊等功能。

## 使用场景

1、实时聊天系统
2、订阅、关注系统
其实更多时候都是使用MQ（消息中间件）来做的。

## springboot用redis实现消息订阅发布

# Redis集群

## Redis集群三种模式

- Redis主从
- Redis高可用哨兵
- Cluster模式

## Redis主从

### Redis主从配置

master正常配置

slave

```shell
slaveof <masterip> <masterport>    # 指定master的ip和port
masterauth <master-password>       # master有验证的情况下
slave-read-only yes                # 设置slave为只读模式
```



https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html
