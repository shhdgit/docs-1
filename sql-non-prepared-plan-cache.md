---
title: SQL Non-Prepared Execution Plan Cache
summary: Learn about the principle, usage, and examples of the SQL non-prepared execution plan cache in TiDB.
---

# SQL 非準備実行計画キャッシュ {#sql-non-prepared-execution-plan-cache}

> **Warning:**
>
> 非準備実行計画キャッシュは実験的な機能です。本番環境で使用しないことをお勧めします。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

TiDBは、[`Prepare`/`Execute`ステートメント](/sql-prepared-plan-cache.md)と同様に、一部の非`PREPARE`ステートメントの実行計画キャッシュをサポートしています。この機能により、これらのステートメントは最適化フェーズをスキップし、パフォーマンスを向上させることができます。

## 原則 {#principle}

非準備実行計画キャッシュはセッションレベルの機能であり、[準備実行計画キャッシュ](/sql-prepared-plan-cache.md)とキャッシュを共有します。非準備実行計画キャッシュの基本原則は次のとおりです。

1. 非準備実行計画キャッシュを有効にした後、TiDBはクエリを抽象構文木（AST）に基づいてパラメータ化します。たとえば、`SELECT * FROM t WHERE b < 10 AND a = 1`は`SELECT * FROM t WHERE b < ? and a = ?`としてパラメータ化されます。
2. 次に、TiDBはパラメータ化されたクエリを使用して実行計画キャッシュを検索します。
3. 再利用可能な計画が見つかった場合、それが直接使用され、最適化フェーズがスキップされます。
4. それ以外の場合、オプティマイザは新しい計画を生成し、それをキャッシュに追加して、後続のクエリで再利用します。

## 使用法 {#usage}

非準備実行計画キャッシュを有効または無効にするには、[`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache)システム変数を設定できます。また、[`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710)システム変数を使用して、非準備実行計画キャッシュのサイズを制御することもできます。キャッシュされた計画の数が`tidb_session_plan_cache_size`を超えると、TiDBは最近使用された（LRU）戦略を使用して計画を削除します。

