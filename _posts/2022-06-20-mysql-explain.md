---
layout: post
title: 'MySQL索引与执行计划'
subtitle: ''
date: 2022-06-20
author: Dave
cover: ''
tags: 数据库
---
## MySQL索引与执行计划

### 1.索引

#### 1.1 SQL执行过程

> SQL语句执行过程如下，索引的选择性使用发生在优化器阶段。

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/SQL执行过程.png)

> 注：MySQL 5.7适用，8.0已无查询缓存。

#### 1.2 MySQL索引归档

##### 索引结构

> MYSQL 中索引是使用 B+ 树的数据结构，区别于B树。也可以创建哈希索引。

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/b-与b+树.png)

> InnoDB存储引擎最小储存单元：页 16K/Page

```java
B+树
1) 叶子节点存数据，内节点（非叶子节点）不存数据，只保存索引，一页16k能存放更多，且查询效率更稳定；
2) 叶子节点链接，区间查找更优；
```



##### 索引分类

> 此处，索引划分依据功能划分与物理实现划分。

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/MySQL索引归档.png)

###### InnoDB聚集索引

当我们基于 InnoDB 引擎创建一张表的时候，都会创建一个聚集索引，每张表都有唯一的聚集索引：

- 如果这张表定义了主键索引，那么这个主键索引就作为聚集索引。

- 如果这张表没有定义主键索引，那么该表的第一个唯一非空索引作为聚集索引。

- 如果这张表也没有唯一非空索引，那么 InnoDB 内部会生成一个隐藏的主键作为聚集索引，这个隐藏的主键是一个 6 个字节的列，该列的值会随着数据的插入自增。



##### 存储引擎差异

> MySQL存储引擎：MyISAM、InnoDB、MERGE、MEMORY(HEAP)、BDB(BerkeleyDB)、EXAMPLE、FEDERATED、ARCHIVE、CSV、BLACKHOLE。此处，着重InnoDB 与 MyISAM。

###### InnoDB 与 MyISAM主要差异（底层）

- 索引存储内容：InnoDB聚集索引叶子节点存储数据，非聚集索引叶子节点存储聚集索引的索引值（常常为主键值）；MyISAM主键索引与普通索引叶子节点均存储数据行指针。
- 索引物理存储：InnoDB表数据与索引存储于同一文件，`ibd`格式；MyISAM表数据与索引分开存储，表数据`MYD`格式（MYData 的缩写），索引数据`MYI`格式（MYIndex 的缩写）。

具体地，如下图所示，以数据类`（id,name,gender,age）`数据为例，对name建立索引进行查询。物理存储上，`frm`为表结构文件。

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/innodb与myisam索引差别与存储差异.png)


> 注:定位mysql索引存储位置，先定位mysql数据路径，如下。

```sql
mysql> SHOW VARIABLES LIKE '%datadir%';
+---------------+-----------------------+
| Variable_name | Value                 |
+---------------+-----------------------+
| datadir       | /usr/local/var/mysql/ |
+---------------+-----------------------+
1 row in set (0.02 sec)
```

###### InnoDB 与 MyISAM主要差异（上层）

- 主键索引：InnoDB 主键索引为聚集索引； MyISAM主键索引仅性质上为非NULL的唯一索引；

- 事务：InnoDB 支持事务； MyISAM不支持事务；

- 锁：InnoDB 提供表锁、行锁； MyISAM提供表锁； InnoDB的行锁实现在索引上，行锁过程会先锁住二级索引再锁住主键索引，而非锁在物理行记录。如果访问没有命中索引，也无法使用行锁，将要退化为表锁。

- 外键：InnoDB 支持外键； MyISAM不支持；

- 读写性能：InnoDB 适合频繁更新插入，并发大； MyISAM适合频繁查询与插入，更新较少；

-  表行数读写：InnoDB不保存表具体行数，执行select count(\*) from table时需全表扫描。而MyISAM有一变量存储了表行数，执行上述语句（无任何WHERE条件）只需读取该变量即可；

  

### 2.Explain执行计划

- [Explain Mysql 官方文档](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)

> Explain 关键字可以模拟优化器执行 SQL 查询语句，执行会返回执行计划的信息，而不是执行这条SQL，当然如果 from 中包含子查询，仍会执行该子查询，将结果放入临时表中 。
> 通过 Explain 从而知道 MySQL 是如何处理你的 SQL 语句的，分析查询语句或是表结构的性能瓶颈；通过Explain 执行计划我们可以获取什么信息？
> •	  表的读取顺序
> •	  数据读取操作的操作类型
> •	  哪些索引可能被使用
> •	  哪些索引实际被使用
> •	  表之间的引用
> •	  每张表估计有多少行会被执行

