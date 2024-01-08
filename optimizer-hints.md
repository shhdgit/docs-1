---
title: Optimizer Hints
summary: Use Optimizer Hints to influence query execution plans
---

# オプティマイザーヒント {#optimizer-hints}

TiDBは、MySQL 5.7で導入されたコメントのような構文に基づくオプティマイザーヒントをサポートしています。たとえば、一般的な構文の1つは `/*+ HINT_NAME([t1_name [, t2_name] ...]) */` です。オプティマイザーヒントの使用は、TiDBオプティマイザーがより最適でないクエリプランを選択する場合に推奨されています。

ヒントが効果を発揮しない状況に遭遇した場合は、[ヒントが効果を発揮しない一般的な問題のトラブルシューティング](#troubleshoot-common-issues-that-hints-do-not-take-effect)を参照してください。

## 構文 {#syntax}

オプティマイザーヒントは大文字小文字を区別せず、SQLステートメントの `SELECT`、`UPDATE`、または`DELETE`キーワードに続く `/*+ ... */` コメント内で指定されます。オプティマイザーヒントは現在、`INSERT`ステートメントではサポートされていません。

複数のヒントは、カンマで区切って指定することができます。たとえば、次のクエリは3つの異なるヒントを使用しています：

```sql
SELECT /*+ USE_INDEX(t1, idx1), HASH_AGG(), HASH_JOIN(t1) */ count(*) FROM t t1, t t2 WHERE t1.a = t2.b;
```

クエリ実行計画に対する最適化ヒントの影響を[`EXPLAIN`](/sql-statements/sql-statement-explain.md)および[`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md)の出力で観察できます。

不正確または不完全なヒントは、ステートメントエラーを引き起こしません。これは、ヒントがクエリ実行に対して*ヒント*（提案）の意味しか持たないように意図されているためです。同様に、TiDBは、ヒントが適用されない場合には最大で警告を返します。

> **Note:**
>
> 指定されたキーワードの後にコメントが続かない場合、一般的なMySQLのコメントとして扱われます。コメントは効果を持たず、警告は報告されません。

現在、TiDBは2つの異なるスコープのヒントをサポートしており、それぞれ異なるスコープで異なります。最初のカテゴリのヒントは、[`/*+ HASH_AGG() */`](#hash_agg)のようなクエリブロックのスコープで効果を発揮します。2番目のカテゴリのヒントは、[`/*+ MEMORY_QUOTA(1024 MB)*/`](#memory_quotan)のようなクエリ全体で効果を発揮します。

ステートメント内の各クエリまたはサブクエリは、異なるクエリブロックに対応し、各クエリブロックには独自の名前があります。例えば：

```sql
SELECT * FROM (SELECT * FROM t) t1, (SELECT * FROM t) t2;
```

上記のクエリステートメントには3つのクエリブロックがあります。最も外側の `SELECT` は最初のクエリブロックに対応し、その名前は `sel_1` です。2つの `SELECT` サブクエリは、それぞれ2番目と3番目のクエリブロックに対応し、その名前はそれぞれ `sel_2` と `sel_3` です。数字の順序は、左から右への `SELECT` の出現に基づいています。最初の `SELECT` を `DELETE` または `UPDATE` に置き換えると、対応するクエリブロックの名前はそれぞれ `del_1` または `upd_1` になります。

## クエリブロックで有効になるヒント {#hints-that-take-effect-in-query-blocks}

このカテゴリのヒントは、**any** `SELECT`、`UPDATE`、または `DELETE` キーワードの後に続くことができます。ヒントの有効なスコープを制御するには、ヒント内でクエリブロックの名前を使用します。クエリ内の各テーブルを正確に識別することで、ヒントのパラメータを明確にすることができます（テーブル名やエイリアスが重複している場合）。ヒント内でクエリブロックが指定されていない場合、ヒントはデフォルトで現在のブロックで有効になります。

例えば：

```sql
SELECT /*+ HASH_JOIN(@sel_1 t1@sel_1, t3) */ * FROM (SELECT t1.a, t1.b FROM t t1, t t2 WHERE t1.a = t2.a) t1, t t3 WHERE t1.b = t3.b;
```

このヒントは、`sel_1` クエリブロックで有効になり、そのパラメータは `sel_1` の `t1` および `t3` テーブルです（`sel_2` にも `t1` テーブルが含まれています）。

上記のように、ヒント内でクエリブロックの名前を次のように指定できます。

- クエリブロックの名前をヒントの最初のパラメータとして設定し、他のパラメータとはスペースで区切ります。`QB_NAME` に加えて、このセクションにリストされているすべてのヒントには、もう 1 つのオプションの隠しパラメータ `@QB_NAME` があります。このパラメータを使用すると、このヒントの有効範囲を指定できます。
- パラメータ内のテーブル名に `@QB_NAME` を追加して、このテーブルがどのクエリブロックに属するかを明示的に指定します。

> **Note:**
>
> ヒントを有効にするクエリブロック内またはその前にヒントを配置する必要があります。クエリブロックの後にヒントを配置すると、ヒントは有効になりません。

### QB\_NAME {#qb-name}

クエリ文が複雑なネストされたクエリを含む場合、特定のクエリブロックの ID と名前を誤って識別することがあります。`QB_NAME` ヒントはこの点で役立ちます。

`QB_NAME` はクエリブロック名を意味します。クエリブロックに新しい名前を指定できます。指定された `QB_NAME` と以前のデフォルト名の両方が有効です。例:

```sql
SELECT /*+ QB_NAME(QB1) */ * FROM (SELECT * FROM t) t1, (SELECT * FROM t) t2;
```

このヒントは、外部の`SELECT`クエリブロックの名前を`QB1`に指定し、`QB1`とデフォルトの名前`sel_1`の両方がクエリブロックに有効になります。

> **Note:**
>
> 上記の例では、ヒントが`QB_NAME`を`sel_2`に指定し、元の2番目の`SELECT`クエリブロックに新しい`QB_NAME`を指定しない場合、`sel_2`は2番目の`SELECT`クエリブロックの無効な名前になります。

### MERGE\_JOIN(t1\_name \[, tl\_name ...]) {#merge-join-t1-name-tl-name}

`MERGE_JOIN(t1_name [, tl_name ...])`ヒントは、指定されたテーブルに対してソートマージ結合アルゴリズムを使用するようにオプティマイザに指示します。一般的に、このアルゴリズムはメモリを少なく消費しますが、処理時間が長くかかります。非常に大きなデータ量やシステムメモリが不足している場合は、このヒントを使用することをお勧めします。例えば：

```sql
select /*+ MERGE_JOIN(t1, t2) */ * from t1, t2 where t1.id = t2.id;
```

> **Note:**
>
> `TIDB_SMJ`は、TiDB 3.0.xおよびそれ以前のバージョンで`MERGE_JOIN`のエイリアスです。これらのバージョンを使用している場合は、ヒントに`TIDB_SMJ(t1_name [, tl_name ...])`の構文を適用する必要があります。TiDBの後のバージョンでは、`TIDB_SMJ`と`MERGE_JOIN`の両方がヒントの有効な名前ですが、`MERGE_JOIN`が推奨されています。

### NO\_MERGE\_JOIN(t1\_name \[, tl\_name ...]) {#no-merge-join-t1-name-tl-name}

`NO_MERGE_JOIN(t1_name [, tl_name ...])`ヒントは、指定されたテーブルに対してソートマージ結合アルゴリズムを使用しないようにオプティマイザに指示します。例えば：

```sql
SELECT /*+ NO_MERGE_JOIN(t1, t2) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

### INL\_JOIN(t1\_name \[, tl\_name ...]) {#inl-join-t1-name-tl-name}

`INL_JOIN(t1_name [, tl_name ...])` ヒントは、指定されたテーブルに対してインデックス入れ子ループ結合アルゴリズムを使用するようにオプティマイザに指示します。このアルゴリズムは、一部のシナリオではシステムリソースを少なく消費し、処理時間を短縮することがありますが、他のシナリオでは逆の結果を生むことがあります。外部テーブルが`WHERE`条件でフィルタリングされた後の結果セットが10,000行未満の場合、このヒントを使用することを推奨します。例：

```sql
select /*+ INL_JOIN(t1, t2) */ * from t1, t2 where t1.id = t2.id;
```

`INL_JOIN()`で指定されたパラメータは、クエリプランを作成する際の内部テーブルの候補テーブルです。たとえば、`INL_JOIN(t1)`は、TiDBがクエリプランを作成する際に`t1`を内部テーブルとして使用することを考慮することを意味します。候補テーブルにエイリアスがある場合は、`INL_JOIN()`のパラメータとしてそのエイリアスを使用する必要があります。エイリアスがない場合は、テーブルの元の名前をパラメータとして使用します。たとえば、`select /*+ INL_JOIN(t1) */ * from t t1, t t2 where t1.a = t2.b;`クエリでは、`INL_JOIN()`のパラメータとして`t`テーブルのエイリアスである`t1`または`t2`を使用する必要があります。

> **Note:**
>
> `TIDB_INLJ`は、TiDB 3.0.xおよびそれ以前のバージョンで`INL_JOIN`のエイリアスです。これらのバージョンを使用している場合は、ヒントに`TIDB_INLJ(t1_name [, tl_name ...])`構文を適用する必要があります。TiDBの後のバージョンでは、`TIDB_INLJ`と`INL_JOIN`の両方がヒントの有効な名前ですが、`INL_JOIN`が推奨されています。

### NO\_INDEX\_JOIN(t1\_name \[, tl\_name ...]) {#no-index-join-t1-name-tl-name}

`NO_INDEX_JOIN(t1_name [, tl_name ...])`ヒントは、指定されたテーブルに対してインデックスネストループ結合アルゴリズムを使用しないようにオプティマイザに指示します。たとえば：

```sql
SELECT /*+ NO_INDEX_JOIN(t1, t2) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

### INL\_HASH\_JOIN {#inl-hash-join}

`INL_HASH_JOIN(t1_name [, tl_name])`ヒントは、オプティマイザにインデックス入れ子ループハッシュ結合アルゴリズムを使用するよう指示します。このアルゴリズムを使用する条件は、インデックス入れ子ループ結合アルゴリズムを使用する条件と同じです。 2つのアルゴリズムの違いは、`INL_JOIN`が結合された内部テーブルにハッシュテーブルを作成するのに対し、`INL_HASH_JOIN`は結合された外部テーブルにハッシュテーブルを作成します。 `INL_HASH_JOIN`はメモリ使用量に固定の制限がありますが、`INL_JOIN`のメモリ使用量は内部テーブルで一致した行の数に依存します。

### NO\_INDEX\_HASH\_JOIN(t1\_name \[, tl\_name ...]) {#no-index-hash-join-t1-name-tl-name}

`NO_INDEX_HASH_JOIN(t1_name [, tl_name ...])`ヒントは、指定されたテーブルに対してインデックス入れ子ループハッシュ結合アルゴリズムを使用しないようオプティマイザに指示します。

### INL\_MERGE\_JOIN {#inl-merge-join}

`INL_MERGE_JOIN(t1_name [, tl_name])`ヒントは、オプティマイザにインデックス入れ子ループマージ結合アルゴリズムを使用するよう指示します。このアルゴリズムを使用する条件は、インデックス入れ子ループ結合アルゴリズムを使用する条件と同じです。

### NO\_INDEX\_MERGE\_JOIN(t1\_name \[, tl\_name ...]) {#no-index-merge-join-t1-name-tl-name}

`NO_INDEX_MERGE_JOIN(t1_name [, tl_name ...])`ヒントは、指定されたテーブルに対してインデックス入れ子ループマージ結合アルゴリズムを使用しないようオプティマイザに指示します。

### HASH\_JOIN(t1\_name \[, tl\_name ...]) {#hash-join-t1-name-tl-name}

`HASH_JOIN(t1_name [, tl_name ...])`ヒントは、指定されたテーブルに対してハッシュ結合アルゴリズムを使用するようオプティマイザに指示します。このアルゴリズムにより、クエリを複数のスレッドで同時に実行でき、処理速度が向上しますが、より多くのメモリを消費します。例：

```sql
select /*+ HASH_JOIN(t1, t2) */ * from t1, t2 where t1.id = t2.id;
```

> **Note:**
>
> `TIDB_HJ`は、TiDB 3.0.xおよびそれ以前のバージョンで`HASH_JOIN`のエイリアスです。これらのバージョンを使用している場合は、ヒントに`TIDB_HJ(t1_name [, tl_name ...])`構文を適用する必要があります。TiDBの後のバージョンでは、`TIDB_HJ`と`HASH_JOIN`の両方がヒントの有効な名前ですが、`HASH_JOIN`が推奨されています。

### NO\_HASH\_JOIN(t1\_name \[, tl\_name ...]) {#no-hash-join-t1-name-tl-name}

`NO_HASH_JOIN(t1_name [, tl_name ...])`ヒントは、指定されたテーブルに対してハッシュ結合アルゴリズムを使用しないようにオプティマイザに指示します。例えば：

```sql
SELECT /*+ NO_HASH_JOIN(t1, t2) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

### HASH\_JOIN\_BUILD(t1\_name \[, tl\_name ...]) {#hash-join-build-t1-name-tl-name}

`HASH_JOIN_BUILD(t1_name [, tl_name ...])` ヒントは、指定されたテーブルでハッシュ結合アルゴリズムを使用するようにオプティマイザに指示します。これらのテーブルがビルド側として機能します。この方法で、特定のテーブルを使用してハッシュテーブルを構築できます。例えば：

```sql
SELECT /*+ HASH_JOIN_BUILD(t1) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

### HASH\_JOIN\_PROBE(t1\_name \[, tl\_name ...]) {#hash-join-probe-t1-name-tl-name}

`HASH_JOIN_PROBE(t1_name [, tl_name ...])` ヒントは、指定されたテーブルでハッシュ結合アルゴリズムを使用するようにオプティマイザに指示します。これにより、これらのテーブルがプローブ側として機能するようになります。この方法で、特定のテーブルをプローブ側として使用してハッシュ結合アルゴリズムを実行できます。例えば：

```sql
SELECT /*+ HASH_JOIN_PROBE(t2) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

### SEMI\_JOIN\_REWRITE() {#semi-join-rewrite}

`SEMI_JOIN_REWRITE()` ヒントは、セミジョインクエリを通常のジョインクエリに書き換えるようにオプティマイザに指示します。現時点では、このヒントは `EXISTS` サブクエリにのみ適用されます。

このヒントを使用してクエリを書き換えない場合、実行計画でハッシュジョインが選択された場合、セミジョインクエリはサブクエリをハッシュテーブルを構築するためにのみ使用できます。この場合、サブクエリの結果が外部クエリの結果よりも大きい場合、実行速度は予想よりも遅くなる可能性があります。

同様に、実行計画でインデックスジョインが選択された場合、セミジョインクエリは外部クエリを駆動テーブルとしてのみ使用できます。この場合、サブクエリの結果が外部クエリの結果よりも小さい場合、実行速度は予想よりも遅くなる可能性があります。

`SEMI_JOIN_REWRITE()` を使用してクエリを書き換えると、オプティマイザは選択範囲を拡張してより良い実行計画を選択できます。

```sql
-- Does not use SEMI_JOIN_REWRITE() to rewrite the query.
EXPLAIN SELECT * FROM t WHERE EXISTS (SELECT 1 FROM t1 WHERE t1.a = t.a);
```

```sql
+-----------------------------+---------+-----------+------------------------+---------------------------------------------------+
| id                          | estRows | task      | access object          | operator info                                     |
+-----------------------------+---------+-----------+------------------------+---------------------------------------------------+
| MergeJoin_9                 | 7992.00 | root      |                        | semi join, left key:test.t.a, right key:test.t1.a |
| ├─IndexReader_25(Build)     | 9990.00 | root      |                        | index:IndexFullScan_24                            |
| │ └─IndexFullScan_24        | 9990.00 | cop[tikv] | table:t1, index:idx(a) | keep order:true, stats:pseudo                     |
| └─IndexReader_23(Probe)     | 9990.00 | root      |                        | index:IndexFullScan_22                            |
|   └─IndexFullScan_22        | 9990.00 | cop[tikv] | table:t, index:idx(a)  | keep order:true, stats:pseudo                     |
+-----------------------------+---------+-----------+------------------------+---------------------------------------------------+
```

```sql
-- Uses SEMI_JOIN_REWRITE() to rewrite the query.
EXPLAIN SELECT * FROM t WHERE EXISTS (SELECT /*+ SEMI_JOIN_REWRITE() */ 1 FROM t1 WHERE t1.a = t.a);
```

```sql
+------------------------------+---------+-----------+------------------------+---------------------------------------------------------------------------------------------------------------+
| id                           | estRows | task      | access object          | operator info                                                                                                 |
+------------------------------+---------+-----------+------------------------+---------------------------------------------------------------------------------------------------------------+
| IndexJoin_16                 | 1.25    | root      |                        | inner join, inner:IndexReader_15, outer key:test.t1.a, inner key:test.t.a, equal cond:eq(test.t1.a, test.t.a) |
| ├─StreamAgg_39(Build)        | 1.00    | root      |                        | group by:test.t1.a, funcs:firstrow(test.t1.a)->test.t1.a                                                      |
| │ └─IndexReader_34           | 1.00    | root      |                        | index:IndexFullScan_33                                                                                        |
| │   └─IndexFullScan_33       | 1.00    | cop[tikv] | table:t1, index:idx(a) | keep order:true                                                                                               |
| └─IndexReader_15(Probe)      | 1.25    | root      |                        | index:Selection_14                                                                                            |
|   └─Selection_14             | 1.25    | cop[tikv] |                        | not(isnull(test.t.a))                                                                                         |
|     └─IndexRangeScan_13      | 1.25    | cop[tikv] | table:t, index:idx(a)  | range: decided by [eq(test.t.a, test.t1.a)], keep order:false, stats:pseudo                                   |
+------------------------------+---------+-----------+------------------------+---------------------------------------------------------------------------------------------------------------+
```

先行する例から、`SEMI_JOIN_REWRITE()` ヒントを使用すると、TiDB はドライビングテーブル `t1` に基づいて IndexJoin の実行方法を選択できることがわかります。

### SHUFFLE\_JOIN(t1\_name \[, tl\_name ...]) {#shuffle-join-t1-name-tl-name}

`SHUFFLE_JOIN(t1_name [, tl_name ...])` ヒントは、指定されたテーブルにシャッフル結合アルゴリズムを使用するようにオプティマイザに指示します。このヒントは MPP モードでのみ効果があります。例えば：

```sql
SELECT /*+ SHUFFLE_JOIN(t1, t2) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

> **Note:**
>
> - このヒントを使用する前に、現在のTiDBクラスターがクエリでTiFlash MPPモードをサポートしていることを確認してください。詳細については、[Use TiFlash MPP Mode](/tiflash/use-tiflash-mpp-mode.md)を参照してください。
> - このヒントは、[`HASH_JOIN_BUILD` hint](#hash_join_buildt1_name--tl_name-)と[`HASH_JOIN_PROBE` hint](#hash_join_probet1_name--tl_name-)と組み合わせて使用して、Shuffle JoinアルゴリズムのBuild側とProbe側を制御することができます。

### BROADCAST\_JOIN(t1\_name \[, tl\_name ...]) {#broadcast-join-t1-name-tl-name}

`BROADCAST_JOIN(t1_name [, tl_name ...])` ヒントは、指定されたテーブルでBroadcast Joinアルゴリズムを使用するようにオプティマイザに指示します。このヒントは、MPPモードでのみ有効です。例：

```sql
SELECT /*+ BROADCAST_JOIN(t1, t2) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

> **Note:**
>
> - このヒントを使用する前に、現在の TiDB クラスターがクエリで TiFlash MPP モードをサポートしていることを確認してください。詳細については、[Use TiFlash MPP Mode](/tiflash/use-tiflash-mpp-mode.md) を参照してください。
> - このヒントは、[`HASH_JOIN_BUILD` hint](#hash_join_buildt1_name--tl_name-) および [`HASH_JOIN_PROBE` hint](#hash_join_probet1_name--tl_name-) と組み合わせて使用して、ブロードキャスト結合アルゴリズムのビルド側とプローブ側を制御できます。

### NO\_DECORRELATE() {#no-decorrelate}

`NO_DECORRELATE()` ヒントは、指定されたクエリブロック内の相関サブクエリに対してデコレレーションを試みないようにオプティマイザに指示します。このヒントは、`EXISTS`、`IN`、`ANY`、`ALL`、`SOME` サブクエリおよび相関列を含むスカラーサブクエリ（つまり、相関サブクエリ）に適用されます。

このヒントをクエリブロックで使用すると、オプティマイザはサブクエリとその外側のクエリブロック間の相関列に対してデコレレーションを試みず、常に Apply 演算子を使用してクエリを実行します。

デフォルトでは、TiDB は相関サブクエリの[デコレレーションを試みます](/correlated-subquery-optimization.md) 、より高い実行効率を達成するため。ただし、[一部のシナリオ](/correlated-subquery-optimization.md#restrictions)では、デコレレーションは実際に実行効率を低下させる可能性があります。この場合、このヒントを使用して、オプティマイザにデコレレーションを試みないように手動で指示できます。例えば：

```sql
create table t1(a int, b int);
create table t2(a int, b int, index idx(b));
```

```sql
-- Not using NO_DECORRELATE().
explain select * from t1 where t1.a < (select sum(t2.a) from t2 where t2.b = t1.b);
```

```sql
+----------------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------+
| id                               | estRows  | task      | access object | operator info                                                                                                |
+----------------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------+
| HashJoin_11                      | 9990.00  | root      |               | inner join, equal:[eq(test.t1.b, test.t2.b)], other cond:lt(cast(test.t1.a, decimal(10,0) BINARY), Column#7) |
| ├─HashAgg_23(Build)              | 7992.00  | root      |               | group by:test.t2.b, funcs:sum(Column#8)->Column#7, funcs:firstrow(test.t2.b)->test.t2.b                      |
| │ └─TableReader_24               | 7992.00  | root      |               | data:HashAgg_16                                                                                              |
| │   └─HashAgg_16                 | 7992.00  | cop[tikv] |               | group by:test.t2.b, funcs:sum(test.t2.a)->Column#8                                                           |
| │     └─Selection_22             | 9990.00  | cop[tikv] |               | not(isnull(test.t2.b))                                                                                       |
| │       └─TableFullScan_21       | 10000.00 | cop[tikv] | table:t2      | keep order:false, stats:pseudo                                                                               |
| └─TableReader_15(Probe)          | 9990.00  | root      |               | data:Selection_14                                                                                            |
|   └─Selection_14                 | 9990.00  | cop[tikv] |               | not(isnull(test.t1.b))                                                                                       |
|     └─TableFullScan_13           | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo                                                                               |
+----------------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------+
```

前の実行計画から、オプティマイザーが自動的にdecorrelationを実行したことがわかります。decorrelatedな実行計画にはApply演算子がありません。代わりに、計画にはサブクエリと外部クエリブロックの間の結合演算があります。相関する列を持つ元のフィルタ条件（`t2.b = t1.b`）は通常の結合条件になります。

```sql
-- Using NO_DECORRELATE().
explain select * from t1 where t1.a < (select /*+ NO_DECORRELATE() */ sum(t2.a) from t2 where t2.b = t1.b);
```

```sql
+------------------------------------------+-----------+-----------+------------------------+--------------------------------------------------------------------------------------+
| id                                       | estRows   | task      | access object          | operator info                                                                        |
+------------------------------------------+-----------+-----------+------------------------+--------------------------------------------------------------------------------------+
| Projection_10                            | 10000.00  | root      |                        | test.t1.a, test.t1.b                                                                 |
| └─Apply_12                               | 10000.00  | root      |                        | CARTESIAN inner join, other cond:lt(cast(test.t1.a, decimal(10,0) BINARY), Column#7) |
|   ├─TableReader_14(Build)                | 10000.00  | root      |                        | data:TableFullScan_13                                                                |
|   │ └─TableFullScan_13                   | 10000.00  | cop[tikv] | table:t1               | keep order:false, stats:pseudo                                                       |
|   └─MaxOneRow_15(Probe)                  | 10000.00  | root      |                        |                                                                                      |
|     └─StreamAgg_20                       | 10000.00  | root      |                        | funcs:sum(Column#14)->Column#7                                                       |
|       └─Projection_45                    | 100000.00 | root      |                        | cast(test.t2.a, decimal(10,0) BINARY)->Column#14                                     |
|         └─IndexLookUp_44                 | 100000.00 | root      |                        |                                                                                      |
|           ├─IndexRangeScan_42(Build)     | 100000.00 | cop[tikv] | table:t2, index:idx(b) | range: decided by [eq(test.t2.b, test.t1.b)], keep order:false, stats:pseudo         |
|           └─TableRowIDScan_43(Probe)     | 100000.00 | cop[tikv] | table:t2               | keep order:false, stats:pseudo                                                       |
+------------------------------------------+-----------+-----------+------------------------+--------------------------------------------------------------------------------------+
```

先行する実行計画から、オプティマイザーが非相関化を実行しないことがわかります。実行計画にはまだApply演算子が含まれています。相関する列を持つフィルタ条件（`t2.b = t1.b`）は、`t2` テーブルにアクセスする際のフィルタ条件のままです。

### HASH\_AGG() {#hash-agg}

`HASH_AGG()` ヒントは、指定されたクエリブロック内のすべての集約関数にハッシュ集約アルゴリズムを使用するようオプティマイザーに指示します。このアルゴリズムにより、クエリを複数のスレッドで同時に実行できるため、処理速度が向上しますが、より多くのメモリを消費します。例えば：

```sql
select /*+ HASH_AGG() */ count(*) from t1, t2 where t1.a > 10 group by t1.id;
```

### STREAM\_AGG() {#stream-agg}

`STREAM_AGG()` ヒントは、指定されたクエリブロック内のすべての集約関数でストリーム集約アルゴリズムを使用するようにオプティマイザに指示します。一般的に、このアルゴリズムはメモリを少なく消費しますが、処理時間が長くなります。非常に大きなデータ量がある場合やシステムメモリが不足している場合は、このヒントを使用することをお勧めします。例えば：

```sql
select /*+ STREAM_AGG() */ count(*) from t1, t2 where t1.a > 10 group by t1.id;
```

### MPP\_1PHASE\_AGG() {#mpp-1phase-agg}

`MPP_1PHASE_AGG()`は、指定されたクエリブロック内のすべての集約関数に対して、オプティマイザに1フェーズの集計アルゴリズムを使用するよう指示します。このヒントはMPPモードでのみ有効です。例えば：

```sql
SELECT /*+ MPP_1PHASE_AGG() */ COUNT(*) FROM t1, t2 WHERE t1.a > 10 GROUP BY t1.id;
```

> **Note:**
>
> このヒントを使用する前に、現在のTiDBクラスターがクエリでTiFlash MPPモードをサポートしていることを確認してください。詳細については、[Use TiFlash MPP Mode](/tiflash/use-tiflash-mpp-mode.md)を参照してください。

### MPP\_2PHASE\_AGG() {#mpp-2phase-agg}

`MPP_2PHASE_AGG()`は、指定されたクエリブロック内のすべての集約関数に対して、オプティマイザに2段階の集約アルゴリズムを使用するよう指示します。このヒントはMPPモードでのみ有効です。例：

```sql
SELECT /*+ MPP_2PHASE_AGG() */ COUNT(*) FROM t1, t2 WHERE t1.a > 10 GROUP BY t1.id;
```

> **Note:**
>
> このヒントを使用する前に、現在の TiDB クラスターがクエリで TiFlash MPP モードをサポートしていることを確認してください。詳細については、[Use TiFlash MPP Mode](/tiflash/use-tiflash-mpp-mode.md) を参照してください。

### USE\_INDEX(t1\_name, idx1\_name \[, idx2\_name ...]) {#use-index-t1-name-idx1-name-idx2-name}

`USE_INDEX(t1_name, idx1_name [, idx2_name ...])` ヒントは、オプティマイザに指定された `t1_name` テーブルに対して指定されたインデックスのみを使用するように指示します。たとえば、次のヒントを適用することは、`select * from t t1 use index(idx1, idx2);` 文を実行するのと同じ効果があります。

```sql
SELECT /*+ USE_INDEX(t1, idx1, idx2) */ * FROM t1;
```

> **Note:**
>
> もし、このヒントでテーブル名のみを指定し、インデックス名を指定しない場合、実行はインデックスを考慮せず、テーブル全体をスキャンします。

### FORCE\_INDEX(t1\_name, idx1\_name \[, idx2\_name ...]) {#force-index-t1-name-idx1-name-idx2-name}

`FORCE_INDEX(t1_name, idx1_name [, idx2_name ...])` ヒントは、オプティマイザに指定されたインデックスのみを使用するよう指示します。

`FORCE_INDEX(t1_name, idx1_name [, idx2_name ...])` の使用法と効果は、`USE_INDEX(t1_name, idx1_name [, idx2_name ...])` の使用法と効果と同じです。

次の4つのクエリは同じ効果を持ちます：

```sql
SELECT /*+ USE_INDEX(t, idx1) */ * FROM t;
SELECT /*+ FORCE_INDEX(t, idx1) */ * FROM t;
SELECT * FROM t use index(idx1);
SELECT * FROM t force index(idx1);
```

### IGNORE\_INDEX(t1\_name, idx1\_name \[, idx2\_name ...]) {#ignore-index-t1-name-idx1-name-idx2-name}

`IGNORE_INDEX(t1_name, idx1_name [, idx2_name ...])` ヒントは、オプティマイザに指定された `t1_name` テーブルの指定されたインデックスを無視するよう指示します。例えば、以下のヒントを適用することは、`select * from t t1 ignore index(idx1, idx2);` 文を実行するのと同じ効果があります。

```sql
select /*+ IGNORE_INDEX(t1, idx1, idx2) */ * from t t1;
```

### ORDER\_INDEX(t1\_name, idx1\_name \[, idx2\_name ...]) {#order-index-t1-name-idx1-name-idx2-name}

`ORDER_INDEX(t1_name, idx1_name [, idx2_name ...])` ヒントは、オプティマイザに指定されたテーブルのみで指定されたインデックスを使用し、指定されたインデックスを順に読み取るように指示します。

> **Warning:**
>
> このヒントは、SQLステートメントの失敗を引き起こす可能性があります。まずはテストすることをお勧めします。テスト中にエラーが発生した場合は、ヒントを削除してください。テストが正常に実行された場合は、引き続き使用できます。

このヒントは通常、次のシナリオで適用されます：

```sql
CREATE TABLE t(a INT, b INT, key(a), key(b));
EXPLAIN SELECT /*+ ORDER_INDEX(t, a) */ a FROM t ORDER BY a LIMIT 10;
```

```sql
+----------------------------+---------+-----------+---------------------+-------------------------------+
| id                         | estRows | task      | access object       | operator info                 |
+----------------------------+---------+-----------+---------------------+-------------------------------+
| Limit_10                   | 10.00   | root      |                     | offset:0, count:10            |
| └─IndexReader_14           | 10.00   | root      |                     | index:Limit_13                |
|   └─Limit_13               | 10.00   | cop[tikv] |                     | offset:0, count:10            |
|     └─IndexFullScan_12     | 10.00   | cop[tikv] | table:t, index:a(a) | keep order:true, stats:pseudo |
+----------------------------+---------+-----------+---------------------+-------------------------------+
```

オプティマイザーは、このクエリに対して2種類のプランを生成します：`Limit + IndexScan(keep order: true)` と `TopN + IndexScan(keep order: false)`。`ORDER_INDEX` ヒントが使用されると、オプティマイザーは、インデックスを順に読み取る最初のプランを選択します。

> **Note:**
>
> - クエリ自体がインデックスを順に読み取る必要がない場合（つまり、ヒントなしで、オプティマイザーはインデックスを順に読み取るプランをどのような状況でも生成しない場合）、`ORDER_INDEX` ヒントが使用されると、エラー `Can't find a proper physical plan for this query` が発生します。この場合、対応する `ORDER_INDEX` ヒントを削除する必要があります。
> - パーティションテーブルのインデックスは順に読み取ることができないため、パーティションテーブルとその関連するインデックスに `ORDER_INDEX` ヒントを使用しないでください。

### NO\_ORDER\_INDEX(t1\_name, idx1\_name \[, idx2\_name ...]) {#no-order-index-t1-name-idx1-name-idx2-name}

`NO_ORDER_INDEX(t1_name, idx1_name [, idx2_name ...])` ヒントは、指定されたテーブルに対して指定されたインデックスのみを使用し、指定されたインデックスを順に読み取らないようにオプティマイザーに指示します。このヒントは通常、次のシナリオで適用されます。

次の例は、クエリ文の効果が `SELECT * FROM t t1 use index(idx1, idx2);` と等価であることを示しています。

```sql
CREATE TABLE t(a INT, b INT, key(a), key(b));
EXPLAIN SELECT /*+ NO_ORDER_INDEX(t, a) */ a FROM t ORDER BY a LIMIT 10;
```

```sql
+----------------------------+----------+-----------+---------------------+--------------------------------+
| id                         | estRows  | task      | access object       | operator info                  |
+----------------------------+----------+-----------+---------------------+--------------------------------+
| TopN_7                     | 10.00    | root      |                     | test.t.a, offset:0, count:10   |
| └─IndexReader_14           | 10.00    | root      |                     | index:TopN_13                  |
|   └─TopN_13                | 10.00    | cop[tikv] |                     | test.t.a, offset:0, count:10   |
|     └─IndexFullScan_12     | 10000.00 | cop[tikv] | table:t, index:a(a) | keep order:false, stats:pseudo |
+----------------------------+----------+-----------+---------------------+--------------------------------+
```

`ORDER_INDEX` ヒントの例と同様に、このクエリに対してオプティマイザは2種類のプランを生成します: `Limit + IndexScan(keep order: true)` と `TopN + IndexScan(keep order: false)`。`NO_ORDER_INDEX` ヒントを使用すると、オプティマイザは後者のプランを選択してインデックスを順序外で読み取ります。

### AGG\_TO\_COP() {#agg-to-cop}

`AGG_TO_COP()` ヒントは、指定されたクエリブロック内の集約操作をコプロセッサにプッシュダウンするようにオプティマイザに指示します。オプティマイザがプッシュダウンするのに適した集約関数をプッシュダウンしない場合、このヒントを使用することを推奨します。例えば：

```sql
select /*+ AGG_TO_COP() */ sum(t1.a) from t t1;
```

### LIMIT\_TO\_COP() {#limit-to-cop}

`LIMIT_TO_COP()` ヒントは、オプティマイザに指定されたクエリブロック内の `Limit` および `TopN` オペレーターをコプロセッサーにプッシュダウンするよう指示します。オプティマイザがそのような操作を行わない場合、このヒントを使用することをお勧めします。例えば：

```sql
SELECT /*+ LIMIT_TO_COP() */ * FROM t WHERE a = 1 AND b > 10 ORDER BY c LIMIT 1;
```

### READ\_FROM\_STORAGE(TiFlash\[t1\_name \[, tl\_name ...]], TiKV\[t2\_name \[, tl\_name ...]]) {#read-from-storage-tiflash-t1-name-tl-name-tikv-t2-name-tl-name}

`READ_FROM_STORAGE(TiFlash[t1_name [, tl_name ...]], TiKV[t2_name [, tl_name ...]])` ヒントは、オプティマイザに特定のテーブルを特定のストレージエンジンから読み取るよう指示します。現在、このヒントは2つのストレージエンジンパラメータ、`TiKV` と `TiFlash` をサポートしています。テーブルにエイリアスがある場合は、`READ_FROM_STORAGE()` のパラメータとしてエイリアスを使用します。テーブルにエイリアスがない場合は、テーブルの元の名前をパラメータとして使用します。例：

```sql
select /*+ READ_FROM_STORAGE(TIFLASH[t1], TIKV[t2]) */ t1.a from t t1, t t2 where t1.a = t2.a;
```

### USE\_INDEX\_MERGE(t1\_name, idx1\_name \[, idx2\_name ...]) {#use-index-merge-t1-name-idx1-name-idx2-name}

`USE_INDEX_MERGE(t1_name, idx1_name [, idx2_name ...])` ヒントは、オプティマイザに特定のテーブルにインデックスマージメソッドでアクセスするよう指示します。 インデックスマージには、2つのタイプがあります：交差タイプと結合タイプ。 詳細については、[Explain Statements Using Index Merge](/explain-index-merge.md) を参照してください。

インデックスのリストを明示的に指定すると、TiDB はリストからインデックスを選択してインデックスマージを構築します。 インデックスのリストを指定しない場合、TiDB は利用可能なすべてのインデックスからインデックスマージを構築するためのインデックスを選択します。

交差タイプのインデックスマージの場合、指定されたインデックスのリストはヒントの必須パラメータです。 結合タイプのインデックスマージの場合、指定されたインデックスのリストはヒントのオプションのパラメータです。 次の例を参照してください。

```sql
SELECT /*+ USE_INDEX_MERGE(t1, idx_a, idx_b, idx_c) */ * FROM t1 WHERE t1.a > 10 OR t1.b > 10;
```

同じテーブルに複数の `USE_INDEX_MERGE` ヒントがある場合、オプティマイザはこれらのヒントで指定されたインデックスセットの和集合からインデックスを選択しようとします。

> **Note:**
>
> `USE_INDEX_MERGE` のパラメータは、カラム名ではなくインデックス名を参照しています。主キーのインデックス名は `primary` です。

### LEADING(t1\_name \[, tl\_name ...]) {#leading-t1-name-tl-name}

`LEADING(t1_name [, tl_name ...])` ヒントは、オプティマイザに、実行計画を生成する際に、ヒントで指定されたテーブル名の順序に従って複数のテーブルの結合の順序を決定するように促します。例えば：

```sql
SELECT /*+ LEADING(t1, t2) */ * FROM t1, t2, t3 WHERE t1.id = t2.id and t2.id = t3.id;
```

上記のクエリにおいて、複数のテーブルを結合する場合、`LEADING()` ヒントで指定されたテーブル名の順に結合の順序が決定されます。オプティマイザはまず `t1` と `t2` を結合し、その後その結果を `t3` と結合します。このヒントは[`STRAIGHT_JOIN`](#straight_join)よりも一般的です。

`LEADING` ヒントは、以下の状況では効果を発揮しません。

- 複数の `LEADING` ヒントが指定されている。
- `LEADING` ヒントで指定されたテーブル名が存在しない。
- `LEADING` ヒントで重複したテーブル名が指定されている。
- オプティマイザが `LEADING` ヒントで指定された順序で結合操作を実行できない。
- すでに `straight_join()` ヒントが存在する。
- クエリに外部結合とデカルト積が含まれている。
- 同時に `MERGE_JOIN`、`INL_JOIN`、`INL_HASH_JOIN`、`HASH_JOIN` ヒントのいずれかが使用されている。

上記のような状況では、警告が生成されます。

```sql
-- Multiple `LEADING` hints are specified.
SELECT /*+ LEADING(t1, t2) LEADING(t3) */ * FROM t1, t2, t3 WHERE t1.id = t2.id and t2.id = t3.id;

-- To learn why the `LEADING` hint fails to take effect, execute `show warnings`.
SHOW WARNINGS;
```

```sql
+---------+------+-------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                           |
+---------+------+-------------------------------------------------------------------------------------------------------------------+
| Warning | 1815 | We can only use one leading hint at most, when multiple leading hints are used, all leading hints will be invalid |
+---------+------+-------------------------------------------------------------------------------------------------------------------+
```

> **Note:**
>
> クエリステートメントに外部結合が含まれる場合、ヒントで結合順序を入れ替えることができるテーブルのみを指定できます。ヒントに結合順序を入れ替えることができないテーブルが含まれている場合、ヒントは無効になります。たとえば、`SELECT * FROM t1 LEFT JOIN (t2 JOIN t3 JOIN t4) ON t1.a = t2.a;` では、`t2`、`t3`、`t4` テーブルの結合順序を制御したい場合、`LEADING` ヒントで `t1` を指定することはできません。

### MERGE() {#merge}

共通テーブル式（CTE）を使用するクエリで `MERGE()` ヒントを使用すると、サブクエリのマテリアライズを無効にし、サブクエリインラインをCTEに展開できます。このヒントは、再帰的でないCTEにのみ適用されます。一部のシナリオでは、`MERGE()` を使用することで、一時領域を割り当てるデフォルトの動作よりも実行効率が向上することがあります。たとえば、クエリ条件をプッシュダウンする場合や、CTEクエリをネストする場合などです。

```sql
-- Uses the hint to push down the predicate of the outer query.
WITH CTE AS (SELECT /*+ MERGE() */ * FROM tc WHERE tc.a < 60) SELECT * FROM CTE WHERE CTE.a < 18;

-- Uses the hint in a nested CTE query to expand a CTE inline into the outer query.
WITH CTE1 AS (SELECT * FROM t1), CTE2 AS (WITH CTE3 AS (SELECT /*+ MERGE() */ * FROM t2), CTE4 AS (SELECT * FROM t3) SELECT * FROM CTE3, CTE4) SELECT * FROM CTE1, CTE2;
```

> **Note:**
>
> `MERGE()`は、単純なCTEクエリにのみ適用されます。次の状況では適用されません。
>
> - [再帰CTE](https://docs.pingcap.com/tidb/stable/dev-guide-use-common-table-expression#recursive-cte)
> - 集約演算子、ウィンドウ関数、および`DISTINCT`など、展開できないインラインを持つサブクエリ。
>
> CTE参照の数が多すぎると、クエリのパフォーマンスがデフォルトのマテリアライゼーション動作よりも低くなる可能性があります。

## グローバルに効果を発揮するヒント {#hints-that-take-effect-globally}

グローバルヒントは[ビュー](/views.md)で機能します。グローバルヒントとして指定すると、クエリで定義されたヒントはビュー内で効果を発揮できます。グローバルヒントを指定するには、まず`QB_NAME`ヒントを使用してクエリブロック名を定義し、次に`ViewName@QueryBlockName`の形式で対象のヒントを追加します。

### ステップ1：`QB_NAME`ヒントを使用してビューのクエリブロック名を定義する {#step-1-define-the-query-block-name-of-the-view-using-the-qb-name-hint}

ビューの各クエリブロックに新しい名前を定義するには、[`QB_NAME`ヒント](#qb_name)を使用します。ビューの`QB_NAME`ヒントの定義は[クエリブロック](#qb_name)の定義と同じですが、構文は`QB_NAME(QB)`から`QB_NAME(QB, ViewName@QueryBlockName [.ViewName@QueryBlockName .ViewName@QueryBlockName ...])`に拡張されています。

> **Note:**
>
> `@QueryBlockName`と直後の`.ViewName@QueryBlockName`の間にはスペースがあります。そうしないと、`.ViewName@QueryBlockName`は`QueryBlockName`の一部と見なされます。たとえば、`QB_NAME(v2_1, v2@SEL_1 .@SEL_1)`は有効ですが、`QB_NAME(v2_1, v2@SEL_1.@SEL_1)`は正しく解析されません。

- 単一のビューとサブクエリがない単純なステートメントの場合、次の例ではビュー`v`の最初のクエリブロック名が指定されます。

  ```sql
  SELECT /* Comment: The name of the current query block is the default @SEL_1 */ * FROM v;
  ```

  ビュー`v`では、クエリステートメントから始まるビュー名リスト(`ViewName@QueryBlockName [.ViewName@QueryBlockName .ViewName@QueryBlockName ...]`)の最初のビュー名は`v@SEL_1`です。ビュー`v`の最初のクエリブロックは`QB_NAME(v_1, v@SEL_1 .@SEL_1)`として宣言できます。または、`@SEL_1`を省略して`QB_NAME(v_1, v)`として単純に書くこともできます。

  ```sql
  CREATE VIEW v AS SELECT /* Comment: The name of the current query block is the default @SEL_1 */ * FROM t;

  -- グローバルヒントを指定する
  SELECT /*+ QB_NAME(v_1, v) USE_INDEX(t@v_1, idx) */ * FROM v;
  ```

- ネストされたビューとサブクエリを持つ複雑なステートメントの場合、次の例ではビュー`v1`と`v2`の各クエリブロックの名前が指定されます。

  ```sql
  SELECT /* Comment: The name of the current query block is the default @SEL_1 */ * FROM v2 JOIN (
      SELECT /* Comment: The name of the current query block is the default @SEL_2 */ * FROM v2) vv;
  ```

  最初のビュー`v2`では、最初のクエリステートメントから始まるビュー名リストは`v2@SEL_1`です。2番目のビュー`v2`では、最初のビュー名は`v2@SEL_2`です。次の例では最初のビュー`v2`のみを考慮しています。

  ビュー`v2`の最初のクエリブロックは`QB_NAME(v2_1, v2@SEL_1 .@SEL_1)`として宣言できます。ビュー`v2`の2番目のクエリブロックは`QB_NAME(v2_2, v2@SEL_1 .@SEL_2)`として宣言できます。

  ```sql
  CREATE VIEW v2 AS
      SELECT * FROM t JOIN /* Comment: For view v2, the name of the current query block is the default @SEL_1. So, the current query block view list is v2@SEL_1 .@SEL_1 */
      (
          SELECT COUNT(*) FROM t1 JOIN v1 /* Comment: For view v2, the name of the current query block is the default @SEL_2. So, the current query block view list is v2@SEL_1 .@SEL_2 */
      ) tt;
  ```

  ビュー`v1`では、直前のステートメントから始まるビュー名リストの最初のビュー名は`v2@SEL_1 .v1@SEL_2`です。ビュー`v1`の最初のクエリブロックは`QB_NAME(v1_1, v2@SEL_1 .v1@SEL_2 .@SEL_1)`として宣言できます。ビュー`v1`の2番目のクエリブロックは`QB_NAME(v1_2, v2@SEL_1 .v1@SEL_2 .@SEL_2)`として宣言できます。

> **Note:**
>
> - ビューでグローバルヒントを使用するには、ビューで対応する`QB_NAME`ヒントを定義する必要があります。そうしないと、グローバルヒントは効果を発揮しません。
>
> - ヒントを使用してビュー内の複数のテーブル名を指定する場合、同じヒントに表示されるテーブル名が同じビューの同じクエリブロックにあることを確認する必要があります。
>
> - ビューの最も外側のクエリブロックに`QB_NAME`ヒントを定義する場合：
>
>   - `QB_NAME`のビューリストの最初の項目に`@SEL_`が明示的に宣言されていない場合、デフォルトは`QB_NAME`が定義されたクエリブロックの位置と一致します。つまり、クエリ`SELECT /*+ QB_NAME(qb1, v2) */ * FROM v2 JOIN (SELECT /*+ QB_NAME(qb2, v2) */ * FROM v2) vv;`は`SELECT /*+ QB_NAME(qb1, v2@SEL_1) */ * FROM v2 JOIN (SELECT /*+ QB_NAME(qb2, v2@SEL_2) */ * FROM v2) vv;`と同等です。
>   - `QB_NAME`の最初の項目以外の項目では、`@SEL_1`のみを省略できます。つまり、現在のビューの最初のクエリブロックで`@SEL_1`が宣言されている場合、`@SEL_1`を省略できます。それ以外の場合は、`@SEL_`を省略できません。前述の例では次のようになります。

> ```
> - ビュー`v2`の最初のクエリブロックは`QB_NAME(v2_1, v2)`として宣言できます。
> - ビュー`v2`の2番目のクエリブロックは`QB_NAME(v2_2, v2.@SEL_2)`として宣言できます。
> - ビュー`v1`の最初のクエリブロックは`QB_NAME(v1_1, v2.v1@SEL_2)`として宣言できます。
> - ビュー`v1`の2番目のクエリブロックは`QB_NAME(v1_2, v2.v1@SEL_2 .@SEL_2)`として宣言できます。
> ```

### ステップ2：ターゲットヒントを追加する {#step-2-add-the-target-hints}

ビューのクエリブロックに`QB_NAME`ヒントを定義した後、`ViewName@QueryBlockName`の形式で必要な[hints that take effect in query blocks](#hints-that-take-effect-in-query-blocks)を追加して、ビュー内でそれらを有効にすることができます。例：

- ビュー`v2`の最初のクエリブロックに`MERGE_JOIN()`ヒントを指定する：

  ```sql
  SELECT /*+ QB_NAME(v2_1, v2) merge_join(t@v2_1) */ * FROM v2;
  ```

- ビュー`v2`の2番目のクエリブロックに`MERGE_JOIN()`と`STREAM_AGG()`ヒントを指定する：

  ```sql
  SELECT /*+ QB_NAME(v2_2, v2.@SEL_2) merge_join(t1@v2_2) stream_agg(@v2_2) */ * FROM v2;
  ```

- ビュー`v1`の最初のクエリブロックに`HASH_JOIN()`ヒントを指定する：

  ```sql
  SELECT /*+ QB_NAME(v1_1, v2.v1@SEL_2) hash_join(t@v1_1) */ * FROM v2;
  ```

- ビュー`v1`の2番目のクエリブロックに`HASH_JOIN()`と`HASH_AGG()`ヒントを指定する：

  ```sql
  SELECT /*+ QB_NAME(v1_2, v2.v1@SEL_2 .@SEL_2) hash_join(t1@v1_2) hash_agg(@v1_2) */ * FROM v2;
  ```

## クエリ全体に影響を与えるヒント {#hints-that-take-effect-in-the-whole-query}

このカテゴリのヒントは、**最初の** `SELECT`、`UPDATE`、または`DELETE`キーワードの後にのみ続けることができます。これは、このクエリが実行されるときに指定されたシステム変数の値を変更することに相当します。ヒントの優先度は、既存のシステム変数よりも高いです。

> **Note:**
>
> このカテゴリのヒントには、オプションの隠し変数`@QB_NAME`もありますが、変数を指定しても、ヒントはクエリ全体に影響を与えます。

### NO\_INDEX\_MERGE() {#no-index-merge}

`NO_INDEX_MERGE()`ヒントは、オプティマイザのインデックスマージ機能を無効にします。

例えば、次のクエリはインデックスマージを使用しません：

```sql
select /*+ NO_INDEX_MERGE() */ * from t where t.a > 0 or t.b > 0;
```

このヒントに加えて、`tidb_enable_index_merge` システム変数の設定もこの機能を有効にするかどうかを制御します。

> **Note:**
>
> - `NO_INDEX_MERGE` は `USE_INDEX_MERGE` よりも優先度が高いです。両方のヒントが使用されると、`USE_INDEX_MERGE` は効果を発揮しません。
> - サブクエリの場合、`NO_INDEX_MERGE` はサブクエリの最も外側に配置されている場合にのみ効果を発揮します。

### USE\_TOJA(boolean\_value) {#use-toja-boolean-value}

`boolean_value` パラメータは `TRUE` または `FALSE` にすることができます。`USE_TOJA(TRUE)` ヒントは、オプティマイザに `in` 条件（サブクエリを含む）を結合および集計操作に変換するようにします。比較的、`USE_TOJA(FALSE)` ヒントはこの機能を無効にします。

例えば、次のクエリは `in (select t2.a from t2) subq` を対応する結合および集計操作に変換します。

```sql
select /*+ USE_TOJA(TRUE) */ t1.a, t1.b from t1 where t1.a in (select t2.a from t2) subq;
```

このヒントに加えて、`tidb_opt_insubq_to_join_and_agg` システム変数を設定することで、この機能を有効にするかどうかも制御できます。

### MAX\_EXECUTION\_TIME(N) {#max-execution-time-n}

`MAX_EXECUTION_TIME(N)` ヒントは、サーバーがそれを終了する前にステートメントが実行を許可される時間の制限 `N` (ミリ秒単位のタイムアウト値) を設定します。次のヒントでは、`MAX_EXECUTION_TIME(1000)` はタイムアウトが 1000 ミリ秒 (つまり、1 秒) であることを意味します。

```sql
select /*+ MAX_EXECUTION_TIME(1000) */ * from t1 inner join t2 where t1.id = t2.id;
```

このヒントに加えて、`global.max_execution_time` システム変数もステートメントの実行時間を制限することができます。

### MEMORY\_QUOTA(N) {#memory-quota-n}

`MEMORY_QUOTA(N)` ヒントは、ステートメントが使用できるメモリ量に上限値 `N` (MB または GB 単位の閾値) を設定します。ステートメントのメモリ使用量がこの制限を超えると、TiDB はステートメントの制限超過動作に基づいてログメッセージを生成するか、単にステートメントを終了します。

次のヒントでは、`MEMORY_QUOTA(1024 MB)` はメモリ使用量が 1024 MB に制限されることを意味します。

```sql
select /*+ MEMORY_QUOTA(1024 MB) */ * from t;
```

このヒントに加えて、[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) システム変数も、ステートメントのメモリ使用量を制限することができます。

### READ\_CONSISTENT\_REPLICA() {#read-consistent-replica}

`READ_CONSISTENT_REPLICA()` ヒントは、TiKV フォロワーノードから一貫したデータを読む機能を有効にします。例えば：

```sql
select /*+ READ_CONSISTENT_REPLICA() */ * from t;
```

このヒントに加えて、`tidb_replica_read`環境変数を`'follower'`または`'leader'`に設定することで、この機能を有効にするかどうかも制御できます。

### IGNORE\_PLAN\_CACHE() {#ignore-plan-cache}

`IGNORE_PLAN_CACHE()`ヒントは、現在の`prepare`ステートメントを処理する際に、オプティマイザにPlan Cacheを使用しないように指示します。

このヒントは、[prepare-plan-cache](/sql-prepared-plan-cache.md)が有効になっているときに、特定のタイプのクエリのPlan Cacheを一時的に無効にするために使用されます。

次の例では、`prepare`ステートメントを実行する際に、Plan Cacheが強制的に無効にされます。

```sql
prepare stmt from 'select  /*+ IGNORE_PLAN_CACHE() */ * from t where t.id = ?';
```

### STRAIGHT\_JOIN() {#straight-join}

`STRAIGHT_JOIN()` ヒントは、結合プランを生成する際に、`FROM` 句内のテーブル名の順にテーブルを結合するようにオプティマイザに思い出させます。

```sql
SELECT /*+ STRAIGHT_JOIN() */ * FROM t t1, t t2 WHERE t1.a = t2.a;
```

> **Note:**
>
> - `STRAIGHT_JOIN` has higher priority over `LEADING`. When both hints are used, `LEADING` does not take effect.
> - It is recommended to use the `LEADING` hint, which is more general than the `STRAIGHT_JOIN` hint.

### NTH\_PLAN(N) {#nth-plan-n}

The `NTH_PLAN(N)` hint reminds the optimizer to select the `N`th physical plan found during the physical optimization. `N` must be a positive integer.

If the specified `N` is beyond the search range of the physical optimization, TiDB will return a warning and select the optimal physical plan based on the strategy that ignores this hint.

This hint does not take effect when the cascades planner is enabled.

In the following example, the optimizer is forced to select the third physical plan found during the physical optimization:

```sql
SELECT /*+ NTH_PLAN(3) */ count(*) from t where a > 5;
```

> **Note:**
>
> `NTH_PLAN(N)`は主にテストに使用され、後のバージョンでの互換性は保証されていません。このヒントを **注意して** 使用してください。

### RESOURCE\_GROUP(resource\_group\_name) {#resource-group-resource-group-name}

`RESOURCE_GROUP(resource_group_name)`は[Resource Control](/tidb-resource-control.md)に使用され、リソースを分離します。このヒントは、指定されたリソースグループを使用して現在のステートメントを一時的に実行します。指定されたリソースグループが存在しない場合、このヒントは無視されます。

Example:

```sql
SELECT /*+ RESOURCE_GROUP(rg1) */ * FROM t limit 10;
```

## 一般的な問題のトラブルシューティング：ヒントが効果を発揮しない場合 {#troubleshoot-common-issues-that-hints-do-not-take-effect}

### ヒントが効果を発揮しない理由：MySQLコマンドラインクライアントがヒントを削除するため {#hints-do-not-take-effect-because-your-mysql-command-line-client-strips-hints}

5.7.7より前のMySQLコマンドラインクライアントは、デフォルトで最適化ヒントを削除します。これらの以前のバージョンでヒント構文を使用する場合は、クライアントを起動する際に `--comments` オプションを追加してください。例：

`mysql -h 127.0.0.1 -P 4000 -uroot --comments`。

### ヒントが効果を発揮しない理由：データベース名が指定されていないため {#hints-do-not-take-effect-because-the-database-name-is-not-specified}

接続を作成する際にデータベース名を指定しない場合、ヒントが効果を発揮しないことがあります。例：

TiDBに接続する際に、`mysql -h127.0.0.1 -P4000 -uroot` コマンドに `-D` オプションを指定せずに接続し、次のSQLステートメントを実行する場合：

```sql
SELECT /*+ use_index(t, a) */ a FROM test.t;
SHOW WARNINGS;
```

TiDBはデータベースを識別できないため、`t`テーブルの`use_index(t, a)`ヒントは効果がありません。

```sql
+---------+------+----------------------------------------------------------------------+
| Level   | Code | Message                                                              |
+---------+------+----------------------------------------------------------------------+
| Warning | 1815 | use_index(.t, a) is inapplicable, check whether the table(.t) exists |
+---------+------+----------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### ヒントは効果を持たないことがあります。クロステーブルクエリでデータベース名が明示的に指定されていないため {#hints-do-not-take-effect-because-the-database-name-is-not-explicitly-specified-in-cross-table-queries}

クロステーブルクエリを実行する際は、データベース名を明示的に指定する必要があります。そうしないと、ヒントが効果を持たないことがあります。例えば：

```sql
USE test1;
CREATE TABLE t1(a INT, KEY(a));
USE test2;
CREATE TABLE t2(a INT, KEY(a));
SELECT /*+ use_index(t1, a) */ * FROM test1.t1, t2;
SHOW WARNINGS;
```

先行する文において、テーブル`t1`が現在の`test2`データベースにないため、`use_index(t1, a)`ヒントは効果がありません。

```sql
+---------+------+----------------------------------------------------------------------------------+
| Level   | Code | Message                                                                          |
+---------+------+----------------------------------------------------------------------------------+
| Warning | 1815 | use_index(test2.t1, a) is inapplicable, check whether the table(test2.t1) exists |
+---------+------+----------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

この場合、`use_index(test1.t1, a)`を使用して、`use_index(t1, a)`の代わりにデータベース名を明示的に指定する必要があります。

### ヒントは間違った場所に配置されているため効果がありません {#hints-do-not-take-effect-because-they-are-placed-in-wrong-locations}

ヒントは特定のキーワードの直後に配置されていない場合、効果がありません。例えば：

```sql
SELECT * /*+ use_index(t, a) */ FROM t;
SHOW WARNINGS;
```

警告は次のとおりです：

```sql
+---------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                                                                                                 |
+---------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1064 | You have an error in your SQL syntax; check the manual that corresponds to your TiDB version for the right syntax to use [parser:8066]Optimizer hint can only be followed by certain keywords like SELECT, INSERT, etc. |
+---------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```

この場合、ヒントを`SELECT`キーワードの直後に配置する必要があります。詳細については、[Syntax](#syntax)セクションを参照してください。

### INL\_JOINヒントは照合の非互換性のために効果がありません {#inl-join-hint-does-not-take-effect-due-to-collation-incompatibility}

結合キーの照合が2つのテーブルの間で非互換性がある場合、`IndexJoin`演算子を使用してクエリを実行することはできません。この場合、[`INL_JOIN`ヒント](#inl_joint1_name--tl_name-)は効果がありません。例えば：

```sql
CREATE TABLE t1 (k varchar(8), key(k)) COLLATE=utf8mb4_general_ci;
CREATE TABLE t2 (k varchar(8), key(k)) COLLATE=utf8mb4_bin;
EXPLAIN SELECT /*+ tidb_inlj(t1) */ * FROM t1, t2 WHERE t1.k=t2.k;
```

実行計画は次のとおりです：

```sql
+-----------------------------+----------+-----------+----------------------+----------------------------------------------+
| id                          | estRows  | task      | access object        | operator info                                |
+-----------------------------+----------+-----------+----------------------+----------------------------------------------+
| HashJoin_19                 | 12487.50 | root      |                      | inner join, equal:[eq(test.t1.k, test.t2.k)] |
| ├─IndexReader_24(Build)     | 9990.00  | root      |                      | index:IndexFullScan_23                       |
| │ └─IndexFullScan_23        | 9990.00  | cop[tikv] | table:t2, index:k(k) | keep order:false, stats:pseudo               |
| └─IndexReader_22(Probe)     | 9990.00  | root      |                      | index:IndexFullScan_21                       |
|   └─IndexFullScan_21        | 9990.00  | cop[tikv] | table:t1, index:k(k) | keep order:false, stats:pseudo               |
+-----------------------------+----------+-----------+----------------------+----------------------------------------------+
5 rows in set, 1 warning (0.00 sec)
```

先行する文において、`t1.k` と `t2.k` の照合順序が互換性がない (`utf8mb4_general_ci` および `utf8mb4_bin` それぞれ) ため、`INL_JOIN` または `TIDB_INLJ` ヒントが効果を発揮しない。

```sql
SHOW WARNINGS;
+---------+------+----------------------------------------------------------------------------+
| Level   | Code | Message                                                                    |
+---------+------+----------------------------------------------------------------------------+
| Warning | 1815 | Optimizer Hint /*+ INL_JOIN(t1) */ or /*+ TIDB_INLJ(t1) */ is inapplicable |
+---------+------+----------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### `INL_JOIN` ヒントは、結合順序のために効果がありません {#inl-join-hint-does-not-take-effect-because-of-join-order}

[`INL_JOIN(t1, t2)`](#inl_joint1_name--tl_name-) または `TIDB_INLJ(t1, t2)` ヒントは、`t1` と `t2` が他のテーブルと結合するために `IndexJoin` 演算子として動作するように指示し、直接 `IndexJoin` 演算子を使用してそれらを結合するのではありません。例えば：

```sql
EXPLAIN SELECT /*+ inl_join(t1, t3) */ * FROM t1, t2, t3 WHERE t1.id = t2.id AND t2.id = t3.id AND t1.id = t3.id;
+---------------------------------+----------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                              | estRows  | task      | access object | operator info                                                                                                                                                           |
+---------------------------------+----------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| IndexJoin_16                    | 15625.00 | root      |               | inner join, inner:TableReader_13, outer key:test.t2.id, test.t1.id, inner key:test.t3.id, test.t3.id, equal cond:eq(test.t1.id, test.t3.id), eq(test.t2.id, test.t3.id) |
| ├─IndexJoin_34(Build)           | 12500.00 | root      |               | inner join, inner:TableReader_31, outer key:test.t2.id, inner key:test.t1.id, equal cond:eq(test.t2.id, test.t1.id)                                                     |
| │ ├─TableReader_40(Build)       | 10000.00 | root      |               | data:TableFullScan_39                                                                                                                                                   |
| │ │ └─TableFullScan_39          | 10000.00 | cop[tikv] | table:t2      | keep order:false, stats:pseudo                                                                                                                                          |
| │ └─TableReader_31(Probe)       | 10000.00 | root      |               | data:TableRangeScan_30                                                                                                                                                  |
| │   └─TableRangeScan_30         | 10000.00 | cop[tikv] | table:t1      | range: decided by [test.t2.id], keep order:false, stats:pseudo                                                                                                          |
| └─TableReader_13(Probe)         | 12500.00 | root      |               | data:TableRangeScan_12                                                                                                                                                  |
|   └─TableRangeScan_12           | 12500.00 | cop[tikv] | table:t3      | range: decided by [test.t2.id test.t1.id], keep order:false, stats:pseudo                                                                                               |
+---------------------------------+----------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

先行する例では、`t1`と`t3`は`IndexJoin`によって直接結合されていません。

`t1`と`t3`の間で直接`IndexJoin`を実行するには、まず[`LEADING(t1, t3)`ヒント](#leadingt1_name--tl_name-)を使用して、`t1`と`t3`の結合順序を指定し、その後`INL_JOIN`ヒントを使用して結合アルゴリズムを指定します。例えば：

```sql
EXPLAIN SELECT /*+ leading(t1, t3), inl_join(t3) */ * FROM t1, t2, t3 WHERE t1.id = t2.id AND t2.id = t3.id AND t1.id = t3.id;
+---------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------+
| id                              | estRows  | task      | access object | operator info                                                                                                       |
+---------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------+
| Projection_12                   | 15625.00 | root      |               | test.t1.id, test.t1.name, test.t2.id, test.t2.name, test.t3.id, test.t3.name                                        |
| └─HashJoin_21                   | 15625.00 | root      |               | inner join, equal:[eq(test.t1.id, test.t2.id) eq(test.t3.id, test.t2.id)]                                           |
|   ├─TableReader_36(Build)       | 10000.00 | root      |               | data:TableFullScan_35                                                                                               |
|   │ └─TableFullScan_35          | 10000.00 | cop[tikv] | table:t2      | keep order:false, stats:pseudo                                                                                      |
|   └─IndexJoin_28(Probe)         | 12500.00 | root      |               | inner join, inner:TableReader_25, outer key:test.t1.id, inner key:test.t3.id, equal cond:eq(test.t1.id, test.t3.id) |
|     ├─TableReader_34(Build)     | 10000.00 | root      |               | data:TableFullScan_33                                                                                               |
|     │ └─TableFullScan_33        | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo                                                                                      |
|     └─TableReader_25(Probe)     | 10000.00 | root      |               | data:TableRangeScan_24                                                                                              |
|       └─TableRangeScan_24       | 10000.00 | cop[tikv] | table:t3      | range: decided by [test.t1.id], keep order:false, stats:pseudo                                                      |
+---------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------+
9 rows in set (0.01 sec)
```

### ヒントを使用すると、`このクエリの適切な物理的な計画が見つかりません` エラーが発生します {#using-hints-causes-the-can-t-find-a-proper-physical-plan-for-this-query-error}

`このクエリの適切な物理的な計画が見つかりません` エラーは、次のシナリオで発生する可能性があります。

- クエリ自体が順番にインデックスを読む必要がない場合。つまり、このクエリに対して、ヒントを使用せずにインデックスを順番に読む計画を生成しない場合にこのエラーが発生します。この場合、`ORDER_INDEX` ヒントが指定されていると、このエラーが発生します。この問題を解決するには、対応する `ORDER_INDEX` ヒントを削除します。
- クエリが `NO_JOIN` 関連のヒントを使用して、すべての可能な結合方法を除外する場合。

```sql
CREATE TABLE t1 (a INT);
CREATE TABLE t2 (a INT);
EXPLAIN SELECT /*+ NO_HASH_JOIN(t1), NO_MERGE_JOIN(t1) */ * FROM t1, t2 WHERE t1.a=t2.a;
ERROR 1815 (HY000): Internal : Can't find a proper physical plan for this query
```

- システム変数[`tidb_opt_enable_hash_join`](/system-variables.md#tidb_opt_enable_hash_join-new-in-v656-and-v712)は`OFF`に設定されており、他のすべての結合タイプも除外されています。

```sql
CREATE TABLE t1 (a INT);
CREATE TABLE t2 (a INT);
set tidb_opt_enable_hash_join=off;
EXPLAIN SELECT /*+ NO_MERGE_JOIN(t1) */ * FROM t1, t2 WHERE t1.a=t2.a;
ERROR 1815 (HY000): Internal : Can't find a proper physical plan for this query
```
