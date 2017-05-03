# 高性能MySQL_2

## 1 MySQL架构

### 1.1 日志

**二进制日志 (log-bin)**
记录所有更改数据的语句。主要用于主从复制和即时点恢复。

**错误日志 (log-error)**
记录严重的警告和错误信息，每次启动和关闭的详细信息等。默认关闭

**查询日志 (log)**
记录建立的客户端连接和执行的语句。默认关闭，如果开启会降低MySQL整体性能

**慢查询日志 (log-slow)**
记录所有执行时间超过long_query_time秒的所有查询或不使用索引的查询。默认关闭

[详细](http://www.jb51.net/article/69301.htm)

### 1.2 存储引擎

对比           |  MyISAM      | InnoDB
------------- | ------------- | ------------
主外键         | 不支持         | 支持
事务           | 不支持         | 支持
行表锁         | 表锁，<br>即使操作一条记录也会锁住整个表，<br>不适合高并发的操作                      | 行锁，<br>操作时只能锁某一行，<br>不对其他行有影响，适合高并发的操作
缓存           | 只缓存索引，不缓存真实数据 | 不仅缓存索引还要缓存真实数据，<br>内存要求较高，而且内存大小对性能有决定性的影响
表空间         |  小            | 大 
关注点         |  性能          | 事务

## 2 索引优化分析 

### 2.1 索引介绍
#### 2.1.1 Sql执行顺序

**手写顺序**

```sql
SELECT DISTINCT 
	< select_list >
FROM
	<left_table> <join_type>
JOIN <right_table> ON <join_condition>
WHERE
	<where_condition>
GROUP BY
	<group_by_list>
HAVING
	<having_condition>
ORDER BY
	<order_by_condition>
LIMIT <limit number>
```

**机读顺序**

```sql
FROM <left table>
ON <join_condition>
<join_type> JOIN <right_table>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
SELECT
DISTINCT <select_list>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```

#### 2.1.2 索引

索引(Index)是帮助MySQL高效获取数据的数据结构

**优势**

1. 提高数据检索效率，降低数据库IO成本
2. 通过索引列对数据进行排序，降低数据排序成本，降低CPU的消耗

**劣势**

1. 索引占用空间，实际上索引是一张表，该表保存了主键与索引字段，并指向实体表的记录
2. 索引会降低更新表速度，如对表进行INSERT、UPDATE的DELETE。更新表时，MySQL不仅要保存数据，还要保存一下索引文件每次更新添加索引列的字段
3. 索引只是提高效率的一个因素

#### 2.1.3 基本语法

**创建**

```sql
CREATE [UNIQUE] INDEX indexName ON table (columnname(length));
```

```sql
ALTER table ADD [UNIQUE] INDEX [indexName] ON (columnname(length))
```

**删除**

```sql
DROP INDEX [indexName] ON table
```

**查看**

```sql
SHOW INDEX FROM tableName
```
#### 2.1.4 需要创建索引

1. 主键自动建立唯一索引
2. 频繁作为查询条件的字段应该创建索引
3. 查询中与其他关联的字段，外键关系建立索引
4. 频繁更新的字段不适合创建索引
5. WHERE条件里用不到的字段不创建索引
6. 选择单键/组合索引
7. 查询中排序的字段，排列字段若通过索引去访问将大大提高排序速度
8. 查询中统计或者分组的字段

#### 2.1.5 不需要创建索引

1. 表记录太少
2. 经常增删改的表
3. 数据重复且分布平均的表字段（为它建立索引没有太大实际效果）

### 2.2 性能分析

#### 2.2.1 EXPLAIN

##### id

MySQL将SELECT查询分为简单和复杂类型，复杂类型可分为三大类：简单子查询，所谓的派生表（在FROM子句中的子查询），UNION查询

表示SELECT子句或操作表的顺序

- id相同，执行顺序由上向下
- id不同，如果是子查询，id的序号会递增，id值越大优先级越高
- id为null，表示是一个结果集，不需要使用它来进行查询

##### select_type

显示了对应行是简单还是复杂SELECT

1. **SIMPLE**: 简单的SELECT查询，查询中不包含子查询或者UNION
2. **PRIMARY**: 查询中若包含任何复杂的子部分，标记最外层查询
3. **SUBQUERY**: 包含在SELECT列表中子查询的SELECT（换句话说，不在FROM子句中）
4. **DERIVED**: 在FROM子句的子查询中的SELECT，MySQL会递归执行并将结果放到一个临时表中。服务器内部称为“派生表”，因为该临时表是从子查询中派生来的。
5. **UNION**: 早UNION中的第二个和随后的SELECT被标记为UNION。若UNION中包含在FROM子句的子查询中，外层SELECT将被标记为DERIVED
6. **UNION RESULT**: 从UNION的匿名临时表检索结果的SELECT

##### table

表示这一行的数据时关于哪张表的

##### type

显示查询使用到了何种类型

> system>const>eq_ref>ref>range>index>ALL
> 依次从好到差

一般来说，要保证查询至少达到range级别，最好达到ref

- **ALL** <br>全表扫描
- **index** <br>和全表扫描一样，只是MySQL扫描表时按索引次序进行而不是行。主要优点是避免排序，缺点是要承担按索引次序读取整个表的开销，意味着若是按照随机次序访问行，开销将会非常大
- **range** <br>范围扫描就是一个有限制的索引扫描，它开始于索引里的某一点，返回匹配这个值域的行。这比全索引扫描好一些，因为不用遍历全部索引。一般出现在where语句中使用了between、<、>、in等的查询。
- **ref** <br>这是一种索引访问，它返回所有匹配某个单个值的行。然而，它可能会找到多个符合条件的行，因此，它是查找和扫描的混合体。此类索引访问只有当使用非唯一性索引或者唯一性索引的非唯一性前缀时才发生
- **eq_ref** <br> 使用这种索引查找，MySQL知道最多只返回一条符合条件的记录。这种访问方法可以在MySQL使用主键或者唯一性索引查找时看到，它会将它们与某个参考值作比较
- **const，system** <br>当MySQL能对查询的某部分进行优化并将其转换成一个常量时，它就会使用这些访问类型。比如，将某一行的主键放入WHERE子句里的方式来选取此行的主键，MySQL就能将这个查询转换成为一个常量，然后高效地将表从连接执行中移除
- **NULL** <br> MySQL能在优化阶段分段查询语句，在执行阶段甚至用不着再访问表或者索引

##### possible_key

显示查询可以使用哪些索引，结果一个或多个
查询涉及的子弹上若存在索引，则该索引将被列出，但不一定被查询实际使用

##### key

显示MySQL决定采用哪个索引来优化对该表的访问

##### key_len

显示MySQL在索引里使用的字节数，可通过该列计算查询中使用的索引长度，在不损失精确度情况下，长度越短越好
<br>显示的值为索引字段的最大可能长度，并非实际使用长度

##### ref

显示之前的表在key列记录的索引中查找值所用的列或变量

##### rows

显示大致估算出为了找到所需的行而要读取的行数

##### Extra

额外信息

- ***Using filesort*** <br>意味着MySQL会对结果使用一个外部索引排序，而不是按索引次序从表里读取行。内部重排序结果，需要优化。
- ***Using temporary*** <br>意味着MySQL在对查询结果排序时会使用一个临时表，常见于order by和group by。使用临时表保存中间结果，需要优化。
- ***Using index*** <br>表示MySQL将使用覆盖索引，以避免访问表的数据行，效率不错<br>如果同时出现using where，表明索引呗用来执行键值的查找<br>如果没有同时出现using where，表明索引用来读取数据而非执行查询动作
- **Using where**<br> 表示MySQL服务器将在存储引擎检索行后再进行过滤。许多WHERE条件里涉及索引中的列，当（并且如果）它读取索引时，就能被存储引擎检验，因此不是所有带where子句的查询都会显示Using where
- **Using join buffer**<br> 使用了连接缓存

### 2.3 Join优化

数据库中JOIN操作的实现主要有三种：嵌套循环连接（Nested Loop Join），归并连接（Merge Join）和散列连接或者哈稀连接（Hash Join）

- 尽量减少Join语句中的嵌套循环连接(nested loops join)总次数，用小结果集驱动大的结果集
- 优先优化nested loop的内层循环
- 保证Join语句中被驱动表上Join条件字段已经被索引
- 当无法保证被驱动表的Join条件字段被索引且内存资源充足的前提下，使用JoinBuffer设置

### 2.4 索引失效

1. 最佳左前缀法则<br>如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列
2. 不在索引列上做任何操作(计算、函数、类型转换)，会导致索引失效而转向全表扫描
3. 范围条件右边的列索引失效，范围查询使字段用于排序，后面使用检索的索引失效
4. 尽量使用覆盖索引（索引列和查询列一致），减少select *
5. MySQL在使用不等于(!=或者<>)的时候无法使用索引会导致全表扫描
6. is null, is not null无法使用索引
7. like以通配符开头('%XX')MySQL索引失效会变成全表扫描操作
8. 字符串不加单引号索引失效
9. 用or链接时索引失效

## 3 查询截取分析

### 3.1 查询优化

#### 3.1.1 小表驱动大表

```sql
SELECT * FROM A WHERE id IN (SELECT id FROM B)
等价于
FOR SELECT id FROM B
FOR SELECT * FROM A WHERE A.id = B.id
```

当B表的数据集小于A表的数据集时，用IN优于EXISTS

```sql
SELECT * FROM A WHERE EXISTS (SELECT 1 FROM B WHERE B.id = A.id)
等价于
FOR SELECT * FROM A
FOR SELECT * FROM B WHERE B.id = A.id
```

当A表的数据集小于B表的数据集时，用EXISTS优于IN

**EXISTS**
将主查询的数据，放到子查询中做条件验证，根据验证结果(TRUE或FALSE)来决定查询的数据结果是否得意保留

#### 3.1.2 ORDER BY排序优化

1. ORDER BY子句，尽量使用index方式排序，避免使用filesort方式排序 <br>满足两种情况，会使用index方式排序
	1. ORDER BY 语句使用索引最左前列
	2. 使用WHERE子句与ORDER BY子句条件列组合满足索引最左前列
2. 尽可能在索引列上完成排序操作，遵守索引建的最佳左前缀
3. 如果不在索引列上，filesort有两种算法：MySQL要启动双路排序和单路排序
4. 优化策略<br>
	- 增大sort_buffer_size参数配置
	- 增大max_length_for_sort_data参数的设置
	
#### 3.1.3 GROUP BY

1. GROUP BY 实质是先排序后进行分组，遵照索引的最佳左前缀
2. 当无法使用索引列，增大max_length_for_sort_data参数的设置，增大sort_buffer_size参数的设置
3. WHERE高于HAVING，能在WHERE限定的条件就不要HAVING限定了

### 3.2 慢查询日志

#### 3.2.1 查看

查看开启
```sql
SHOW VARIABLES LIKE '%slow_query_log%';
```
开启
```
set global slow_query_log=1;
```

查看时间，默认10s
```sql
SHOW VARIABLES LIKE 'long_query_time%'; 
```

设置
```
set global long_query_time=3
```

#### 3.2.2 mysqldumpslow

得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/log-slow.log

得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/log-slow.log

得到按照时间排序的前10条里面含有左链接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/log-slow.log

结合 | more 使用
mysqldumpslow -s t -t 10 /var/lib/mysql/log-slow.log | more
