---
title: EXPLAIN | TiDB SQL Statement Reference
summary: An overview of the usage of EXPLAIN for the TiDB database.
---

# `EXPLAIN` {#explain}

`EXPLAIN`ステートメントは、クエリの実行計画を実行せずに表示します。`EXPLAIN ANALYZE`と組み合わせて使用すると、クエリを実行します。`EXPLAIN`の出力が期待される結果と一致しない場合は、クエリ内の各テーブルで`ANALYZE TABLE`を実行することを検討してください。

ステートメント`DESC`および`DESCRIBE`は、このステートメントのエイリアスです。`EXPLAIN <tableName>`の代替使用法については、[`SHOW [FULL] COLUMNS FROM`](/sql-statements/sql-statement-show-columns-from.md)で文書化されています。

TiDBは`EXPLAIN [options] FOR CONNECTION connection_id`ステートメントをサポートしています。ただし、このステートメントはMySQLの`EXPLAIN FOR`ステートメントとは異なります。詳細については、[`EXPLAIN FOR CONNECTION`](#explain-for-connection)を参照してください。

## 概要 {#synopsis}

```ebnf+diagram
ExplainSym ::=
    'EXPLAIN'
|   'DESCRIBE'
|   'DESC'

ExplainStmt ::=
    ExplainSym ( TableName ColumnName? | 'ANALYZE'? ExplainableStmt | 'FOR' 'CONNECTION' NUM | 'FORMAT' '=' ( stringLit | ExplainFormatType ) ( 'FOR' 'CONNECTION' NUM | ExplainableStmt ) )

ExplainableStmt ::=
    SelectStmt
|   DeleteFromStmt
|   UpdateStmt
|   InsertIntoStmt
|   ReplaceIntoStmt
|   UnionStmt
```

## `EXPLAIN`の出力形式 {#explain-output-format}

> **Note:**
>
> TiDBに接続するためにMySQLクライアントを使用すると、ラップされた行なしで出力結果をより明確に読むために、`pager less -S`コマンドを使用できます。その後、`EXPLAIN`の結果が出力されたら、キーボードの右矢印<kbd>→</kbd>ボタンを押して、出力を水平にスクロールすることができます。

> **Note:**
>
> 返された実行計画では、`IndexJoin`および`Apply`演算子のすべてのプローブ側の子ノードについて、v6.4.0以降の`estRows`の意味は、v6.4.0より前とは異なります。詳細は[TiDBクエリ実行計画の概要](/explain-overview.md#understand-explain-output)を参照してください。

現在、TiDBの`EXPLAIN`は5つのカラムを出力します：`id`、`estRows`、`task`、`access object`、`operator info`。実行計画内の各演算子は、これらの属性によって説明され、`EXPLAIN`の出力内の各行が演算子を説明しています。各属性の説明は次のとおりです：

| 属性名           | 説明                                                                                                                                                                                                                                                                                    |
| :------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| id            | 演算子IDは、実行計画全体での演算子の一意の識別子です。TiDB 2.1では、IDは演算子のツリー構造を表示するようにフォーマットされています。データは子ノードから親ノードに流れます。各演算子には1つだけ親ノードがあります。                                                                                                                                                                      |
| estRows       | 演算子が出力すると予想される行数。この数値は統計情報と演算子の論理に基づいて推定されます。`estRows`は、TiDB 4.0の以前のバージョンでは`count`と呼ばれています。                                                                                                                                                                                           |
| task          | 演算子が属するタスクの種類。現在、実行計画は2つのタスクに分かれています：**root**タスク（tidb-serverで実行される）と**cop**タスク（TiKVまたはTiFlashで並列に実行される）。タスクレベルでの実行計画のトポロジーは、rootタスクの後に多くのcopタスクが続く形です。rootタスクはcopタスクの出力を入力として使用します。copタスクは、TiDBがTiKVまたはTiFlashにプッシュダウンするタスクを指します。各copタスクは、TiKVクラスタまたはTiFlashクラスタに分散され、複数のプロセスで実行されます。 |
| access object | 演算子によってアクセスされるデータアイテム情報。情報には`table`、`partition`、および（あれば）`index`が含まれます。データに直接アクセスする演算子のみがこのような情報を持っています。                                                                                                                                                                               |
| operator info | 演算子に関するその他の情報。各演算子の`operator info`は異なります。以下の例を参照してください。                                                                                                                                                                                                                               |

## Examples {#examples}

```sql
EXPLAIN SELECT 1;
```

```sql
+-------------------+---------+------+---------------+---------------+
| id                | estRows | task | access object | operator info |
+-------------------+---------+------+---------------+---------------+
| Projection_3      | 1.00    | root |               | 1->Column#1   |
| └─TableDual_4     | 1.00    | root |               | rows:1        |
+-------------------+---------+------+---------------+---------------+
2 rows in set (0.00 sec)
```

```sql
CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
```

```sql
Query OK, 0 rows affected (0.10 sec)
```

```sql
INSERT INTO t1 (c1) VALUES (1), (2), (3);
```

```sql
Query OK, 3 rows affected (0.02 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

```sql
EXPLAIN SELECT * FROM t1 WHERE id = 1;
```

```sql
+-------------+---------+------+---------------+---------------+
| id          | estRows | task | access object | operator info |
+-------------+---------+------+---------------+---------------+
| Point_Get_1 | 1.00    | root | table:t1      | handle:1      |
+-------------+---------+------+---------------+---------------+
1 row in set (0.00 sec)
```

```sql
DESC SELECT * FROM t1 WHERE id = 1;
```

```sql
+-------------+---------+------+---------------+---------------+
| id          | estRows | task | access object | operator info |
+-------------+---------+------+---------------+---------------+
| Point_Get_1 | 1.00    | root | table:t1      | handle:1      |
+-------------+---------+------+---------------+---------------+
1 row in set (0.00 sec)
```

```sql
DESCRIBE SELECT * FROM t1 WHERE id = 1;
```

```sql
+-------------+---------+------+---------------+---------------+
| id          | estRows | task | access object | operator info |
+-------------+---------+------+---------------+---------------+
| Point_Get_1 | 1.00    | root | table:t1      | handle:1      |
+-------------+---------+------+---------------+---------------+
1 row in set (0.00 sec)
```

```sql
EXPLAIN INSERT INTO t1 (c1) VALUES (4);
```

```sql
+----------+---------+------+---------------+---------------+
| id       | estRows | task | access object | operator info |
+----------+---------+------+---------------+---------------+
| Insert_1 | N/A     | root |               | N/A           |
+----------+---------+------+---------------+---------------+
1 row in set (0.00 sec)
```

```sql
EXPLAIN UPDATE t1 SET c1=5 WHERE c1=3;
```

```sql
+---------------------------+---------+-----------+---------------+--------------------------------+
| id                        | estRows | task      | access object | operator info                  |
+---------------------------+---------+-----------+---------------+--------------------------------+
| Update_4                  | N/A     | root      |               | N/A                            |
| └─TableReader_8           | 0.00    | root      |               | data:Selection_7               |
|   └─Selection_7           | 0.00    | cop[tikv] |               | eq(test.t1.c1, 3)              |
|     └─TableFullScan_6     | 3.00    | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+---------------------------+---------+-----------+---------------+--------------------------------+
4 rows in set (0.00 sec)
```

```sql
EXPLAIN DELETE FROM t1 WHERE c1=3;
```

```sql
+---------------------------+---------+-----------+---------------+--------------------------------+
| id                        | estRows | task      | access object | operator info                  |
+---------------------------+---------+-----------+---------------+--------------------------------+
| Delete_4                  | N/A     | root      |               | N/A                            |
| └─TableReader_8           | 0.00    | root      |               | data:Selection_7               |
|   └─Selection_7           | 0.00    | cop[tikv] |               | eq(test.t1.c1, 3)              |
|     └─TableFullScan_6     | 3.00    | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+---------------------------+---------+-----------+---------------+--------------------------------+
4 rows in set (0.01 sec)
```

指定`EXPLAIN`输出的格式，可以使用`FORMAT = xxx`语法。目前，TiDB支持以下格式：

| FORMAT       | Description                                                                                                 |
| ------------ | ----------------------------------------------------------------------------------------------------------- |
| 未指定          | 如果未指定格式，`EXPLAIN`将使用默认格式`row`。                                                                              |
| `brief`      | `EXPLAIN`语句输出的操作符ID与未指定`FORMAT`时相比，简化了。                                                                     |
| `dot`        | `EXPLAIN`语句输出DOT执行计划，可以通过`dot`程序（在`graphviz`包中）生成PNG文件。                                                     |
| `row`        | `EXPLAIN`语句以表格格式输出结果。有关更多信息，请参阅[了解查询执行计划](/explain-overview.md)。                                            |
| `tidb_json`  | `EXPLAIN`语句以JSON格式输出执行计划，并将操作符信息存储在JSON数组中。                                                                 |
| `verbose`    | `EXPLAIN`语句以`row`格式输出结果，并在结果中添加一个`estCost`列，用于查询的估计成本。有关如何使用此格式的更多信息，请参阅[SQL计划管理](/sql-plan-management.md)。 |
| `plan_cache` | `EXPLAIN`语句以`row`格式输出结果，并将[计划缓存](/sql-non-prepared-plan-cache.md#diagnostics)信息作为警告。                        |

<SimpleTab>

<div label="brief">

以下是`EXPLAIN`中`FORMAT`为`"brief"`时的示例：

```sql
EXPLAIN FORMAT = "brief" DELETE FROM t1 WHERE c1 = 3;
```

```sql
+-------------------------+---------+-----------+---------------+--------------------------------+
| id                      | estRows | task      | access object | operator info                  |
+-------------------------+---------+-----------+---------------+--------------------------------+
| Delete                  | N/A     | root      |               | N/A                            |
| └─TableReader           | 0.00    | root      |               | data:Selection                 |
|   └─Selection           | 0.00    | cop[tikv] |               | eq(test.t1.c1, 3)              |
|     └─TableFullScan     | 3.00    | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+-------------------------+---------+-----------+---------------+--------------------------------+
4 rows in set (0.001 sec)
```

</div>

<div label="DotGraph">

MySQLの標準結果形式に加えて、TiDBはDotGraphもサポートしており、次の例のように`FORMAT = "dot"`を指定する必要があります。**Note:**

```sql
CREATE TABLE t(a bigint, b bigint);
EXPLAIN format = "dot" SELECT A.a, B.b FROM t A JOIN t B ON A.a > B.b WHERE A.a < 10;
```

```sql
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| dot contents                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|
digraph Projection_8 {
subgraph cluster8{
node [style=filled, color=lightgrey]
color=black
label = "root"
"Projection_8" -> "HashJoin_9"
"HashJoin_9" -> "TableReader_13"
"HashJoin_9" -> "Selection_14"
"Selection_14" -> "TableReader_17"
}
subgraph cluster12{
node [style=filled, color=lightgrey]
color=black
label = "cop"
"Selection_12" -> "TableFullScan_11"
}
subgraph cluster16{
node [style=filled, color=lightgrey]
color=black
label = "cop"
"Selection_16" -> "TableFullScan_15"
}
"TableReader_13" -> "Selection_12"
"TableReader_17" -> "Selection_16"
}
 |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

もしコンピューターに`dot`プログラムがある場合、次の方法を使用してPNGファイルを生成できます：

```bash
dot xx.dot -T png -O

The xx.dot is the result returned by the above statement.
```

If your computer has no `dot` program, copy the result to [this website](http://www.webgraphviz.com/) to get a tree diagram:

![EXPLAIN Dot](/media/explain_dot.png)

</div>

<div label="JSON">

To get the output in JSON, specify `FORMAT = "tidb_json"` in the `EXPLAIN` statement. The following is an example:

```sql
CREATE TABLE t(id int primary key, a int, b int, key(a));
EXPLAIN FORMAT = "tidb_json" SELECT id FROM t WHERE a = 1;
```

```
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| TiDB_JSON                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| [
    {
        "id": "Projection_4",
        "estRows": "10.00",
        "taskType": "root",
        "operatorInfo": "test.t.id",
        "subOperators": [
            {
                "id": "IndexReader_6",
                "estRows": "10.00",
                "taskType": "root",
                "operatorInfo": "index:IndexRangeScan_5",
                "subOperators": [
                    {
                        "id": "IndexRangeScan_5",
                        "estRows": "10.00",
                        "taskType": "cop[tikv]",
                        "accessObject": "table:t, index:a(a)",
                        "operatorInfo": "range:[1,1], keep order:false, stats:pseudo"
                    }
                ]
            }
        ]
    }
]
 |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```

以下は、翻訳されたテキストです。

出力では、`id`、`estRows`、`taskType`、`accessObject`、および`operatorInfo`は、デフォルトのフォーマットのカラムと同じ意味を持ちます。 `subOperators`は、サブノードを格納する配列です。 サブノードのフィールドと意味は親ノードと同じです。 フィールドが欠落している場合、そのフィールドは空であることを意味します。

</div>

</SimpleTab>

## MySQL互換性 {#mysql-compatibility}

- `EXPLAIN`のフォーマットとTiDBでの潜在的な実行計画の両方がMySQLと大きく異なります。
- TiDBは`FORMAT=JSON`または`FORMAT=TREE`オプションをサポートしていません。
- TiDBの`FORMAT=tidb_json`は、デフォルトの`EXPLAIN`結果のJSON形式の出力です。フォーマットとフィールドはMySQLの`FORMAT=JSON`出力とは異なります。

### `EXPLAIN FOR CONNECTION` {#explain-for-connection}

`EXPLAIN FOR CONNECTION`は、現在実行中のSQLクエリまたは接続で最後に実行されたSQLクエリの実行計画を取得するために使用されます。出力形式は`EXPLAIN`と同じです。ただし、TiDBでの`EXPLAIN FOR CONNECTION`の実装はMySQLとは異なります。出力形式以外の違いは次のとおりです。

- 接続がスリープ状態の場合、MySQLは空の結果を返しますが、TiDBは最後に実行されたクエリプランを返します。
- 現在のセッションの実行計画を取得しようとすると、MySQLはエラーを返しますが、TiDBは通常結果を返します。
- MySQLでは、ログインユーザーはクエリされている接続と同じであるか、ログインユーザーには\*\*`PROCESS`**権限が必要です。一方、TiDBでは、ログインユーザーはクエリされている接続と同じであるか、ログインユーザーには**`SUPER`\*\*権限が必要です。

## 関連項目 {#see-also}

- [クエリ実行計画の理解](/explain-overview.md)
- [EXPLAIN ANALYZE](/sql-statements/sql-statement-explain-analyze.md)
- [ANALYZE TABLE](/sql-statements/sql-statement-analyze-table.md)
- [TRACE](/sql-statements/sql-statement-trace.md)"
