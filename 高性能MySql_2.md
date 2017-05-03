# 高性能MySql_2

## 1 MySql架构

### 1.1 日志

**二进制日志 (log-bin)**
记录所有更改数据的语句。主要用于主从复制和即时点恢复。

**错误日志 (log-error)**
记录严重的警告和错误信息，每次启动和关闭的详细信息等。默认关闭

**查询日志 (log)**
记录建立的客户端连接和执行的语句。默认关闭，如果开启会降低MySql整体性能

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

## 2 索引 

### 2.1 Sql执行顺序

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

### 2.2 索引

索引(Index)是帮助MySql高效获取数据的数据结构

**优势**

1. 提高数据检索效率，降低数据库IO成本
2. 通过索引列对数据进行排序，降低数据排序成本，降低CPU的消耗

**劣势**

1. 索引占用空间，实际上索引是一张表，该表保存了主键与索引字段，并指向实体表的记录
2. 索引会降低更新表速度，如对表进行INSERT、UPDATE的DELETE。更新表时，MySql不仅要保存数据，还要保存一下索引文件每次更新添加索引列的字段
3. 索引只是提高效率的一个因素

### 2.3 基本语法

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
### 2.4 需要创建索引

1. 主键自动建立唯一索引
2. 频繁作为查询条件的字段应该创建索引
3. 查询中与其他关联的字段，外键关系建立索引
4. 频繁更新的字段不适合创建索引
5. WHERE条件里用不到的字段不创建索引
6. 选择单键/组合索引
7. 查询中排序的字段，排列字段若通过索引去访问将大大提高排序速度
8. 查询中统计或者分组的字段

### 2.5 不需要创建索引

1. 表记录太少
2. 经常增删改的表
3. 数据重复且分布平均的表字段（为它建立索引没有太大实际效果）

## 3 EXPLAIN

### id

表示SELECT子句或操作表的顺序

- id相同，执行顺序由上向下
- id不同，如果是子查询，id的序号会递增，id值越大优先级越高
- id为null，表示是一个结果集，不需要使用它来进行查询

### select_type

1. SIMPLE: 简单的SELECT查询，查询中不包含子查询或者UNION
2. PRIMARY: 查询中若包含任何复杂的子部分，标记最外层查询
3. SUBQUERY: 在SELECT或WHERE列表中包含了子查询
4. DERIVED: 在FROM列表中包含的子查询被标记为DERVIED（衍生）MySql会递归执行这些子查询，把结果放在临时表里
5. UNION: 若第二个SELECT出现在UNION之后，则被标记为UNION。若UNION中包含在FROM子句的子查询中，外层SELECT将被标记为DERIVED
6. UNION RESULT: 从UNION表获取结果的SELECT

### table

表示这一行的数据时关于哪张表的

### type

显示查询使用到了何种类型

> system>const>eq_ref>ref>range>index>ALL
> 依次从好到差

一般来说，要保证查询至少达到range级别，最好达到ref

1. system: 表中只有一行数据或者是空表，const类型的特例，一般不出现
2. const: 通过索引一次就找到了，const用于比较primary key或者unique索引。如将主键置于where列表中，MySql就能将该查询转换为一个常量
3. eq_ref: 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
4. ref: 非唯一索引扫描，返回匹配某个单独值的所有行，本质上也是一中索引访问，它返回所有匹配某个单独值得行，然而，它可能会找到多个符合条件的行
5. range: 只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引，一般出现在where语句中使用了between、<、>、in等的查询。性能优于全表扫描，因为只需要开始于索引的某一点，而结束于另一点，不要扫描全部索引
6. index: Full Index Scan，index与ALL区别为index类型只遍历索引树。通常比ALL快，因为索引文件通常比数据文件小
7. all: Full Table Scan，将遍历全表以找到匹配的行

### possible_key

显示可能应用到这张表中的索引，一个或多个


