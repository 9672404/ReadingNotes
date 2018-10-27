# Redis 入门

##  1. NoSQL入门概述

### 1.1 概述

NoSQL = Not Only SQL 泛指菲关系型数据库。

这些类型的数据存储不需要固定的模式，无需多余的操作就可以横向扩展。

### 1.2  特性

1. 易扩展：NoSQL数据库共同特点就是去掉关系数据库的关系特性，数据之间无关系，就很容易扩展。
2. 大数据量高性能
3. 多样灵活的数据模型：无需事先为要存储的数据建立字段。

### 1.3 分类

1. K-V键值
2. 文档型数据库（BSon格式比较多）
3. 列存储数据库
4. 图关系数据库

### 1.4 分布式数据库中CAP原理 CAP+BASE

#### 1.4.1 传统关系型数据库ACID原理

1. A   (Atomicity)原子性
2. C（Consistency）一致性
3. I（Isolation）独立性
4. D（Durability）持久性

#### 1.4.2 非关系型数据库CAP原理

1. C（Consistency）强一致性
2. A（Availability）可用性
3. P（Partition tolerance）分区容错性 

CAP理论核心：一个分布式系统不可能同时满足上面三种需求，最多只能同时满足两个。

CA：单点集群，满足一致性、可用性，通常在扩展性上不强。

CP：满足一致性、分区容错的系统，通常性能不是特别高。（Redis、mangoDB）

AP：满足可用性、分区容错性通常一致性要求低（**一般电商选择这种**）



**Base = Basically Available（基本可用） + Soft state (软状态) + Eventually consistent(最终一致性)**

思想是通过系统对某一时刻数据一致性的要求来换取系统整体的伸缩性和性能改观。



#### 1.4.3 分布式+ 集群简介

1. 分布式：不同的多台服务器上部署不同的模块，通过RPC/RMI之间的通信和调用，对外提供服务和组内协作。
2. 集群：不同的多台服务器部署相同的模块，通过分布式调度软件进行统一调度，对外提供服务。

## 2. Redis的入门介绍

### 2.1 入门概述

#### 2.1.1 是什么

**Redise：REmote DIctionary Server (远程字典服务器)**

开源、C语言编写、遵守BSD协议，支持持久化NoSQL数据库，被称为数据结构服务器。

特点：

> 1. Redis支持数据持久化，将内存数据保存在磁盘中，重启时可以加载使用。
> 2. Redis 不仅支持 key-value 类型数据，还支持 list、set、zset、hash等数据结构
> 3. Redis 支持 master-slave 模式的数据备份

#### 2.1.2 杂项基础知识

1. 单进程：通过对Linux的epoll函数包装在大批量的文件处理中的多路IO复用。
2. 默认16个库，从0开始，默认使用0号库。
3. 通过 select 1  来选取第几个库。
4.  Dbsize可以看到当前数据库的key总数。
5. FLUSHDB 清当前库
6. FLUSHALL清所有库
7. 统一密码管理，16个库都是同样的密码。
8. reids索引从0开始
9. 默认端口 6379

## 3. Redis 的数据类型

### 3.1 Redis的五大数据类型

1. String 类型：基础类型 key-value 对应；String类型是二进制安全的，意思是Reids的String可以包含任何数据。一个String类型的Value最多为 512M。
2. Hash：类似java里的Map，是一个键值对集合，是一个String类型的field和value的映射表，hash特别适合用户储存对象。类型java里的Map<String,Object>
3. List：简单的字符串列表，按照插入顺序排序，可以添加元素到列表头或尾，底层实际是链表。有序有重复。
4. Set：无序集合，无序无重复。散列。
5. ZSet：（sorted set）有序集合。Zset和set是String类型元素的集合，且不允许重复成员。不同的是 ZSet每个元素都会关联一个double 类型的分数，正是通过分数为集合中的成员排序。zset中的分数可以重复。

### 3.2 哪里获得Redis常见数据类型操作命令

http://redisdoc.com

### 3.3 Redis 键(Key)

```shell
keys * #查询所有的key
exists key #判断是否存在某个Key
move key 2 # 移动key到某个库
expire key 秒 # 给key设置过期时间
ttl key #查询key还有多少秒过期 -1表示永远  -2表示已过期
type key #查看key是什么类型
```