> 以下以5.7版本MySQL为例

```sql
mysql> select version() from dual;
+-----------+
| version() |
+-----------+
| 5.7.37    |
+-----------+
1 row in set (0.00 sec)
mysql> explain select oa_account from t_members where exists (select * from t_members where oa_account != 'dave1' and born_year = '1999' and position = 'SDE');
+----+-------------+-----------+------------+-------+---------------------------+---------------------------+---------+------+-------+----------+--------------------------+
| id | select_type | table     | partitions | type  | possible_keys             | key                       | key_len | ref  | rows  | filtered | Extra                    |
+----+-------------+-----------+------------+-------+---------------------------+---------------------------+---------+------+-------+----------+--------------------------+
|  1 | PRIMARY     | t_members | NULL       | index | NULL                      | idx_oa_born_year_position | 137     | NULL | 60187 |   100.00 | Using index              |
|  2 | SUBQUERY    | t_members | NULL       | range | idx_oa_born_year_position | idx_oa_born_year_position | 74      | NULL | 60186 |     1.00 | Using where; Using index |
+----+-------------+-----------+------------+-------+---------------------------+---------------------------+---------+------+-------+----------+--------------------------+
2 rows in set (0.00 sec)
```



#### 2.1 Explain 执行计划的属性

##### id

> id 列表示的编号是 select 的序列号，有几个 select 就有几个 id，并且 id 的顺序是按 select 出现的顺序增长的，id 列越大执行优先级越高，id 相同则从上往下执行，id 为 NULL 最后执行。

##### select_type

> select_type 列表示对应行查询的类型。
> •	1) SIMPLE：简单查询，不包含子查询和 union。
> •	2) PRIMARY：复杂查询中最外层的 select。
> •	3) SUBQUERY：包含在 select 中的子查询(不在 from 子句中)。
> •	4) DERIVED：包含在 from 子句中的子查询。MySQL 会将结果存放在一个临时表中，也称为派生表(derived的英文含义)。
> •	5) UNION：在 union 中的第二个和随后的 select。
> •	6) UNION RESULT：从 union 临时表检索结果的 select。

##### table

> table 列表示 explain 的一行正在访问哪个表。

##### partitions

> 表示匹配的分区

##### type(重点关注)

> **type 列表示关联类型或访问类型，即 MySQL 决定如何查找表中的行，查找数据行记录的大概范围。**
> 依次从最优到最差分别为：system > const > eq_ref > ref > range > index > ALL ，一般来说，得保证查询达到 range 级别，最好达到 ref 级别。
> •	1）const、system：MySQL 能对查询的某部分进行优化并将其转化成一个常量(可以看 show warnings 的结果)。用于 primary key 或 unique key 的所有列与常数比较时，所以表最多有一个匹配行，读取1次，速度比较快。system是const的特例，表里只有一条数据匹配时为system
> •	2）eq_ref：primary key 或 unique key 索引的所有部分被连接使用 ，最多只会返回一条符合条件的记录。这可能是在 const 之外最好的联接类型了，简单的 select 查询不会出现这种 type。
> •	3）ref：相比 eq_ref，不使用唯一索引，而是使用普通索引或者联合索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行。
> •	4）range：范围扫描通常出现在 in，between，>，<，>=，<= 等操作中，使用一个索引来检索给定范围的行。
> •	5）index：扫描全索引就能拿到结果，一般是扫描某个二级索引或者联合索引，这种扫描不会从索引树根节点开始快速查找，而是直接对二级索引或者联合索引的叶子节点遍历和扫描，速度还是比较慢的，这种查询一般为使用 覆盖索引(索引覆盖)，二级索引一般比较小，所以这种通常比 ALL 快一些。
> •	6）ALL：即全表扫描，扫描你的聚簇索引的所有叶子节点。通常情况下这需要增加索引来进行优化了。

##### possible_keys

