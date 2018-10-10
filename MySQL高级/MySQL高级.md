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

#### 5.2.1 SQL执行顺序

* 手写：

  ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-10_232901.jpg)


* 机读，服务层会解析SQL并生成内部解析树

 ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-10_233538.jpg)

MySQL机读是从From开始

* 总结

  ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/2018-10-10_233910.jpg)

SQL解析，从From开始，查出n个表的笛卡尔积（3 X 4 X 5),再按照各个条件筛选，最后选出用户需要查询的字段，排序返回。

#### 5.2.2 Join图

1. inner join （内连接）

   > a,b的共有

   ![](https://readingnotes.oss-cn-beijing.aliyuncs.com/MySQL%E9%AB%98%E7%BA%A7/INNER_JOIN.png)

```sql
select <select_list> from TableA a inner jion Table b on a.key = b.key;
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
select <select_list> from tableA a right jion tableB b on a.key = b.key;
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

