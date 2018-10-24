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

1. 表的读取顺序 (Id)
2. 数据读取操作的操作类型（Select_Type）
3. 哪些索引可以使用
4. 哪些索引被实际使用
5. 标准件的引用
6. 每张表有多少行被优化器查询

##### 5.4.3.3 怎么用？

1. Explain + SQL

2. 执行计划包含的信息:

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-13_173446.jpg)

##### 5.4.3.4 ID

> select 查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序。

1.  **ID相同，执行顺序由上至下。**

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/ID%E7%9B%B8%E5%90%8C.jpg)

------



2. **ID不相同。**

![ID不相同](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/ID%E4%B8%8D%E7%9B%B8%E5%90%8C.jpg)

------



3. **ID相同不同，同时存在。**

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/ID%E7%9B%B8%E5%90%8C%E5%8F%88%E4%B8%8D%E5%90%8C.jpg)

##### 5.4.3.5 Select_Type

> 查询的类型，主要用于区别简单查询、联合查询、子查询等复杂查询。

1. SIMPLE：简单的查询，查询中不包含子查询或者Union。

2. PRIMARY：查询中如果包含任何复杂的子部分，则最外层被标记为。

3. SUBQUERY：在select或Where列表中包含子查询。

4. DERIVED：在From列表中包含的子查询被标记为DERIVED（衍生），MySQL会递归执行这些子查询，把结果放到临时表里。

5.  UNION：若第二个SELECT出现在UNION之后，则被标记为UNION；

   若UNION包含在FROM子句的子查询中，外层的SELECT将被标记为：DERIVED。

6. UNION RESULT：从UNION表获取结果的SELECT。

##### 5.4.3.6 Table

就是哪张表的数据。

##### 5.4.3.7 Type

和SQL是否优化过，是否是最佳状态有关。

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Explain_Type.jpg)

显示查询使用了何种类型：

从最好到最差依次为：system > const > eq_ref > range > index > ALL

百万以上的All需要优化，全表扫描。

一般来说，得保证查询至少达到range级别，最好能达到ref。 

1. **System：**表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，可以忽略不计。

2. **Const：**表示通过索引一次就找到，用于比较Primary Key或Unique索引。因为只匹配一行数据，所以很快。如将主键置于Where列表中，MySQL就能将该查询转换为一个常量。

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Type_Const.jpg)

   > id 为主键 只通过一次查询就可以找到结果。
   >
   > select * from d1;单表且只有一条记录，所以为System。

   ------

3.  **Eq_Ref：**唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Type_eq_ref.jpg)

   > 使用了索引，但是恰巧只有一条记录，例如：员工表和部门表，员工表只有一条记录的 部门ID 与 部门表ID匹配，这种情况为  eq_ref.

   ---



4. **Ref：**非唯一索引扫描，返回匹配某个单独值得所有行。本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体。

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Type_ref.jpg)

   > 查询 col1 为固定值得所有行。

   ---

    

5.  **Range：**只检索给定范围的行，使用一个索引来选择行。Key列显示使用了哪个索引。一般就是在where语句中出现了between、<、>、in 、等的查询。这种范围扫描比全表扫描要好。

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Type_range.jpg)

   > 按照开始时间 & 结束时间 查询记录 就是 range

   ---

    

6. **Index：**Full Index scan 全索引扫描。只遍历索引树，通常比All快，索引文件比较小，并且 index是从索引读取的（索引文件会加载到内存中），All是从硬盘读取的，减少IO。

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Type_index.jpg)

   > id 为 主键  唯一索引 ，如果只查ID 则为 全索引扫描。

   ---

    

7. **All：**全表扫描

##### 5.4.3.8 Key

1. possible_kyes：可能使用到的索引，查询涉及到的字段上存在索引则列出，但实际查询不一定用到。

2. key：实际使用的索引，如果为Null则没有使用索引。

3. 若查询中使用了覆盖索引，则该索引只会出现在key中。

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Explain_key.jpg)

   > 如果查询的字段的个数和顺序  和  创建的复合索引的字段的个数顺序一致，就会出现MySQL认为不使用索引，但是实际使用索引的情况。

4. key_len：表示索引中使用的字节数，在不损失精度的情况下，字节数越少越好。显示的值为索引字段最大可能长度，并非实际使用长度。

##### 5.4.3.9 Ref

显示索引的哪一行被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值。

##### 5.4.3.10 Rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录要读取的行数。

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Explain_Rows.jpg)

##### 5.4.3.11 Extra

包含不适合在其他列显示但十分重要的额外信息。

1. **Using filesort：**说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序操作称为“文件排序”。

   > MySQL一般情况下是按照SQL字段的顺序排序，但是如果出现特殊情况MySQL则会自己重新排序。

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Extra_Using_filesort.jpg)


第一个没有使用 col2索引，第二个使用了col2索引，查询条件的顺序和个数  同创建的索引的字段的个数顺序相同，查询效率将大大增大。