> possible_keys 列表示查询 可能使用 哪些索引来查找，但是最终查询可能不使用索引。
>
> explain 时可能出现 possible_keys 列有可以使用的索引，但是 key 列显示 NULL 的情况，这种情况是因为 MySQL 经过查询成本计算，MySQL 认为索引对此查询速度不如全表扫描，最终选择了全表查询。
> 如果该列是 NULL，则没有相关的索引。在这种情况下，可以通过检查 where 子句看是否可以创造一个适当的索引来提高查询性能，然后用 explain 查看效果。

##### key(重点关注)

> key 列表示 MySQL 实际使用 哪个索引来对该表进行查询。
> 如果没有使用索引，则该列是 NULL。如果想强制 MySQL 使用或忽视 possible_keys 列中的索引，在查询中使用 force index (强制走某个索引)、ignore index (强制不走某个索引)。

##### key_len(重点关注)

> key_ len 列表示 MySQL 在索引里使用的字节数，通过这个值可以算出具体使用了索引中的哪些列。
>
> key_len 计算原则
> 1、字段类型 year为1个，int为4个，date为3，datetime为4，char(n)为3n，varchar(n)为3n+2
> 2、如果字段可为 null，则需要额外再加1

##### ref

> ref 列表示在 key 列记录的索引中，表查找值所用到的列或常量，常见的有：const(常量)，字段名(例如film.id)。

##### rows(重点关注)

> rows 列表示 MySQL 估计要读取并检测的行数，需要注意的是，是估计值，并非最后结果集里的行数。

##### filtered(重点关注)

> 查询条件过滤的行数的百分比

##### Extra(重点关注)

> Extra 列表示的是一些额外信息，常见重要值如下：
> •	1）Using index：使用覆盖索引;
> •	2）Using where：使用 where 来处理结果,查询的列未被索引覆盖;
> •	3）Using index condition：查询的列不完全被索引覆盖，需要回表查询;
> •	4）Using temporary：需要创建一张临时表来处理查询，可用索引优化;
> •	5）Using filesort：将用外部排序而不是索引排序，Using filesort ，数据较小时从内存排序，
>  否则在磁盘排序，可用索引优化;
> •	6）Select tables optimized away：使用某些聚合函数(比如 max、min)来访问存在索引的某个字段时出现；

### 2.2 执行计划测试

#### 测试表与测试数据

##### 建表语句

```sql
CREATE TABLE `t_members` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增主键',
  `empl_no` varchar(20) DEFAULT '***' COMMENT '员工编号',
  `oa_account` varchar(24) NOT NULL DEFAULT '' COMMENT 'oa账户',
  `user_name` varchar(24) NOT NULL DEFAULT '***' COMMENT '姓名',
  `born_year` year NOT NULL DEFAULT '1996' COMMENT '出生年',
  `birth_date` date NOT NULL DEFAULT '1996-05-10' COMMENT '生日',
  `position` varchar(20) NOT NULL DEFAULT 'SDE' COMMENT '职位（SDE、SDM、PM、QA、SRE）',
  `entry_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
  `create_time` timestamp DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `idx_oa_born_year_position` (`oa_account`,`born_year`,`position`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='成员记录表';
```
##### 测试数据

```sql
## 一条insert
INSERT INTO `piccolo_currency`.`t_members` (`id`, `empl_no`, `oa_account`, `user_name`, `born_year`, `birth_date`, `position`, `entry_date`, `create_time`) VALUES (1, '123', 'dave', '***', 1996, '1996-05-10', 'SDE', '2021-07-05 10:04:26', '2022-07-23 04:04:26');


## 新增语句 此处我造了多个名类型 dave,mia,cathy,nora,carl,jay 约6w条数据，其中仅dave存在无编号的一行数据，其余name均带有编号
DROP PROCEDURE IF EXISTS proc_initData;
DELIMITER $  
CREATE PROCEDURE proc_initData()  
BEGIN  
    DECLARE i INT DEFAULT 1;  
    WHILE i<=10000 DO  
        INSERT INTO t_members(empl_no, oa_account, born_year, birth_date, position) VALUES ( CONCAT(123+i), CONCAT('dave',i),FLOOR(1990 + RAND() * 10), '1996-05-10', if(i % 5<3, if(i % 5<1,'SDE',if(i % 5<2,'SDM','QA')), if(i % 5<4,'PM','SRE')));
        SET i = i+1;  
    END WHILE;  
END $  
CALL proc_initData(); 

## 职位分支
if(i % 5<3, if(i % 5<1,'SDE',if(i % 5<2,'SDM','QA')), if(i % 5<4,'PM','SRE'))
```

##### 索引key_len

```sql
explain select * from t_members where oa_account = 'dave1' ;
explain select * from t_members where oa_account = 'dave1' and born_year = '1999' ;
explain select * from t_members where oa_account = 'dave1' and born_year = '1999' and position = 'SDE';
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引_key_len.png)