### 3.4 Redis 字符串(String)

```shell
set、get、del、append、strlen #字面意思
Incr/decr # 每次加/减1   在快速的统计时可以用
incrby key 2 /decrby key 2 # 每次可以批量加减 只有是数字才能加减 
getrange/setrange #获取/设置字符串 从第几位到第几位的值  getrange key 0 3
setex key 10 value #设置存在的键 秒 值
setnx key 10 value #设置不存在的 键 秒 值
mset/mget/msetnx #批量 键值 设置/获取 mset k1 v1 k2 v2 k3 v3;mget k1 k2 k3;msetnx 设置的key都不存在才生效  

```

### 3.5 Redis 列表(List)

```shell
lpush/rpush/lrange #left/rigth push  lpush list1 1 2 3 4  从list的左侧插入 元素，lrange 获取从 第几位到第几位的 值  lrange list1 0 2(-1表示全部) 
lpop/rpop #从栈顶移除一位元素
lindex #按照索引下标获得元素 （从左到右/从上到下）
llen #list 的长度
lrem key num # 删除n个 key 的 值  lrem list1 2 3  从list1中删除 2个 的值为3的值
ltrim key x y # 截取list 从第几位 到 第几位 并赋值到 list
rpoplpush list1 list2 #从list1 的尾部（rpop）弹处一个元素到 list2的 头部 （lpush）

```

性能总结

> 它是一个字符串链表，left、right 都可以插入添加
>
> 如果键不存在会插入新的链表
>
> 如果键已存在，新增内容
>
> 如果值全移除，对应的键也消失
>
> 链表的操作无论是对头和尾操作效率极高，对中间元素操作效率差

### 3.6 Redis 集合(Set)

```shell
sadd/smembers/sismember #sadd 会自动把重复的值过滤  sadd set1 1 2 3 ;smember set1 获取所有成员；sismember set1 1 检查 1 是否在 set中
scard #获取集合里面的元素个数
srem key value #删除集合中元素 srem set1 3 
srandmember key 3 #随机从set 中取3个数  可以用作抽检等 
spop key #随机出栈
smove key1 key2 key1里的某值 #在key1 里某个值 赋值给 key2
sdiff 差集      sinter 交集    sunion 并集 
```

### 3.7 Redis 哈希(Hash)

k-v模式不变  ，但V是一个键值对

```shell
hset/hget/hmset/hmget/hgetall/hdel 
hset user name ze # 单键值对
hmset user id 11 name 22 age 33 #多键值对

hlen #长度
hexists key # 判断key是否存在
hkeys/hvals #获取所有的keys/values

```

### 3.8 Redis有序集合Zset(sorted set)

在set的基础上加了一个 score 值，set 是  k1 v1 v2 v3  ;zset 是  k1 score1 v1  score2 v2

```sh
zadd #添加值  zadd zset1 60 v1 70 v2 
zrange #选取zset 范围的值（按下标算）  zrange zset1 0 1 (只显示值)  withscores （显示分数）
zrangebyscore key score1 score2 #选取 score的范围的值  zrangebyscore key 60 (90  表示分数大于60 小于90的值；zrangebyscore key 60 (90  limit 2 2  从第二个 截取两个 
zcard key #统计几组值（分数和值为一组）
zcount key score1 score2 #统计分数范围的值得个数
zrank key values值  #作用是获取这个值得下标
zscore key 对应值 #获取值得分数
zrevrank key values值/zrevrange/zrevrangebyscore key score1 score2  #表示逆序的相关功能

```

## 4. Redis的持久化

### 4.1 RDB(Redis DataBase)

#### 4.1.1 是什么？

在指定的时间间隔内将内存中的数据集快照写入到磁盘中。也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存中。

Redis会单独创建（Frok）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，在用这个临时文件替换上一次持久化好的文件。整个过程主进程都不进行任何IO操作，这就确保了极高的性能。

如果进行大规模的数据恢复，且对数据恢复的完整性不是特别敏感，那么RDB方式要比AOF更加高效。RDB的缺点是最后一次持久化后的数据可能丢失。

#### 4.1.2 Fork？

 Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）都和原进程一致，但是是一个全新的进程，并作为原进程的子进程。

缺点：备份的需要的资源和主进程一致，内存需要处理双倍的资源。

