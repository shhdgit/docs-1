---
title: SHOW TABLE REGIONS
summary: 学习如何在 TiDB 中使用 SHOW TABLE REGIONS。
---

# SHOW TABLE REGIONS

`SHOW TABLE REGIONS` 语句用于显示 TiDB 中某个表的 Region 信息。

> **Note:**
>
> 该功能在 [{{{ .starter }}}](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-cloud-serverless) 集群上不可用。

## 语法

```sql
SHOW TABLE [table_name] REGIONS [WhereClauseOptional];
SHOW TABLE [table_name] INDEX [index_name] REGIONS [WhereClauseOptional];
```

## 概述

```ebnf+diagram
ShowTableRegionStmt ::=
    "SHOW" "TABLE" TableName PartitionNameList? ("INDEX" IndexName)? "REGIONS" ("WHERE" Expression)?

TableName ::=
    (SchemaName ".")? Identifier
```

执行 `SHOW TABLE REGIONS` 后返回以下列：

* `REGION_ID`：Region ID。
* `START_KEY`：Region 的起始键。
* `END_KEY`：Region 的结束键。
* `LEADER_ID`：Region 的 Leader ID。
* `LEADER_STORE_ID`：Region Leader 所在的存储（TiKV）ID。
* `PEERS`：所有 Region 副本的 ID。
* `SCATTERING`：Region 是否正在调度。`1` 表示是。
* `WRITTEN_BYTES`：在一次心跳周期内估算写入 Region 的数据量，单位为字节。
* `READ_BYTES`：在一次心跳周期内估算从 Region 读取的数据量，单位为字节。
* `APPROXIMATE_SIZE(MB)`：Region 中估算的数据量，单位为兆字节（MB）。
* `APPROXIMATE_KEYS`：Region 中估算的 Keys 数量。

<CustomContent platform="tidb">

* `SCHEDULING_CONSTRAINTS`：与该 Region 所属表或分区相关联的 [放置策略设置](/placement-rules-in-sql.md)。

</CustomContent>

<CustomContent platform="tidb-cloud">

* `SCHEDULING_CONSTRAINTS`：与该 Region 所属表或分区相关联的放置策略设置。

</CustomContent>

* `SCHEDULING_STATE`：具有放置策略的 Region 的调度状态。

> **Note:**
>
> `WRITTEN_BYTES`、`READ_BYTES`、`APPROXIMATE_SIZE(MB)`、`APPROXIMATE_KEYS` 的值不是精确数据，而是根据 Region 从 PD 接收心跳信息后估算得出的数据。

## 示例

创建一个数据量足够填充多个 Region 的示例表：

```sql
CREATE TABLE t1 (
 id INT NOT NULL PRIMARY KEY auto_increment,
 b INT NOT NULL,
 pad1 VARBINARY(1024),
 pad2 VARBINARY(1024),
 pad3 VARBINARY(1024)
);
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM dual;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
SELECT SLEEP(5);
SHOW TABLE t1 REGIONS;
```

输出应显示该表被拆分成多个 Region。`REGION_ID`、`START_KEY` 和 `END_KEY` 可能不会完全匹配：

```sql
...
mysql> SHOW TABLE t1 REGIONS;
+-----------+--------------+--------------+-----------+-----------------+-------+------------+---------------+------------+----------------------+------------------+------------------------+------------------+
| REGION_ID | START_KEY    | END_KEY      | LEADER_ID | LEADER_STORE_ID | PEERS | SCATTERING | WRITTEN_BYTES | READ_BYTES | APPROXIMATE_SIZE(MB) | APPROXIMATE_KEYS | SCHEDULING_CONSTRAINTS | SCHEDULING_STATE |
+-----------+--------------+--------------+-----------+-----------------+-------+------------+---------------+------------+----------------------+------------------+------------------------+------------------+
|        94 | t_75_        | t_75_r_31717 |        95 |               1 | 95    |          0 |             0 |          0 |                  112 |           207465 |                        |                  |
|        96 | t_75_r_31717 | t_75_r_63434 |        97 |               1 | 97    |          0 |             0 |          0 |                   97 |                0 |                        |                  |
|         2 | t_75_r_63434 |              |         3 |               1 | 3     |          0 |     269323514 |   66346110 |                  245 |           162020 |                        |                  |
+-----------+--------------+--------------+-----------+-----------------+-------+------------+---------------+------------+----------------------+------------------+------------------------+------------------+
```