```
key_len 计算原则
1、字段类型 year为1个，int为4个，date为3，datetime为4，char(n)为3n，varchar(n)为3n+2
2、如果字段可为 null，则需要额外再加1

`oa_account`,`born_year`,`position` 类型分别为 varchar(24),year,varchar(20)其对应字节数24*3+2，1，20*3+2
```



#### trace工具数据库查询

> 这里用mysql的trace工具来看一看mysql执行计划的大致过程。

```sql
-- 开启trace
mysql> set session optimizer_trace="enabled=on", end_markers_in_json=on;
-- 执行SQL
mysql> select * from t_members where oa_account > 'dave1' and born_year != '1999' and position = 'SDE' order by id
-- 执行结果：7203 rows in set (0.06 sec)

-- 查询trace字段
mysql> SELECT * FROM information_schema.OPTIMIZER_TRACE;

-- 分析完后
-- 分析完记得关闭trace
mysql> set session optimizer_trace="enabled=off";
```



trace日志注解

```sql
select * from t_members where oa_account > 'dave1' and born_year != '1999' and position = 'SDE' order by id | {
  "steps": [
    {
      "join_preparation": {-- 第一阶段：SQL准备阶段，格式化SQL
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `t_members`.`id` AS `id`,`t_members`.`empl_no` AS `empl_no`,`t_members`.`oa_account` AS `oa_account`,`t_members`.`user_name` AS `user_name`,`t_members`.`born_year` AS `born_year`,`t_members`.`birth_date` AS `birth_date`,`t_members`.`position` AS `position`,`t_members`.`entry_date` AS `entry_date`,`t_members`.`create_time` AS `create_time` from `t_members` where ((`t_members`.`oa_account` > 'dave1') and (`t_members`.`born_year` <> 1999) and (`t_members`.`position` = 'SDE')) order by `t_members`.`id`"
          }
        ] /* steps */
      } /* join_preparation */
    },
    {
      "join_optimization": {-- 第二阶段：SQL优化阶段
        "select#": 1,
        "steps": [
          {
            "condition_processing": {-- 条件处理，where，1=1之类的SQL优化
              "condition": "WHERE",
              "original_condition": "((`t_members`.`oa_account` > 'dave1') and (`t_members`.`born_year` <> 1999) and (`t_members`.`position` = 'SDE'))",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "((`t_members`.`oa_account` > 'dave1') and (`t_members`.`born_year` <> 1999) and (`t_members`.`position` = 'SDE'))"
                },
                {
                  "transformation": "constant_propagation",
                  "resulting_condition": "((`t_members`.`oa_account` > 'dave1') and (`t_members`.`born_year` <> 1999) and (`t_members`.`position` = 'SDE'))"
                },
                {
                  "transformation": "trivial_condition_removal",
                  "resulting_condition": "((`t_members`.`oa_account` > 'dave1') and (`t_members`.`born_year` <> 1999) and (`t_members`.`position` = 'SDE'))"
                }
              ] /* steps */
            } /* condition_processing */
          },
          {
            "substitute_generated_columns": {
            } /* substitute_generated_columns */
          },
          {
            "table_dependencies": [-- 表依赖详情
              {
                "table": "`t_members`",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": [
                ] /* depends_on_map_bits */
              }
            ] /* table_dependencies */
          },
          {
            "ref_optimizer_key_uses": [
            ] /* ref_optimizer_key_uses */
          },
          {
            "rows_estimation": [    -- 预估表的访问成本
              {
                "table": "`t_members`",
                "range_analysis": {
                  "table_scan": {   -- 全表扫描情况
                    "rows": 60187,  -- 扫描行数，innodb中这个是预估值,MyISAM中时准确值
                    "cost": 12329   -- 查询成本
                  } /* table_scan */,
                  "potential_range_indexes": [   -- 查询可能使用的索引
                    {
                      "index": "PRIMARY",    -- 主键索引
                      "usable": false,
                      "cause": "not_applicable"
                    },
                    {
                      "index": "idx_oa_born_year_position",  -- 联合索引
                      "usable": true,
                      "key_parts": [
                        "oa_account",
                        "born_year",
                        "position",
                        "id"
                      ] /* key_parts */
                    }
                  ] /* potential_range_indexes */,
                  "setup_range_conditions": [
                  ] /* setup_range_conditions */,
                  "group_index_range": {
                    "chosen": false,
                    "cause": "not_group_by_or_distinct"
                  } /* group_index_range */,
                  "analyzing_range_alternatives": {      -- 分析各个索引使用成本
                    "range_scan_alternatives": [
                      {
                        "index": "idx_oa_born_year_position",
                        "ranges": [
                          "dave1 < oa_account"						-- 索引使用范围
                        ] /* ranges */,
                        "index_dives_for_eq_ranges": true,
                        "rowid_ordered": false,						-- 使用该索引获取的记录是否按照主键排序
                        "using_mrr": false,	-- 是否使用多范围读(MRR) , 作用为针对基于辅助/第二索引的查询，减少随机IO，并且将随机IO转化为顺序IO，提高查询效率。
                        "index_only": false,-- 是否使用覆盖索引
                        "rows": 30093,			-- 索引扫描行数，也是个预估值
                        "cost": 36113,			-- 索引使用成本
                        "chosen": false,		-- 是否选择索引
                        "cause": "cost"			-- 未选择索引原因选择成功则无cause
                      }
                    ] /* range_scan_alternatives */,
                    "analyzing_roworder_intersect": {
                      "usable": false,
                      "cause": "too_few_roworder_scans"
                    } /* analyzing_roworder_intersect */
                  } /* analyzing_range_alternatives */
                } /* range_analysis */
              }
            ] /* rows_estimation */
          },
          {
            "considered_execution_plans": [
              {
                "plan_prefix": [
                ] /* plan_prefix */,
                "table": "`t_members`",
                "best_access_path": {						-- 最优访问路径
                  "considered_access_paths": [	-- 最终访问路径
                    {
                      "rows_to_scan": 60187,
                      "access_type": "scan",		-- 访问类型 range：访问扫描；scan：全表扫
                      "resulting_rows": 60187,
                      "cost": 12326,
                      "chosen": true,
                      "use_tmp_table": true
                    }
                  ] /* considered_access_paths */
                } /* best_access_path */,
                "condition_filtering_pct": 100,
                "rows_for_plan": 60187,
                "cost_for_plan": 12326,
                "sort_cost": 60187,
                "new_cost_for_plan": 72513,
                "chosen": true
              }
            ] /* considered_execution_plans */
          },
          {
            "attaching_conditions_to_tables": {
              "original_condition": "((`t_members`.`oa_account` > 'dave1') and (`t_members`.`born_year` <> 1999) and (`t_members`.`position` = 'SDE'))",
              "attached_conditions_computation": [
              ] /* attached_conditions_computation */,
              "attached_conditions_summary": [
                {
                  "table": "`t_members`",
                  "attached": "((`t_members`.`oa_account` > 'dave1') and (`t_members`.`born_year` <> 1999) and (`t_members`.`position` = 'SDE'))"
                }
              ] /* attached_conditions_summary */
            } /* attaching_conditions_to_tables */
          },
          {
            "clause_processing": {
              "clause": "ORDER BY",
              "original_clause": "`t_members`.`id`",
              "items": [
                {
                  "item": "`t_members`.`id`"
                }
              ] /* items */,
              "resulting_clause_is_simple": true,
              "resulting_clause": "`t_members`.`id`"
            } /* clause_processing */
          },
          {
            "reconsidering_access_paths_for_index_ordering": {
              "clause": "ORDER BY",
              "steps": [
              ] /* steps */,
              "index_order_summary": {
                "table": "`t_members`",
                "index_provides_order": true,
                "order_direction": "asc",
                "index": "PRIMARY",
                "plan_changed": true,
                "access_type": "index"
              } /* index_order_summary */
            } /* reconsidering_access_paths_for_index_ordering */
          },
          {
            "refine_plan": [
              {
                "table": "`t_members`"
              }
            ] /* refine_plan */
          }
        ] /* steps */
      } /* join_optimization */
    },
    {
      "join_execution": {
        "select#": 1,
        "steps": [
        ] /* steps */
      } /* join_execution */
    }
  ] /* steps */
} 
```



