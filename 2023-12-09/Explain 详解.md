# Explain 作用

一条查询语句在经过 MySQL 查询优化器的各种基于成本和规则的优化后会生成一个所谓的执行计划，Explain 语句就是用来帮助我们查看某个查询语句的具体执行计划的信息，以及如何解释输出



# Explain 列名概述

| 列名          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | 语句的唯一标识，每个 select 关键字都对应一个唯一的 id        |
| select_type   | 查询类型                                                     |
| table         | 表名                                                         |
| partitions    | 分区                                                         |
| type          | 访问类型 —— MySQL 决定如何查找表中的行                       |
| possible_keys | 可能用到的索引                                               |
| key           | 实际用到的索引                                               |
| key_len       | 索引使用的字节数，主要是为了区分某个使用联合索引的查询具体用了几个索引列 |
| ref           | 当使用索引列等值查询时（也就是访问类型是const、eq_ref、ref、ref_or_null、unique_subquery、index_subquery其中之一时），与索引列进行等值匹配的对象信息 |
| rows          | 预估的需要读取的记录条数                                     |
| filtered      | 符合查询条件的数据百分比                                     |
| Extra         | 不适合在其他列显示的额外信息                                 |



# Explain 列名详解

## id

该语句的唯一标识。如果explain的结果包含多个 id 值，则数字越大越先执行；而对于相同 id 的行，则表示从上往下依次执行



一条查询语句可以包含多个 select 关键字（每个 select 关键字都会分配一个唯一的 id 值），而每个 select 关键字的 from 子句中又可以包含多张表（连接查询），每一张表都对应着执行计划输出中的一条记录（这些记录的 id 值都是相同的，出现在前面的表表示驱动表，出现在后边的表表示被驱动表）



一条查询语句中出现多个 select 关键字的情况：

* 查询中包含子查询的情况
* 查询中包含 union 语句的情况



## select_type

MySQL 将 select 查询分为简单和复杂类型，复杂类型可分成三大类：简单子查询、派生表（在 from 子句中的子查询），以及 union 查询



**查询类型可能取值：**

| 查询类型             | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| simple               | 简单查询（未使用 union 或子查询）                            |
| primary              | 最外层的查询                                                 |
| subquery             | 子查询中的第一个 select                                      |
| dependent subquery   | 子查询中的第一个 select，依赖了外面的查询                    |
| union                | 在 union 中的第二个和随后的 select 被标记为 union。如果 union 被 from 子句中的子查询包含，那么它的第一个 select 会被标记为 derived |
| dependent union      | 子查询里 union 中的第二个或后面的查询，依赖了外面的查询      |
| union result         | 用来从 union 的匿名临时表检索结果的 select                   |
| derived              | 派生表，用来表示包含在 from 子句的子查询中的 select          |
| dependent derived    | 派生表，依赖了其他的表                                       |
| materialized         | 物化子查询（存储子查询结果集的临时表称之为物化表，物化表中的记录都不重复并且都建立了索引，基于内存的物化表是哈希索引，基于磁盘的是B+树索引） |
| uncacheable subquery | 子查询，结果无法缓存，必须针对外部查询的每一行重新评估       |
| uncacheable union    | union 属于 uncacheable subquery 的第二个或后面的查询         |



## type

**性能类型从好到坏排序：**system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery> index_subquery > range > index > all

| 访问类型        | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| system          | 该表只有一条记录并且该表使用的存储引擎的统计数据是精确的，比如MyISAM、Memory |
| const           | 主键或者唯一二级索引等值查询                                 |
| eq_ref          | 在连接查询时，被驱动表是主键或者唯一二级索引列等值查询（如果该主键或者唯一二级索引是联合索引的话，所有的索引列都必须进行等值比较） |
| ref             | 普通二级索引等值查询                                         |
| fulltext        | 全文索引                                                     |
| ref_or_null     | 普通二级索引等值查询，并且索引列的值可以是 NULL              |
| index_merge     | Intersection、Union、Sort-Union 索引合并                     |
| unique_subquery | 子查询，且子查询是主键或唯一二级索引等值查询                 |
| index_subquery  | 子查询，且子查询是普通二级索引等值查询                       |
| range           | 范围扫描，比较常见的范围扫描是 where 子句里有 >、>=、<、<=、 <>、!=、between、like、in() 和 or 等操作符 |
| index           | 全索引扫描                                                   |
| all             | 全表扫描                                                     |



## key_len

key_len 表示索引使用的最大字节数，它有三个部分组成：

* 对于固定字节数的索引列来说，最大存储空间就是该固定字节数；对于指定字符集的变长类型的索引列来说，最大存储空间就是该索引列占用的最大字节数
* 可以存储空值的索引列比不可以存储空值的索引列多占用 1 个字节
* 变长类型索引列还会有 2 个字节来存储该变长类型索引列的实际长度