在上面输出中，`START_KEY` 为 `t_75_r_31717` 和 `END_KEY` 为 `t_75_r_63434` 表示在这个范围内的主键数据存储在这个 Region 中。前缀 `t_75_` 表示这是表（`t`）的 Region，内部表 ID 为 `75`。空的 `START_KEY` 或 `END_KEY` 表示负无穷或正无穷。

TiDB 会根据需要自动重新平衡 Regions。若要手动重新平衡，可以使用 `SPLIT TABLE REGION` 语句：

```sql
mysql> SPLIT TABLE t1 BETWEEN (31717) AND (63434) REGIONS 2;
+--------------------+----------------------+
| TOTAL_SPLIT_REGION | SCATTER_FINISH_RATIO |
+--------------------+----------------------+
|                  1 |                    1 |
+--------------------+----------------------+
1 行结果（42.34 秒）

mysql> SHOW TABLE t1 REGIONS;
+-----------+--------------+--------------+-----------+-----------------+-------+------------+---------------+------------+----------------------+------------------+------------------------+------------------+
| REGION_ID | START_KEY    | END_KEY      | LEADER_ID | LEADER_STORE_ID | PEERS | SCATTERING | WRITTEN_BYTES | READ_BYTES | APPROXIMATE_SIZE(MB) | APPROXIMATE_KEYS | SCHEDULING_CONSTRAINTS | SCHEDULING_STATE |
+-----------+--------------+--------------+-----------+-----------------+-------+------------+---------------+------------+----------------------+------------------+------------------------+------------------+
|        94 | t_75_        | t_75_r_31717 |        95 |               1 | 95    |          0 |             0 |          0 |                  112 |           207465 |                        |                  |
|        98 | t_75_r_31717 | t_75_r_47575 |        99 |               1 | 99    |          0 |          1325 |          0 |                   53 |            12052 |                        |                  |
|        96 | t_75_r_47575 | t_75_r_63434 |        97 |               1 | 97    |          0 |          1526 |          0 |                   48 |                0 |                        |                  |
|         2 | t_75_r_63434 |              |         3 |               1 | 3     |          0 |             0 |   55752049 |                   60 |                0 |                        |                  |
+-----------+--------------+--------------+-----------+-----------------+-------+------------+---------------+------------+----------------------+------------------+------------------------+------------------+
```

上述输出显示，Region 96 被拆分，生成了新的 Region 98。表中的其他 Region 未受拆分操作影响。统计信息确认了这一点：

* `TOTAL_SPLIT_REGION` 表示新拆分的 Region 数量，此例为 1。
* `SCATTER_FINISH_RATIO` 表示新拆分的 Region 成功散布的比率，`1.0` 表示全部散布完成。

更详细的示例：

```sql
mysql> SHOW TABLE t REGIONS;
+-----------+--------------+--------------+-----------+-----------------+---------------+------------+---------------+------------+----------------------+------------------+------------------------+------------------+
| REGION_ID | START_KEY    | END_KEY      | LEADER_ID | LEADER_STORE_ID | PEERS         | SCATTERING | WRITTEN_BYTES | READ_BYTES | APPROXIMATE_SIZE(MB) | APPROXIMATE_KEYS | SCHEDULING_CONSTRAINTS | SCHEDULING_STATE |
+-----------+--------------+--------------+-----------+-----------------+---------------+------------+---------------+------------+----------------------+------------------+------------------------+------------------+
| 102       | t_43_r       | t_43_r_20000 | 118       | 7               | 105, 118, 119 | 0          | 0             | 0          | 1                    | 0                |                        |                  |
| 106       | t_43_r_20000 | t_43_r_40000 | 120       | 7               | 107, 108, 120 | 0          | 23            | 0          | 1                    | 0                |                        |                  |
| 110       | t_43_r_40000 | t_43_r_60000 | 112       | 9               | 112, 113, 121 | 0          | 0             | 0          | 1                    | 0                |                        |                  |
| 114       | t_43_r_60000 | t_43_r_80000 | 122       | 7               | 115, 122, 123 | 0          | 35            | 0          | 1                    | 0                |                        |                  |
| 3         | t_43_r_80000 |              | 93        | 8               | 5, 73, 93     | 0          | 0             | 0          | 1                    | 0                |                        |                  |
| 98        | t_43_        | t_43_r       | 99        | 1               | 99, 100, 101  | 0          | 0             | 0          | 1                    | 0                |                        |                  |
+-----------+--------------+--------------+-----------+-----------------+---------------+------------+---------------+------------+----------------------+------------------+------------------------+------------------+
```

## MySQL 兼容性

此语句为 TiDB 对 MySQL 语法的扩展。

## 相关链接

* [SPLIT REGION](/sql-statements/sql-statement-split-region.md)
* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)