从上述trace日志也可以发现，mysql的执行计划过程：`分析器 --> 优化器 --> 执行器`

#### 问题: mysql执行计划存在一个预估访问成本，那么优化器是否会选择不合适的索引或不走索引？
- 扫描行数
MySQL通过采样统计在执行SQL前获取到一个不那么精确的索引基数，影响到MySQL根据索引基数，来预估使用此索引的扫描行数。
索引创建，是一个建索引树状存储的过程，但索引删除是标记删除。优化器采用采样评估出现误差的原因也受索引的标记删除影响。

- 回表代价
MySQL对普通索引的检索后若还需要回主键拉取数据，则预估的回表代价也是考虑是否选择这一普通索引的重要原因。

MySQL 内部优化器会根据扫描行、回表次数、临时表使用情况、检索比例、是否排序、表大小等多个因素计算查询成本是否使用索引、使用哪个索引。

### 3.SQL索引优化指南

#### 3.1 优先全值匹配

```sql
explain select * from t_members where oa_account = 'dave1' ;
explain select * from t_members where oa_account = 'dave1' and born_year = '1999' ;
explain select * from t_members where oa_account = 'dave1' and born_year = '1999' and position = 'SDE';
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引_key_len.png)

尽可能全值匹配去走索引



#### 3.2 最左前缀原则

由于索引的树状存储，因而多列联合索引遵循最左匹配原则，使用过程无法跳过中间索引。

```sql
explain select * from t_members where oa_account = 'dave1' ;
explain select * from t_members where oa_account = 'dave1' and born_year = '1999' ;
explain select * from t_members where oa_account = 'dave1' and born_year = '1999' and position = 'SDE';

