---
title: TiFlash Query Result Materialization
summary: Learn how to save the query results of TiFlash in a transaction.
---

# TiFlash クエリ結果のマテリアライズ {#tiflash-query-result-materialization}

このドキュメントでは、TiFlash クエリ結果を指定された TiDB テーブルに `INSERT INTO SELECT` トランザクションで保存する方法を紹介します。

v6.5.0 以降、TiDB は TiFlash クエリ結果のテーブルへの保存、つまり TiFlash クエリ結果のマテリアライズをサポートしています。`INSERT INTO SELECT` 文の実行中に、TiDB が `SELECT` サブクエリを TiFlash にプッシュダウンする場合、TiFlash クエリ結果は `INSERT INTO` 句で指定された TiDB テーブルに保存されます。v6.5.0 より前の TiDB バージョンでは、TiFlash クエリ結果は読み取り専用なので、TiFlash クエリ結果を保存するには、アプリケーションレベルで取得し、別のトランザクションやプロセスで保存する必要があります。

> **Note:**
>
> デフォルトでは（[`tidb_allow_mpp = ON`](/system-variables.md#tidb_allow_mpp-new-in-v50)）、オプティマイザーは [SQL モード](/sql-mode.md) と TiFlash レプリカのコスト推定に基づいて、クエリを TiFlash にプッシュダウンするかどうかを適切に決定します。
>
> - 現在のセッションの [SQL モード](/sql-mode.md) が strict でない場合（つまり、`sql_mode` の値に `STRICT_TRANS_TABLES` または `STRICT_ALL_TABLES` が含まれない場合）、オプティマイザーは TiFlash レプリカのコスト推定に基づいて、`INSERT INTO SELECT` の `SELECT` サブクエリを TiFlash にプッシュダウンするかどうかを適切に決定します。このモードでは、オプティマイザーのコスト推定を無視し、クエリを強制的に TiFlash にプッシュダウンするには、[`tidb_enforce_mpp`](/system-variables.md#tidb_enforce_mpp-new-in-v51) システム変数を `ON` に設定できます。
> - 現在のセッションの [SQL モード](/sql-mode.md) が strict である場合（つまり、`sql_mode` の値に `STRICT_TRANS_TABLES` または `STRICT_ALL_TABLES` のいずれかが含まれる場合）、`INSERT INTO SELECT` の `SELECT` サブクエリは TiFlash にプッシュダウンできません。

`INSERT INTO SELECT` の構文は次のとおりです。

```sql
INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    SELECT ...
    [ON DUPLICATE KEY UPDATE assignment_list]value:
    {expr | DEFAULT}

assignment:
    col_name = valueassignment_list:
    assignment [, assignment] ...
```

例えば、次の`INSERT INTO SELECT`文を使用して、`SELECT`句でテーブル`t1`からクエリ結果をテーブル`t2`に保存することができます。

```sql
INSERT INTO t2 (name, country)
SELECT app_name, country FROM t1;
```

## 典型的で推奨される使用シナリオ {#typical-and-recommended-usage-scenarios}

- 効率的なBIソリューション

多くのBIアプリケーションでは、分析クエリリクエストが非常に重くなります。例えば、多数のユーザーが同時にレポートにアクセスして更新する場合、BIアプリケーションは多数の並行クエリリクエストを処理する必要があります。このような状況を効果的に処理するために、`INSERT INTO SELECT`を使用してレポートのクエリ結果をTiDBテーブルに保存することができます。その後、エンドユーザーはレポートが更新されるときに結果テーブルから直接データをクエリできるため、複数の繰り返し計算や分析を回避することができます。同様に、歴史的な分析結果を保存することで、長期間の歴史データ分析の計算量をさらに減らすことができます。例えば、日々の売上利益を分析するために使用されるレポート`A`がある場合、`INSERT INTO SELECT`を使用してレポート`A`の結果を結果テーブル`T`に保存することができます。その後、過去1ヶ月の売上利益を分析するためにレポート`B`を生成する必要がある場合、結果テーブル`T`の日次分析結果を直接使用することができます。これにより、計算量が大幅に減少するだけでなく、クエリの応答速度が向上し、システムの負荷が軽減されます。

- TiFlashを使用したオンラインアプリケーションの提供

TiFlashがサポートする同時リクエストの数は、データの量やクエリの複雑さによって異なりますが、通常は100 QPSを超えません。`INSERT INTO SELECT`を使用してTiFlashのクエリ結果を保存し、その後クエリ結果テーブルを使用して高度に並行したオンラインリクエストをサポートすることができます。結果テーブルのデータは、低頻度（例えば0.5秒間隔）でバックグラウンドで更新することができます。これはTiFlashの並行性制限を下回るため、データの新鮮さを高いレベルで維持することができます。

## 実行プロセス {#execution-process}

- `INSERT INTO SELECT`ステートメントの実行中、TiFlashはまずクラスター内のTiDBサーバーに`SELECT`句のクエリ結果を返し、その後結果をターゲットテーブルに書き込みます。ターゲットテーブルにはTiFlashレプリカがある場合があります。
- `INSERT INTO SELECT`ステートメントの実行はACIDプロパティを保証します。

## 制限事項 {#restrictions}

<CustomContent platform="tidb">

- `INSERT INTO SELECT`ステートメントのTiDBメモリ制限は、システム変数[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)を使用して調整することができます。v6.5.0以降、トランザクションメモリサイズを制御するために[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)を使用することは推奨されません。

  詳細については、[TiDBメモリ制御](/configure-memory-usage.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- `INSERT INTO SELECT`ステートメントのTiDBメモリ制限は、システム変数[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)を使用して調整することができます。v6.5.0以降、トランザクションメモリサイズを制御するために[`txn-total-size-limit`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#txn-total-size-limit)を使用することは推奨されません。

  詳細については、[TiDBメモリ制御](https://docs.pingcap.com/tidb/stable/configure-memory-usage)を参照してください。

</CustomContent>

- TiDBには`INSERT INTO SELECT`ステートメントの並行性に対する厳密な制限はありませんが、次のようなプラクティスを考慮することを推奨します。

  - 「書き込みトランザクション」が大きい場合、例えば1 GiBに近い場合、並行性を10を超えないように制御することを推奨します。
  - 「書き込みトランザクション」が小さい場合、例えば100 MiB未満の場合、並行性を30を超えないように制御することを推奨します。
  - テスト結果と具体的な状況に基づいて並行性を決定します。

## 例 {#example}

データ定義：

```sql
CREATE TABLE detail_data (
    ts DATETIME,                -- Fee generation time
    customer_id VARCHAR(20),    -- Customer ID
    detail_fee DECIMAL(20,2));  -- Amount of fee

CREATE TABLE daily_data (
    rec_date DATE,              -- Date when data is collected
    customer_id VARCHAR(20),    -- Customer ID
    daily_fee DECIMAL(20,2));   -- Amount of fee for per day

ALTER TABLE detail_data SET TIFLASH REPLICA 1;
ALTER TABLE daily_data SET TIFLASH REPLICA 1;

-- ... (detail_data table continues updating)
INSERT INTO detail_data(ts,customer_id,detail_fee) VALUES
('2023-1-1 12:2:3', 'cus001', 200.86),
('2023-1-2 12:2:3', 'cus002', 100.86),
('2023-1-3 12:2:3', 'cus002', 2200.86),
('2023-1-4 12:2:3', 'cus003', 2020.86),
('2023-1-5 12:2:3', 'cus003', 1200.86),
('2023-1-6 12:2:3', 'cus002', 20.86),
('2023-1-7 12:2:3', 'cus004', 120.56),
('2023-1-8 12:2:3', 'cus005', 320.16);

-- Execute the following SQL statement 13 times to insert a cumulative total of 65,536 rows into the table.
INSERT INTO detail_data SELECT * FROM detail_data;
```

毎日の分析結果を保存する：

```sql
SET @@sql_mode='NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO';

INSERT INTO daily_data (rec_date, customer_id, daily_fee)
SELECT DATE(ts), customer_id, sum(detail_fee) FROM detail_data WHERE DATE(ts) > DATE('2023-1-1 12:2:3') GROUP BY DATE(ts), customer_id;
```

毎月のデータを、日次分析データに基づいて分析します。

```sql
SELECT MONTH(rec_date), customer_id, sum(daily_fee) FROM daily_data GROUP BY MONTH(rec_date), customer_id;
```

前述の例では、日々の分析結果を具現化し、日次結果テーブルに保存し、月次データ分析を加速させ、データ分析の効率を向上させます。
