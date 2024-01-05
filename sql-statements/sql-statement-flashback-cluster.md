---
title: FLASHBACK CLUSTER
summary: Learn the usage of FLASHBACK CLUSTER in TiDB databases.
aliases: ['/tidb/v7.1/sql-statement-flashback-to-timestamp', '/tidbcloud/sql-statement-flashback-to-timestamp']
---

# FLASHBACK CLUSTER {#flashback-cluster}

TiDB v6.4.0は、`FLASHBACK CLUSTER TO TIMESTAMP`構文を導入しました。これを使用して、クラスタを特定の時点に復元できます。タイムスタンプを指定する場合、日時値を設定するか、時間関数を使用できます。日時の形式は '2016-10-08 16:45:26.999' のようなもので、最小の時間単位はミリ秒です。ただし、ほとんどの場合、秒を時間単位としてタイムスタンプを指定するだけで十分です。たとえば、'2016-10-08 16:45:26' です。

v6.5.6およびv7.1.3から、TiDBは`FLASHBACK CLUSTER TO TSO`構文を導入しました。この構文を使用すると、[TSO](/tso.md)を使用して、より正確な復旧時点を指定できるため、データの復旧の柔軟性が向上します。

> **Warning:**
>
> `FLASHBACK CLUSTER TO [TIMESTAMP|TSO]`構文は、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタには適用されません。予期しない結果を避けるため、TiDB Serverlessクラスタでこのステートメントを実行しないでください。

<CustomContent platform="tidb">

