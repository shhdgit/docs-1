---
title: FOREIGN KEY Constraints
summary: An overview of the usage of FOREIGN KEY constraints for the TiDB database.
---

# 外部キー制約 {#foreign-key-constraints}

v6.6.0から、TiDBは外部キー機能をサポートしており、関連するデータのクロステーブル参照やデータ整合性の維持のための外部キー制約を可能にします。

> **警告:**
>
> - 現在、外部キー機能は実験的なものです。本番環境で使用することは推奨されません。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を報告することができます。
> - 外部キー機能は通常、[参照整合性](https://en.wikipedia.org/wiki/Referential_integrity)制約チェックを強制するために使用されます。パフォーマンスの低下を引き起こす可能性があるため、パフォーマンスに影響を与えるシナリオで使用する前に徹底的なテストを行うことを推奨します。

外部キーは子テーブルで定義されます。構文は以下の通りです:

```ebnf+diagram
ForeignKeyDef
         ::= ( 'CONSTRAINT' Identifier )? 'FOREIGN' 'KEY'
             Identifier? '(' ColumnName ( ',' ColumnName )* ')'
             'REFERENCES' TableName '(' ColumnName ( ',' ColumnName )* ')'
             ( 'ON' 'DELETE' ReferenceOption )?
             ( 'ON' 'UPDATE' ReferenceOption )?

ReferenceOption
         ::= 'RESTRICT'
           | 'CASCADE'
           | 'SET' 'NULL'
           | 'SET' 'DEFAULT'
           | 'NO' 'ACTION'
```

## 命名 {#naming}

外部キーの命名は以下のルールに従います：

- `CONSTRAINT identifier` で名前が指定されている場合、指定された名前が使用されます。
- `CONSTRAINT identifier` で名前が指定されていないが、`FOREIGN KEY identifier` で名前が指定されている場合、`FOREIGN KEY identifier` で指定された名前が使用されます。
- `CONSTRAINT identifier` と `FOREIGN KEY identifier` のどちらも名前が指定されていない場合、`fk_1`、`fk_2`、`fk_3` などのように自動的に名前が生成されます。
- 外部キーの名前は現在のテーブルで一意である必要があります。そうでない場合、外部キーが作成されるときに `ERROR 1826: Duplicate foreign key constraint name 'fk'` のエラーが報告されます。

## 制限 {#restrictions}

外部キーを作成する際には、以下の条件を満たす必要があります：

- 親テーブルも子テーブルも一時テーブルではないこと。
- ユーザーが親テーブルに対して `REFERENCES` 権限を持っていること。
- 親テーブルと子テーブルの外部キーで参照されるカラムが同じデータ型であり、サイズ、精度、長さ、文字セット、照合順序が同じであること。
- 外部キーで参照されるカラムは自分自身を参照することはできないこと。
- 外部キーで参照されるカラムと親テーブルのカラムが同じインデックスを持ち、インデックス内のカラムの順序が外部キーと一致すること。これにより、外部キー制約のチェック時にフルテーブルスキャンを避けるためにインデックスを使用します。
  - 親テーブルに対応する外部キーインデックスが存在しない場合、`ERROR 1822: Failed to add the foreign key constraint. Missing index for constraint 'fk' in the referenced table 't'` のエラーが報告されます。
  - 子テーブルに対応する外部キーインデックスが存在しない場合、同じ名前のインデックスが自動的に作成されます。
- `BLOB` や `TEXT` 型のカラムに対して外部キーを作成することはサポートされていません。
- パーティションテーブルに対して外部キーを作成することはサポートされていません。
- 仮想生成カラムに対して外部キーを作成することはサポートされていません。

## 参照操作 {#reference-operations}

`UPDATE` や `DELETE` 操作が親テーブルの外部キーの値に影響を与える場合、外部キー定義の `ON UPDATE` や `ON DELETE` 句で定義された参照操作によって子テーブルの対応する外部キーの値が決定されます。参照操作には以下のものがあります：

- `CASCADE`：`UPDATE` や `DELETE` 操作が親テーブルに影響を与えると、子テーブルの対応する行が自動的に更新または削除されます。カスケード操作は深さ優先で実行されます。
- `SET NULL`：`UPDATE` や `DELETE` 操作が親テーブルに影響を与えると、子テーブルの対応する外部キーのカラムが `NULL` に自動的に設定されます。
- `RESTRICT`：子テーブルに対応する行が存在する場合、`UPDATE` や `DELETE` 操作を拒否します。
- `NO ACTION`：`RESTRICT` と同じです。
- `SET DEFAULT`：`RESTRICT` と同じです。

親テーブルに対応する外部キーの値が存在しない場合、子テーブルでの `INSERT` や `UPDATE` 操作は拒否されます。

外部キー定義で `ON DELETE` や `ON UPDATE` を指定しない場合、デフォルトの動作は `NO ACTION` です。

外部キーが `STORED GENERATED COLUMN` に定義されている場合、`CASCADE`、`SET NULL`、`SET DEFAULT` の参照操作はサポートされていません。

## 外部キーの使用例 {#usage-examples-of-foreign-keys}

以下の例では、親テーブルと子テーブルを関連付けるために単一のカラムの外部キーを使用しています：

```sql
CREATE TABLE parent (
    id INT KEY
);

CREATE TABLE child (
    id INT,
    pid INT,
    INDEX idx_pid (pid),
    FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE CASCADE
);
```

以下は、`product_order`テーブルが他の2つのテーブルを参照する2つの外部キーを持つより複雑な例です。1つの外部キーは、`product`テーブルの2つのインデックスを参照し、もう1つは`customer`テーブルの単一のインデックスを参照します。

```sql
CREATE TABLE product (
    category INT NOT NULL,
    id INT NOT NULL,
    price DECIMAL(20,10),
    PRIMARY KEY(category, id)
);

CREATE TABLE customer (
    id INT KEY
);

CREATE TABLE product_order (
    id INT NOT NULL AUTO_INCREMENT,
    product_category INT NOT NULL,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,

    PRIMARY KEY(id),
    INDEX (product_category, product_id),
    INDEX (customer_id),

    FOREIGN KEY (product_category, product_id)
      REFERENCES product(category, id)
      ON UPDATE CASCADE ON DELETE RESTRICT,

    FOREIGN KEY (customer_id)
      REFERENCES customer(id)
);
```

## 外部キー制約の作成 {#create-a-foreign-key-constraint}

外部キー制約を作成するには、次の `ALTER TABLE` ステートメントを使用できます。

```sql
ALTER TABLE table_name
    ADD [CONSTRAINT [identifier]] FOREIGN KEY
    [identifier] (col_name, ...)
    REFERENCES tbl_name (col_name,...)
    [ON DELETE reference_option]
    [ON UPDATE reference_option]
```

外部キーは自己参照することができます。つまり、同じテーブルを参照することができます。`ALTER TABLE`を使用してテーブルに外部キー制約を追加する場合、最初に外部キーが参照する親テーブルのカラムにインデックスを作成する必要があります。

## 外部キー制約の削除 {#delete-a-foreign-key-constraint}

外部キー制約を削除するには、次の`ALTER TABLE`ステートメントを使用できます。

```sql
ALTER TABLE table_name DROP FOREIGN KEY fk_identifier;
```

外部キー制約が作成時に名前が付けられている場合は、その名前を参照して外部キー制約を削除することができます。そうでない場合は、自動的に生成された制約名を使用して制約を削除する必要があります。外部キー名を確認するには、`SHOW CREATE TABLE`を使用できます。

```sql
mysql> SHOW CREATE TABLE child\G
*************************** 1. row ***************************
       Table: child
Create Table: CREATE TABLE `child` (
  `id` int(11) DEFAULT NULL,
  `pid` int(11) DEFAULT NULL,
  KEY `idx_pid` (`pid`),
  CONSTRAINT `fk_1` FOREIGN KEY (`pid`) REFERENCES `test`.`parent` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```

## 外部キー制約のチェック {#foreign-key-constraint-check}

TiDBでは、システム変数[`foreign_key_checks`](/system-variables.md#foreign_key_checks)で制御される外部キー制約のチェックをサポートしています。デフォルトでは、この変数は`ON`に設定されており、外部キー制約のチェックが有効になっています。この変数には`GLOBAL`と`SESSION`の2つのスコープがあります。この変数を有効にしておくことで、外部キー参照関係の整合性を保証することができます。

外部キー制約のチェックを無効にすると、次のような効果があります。

- 外部キーで参照されている親テーブルを削除する場合、外部キー制約のチェックが無効になっている場合にのみ削除が成功します。
- データをデータベースにインポートする際、テーブルの作成順序が外部キーの依存順序と異なる場合、テーブルの作成に失敗する可能性があります。外部キー制約のチェックが無効になっている場合のみ、テーブルを正常に作成することができます。また、外部キー制約のチェックを無効にすることで、データのインポートを高速化することができます。
- データをデータベースにインポートする際、子テーブルのデータが最初にインポートされる場合、エラーが報告されます。外部キー制約のチェックが無効になっている場合のみ、子テーブルのデータを正常にインポートすることができます。
- 実行する`ALTER TABLE`操作が外部キーの変更を伴う場合、外部キー制約のチェックが無効になっている場合のみ、この操作が成功します。

外部キー制約のチェックが無効になっている場合、外部キー制約のチェックや参照操作は、次のシナリオを除いて実行されません。

- `ALTER TABLE`の実行によって外部キーの定義が間違ってしまう可能性がある場合、実行中にエラーが報告されます。
- 外部キーに必要なインデックスを削除する場合、まず外部キーを削除する必要があります。そうしないと、エラーが報告されます。
- 外部キーを作成する際、関連する条件や制限を満たさない場合、エラーが報告されます。

## ロック {#locking}

子テーブルに`INSERT`または`UPDATE`を行う際、外部キー制約は親テーブルにおいて対応する外部キー値が存在するかどうかをチェックし、外部キー制約に違反する他の操作によって外部キー値が削除されることを防ぐために、親テーブルの行をロックします。このロックの動作は、親テーブルにおいて外部キー値が存在する行に対して`SELECT FOR UPDATE`操作を実行するのと同等です。

現在、TiDBでは`LOCK IN SHARE MODE`をサポートしていないため、子テーブルが大量の並行書き込みを受け入れ、参照される外部キー値の大部分が同じ場合、深刻なロックの競合が発生する可能性があります。大量の子テーブルデータを書き込む場合は、[`foreign_key_checks`](/system-variables.md#foreign_key_checks)を無効にすることをお勧めします。

## 外部キーの定義とメタデータ {#definition-and-metadata-of-foreign-keys}

外部キー制約の定義を表示するには、[`SHOW CREATE TABLE`](/sql-statements/sql-statement-show-create-table.md)ステートメントを実行します。

```sql
mysql> SHOW CREATE TABLE child\G
*************************** 1. row ***************************
       Table: child
Create Table: CREATE TABLE `child` (
  `id` int(11) DEFAULT NULL,
  `pid` int(11) DEFAULT NULL,
  KEY `idx_pid` (`pid`),
  CONSTRAINT `fk_1` FOREIGN KEY (`pid`) REFERENCES `test`.`parent` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```

次のシステムテーブルのいずれかを使用して、外部キーに関する情報を取得することもできます：

- [`INFORMATION_SCHEMA.KEY_COLUMN_USAGE`](/information-schema/information-schema-key-column-usage.md)
- [`INFORMATION_SCHEMA.TABLE_CONSTRAINTS`](/information-schema/information-schema-table-constraints.md)
- [`INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS`](/information-schema/information-schema-referential-constraints.md)

以下に例を示します：

`INFORMATION_SCHEMA.KEY_COLUMN_USAGE`システムテーブルから外部キーに関する情報を取得する：

```sql
mysql> SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, CONSTRAINT_NAME FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE WHERE REFERENCED_TABLE_SCHEMA IS NOT NULL;
+--------------+---------------+------------------+-----------------+
| TABLE_SCHEMA | TABLE_NAME    | COLUMN_NAME      | CONSTRAINT_NAME |
+--------------+---------------+------------------+-----------------+
| test         | child         | pid              | fk_1            |
| test         | product_order | product_category | fk_1            |
| test         | product_order | product_id       | fk_1            |
| test         | product_order | customer_id      | fk_2            |
+--------------+---------------+------------------+-----------------+
```

`INFORMATION_SCHEMA.TABLE_CONSTRAINTS`システムテーブルから外部キーに関する情報を取得します：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS WHERE CONSTRAINT_TYPE='FOREIGN KEY'\G
***************************[ 1. row ]***************************
CONSTRAINT_CATALOG | def
CONSTRAINT_SCHEMA  | test
CONSTRAINT_NAME    | fk_1
TABLE_SCHEMA       | test
TABLE_NAME         | child
CONSTRAINT_TYPE    | FOREIGN KEY
```

`INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS`システムテーブルから外部キーに関する情報を取得します：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS\G
***************************[ 1. row ]***************************
CONSTRAINT_CATALOG        | def
CONSTRAINT_SCHEMA         | test
CONSTRAINT_NAME           | fk_1
UNIQUE_CONSTRAINT_CATALOG | def
UNIQUE_CONSTRAINT_SCHEMA  | test
UNIQUE_CONSTRAINT_NAME    | PRIMARY
MATCH_OPTION              | NONE
UPDATE_RULE               | NO ACTION
DELETE_RULE               | CASCADE
TABLE_NAME                | child
REFERENCED_TABLE_NAME     | parent
```

## 外部キーを使用して実行計画を表示する {#view-execution-plans-with-foreign-keys}

`EXPLAIN`ステートメントを使用して実行計画を表示することができます。`Foreign_Key_Check`オペレーターは、実行されるDMLステートメントに対して外部キー制約のチェックを実行します。

```sql
mysql> explain insert into child values (1,1);
+-----------------------+---------+------+---------------+-------------------------------+
| id                    | estRows | task | access object | operator info                 |
+-----------------------+---------+------+---------------+-------------------------------+
| Insert_1              | N/A     | root |               | N/A                           |
| └─Foreign_Key_Check_3 | 0.00    | root | table:parent  | foreign_key:fk_1, check_exist |
+-----------------------+---------+------+---------------+-------------------------------+
```

`EXPLAIN ANALYZE`ステートメントを使用すると、外部キー参照の動作を表示できます。`Foreign_Key_Cascade`オペレーターは、実行されるDMLステートメントに対して外部キー参照を実行します。

```sql
mysql> explain analyze delete from parent where id = 1;
+----------------------------------+---------+---------+-----------+---------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------+-----------+------+
| id                               | estRows | actRows | task      | access object                   | execution info                                                                                                                                                                               | operator info                               | memory    | disk |
+----------------------------------+---------+---------+-----------+---------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------+-----------+------+
| Delete_2                         | N/A     | 0       | root      |                                 | time:117.3µs, loops:1                                                                                                                                                                        | N/A                                         | 380 Bytes | N/A  |
| ├─Point_Get_1                    | 1.00    | 1       | root      | table:parent                    | time:63.6µs, loops:2, Get:{num_rpc:1, total_time:29.9µs}                                                                                                                                     | handle:1                                    | N/A       | N/A  |
| └─Foreign_Key_Cascade_3          | 0.00    | 0       | root      | table:child, index:idx_pid      | total:1.28ms, foreign_keys:1                                                                                                                                                                 | foreign_key:fk_1, on_delete:CASCADE         | N/A       | N/A  |
|   └─Delete_7                     | N/A     | 0       | root      |                                 | time:904.8µs, loops:1                                                                                                                                                                        | N/A                                         | 1.11 KB   | N/A  |
|     └─IndexLookUp_11             | 10.00   | 1       | root      |                                 | time:869.5µs, loops:2, index_task: {total_time: 371.1µs, fetch_handle: 357.3µs, build: 1.25µs, wait: 12.5µs}, table_task: {total_time: 382.6µs, num: 1, concurrency: 5}                      |                                             | 9.13 KB   | N/A  |
|       ├─IndexRangeScan_9(Build)  | 10.00   | 1       | cop[tikv] | table:child, index:idx_pid(pid) | time:351.2µs, loops:3, cop_task: {num: 1, max: 282.3µs, proc_keys: 0, rpc_num: 1, rpc_time: 263µs, copr_cache_hit_ratio: 0.00, distsql_concurrency: 15}, tikv_task:{time:220.2µs, loops:0}   | range:[1,1], keep order:false, stats:pseudo | N/A       | N/A  |
|       └─TableRowIDScan_10(Probe) | 10.00   | 1       | cop[tikv] | table:child                     | time:223.9µs, loops:2, cop_task: {num: 1, max: 168.8µs, proc_keys: 0, rpc_num: 1, rpc_time: 154.5µs, copr_cache_hit_ratio: 0.00, distsql_concurrency: 15}, tikv_task:{time:145.6µs, loops:0} | keep order:false, stats:pseudo              | N/A       | N/A  |
+----------------------------------+---------+---------+-----------+---------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------+-----------+------+
```

## 互換性 {#compatibility}

### TiDBバージョン間の互換性 {#compatibility-between-tidb-versions}

v6.6.0より前のバージョンでは、TiDBは外部キーを作成する構文をサポートしていますが、作成された外部キーは無効です。v6.6.0以前に作成されたTiDBクラスターをv6.6.0以降にアップグレードすると、アップグレード前に作成された外部キーは引き続き無効です。v6.6.0以降のバージョンで作成された外部キーのみが有効です。無効な外部キーを削除し、新しいものを作成することで外部キー制約を有効にすることができます。`SHOW CREATE TABLE`ステートメントを使用して、外部キーが有効かどうかを確認することができます。無効な外部キーには`/* FOREIGN KEY INVALID */`のコメントが付いています。

```sql
mysql> SHOW CREATE TABLE child\G
***************************[ 1. row ]***************************
Table        | child
Create Table | CREATE TABLE `child` (
  `id` int(11) DEFAULT NULL,
  `pid` int(11) DEFAULT NULL,
  KEY `idx_pid` (`pid`),
  CONSTRAINT `fk_1` FOREIGN KEY (`pid`) REFERENCES `test`.`parent` (`id`) ON DELETE CASCADE /* FOREIGN KEY INVALID */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```

### TiDBツールとの互換性 {#compatibility-with-tidb-tools}

<CustomContent platform="tidb">

- [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md)は外部キーをサポートしていません。
- [DM](/dm/dm-overview.md)は外部キーをサポートしていません。DM v6.6.0では、データをTiDBにレプリケートする際に、ダウンストリームのTiDBの[`foreign_key_checks`](/system-variables.md#foreign_key_checks)が無効になります。そのため、外部キーによって引き起こされるカスケード操作は、アップストリームからダウンストリームにレプリケートされず、データの不整合が発生する可能性があります。この動作は、以前のDMバージョンと同じです。
- [TiCDC](/ticdc/ticdc-overview.md) v6.6.0は外部キーと互換性があります。以前のTiCDCバージョンでは、外部キーを持つテーブルをレプリケートする際にエラーが発生する可能性があります。TiCDCのバージョンがv6.6.0よりも前の場合は、ダウンストリームのTiDBクラスタの`foreign_key_checks`を無効にすることをお勧めします。
- [BR](/br/backup-and-restore-overview.md) v6.6.0は外部キーと互換性があります。以前のBRバージョンでは、外部キーを持つテーブルをv6.6.0以降のクラスタにリストアする際にエラーが発生する可能性があります。BRのバージョンがv6.6.0よりも前の場合は、クラスタをリストアする前に、ダウンストリームのTiDBクラスタの`foreign_key_checks`を無効にすることをお勧めします。
- [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用する場合は、データをインポートする前に、ダウンストリームのTiDBクラスタの`foreign_key_checks`を無効にすることをお勧めします。

</CustomContent>

- [Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview)は外部キーと互換性があります。

<CustomContent platform="tidb">

- アップストリームとダウンストリームのデータベースを比較するために[sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md)を使用する場合、データベースのバージョンが異なり、ダウンストリームのTiDBに[無効な外部キーがある場合](#compatibility-between-tidb-versions)、sync-diff-inspectorはテーブルスキーマの不整合エラーを報告する可能性があります。これは、TiDB v6.6.0が無効な外部キーに`/* FOREIGN KEY INVALID */`コメントを追加するためです。

</CustomContent>

### MySQLとの互換性 {#compatibility-with-mysql}

外部キーを名前を指定せずに作成すると、TiDBで生成される名前はMySQLで生成される名前と異なります。例えば、TiDBで生成される外部キーの名前は`fk_1`、`fk_2`、`fk_3`ですが、MySQLで生成される外部キーの名前は`table_name_ibfk_1`、`table_name_ibfk_2`、`table_name_ibfk_3`です。

MySQLとTiDBの両方が「インラインの`REFERENCES`仕様」を解析しますが、無視します。`FOREIGN KEY`定義の一部である`REFERENCES`仕様のみがチェックされ、強制されます。次の例では、`REFERENCES`句を使用して外部キー制約を作成します：

```sql
CREATE TABLE parent (
    id INT KEY
);

CREATE TABLE child (
    id INT,
    pid INT REFERENCES parent(id)
);

SHOW CREATE TABLE child;
```

出力によると、 `child` テーブルには外部キーが含まれていません。

```sql
+-------+-------------------------------------------------------------+
| Table | Create Table                                                |
+-------+-------------------------------------------------------------+
| child | CREATE TABLE `child` (                                      |
|       |   `id` int(11) DEFAULT NULL,                                |
|       |   `pid` int(11) DEFAULT NULL                                |
|       | ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin |
+-------+-------------------------------------------------------------+
```