#### 4.1.3 如何触发RDB快照

 设置备份策略：save second change

默认出厂设置：

> 1分钟内改了1万次
>
> 5分钟内改了10次
>
> 15分钟内改了1次

也可以使用 save（只管保存，其他全部阻塞）或bgsave（会在后台异步进行快照操作） 命令手动保存

可以通过 lastsave 获取最后一次备份的时间。



查看Redis有没有启动： ps -f|grep redis 

备份一般是备份到远程机，防止出现物理损坏。

如果 flashAll 退出，先清空所有库，退出时系统自动生成dump.rdb文件，所以生成的是空的rdb文件。



#### 4.1.4 如何恢复

将备份文件（dump.rdb） 移动到redis安装目录并启动服务即可

config get dir 获取目录

#### 4.1.5 优势&劣势

优势：适合大规模的数据恢复，对数据完整性和一致性要求不高。

劣势：间隔一段时间备份一次，如果redis意外dwon掉，最后一次备份的数据就没有了

	    Frok的时候，内存内存中的数据被克隆了一份，大概2倍的膨胀性需求需要考虑。

#### 4.1.6 总结

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/Redis%E5%85%A5%E9%97%A8/RDB%E6%80%BB%E7%BB%93.jpg)





###4.2 AOF(Append Only File)

#### 4.2.1 是什么

以日志的形式来记录每个写操作。redis在启动的时候会按照日志的内容将写指令从前到后执行一遍完成数据的恢复工作。

Aof 保存的是 appendonly.aof 文件

#### 4.2.2 启动/修复/恢复

正常恢复：修改默认的 appendonly no 为yes；将有数据的aof文件复制一份保存到对应目录（config get dir）;重启redis然后重新加载。

异常恢复： 修改默认的 appendonly no 为yes；备份被写坏的AOF文件；"redis-check-aof --fix  文件名 " 进行修复 ；重启redis进行重新加载 

####  4.2.3 Rewrite

1. AOF 采用文件追加的方式，文件会越来越大为避免此种情况，新增了重写机制，当AOF文件的大小超过设定的阙值时，Redis会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用 bgrewriteaof
2. 重写原理： AOF文件持续增长而过大时，会fork出一个新进程来重写aof文件（也就是先写临时文件再rename），遍历新进程的内存中的数据，每条记录有一条set语句。重写AOF文件的操作，并没有读取就得aof文件，而是将内存中的额数据内容用命令的方式重新写了一个新的aof文件，这点和快照类似。
3. 触发机制：Redis会记录上次重写时的AOF大小，默认配置是当前AOF文件大小是上次rewrite后大小的一倍且文件大小超过64M时触发。

#### 4.2.4 优势

1. 优势 可以灵活的配置同步规则
   * 每秒同步：appendfsync always 同步持久化，每次发生数据变更就会立即记录到磁盘，性能较差但数据完整性较好。
   * 每秒同步：appendfsync everysec 异步操作 每秒记录 如果一秒内宕机  有数据丢失
   * 不同步：appendfsync no 从不同步
2. 劣势
   * 相同数据集数据aof文件远远大于rdb文件，恢复慢于rdb。
   * aof运行效率鳗鱼rdb，每秒同步策略较好，不同步效率和rdb相同。

#### 4.2.5 总结

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/Redis%E5%85%A5%E9%97%A8/AOF%E6%80%BB%E7%BB%93.jpg)

## 5. Redis的事务

### 5.1 是什么？

可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其他命令插入，不许加塞。

**一个队列中，一次性的、顺序的、排他性的执行一系列命令**

### 5.2 具体使用

Redis部分支持事务：如果出现致命错误 比如命令参数不正确 则还没有到esec 在 如queen的时候就直接报错，

并且所有命令都不生效；如果非致命错误，等到 esec 事务提交的时候，对的执行，错误的抛出异常。

### 5.3 watch 监控

1. 悲观锁：每次拿数据在操作之前都会上锁，别人在拿数据的时候就会block直到拿到锁。
2. 乐观锁：在拿到数据的时候不会上锁，在跟新的时候判断别人是否修改了数据，可以使用版本号机制，提交的版本号必须大于当前记录的版本号才能执行跟新。乐观锁用于多读的应用类型。
3. CAS：Check And Set