| 类型       | tinyint | smallint | mediumint | int  | bigint | float | double | datetime | timestamp |
| ---------- | ------- | -------- | --------- | ---- | ------ | ----- | ------ | -------- | --------- |
| 固定字节数 | 1       | 2        | 3         | 4    | 8      | 4     | 8      | 8        | 4         |



## Extra

| 额外信息                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| No tables used                                               | 查询没有 from 子句或有 from dual 子句                        |
| Impossible WHERE                                             | where 子句始终为 false                                       |
| No matching min/max row                                      | 查询有 min 或 max 聚集函数，但是并没有符合 where 子句中的搜索条件的记录 |
| Using index                                                  | 覆盖索引                                                     |
| Using index condition                                        | 索引条件下推（每次执行回表操作，都需要将一个聚簇索引页面加载到内存里。而索引条件下推就是在扫描某个范围区间内的二级索引记录时，尽可能的减少回表次数），存在 like 可能发生索引条件下推 |
| Using where                                                  | 搜索条件需要在 server 层过滤时                               |
| Using join buffer（Blocked Nested Loop）                     | 基于块的嵌套循环算法（join buffer 就是执行连接查询前申请的一块固定大小的内存，先把若干条驱动表结果集中的记录装在这个 join buffer 中，然后开始扫描被驱动表，每一条被驱动表的记录一次性和 join buffer中的多条驱动表记录做匹配，因为匹配过程是在内存中完成的，可以显著减少被驱动表的 I/O 代价） |
| Not exists                                                   | MySQL 对 left join 的优化，如果被驱动表的某个列包含等于 NULL 值的搜索条件，但是那个列是不允许存储空值的 |
| Using intersect(...)、Using union(...)、Using sort_union(...) | 索引合并的方式                                               |
| Zero limit                                                   | 该查询有 limit 0 子句                                        |
| Using filesort                                               | 文件排序，包括内存和磁盘排序                                 |
| Using temporary                                              | 内部临时表，出现在 distinct、group by、union 等子句的查询中  |
| Start temporary，End temporary                               | In 子查询转为 semi-join时，且semi-join 的执行策略为 DuplicateWeedout 时，也就是通过建立临时表来实现为外层查询的记录进行去重操作时，驱动表查询计划的 Extra 列将显示 Start temporary 提示，被驱动表查询计划的 Extra 列将显示 End temporary 提示 |
| LooseScan                                                    | In 子查询转为 semi-join 时，采用的执行策略为LooseScan        |
| FirshMatch(tbl_name)                                         | In 子查询转为 semi-join时，采用的执行策略为 FirstMatch       |



# 名词解释

## index merge 索引合并

### Intersection 合并

Intersection 是交集的意思，这适用于使用不同索引的搜索条件之间使用 AND 连接起来的情况

**可能使用 Intersection 索引合并的情况：**

* 二级索引列是等值匹配的情况（对于联合索引来说，在联合索引中的每个列都必须等值匹配，不能出现匹配部分列的情况）
* 主键列可以是范围匹配（索引列的值相同的记录又是按照主键排序的）



### Union 合并

Union 是并集的意思，适用于不同索引的搜索条件之间使用 OR 连接起来的情况

**可能使用 Union 索引合并的情况：**

* 二级索引列是等值匹配的情况（对于联合索引来说，在联合索引中的每个列都必须等值匹配，不能出现匹配部分列的情况）
* 主键列可以是范围匹配（索引列的值相同的记录又是按照主键排序的）
* 使用 Intersection 索引合并的搜索条件



### Sort-Union 合并

Sort-Union 索引合并是先按照二级索引记录的主键值进行排序，之后按照 Union 索引合并执行的方式称之为 Sort-Union 索引合并。Sort-Union 索引合并比 Union 索引合并多了一步对二级索引记录的主键值排序的过程



## semi-join

**必须符合所有这些条件才可能使用 semi-join 半连接：**

* 子查询必须是和 in 语句组成的布尔表达式，并且在外层查询的 where 或者 on 子句中出现
* 外层查询的搜索条件和 in 子查询的搜索条件必须使用 and 连接
* 子查询必须是一个单一的查询，不能由 union 连接，不能包含 group by 或者 having 语句或者聚集函数



### Table pollout（子查询中的表上拉）

当子查询的查询列表处只有主键或者唯一索引时，可以直接把子查询中的表上拉到外层查询的 from 子句中，并把子查询中的搜索条件合并到外层查询的搜索条件中

### DuplicateWeedout（重复值消除）

把 semi-join 作为一个常规的 inner join，使用一个临时表，将重复的记录排除

### LooseScan（松散扫描）

先扫描内表（子查询中的表），然后到外表中寻找符合匹配条件的第一条记录

### Materialization（物化表）

先把外层查询的 in 子句中的不相关子查询进行物化（物化表中没有重复的记录），然后再进行外层查询的表和物化表的连接

### FirstMatch（首次匹配）

先取外层查询中的一条记录，到子查询中的表中寻找符合匹配条件的第一条记录























