---
title: FOREIGN KEY Constraints
summary: An overview of the usage of FOREIGN KEY constraints for the TiDB database.
---

# FOREIGN KEY Constraints {#foreign-key-constraints}

v6.6.0から、TiDBは外部キー機能をサポートしており、関連するデータのクロステーブル参照とデータ整合性を維持するための外部キー制約を可能にします。

> **Warning:**
>
> - 現在、外部キー機能は実験的なものです。本番環境で使用することは推奨されません。この機能は事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。
> - 外部キー機能は通常、[参照整合性](https://en.wikipedia.org/wiki/Referential_integrity)制約チェックを強制するために使用されます。これはパフォーマンスの低下を引き起こす可能性がありますので、パフォーマンスに敏感なシナリオで使用する前に徹底的なテストを行うことをお勧めします。

外部キーは子テーブルで定義されます。構文は次のようになります：

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

外部キーの命名は次のルールに従います。

- `CONSTRAINT identifier` で名前が指定されている場合、指定された名前が使用されます。
- `CONSTRAINT identifier` で名前が指定されていないが、`FOREIGN KEY identifier` で名前が指定されている場合、`FOREIGN KEY identifier` で指定された名前が使用されます。
- `CONSTRAINT identifier` と `FOREIGN KEY identifier` の両方が名前を指定していない場合、`fk_1`、`fk_2`、`fk_3` など、名前が自動的に生成されます。
- 外部キーの名前は、現在のテーブルで一意でなければなりません。そうでない場合、外部キーが作成されるときにエラー `ERROR 1826: Duplicate foreign key constraint name 'fk'` が報告されます。

## 制限 {#restrictions}

外部キーを作成する際には、次の条件を満たす必要があります。

- 親テーブルも子テーブルも一時テーブルでないこと。

- ユーザーが親テーブルに対する `REFERENCES` 権限を持っていること。

- 親テーブルの外部キーで参照される列と子テーブルの列が、データ型、サイズ、精度、長さ、文字セット、および照合順序が同じであること。

- 外部キーで参照される列が自分自身を参照していないこと。

- 外部キーの列と参照される親テーブルの列が同じインデックスを持ち、インデックス内の列の順序が外部キー内の順序と一致していること。これは、外部キー制約のチェックを行う際にフルテーブルスキャンを避けるためにインデックスを使用するためです。

  - 親テーブルに対応する外部キーインデックスがない場合、エラー `ERROR 1822: Failed to add the foreign key constraint. Missing index for constraint 'fk' in the referenced table 't'` が報告されます。
  - 子テーブルに対応する外部キーインデックスがない場合、外部キーと同じ名前のインデックスが自動的に作成されます。

- `BLOB` や `TEXT` 型の列に外部キーを作成することはサポートされていません。

- パーティションテーブルに外部キーを作成することはサポートされていません。

- 仮想生成列に外部キーを作成することはサポートされていません。

## 参照操作 {#reference-operations}

`UPDATE` または `DELETE` 操作が親テーブルの外部キー値に影響を与える場合、`ON UPDATE` または `ON DELETE` 句で定義された参照操作によって、子テーブルの対応する外部キー値が決定されます。参照操作には次のものがあります。

- `CASCADE`: 親テーブルに影響を与える `UPDATE` または `DELETE` 操作が行われたとき、子テーブルの一致する行が自動的に更新または削除されます。カスケード操作は深さ優先で行われます。
- `SET NULL`: 親テーブルに影響を与える `UPDATE` または `DELETE` 操作が行われたとき、子テーブルの一致する外部キー列が自動的に `NULL` に設定されます。
- `RESTRICT`: 子テーブルに一致する行が含まれている場合、`UPDATE` または `DELETE` 操作を拒否します。
- `NO ACTION`: `RESTRICT` と同じです。
- `SET DEFAULT`: `RESTRICT` と同じです。

親テーブルに一致する外部キー値がない場合、子テーブルの `INSERT` または `UPDATE` 操作は拒否されます。

外部キーの定義で `ON DELETE` または `ON UPDATE` が指定されていない場合、デフォルトの動作は `NO ACTION` です。

外部キーが `STORED GENERATED COLUMN` に定義されている場合、`CASCADE`、`SET NULL`、`SET DEFAULT` の参照操作はサポートされていません。

## 外部キーの使用例 {#usage-examples-of-foreign-keys}

次の例では、単一列の外部キーを使用して親テーブルと子テーブルを関連付けます。

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

以下は、`product_order` テーブルが他の2つのテーブルを参照する2つの外部キーを持つより複雑な例です。1つの外部キーは `product` テーブルの2つのインデックスを参照し、もう1つは `customer` テーブルの単一のインデックスを参照しています。**Warning:** このクエリは実験的な機能であり、本番環境での使用はお勧めしません。

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

外部キー制約を作成するには、次の `ALTER TABLE` ステートメントを使用できます:

```sql
ALTER TABLE table_name
    ADD [CONSTRAINT [identifier]] FOREIGN KEY
    [identifier] (col_name, ...)
    REFERENCES tbl_name (col_name,...)
    [ON DELETE reference_option]
    [ON UPDATE reference_option]
```

外部キーは自己参照することができ、つまり、同じテーブルを参照することができます。`ALTER TABLE`を使用してテーブルに外部キー制約を追加する場合、最初に外部キーが参照する親テーブルの列にインデックスを作成する必要があります。

## 外部キー制約の削除 {#delete-a-foreign-key-constraint}

外部キー制約を削除するには、次の`ALTER TABLE`ステートメントを使用できます：

```sql
ALTER TABLE table_name DROP FOREIGN KEY fk_identifier;
```

外部キー制約が作成される際に名前が付けられている場合、その名前を参照して外部キー制約を削除することができます。それ以外の場合、制約を削除するために自動的に生成された制約名を使用する必要があります。外部キー名を表示するには、`SHOW CREATE TABLE`を使用できます。**Warning:** Don't translate the key in the content. You should replace the key with the value.

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

TiDBは外部キー制約のチェックをサポートしており、これはシステム変数[`foreign_key_checks`](/system-variables.md#foreign_key_checks)で制御されます。デフォルトでは、この変数は`ON`に設定されており、外部キー制約のチェックが有効になっています。この変数には`GLOBAL`と`SESSION`の2つのスコープがあります。この変数を有効にしておくことで、外部キー参照関係の整合性を確保できます。

外部キー制約のチェックを無効にすると、次のような効果があります。

- 外部キーによって参照されている親テーブルを削除する場合、外部キー制約のチェックが無効になっているときのみ削除が成功します。
- データをデータベースにインポートする際、テーブルの作成順序が外部キーの依存順序と異なる場合、テーブルの作成に失敗する可能性があります。外部キー制約のチェックが無効になっているときのみ、テーブルを正常に作成できます。また、外部キー制約のチェックを無効にすることで、データのインポートを高速化できます。
- データをデータベースにインポートする際、子テーブルのデータが最初にインポートされるとエラーが報告されます。外部キー制約のチェックが無効になっているときのみ、子テーブルのデータを正常にインポートできます。
- 実行する`ALTER TABLE`操作が外部キーの変更を含む場合、この操作は外部キー制約のチェックが無効になっているときのみ成功します。

外部キー制約のチェックが無効になっているときは、外部キー制約のチェックと参照操作は実行されませんが、次のシナリオでは例外があります。

- `ALTER TABLE`の実行が外部キーの定義を誤ってしまう可能性がある場合、実行中にエラーが報告されます。
- 外部キーに必要なインデックスを削除する場合、まず外部キーを削除する必要があります。そうでない場合、エラーが報告されます。
- 外部キーを作成する際に関連する条件や制限を満たさない場合、エラーが報告されます。

## ロック {#locking}

子テーブルに`INSERT`または`UPDATE`を行う際、外部キー制約は対応する外部キー値が親テーブルに存在するかどうかをチェックし、外部キー制約に違反する他の操作によって外部キー値が削除されるのを防ぐために、親テーブルの行をロックします。このロックの動作は、親テーブルの外部キー値が存在する行に`SELECT FOR UPDATE`操作を実行するのと同等です。

TiDBは現在`LOCK IN SHARE MODE`をサポートしていないため、子テーブルが大量の同時書き込みを受け入れ、参照される外部キー値のほとんどが同じ場合、深刻なロックの競合が発生する可能性があります。大量の子テーブルデータを書き込む際には、[`foreign_key_checks`](/system-variables.md#foreign_key_checks)を無効にすることをお勧めします。

## 外部キーの定義とメタデータ {#definition-and-metadata-of-foreign-keys}

外部キー制約の定義を表示するには、[`SHOW CREATE TABLE`](/sql-statements/sql-statement-show-create-table.md)ステートメントを実行します。"

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

あなたは次のいずれかのシステムテーブルを使用して外部キーに関する情報を取得することもできます。

- [`INFORMATION_SCHEMA.KEY_COLUMN_USAGE`](/information-schema/information-schema-key-column-usage.md)
- [`INFORMATION_SCHEMA.TABLE_CONSTRAINTS`](/information-schema/information-schema-table-constraints.md)
- [`INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS`](/information-schema/information-schema-referential-constraints.md)

以下は例です。

`INFORMATION_SCHEMA.KEY_COLUMN_USAGE` システムテーブルから外部キーに関する情報を取得する:

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

`INFORMATION_SCHEMA.TABLE_CONSTRAINTS` システムテーブルから外部キーに関する情報を取得します。

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

`INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS` システムテーブルから外部キーに関する情報を取得します。

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

## 外部キーを持つ実行計画を表示する {#view-execution-plans-with-foreign-keys}

`EXPLAIN`ステートメントを使用して実行計画を表示できます。 `Foreign_Key_Check`オペレーターは、実行されるDMLステートメントの外部キー制約チェックを実行します。

```sql
mysql> explain insert into child values (1,1);
+-----------------------+---------+------+---------------+-------------------------------+
| id                    | estRows | task | access object | operator info                 |
+-----------------------+---------+------+---------------+-------------------------------+
| Insert_1              | N/A     | root |               | N/A                           |
| └─Foreign_Key_Check_3 | 0.00    | root | table:parent  | foreign_key:fk_1, check_exist |
+-----------------------+---------+------+---------------+-------------------------------+
```

`EXPLAIN ANALYZE`ステートメントを使用して、外部キー参照動作の実行を表示できます。 `Foreign_Key_Cascade`演算子は、実行されるDMLステートメントの外部キー参照を実行します。

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

### TiDB バージョン間の互換性 {#compatibility-between-tidb-versions}

v6.6.0 より前のバージョンでは、TiDB は外部キーを作成する構文をサポートしていますが、作成された外部キーは無効です。v6.6.0 より前に作成された TiDB クラスタを v6.6.0 以降にアップグレードすると、アップグレード前に作成された外部キーは引き続き無効です。v6.6.0 以降のバージョンで作成された外部キーのみが有効です。無効な外部キーを削除し、新しい外部キーを作成して外部キー制約を有効にすることができます。`SHOW CREATE TABLE` ステートメントを使用して、外部キーが有効かどうかを確認できます。無効な外部キーには `/* FOREIGN KEY INVALID */` コメントが付いています。

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

- [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md) は外部キーをサポートしていません。
- [DM](/dm/dm-overview.md) は外部キーをサポートしていません。 DM v6.6.0は、データをTiDBにレプリケートする際に、下流のTiDBの[`foreign_key_checks`](/system-variables.md#foreign_key_checks)を無効にします。そのため、外部キーによって引き起こされる連鎖的な操作は上流から下流にレプリケートされず、データの不整合が発生する可能性があります。この動作は以前のDMバージョンと一貫しています。
- [TiCDC](/ticdc/ticdc-overview.md) v6.6.0は外部キーと互換性があります。以前のTiCDCバージョンでは、外部キーを持つテーブルをレプリケートする際にエラーが発生することがあります。TiCDCのバージョンがv6.6.0よりも古い場合は、使用する下流のTiDBクラスタの`foreign_key_checks`を無効にすることをお勧めします。
- [BR](/br/backup-and-restore-overview.md) v6.6.0は外部キーと互換性があります。以前のBRバージョンでは、外部キーを持つテーブルをv6.6.0以降のクラスタに復元する際にエラーが発生することがあります。BRをv6.6.0よりも古いバージョンでクラスタを復元する場合は、復元前に下流のTiDBクラスタの`foreign_key_checks`を無効にすることをお勧めします。
- [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用する場合は、データをインポートする前に下流のTiDBクラスタの`foreign_key_checks`を無効にすることをお勧めします。

</CustomContent>

- [Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview) は外部キーと互換性があります。

<CustomContent platform="tidb">

- [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md)を使用して上流と下流のデータを比較する場合、データベースのバージョンが異なり、下流のTiDBに[無効な外部キー](#compatibility-between-tidb-versions)がある場合、sync-diff-inspectorはテーブルスキーマの不整合エラーを報告することがあります。これは、TiDB v6.6.0が無効な外部キーに`/* FOREIGN KEY INVALID */`コメントを追加するためです。

</CustomContent>

### MySQLとの互換性 {#compatibility-with-mysql}

外部キーを名前を指定せずに作成すると、TiDBによって生成される名前はMySQLによって生成される名前とは異なります。たとえば、TiDBによって生成される外部キーの名前は`fk_1`、`fk_2`、`fk_3`ですが、MySQLによって生成される外部キーの名前は`table_name_ibfk_1`、`table_name_ibfk_2`、`table_name_ibfk_3`です。

MySQLとTiDBの両方が、「インラインの`REFERENCES`仕様」を解析しますが無視します。`FOREIGN KEY`定義の一部である`REFERENCES`仕様のみがチェックおよび強制されます。次の例は、`REFERENCES`句を使用して外部キー制約を作成します。"

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

出力によると、 `child` テーブルには外部キーが含まれていないことが示されています。

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
