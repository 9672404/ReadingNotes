## 1.简介

关系型数据库

## 2.安装

Linux 安装不含中文和空格。

vi 在当前行的下一行输入 用 “o";

## 3.MySQL逻辑架构介绍

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/MySQL%E9%80%BB%E8%BE%91%E6%9E%B6%E6%9E%84.jpg)

1. 连接层：是一些客户端和链接服务，包含本地sock通信和大多数基于客户端/服务端工具实现的类似tcp/ip 的通信，主要完成一些链接处理、授权认证、及相关的安全方案。在改成上音引入了线程池的概念，为客户端提供线程。在该层上是实现了SSL安全链接。
2. 服务层：这一层完成大多数的核心业务。如SQL接口，并完成缓存查询，SQL的分析和优化及部分内置函数的执行。所有的夸储存引擎的功能也在这部分实现，如过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定查询表的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句,服务器还会查询内部的缓存。如果缓存足够大，在解决大量读操作的时候能很好的提升系统性能。
3. 引擎层：存储引擎真正的负责MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。不通的存储引擎有不同的功能。最长用的是InnoDB和MyISAM 引擎。
4. 存储层：主要是将数据存储在运行于裸设备的文件系统之上，并完成与存储引擎的交互。

## 4.存储引擎简介

1. 查看当前mysql的存储引擎

   ```shell
   show engines;
   ```



   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/show%20engines.jpg)

```shell
show variables like '%storage_engine%';
```

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E9%BB%98%E8%AE%A4%E5%BC%95%E6%93%8E.jpg)





2. MyISAM和InnoDB对比

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/MyISAM%E5%92%8CInnoDB%E5%AF%B9%E6%AF%94.jpg)

## 5.优化分析

### 5.1 性能下降SQL慢

> 表现：执行时间长，等待时间长

1. 先连接Linux服务器，执行Top命令，查看是否由于服务器硬件导致慢，内存不足、磁盘空间满了

2. 查询语句写的烂

3. 索引失效或者没创建索引

   > 单值索引： create index idx_user_name on user(name);
   >
   > 复合索引：create index idx_user_nameAge on user(name,age);
   >
   > idx开头表示索引，在user表的name字段创建索引，idx_user_name 索引名。

4. 关联查询太多的Join（设计缺陷或不得已的需求）

5. 服务器调优及各个参数设置（缓冲、线程池等）

### 5.2 常见通用的Join查询

```sql
select * from a(4记录),b(5记录);--笛卡尔积为20条记录
```

#### 5.2.1 SQLj加载执行顺序

* 手写：

  ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-10_232901.jpg)


* 机读，服务层会解析SQL并生成内部解析树

 ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-10_233538.jpg)

MySQL机读是从From开始

* 总结

  ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-10_233910.jpg)

SQL解析，从From开始，查出n个表的笛卡尔积（3 X 4 X 5),再按照各个条件筛选，最后选出用户需要查询的字段，排序返回。

#### 5.2.2 7种 Join的SQL

1. inner join （内连接）

   > a,b的共有

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/INNER_JOIN.png)

```sql
select <select_list> from TableA a inner join Table b on a.key = b.key;
```

2. left jion（左连接）

   > 左表的全部

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/LEFT_JOIN.png)

```sql
select <select_list> from TableA a left join TableB b on a.key = b.key;
```

3. right jion (右连接)

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/RIGHT_JOIN.png)

```sql
select <select_list> from tableA a right join tableB b on a.key = b.key;
```

4. LEFT JOIN EXCLUDING INNER JOIN （左连接排除内连接结果）

   > 左连接排除掉公共部分，公共部分可以用排除整个b来表示

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/LEFT_EXCLUDING_JOIN.png)

```sql
select <select_list> from TableA a left join TableB b on a.key = b.key where b.key is null;
```

5. RIGHT JOIN EXCLUDING INNER JOIN （右连接排除内连接结果）

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/RIGHT_EXCLUDING_JOIN.png)

```sql
select <select_list> from TableA a right join TableB b on a.key = b.key where b.key is null;
```

6. FULL OUTER JOIN （外连接）

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/FULL_OUTER_JOIN.png)

```sql
select <select_list> from tableA a full outer join TableB b on a.key = b.key;
```

7. OUTER JOIN EXCLUDING INNER JOIN （外连接排除内连接结果）

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/OUTER_EXCLUDING_JOIN.png)

```sql
select <select_list> from tableA a full outer join tableB b on a.key = b.key where a.key is null or b.key is null;
```

### 5.3 索引简介

#### 5.3.1 索引是什么？

> MySQL官方对索引的定义：索引（Index）是帮助MySQL高效获取数据的数据结构。本质 索引是数据结构。
>
> 索引的目的是提高查询效率，可以类比字典。

**索引会影响 where 后面的 查找 和 order by 后面的排序**

**可以理解为：排好序的快速查询数据结构。**

* 详解（重要）：在数据本身之外，**数据库系统还维护这满足特定查找算法的数据结构**，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构就是索引。

  ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-11_154155.jpg)

左边是表数据，一共两列七条记录，最左边是数据记录的物理地址。

为了加快Col2的查找，可以维护一个右边所示的二叉查找树（B树索引），每个节点分别包含索引键值和一个指向对应数据记录物理地址的指针，这样就可以运用二叉查找在一定的复杂度内获取到相应的数据，从而快速检索出符合条件的记录。

 

索引失效会引起查询慢。--> 重建索引

