# 高性能MySql

# 1. MySql逻辑架构

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/mysql/1.png)

第一层的服务包含连接管理、授权认证、安全等等

第二次架构是MySql核心服务功能，包括查询解析、分析、优化、缓存以及所有的内置函数（例如，日期、时间、数学和加密函数），所有跨存储引擎的功能都在这一层实现：存储过程、触发器、试图等。

第三次包含了存储引擎。存储引擎负责MySql中数据的存储和提取。服务器通过API与存储引擎进行通信。这些接口屏蔽了不同存储引擎之间的差异，使得这些差异对上层的查询过程透明。存储引擎不会去解析sql，不同的存储引擎之间也不会互相通信，而只是简单地响应上层服务器请求。

## 1.1 连接管理与安全性

每个客户端连接都会在服务器进程中拥有一个线程，这个连接的查询只会在这个单独的线程中执行，该线程只能轮流在某个CPU核心或者CPU中运行。服务器会负载缓存线程，因此不需要为每一个新建的连接创建或者销毁线程。

## 1.2 优化与执行

MySql会解析查询，并创建内部数据结构（解析树），然后对其进行各种优化，包括重写查询、决定表的读取顺序，以及选择合适的索引等。用户可以通过特殊的关键字提示优化器，影响它的决策过程。也可以请求优化器解析优化过程的各个因素，使用户可以知道服务器是如何进行优化决策的，并提供一个参考基准，便于用户重构查询和schema、修改相关配置，使应用尽可能高效运行。

优化器并不关心表使用的是什么存储引擎，但存储引擎对于优化查询是有影响的。优化器会请求存储引擎提供容量或某个具体操作的开销信息，以及表数据的统计信息等。

对于SELECT语句，在解析查询之前，服务器会先检查查询缓存，如果能够在其中找到对应的查询，服务器就不必再执行查询解析、优化和执行的整个过程，而是直接返回查询缓存中的结果集

# 2 并发控制

## 2.1 读写锁

## 2.2 锁粒度

表锁（table lock）

行级锁（row lock）

# 3 事务

**原子性（atomicity）**

一个事务必须被视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚，对于一个事务来说，不可能只执行其中的一个小部分操作，这就是事务的原子性

**一致性（consistency）**

数据库总是从一个一致性的状态转换到另一个一致性的状态

**隔离线（isolation）**

通常来说，一个事物所做的修改在最终提交以前，对其他事务是不可见的。

**持久性（durability）**

一旦事务提交，则其所做的修改就会永久保存到数据库中。此时即使系统崩溃，修改的数据也不会丢失。

## 3.1 隔离级别

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/mysql/2.png)

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/mysql/3.png)

## 3.2 死锁

## 3.3 事务日志

事务日志可以帮助提高事务的效率。使用事务日志，存储引擎在修改表的数据使只需要修改其内存拷贝，再把该修改行为记录到持久在硬盘上的事务日志中，而不用每次都将修改的数据本身持久到磁盘。事务日志采用的是追加的方式

## 3.4 事务

### 3.4.1 自动提交(AUTOCOMMIT)

MySQL默认采用自动提交（AUTOCOMMIT）模式。也就是说，如果不是显示地开始一个事物，则每个查询都被当做一个事务执行提交操作。在当前连接中，可以通过设置AUTOCOMMIT变量来启用或者禁止自动提交模式

### 3.4.2 隐式和显式锁定

InnoDB采用的是两阶段锁定协议（two-phase  locking protocol）。在事务执行过程中，随时都可以执行锁定，锁只有在执行COMMIT或者ROLLBACK的时候才会释放，并且所有的锁是在同一时刻被释放。<br>
InnoDB会根据隔离级别在需要的时候自动加锁<br>
InnoDB也支持通过特定语句进行显示锁定

- SELECT ... LOCK IN SHARE MODE
- SELECT ... FOR UPDATE

## 3.5 MVCC多版本并发控制

MVCC的实现，是通过保存数据在某个时间点的快照来实现的。也就是说，不管需要执行多长时间，每个事物看到的数据都是一致的。根据事务开始的时间不同，每个事物对同一张表，同一时刻看到的数据可能是不一样的。<br>
不同引擎的MVCC实现是不同的，典型的有乐观（optimistic）并发控制和悲观（pessimistic）并发控制。

InnoDB的MVCC，是通过在每行记录后面保存两个隐藏的列来实现的。这两个列，一个保存了行的创建时间，一个保存行的过期时间（或删除时间）。当然存储的不是实际的时间值，而是系统版本号。没开始一个新的事物，系统版本号都会自动递增。

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/mysql/9.png)

保存这两个额外的系统版本号，使大多数读操作都可以不用加锁。这样设计使得数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行。不足之处是每行记录都需要额外的存储空间，需要做更多的行检查工作，以及一些额外的维护工作。