2. **Using temporary：**使用了临时表保存中间结果，MySQL在对查询结果排序的时使用临时表。常用于排序order by 和 分组查询  group by。
   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Extra_Using_temporary.jpg)

如果有索引，group尽量按照索引的个数和顺序来，否则很容易出现文件内排序和临时表。临时表是很影响系统性能的。



3. **Using Index：**表示相应的select使用了覆盖索引，数据直接从索引中来，索引已经加载到了内存中，省去了很多次的IO，所以速度快。



### 5.5 索引优化

#### 5.5.1 索引分析

1. 索引，左连接建到右表，右连接建到左表。由查询特性  左（右）表的数据为全部数据，left（right）Join 条件用于确定如何从右（左边）开始搜索。

   > 1. 尽可能减少Join语句中的NestedLoop（嵌套循环）的循环总次数，”永远用小结果集驱动大的结果集“。
   >
   > 2. 优先优化NestedLoop的内层循环。
   > 3. 保证Join语句被驱动表（大表）上Join条件字段已经被索引。
   > 4. 当无法保证被驱动表的Join条件字段被索引且内存充足的情况下，调大JoinBuffer的设置。


#### 5.5.2 索引失效

1. **全值匹配我我最爱**

2. **最佳左前缀法则**

   > 如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且**不跳过索引中的列**。按照顺序来，不能不使用第一个或第二个 直接使用后面作为查询条件，否则索引会失效。
   >
   > *带头大哥不能死，中间兄弟不能断。*

3. **不在索引列上做任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描**

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E7%B4%A2%E5%BC%95%E5%88%97%E4%B8%8A%E5%B0%91%E8%AE%A1%E7%AE%97.jpg)

   > *索引列上少计算。*

4. **存储引擎不能使用索引中范围条件右边的列**。

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E8%8C%83%E5%9B%B4%E4%B9%8B%E5%90%8E%E5%85%A8%E5%A4%B1%E6%95%88.jpg)

   > *范围之后全失效*

5. **尽量使用覆盖索引（只访问索引的查询（索引列和查询列一直），减少select * ）**

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E5%B0%BD%E9%87%8F%E4%BD%BF%E7%94%A8%E8%A6%86%E7%9B%96%E7%B4%A2%E5%BC%95.jpg)

6. **mysql 在使用不等于 （!= 或 <>）的时候无法使用索引会导致全表扫描**

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E4%B8%8D%E7%AD%89%E5%8F%B7%E4%BD%BF%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88.jpg)

7. **is null ,is not null 也无法使用索引。**

8. **like 以通配符开头（'%abc...'）mysql索引失效会变成全表扫描的操作。**

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%25%E5%8F%B7%E5%8A%A0%E5%9C%A8like%E5%8F%B3%E8%A1%A8.jpg)

   > % 加在like 字符串的 右边，索引失效 ，两边和前边都失效。如果生产环境非得使用 "%aa%",使用覆盖索引解决。

9. **字符串不加单引号索引失效。**

   > varchar  必须使用 单引号。如果不加单引号会发生自动的类型转换，会造成索引失效（见3）
   >
   > *字符串里有引号*

10. **少用or,用它来连接时会索引失效。**



11. **总结**

    ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E6%80%BB%E7%BB%93.jpg)

12. 面试题分析

    group by 分组之前必排序，所以group by 和 order by 的排序规则一致。


#### 5.5.3 优化总结口诀

> 全值匹配我最爱，最左前缀要遵守；
>
> 带头大哥不能死，中间兄弟不能断；
>
> 索引列上少计算，范围之后全失效；
>
> LIKE百分写最右，覆盖索引不写星；
>
> 不等空值还有or，索引失效要少用；
>
> VAR引号不可丢，SQL高级也不难！



### 5.6 查询截取分析

#### 5.6.1 查询优化

##### 5.6.1 解决慢查询的步骤

> * 慢查询的开启并捕获。
> * explain + 慢SQL 执行计划分析。
> * show profile 查询SQL在MySQL服务器里面的执行细节和生命周期情况。
> * SQL数据库服务器的参数调优。



##### 5.6.2 永远小表驱动大表

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/in%E5%92%8Cexists%E5%8C%BA%E5%88%AB.jpg)



![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/exists%E7%9F%A5%E8%AF%86%E7%82%B9.jpg)



##### 5.6.3 Order By 关键字优化

1. Order By子句尽量使用Index方式排序，避免使用FileSort方式排序。MySQL支持 Index 和  FileSort 排序，前者效率高，表示扫描索引本身完成排序。

   Order By 满足两种情况会使用Index排序：（依旧符合最佳左前缀原则）

   * Order By 语句使用索引最左前列。
   * 使用Where 子句与Order By 子句条件列组合满足索引最左前列。

2. 尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀。