v7.1.0からは、システム変数[`tidb_plan_cache_max_plan_size`](/system-variables.md#tidb_plan_cache_max_plan_size-new-in-v710)を使用して、キャッシュできる計画の最大サイズを制御できます。デフォルト値は2MBです。計画のサイズがこの値を超えると、計画はキャッシュされません。

> **Note:**
>
> `tidb_session_plan_cache_size`で指定されたメモリは、準備実行計画キャッシュと非準備実行計画キャッシュで共有されます。現在のクラスターで準備実行計画キャッシュを有効にしている場合、非準備実行計画キャッシュを有効にすると、元の準備実行計画キャッシュのヒット率が低下する可能性があります。

## 例 {#example}

次の例は、非準備実行計画キャッシュの使用方法を示しています。

1. テスト用のテーブル`t`を作成します。

   ```sql
   CREATE TABLE t (a INT, b INT, KEY(b));
   ```

2. 非準備実行計画キャッシュを有効にします。

   ```sql
   SET tidb_enable_non_prepared_plan_cache = ON;
   ```

3. 次の2つのクエリを実行します。

   ```sql
   SELECT * FROM t WHERE b < 10 AND a = 1;
   SELECT * FROM t WHERE b < 5 AND a = 2;
   ```

4. 2番目のクエリがキャッシュにヒットしたかどうかを確認します。

   ```sql
   SELECT @@last_plan_from_cache;
   ```

   出力の`last_plan_from_cache`の値が`1`であれば、2番目のクエリの実行計画がキャッシュから取得されたことを意味します。

   ```sql
   +------------------------+
   | @@last_plan_from_cache |
   +------------------------+
   |                      1 |
   +------------------------+
   1 row in set (0.00 sec)
   ```

## 制限事項 {#restrictions}

### 最適でない計画をキャッシュする {#cache-suboptimal-plans}

TiDBは、パラメータ化されたクエリに対して1つの計画のみをキャッシュします。たとえば、クエリ`SELECT * FROM t WHERE a < 1`と`SELECT * FROM t WHERE a < 100000`は、`SELECT * FROM t WHERE a < ?`という同じパラメータ化された形式を共有し、したがって同じ計画を共有します。

これによりパフォーマンスの問題が発生する場合は、`ignore_plan_cache()`ヒントを使用してキャッシュ内の計画を無視し、オプティマイザがSQLのたびに新しい実行計画を生成するようにすることができます。SQLを変更できない場合は、問題を解決するためにバインディングを作成することができます。たとえば、`CREATE BINDING FOR SELECT ... USING SELECT /*+ ignore_plan_cache() */ ...`。

### 使用制限 {#usage-restrictions}

前述のリスクと、実行計画キャッシュが単純なクエリに対してのみ有益であること（クエリが複雑で実行に時間がかかる場合、実行計画キャッシュを使用してもあまり役立たないことがあります）から、TiDBは非準備済みプランキャッシュの範囲に厳しい制限を設けています。制限は次のとおりです。

- [準備済みプランキャッシュ](/sql-prepared-plan-cache.md) でサポートされていないクエリまたはプランは、非準備済みプランキャッシュでもサポートされません。
- `Window` や `Having` などの複雑な演算子を含むクエリはサポートされません。
- 3つ以上の `Join` テーブルやサブクエリを含むクエリはサポートされません。
- `ORDER BY` や `GROUP BY` の直後に数値や式を含むクエリはサポートされません。たとえば、`ORDER BY 1` や `GROUP BY a+1` などです。`ORDER BY column_name` と `GROUP BY column_name` のみがサポートされます。
- `JSON`、`ENUM`、`SET`、または `BIT` 型の列でフィルタリングするクエリはサポートされません。たとえば、`SELECT * FROM t WHERE json_col = '{}'` などです。
- `NULL` 値でフィルタリングするクエリはサポートされません。たとえば、`SELECT * FROM t WHERE a is NULL` などです。
- パラメータ化後のパラメータが200個を超えるクエリはデフォルトでサポートされません。たとえば、`SELECT * FROM t WHERE a in (1, 2, 3, ... 201)` などです。v7.1.1 以降では、[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v710) システム変数を使用してこの制限を変更できます。
- パーティションテーブル、仮想列、一時テーブル、ビュー、またはメモリテーブルにアクセスするクエリはサポートされません。たとえば、`SELECT * FROM INFORMATION_SCHEMA.COLUMNS` のようなクエリです。ここで `COLUMNS` は TiDB メモリテーブルです。
- ヒントやバインディングを含むクエリはサポートされません。
- デフォルトでは、DML 文または `FOR UPDATE` 句を含む `SELECT` 文はサポートされません。この制限を解除するには、`SET tidb_enable_non_prepared_plan_cache_for_dml = ON` を実行できます。

この機能を有効にした後、オプティマイザーはクエリを迅速に評価します。非準備済みプランキャッシュのサポート条件を満たさない場合、クエリは通常の最適化プロセスにフォールバックします。

## パフォーマンスの利点 {#performance-benefits}

内部テストでは、非準備済みプランキャッシュ機能を有効にすることで、ほとんどの TP シナリオで著しいパフォーマンスの利点が得られます。ただし、クエリがサポートされるかどうかを判断し、クエリをパラメータ化するなど、追加のパフォーマンスオーバーヘッドも発生します。この機能がワークロードの大部分のクエリをサポートできない場合、有効にすることで実際にパフォーマンスに悪影響を及ぼす可能性があります。

この場合、Grafana の **Queries Using Plan Cache OPS** パネルの `non-prepared` メトリクスと **Plan Cache Miss OPS** パネルの `non-prepared-unsupported` メトリクスを観察する必要があります。ほとんどのクエリがサポートされず、プランキャッシュにヒットするのはわずかな場合、この機能を無効にできます。

![non-prepared-unsupported](/media/non-prepapred-plan-cache-unsupprot.png)

## 診断 {#diagnostics}

非準備済みプランキャッシュを有効にした後、`EXPLAIN FORMAT='plan_cache' SELECT ...` 文を実行して、クエリがキャッシュにヒットするかどうかを確認できます。キャッシュにヒットしないクエリについては、システムが警告で理由を返します。

`FORMAT='plan_cache'` を追加しない場合、`EXPLAIN` 文は決してキャッシュにヒットしません。

クエリがキャッシュにヒットするかどうかを確認するには、次の `EXPLAIN FORMAT='plan_cache'` 文を実行してください。

```sql
EXPLAIN FORMAT='plan_cache' SELECT * FROM (SELECT a+1 FROM t) t;
```

The output is as follows:

```sql
3 rows in set, 1 warning (0.00 sec)
```

`SHOW warnings;` を実行してキャッシュにヒットしないクエリを表示します。

```sql
SHOW warnings;
```

入力コンテンツ：「出力は次のとおりです。」

```sql
+---------+------+-------------------------------------------------------------------------------+
| Level   | Code | Message                                                                       |
+---------+------+-------------------------------------------------------------------------------+
| Warning | 1105 | skip non-prepared plan-cache: queries that have sub-queries are not supported |
+---------+------+-------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

前述の例では、準備されていないプランキャッシュが`+`演算をサポートしていないため、クエリはキャッシュにヒットすることができません。

## モニタリング {#monitoring}

準備されていないプランキャッシュを有効にした後、次のパネルでメモリ使用量、キャッシュ内のプラン数、およびキャッシュヒット率をモニタリングできます。

![non-prepare-plan-cache](/media/tidb-non-prepared-plan-cache-metrics.png)

また、`statements_summary`テーブルと遅いクエリログでキャッシュヒット率をモニタリングすることもできます。次に、`statements_summary`テーブルでキャッシュヒット率を表示する方法を示します。

1. テーブル`t`を作成します：

   ```sql
   CREATE TABLE t (a int);
   ```

2. 準備されていないプランキャッシュを有効にします：

   ```sql
   SET @@tidb_enable_non_prepared_plan_cache=ON;
   ```

3. 次の3つのクエリを実行します：

   ```sql
   SELECT * FROM t WHERE a<1;
   SELECT * FROM t WHERE a<2;
   SELECT * FROM t WHERE a<3;
   ```

4. `statements_summary`テーブルをクエリしてキャッシュヒット率を表示します：

   ```sql
   SELECT digest_text, query_sample_text, exec_count, plan_in_cache, plan_cache_hits FROM INFORMATION_SCHEMA.STATEMENTS_SUMMARY WHERE query_sample_text LIKE '%SELECT * FROM %';
   ```

   出力は次のようになります：

   ```sql
   +---------------------------------+------------------------------------------+------------+---------------+-----------------+
   | digest_text                     | query_sample_text                        | exec_count | plan_in_cache | plan_cache_hits |
   +---------------------------------+------------------------------------------+------------+---------------+-----------------+
   | SELECT * FROM `t` WHERE `a` < ? | SELECT * FROM t WHERE a<1                |          3 |             1 |               2 |
   +---------------------------------+------------------------------------------+------------+---------------+-----------------+
   1 row in set (0.01 sec)
   ```

   出力から、クエリが3回実行され、キャッシュが2回ヒットしたことがわかります。