explain select * from t_members where oa_account = 'dave1' and position = 'SDE';
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引_最左匹配.png)

#### 3.3 不要在索引列上做操作/隐式类型转换

索引列上做计算、函数、(自动/手动)类型转换、隐式类型转换，会导致索引失效而转向全表扫描

```sql
explain select * from t_members where oa_account = 'dave1' and born_year = '1999' and position = 123; -- 123涉及int转varchar 
explain select * from t_members where oa_account = 'dave1' and born_year = 1999 and position = 'SDE'; -- 1999涉及int转year,但可以走索引
explain select * from t_members where oa_account = 'dave1' and born_year = CONCAT('199','9') and position = 'SDE'; -- 等号右侧使用concat函数但是可以走索引
explain select * from t_members where oa_account = 'dave1' and born_year = CONCAT('199','9') and SUBSTR(position,1,2) = 'SD'; -- 等号左侧使用SUBSTR函数导致无法走索引
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引_索引列操作.png)

#### 3.4 范围查询本身不一定能走索引，且范围查询右侧的列不走索引

##### 大于 小于

```sql
explain select * from t_members where oa_account = 'dave1' and born_year = '1999' and position = 'SDE'; 
explain select * from t_members where oa_account = 'dave1' and born_year > '1999' and position = 'SDE'; 
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引_范围查询_列走索引.png)

上图，语句2 position没走索引，born_year 进行范围查询成功 走索引

```sql
explain select * from t_members where oa_account > 'dave1' and born_year = '1999' and position = 'SDE'; 
explain select * from t_members where oa_account < 'dave1' and born_year > '1999' and position = 'SDE'; 
explain select * from t_members where oa_account > 'zhorah' and born_year = '1999' and position = 'SDE'; 
explain select * from t_members where oa_account < 'zhorah' and born_year > '1999' and position = 'SDE'; 
explain select * from t_members where oa_account > 'adele' and born_year = '1999' and position = 'SDE'; 
explain select * from t_members where oa_account < 'adele' and born_year > '1999' and position = 'SDE'; 
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引_范围查询_列不一定走索引.png)

现象
- `oa_account > 'dave1'`  与 `oa_account < 'dave1'` 不走索引
- `oa_accounte > 'zhorah'` 和 `oa_account < 'adele'`成功走了oa_account的索引

> 首先, 走不走索引由优化器对当前数据库内数据采样结果进行预估访问成本进行决定，即与数据库的数据分布。

String 排序是ascall码排序，oa_account < 'dave1'在优化器阶段可以理解为针对varchar的一个字典序判断校对。

oa_account < 'dave1'，当前数据库中存在1w条carl的数据，oa_account > 'dave1'，存在约4w条数据，且此时查询非索引覆盖情况存在回表，因此优化器判定dave1的查询结果集较大，回表成本太大导致mysql预估查询成本不如全表扫描来的快而不走索引，不如全表扫描。

zhorah与adele均不在数据库中，且字典序明显oa_accounte > 'zhorah' 和 oa_account < 'adele'查询的结果集较小，mysql判断走索引的查询成本更低，而dave的查询结果集较大，回表成本太大导致mysql预估查询成本不如全表扫描来的快而不走索引。

##### 'like'

```sql
explain select * from t_members where oa_account like '%%%zhorah' and born_year = '1999' and position = 'SDE'; 
explain select * from t_members where oa_account like 'zhorah%%%' and born_year = '1999' and position = 'SDE'; 
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引_like查询_列左匹配才走索引.png)