3. 如果不在索引列上，fileSort有两种算法：

   * 双路排序：MySQL4.1以前 使用， 扫两次磁盘。取一次数据需要发生两次扫面，I/O是很耗时的。

   * 单路排序： 从磁盘上读取查询需要的所有列，按照Order By 在 sortBuffer 对它们进行排序，扫描排序后的列表进行输出，因为只发生了一次I/O在内存中排序，所以单路相对快一点

     单路总体由于双路，但是单路有问题

     ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E5%8D%95%E8%B7%AF%E6%8E%92%E5%BA%8F%E7%BC%BA%E7%82%B9.jpg)

   * 增大sort_buffer_size 参数的设置
   * 增大max_length_for_sort_data 参数的设置

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E6%8F%90%E9%AB%98Order%20By%20%E9%80%9F%E5%BA%A6.jpg)

4. 总结

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/Order%20By%E7%B4%A2%E5%BC%95%E4%BD%BF%E7%94%A8%E6%83%85%E5%86%B5.jpg)

##### 5.6.4 Group By 关键字优化

* Group By 实质是先排序后进行分组，遵照索引建的最佳左前缀。
* 当无法使用索引列，增大max_length_for_sort_data 参数的设置 + 增大sort_buffer_size 参数的设置。
* where 高于having ，能写在where限定的条件就不要去having限定了。

#### 5.6.2 慢查询日志

打开慢查询日志，设置慢SQL阙值，捕获到慢SQL，使用exlpain, mysqldumpslow 分析慢SQL。

#### 5.6.3 ShowProfile

默认关闭，并保存最近15次的运行结果。

##### 5.6.3.1 是否支持

```sql
show variables like 'profiling';
```

##### 5.6.3.2 开启

```sql
set profiloing=on;
```

##### 5.6.3.3 查看结果

```sql
show profiles;
```

##### 5.6.3.4 诊断SQL，查看SQL的生命周期

```sql
show profile cpu,block io for query 10;
```

##### 5.6.3.5 日常开发的结论

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/show%20profile%20%E5%BC%82%E5%B8%B8%E6%83%85%E5%86%B5.jpg)



##6.MySQL锁机制

### 6.1 表锁（偏读）

> 偏向MyISAM存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最低，并发度最低。



session1 给 table1 加了表锁的读锁，session1 可以读table1，不能跟新table1，不能读别的表。session2 可以读table1 阻塞，不能跟新table1。

session1 给 table1加了表锁的写锁，session1可以读,写table1，不能读别的表；session2 读table1 阻塞。

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/MyISAM%E8%A1%A8%E9%94%81%E7%9A%84%E7%89%B9%E7%82%B9.jpg)

**总而言之，就是读锁会阻塞写，但是不会阻塞读。而写锁则会把读和写都阻塞。**

**Myisam的读写锁调度是写优先，这也是myisam不适合做写为主表的引擎。因为写锁后，其他线程不能做任何操作，大量的跟会使查询很难得到锁，从而造成永久的阻塞。**

### 6.2 行锁（偏写）

> 偏向InnoDB存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生所冲突的概率最低，并发度也最高。
>
> InnoDB区别于 MyISAM最大的不同：1. 支持事物；2.采用了行锁。

#### 6.2.1 事务的相关知识

1. 事务（Transaction）及其ACID属性

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E4%BA%8B%E5%8A%A1%E7%9A%84ACID%E5%B1%9E%E6%80%A7.jpg)

2. 并发事务处理带来的问题：更新丢失、脏读（已修改但未提交）、不可重复读、幻读。

3. 事务的隔离级别。

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E4%BA%8B%E5%8A%A1%E7%9A%84%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB.jpg)

#### 6.2.2 无效索引行锁升级为表锁

正常情况下，两个连接操作不同的行数据互不干扰，如果这个表中创建了索引，在查询的时候导致索引失效，比如查询时字符串没有加【’单引号‘】，就会是行锁变为表锁，别的连接在更新此表的别的数据也会发生阻塞。

#### 6.2.3 间隙锁的危害

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E9%97%B4%E9%9A%99%E9%94%81%E7%9A%84%E5%8D%B1%E5%AE%B3.jpg)

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E9%97%B4%E9%9A%99%E9%94%81.jpg)

#### 6.2.4 如何手动锁定一行

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/%E6%89%8B%E5%8A%A8%E9%94%81%E5%AE%9A%E4%B8%80%E8%A1%8C.jpg)

#### 6.2.5 总结

InnoDB实现了行锁，性能损耗会比表锁高一些，但是在高并发下处理能力远远高于myisam的表锁。

InnoDB也有脆弱的一面，使用不当时行锁变表锁，会让InnoDB整体性能低甚至比表锁差。

#### 6.2.6 如何分析行锁定

```sql
show status like 'innodb_row_lock%';
```

#### 6.2.7 行锁优化建议

* 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁
* 合理设计索引，尽量缩小锁的范围
* 尽量减少检索条件，避免间隙锁
* 尽量控制事务的大小，减小锁定资源量和时间长度
* 尽可能低级别事务隔离