为什么查询快增删慢？   在更新记录的时候还要更新索引所以慢。

* 一般来说索引本身也很大，不可能全部储存在内存中，因此索引往往是以索引文件的形式储存在磁盘上。

* **我们平常所说的索引，如果没有特别指明，都是指B树（多路搜索树，不一定是二叉树）结构组织的索引。**其中聚集索引、次要索引、覆盖索引、复合索引、前缀索引、唯一默认索都是使用B+树索引，统称索引。当然除了B+树这种类型的索引之外，还有哈希索引（hash Index）等。


#### 5.3.2 优势

* 提高数据检索的效率，降低数据库的IO成本。
* 降低数据排序的成本，降低了CPU的消耗。

#### 5.3.3 劣势

* 实际上索引也是一张表（.MYI文件），该表保存了主键与索引字段，并指向实体表的记录，所以索引列也是要占空间的。
* 虽然索引大大提高了查询速度，同时会降低表的insert、update、delete 速度，更新表的时候，MySQL不仅要更新数据，还要更新一下索引文件，带来额外的资源消耗。

#### 5.3.4 MySQ索引分类

1. 单值索引

   > 一个索引只包含单个列，一个表中可以有多个单列索引。一张表最多不要超过5个索引。

2. 唯一索引

   > 索引列的值必须唯一，但允许有空值。例如：银行卡号不能重复，建唯一索引。

3. 复合索引

   > 一个索引包含多个列

4. 基本语法

   创建：

   ```sql
   create [unique] index indexName on tableName(columname(length));
   alert tableName add [unique] index [indexName] on (columnane(length));
   ```

   删除：

   ```sql
   drop index [indexName] on tableName;
   ```

   查看

   ```sql
   show index from tableName;
   ```


#### 5.3.5 MySQL索引结构

1. BTree索引 - **检索原理**

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-12_213221.jpg)



> 一颗B+树 ，真实的数据存在叶子节点，非叶子节点不存储真实的数据，只存储指引搜索方向的数据项。
>
> **过程：**如果要查询数据项29，首先会把磁块1由磁盘加载到内存中，此时发生一次IO，在内存中用二分查找确定，29在17和35之间，锁定磁盘块1的P2指针，内存时间非常短（相比磁盘的IO）可以忽略不计，通过磁盘块1的P2指针的磁盘地址把磁盘块3有磁盘加载到内存中，发生第二次IO，29在26和30之间，锁定磁盘块3的P2指针，通过指针架子啊磁盘块8到内存，发生第3次IO，同时内存中做二分查找找到29，结束查询，总计3次IO。



1. Hash索引、full-text索引、R-Tree索引    （了解）

#### 5.3.6 哪些情况适合建索引？

1. 主键自动创建唯一索引。
2. 频繁作为查询条件的字段需要创建索引。
3. 查询中与其他表关联的字段，外键关系建立索引。
4. 在高并发下倾向创建复合索引。
5. 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度。
6. 查询中统计或分组字段（分组的前提是必排序）





#### 5.3.7 哪些情况不要建索引？



1. 表记录太少（300W以下不用）

2. 频繁跟新的字段不适合（提高查询速度，但是更新的同时要更新索引文件，所以慢）

3. where条件用不到的字段不要建索引。

4. 如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

   > 假如一个表有10条记录，有一个字段只有True、false两种值，每种值的分布概率为50%，对这种字段创建索引一般不会提高数据库的查询效率。
   >
   > **索引的选择性是指索引列中不同值得数目与表中记录数的对比。如果一个表有2000条记录，表索引列有1980个不同的值，这个索引的选择性就为 1980/2000 = 0.99 ,一个索引的选择性越接近1，索引的效率就越高**。



### 5.4 性能分析

#### 5.4.1 MySQL Query Optimizer(查询优化器)

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-12_230221.jpg)

#### 5.4.2 MySQL的常见瓶颈

1. CPU：CPU饱和的时候一般发生在数据装入内存或从磁盘读取数据的时候。
2. IO：磁盘I/O瓶颈发生在装入的数据远大于内存容量的时候。
3. 服务器硬件性能瓶颈：top，free，iostat和vmstat来查看系统的性能状态。

#### 5.4.3 Explain

##### 5.4.3.1 是什么？（查看执行计划）

> 使用Explain关键字可以模拟优化器执行SQL语句，从而知道MySQL是如何处理你的SQL，从而查看表结构或查询语句是否存在瓶颈。

##### 5.4.3.2 能干什么？

1. 表的读取顺序
2. 数据读取操作的操作类型
3. 哪些索引可以使用
4. 哪些索引被实际使用
5. 标准件的引用
6. 每张表有多少行被优化器查询

##### 5.4.3.2 怎么用？

1. Explain + SQL

2. 执行计划包含的信息:

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-13_173446.jpg)

##### 5.4.3.2 ID

1. select 查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序。

2. 三种情况

   1. **<font color="#dd0000">ID相同，执行顺序由上至下。</font>**

      ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/ID%E7%9B%B8%E5%90%8C.jpg)

   2. **<font color="#dd0000">ID不相同</font>**

      ![ID不相同](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/ID%E4%B8%8D%E7%9B%B8%E5%90%8C.jpg)

   3. **<font color="#dd0000">ID相同不同，同时存在。</font>**

      ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/ID%E7%9B%B8%E5%90%8C%E5%8F%88%E4%B8%8D%E5%90%8C.jpg)

##### 5.4.3.2 Select_Type

1. 
































