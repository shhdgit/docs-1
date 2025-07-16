---
title: RECOVER TABLE
summary: 关于在 TiDB 数据库中使用 RECOVER TABLE 的概述。
---

# RECOVER TABLE

`RECOVER TABLE` 用于在执行 `DROP TABLE` 语句后，恢复已删除的表及其上的数据，前提是仍在 GC（垃圾回收）存活期内。

## 语法

```sql
RECOVER TABLE table_name;
```

```sql
RECOVER TABLE BY JOB JOB_ID;
```

## 概述

```ebnf+diagram
RecoverTableStmt ::=
    'RECOVER' 'TABLE' ( 'BY' 'JOB' Int64Num | TableName Int64Num? )

TableName ::=
    Identifier ( '.' Identifier )?

Int64Num ::= NUM

NUM ::= intLit
```

> **注意：**
>
> 如果表已被删除且超出 GC 存活期，则无法使用 `RECOVER TABLE` 进行恢复。在这种情况下执行 `RECOVER TABLE` 会返回类似以下的错误：`snapshot is older than GC safe point 2019-07-10 13:45:57 +0800 CST`。

## 示例

+ 根据表名恢复已删除的表。

    
    ```sql
    DROP TABLE t;
    ```

    
    ```sql
    RECOVER TABLE t;
    ```

    该方法会搜索最近的 DDL 任务历史，找到第一个类型为 `DROP TABLE` 的 DDL 操作，然后恢复与 `RECOVER TABLE` 语句中指定的表名相同的已删除表。

+ 根据表的 `DDL JOB ID` 进行恢复。

    假设你删除了表 `t`，又创建了另一个 `t`，之后再次删除了新创建的 `t`。如果你想恢复最初删除的 `t`，必须使用指定 `DDL JOB ID` 的方法。

    
    ```sql
    DROP TABLE t;
    ```

    
    ```sql
    ADMIN SHOW DDL JOBS 1;
    ```

    上述第二条语句用于查找删除 `t` 的 `DDL JOB ID`。在下面的示例中，ID 为 `53`。

    ```
    +--------+---------+------------+------------+--------------+-----------+----------+-----------+-----------------------------------+--------+
    | JOB_ID | DB_NAME | TABLE_NAME | JOB_TYPE   | SCHEMA_STATE | SCHEMA_ID | TABLE_ID | ROW_COUNT | START_TIME                        | STATE  |
    +--------+---------+------------+------------+--------------+-----------+----------+-----------+-----------------------------------+--------+
    | 53     | test    |            | drop table | none         | 1         | 41       | 0         | 2019-07-10 13:23:18.277 +0800 CST | synced |
    +--------+---------+------------+------------+--------------+-----------+----------+-----------+-----------------------------------+--------+
    ```

    
    ```sql
    RECOVER TABLE BY JOB 53;
    ```

    该方法通过 `DDL JOB ID` 恢复已删除的表。如果对应的 DDL 任务不是 `DROP TABLE` 类型，则会报错。

## 实现原理

在删除表时，TiDB 只会删除表的元数据，并将待删除的表数据（行数据和索引数据）写入 `mysql.gc_delete_range` 表。TiDB 后台的 GC Worker 会定期从 `mysql.gc_delete_range` 表中删除超出 GC 存活期的键。

因此，要恢复一张表，只需恢复表的元数据，并在 GC Worker 删除表数据之前，删除 `mysql.gc_delete_range` 表中对应的行记录。你可以使用 TiDB 的快照读取功能来恢复表的元数据。详细信息请参考 [Read Historical Data](/read-historical-data.md)。

表的恢复是通过 TiDB 通过快照读取获取表元数据，然后进行类似 `CREATE TABLE` 的表创建过程完成的。因此，`RECOVER TABLE` 本质上也是一种 DDL 操作。

## MySQL 兼容性

该语句是 TiDB 对 MySQL 语法的扩展。