```shell
multi #开启事务
watch key[key1,key2...] #首先要先监控这个key
set key value #对key进行修改操作
exec #提交事务 如果其中没有人改过则成功，否则失败要重新watch	 
	 #执行了exec后所有加的监控锁都被取消掉


unwatch # 对所有key取消监视
```

### 5.4 阶段

1. 开启：以multi开始一个事务
2. 入队：将多个命令入队到事务中，接到这个命令不会立即执行，而是放到等待队列的事务队列里面。
3. 执行：有exec命令触发事务。

### 5.5 特性

1. 单独的隔离操作：事务中所有命令都会序列化、按顺序地执行。事务在执行过程中，不会被其他客户端发来的命令打断。
2. 没有隔离级别的概念：队列中的命令没有提交之前不会被执行。
3. 不保证原子性：redis同一个事务中如果有一条命令失败（非严重），其后的命令扔会被执行，没有回滚。

## 6. Redis 的复制（Master/Slave）

### 6.1 是什么

主从复制，主机跟新后根据配置自动同步到从机的Master/Slave机制。Master写为主，Slave读为主。

### 6.2 能干嘛

读写分离、容灾恢复。

### 6.3 怎么使用

配置从库不配置主库。

#### 6.3.1 从库配置

```shell
slaveof 127.0.0.1 8179 #主机IP  主机端口
```

如果从机与主机断开连接后，需要重新执行一下从机的配置命令，除非配置到redis.conf 文件里



#### 6.3.2 修改配置文件细节操作

1. 拷贝多个redis.conf 文件
2. 开启 daemonize yes
3. 修改 pid文件的名字
4. 指定端口号
5. Log文件名字
6. Dump.rdb 名字  

####6.3.3 常用的配置方式

使用 info replication 查看状态

1. 一主二仆

   只要设置了主从复制，第一次从机会把主机的所有数据都备份。

   主机才有写入权限，从机只能读。

   如果主机dwon掉，从机一直等待主机恢复。主机恢复后，从机立即会连接备份主机。

   从机 down 掉，恢复以后 从机变成 master主机，需要重新执行从机配置。除非写入配置文件。

2. 薪火相传

   * 普通的一主N从 中心化太严重，会影响主的性能，可以使用串联的方式。

   * 上一个Slave 可以是下一个 slave 的master，可以减轻master的写压力，而且整个复制集群中，只有一个master。别的都是slave，即使他作为别的slave的master。

   * 中途变更转向：会清除之前的数据，重新建立拷贝最新的数据。

3. 反客为主

   一主多从，如果主机down掉，需要选择一个新的从机作为主机。

   slaveof no one  手动使一台机器作为主机。

### 6.4 复制原理

1. Slave启动成功连接到master后会发送一个sync命令，Master接到命令启动后台的存盘进程，同时手机所有接收到的用于修改数据集命令，在后台进程执行完毕后，maser将传送整个数据文件到slave，已完成一次完全同步。
2. 全量复制：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
3. 增量复制：Master继续将新的所有收集到的修改命令一次传给slave，完成同步。
4. 但是只要是重新连接master，一次完全同步（全量复制）将被自动执行。

### 6.5 哨兵模式（sten）

####  6.5.1 是什么

反客为主的自动版，能够从后台监控主机是否故障，如果故障了根据投票数自动将从机转换为主机。

#### 6.5.2 操作步骤

1. 调整结构，改为一主二从的模式。

2. 新建sentinel.conf 配置文件，名字不能错。

3. 命令 touch sentinel.conf 创建一个空文件。

4. 编辑 sentinel.conf 文件 

   ```shell
   sentinel monitor host6379 127.0.1 6379 1
   #         被监控的主机的名字     ip   端口 超过多少票选举为Master
   ```

5. 启动哨兵

   ```shell
   redis-sentinel sentinel.conf  #实际配置文件路径
   ```

6. 如果原来的Master down机了，哨兵会组织投票，从原来的 slave 中选举一个Master

7. **如果原来的Master重新启动，哨兵会把原Master作为新Master的Slave**

8. 一组sentinel 可以同时监控多个Master

复制的缺点：所有的写操作都是在Master上，同步更新到Slave上，所以会有一定的延时，Slave机器数量的增加也会使这个问题更加严重。