# 4 Schema与数据类型优化

## 4.1 选择优化的数据类型

1. 更小的通常更好
2. 简单就好
3. 尽量避免NULL
	如果查询中包含可为NULL的列，对MySql来说更难优化，因为可为NULL的列使得索引、索引统计和值比较都更复杂。可为NULL的列会使用更多的存储空间，在MySql里也需要做特殊处理。

**VARCHAR**

VARCHAR类型用于存储可变长字符创，它比定长类型更节省空间。有一中情况例外，如果MySql表使用ROW_FORMAT=FIXED创建的话，每一行都会使用定长存储，这会浪费空间。

VARCHAR需要使用1或2个额外字节记录字符串的长度

VARCHAR子UPDATE时可能使行变得比原来的更长，这就导致需要额外的工作。如果一个行占用的空间增长，并且在页内没有更多的空间存储，在这种情况下，不同的存储引擎的处理方式不一样。例如MyISAM会将行拆成不同的片段存储，InnoDB则需要分裂页来使行可以放进页内。

**CHAR**

CHAR类型是定长的：MySql总是根据定义的字符串长度分配足够的空间。当存储CHAR值时，MySql会删除所有的末尾空格

**BLOB和TEXT**

BLOB和TEXT都是为存储很大的数据而设计的字符串数据类型，分别采用二进制和字符方式存储
字符类型：TINYTEXT，SMALLTEXT，TEXT，MEDIUMTEXT，LONGTEXT
二进制类型：TINYBLOB，SMALLBLOB，BLOB，MEDIUMBLOB，LONGBLOB
BLOB是SMALLBLOB的同义词，TEXT是SMALLTEXT的同义词
BLOB类型存储的是二进制数据，没有排序规则或字符集，而TEXT类型有字符集和排序规则

**DATETIME**

这个类型能保存大范围的值，从1001年到9999年，精度为秒。它把日期和时间封装到格式为YYYYMMDDHHMMSS的整数中，与时区无关。使用8个字节的存储空间
**TIMESTAMP**
TIMESTAMP类型保存了从1970年1月1日午休依赖的秒数，它和UNIX的时间戳相同。TIMESTAMP只使用4个字节的存储空间，因此它的范围比DATETIME小得多：只能表示从1970年到2038年。MySql提供了FROM_UNIXTIME()函数把Unix时间戳转换为日期，并提供UNIX_TIMESTAMP()函数把日期转换为Unix时间戳
TIMESTAMP显示的值也依赖于时区
默认情况下，如果插入时没有指定第一个TIMESTAMP列的值，MySql则设置这个列的值为当前时间。在插入一行记录时，MySql默认也会更新第一个TIMESTAMP列的值

# 5 索引

## 5.1 索引的类型

**B-Tree索引**

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/mysql/4.png)

**哈希索引**

只有Memory引擎显示支持哈希索引

InnoDB引擎有一个特殊的功能叫“自适应哈希索引（adaptive hash index）”当InnoDB注意到某些索引值被使用的非常频繁时，它会在内存中基于B-Tree索引之上再创建一个哈希索引

**空间数据索引（R-Tree）**

MyISAM表支持空间索引，可用作地理数据存储

**全文索引**

## 5.2 索引的优点

1. 索引大大减少了服务器需要扫描的数据量
2. 索引可以帮助服务器避免排序和临时表
3. 索引可以将随机I/O变为顺序I/O

## 5.3 高性能的索引策略

1. 独立的列

	```sql
	SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;
	```
	
	这个查询将无法使用actor_id列的索引，操作将在每个行上进行运算，这将导致索引失效而进行全表扫描
	
2. 前缀索引和索引选择性

	有时候需要索引很长的字符列，会让索引变得大且慢。一种策略是模拟哈希索引。通常可以索引开始的部分字符，这样可以大大节约索引空间，提供效率

3. 多列索引

	当服务器对多个索引做相交（AND）或联合（OR）操作时

4. 合适的索引列顺序
	
5. 聚簇索引

	聚簇索引不是一种单独的索引类型，而是一种数据存储方式。具体的细节依赖其实现方式，但InnoDB的聚簇索引实际上在同一个结构中保存了B—Tree索引和数据行
	
	当表有聚簇索引时，它的数据行实际上存放在索引的叶子页（leaf page）中。“聚簇”表示数据行和相邻的键值紧凑的存储在一起。
	
	![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/mysql/5.png)
	
	聚簇索引优点：
	
- 可以把相关数据保存在一起。例如实现电子邮件时，可以根据用户ID来聚集数据，这样只需要从磁盘读取少量的数据页就能获取某个用户的全部邮件。如果没有使用聚簇索引，则每封邮件都可能导致一次磁盘I/O
- 数据访问更快。聚簇索引将索引和数据保存在同一个B-Tree中，因此从聚簇索引中获取数据通常比非聚簇索引中查找要快
- 使用覆盖索引扫描的查询可以直接使用页节点中的主键值

	缺点：
	
