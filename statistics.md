---
title: Introduction to Statistics
summary: Learn how the statistics collect table-level and column-level information.
---

# 統計の紹介 {#introduction-to-statistics}

TiDBは統計を使用して、[どのインデックスを選択するか](/choose-index.md)を決定します。

## 統計のバージョン {#versions-of-statistics}

`tidb_analyze_version`変数は、TiDBによって収集される統計を制御します。現在、2つの統計バージョンがサポートされています：`tidb_analyze_version = 1`と`tidb_analyze_version = 2`。

- TiDB Self-Hostedの場合、この変数のデフォルト値はv5.3.0から`1`から`2`に変更されます。
- TiDB Cloudの場合、この変数のデフォルト値はv6.5.0から`1`から`2`に変更されます。
- クラスターが以前のバージョンからアップグレードされた場合、`tidb_analyze_version`のデフォルト値はアップグレード後も変更されません。

バージョン1と比較して、バージョン2の統計は、データボリュームが膨大な場合にハッシュ衝突によって引き起こされる潜在的な不正確さを回避します。また、ほとんどのシナリオで推定精度を維持します。

これらの2つのバージョンには、TiDBで異なる情報が含まれています：

| 情報                       | バージョン1                          | バージョン2                                                    |
| ------------------------ | ------------------------------- | --------------------------------------------------------- |
| テーブル内の行の総数               | √                               | √                                                         |
| カラム数のCount-Min Sketch    | √                               | ×                                                         |
| インデックス数のCount-Min Sketch | √                               | ×                                                         |
| カラムのTop-N                | √                               | √（メンテナンス方法と精度が向上）                                         |
| インデックスのTop-N             | √（メンテナンス精度が不十分な場合、不正確さが引き起こされる） | √（メンテナンス方法と精度が向上）                                         |
| カラムのヒストグラム               | √                               | √（ヒストグラムにはTop-Nの値が含まれません。）                                |
| インデックスのヒストグラム            | √                               | √（ヒストグラムのバケットは、各バケット内の異なる値の数を記録し、ヒストグラムにはTop-Nの値が含まれません。） |
| カラム内の`NULL`の数            | √                               | √                                                         |
| インデックス内の`NULL`の数         | √                               | √                                                         |
| カラムの平均長さ                 | √                               | √                                                         |
| インデックスの平均長さ              | √                               | √                                                         |

`tidb_analyze_version = 2`の場合、`ANALYZE`の実行後にメモリオーバーフローが発生した場合、`tidb_analyze_version = 1`に設定してバージョン1にフォールバックし、次の操作のいずれかを実行する必要があります：

- `ANALYZE`ステートメントが手動で実行される場合、分析するテーブルごとに手動で分析します。

  ```sql
  SELECT DISTINCT(CONCAT('ANALYZE TABLE ', table_schema, '.', table_name, ';')) FROM information_schema.tables, mysql.stats_histograms WHERE stats_ver = 2 AND table_id = tidb_table_id;
  ```

- TiDBが自動的に`ANALYZE`ステートメントを実行する場合、`DROP STATS`ステートメントを生成する次のステートメントを実行します。

  ```sql
  SELECT DISTINCT(CONCAT('DROP STATS ', table_schema, '.', table_name, ';')) FROM information_schema.tables, mysql.stats_histograms WHERE stats_ver = 2 AND table_id = tidb_table_id;
  ```

- 前述のステートメントの結果がコピー＆ペーストするには長すぎる場合、結果を一時的なテキストファイルにエクスポートし、次のようにファイルから実行します：

  ```sql
  SELECT DISTINCT ... INTO OUTFILE '/tmp/sql.txt';
  mysql -h ${TiDB_IP} -u user -P ${TIDB_PORT} ... < '/tmp/sql.txt'
  ```

このドキュメントは、ヒストグラム、Count-Min Sketch、およびTop-Nの簡単な紹介を行い、統計の収集とメンテナンスの詳細を説明します。

## ヒストグラム {#histogram}

ヒストグラムは、データの分布の近似表現です。値の全範囲をいくつかのバケットに分割し、各バケットを説明するために単純なデータを使用します。TiDBでは、各テーブルの特定のカラムに対して等深度のヒストグラムが作成されます。等深度のヒストグラムは、間隔クエリを推定するために使用できます。

ここでの「等深度」とは、各バケットに含まれる値の数が可能な限り等しいことを意味します。たとえば、与えられたセット{1.6, 1.9, 1.9, 2.0, 2.4, 2.6, 2.7, 2.7, 2.8, 2.9, 3.4, 3.5}に対して4つのバケットを生成したい場合、等深度のヒストグラムは次のようになります。それは、\[1.6, 1.9]、\[2.0, 2.6]、\[2.7, 2.8]、\[2.9, 3.5]の4つのバケットを含んでいます。バケットの深さは3です。

![等深度ヒストグラムの例](/media/statistics-1.png)