> **Warning:**
>
> TiDB v7.1.0でこの機能を使用すると、一部のリージョンがFLASHBACK操作の完了後もFLASHBACKプロセスに残ることがあります。v7.1.0でこの機能を使用しないことをお勧めします。詳細については、issue [#44292](https://github.com/pingcap/tidb/issues/44292)を参照してください。
>
> この問題に遭遇した場合は、[TiDBスナップショットバックアップとリストア](/br/br-snapshot-guide.md)機能を使用してデータを復元できます。

</CustomContent>

> **Note:**
>
> `FLASHBACK CLUSTER TO [TIMESTAMP|TSO]`の動作原理は、特定の時点の古いデータを最新のタイムスタンプで書き込み、現在のデータを削除しないことです。したがって、この機能を使用する前に、古いデータと現在のデータのための十分なストレージスペースがあることを確認する必要があります。

## Syntax {#syntax}

```sql
FLASHBACK CLUSTER TO TIMESTAMP '2022-09-21 16:02:50';
FLASHBACK CLUSTER TO TSO 445494839813079041;
```

### 概要 {#synopsis}

```ebnf+diagram
FlashbackToTimestampStmt
         ::= 'FLASHBACK' 'CLUSTER' 'TO' 'TIMESTAMP' stringLit
           | 'FLASHBACK' 'CLUSTER' 'TO' 'TSO' LengthNum
```

## ノート {#notes}

- `FLASHBACK` ステートメントで指定された時間は、ガベージコレクション（GC）の寿命内でなければなりません。システム変数[`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50)（デフォルト：`10m0s`）は、行の以前のバージョンの保持期間を定義します。ガベージコレクションが実行された`safePoint`は、次のクエリで取得できます：

  ```sql
  SELECT * FROM mysql.tidb WHERE variable_name = 'tikv_gc_safe_point';
  ```

<CustomContent platform='tidb'>

- `SUPER` 権限を持つユーザーのみが `FLASHBACK CLUSTER` SQL ステートメントを実行できます。
- `FLASHBACK CLUSTER` は、`ALTER TABLE ATTRIBUTE`、`ALTER TABLE REPLICA`、`CREATE PLACEMENT POLICY`など、PD 関連情報を変更する DDL ステートメントのロールバックをサポートしません。
- `FLASHBACK` ステートメントで指定された時間に、完全に実行されていない DDL ステートメントが存在してはいけません。そのような DDL が存在する場合、TiDB はそれを拒否します。
- `FLASHBACK CLUSTER` を実行する前に、TiDB は関連する接続をすべて切断し、これらのテーブルでの読み書き操作を禁止します。`FLASHBACK CLUSTER` ステートメントが完了するまで、これらのテーブルでの読み書き操作を禁止します。
- `FLASHBACK CLUSTER` ステートメントは、実行後にキャンセルすることはできません。TiDB は成功するまで繰り返し試行します。
- `FLASHBACK CLUSTER` の実行中にデータのバックアップが必要な場合、[バックアップ＆リストア](/br/br-snapshot-guide.md)を使用し、`BackupTS`を`FLASHBACK CLUSTER`の開始時間よりも前の時間に指定する必要があります。また、`FLASHBACK CLUSTER` の実行中に[ログバックアップ](/br/br-pitr-guide.md)を有効にすると失敗します。そのため、`FLASHBACK CLUSTER` の完了後にログバックアップを有効にしてください。
- `FLASHBACK CLUSTER` ステートメントによってメタデータ（テーブル構造、データベース構造）がロールバックされる場合、関連する変更は TiCDC によってレプリケートされません。そのため、タスクを手動で一時停止し、`FLASHBACK CLUSTER` の完了を待ち、上流と下流のスキーマ定義を手動でレプリケートして、一貫性を確認する必要があります。その後、TiCDC changefeed を再作成する必要があります。

</CustomContent>

<CustomContent platform='tidb-cloud'>

- `SUPER` 権限を持つユーザーのみが `FLASHBACK CLUSTER` SQL ステートメントを実行できます。
- `FLASHBACK CLUSTER` は、`ALTER TABLE ATTRIBUTE`、`ALTER TABLE REPLICA`、`CREATE PLACEMENT POLICY`など、PD 関連情報を変更する DDL ステートメントのロールバックをサポートしません。
- `FLASHBACK` ステートメントで指定された時間に、完全に実行されていない DDL ステートメントが存在してはいけません。そのような DDL が存在する場合、TiDB はそれを拒否します。
- `FLASHBACK CLUSTER` を実行する前に、TiDB は関連する接続をすべて切断し、これらのテーブルでの読み書き操作を禁止します。`FLASHBACK CLUSTER` ステートメントが完了するまで、これらのテーブルでの読み書き操作を禁止します。
- `FLASHBACK CLUSTER` ステートメントは、実行後にキャンセルすることはできません。TiDB は成功するまで繰り返し試行します。
- `FLASHBACK CLUSTER` ステートメントによってメタデータ（テーブル構造、データベース構造）がロールバックされる場合、関連する変更は TiCDC によってレプリケートされません。そのため、タスクを手動で一時停止し、`FLASHBACK CLUSTER` の完了を待ち、上流と下流のスキーマ定義を手動でレプリケートして、一貫性を確認する必要があります。その後、TiCDC changefeed を再作成する必要があります。

</CustomContent>

## 例 {#example}

次の例は、特定のタイムスタンプにクラスタをフラッシュバックして新しく挿入されたデータを復元する方法を示しています：

```sql
mysql> CREATE TABLE t(a INT);
Query OK, 0 rows affected (0.09 sec)

mysql> SELECT * FROM t;
Empty set (0.01 sec)

mysql> SELECT now();
+---------------------+
| now()               |
+---------------------+
| 2022-09-28 17:24:16 |
+---------------------+
1 row in set (0.02 sec)

mysql> INSERT INTO t VALUES (1);
Query OK, 1 row affected (0.02 sec)

mysql> SELECT * FROM t;
+------+
| a    |
+------+
|    1 |
+------+
1 row in set (0.01 sec)

mysql> FLASHBACK CLUSTER TO TIMESTAMP '2022-09-28 17:24:16';
Query OK, 0 rows affected (0.20 sec)

mysql> SELECT * FROM t;
Empty set (0.00 sec)
```

以下の例では、特定のTSOにクラスタをフラッシュバックして、誤って削除されたデータを正確に復元する方法を示しています。

```sql
mysql> INSERT INTO t VALUES (1);
Query OK, 1 row affected (0.02 sec)

mysql> SELECT * FROM t;
+------+
| a    |
+------+
|    1 |
+------+
1 row in set (0.01 sec)


mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@tidb_current_ts;  -- Get the current TSO
+--------------------+
| @@tidb_current_ts  |
+--------------------+
| 446113975683252225 |
+--------------------+
1 row in set (0.00 sec)

mysql> ROLLBACK;
Query OK, 0 rows affected (0.00 sec)


mysql> DELETE FROM t;
Query OK, 1 rows affected (0.00 sec)


mysql> FLASHBACK CLUSTER TO TSO 446113975683252225;
Query OK, 0 rows affected (0.20 sec)

mysql> SELECT * FROM t;
+------+
| a    |
+------+
|    1 |
+------+
1 row in set (0.01 sec)
```

もし`FLASHBACK`ステートメントで指定された時間に完全に実行されていないDDLステートメントがある場合、`FLASHBACK`ステートメントは失敗します：

```sql
mysql> ALTER TABLE t ADD INDEX k(a);
Query OK, 0 rows affected (0.56 sec)

mysql> ADMIN SHOW DDL JOBS 1;
+--------+---------+-----------------------+------------------------+--------------+-----------+----------+-----------+---------------------+---------------------+---------------------+--------+
| JOB_ID | DB_NAME | TABLE_NAME            | JOB_TYPE               | SCHEMA_STATE | SCHEMA_ID | TABLE_ID | ROW_COUNT | CREATE_TIME         | START_TIME          | END_TIME            | STATE  |
+--------+---------+-----------------------+------------------------+--------------+-----------+----------+-----------+---------------------+---------------------+---------------------+--------+
|     84 | test    | t                     | add index /* ingest */ | public       |         2 |       82 |         0 | 2023-01-29 14:33:11 | 2023-01-29 14:33:11 | 2023-01-29 14:33:12 | synced |
+--------+---------+-----------------------+------------------------+--------------+-----------+----------+-----------+---------------------+---------------------+---------------------+--------+
1 rows in set (0.01 sec)

mysql> FLASHBACK CLUSTER TO TIMESTAMP '2023-01-29 14:33:12';
ERROR 1105 (HY000): Detected another DDL job at 2023-01-29 14:33:12 +0800 CST, can't do flashback
```

ログを通じて、`FLASHBACK`の実行進捗を取得できます。以下は例です。

```
[2022/10/09 17:25:59.316 +08:00] [INFO] [cluster.go:463] ["flashback cluster stats"] ["complete regions"=9] ["total regions"=10] []
```

## MySQLの互換性 {#mysql-compatibility}

この文は、MySQLの構文に対するTiDBの拡張です。