- 聚簇数据最大限度的提高了I/O密集型应用的性能，但如果数据全部都放在内存中，则访问的顺序就没那么重要了，聚簇索引也就没有什么优势了
- 插入速度严重依赖插入顺序
- 更新聚簇索引列的代价很高
- 基于聚簇索引的表在插入新行，或者主键被更新导致需要移动行的时候，可能面临“页分裂（page split）”的问题
- 聚簇索引可能导致全表扫描变慢，尤其是行比较稀疏，或者由于页分裂导致数据数据存储不连续的时候
- 二级索引可能比想象的要更大

6. 覆盖索引

## 5.5 总结

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/mysql/6.png)

# 6 查询性能优化

## 6.1 慢查询基础：优化数据访问

查询性能低下最基本的原因是访问的数据太多，通过两个步骤分析：

1. 确认应用程序是否在检索大量超过需要的数据。这通常意味着访问了太多的行，但有时候也可能是访问了太多的列
2. 确认MySql服务器层是否在分析大量超过需要额数据行

### 6.1.2 是否向数据库请求了不需要的数据

有些查询会请求超过实际需求的数据，然后这些多余的数据会被应用程序丢弃。这会给MySql服务器带来额外的负担，并增加网络开销，另外也会消耗应用服务器的CPU和内存资源

- 查询不需要的记录
- 多表关联时返回全部列
- 总是取出全部列
- 重复查询相同的数据

### 6.1.2 MySql是否在扫描额外的记录

对于MySql，最简单的衡量查询开销的三个指标：

1. 响应时间
2. 扫描的行数
3. 返回的行数

如果发现查询需要扫描大量的数据但只返回少数的行，可以使用的优化技巧：

- 使用索引覆盖扫描，把所有需要的列都放在索引中，这样存储引擎无须回表获取对应行就可以返回结果了
- 改变库表结构。例如使用单独的汇总表
- 重写这个复杂的查询，让MySql优化器能够以更优化的方式执行这个查询

## 6.2 重构查询方式

### 6.2.1 一个复杂查询还是多个简单查询

在其他条件都相同的时候，使用尽可能少的查询当然是更好的。但是有时候，讲一个大查询分解为多个小查询是很有必要的

### 6.2.2 切分查询

将大查询切分成小查询，每个查询功能完全一样，只完成一小部分，每次只返回一小部分查询结果。

eg：删除旧的数据：定期地清楚大量数据时，如果用一个大的语句一次性完成的话，则可能需要一次锁住很多数据、占满整个事务日志、耗尽系统资源、阻塞很多小的但重要的查询。将一个大的DELETE语句切分成多个较小的查询可以尽可能小地影响MySql性能，同时可以减少MySql复制的延迟。

```sql
DELETE FROM messages WHERE created < DATE_SUB(NOW(), INTERVAL 3 MONTH);
```

可以用类似的方法

```sql
rows_affetced=0
do {
  rows_addfected = do_query(
   "DELETE FROM messages WHERE created < DATE_SUB(NOW(), INTERVAL 3 MONTH) LIMIT 10000" )
} while rows_affected > 0
```

### 6.2.3 分解关联查询

很多高性能的应用都会对关联查询进行分解。简单地，可以对每一个表进行一次单表查询，然后将结果在应用程序中进行关联

```sql
SELECT * FROM tag
	JOIN tag_post ON tag_post.tag_id=tag.id
	JOIN post ON tag_post.post_id=post.id
WHERE tag.tag='mysql'
```

可以分解为

```sql
SELECT * FROM tag WHERE tag='mysql';
SELECT * FROM tag_post WHERE tag_id=1234;
SELECT * FROM post WHERE post.id in (123,456,789);
```

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/mysql/7.png)

## 6.3 查询执行的基础

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/mysql/8.png)

### 6.3.1 MySql客户端/服务器通信协议

MySql客户端和服务器之间通行协议是“半双工”的，这意味着，在任何一个时刻，要么是由服务器向客户端发送数据，要么是由客户端向服务器发送数据，这两个动作不能同时发送。

### 6.3.2 查询缓存

### 6.3.3 查询优化处理

### 6.3.4 查询执行引擎

## 6.4 MySql查询优化器的局限性

### 6.4.1 关联子查询

MySql的子查询实现非常糟糕。最糟糕的一类查询是WHERE条件中包含IN()的子查询语句。

```sql
SELECT * FROM sakila.film WHERE film_id IN (SELECT film_id FROM sakila.film_actor WHERE actor_id = 1);
```

MySql会将相关的外层表压到子查询中，MySql会将查询改写成下面的样子:

```sql
SELECT * FROM sakila.film 
WHERE EXISTS (
	SELECT * FROM sakila.film_actor WHERE actor_id = 1 AND film_actor.film_id = film.film_id);
```

这时，子查询需要根据film_id来关联外部表film，因为需要film_id字段，MySql先选择对file表进行全表扫描，然后根据返回的film_id逐个执行子查询。

```sql
SELECT film.* FROM sakila.flim INNER JOIN sakila.film_actor USING(film_id) WHERE actor_id = 1;
```

### 6.4.2 UNION的限制

有时，MySql无法将限制条件从外层“下推”到内层，这使得原本能够限制部分返回结果条件无法应用到内层查询的优化上
如果希望UNION的各个字句能够根据LIMIT只取部分结果集，或者希望能够先排好序再合并结果集的话，就需要在UNION的各个子句中分别使用这些子句。

例如，想将两个子查询结果联合起来，然后再取前20条记录，那么MySql会将两个表都放在同一个临时表中，然后再取出前20行记录：

```sql
(SELECT first_name, last_name FROM sakila.actor ORDER BY last_name) UNION ALL (SELECT first_name, last_name FROM sakila.customer ORDER BY last_name) LIMIT 20;
```

这条查询会将actor中的200条记录和customer中的599条记录放在一个临时表中，然后再从临时表中取出前20条，可以通过在UNION的两个子查询中分别加上一个LIMIT 20来减少临时表中的数据

```sql
(SELECT first_name, last_name FROM sakila.actor ORDER BY last_name LIMIT 20) UNION ALL (SELECT first_name, last_name FROM sakila.customer ORDER BY last_name LIMIT 20) LIMIT 20
```

### 6.4.3 索引合并优化

当WHERE子句中包含多个复杂条件的时候，MySql能够访问单个表的多个索引以合并和交叉过滤的方式来定位需要查找的行

### 6.4.4 并行执行

MySql无法利用多核特性来并行执行查询

### 6.4.5 最大值和最小值优化

```sql
SELECT MIN(actor_id) FROM sakila.actor WHERE first_name = 'PENELOPE'
```

因为在first_name字段上没有索引，MySql将会进行一次全表扫描，如果MySql能够进行主键扫描，那么理论上，当MySql读到第一个满足条件的记录时候，就是我们要的最小值了，因为主键是严格按照actor_id字段的大小顺序排列的。但MySql只会做全表扫描
一种曲线优化的方法是移除MIN(),然后用LIMIT来查询重写如下

```sql
SELECT actor_id FROM sakila.actor USE INDEX(PRIMARY) WHERE first_name = 'PENLOPE' LIMIT 1;
```

### 6.4.6 在同一个表上查询和更新

MySql不允许对同一个表同时进行查询和更新

```sql
UPDATE tb1 AS outer_tb1 SET cnt = (
	SELECT count(*) FROM tb1 AS inner_tb1 WHERE innertb1.type = outer_tb1.type
);
```

可以通过生成表的形式来绕过上面的限制，因为MySql只会把这个表当做一个临时表来处理

```sql
UPDATE tb1 INNER JOIN (
	SELECT type, count(*) AS cnt FROM tb1 GROUP BY type
) AS der USING(type) SET tb1.cnt = der.cnt;
```

## 6.5 优化特定类型的查询

### 6.5.1 优化COUNT()查询

**COUNT()的作用**

COUNT()是一个特殊的函数，有两种不同的作用：它可以统计某个列值的数量，也可以统计函数。在统计列值时要求列值是非空的（不统计NULL）。如果在COUNT()的括号中指定了列或者列的表达式，则统计的就是这个表达式有值得结果数
COUNT()的另一个作用是统计结果集的行数。当MySql确认括号内的表达式值不可能为空时，实际上就是在统计行数。

### 6.5.2 优化关联查询

- 确保ON或者USING子句中的列上有索引
- 确保任何的GROUP BY和OEDER BY中的表达式只涉及到一个表中的列

### 6.5.3 优化GROUP BY和DISTINCT

SQL_BIG_RESULT和SQL_SMALL_RESULT

### 6.5.4 优化LIMIT分页

优化分页查询一个最简单的办法就是尽可能地使用索引覆盖扫描，而不是查询所有的列。然后根据需要做一次关联操作再返回所需的列

```sql
SELECT film_id, description FROM sakila.film ORDER BY title LIMIT 50, 5;
```

可以写为

```sql
SELECT film.film_id, film.description FROM sakila.film INNER JOIN (
	SELECT film_id FROM sakila.film ORDER BY title LIMIT 50, 5
) AS lim USING(film_id);
```

这里的“延迟关联”将大大提升查询效率，它让MySql扫描尽可能少的页面，获取需要访问的记录后再根据关联列回原表查询需要的所有列。