ヒストグラムバケットの上限を決定するパラメータの詳細については、[Manual Collection](#manual-collection)を参照してください。バケットの数が多いほど、ヒストグラムの精度が高くなりますが、高い精度はメモリリソースの使用にコストがかかります。実際のシナリオに応じて、この数値を適切に調整できます。

## Count-Min Sketch {#count-min-sketch}

Count-Min Sketchはハッシュ構造です。等価クエリに`a = 1`または`IN`クエリ（たとえば、`a in (1, 2, 3)`）が含まれる場合、TiDBはこのデータ構造を使用します。

Count-Min Sketchはハッシュ構造であるため、ハッシュ衝突が発生する可能性があります。`EXPLAIN`ステートメントでは、等価クエリの推定値が実際の値から大きく逸脱する場合、大きな値と小さな値が一緒にハッシュ化されたと考えられます。この場合、ハッシュ衝突を回避するために次の方法のいずれかを取ることができます：

- `WITH NUM TOPN`パラメータを変更します。TiDBは高頻度（トップx）のデータを別々に保存し、他のデータをCount-Min Sketchに保存します。したがって、大きな値と小さな値が一緒にハッシュ化されるのを防ぐために、`WITH NUM TOPN`の値を増やすことができます。TiDBでは、デフォルト値は20です。最大値は1024です。このパラメータについての詳細は、[Full Collection](#full-collection)を参照してください。
- 2つのパラメータ`WITH NUM CMSKETCH DEPTH`および`WITH NUM CMSKETCH WIDTH`を変更します。両方ともハッシュバケットの数と衝突確率に影響します。実際のシナリオに応じて、2つのパラメータの値を適切に増やして、ハッシュ衝突の確率を減らすことができますが、その結果として統計のメモリ使用量が増加します。TiDBでは、`WITH NUM CMSKETCH DEPTH`のデフォルト値は5で、`WITH NUM CMSKETCH WIDTH`のデフォルト値は2048です。これらの2つのパラメータについての詳細は、[Full Collection](#full-collection)を参照してください。

## Top-Nの値 {#top-n-values}

Top-Nの値は、カラムまたはインデックス内で最も頻繁に発生する上位N個の値です。TiDBはTop-Nの値とその出現回数を記録します。

## 統計の収集 {#collect-statistics}

### 手動収集 {#manual-collection}

`ANALYZE`ステートメントを実行して統計情報を収集できます。

> **Note:**
>
> TiDBでの`ANALYZE TABLE`の実行時間は、MySQLまたはInnoDBでの実行時間よりも長くなります。InnoDBでは、サンプリングされるページ数が少ないのに対し、TiDBでは包括的な統計情報が完全に再構築されます。MySQL向けに書かれたスクリプトは、`ANALYZE TABLE`が短時間で終了することを単純に期待するかもしれません。
>
> より迅速な分析のために、`tidb_enable_fast_analyze`を`1`に設定してクイック分析機能を有効にすることができます。このパラメータのデフォルト値は`0`です。
>
> クイック分析が有効になると、TiDBはデータの約10,000行をランダムにサンプリングして統計情報を作成します。したがって、データの分布が不均一であるか、データ量が比較的少ない場合、統計情報の精度は比較的低くなります。これにより、誤ったインデックスの選択など、実行計画が悪化する可能性があります。通常の`ANALYZE`ステートメントの実行時間が許容できる場合は、クイック分析機能を無効にすることをお勧めします。
>
> `tidb_enable_fast_analyze`は実験的な機能であり、現在は`tidb_analyze_version=2`の統計情報と完全に一致しません。そのため、`tidb_enable_fast_analyze`が有効になっている場合は、`tidb_analyze_version`の値を`1`に設定する必要があります。"

#### フルコレクション {#full-collection}

以下の構文を使用してフルコレクションを実行できます。

- `TableNameList`のすべてのテーブルの統計を収集するには：

  ```sql
  ANALYZE TABLE TableNameList [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
  ```

- `WITH NUM BUCKETS`は、生成されたヒストグラムの最大バケット数を指定します。

- `WITH NUM TOPN`は、生成された`TOPN`の最大数を指定します。

- `WITH NUM CMSKETCH DEPTH`は、CMスケッチの深さを指定します。

- `WITH NUM CMSKETCH WIDTH`は、CMスケッチの幅を指定します。

- `WITH NUM SAMPLES`は、サンプルの数を指定します。

- `WITH FLOAT_NUM SAMPLERATE`は、サンプリング率を指定します。

`WITH NUM SAMPLES`と`WITH FLOAT_NUM SAMPLERATE`は、異なるアルゴリズムに対応しています。

- `WITH NUM SAMPLES`は、TiDBのリザーバサンプリングメソッドで実装されたサンプリングセットのサイズを指定します。テーブルが大きい場合、このメソッドを使用して統計を収集することはお勧めしません。リザーバサンプリングの中間結果セットには冗長な結果が含まれており、メモリなどのリソースに追加の負荷をかけるためです。
- `WITH FLOAT_NUM SAMPLERATE`は、v5.3.0で導入されたサンプリングメソッドです。値の範囲が`(0, 1]`で、このパラメータはサンプリング率を指定します。これはTiDBでのベルヌーイサンプリングの方法で実装されており、より大きなテーブルのサンプリングに適しており、収集効率とリソース使用の点で優れています。

v5.3.0より前、TiDBはリザーバサンプリングメソッドを使用して統計を収集していました。v5.3.0以降、TiDBバージョン2の統計はデフォルトでベルヌーイサンプリングメソッドを使用して統計を収集します。リザーバサンプリングメソッドを再利用するには、`WITH NUM SAMPLES`ステートメントを使用できます。

現在のサンプリング率は、適応アルゴリズムに基づいて計算されます。[`SHOW STATS_META`](/sql-statements/sql-statement-show-stats-meta.md)を使用してテーブルの行数を観測できる場合、この行数を使用して、10万行に対応するサンプリング率を計算できます。この数を観測できない場合は、別の参照として[`TABLE_STORAGE_STATS`](/information-schema/information-schema-table-storage-stats.md)テーブルの`TABLE_KEYS`列を使用してサンプリング率を計算できます。

<CustomContent platform="tidb">

> **Note:**
>
> 通常、`STATS_META`は`TABLE_KEYS`よりも信頼性が高いです。ただし、[TiDB Lightning](https://docs.pingcap.com/tidb/stable/tidb-lightning-overview)などの方法でデータをインポートした後、`STATS_META`の結果は`0`になります。このような状況を処理するには、`STATS_META`の結果が`TABLE_KEYS`の結果よりもはるかに小さい場合、`TABLE_KEYS`を使用してサンプリング率を計算できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> 通常、`STATS_META`は`TABLE_KEYS`よりも信頼性が高いです。ただし、TiDB Cloudコンソール（[サンプルデータのインポート](/tidb-cloud/import-sample-data.md)を参照）を介してデータをインポートした後、`STATS_META`の結果は`0`になります。このような状況を処理するには、`STATS_META`の結果が`TABLE_KEYS`の結果よりもはるかに小さい場合、`TABLE_KEYS`を使用してサンプリング率を計算できます。

</CustomContent>

##### いくつかのカラムの統計を収集する {#collect-statistics-on-some-columns}

ほとんどの場合、SQLステートメントを実行する際、オプティマイザは（`WHERE`、`JOIN`、`ORDER BY`、`GROUP BY`ステートメントのカラムなどの）いくつかのカラムの統計のみを使用します。これらのカラムは`PREDICATE COLUMNS`と呼ばれます。

テーブルに多くのカラムがある場合、すべてのカラムの統計を収集すると大きなオーバーヘッドが発生する可能性があります。オーバーヘッドを減らすためには、オプティマイザが使用する特定のカラムまたは`PREDICATE COLUMNS`のみに統計を収集することができます。

> **Note:**
>
> いくつかのカラムの統計を収集するのは、`tidb_analyze_version = 2`にのみ適用されます。

- 特定のカラムの統計を収集するには、次の構文を使用します：

  ```sql
  ANALYZE TABLE TableName COLUMNS ColumnNameList [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
  ```

  この構文では、`ColumnNameList`は対象のカラムの名前リストを指定します。複数のカラムを指定する必要がある場合は、カンマ`,`を使用してカラム名を区切ります。例えば、`ANALYZE table t columns a, b`のようになります。この構文は、特定のテーブルの特定のカラムに加えて、同時にそのテーブルのインデックスカラムとすべてのインデックスの統計も収集します。

  > **Note:**
  >
  > 上記の構文はフルコレクションです。たとえば、この構文を使用してカラム`a`と`b`の統計を収集した後、カラム`c`の統計も収集したい場合は、`ANALYZE table t columns a, b, c`としてすべての3つのカラムを指定する必要があります。`ANALYZE TABLE t COLUMNS c`を使用して追加のカラム`c`のみを指定するのではなく、すべてのカラムを指定する必要があります。

- `PREDICATE COLUMNS`の統計を収集するには、次の手順を実行します：

  > **Warning:**
  >
  > 現在、`PREDICATE COLUMNS`の統計を収集することは実験的な機能です。本番環境で使用することはお勧めしません。

  1. [`tidb_enable_column_tracking`](/system-variables.md#tidb_enable_column_tracking-new-in-v540)システム変数の値を`ON`に設定して、TiDBが`PREDICATE COLUMNS`を収集できるようにします。

      <CustomContent platform="tidb">

     この設定後、TiDBは100 \* [`stats-lease`](/tidb-configuration-file.md#stats-lease)ごとに`mysql.column_stats_usage`システムテーブルに`PREDICATE COLUMNS`情報を書き込みます。

      </CustomContent>

      <CustomContent platform="tidb-cloud">

     この設定後、TiDBは300秒ごとに`mysql.column_stats_usage`システムテーブルに`PREDICATE COLUMNS`情報を書き込みます。

      </CustomContent>

  2. ビジネスのクエリパターンが比較的安定した後、次の構文を使用して`PREDICATE COLUMNS`の統計を収集します：

     ```sql
     ANALYZE TABLE TableName PREDICATE COLUMNS [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
     ```

     この構文は、特定のテーブルの`PREDICATE COLUMNS`に加えて、同時にそのテーブルのインデックスカラムとすべてのインデックスの統計も収集します。

     > **Note:**
     >
     > - `mysql.column_stats_usage`システムテーブルにそのテーブルの`PREDICATE COLUMNS`レコードが含まれていない場合、前述の構文はそのテーブルのすべてのカラムとすべてのインデックスの統計を収集します。
     > - この構文を使用して統計を収集した後、新しいタイプのSQLクエリを実行すると、オプティマイザは一時的に古いまたは擬似的なカラム統計を使用する場合があり、次回からTiDBは使用されたカラムの統計を収集します。

- すべてのカラムとインデックスの統計を収集するには、次の構文を使用します：

  ```sql
  ANALYZE TABLE TableName ALL COLUMNS [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
  ```

`ANALYZE`ステートメントでカラムの構成を永続化する場合（`COLUMNS ColumnNameList`、`PREDICATE COLUMNS`、`ALL COLUMNS`を含む）、`tidb_persist_analyze_options`システム変数の値を`ON`に設定して、[ANALYZE構成の永続化](#persist-analyze-configurations)機能を有効にします。ANALYZE構成の永続化機能を有効にした後：

- TiDBが自動的に統計を収集するか、最新の`ANALYZE`ステートメントでカラムの構成を指定せずに統計を手動で収集する場合、TiDBは統計の収集について以前に永続化された構成を引き続き使用します。
- カラムの構成を指定して複数回`ANALYZE`ステートメントを手動で実行すると、TiDBは最新の`ANALYZE`ステートメントで指定された新しい構成で以前に記録された永続化構成を上書きします。

`PREDICATE COLUMNS`および統計が収集されたカラムを特定するには、次の構文を使用します：

```sql
SHOW COLUMN_STATS_USAGE [ShowLikeOrWhere];
```

`SHOW COLUMN_STATS_USAGE` ステートメントは、次の6つのカラムを返します：

| カラム名               | 説明                     |
| ------------------ | ---------------------- |
| `Db_name`          | データベース名                |
| `Table_name`       | テーブル名                  |
| `Partition_name`   | パーティション名               |
| `Column_name`      | カラム名                   |
| `Last_used_at`     | クエリ最適化でカラム統計が使用された最終時刻 |
| `Last_analyzed_at` | カラム統計が収集された最終時刻        |

次の例では、`ANALYZE TABLE t PREDICATE COLUMNS;` を実行した後、TiDB は `b`、`c`、`d` のカラムについて統計を収集します。ここで、カラム `b` は `PREDICATE COLUMN` であり、カラム `c` および `d` はインデックスカラムです。

```sql
SET GLOBAL tidb_enable_column_tracking = ON;
Query OK, 0 rows affected (0.00 sec)

CREATE TABLE t (a INT, b INT, c INT, d INT, INDEX idx_c_d(c, d));
Query OK, 0 rows affected (0.00 sec)

-- The optimizer uses the statistics on column b in this query.
SELECT * FROM t WHERE b > 1;
Empty set (0.00 sec)

-- After waiting for a period of time (100 * stats-lease), TiDB writes the collected `PREDICATE COLUMNS` to mysql.column_stats_usage.
-- Specify `last_used_at IS NOT NULL` to show the `PREDICATE COLUMNS` collected by TiDB.
SHOW COLUMN_STATS_USAGE WHERE db_name = 'test' AND table_name = 't' AND last_used_at IS NOT NULL;
+---------+------------+----------------+-------------+---------------------+------------------+
| Db_name | Table_name | Partition_name | Column_name | Last_used_at        | Last_analyzed_at |
+---------+------------+----------------+-------------+---------------------+------------------+
| test    | t          |                | b           | 2022-01-05 17:21:33 | NULL             |
+---------+------------+----------------+-------------+---------------------+------------------+
1 row in set (0.00 sec)

ANALYZE TABLE t PREDICATE COLUMNS;
Query OK, 0 rows affected, 1 warning (0.03 sec)

-- Specify `last_analyzed_at IS NOT NULL` to show the columns for which statistics have been collected.
SHOW COLUMN_STATS_USAGE WHERE db_name = 'test' AND table_name = 't' AND last_analyzed_at IS NOT NULL;
+---------+------------+----------------+-------------+---------------------+---------------------+
| Db_name | Table_name | Partition_name | Column_name | Last_used_at        | Last_analyzed_at    |
+---------+------------+----------------+-------------+---------------------+---------------------+
| test    | t          |                | b           | 2022-01-05 17:21:33 | 2022-01-05 17:23:06 |
| test    | t          |                | c           | NULL                | 2022-01-05 17:23:06 |
| test    | t          |                | d           | NULL                | 2022-01-05 17:23:06 |
+---------+------------+----------------+-------------+---------------------+---------------------+
3 rows in set (0.00 sec)
```

##### 統計を収集する {#collect-statistics-on-indexes}

`TableName`の`IndexNameList`のすべてのインデックスの統計を収集するには、次の構文を使用します：

```sql
ANALYZE TABLE TableName INDEX [IndexNameList] [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
```

`IndexNameList` が空の場合、この構文は `TableName` のすべてのインデックスの統計情報を収集します。

> **Note:**
>
> 収集前後の統計情報が一貫していることを確認するために、`tidb_analyze_version` が `2` の場合、この構文はインデックスのみでなく、テーブル全体（すべてのカラムとインデックスを含む）の統計情報を収集します。

##### パーティションの統計情報を収集 {#collect-statistics-on-partitions}

- `TableName` の `PartitionNameList` のすべてのパーティションの統計情報を収集するには、次の構文を使用します。

  ```sql
  ANALYZE TABLE TableName PARTITION PartitionNameList [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
  ```

- `TableName` の `PartitionNameList` のすべてのパーティションのインデックス統計情報を収集するには、次の構文を使用します。

  ```sql
  ANALYZE TABLE TableName PARTITION PartitionNameList INDEX [IndexNameList] [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
  ```

- テーブルの一部のパーティションの一部のカラムの統計情報を収集する場合は、次の構文を使用します。

  > **Warning:**
  >
  > 現在、`PREDICATE COLUMNS` の統計情報を収集することは実験的な機能です。本番環境で使用しないでください。

  ```sql
  ANALYZE TABLE TableName PARTITION PartitionNameList [COLUMNS ColumnNameList|PREDICATE COLUMNS|ALL COLUMNS] [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
  ```

##### 動的剪定モードでのパーティションテーブルの統計情報を収集 {#collect-statistics-of-partitioned-tables-in-dynamic-pruning-mode}

[動的剪定モード](/partitioned-table.md#dynamic-pruning-mode)でパーティションテーブルにアクセスする場合、TiDB はテーブルレベルの統計情報である GlobalStats を収集します。現在、GlobalStats はすべてのパーティションの統計情報から集約されています。動的剪定モードでは、パーティションテーブルの統計情報の更新がトリガーとなります。

> **Note:**
>
> - GlobalStats 更新がトリガーされる場合：
>
>   - 一部のパーティションに統計情報がない場合（つまり、分析されたことのない新しいパーティションなど）、GlobalStats 生成が中断され、パーティションに統計情報がないという警告メッセージが表示されます。
>   - 特定のパーティションで特定のカラムの統計情報が欠落している場合（これらのパーティションで異なるカラムが指定されて分析されている）、これらのカラムの統計情報が集約されると GlobalStats 生成が中断され、特定のパーティションで特定のカラムの統計情報が欠落しているという警告メッセージが表示されます。
> - 動的剪定モードでは、パーティションとテーブルの分析構成は同じである必要があります。したがって、`ANALYZE TABLE TableName PARTITION PartitionNameList` ステートメントに続いて `COLUMNS` 構成を指定するか、`WITH` に続いて `OPTIONS` 構成を指定すると、TiDB はそれらを無視して警告を返します。

#### 増分収集 {#incremental-collection}

完全収集後の分析速度を向上させるために、増分収集を使用して、時間カラムなどの単調非減少カラムに新しく追加されたセクションを分析することができます。

> **Note:**
>
> - 現在、増分収集はインデックスのみに提供されています。
> - 増分収集を使用する場合、テーブルには `INSERT` 操作のみが存在し、インデックスカラムに新しく挿入された値が単調非減少であることを確認する必要があります。そうでない場合、統計情報が不正確になり、TiDB オプティマイザが適切な実行計画を選択するのに影響を与える可能性があります。

次の構文を使用して増分収集を実行できます。

- `TableName` のすべての `IndexNameLists` でインデックスカラムの統計情報を増分的に収集するには：

  ```sql
  ANALYZE INCREMENTAL TABLE TableName INDEX [IndexNameList] [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
  ```

- `TableName` のすべての `PartitionNameLists` のパーティションでインデックスカラムの統計情報を増分的に収集するには：

  ```sql
  ANALYZE INCREMENTAL TABLE TableName PARTITION PartitionNameList INDEX [IndexNameList] [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
  ```

### 自動更新 {#automatic-update}

<CustomContent platform="tidb">

`INSERT`、`DELETE`、または `UPDATE` ステートメントの場合、TiDB は行数と変更された行数を自動的に更新します。TiDB は定期的にこの情報を永続化し、更新サイクルは 20 \* [`stats-lease`](/tidb-configuration-file.md#stats-lease) です。`stats-lease` のデフォルト値は `3s` です。値を `0` に指定すると、TiDB は統計情報を自動的に更新しません。

</CustomContent>

<CustomContent platform="tidb-cloud">

`INSERT`、`DELETE`、または `UPDATE` ステートメントの場合、TiDB は行数と変更された行数を自動的に更新します。TiDB は定期的にこの情報を永続化し、更新サイクルは 20 \* `stats-lease` です。`stats-lease` のデフォルト値は `3s` です。

</CustomContent>

### 関連するシステム変数 {#relevant-system-variables}

統計情報の自動更新に関連する 3 つのシステム変数は次のとおりです：

| システム変数                                                                                                              | デフォルト値        | 説明                                                                    |
| ------------------------------------------------------------------------------------------------------------------- | ------------- | --------------------------------------------------------------------- |
| [`tidb_auto_analyze_ratio`](/system-variables.md#tidb_auto_analyze_ratio)                                           | 0.5           | 自動更新の閾値値                                                              |
| [`tidb_auto_analyze_start_time`](/system-variables.md#tidb_auto_analyze_start_time)                                 | `00:00 +0000` | TiDB が自動更新を実行できる 1 日の開始時刻                                             |
| [`tidb_auto_analyze_end_time`](/system-variables.md#tidb_auto_analyze_end_time)                                     | `23:59 +0000` | TiDB が自動更新を実行できる 1 日の終了時刻                                             |
| [`tidb_auto_analyze_partition_batch_size`](/system-variables.md#tidb_auto_analyze_partition_batch_size-new-in-v640) | `1`           | TiDB がパーティションテーブルを自動的に分析する際（つまり、パーティションテーブルの統計情報を自動的に更新する際）のパーティションの数 |

`tbl` の変更された行数の割合が `tidb_auto_analyze_ratio` よりも大きく、現在の時刻が `tidb_auto_analyze_start_time` と `tidb_auto_analyze_end_time` の間である場合、TiDB はこのテーブルの統計情報を自動的に更新するためにバックグラウンドで `ANALYZE TABLE tbl` ステートメントを実行します。

小さなテーブルで頻繁に少量のデータを変更することが自動更新を頻繁にトリガーする状況を避けるため、テーブルの行数が 1000 未満の場合、TiDB はこのようなデータの変更を自動更新しません。`SHOW STATS_META` ステートメントを使用してテーブルの行数を表示できます。

> **Note:**
>
> 現在、自動更新は手動で `ANALYZE` を入力した構成項目を記録しません。そのため、`ANALYZE` の収集動作を制御するために `WITH` 構文を使用する場合は、統計情報を手動で収集するためのスケジュールされたタスクを設定する必要があります。

#### 自動更新の無効化 {#disable-automatic-update}

自動更新が過剰なリソースを消費し、オンラインアプリケーションの操作に影響を与えることがわかった場合、[`tidb_enable_auto_analyze`](/system-variables.md#tidb_enable_auto_analyze-new-in-v610) システム変数を使用して自動更新を無効にできます。

#### バックグラウンドの`ANALYZE`タスクを終了する {#terminate-background-analyze-tasks}

TiDB v6.0以降、TiDBは`KILL`ステートメントを使用してバックグラウンドで実行されている`ANALYZE`タスクを終了することをサポートしています。バックグラウンドで実行されている`ANALYZE`タスクが多くのリソースを消費し、アプリケーションに影響を与えている場合は、次の手順で`ANALYZE`タスクを終了することができます。

1. 次のSQLステートメントを実行します：

   ```sql
   SHOW ANALYZE STATUS
   ```

   結果の`instance`列と`process_id`列を確認することで、TiDBインスタンスのアドレスとバックグラウンドで実行されている`ANALYZE`タスクの`ID`を取得できます。

2. バックグラウンドで実行されている`ANALYZE`タスクを終了します。

    <CustomContent platform="tidb">

   - [`enable-global-kill`](/tidb-configuration-file.md#enable-global-kill-new-in-v610)が`true`（デフォルトで`true`）の場合、直接`KILL TIDB ${id};`ステートメントを実行できます。ここで`${id}`は前の手順で取得したバックグラウンドで実行されている`ANALYZE`タスクの`ID`です。
   - `enable-global-kill`が`false`の場合、バックエンドで`ANALYZE`タスクを実行しているTiDBインスタンスにクライアントを使用して接続し、`KILL TIDB ${id};`ステートメントを実行する必要があります。クライアントを使用して別のTiDBインスタンスに接続するか、クライアントとTiDBクラスターの間にプロキシがある場合、`KILL`ステートメントではバックグラウンドで実行されている`ANALYZE`タスクを終了できません。

    </CustomContent>

    <CustomContent platform="tidb-cloud">

   `ANALYZE`タスクを終了するには、前の手順で取得したバックグラウンドで実行されている`ANALYZE`タスクの`ID`を使用して`KILL TIDB ${id};`ステートメントを実行できます。

    </CustomContent>

`KILL`ステートメントの詳細については、[`KILL`](/sql-statements/sql-statement-kill.md)を参照してください。

### `ANALYZE`の並行性を制御する {#control-analyze-concurrency}

`ANALYZE`ステートメントを実行する際、次のパラメータを使用して並行性を調整し、システムへの影響を制御できます。

#### `tidb_build_stats_concurrency` {#tidb-build-stats-concurrency}

現在、`ANALYZE`ステートメントを実行すると、タスクが複数の小さなタスクに分割されます。各タスクは1つのカラムまたはインデックスのみで動作します。`tidb_build_stats_concurrency`パラメータを使用して同時タスクの数を制御できます。デフォルト値は`4`です。

#### `tidb_distsql_scan_concurrency` {#tidb-distsql-scan-concurrency}

通常のカラムを解析する場合、`tidb_distsql_scan_concurrency`パラメータを使用して一度に読み取るリージョンの数を制御できます。デフォルト値は`15`です。

#### `tidb_index_serial_scan_concurrency` {#tidb-index-serial-scan-concurrency}

インデックスカラムを解析する場合、`tidb_index_serial_scan_concurrency`パラメータを使用して一度に読み取るリージョンの数を制御できます。デフォルト値は`1`です。

### `ANALYZE`構成を永続化する {#persist-analyze-configurations}

v5.4.0以降、TiDBは一部の`ANALYZE`構成を永続化する機能をサポートしています。この機能により、既存の構成を将来の統計収集に簡単に再利用できます。

次の`ANALYZE`構成が永続化をサポートしています：

| 構成              | 対応するANALYZE構文                                                                                |
| --------------- | -------------------------------------------------------------------------------------------- |
| ヒストグラムバケットの数    | WITH NUM BUCKETS                                                                             |
| Top-Nの数         | WITH NUM TOPN                                                                                |
| サンプルの数          | WITH NUM SAMPLES                                                                             |
| サンプリングレート       | WITH FLOATNUM SAMPLERATE                                                                     |
| `ANALYZE`カラムタイプ | AnalyzeColumnOption ::= ( 'ALL COLUMNS' \| 'PREDICATE COLUMNS' \| 'COLUMNS' ColumnNameList ) |
| `ANALYZE`カラム    | ColumnNameList ::= Identifier ( ',' Identifier )\*                                           |

#### `ANALYZE`構成の永続化を有効にする {#enable-analyze-configuration-persistence}

<CustomContent platform="tidb">

`ANALYZE`構成の永続化機能はデフォルトで有効になっています（システム変数`tidb_analyze_version`はデフォルトで`2`であり、`tidb_persist_analyze_options`はデフォルトで`ON`です）。

</CustomContent>

<CustomContent platform="tidb-cloud">

`ANALYZE`構成の永続化機能はデフォルトで無効になっています。機能を有効にするには、システム変数`tidb_persist_analyze_options`が`ON`であることを確認し、システム変数`tidb_analyze_version`を`2`に設定してください。

</CustomContent>

この機能を使用して、`ANALYZE`ステートメントで指定された永続化構成を手動で実行することができます。記録された後、次回TiDBが自動的に統計を更新するか、または手動で統計を収集する際にこれらの構成を指定しない場合、TiDBは記録された構成に従って統計を収集します。

特定のテーブルで自動解析操作に使用される永続化構成をクエリするには、次のSQLステートメントを使用できます：

```sql
SELECT sample_num, sample_rate, buckets, topn, column_choice, column_ids FROM mysql.analyze_options opt JOIN information_schema.tables tbl ON opt.table_id = tbl.tidb_table_id WHERE tbl.table_schema = '{db_name}' AND tbl.table_name = '{table_name}';
```

TiDBは、最新の`ANALYZE`ステートメントで指定された新しい構成で以前に記録された永続的な構成を上書きします。たとえば、`ANALYZE TABLE t WITH 200 TOPN;`を実行すると、`ANALYZE`ステートメントで上位200の値が設定されます。その後、`ANALYZE TABLE t WITH 0.1 SAMPLERATE;`を実行すると、自動`ANALYZE`ステートメントの上位200の値とサンプリングレート0.1が設定されます。これは、`ANALYZE TABLE t WITH 200 TOPN, 0.1 SAMPLERATE;`と同様です。

#### ANALYZE構成の永続化を無効にする {#disable-analyze-configuration-persistence}

`ANALYZE`構成の永続化機能を無効にするには、`tidb_persist_analyze_options`システム変数を`OFF`に設定します。`tidb_analyze_version = 1`に設定すると、`ANALYZE`構成の永続化機能が無効になります。

`ANALYZE`構成の永続化機能を無効にした後、TiDBは永続化された構成レコードをクリアしません。したがって、この機能を再度有効にした場合、TiDBは以前に記録された永続的な構成を使用して統計情報を収集し続けます。

> **Note:**
>
> `ANALYZE`構成の永続化機能を再度有効にすると、以前に記録された永続化構成が最新のデータに適用されなくなった場合は、`ANALYZE`ステートメントを手動で実行し、新しい永続化構成を指定する必要があります。

### 統計情報の収集用メモリクォータ {#the-memory-quota-for-collecting-statistics}

> **Warning:**
>
> 現在、`ANALYZE`メモリクォータは実験的な機能であり、本番環境でメモリ統計情報が正確でない可能性があります。

TiDB v6.1.0以降、システム変数[`tidb_mem_quota_analyze`](/system-variables.md#tidb_mem_quota_analyze-new-in-v610)を使用して、TiDBで統計情報の収集用のメモリクォータを制御できます。

`tidb_mem_quota_analyze`の適切な値を設定するには、クラスターのデータサイズを考慮してください。デフォルトのサンプリングレートを使用する場合、主な考慮事項は列の数、列値のサイズ、およびTiDBのメモリ構成です。最大値と最小値を構成する際には、次の提案を考慮してください。

> **Note:**
>
> 以下の提案は参考用です。実際のシナリオに基づいて値を構成する必要があります。

- 最小値：TiDBが最も多くの列を持つテーブルから統計情報を収集する際の最大メモリ使用量よりも大きくする必要があります。おおよその参考値：デフォルト構成を使用して20列のテーブルから統計情報を収集する場合、最大メモリ使用量は約800 MiBです。デフォルト構成を使用して160列のテーブルから統計情報を収集する場合、最大メモリ使用量は約5 GiBです。
- 最大値：TiDBが統計情報を収集していない場合の利用可能なメモリよりも小さくする必要があります。

### `ANALYZE`ステートの表示 {#view-analyze-state}

`ANALYZE`ステートメントを実行する際に、次のSQLステートメントを使用して現在の`ANALYZE`ステートを表示できます。

```sql
SHOW ANALYZE STATUS [ShowLikeOrWhere]
```

このステートメントは`ANALYZE`の状態を返します。`ShowLikeOrWhere`を使用して必要な情報をフィルタリングできます。

現在、`SHOW ANALYZE STATUS`ステートメントは次の11のカラムを返します：

| カラム名            | 説明                                                                                                |
| :-------------- | :------------------------------------------------------------------------------------------------ |
| table\_schema   | データベース名                                                                                           |
| table\_name     | テーブル名                                                                                             |
| partition\_name | パーティション名                                                                                          |
| job\_info       | タスク情報。インデックスが分析されている場合、この情報にはインデックス名が含まれます。`tidb_analyze_version =2`の場合、この情報にはサンプル率などの構成項目が含まれます。 |
| processed\_rows | 分析された行数                                                                                           |
| start\_time     | タスクが開始した時間                                                                                        |
| state           | タスクの状態、`pending`、`running`、`finished`、`failed`を含む                                                 |
| fail\_reason    | タスクが失敗した理由。実行が成功した場合、値は`NULL`です。                                                                  |
| instance        | タスクを実行するTiDBインスタンス                                                                                |
| process\_id     | タスクを実行するプロセスID                                                                                    |

TiDB v6.1.0から、`SHOW ANALYZE STATUS`ステートメントはクラスターレベルのタスクを表示するようになりました。TiDBを再起動しても、このステートメントを使用して再起動前のタスクレコードを表示できます。TiDB v6.1.0より前では、`SHOW ANALYZE STATUS`ステートメントはインスタンスレベルのタスクのみを表示し、タスクレコードはTiDBの再起動後にクリアされます。

`SHOW ANALYZE STATUS`は最新のタスクレコードのみを表示します。TiDB v6.1.0から、システムテーブル`mysql.analyze_jobs`を使用して過去7日間のタスク履歴を表示できます。

[`tidb_mem_quota_analyze`](/system-variables.md#tidb_mem_quota_analyze-new-in-v610)が設定されており、TiDBバックグラウンドで自動`ANALYZE`タスクがこの閾値よりも多くのメモリを使用する場合、タスクは再試行されます。`SHOW ANALYZE STATUS`ステートメントの出力で失敗したタスクと再試行されたタスクを確認できます。

[`tidb_max_auto_analyze_time`](/system-variables.md#tidb_max_auto_analyze_time-new-in-v610)が0より大きい場合、TiDBバックグラウンドで実行されている自動`ANALYZE`タスクがこの閾値よりも長い時間を要する場合、タスクは中止されます。

```sql
mysql> SHOW ANALYZE STATUS [ShowLikeOrWhere];
+--------------+------------+----------------+-------------------------------------------------------------------------------------------+----------------+---------------------+---------------------+----------+-------------------------------------------------------------------------------|
| Table_schema | Table_name | Partition_name | Job_info                                                                                  | Processed_rows | Start_time          | End_time            | State    | Fail_reason                                                                   |
+--------------+------------+----------------+-------------------------------------------------------------------------------------------+----------------+---------------------+---------------------+----------+-------------------------------------------------------------------------------|
| test         | sbtest1    |                | retry auto analyze table all columns with 100 topn, 0.055 samplerate                      |        2000000 | 2022-05-07 16:41:09 | 2022-05-07 16:41:20 | finished | NULL                                                                          |
| test         | sbtest1    |                | auto analyze table all columns with 100 topn, 0.5 samplerate                              |              0 | 2022-05-07 16:40:50 | 2022-05-07 16:41:09 | failed   | analyze panic due to memory quota exceeds, please try with smaller samplerate |
```

## 統計を表示 {#view-statistics}

以下のステートメントを使用して統計の状態を表示できます。

### テーブルのメタデータ {#metadata-of-tables}

`SHOW STATS_META` ステートメントを使用して、総行数と更新された行数を表示できます。

```sql
SHOW STATS_META [ShowLikeOrWhere];
```

`ShowLikeOrWhereOpt`の構文は次のとおりです：

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

現在、`SHOW STATS_META`ステートメントは次の6つのカラムを返します：

| カラム名             | 説明       |
| :--------------- | :------- |
| `db_name`        | データベース名  |
| `table_name`     | テーブル名    |
| `partition_name` | パーティション名 |
| `update_time`    | 更新時間     |
| `modify_count`   | 変更された行の数 |
| `row_count`      | 合計行数     |

> **Note:**
>
> TiDBがDMLステートメントに従って合計行数と変更された行数を自動的に更新すると、`update_time`も更新されます。したがって、`update_time`は必ずしも`ANALYZE`ステートメントが実行された最後の時間を示すわけではありません。

### テーブルの健康状態 {#health-state-of-tables}

`SHOW STATS_HEALTHY`ステートメントを使用して、テーブルの健康状態をチェックし、統計の精度をおおよそ推定できます。`modify_count` >= `row_count`の場合、健康状態は0です。`modify_count` < `row_count`の場合、健康状態は(1 - `modify_count`/`row_count`) \* 100です。

構文は次のとおりです：

```sql
SHOW STATS_HEALTHY [ShowLikeOrWhere];
```

`SHOW STATS_HEALTHY`の概要は次のとおりです：

![ShowStatsHealthy](/media/sqlgram/ShowStatsHealthy.png)

現在、`SHOW STATS_HEALTHY`ステートメントは次の4つのカラムを返します：

| カラム名             | 説明        |
| :--------------- | :-------- |
| `db_name`        | データベース名   |
| `table_name`     | テーブル名     |
| `partition_name` | パーティション名  |
| `healthy`        | テーブルの健全状態 |

### カラムのメタデータ {#metadata-of-columns}

`SHOW STATS_HISTOGRAMS`ステートメントを使用して、すべてのカラムの異なる値の数と`NULL`の数を表示できます。

構文は次のとおりです：

```sql
SHOW STATS_HISTOGRAMS [ShowLikeOrWhere]
```

このステートメントは、すべてのカラムの異なる値の数と`NULL`の数を返します。`ShowLikeOrWhere`を使用して、必要な情報をフィルタリングできます。

現在、`SHOW STATS_HISTOGRAMS`ステートメントは、以下の10のカラムを返します：

| カラム名             | 説明                                                   |
| :--------------- | :--------------------------------------------------- |
| `db_name`        | データベース名                                              |
| `table_name`     | テーブル名                                                |
| `partition_name` | パーティション名                                             |
| `column_name`    | カラム名（`is_index`が`0`の場合）またはインデックス名（`is_index`が`1`の場合） |
| `is_index`       | インデックスカラムであるかどうか                                     |
| `update_time`    | 更新時間                                                 |
| `distinct_count` | 異なる値の数                                               |
| `null_count`     | `NULL`の数                                             |
| `avg_col_size`   | カラムの平均長                                              |
| correlation      | カラムと整数主キーのピアソン相関係数であり、2つのカラム間の関連度を示す                 |

### ヒストグラムのバケツ {#buckets-of-histogram}

`SHOW STATS_BUCKETS`ステートメントを使用して、ヒストグラムの各バケツを表示できます。

構文は次のとおりです：

```sql
SHOW STATS_BUCKETS [ShowLikeOrWhere]
```

次の図は次のとおりです：

![SHOW STATS\_BUCKETS](/media/sqlgram/SHOW_STATS_BUCKETS.png)

この文はすべてのバケツに関する情報を返します。 `ShowLikeOrWhere` を使用して必要な情報をフィルタリングできます。

現在、 `SHOW STATS_BUCKETS` 文は次の11個のカラムを返します：

| カラム名             | 説明                                                                           |
| :--------------- | :--------------------------------------------------------------------------- |
| `db_name`        | データベース名                                                                      |
| `table_name`     | テーブル名                                                                        |
| `partition_name` | パーティション名                                                                     |
| `column_name`    | カラム名（`is_index` が `0` の場合）またはインデックス名（`is_index` が `1` の場合）                   |
| `is_index`       | インデックスカラムであるかどうか                                                             |
| `bucket_id`      | バケツのID                                                                       |
| `count`          | バケツと前のバケツに含まれるすべての値の数                                                        |
| `repeats`        | 最大値の発生回数                                                                     |
| `lower_bound`    | 最小値                                                                          |
| `upper_bound`    | 最大値                                                                          |
| `ndv`            | バケツ内の異なる値の数。 `tidb_analyze_version` = `1` の場合、`ndv` は常に `0` であり、実際の意味を持ちません。 |

### Top-N 情報 {#top-n-information}

TiDB が現在収集している Top-N 情報を表示するには、 `SHOW STATS_TOPN` 文を使用できます。

構文は次のとおりです：

```sql
SHOW STATS_TOPN [ShowLikeOrWhere];
```

現在、`SHOW STATS_TOPN` ステートメントは次の7つのカラムを返します：

| カラム名             | 説明                                                         |
| ---------------- | ---------------------------------------------------------- |
| `db_name`        | データベース名                                                    |
| `table_name`     | テーブル名                                                      |
| `partition_name` | パーティション名                                                   |
| `column_name`    | カラム名（`is_index` が `0` の場合）またはインデックス名（`is_index` が `1` の場合） |
| `is_index`       | インデックスカラムであるかどうか                                           |
| `value`          | このカラムの値                                                    |
| `count`          | 値が現れる回数                                                    |

## 統計の削除 {#delete-statistics}

`DROP STATS` ステートメントを実行して統計情報を削除できます。

```sql
DROP STATS TableName
```

前述の文は`TableName`のすべての統計を削除します。パーティションテーブルが指定されている場合、この文はこのテーブルのすべてのパーティションの統計および動的剪定モードで生成されたGlobalStatsを削除します。

```sql
DROP STATS TableName PARTITION PartitionNameList;
```

この前述の文は、`PartitionNameList` で指定されたパーティションの統計のみを削除します。

```sql
DROP STATS TableName GLOBAL;
```

指定されたテーブルの動的剪定モードで生成されたGlobalStatsのみを削除します。

## 統計の読み込み {#load-statistics}

> **Note:**
>
> 統計の読み込みは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

デフォルトでは、カラム統計のサイズに応じて、TiDBは次のように異なる方法で統計を読み込みます。

- メモリを少量消費する統計（count、distinctCount、nullCountなど）の場合、カラムデータが更新される限り、TiDBは対応する統計を自動的にメモリに読み込んでSQL最適化段階で使用します。
- メモリを大量に消費する統計（ヒストグラム、TopN、Count-Min Sketchなど）の場合、SQL実行のパフォーマンスを確保するため、TiDBは必要に応じて統計を非同期で読み込みます。ヒストグラムの統計を例に取ると、オプティマイザがそのカラムのヒストグラム統計を使用する場合のみ、TiDBはそのカラムのヒストグラム統計をメモリに読み込みます。オンデマンドの非同期統計の読み込みはSQL実行のパフォーマンスに影響を与えませんが、SQL最適化のための統計が不完全な場合があります。

v5.4.0以降、TiDBは同期的な統計の読み込み機能を導入しました。この機能により、SQLステートメントを実行する際に、TiDBはヒストグラム、TopN、Count-Min Sketchなどの大きなサイズの統計をメモリに同期的に読み込むことができます。これにより、SQL最適化のための統計の完全性が向上します。

この機能を有効にするには、[`tidb_stats_load_sync_wait`](/system-variables.md#tidb_stats_load_sync_wait-new-in-v540)システム変数の値をタイムアウト（ミリ秒単位）に設定します。この変数のデフォルト値は`100`で、この値は機能が有効になっていることを示します。

<CustomContent platform="tidb">

同期的な統計の読み込み機能を有効にした後、次のように機能をさらに構成できます。

- SQL最適化の待機時間がタイムアウトに達した場合のTiDBの動作を制御するには、[`tidb_stats_load_pseudo_timeout`](/system-variables.md#tidb_stats_load_pseudo_timeout-new-in-v540)システム変数の値を変更します。この変数のデフォルト値は`ON`で、タイムアウト後、SQL最適化プロセスは任意のカラムのヒストグラム、TopN、またはCMSketch統計を使用しません。この変数が`OFF`に設定されている場合、タイムアウト後、SQL実行が失敗します。
- 同期的な統計の読み込み機能が同時に処理できる最大カラム数を指定するには、TiDB構成ファイルの[`stats-load-concurrency`](/tidb-configuration-file.md#stats-load-concurrency-new-in-v540)オプションの値を変更します。デフォルト値は`5`です。
- 同期的な統計の読み込み機能がキャッシュできる最大カラムリクエスト数を指定するには、TiDB構成ファイルの[`stats-load-queue-size`](/tidb-configuration-file.md#stats-load-queue-size-new-in-v540)オプションの値を変更します。デフォルト値は`1000`です。

TiDBの起動中に、初期統計が完全に読み込まれる前に実行されたSQLステートメントは、最適でない実行計画を持つ可能性があり、パフォーマンスの問題を引き起こすことがあります。このような問題を回避するために、TiDB v7.1.0では、[`force-init-stats`](/tidb-configuration-file.md#force-init-stats-new-in-v710)構成パラメータを導入しました。このオプションを使用すると、TiDBは起動中に統計の初期化が完了した後にのみサービスを提供するかどうかを制御できます。このパラメータはデフォルトで無効になっています。

> **Warning:**
>
> 軽量統計の初期化は実験的な機能です。本番環境で使用しないことをお勧めします。この機能は予告なく変更または削除される場合があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

v7.1.0以降、TiDBは軽量統計の初期化のために[`lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710)を導入しました。

- `lite-init-stats`の値が`true`の場合、統計の初期化はインデックスまたはカラムのヒストグラム、TopN、またはCount-Min Sketchをメモリに読み込みません。
- `lite-init-stats`の値が`false`の場合、統計の初期化はインデックスとプライマリキーのヒストグラム、TopN、およびCount-Min Sketchをメモリに読み込みますが、非プライマリキーのカラムのヒストグラム、TopN、およびCount-Min Sketchをメモリに読み込みません。オプティマイザが特定のインデックスまたはカラムのヒストグラム、TopN、およびCount-Min Sketchを必要とする場合、必要な統計は同期的または非同期的にメモリに読み込まれます。

`lite-init-stats`のデフォルト値は`false`で、軽量統計の初期化を無効にします。`lite-init-stats`を`true`に設定すると、統計の初期化が高速化され、不要な統計の読み込みを避けることでTiDBのメモリ使用量が減少します。

</CustomContent>

<CustomContent platform="tidb-cloud">

同期的な統計の読み込み機能を有効にした後、SQL最適化の待機時間がタイムアウトに達した場合のTiDBの動作を[`tidb_stats_load_pseudo_timeout`](/system-variables.md#tidb_stats_load_pseudo_timeout-new-in-v540)システム変数の値を変更することで制御できます。この変数のデフォルト値は`ON`で、タイムアウト後、SQL最適化プロセスは任意のカラムのヒストグラム、TopN、またはCMSketch統計を使用しません。この変数が`OFF`に設定されている場合、タイムアウト後、SQL実行が失敗します。

</CustomContent>

## 統計のインポートとエクスポート {#import-and-export-statistics}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> このセクションはTiDB Cloudには適用されません。

</CustomContent>

### 統計のエクスポート {#export-statistics}

統計をエクスポートするためのインターフェースは次のとおりです。

- `${db_name}`データベースの`${table_name}`テーブルのJSON形式の統計を取得するには：

  ```
  http://${tidb-server-ip}:${tidb-server-status-port}/stats/dump/${db_name}/${table_name}
  ```

  例：

  ```
  curl -s http://127.0.0.1:10080/stats/dump/test/t1 -o /tmp/t1.json
  ```

- 特定の時刻に`${db_name}`データベースの`${table_name}`テーブルのJSON形式の統計を取得するには：

  ```
  http://${tidb-server-ip}:${tidb-server-status-port}/stats/dump/${db_name}/${table_name}/${yyyyMMddHHmmss}
  ```

### 統計のインポート {#import-statistics}

> **Note:**
>
> MySQLクライアントを起動する際に、`--local-infile=1`オプションを使用してください。

一般的に、インポートされた統計はエクスポートインターフェースを使用して取得したJSONファイルを参照します。

構文："

```
LOAD STATS 'file_name'
```

`file_name`はインポートされる統計のファイル名です。

## ロック統計 {#lock-statistics}

> **警告:**
>
> ロック統計は現在のバージョンの実験的な機能です。本番環境で使用することは推奨されません。

v6.5.0以降、TiDBはロック統計をサポートしています。テーブルの統計がロックされると、テーブルの統計を変更することはできず、`ANALYZE`ステートメントをテーブルで実行することもできません。例えば：

`t`というテーブルを作成し、データを挿入します。テーブル`t`の統計がロックされていない場合、`ANALYZE`ステートメントは正常に実行されます。

```sql
mysql> create table t(a int, b int);
Query OK, 0 rows affected (0.03 sec)

mysql> insert into t values (1,2), (3,4), (5,6), (7,8);
Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> analyze table t;
Query OK, 0 rows affected, 1 warning (0.02 sec)

mysql> show warnings;
+-------+------+-----------------------------------------------------------------+
| Level | Code | Message                                                         |
+-------+------+-----------------------------------------------------------------+
| Note  | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t |
+-------+------+-----------------------------------------------------------------+
1 row in set (0.00 sec)
```

テーブル`t`の統計をロックし、`ANALYZE`を実行します。`SHOW STATS_LOCKED`の出力から、テーブル`t`の統計がロックされていることがわかります。警告メッセージには、`ANALYZE`ステートメントがテーブル`t`をスキップしたことが表示されます。

```sql
mysql> lock stats t;
Query OK, 0 rows affected (0.00 sec)

mysql> show stats_locked;
+---------+------------+----------------+--------+
| Db_name | Table_name | Partition_name | Status |
+---------+------------+----------------+--------+
| test    | t          |                | locked |
+---------+------------+----------------+--------+
1 row in set (0.01 sec)

mysql> analyze table t;
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> show warnings;
+---------+------+-----------------------------------------------------------------+
| Level   | Code | Message                                                         |
+---------+------+-----------------------------------------------------------------+
| Note    | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t |
| Warning | 1105 | skip analyze locked table: t                                    |
+---------+------+-----------------------------------------------------------------+
2 rows in set (0.00 sec)
```

テーブル `t` の統計を解除し、`ANALYZE`を再度正常に実行できます。

```sql
mysql> unlock stats t;
Query OK, 0 rows affected (0.01 sec)

mysql> analyze table t;
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> show warnings;
+-------+------+-----------------------------------------------------------------+
| Level | Code | Message                                                         |
+-------+------+-----------------------------------------------------------------+
| Note  | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t |
+-------+------+-----------------------------------------------------------------+
1 row in set (0.00 sec)
```

## 参照も見てください {#see-also}

<CustomContent platform="TiDB">

- [LOAD STATS](/sql-statements/sql-statement-load-stats.md)
- [DROP STATS](/sql-statements/sql-statement-drop-stats.md)
- [LOCK STATS](/sql-statements/sql-statement-lock-stats.md)
- [UNLOCK STATS](/sql-statements/sql-statement-unlock-stats.md)
- [SHOW STATS\_LOCKED](/sql-statements/sql-statement-show-stats-locked.md)

</CustomContent>

<CustomContent platform="TiDB Cloud">

- [LOAD STATS](/sql-statements/sql-statement-load-stats.md)
- [LOCK STATS](/sql-statements/sql-statement-lock-stats.md)
- [UNLOCK STATS](/sql-statements/sql-statement-unlock-stats.md)
- [SHOW STATS\_LOCKED](/sql-statements/sql-statement-show-stats-locked.md)

</CustomContent>