现象
- `like 'zhorah%%%' `走索引 且不影响走全匹配联合索引
- `like '%%zhorah'` 不走索引
like 做模糊时 支持 列前缀匹配，可正常走联合索引

##### 'in' 'not in'

```sql
explain select * from t_members where oa_account in ('dave','nora','cathy')  and born_year = '1999' and position = 'SDE'; 
explain select * from t_members where oa_account not in ('dave','nora','cathy')  and born_year = '1999' and position = 'SDE'; 
explain select * from t_members where oa_account ='dave' and born_year = '1999' and position not in ('SDE','SDM'); 
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引_in查询_in走notin不一定.png)



现象
- `in ('dave','nora','cathy')` 走索引 且不影响走全匹配联合索引
- `not in ('dave','nora','cathy')` 没走索引但not in 不一定走索引 要看预估结果集大小、select内容与索引字段关系等
基本原理，同查询成本，在索引未覆盖情况下，in 结果集较小，mysql预估走索引查询成本更低，而not in 结果集较大时又需要回表查询，优化器判定索引代价过大，不走索引，若结果集较小仍然可走索引，比如`not in ('SDE','SDM')`;。

#####  范围查询汇总

```sql

```



不一定走索引
- </>  （当前列在查询代价小时可走单列索引，其右侧失效）
- in （通常结果集都较小，其右仍可走完整联合索引）
- or
- like （左匹配时，其右仍可走完整联合索引）
- != （当前列可走索引，其右侧失效）
- not in （预估结果集较小时可走当前列）
- not exists
- is not null (当前列不走索引，对应联合索引其左侧列可走)

无法使用索引
- is null （整个联合索引会失效）

##### limit的使用往往可以“曲线救国“走索引

大小于 可救

```sql
explain select * from t_members where oa_account > 'dave1' and born_year = '1999' and position = 'SDE' limit 10; 
explain select * from t_members where oa_account < 'dave1' and born_year > '1999' and position = 'SDE' limit 10;  
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引-limit-大小于.png)

左模糊 like 救不回

```sql
explain select * from t_members where oa_account like '%%%dave1' and born_year = '1999' and position = 'SDE' limit 10;  
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引-limit-like.png)

is null 救不回

```sql
explain select * from t_members where oa_account is null and born_year = '1999' and position = 'SDE' limit 10;  
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引-limit-is null.png)

not in 视情况可救，救回 仅支持到当前列左侧与当前列

> 注：我的建表数据中其中仅dave存在无编号的一行数据，其余name均带有编号，因此not in ('dave','nora','cathy')  结果集几乎100%当前数据库，就算有limit也无法走索引，代价过大。

