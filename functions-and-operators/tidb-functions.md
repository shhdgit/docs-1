---
title: TiDB Specific Functions
summary: Learn about the usage of TiDB specific functions.
---

# TiDB特定の関数 {#tidb-specific-functions}

以下の関数はTiDBの拡張機能であり、MySQLには存在しません。

<CustomContent platform="tidb">

| 関数名                                                                                | 関数の説明                                                                                                                                                                                                                                                                                        |
| :--------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TIDB_BOUNDED_STALENESS()`                                                         | `TIDB_BOUNDED_STALENESS`関数は、TiDBにデータをできるだけ新しく読み取るよう指示します。詳細はこちら：[`AS OF TIMESTAMP`句を使用した過去のデータの読み取り](/as-of-timestamp.md)                                                                                                                                                                    |
| [`TIDB_DECODE_KEY(str)`](#tidb_decode_key)                                         | `TIDB_DECODE_KEY`関数は、TiDBでエンコードされたキーエントリを`_tidb_rowid`と`table_id`を含むJSON構造にデコードするために使用できます。これらのエンコードされたキーは、一部のシステムテーブルやログ出力で見つけることができます。                                                                                                                                                     |
| [`TIDB_DECODE_PLAN(str)`](#tidb_decode_plan)                                       | `TIDB_DECODE_PLAN`関数は、TiDBの実行プランをデコードするために使用できます。                                                                                                                                                                                                                                            |
| `TIDB_IS_DDL_OWNER()`                                                              | `TIDB_IS_DDL_OWNER`関数は、接続しているTiDBインスタンスがDDLオーナーであるかどうかを確認するために使用できます。DDLオーナーは、クラスタ内の他のすべてのノードを代表してDDLステートメントを実行するTiDBインスタンスです。                                                                                                                                                               |
| [`TIDB_PARSE_TSO(num)`](#tidb_parse_tso)                                           | `TIDB_PARSE_TSO`関数は、TiDBのTSOタイムスタンプから物理タイムスタンプを抽出するために使用できます。詳細はこちら：[`tidb_current_ts`](/system-variables.md#tidb_current_ts)                                                                                                                                                                |
| [`TIDB_VERSION()`](#tidb_version)                                                  | `TIDB_VERSION`関数は、追加のビルド情報を含むTiDBのバージョンを返します。                                                                                                                                                                                                                                                |
| [`TIDB_DECODE_SQL_DIGESTS(digests, stmtTruncateLength)`](#tidb_decode_sql_digests) | `TIDB_DECODE_SQL_DIGESTS()`関数は、クラスタ内のSQLダイジェストに対応する正規化されたSQLステートメント（フォーマットや引数のない形式）をクエリするために使用されます。                                                                                                                                                                                          |
| `VITESS_HASH(str)`                                                                 | `VITESS_HASH`関数は、Vitessの`HASH`関数と互換性のある文字列のハッシュを返します。これはVitessからのデータ移行を支援することを意図しています。                                                                                                                                                                                                       |
| `TIDB_SHARD()`                                                                     | `TIDB_SHARD`関数は、インデックスのホットスポットを散らすためのシャードインデックスを作成するために使用できます。シャードインデックスは、プレフィックスとして`TIDB_SHARD`関数を持つ式インデックスです。                                                                                                                                                                              |
| `TIDB_ROW_CHECKSUM()`                                                              | `TIDB_ROW_CHECKSUM`関数は、行のチェックサム値をクエリするために使用されます。この関数はFastPlanプロセス内の`SELECT`ステートメントでのみ使用できます。つまり、`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id = ?`や`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id IN (?, ?, ...)`のようなステートメントを通じてクエリできます。詳細はこちら：[単一行データのデータ整合性検証](/ticdc/ticdc-integrity-check.md) |
| `CURRENT_RESOURCE_GROUP()`                                                         | `CURRENT_RESOURCE_GROUP`関数は、現在のセッションがバインドされているリソースグループ名を返すために使用されます。詳細はこちら：[リソース制御を使用してリソースの分離を実現する](/tidb-resource-control.md)                                                                                                                                                              |

</CustomContent>

<CustomContent platform="tidb-cloud">

| 関数名                                                                                | 関数の説明                                                                                                                                                                                                                                                                                                  |
| :--------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TIDB_BOUNDED_STALENESS()`                                                         | `TIDB_BOUNDED_STALENESS`関数は、TiDBにデータをできるだけ新しく読み取るよう指示します。詳細はこちら：[`AS OF TIMESTAMP`句を使用した過去のデータの読み取り](/as-of-timestamp.md)                                                                                                                                                                              |
| [`TIDB_DECODE_KEY(str)`](#tidb_decode_key)                                         | `TIDB_DECODE_KEY`関数は、TiDBでエンコードされたキーエントリを`_tidb_rowid`と`table_id`を含むJSON構造にデコードするために使用できます。これらのエンコードされたキーは、一部のシステムテーブルやログ出力で見つけることができます。                                                                                                                                                               |
| [`TIDB_DECODE_PLAN(str)`](#tidb_decode_plan)                                       | `TIDB_DECODE_PLAN`関数は、TiDBの実行プランをデコードするために使用できます。                                                                                                                                                                                                                                                      |
| `TIDB_IS_DDL_OWNER()`                                                              | `TIDB_IS_DDL_OWNER`関数は、接続しているTiDBインスタンスがDDLオーナーであるかどうかを確認するために使用できます。DDLオーナーは、クラスタ内の他のすべてのノードを代表してDDLステートメントを実行するTiDBインスタンスです。                                                                                                                                                                         |
| [`TIDB_PARSE_TSO(num)`](#tidb_parse_tso)                                           | `TIDB_PARSE_TSO`関数は、TiDBのTSOタイムスタンプから物理タイムスタンプを抽出するために使用できます。詳細はこちら：[`tidb_current_ts`](/system-variables.md#tidb_current_ts)                                                                                                                                                                          |
| [`TIDB_VERSION()`](#tidb_version)                                                  | `TIDB_VERSION`関数は、追加のビルド情報を含むTiDBのバージョンを返します。                                                                                                                                                                                                                                                          |
| [`TIDB_DECODE_SQL_DIGESTS(digests, stmtTruncateLength)`](#tidb_decode_sql_digests) | `TIDB_DECODE_SQL_DIGESTS()`関数は、クラスタ内のSQLダイジェストに対応する正規化されたSQLステートメント（フォーマットや引数のない形式）をクエリするために使用されます。                                                                                                                                                                                                    |
| `VITESS_HASH(str)`                                                                 | `VITESS_HASH`関数は、Vitessの`HASH`関数と互換性のある文字列のハッシュを返します。これはVitessからのデータ移行を支援することを意図しています。                                                                                                                                                                                                                 |
| `TIDB_SHARD()`                                                                     | `TIDB_SHARD`関数は、インデックスのホットスポットを散らすためのシャードインデックスを作成するために使用できます。シャードインデックスは、プレフィックスとして`TIDB_SHARD`関数を持つ式インデックスです。                                                                                                                                                                                        |
| `TIDB_ROW_CHECKSUM()`                                                              | `TIDB_ROW_CHECKSUM`関数は、行のチェックサム値をクエリするために使用されます。この関数はFastPlanプロセス内の`SELECT`ステートメントでのみ使用できます。つまり、`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id = ?`や`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id IN (?, ?, ...)`のようなステートメントを通じてクエリできます。詳細はこちら：<https://docs.pingcap.com/tidb/stable/ticdc-integrity-check> |
| `CURRENT_RESOURCE_GROUP()`                                                         | `CURRENT_RESOURCE_GROUP`関数は、現在のセッションがバインドされているリソースグループ名を返すために使用されます。詳細はこちら：[リソース制御を使用してリソースの分離を実現する](/tidb-resource-control.md)                                                                                                                                                                        |

</CustomContent>

## 例 {#examples}

このセクションでは、上記のいくつかの関数の例を提供します。

### TIDB\_DECODE\_KEY {#tidb-decode-key}

次の例では、テーブル`t1`にはTiDBによって生成された非表示の`rowid`があります。ステートメントで`TIDB_DECODE_KEY`が使用されています。結果から、非表示の`rowid`がデコードされて出力されることがわかります。

```sql
SELECT START_KEY, TIDB_DECODE_KEY(START_KEY) FROM information_schema.tikv_region_status WHERE table_name='t1' AND REGION_ID=2\G
```

```sql
*************************** 1. row ***************************
                 START_KEY: 7480000000000000FF3B5F728000000000FF1DE3F10000000000FA
TIDB_DECODE_KEY(START_KEY): {"_tidb_rowid":1958897,"table_id":"59"}
1 row in set (0.00 sec)
```

以下の例では、テーブル `t2` に複合クラスタードプライマリキーがあります。JSON出力から、プライマリキーの一部である両方の列の名前と値を含む `handle` を見ることができます。

```sql
SHOW CREATE TABLE t2\G
```

```sql
*************************** 1. row ***************************
       Table: t2
Create Table: CREATE TABLE `t2` (
  `id` binary(36) NOT NULL,
  `a` tinyint(3) unsigned NOT NULL,
  `v` varchar(512) DEFAULT NULL,
  PRIMARY KEY (`a`,`id`) /*T![clustered_index] CLUSTERED */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
1 row in set (0.001 sec)
```

```sql
SELECT * FROM information_schema.tikv_region_status WHERE table_name='t2' LIMIT 1\G
```

```sql
*************************** 1. row ***************************
                REGION_ID: 48
                START_KEY: 7480000000000000FF3E5F720400000000FF0000000601633430FF3338646232FF2D64FF3531632D3131FF65FF622D386337352DFFFF3830653635303138FFFF61396265000000FF00FB000000000000F9
                  END_KEY:
                 TABLE_ID: 62
                  DB_NAME: test
               TABLE_NAME: t2
                 IS_INDEX: 0
                 INDEX_ID: NULL
               INDEX_NAME: NULL
           EPOCH_CONF_VER: 1
            EPOCH_VERSION: 38
            WRITTEN_BYTES: 0
               READ_BYTES: 0
         APPROXIMATE_SIZE: 136
         APPROXIMATE_KEYS: 479905
  REPLICATIONSTATUS_STATE: NULL
REPLICATIONSTATUS_STATEID: NULL
1 row in set (0.005 sec)
```

```sql
SELECT tidb_decode_key('7480000000000000FF3E5F720400000000FF0000000601633430FF3338646232FF2D64FF3531632D3131FF65FF622D386337352DFFFF3830653635303138FFFF61396265000000FF00FB000000000000F9');
```

```sql
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tidb_decode_key('7480000000000000FF3E5F720400000000FF0000000601633430FF3338646232FF2D64FF3531632D3131FF65FF622D386337352DFFFF3830653635303138FFFF61396265000000FF00FB000000000000F9') |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| {"handle":{"a":"6","id":"c4038db2-d51c-11eb-8c75-80e65018a9be"},"table_id":62}                                                                                                        |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.001 sec)
```

最初のテーブルのリージョンは、テーブルの`table_id`のみを持つキーで始まります。テーブルの最後のリージョンは、`table_id + 1`で終わります。その間の任意のリージョンには、`_tidb_rowid`または`handle`を含むより長いキーがあります。

```sql
SELECT
  TABLE_NAME,
  TIDB_DECODE_KEY(START_KEY),
  TIDB_DECODE_KEY(END_KEY)
FROM
  information_schema.TIKV_REGION_STATUS
WHERE
  TABLE_NAME='stock'
  AND IS_INDEX=0
ORDER BY
  START_KEY;
```

```sql
+------------+-----------------------------------------------------------+-----------------------------------------------------------+
| TABLE_NAME | TIDB_DECODE_KEY(START_KEY)                                | TIDB_DECODE_KEY(END_KEY)                                  |
+------------+-----------------------------------------------------------+-----------------------------------------------------------+
| stock      | {"table_id":143}                                          | {"handle":{"s_i_id":"32485","s_w_id":"3"},"table_id":143} |
| stock      | {"handle":{"s_i_id":"32485","s_w_id":"3"},"table_id":143} | {"handle":{"s_i_id":"64964","s_w_id":"5"},"table_id":143} |
| stock      | {"handle":{"s_i_id":"64964","s_w_id":"5"},"table_id":143} | {"handle":{"s_i_id":"97451","s_w_id":"7"},"table_id":143} |
| stock      | {"handle":{"s_i_id":"97451","s_w_id":"7"},"table_id":143} | {"table_id":145}                                          |
+------------+-----------------------------------------------------------+-----------------------------------------------------------+
4 rows in set (0.031 sec)
```

`TIDB_DECODE_KEY`は成功した場合に有効なJSONを返し、デコードに失敗した場合は引数の値を返します。

### TIDB\_DECODE\_PLAN {#tidb-decode-plan}

遅いクエリログでエンコードされた形式でTiDB実行計画を見つけることができます。その後、`TIDB_DECODE_PLAN()`関数を使用してエンコードされた計画を人間が読める形式にデコードします。

この関数は、計画がステートメントが実行された時点でキャプチャされるため便利です。`EXPLAIN`でステートメントを再実行すると、データの分布や統計が時間とともに進化するため、異なる結果が生じる可能性があります。

```sql
SELECT tidb_decode_plan('8QIYMAkzMV83CQEH8E85LjA0CWRhdGE6U2VsZWN0aW9uXzYJOTYwCXRpbWU6NzEzLjHCtXMsIGxvb3BzOjIsIGNvcF90YXNrOiB7bnVtOiAxLCBtYXg6IDU2OC41wgErRHByb2Nfa2V5czogMCwgcnBjXxEpAQwFWBAgNTQ5LglZyGNvcHJfY2FjaGVfaGl0X3JhdGlvOiAwLjAwfQkzLjk5IEtCCU4vQQoxCTFfNgkxXzAJMwm2SGx0KHRlc3QudC5hLCAxMDAwMCkNuQRrdgmiAHsFbBQzMTMuOMIBmQnEDDk2MH0BUgEEGAoyCTQzXzUFVwX1oGFibGU6dCwga2VlcCBvcmRlcjpmYWxzZSwgc3RhdHM6cHNldWRvCTk2ISE2aAAIMTUzXmYA')\G
```

```sql
*************************** 1. row ***************************
  tidb_decode_plan('8QIYMAkzMV83CQEH8E85LjA0CWRhdGE6U2VsZWN0aW9uXzYJOTYwCXRpbWU6NzEzLjHCtXMsIGxvb3BzOjIsIGNvcF90YXNrOiB7bnVtOiAxLCBtYXg6IDU2OC41wgErRHByb2Nfa2V5czogMCwgcnBjXxEpAQwFWBAgNTQ5LglZyGNvcHJfY2FjaGVfaGl0X3JhdGlvOiAwLjAwfQkzLjk5IEtCCU4vQQoxCTFfNgkxXz:     id                     task         estRows    operator info                              actRows    execution info                                                                                                                         memory     disk
    TableReader_7          root         319.04     data:Selection_6                           960        time:713.1µs, loops:2, cop_task: {num: 1, max: 568.5µs, proc_keys: 0, rpc_num: 1, rpc_time: 549.1µs, copr_cache_hit_ratio: 0.00}    3.99 KB    N/A
    └─Selection_6          cop[tikv]    319.04     lt(test.t.a, 10000)                        960        tikv_task:{time:313.8µs, loops:960}                                                                                                   N/A        N/A
      └─TableFullScan_5    cop[tikv]    960        table:t, keep order:false, stats:pseudo    960        tikv_task:{time:153µs, loops:960}                                                                                                     N/A        N/A
```

### TIDB\_PARSE\_TSO {#tidb-parse-tso}

`TIDB_PARSE_TSO`関数は、TiDB TSOタイムスタンプから物理タイムスタンプを抽出するために使用できます。TSOはTime Stamp Oracleの略であり、PD（Placement Driver）によって各トランザクションに対して与えられる単調増加タイムスタンプです。

TSOは2つの部分で構成される番号です：

- 物理タイムスタンプ
- 論理カウンタ

**Note:** コードブロック内の内容は翻訳しないでください。

```sql
BEGIN;
SELECT TIDB_PARSE_TSO(@@tidb_current_ts);
ROLLBACK;
```

```sql
+-----------------------------------+
| TIDB_PARSE_TSO(@@tidb_current_ts) |
+-----------------------------------+
| 2021-05-26 11:33:37.776000        |
+-----------------------------------+
1 row in set (0.0012 sec)
```

ここでは、`TIDB_PARSE_TSO`が使用され、`tidb_current_ts`セッション変数で利用可能なタイムスタンプ番号から物理タイムスタンプを抽出するために使用されます。タイムスタンプはトランザクションごとに与えられるため、この関数はトランザクション内で実行されます。

### TIDB\_VERSION {#tidb-version}

`TIDB_VERSION`関数は、接続しているTiDBサーバーのバージョンとビルドの詳細を取得するために使用できます。GitHubで問題を報告する際にこの関数を使用できます。

```sql
SELECT TIDB_VERSION()\G
```

```sql
*************************** 1. row ***************************
TIDB_VERSION(): Release Version: v5.1.0-alpha-13-gd5e0ed0aa-dirty
Edition: Community
Git Commit Hash: d5e0ed0aaed72d2f2dfe24e9deec31cb6cb5fdf0
Git Branch: master
UTC Build Time: 2021-05-24 14:39:20
GoVersion: go1.13
Race Enabled: false
TiKV Min Version: v3.0.0-60965b006877ca7234adaced7890d7b029ed1306
Check Table Before Drop: false
1 row in set (0.00 sec)
```

### TIDB\_DECODE\_SQL\_DIGESTS {#tidb-decode-sql-digests}

`TIDB_DECODE_SQL_DIGESTS（）`関数は、クラスタ内のSQLダイジェストに対応する正規化されたSQLステートメント（フォーマットや引数のない形式）をクエリするために使用されます。この関数は1つまたは2つの引数を受け入れます。

- `digests`：文字列。このパラメータはJSON文字列配列の形式であり、配列内の各文字列はSQLダイジェストです。
- `stmtTruncateLength`：整数（オプション）。これは、返された結果の各SQLステートメントの長さを制限するために使用されます。指定された長さを超えるSQLステートメントは切り捨てられます。 `0`は長さが制限されていないことを意味します。

この関数は、JSON文字列配列の形式で文字列を返します。配列内の*i*番目のアイテムは、`digests`パラメータの*i*番目の要素に対応する正規化されたSQLステートメントです。`digests`パラメータの要素が有効なSQLダイジェストでないか、システムが対応するSQLステートメントを見つけられない場合、返された結果の対応するアイテムは`null`です。切り捨て長が指定されている場合（`stmtTruncateLength > 0`）、返された結果の各ステートメントがこの長さを超える場合、最初の`stmtTruncateLength`文字が保持され、末尾に切り捨てを示す`"..."`が追加されます。`digests`パラメータが`NULL`の場合、関数の返り値は`NULL`です。

> **Note:**
>
> - [PROCESS](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process)権限を持つユーザーのみがこの関数を使用できます。
> - `TIDB_DECODE_SQL_DIGESTS`が実行されると、TiDBはステートメントサマリーテーブルから各SQLダイジェストに対応するステートメントをクエリします。そのため、任意のSQLダイジェストに対応するステートメントが常に見つかる保証はありません。クラスタで実行されたステートメントのみが見つかり、これらのSQLステートメントがクエリできるかどうかは、ステートメントサマリーテーブルの関連する構成にも影響を受けます。ステートメントサマリーテーブルの詳細な説明については、[ステートメントサマリーテーブル](/statement-summary-tables.md)を参照してください。
> - この関数には高いオーバーヘッドがあります。大量の行を持つクエリ（たとえば、大規模で多忙なクラスタの`information_schema.cluster_tidb_trx`のフルテーブルをクエリする場合）では、この関数を使用するとクエリが長すぎる可能性があります。注意して使用してください。
>   - この関数には高いオーバーヘッドがあります。なぜなら、呼び出されるたびに`STATEMENTS_SUMMARY`、`STATEMENTS_SUMMARY_HISTORY`、`CLUSTER_STATEMENTS_SUMMARY`、および`CLUSTER_STATEMENTS_SUMMARY_HISTORY`テーブルを内部的にクエリし、クエリには`UNION`操作が含まれるからです。この関数は現在、ベクトル化をサポートしていないため、複数のデータ行に対してこの関数を呼び出すと、上記のクエリがそれぞれの行に対して個別に実行されます。

```sql
set @digests = '["e6f07d43b5c21db0fbb9a31feac2dc599787763393dd5acbfad80e247eb02ad5","38b03afa5debbdf0326a014dbe5012a62c51957f1982b3093e748460f8b00821","e5796985ccafe2f71126ed6c0ac939ffa015a8c0744a24b7aee6d587103fd2f7"]';

select tidb_decode_sql_digests(@digests);
```

```sql
+------------------------------------+
| tidb_decode_sql_digests(@digests)  |
+------------------------------------+
| ["begin",null,"select * from `t`"] |
+------------------------------------+
1 row in set (0.00 sec)
```

**Note:** 上記の例では、パラメータは3つのSQLダイジェストを含むJSON配列であり、対応するSQLステートメントはクエリ結果の3つのアイテムです。ただし、2番目のSQLダイジェストに対応するSQLステートメントはクラスタから見つかりませんので、結果の2番目のアイテムは `null` です。

```sql
select tidb_decode_sql_digests(@digests, 10);
```

```sql
+---------------------------------------+
| tidb_decode_sql_digests(@digests, 10) |
+---------------------------------------+
| ["begin",null,"select * f..."]        |
+---------------------------------------+
1 row in set (0.01 sec)
```

上記の呼び出しでは、第2パラメータ（すなわち切り捨て長）を10として指定し、クエリ結果の第3文の長さが10よりも大きいため、最初の10文字のみが保持され、末尾に`"..."`が追加され、これは切り捨てを示しています。

参照も次を参照してください：

- [`ステートメントサマリーテーブル`](/statement-summary-tables.md)
- [`INFORMATION_SCHEMA.TIDB_TRX`](/information-schema/information-schema-tidb-trx.md)

### TIDB\_SHARD {#tidb-shard}

`TIDB_SHARD`関数は、インデックスのホットスポットを散乱させるためのシャードインデックスを作成するために使用できます。シャードインデックスは、`TIDB_SHARD`関数で接頭辞が付けられた式インデックスです。

- 作成：

  インデックスフィールド`a`のためのシャードインデックスを作成するには、`uk((tidb_shard(a)), a))`を使用できます。ユニークセカンダリインデックス`uk((tidb_shard(a)), a))`のインデックスフィールド`a`に単調増加または単調減少するデータによってホットスポットが引き起こされる場合、インデックスの接頭辞`tidb_shard(a)`はホットスポットを散乱させてクラスターの拡張性を向上させることができます。

- シナリオ：

  - ユニークセカンダリインデックスに単調増加または単調減少するキーによって引き起こされる書き込みホットスポットがあり、インデックスには整数型フィールドが含まれている。
  - SQLステートメントがセカンダリインデックスのすべてのフィールドに基づいて等価クエリを実行し、それが別々の`SELECT`としてまたは`UPDATE`、`DELETE`などによって生成された内部クエリとして実行される場合。等価クエリには2つの方法があります：`a = 1`または`a IN (1, 2, ......)`。

- 制限：

  - 不等号クエリで使用できません。
  - 最外部の`AND`演算子と混在した`OR`を含むクエリで使用できません。
  - `GROUP BY`句で使用できません。
  - `ORDER BY`句で使用できません。
  - `ON`句で使用できません。
  - `WHERE`サブクエリで使用できません。
  - 整数フィールドのユニークインデックスのみに使用できます。
  - 複合インデックスで効果がない場合があります。
  - FastPlanプロセスを通過できず、オプティマイザのパフォーマンスに影響を与える可能性があります。
  - 実行計画キャッシュを準備するために使用できません。

次の例は、`TIDB_SHARD`関数の使用方法を示しています。

- `TIDB_SHARD`関数を使用してSHARD値を計算します。

  次のステートメントは、`TIDB_SHARD`関数を使用して`12373743746`のSHARD値を計算する方法を示しています：

  ```sql
  SELECT TIDB_SHARD(12373743746);
  ```

- SHARD値は：

  ```sql
  +-------------------------+
  | TIDB_SHARD(12373743746) |
  +-------------------------+
  |                     184 |
  +-------------------------+
  1行がセットされました (0.00秒)

  ```

- `TIDB_SHARD`関数を使用してシャードインデックスを作成します：

  ```sql
  CREATE TABLE test(id INT PRIMARY KEY CLUSTERED, a INT, b INT, UNIQUE KEY uk((tidb_shard(a)), a));
  ```

### TIDB\_ROW\_CHECKSUM {#tidb-row-checksum}

`TIDB_ROW_CHECKSUM`関数は、行のチェックサム値をクエリするために使用されます。この関数は、FastPlanプロセス内でのみ`SELECT`ステートメントで使用できます。つまり、`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id = ?`や`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id IN (?, ?, ...)`などのステートメントを通じてクエリできます。

TiDBの単一行データのチェックサム機能を有効にするには（システム変数[`tidb_enable_row_level_checksum`](/system-variables.md#tidb_enable_row_level_checksum-new-in-v710)で制御されます）、次のステートメントを実行します："}

```sql
SET GLOBAL tidb_enable_row_level_checksum = ON;
```

{ "input\_content": "テーブル `t` を作成し、データを挿入します:" }

```sql
USE test;
CREATE TABLE t (id INT PRIMARY KEY, k INT, c int);
INSERT INTO TABLE t values (1, 10, a);
```

次の文は、テーブル「t」で `id = 1` の行のチェックサム値をクエリする方法を示しています。

```sql
SELECT *, TIDB_ROW_CHECKSUM() FROM t WHERE id = 1;
```

The output is as follows:

```sql
+----+------+------+---------------------+
| id | k    | c    | TIDB_ROW_CHECKSUM() |
+----+------+------+---------------------+
|  1 |   10 | a    | 3813955661          |
+----+------+------+---------------------+
1 row in set (0.000 sec)
```

### CURRENT\_RESOURCE\_GROUP {#current-resource-group}

`CURRENT_RESOURCE_GROUP`関数は、現在のセッションがバインドされているリソースグループの名前を表示するために使用されます。[リソース制御](/tidb-resource-control.md)機能が有効になっている場合、SQLステートメントで使用できる利用可能なリソースは、バインドされたリソースグループのリソースクォータによって制限されます。

セッションが確立されると、TiDBはデフォルトでログインユーザーがバインドされているリソースグループにセッションをバインドします。ユーザーがリソースグループにバインドされていない場合、セッションは`default`リソースグループにバインドされます。セッションが確立されると、デフォルトでバインドされたリソースグループは変更されません。たとえユーザーのバインドされたリソースグループが[ユーザーにバインドされたリソースグループの変更](/sql-statements/sql-statement-alter-user.md#modify-basic-user-information)を介して変更された場合でもです。現在のセッションのバインドされたリソースグループを変更するには、[`SET RESOURCE GROUP`](/sql-statements/sql-statement-set-resource-group.md)を使用できます。

#### 例 {#example}

ユーザー`user1`を作成し、2つのリソースグループ`rg1`と`rg2`を作成し、ユーザー`user1`をリソースグループ`rg1`にバインドします。

```sql
CREATE USER 'user1';
CREATE RESOURCE GROUP 'rg1' RU_PER_SEC = 1000;
CREATE RESOURCE GROUP 'rg2' RU_PER_SEC = 2000;
ALTER USER 'user1' RESOURCE GROUP `rg1`;
```

以下は、現在のユーザーにバインドされたリソースグループを表示するために `user1` を使用してログインしてください。

```sql
SELECT CURRENT_RESOURCE_GROUP();
```

```
+--------------------------+
| CURRENT_RESOURCE_GROUP() |
+--------------------------+
| rg1                      |
+--------------------------+
1 row in set (0.00 sec)
```

実行`SET RESOURCE GROUP`現在のセッションのリソースグループを`rg2`に設定し、次に現在のユーザーにバインドされたリソースグループを表示します。

```sql
SET RESOURCE GROUP `rg2`;
SELECT CURRENT_RESOURCE_GROUP();
```

```
+--------------------------+
| CURRENT_RESOURCE_GROUP() |
+--------------------------+
| rg2                      |
+--------------------------+
1 row in set (0.00 sec)
```
