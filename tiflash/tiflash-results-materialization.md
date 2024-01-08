---
title: TiFlash Query Result Materialization
summary: Learn how to save the query results of TiFlash in a transaction.
---

# TiFlash クエリ結果のマテリアリゼーション {#tiflash-query-result-materialization}

このドキュメントでは、`INSERT INTO SELECT` トランザクションで TiFlash クエリ結果を指定された TiDB テーブルに保存する方法について紹介します。

v6.5.0 から、TiDB は TiFlash クエリ結果をテーブルに保存すること、つまり TiFlash クエリ結果のマテリアリゼーションをサポートしています。`INSERT INTO SELECT` ステートメントの実行中、TiDB が `SELECT` サブクエリを TiFlash にプッシュダウンする場合、TiFlash クエリ結果は `INSERT INTO` 句で指定された TiDB テーブルに保存できます。v6.5.0 より前の TiDB バージョンでは、TiFlash クエリ結果は読み取り専用なので、TiFlash クエリ結果を保存するには、アプリケーションレベルで取得し、別のトランザクションまたはプロセスで保存する必要があります。

> **Note:**
>
> デフォルトでは ([`tidb_allow_mpp = ON`](/system-variables.md#tidb_allow_mpp-new-in-v50))、オプティマイザは [SQL モード](/sql-mode.md) と TiFlash レプリカのコスト見積もりに基づいてクエリを TiFlash にプッシュダウンするかどうかをインテリジェントに決定します。
>
> - 現在のセッションの [SQL モード](/sql-mode.md) が厳密でない場合（つまり、`sql_mode` の値に `STRICT_TRANS_TABLES` と `STRICT_ALL_TABLES` が含まれていない場合）、オプティマイザは TiFlash レプリカのコスト見積もりに基づいて `INSERT INTO SELECT` の `SELECT` サブクエリを TiFlash にプッシュダウンするかどうかをインテリジェントに決定します。このモードでは、オプティマイザのコスト見積もりを無視し、クエリを強制的に TiFlash にプッシュダウンする場合は、[`tidb_enforce_mpp`](/system-variables.md#tidb_enforce_mpp-new-in-v51) システム変数を `ON` に設定できます。
> - 現在のセッションの [SQL モード](/sql-mode.md) が厳密である場合（つまり、`sql_mode` の値に `STRICT_TRANS_TABLES` または `STRICT_ALL_TABLES` のいずれかが含まれている場合）、`INSERT INTO SELECT` の `SELECT` サブクエリは TiFlash にプッシュダウンできません。

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

たとえば、次の `INSERT INTO SELECT` ステートメントを使用して、テーブル `t1` の `SELECT` 句からのクエリ結果をテーブル `t2` に保存できます。

```sql
INSERT INTO t2 (name, country)
SELECT app_name, country FROM t1;
```

## 典型的で推奨される使用シナリオ {#typical-and-recommended-usage-scenarios}

- 効率的な BI ソリューション

  多くの BI アプリケーションでは、分析クエリリクエストが非常に重くなります。たとえば、多くのユーザーが同時にレポートにアクセスして更新する場合、BI アプリケーションは多くの同時クエリリクエストを処理する必要があります。このような状況に効果的に対処するために、`INSERT INTO SELECT` を使用してレポートのクエリ結果を TiDB テーブルに保存できます。その後、レポートが更新されると、エンドユーザーは結果テーブルからデータを直接クエリできるため、複数の繰り返し計算と分析を回避できます。同様に、歴史的な分析結果を保存することで、長期間の歴史データ分析の計算量をさらに減らすことができます。たとえば、日次売上利益を分析するためのレポート `A` がある場合、`INSERT INTO SELECT` を使用してレポート `A` の結果を結果テーブル `T` に保存できます。その後、過去の月の売上利益を分析するためのレポート `B` を生成する必要がある場合、テーブル `T` の日次分析結果を直接使用できます。これにより、計算量が大幅に減少し、クエリ応答速度が向上し、システム負荷が軽減されます。

- TiFlash でオンラインアプリケーションを提供する

  TiFlash がサポートする同時リクエスト数は、データのボリュームとクエリの複雑さに依存しますが、通常は 100 QPS を超えることはありません。TiFlash クエリ結果を保存し、その後クエリ結果テーブルを使用して高度に同時実行されるオンラインリクエストをサポートできます。結果テーブルのデータは、バックグラウンドで低頻度（たとえば、0.5 秒間隔）で更新できます。これは TiFlash の同時実行制限を大幅に下回るため、データの新鮮さを維持しながら高いレベルのデータを提供できます。

## 実行プロセス {#execution-process}

- `INSERT INTO SELECT` ステートメントの実行中、TiFlash はまず `SELECT` 句のクエリ結果をクラスタ内の TiDB サーバーに返し、その後結果をターゲットテーブルに書き込みます。ターゲットテーブルには TiFlash レプリカが存在する可能性があります。
- `INSERT INTO SELECT` ステートメントの実行は ACID 特性を保証します。

## 制限 {#restrictions}

<CustomContent platform="tidb">

- `INSERT INTO SELECT` ステートメントの TiDB メモリ制限は、システム変数 [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) を使用して調整できます。v6.5.0 以降、トランザクションメモリサイズを制御するために [`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit) を使用することは推奨されません。

  詳細については、[TiDB メモリ制御](/configure-memory-usage.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- `INSERT INTO SELECT` ステートメントの TiDB メモリ制限は、システム変数 [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) を使用して調整できます。v6.5.0 以降、トランザクションメモリサイズを制御するために [`txn-total-size-limit`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#txn-total-size-limit) を使用することは推奨されません。

  詳細については、[TiDB メモリ制御](https://docs.pingcap.com/tidb/stable/configure-memory-usage) を参照してください。

</CustomContent>

- `INSERT INTO SELECT` ステートメントの同時実行には TiDB に厳格な制限はありませんが、次の慣行を考慮することをお勧めします。

  - "書き込みトランザクション" が大きい場合、たとえば 1 GiB に近い場合、同時実行を 10 以下に制御することをお勧めします。
  - "書き込みトランザクション" が小さい場合、たとえば 100 MiB 未満の場合、同時実行を 30 以下に制御することをお勧めします。
  - 同時実行は、テスト結果と具体的な状況に基づいて決定してください。

## 例 {#example}

データ定義:

```sql
CREATE TABLE detail_data (
    ts DATETIME,                -- 費用発生時刻
    customer_id VARCHAR(20),    -- 顧客 ID
    detail_fee DECIMAL(20,2));  -- 費用の金額

CREATE TABLE daily_data (
    rec_date DATE,              -- データ収集日
    customer_id VARCHAR(20),    -- 顧客 ID
    daily_fee DECIMAL(20,2));   -- 1 日あたりの費用

ALTER TABLE detail_data SET TIFLASH REPLICA 1;
ALTER TABLE daily_data SET TIFLASH REPLICA 1;

-- ...（detail_data テーブルは引き続き更新されます）
INSERT INTO detail_data(ts,customer_id,detail_fee) VALUES
('2023-1-1 12:2:3', 'cus001', 200.86),
('2023-1-2 12:2:3', 'cus002', 100.86),
('2023-1-3 12:2:3', 'cus002', 2200.86),
('2023-1-4 12:2:3', 'cus003', 2020.86),
('2023-1-5 12:2:3', 'cus003', 1200.86),
('2023-1-6 12:2:3', 'cus002', 20.86),
('2023-1-7 12:2:3', 'cus004', 120.56),
('2023-1-8 12:2:3', 'cus005', 320.16);

-- 次の SQL ステートメントを 13 回実行して、テーブルに累積合計 65,536 行を挿入します。
INSERT INTO detail_data SELECT * FROM detail_data;
```

日次分析結果を保存:

```sql
SET @@sql_mode='NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO';

INSERT INTO daily_data (rec_date, customer_id, daily_fee)
SELECT DATE(ts), customer_id, sum(detail_fee) FROM detail_data WHERE DATE(ts) > DATE('2023-1-1 12:2:3') GROUP BY DATE(ts), customer_id;
```

日次分析データに基づいて月次データを分析:

```sql
SELECT MONTH(rec_date), customer_id, sum(daily_fee) FROM daily_data GROUP BY MONTH(rec_date), customer_id;
```

上記の例では、日次分析結果をマテリアライズし、それを日次結果テーブルに保存し、その基に月次データ分析を加速し、データ分析効率を向上させています。