```sql
explain select * from t_members where oa_account not in ('dave','nora','cathy')  and born_year = '1999' and position = 'SDE' limit 10; 
explain select * from t_members where oa_account ='dave'  and born_year not in ('1999','1997') and position = 'SDE' limit 10; 
explain select * from t_members where oa_account ='dave' and born_year = '1999' and position not in ('SDE','SDM') limit 10; 
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引-limit-not in.png)

#### 3.5 尽可能覆盖索引，尽可能不用 select * 语句

使用select * 取出全部列，存在回表查询过程，会让优化器无法完成索引覆盖扫描这类优化，会影响优化器对执行计划的选择，也会增加网络带宽消耗，更会带来额外的I/O,内存和CPU消耗。

> 前文范围查询的例子，使用索引覆盖均oa_account 列能走索引。

```sql
explain select oa_account,born_year,position from t_members where oa_account > 'dave1' and born_year = '1999' and position = 'SDE'; 
explain select oa_account,born_year,position from t_members where oa_account < 'dave1' and born_year > '1999' and position = 'SDE'; 
explain select oa_account,born_year,position from t_members where oa_account > 'zhorah' and born_year = '1999' and position = 'SDE'; 
explain select oa_account,born_year,position from t_members where oa_account < 'zhorah' and born_year > '1999' and position = 'SDE'; 
explain select oa_account,born_year,position from t_members where oa_account > 'adele' and born_year = '1999' and position = 'SDE'; 
explain select oa_account,born_year,position from t_members where oa_account < 'adele' and born_year > '1999' and position = 'SDE'; 
```

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引_索引覆盖情况_范围查询就可以走完索引.png)

区别于count(*)
•	对于count(1)和count(*)，MySQL的优化是完全一样的，反正都比count(字段)快就完了。

### 4.索引设计原则

#### 4.1 合理设计数据表字段

> 万物起于初，数据库索引设计的前端，数据表字段设计必须合理。

`数据表字段类型的合理设计，即选择合适字段类型和设置合适的长度。`
选择正确的数据类型与长度，那么在字段上建立索引时，一个数据页可以存储更多的索引，一次读取加载到内存的索引个数更多，同时降低B+tree的高度，减少磁盘IO，更有利于提升MySQL的性能。

- Integer 适合建立索引,主键也不宜过长；
- 字段默认值 null的选择与否
- 过长字段 可考虑前缀索引
  假设想用身份证号码快速查询数据库对应信息，直接建立身份证号主键，这其实是个不太合理的索引建立，主键存储过长，可以身份证号前缀建立索引。

#### 4.2 联合索引尽量覆盖条件/优化索引数量

> 覆盖索引(索引覆盖)定义：首先得说明的是覆盖索引不是一种索引类型，而是二级索引或者联合索引就包含所需要查询的所有字段，不需要再 回表 进行查询数据行获取其它字段值，这种情况一般可以说是用到了覆盖索引。

除唯一索引外，尽量少建立单值索引，应当设计联合索引，可优化索引数量，且尽量覆盖条件，让联合索引尽量包含SQL语句中的where、order by、group by的字段，同时确保联合索引的字段顺序尽量满足SQL查询的最左前缀原则。

**注意：对于业务场景的查询需求不那么明确和精确时，单值索引的建立是保守有效的办法。**

#### 4.3 where与order by：冲突时优先where

**在where和order by出现索引设计冲突时，是优先针对where去设计索引？还是优先针对order by设计索引？**
通常情况下都是优先针对where来设计索引，因为通常情况下都是先where条件使用索引快速筛选出来符合条件的数据，然后对进行筛选出来的数据进行排序和分组，而where条件快速筛选出来的的数据往往不会很多。

#### 4.4 优化慢查询SQL

生产环境，或者测试环境大数据量测试过程中开启慢查询日志后若发现慢查询SQL，需根据业务场景进行特定的索引优化、代码优化等策略。

#### 4.5 小表驱动大表：in 与 exsits

in是把外表和内表作hash 连接，而exists 是对外表作loop 循环，每次loop 循环再对内表进行查询。

```sql
-- in的话是先执行括号里面的，所以适应情况是B表数据小
select * from A where id in (select id from B)                    -- 使用了a表id列索引
-- exists是先执行括号外面的，所以适应情况是A表数据小
select * from A where exists (select 1 from B where B.id = A.id)  -- 使用了b表id列索引

-- 对数据规模大的表用索引快速筛选，可以提高效率。
-- EXISTS (subquery) 只进行二值判定，只返回TRUE或FALSE，因此子查询中用SELECT * 可用SELECT 1替换，官方说法是实际执行时会忽略SELECT清单，因此没有区别。（EXISTS子查询的实际执行过程存在优化）

## 效率对比
-- A、B规模相当，in exsits 效率相当；
-- A表规模大于B ，in效率高；
-- A表规模小于B ，exists效率高；
```

备注：not in 与 not exists 

> not exists 效率优于 not in 

- 查询语句使用了not in，内外表都进行全表扫描，没有用到索引；

  > 有别于前文的not in，这里的not in(subquery) ，subquery为需要执行的子查询；前文探讨的not in(set)，set为固定集合，内部优化器可根据索引页采样进行估算代价进而决定是否选择索引。

- not extsts 的子查询依然能用到表上的索引；

```sql
-- t_members_myisam与t_members数据结构一致，为myisam引擎，仅存放一条数据。
select * from t_members where id in (select id from t_members_myisam);
select * from t_members where id not in (select id from t_members_myisam);
```



![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/explain-索引优化-not-in 子查询内外都不走索引.png)

