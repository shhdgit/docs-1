---
title: Periodically Delete Data Using TTL (Time to Live)
summary: Time to live (TTL) is a feature that allows you to manage TiDB data lifetime at the row level. In this document, you can learn how to use TTL to automatically expire and delete old data.
---

# TTL（Time to Live）を使用して期限切れのデータを定期的に削除する {#periodically-delete-expired-data-using-ttl-time-to-live}

TTL（Time to Live）は、TiDBのデータのライフタイムを行レベルで管理する機能です。TTL属性を持つテーブルでは、TiDBが自動的にデータのライフタイムをチェックし、行レベルで期限切れのデータを削除します。この機能は、特定のシナリオでストレージスペースを効率的に節約し、パフォーマンスを向上させることができます。

以下は、TTLの一般的な使用シナリオです：

- 頻繁に検証コードや短縮URLを削除する。
- 不要な過去の注文を定期的に削除する。
- 計算の中間結果を自動的に削除する。

TTLは、オンラインの読み書きワークロードに影響を与えることなく、ユーザーが不要なデータを定期的かつタイムリーにクリーンアップするのを支援するように設計されています。TTLは、テーブル単位で異なるジョブを異なるTiDBノードに並行してデータを削除するように調整します。TTLは、すべての期限切れのデータがすぐに削除されることを保証しません。つまり、一部のデータが期限切れになっても、クライアントは期限切れのデータが削除されるまで、そのデータを読み取ることができます。

## 構文 {#syntax}

[`CREATE TABLE`](/sql-statements/sql-statement-create-table.md)または[`ALTER TABLE`](/sql-statements/sql-statement-alter-table.md)ステートメントを使用して、テーブルのTTL属性を構成できます。

### TTL属性を持つテーブルを作成する {#create-a-table-with-a-ttl-attribute}

- TTL属性を持つテーブルを作成する：

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP
  ) TTL = `created_at` + INTERVAL 3 MONTH;
  ```

  上記の例では、テーブル`t1`を作成し、TTLタイムスタンプ列として`created_at`を指定し、データの作成時刻を示します。また、`INTERVAL 3 MONTH`を使用して、テーブル内の行が許可される最長の生存期間を3ヶ月に設定します。この値を超えるデータは後で削除されます。

- `TTL_ENABLE`属性を設定して、期限切れのデータをクリーンアップする機能を有効または無効にする：

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP
  ) TTL = `created_at` + INTERVAL 3 MONTH TTL_ENABLE = 'OFF';
  ```

  `TTL_ENABLE`が`OFF`に設定されている場合、他のTTLオプションが設定されていても、TiDBはこのテーブルの期限切れのデータを自動的にクリーンアップしません。TTL属性を持つテーブルでは、`TTL_ENABLE`はデフォルトで`ON`に設定されています。

- MySQLとの互換性を保つために、コメントを使用してTTL属性を設定することもできます：

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP
  ) /*T![ttl] TTL = `created_at` + INTERVAL 3 MONTH TTL_ENABLE = 'OFF'*/;
  ```

  TiDBでは、テーブルのTTL属性を使用するか、コメントを使用してTTLを構成することは同じです。MySQLでは、コメントは無視され、通常のテーブルが作成されます。

### テーブルのTTL属性を変更する {#modify-the-ttl-attribute-of-a-table}

- テーブルのTTL属性を変更する：

  ```sql
  ALTER TABLE t1 TTL = `created_at` + INTERVAL 1 MONTH;
  ```

  上記のステートメントを使用して、既存のTTL属性を持つテーブルを変更するか、TTL属性を持たないテーブルにTTL属性を追加することができます。

- TTL属性を持つテーブルの`TTL_ENABLE`の値を変更する：

  ```sql
  ALTER TABLE t1 TTL_ENABLE = 'OFF';
  ```

- テーブルのすべてのTTL属性を削除する：

  ```sql
  ALTER TABLE t1 REMOVE TTL;
  ```

### TTLとデータ型のデフォルト値 {#ttl-and-the-default-values-of-data-types}

TTLを[データ型のデフォルト値](/data-type-default-values.md)と一緒に使用することができます。以下は、2つの一般的な使用例です：

- `DEFAULT CURRENT_TIMESTAMP`を使用して、列のデフォルト値を現在の作成時刻として指定し、この列をTTLタイムスタンプ列として使用します。3ヶ月前に作成されたレコードは期限切れです：

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  ) TTL = `created_at` + INTERVAL 3 MONTH;
  ```

- 列のデフォルト値を作成時刻または最新の更新時刻として指定し、この列をTTLタイムスタンプ列として使用します。3ヶ月間更新されていないレコードは期限切れです：

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  ) TTL = `created_at` + INTERVAL 3 MONTH;
  ```

### TTLと生成された列 {#ttl-and-generated-columns}

TTLを[生成された列](/generated-columns.md)と一緒に使用して、複雑な期限切れのルールを構成することができます。例：

// 入力コンテンツ終了

```sql
CREATE TABLE message (
    id int PRIMARY KEY,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    image bool,
    expire_at TIMESTAMP AS (IF(image,
            created_at + INTERVAL 5 DAY,
            created_at + INTERVAL 30 DAY
    ))
) TTL = `expire_at` + INTERVAL 0 DAY;
```

前述のステートメントでは、`expire_at`カラムをTTLタイムスタンプカラムとして使用し、メッセージのタイプに応じて有効期限を設定します。メッセージが画像の場合は5日で、それ以外の場合は30日で有効期限が切れます。

[JSONタイプ](/data-type-json.md)とTTLを一緒に使用することができます。例えば：

```sql
CREATE TABLE orders (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    order_info JSON,
    created_at DATE AS (JSON_EXTRACT(order_info, '$.created_at')) VIRTUAL
) TTL = `created_at` + INTERVAL 3 month;
```

## TTLジョブ {#ttl-job}

各テーブルにはTTL属性があり、TiDB内部で期限切れのデータをクリーンアップするバックグラウンドジョブがスケジュールされます。テーブルの`TTL_JOB_INTERVAL`属性を設定することで、これらのジョブの実行間隔をカスタマイズすることができます。次の例では、テーブル`orders`のバックグラウンドクリーンアップジョブを24時間ごとに実行するように設定しています。

```sql
ALTER TABLE orders TTL_JOB_INTERVAL = '24h';
```

`TTL_JOB_INTERVAL`はデフォルトで`1h`に設定されています。

TTLジョブを実行する際、TiDBはリージョンを最小単位として最大64のタスクにテーブルを分割します。これらのタスクは分散して実行されます。システム変数[`tidb_ttl_running_tasks`](/system-variables.md#tidb_ttl_running_tasks-new-in-v700)を設定することで、クラスタ全体で並行して実行されるTTLタスクの数を制限することができます。ただし、すべての種類のテーブルのTTLジョブをタスクに分割することはできません。どの種類のテーブルのTTLジョブがタスクに分割できないかの詳細については、[制限事項](#limitations)セクションを参照してください。

TTLジョブの実行を無効にするには、`TTL_ENABLE='OFF'`テーブルオプションを設定するだけでなく、[`tidb_ttl_job_enable`](/system-variables.md#tidb_ttl_job_enable-new-in-v650)グローバル変数を設定することでクラスタ全体でTTLジョブの実行を無効にすることもできます。

```sql
SET @@global.tidb_ttl_job_enable = OFF;
```

あるシナリオでは、TTLジョブを特定の時間枠でのみ実行するようにしたい場合があります。その場合、[`tidb_ttl_job_schedule_window_start_time`](/system-variables.md#tidb_ttl_job_schedule_window_start_time-new-in-v650)と[`tidb_ttl_job_schedule_window_end_time`](/system-variables.md#tidb_ttl_job_schedule_window_end_time-new-in-v650)のグローバル変数を設定して、時間枠を指定することができます。例えば：

```sql
SET @@global.tidb_ttl_job_schedule_window_start_time = '01:00 +0000';
SET @@global.tidb_ttl_job_schedule_window_end_time = '05:00 +0000';
```

前述のステートメントでは、TTLジョブをUTCの1時から5時の間にのみスケジュールできるようになっています。デフォルトでは、タイムウィンドウは`00:00 +0000`から`23:59 +0000`に設定されており、ジョブをいつでもスケジュールできるようになっています。

## オブザーバビリティ {#observability}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> このセクションはTiDB Self-Hostedにのみ適用されます。現在、TiDB CloudではTTLメトリクスは提供されていません。

</CustomContent>

TiDBは定期的にTTLに関するランタイム情報を収集し、これらのメトリクスをGrafanaで視覚化したチャートを提供します。GrafanaのTiDB -> TTLパネルでこれらのメトリクスを確認できます。

<CustomContent platform="tidb">

メトリクスの詳細については、[TiDBモニタリングメトリクス](/grafana-tidb-dashboard.md)のTTLセクションを参照してください。

</CustomContent>

さらに、TiDBはTTLジョブに関する詳細情報を取得するための3つのテーブルを提供しています：

- `mysql.tidb_ttl_table_status`テーブルには、すべてのTTLテーブルの以前に実行されたTTLジョブと現在実行中のTTLジョブに関する情報が含まれています。

  ```sql
  MySQL [(none)]> SELECT * FROM mysql.tidb_ttl_table_status LIMIT 1\G;
  *************************** 1. row ***************************
                        table_id: 85
                parent_table_id: 85
                table_statistics: NULL
                    last_job_id: 0b4a6d50-3041-4664-9516-5525ee6d9f90
            last_job_start_time: 2023-02-15 20:43:46
            last_job_finish_time: 2023-02-15 20:44:46
            last_job_ttl_expire: 2023-02-15 19:43:46
                last_job_summary: {"total_rows":4369519,"success_rows":4369519,"error_rows":0,"total_scan_task":64,"scheduled_scan_task":64,"finished_scan_task":64}
                  current_job_id: NULL
            current_job_owner_id: NULL
          current_job_owner_addr: NULL
      current_job_owner_hb_time: NULL
          current_job_start_time: NULL
          current_job_ttl_expire: NULL
              current_job_state: NULL
              current_job_status: NULL
  current_job_status_update_time: NULL
  1 row in set (0.040 sec)
  ```

  カラム`table_id`はパーティションテーブルのIDであり、`parent_table_id`はテーブルのIDであり、`infomation_schema.tables`のIDと対応しています。テーブルがパーティションテーブルでない場合、2つのIDは同じです。

  カラム`{last, current}_job_{start_time, finish_time, ttl_expire}`は、それぞれ最後に実行されたTTLタスクまたは現在の実行に使用される開始時刻、終了時刻、および有効期限を示します。`last_job_summary`カラムには、最後のTTLタスクの実行状況が含まれており、総行数、成功した行数、および失敗した行数が記載されています。

- `mysql.tidb_ttl_task`テーブルには、現在実行中のTTLサブタスクに関する情報が含まれています。TTLジョブは多数のサブタスクに分割され、このテーブルには現在実行中のサブタスクが記録されます。

- `mysql.tidb_ttl_job_history`テーブルには、実行されたTTLジョブに関する情報が含まれています。TTLジョブの履歴は90日間保持されます。

  ```sql
  MySQL [(none)]> SELECT * FROM mysql.tidb_ttl_job_history LIMIT 1\G;
  *************************** 1. row ***************************
            job_id: f221620c-ab84-4a28-9d24-b47ca2b5a301
          table_id: 85
    parent_table_id: 85
      table_schema: test_schema
        table_name: TestTable
    partition_name: NULL
        create_time: 2023-02-15 17:43:46
        finish_time: 2023-02-15 17:45:46
        ttl_expire: 2023-02-15 16:43:46
      summary_text: {"total_rows":9588419,"success_rows":9588419,"error_rows":0,"total_scan_task":63,"scheduled_scan_task":63,"finished_scan_task":63}
      expired_rows: 9588419
      deleted_rows: 9588419
  error_delete_rows: 0
            status: finished
  ```

  カラム`table_id`はパーティションテーブルのIDであり、`parent_table_id`はテーブルのIDであり、`infomation_schema.tables`のIDと対応しています。`table_schema`、`table_name`、および`partition_name`は、それぞれデータベース、テーブル名、およびパーティション名に対応します。`create_time`、`finish_time`、および`ttl_expire`は、TTLタスクの作成時刻、終了時刻、および有効期限を示します。`expired_rows`および`deleted_rows`は、有効期限が切れた行数と正常に削除された行数を示します。

## TiDBツールとの互換性 {#compatibility-with-tidb-tools}

TTLは、他のTiDBの移行、バックアップ、およびリカバリツールと併用できます。

| ツール名             | 最小サポートバージョン | 説明                                                                                                                                                               |
| ---------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| バックアップとリストア (BR) | v6.6.0      | BRを使用してデータをリストアすると、テーブルの `TTL_ENABLE` 属性が `OFF` に設定されます。これにより、バックアップとリストア後にTiDBが期限切れのデータをすぐに削除することが防止されます。各テーブルの `TTL_ENABLE` 属性を手動でオンにする必要があります。                |
| TiDB Lightning   | v6.6.0      | TiDB Lightningを使用してデータをインポートすると、インポートされたテーブルの `TTL_ENABLE` 属性が `OFF` に設定されます。これにより、インポート後にTiDBが期限切れのデータをすぐに削除することが防止されます。各テーブルの `TTL_ENABLE` 属性を手動でオンにする必要があります。 |
| TiCDC            | v7.0.0      | 下流の `TTL_ENABLE` 属性は自動的に `OFF` に設定されます。上流のTTL削除は下流に同期されます。そのため、重複削除を防ぐために、下流のテーブルの `TTL_ENABLE` 属性は強制的に `OFF` に設定されます。                                           |

## SQLとの互換性 {#compatibility-with-sql}

| 機能名                                                                         | 説明                                                                                                                                                                         |
| :-------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`FLASHBACK TABLE`](/sql-statements/sql-statement-flashback-table.md)       | `FLASHBACK TABLE` はテーブルの `TTL_ENABLE` 属性を `OFF` に設定します。これにより、FLASHBACK後にTiDBが期限切れのデータをすぐに削除することが防止されます。各テーブルの `TTL_ENABLE` 属性を手動でオンにする必要があります。                             |
| [`FLASHBACK DATABASE`](/sql-statements/sql-statement-flashback-database.md) | `FLASHBACK DATABASE` はテーブルの `TTL_ENABLE` 属性を `OFF` に設定しますが、`TTL_ENABLE` 属性は変更されません。これにより、FLASHBACK後にTiDBが期限切れのデータをすぐに削除することが防止されます。各テーブルの `TTL_ENABLE` 属性を手動でオンにする必要があります。 |
| [`FLASHBACK CLUSTER`](/sql-statements/sql-statement-flashback-cluster.md)   | `FLASHBACK CLUSTER` はシステム変数 [`TIDB_TTL_JOB_ENABLE`](/system-variables.md#tidb_ttl_job_enable-new-in-v650) を `OFF` に設定し、`TTL_ENABLE` 属性の値は変更されません。                          |

## 制限事項 {#limitations}

現在、TTL機能には次の制限があります。

- TTL属性は、ローカル一時テーブルとグローバル一時テーブルを含む一時テーブルに設定できません。
- TTL属性を持つテーブルは、外部キー制約の主テーブルとして他のテーブルに参照されることはできません。
- すべての期限切れのデータがすぐに削除されることは保証されません。期限切れのデータが削除されるタイミングは、バックグラウンドのクリーンアップジョブのスケジューリング間隔とスケジューリングウィンドウに依存します。
- [クラスター化インデックス](/clustered-indexes.md)を使用するテーブルの場合、主キーが整数型またはバイナリ文字列型でない場合、TTLジョブを複数のタスクに分割することはできません。これにより、TTLジョブが単一のTiDBノードで順次実行されることになります。テーブルに大量のデータが含まれる場合、TTLジョブの実行が遅くなる可能性があります。
- TTLは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では使用できません。

## よくある質問 {#faqs}

<CustomContent platform="tidb">

- データサイズを比較的安定させるために、削除が十分に速いかどうかをどのように判断できますか？

  [Grafana `TiDB` ダッシュボード](/grafana-tidb-dashboard.md)では、パネル `TTL Insert Rows Per Hour` に前の1時間に挿入された行の総数が記録されます。対応する `TTL Delete Rows Per Hour` には、前の1時間にTTLタスクによって削除された行の総数が記録されます。長期間 `TTL Insert Rows Per Hour` が `TTL Delete Rows Per Hour` よりも高い場合、挿入の速度が削除の速度よりも高く、データの総量が増加することを意味します。例えば：

  ![insert fast example](/media/ttl/insert-fast.png)

  ただし、TTLは期限切れの行がすぐに削除されることを保証しないため、現在挿入されている行は将来のTTLタスクで削除されます。そのため、短期間においてTTLの削除速度が挿入速度よりも低い場合でも、TTLの速度が遅すぎるとは限りません。状況を考慮する必要があります。

- TTLタスクのボトルネックがスキャンにあるか削除にあるかをどのように判断できますか？

  `TTL Scan Worker Time By Phase` と `TTL Delete Worker Time By Phase` パネルを見てください。スキャンワーカーが `dispatch` フェーズに長い時間を費やし、削除ワーカーが `idle` フェーズに滅多にない場合、スキャンワーカーは削除ワーカーが削除を完了するのを待っています。この時点でクラスターのリソースがまだ空いている場合、`tidb_ttl_delete_worker_count` を増やして削除ワーカーの数を増やすことを検討できます。例えば：

  ![scan fast example](/media/ttl/scan-fast.png)

  一方、スキャンワーカーが `dispatch` フェーズに滅多になく、削除ワーカーが長い時間 `idle` フェーズにある場合、スキャンワーカーは比較的忙しい状態です。例えば：

  ![delete fast example](/media/ttl/delete-fast.png)

  TTLジョブにおけるスキャンと削除の割合は、マシンの構成やデータの分布に関連しているため、各時点でのモニタリングデータは実行されているTTLジョブを表しているだけです。特定の時点でどのTTLジョブが実行されているか、およびそのジョブの対応するテーブルを判断するには、テーブル `mysql.tidb_ttl_job_history` を読み取ることができます。

- `tidb_ttl_scan_worker_count` と `tidb_ttl_delete_worker_count` を適切に設定するにはどうすればよいですか？

  1. TTLタスクのボトルネックがスキャンにあるか削除にあるかを判断する方法については、「TTLタスクのボトルネックがスキャンにあるか削除にあるかをどのように判断できますか？」を参照してください。
  2. TiKVノードの数が多い場合、`tidb_ttl_scan_worker_count` の値を増やすと、TTLタスクのワークロードをよりバランスよくすることができます。

  TTLワーカーが多すぎると、多くの圧力がかかるため、TiDBのCPUレベルとTiKVのディスクおよびCPU使用状況を一緒に評価する必要があります。さまざまなシナリオやニーズに応じて（TTLをできるだけ高速化する必要があるか、または他のクエリに対するTTLの影響を減らす必要があるか）、`tidb_ttl_scan_worker_count` と `tidb_ttl_delete_worker_count` の値を調整して、TTLのスキャンと削除の速度を向上させるか、TTLタスクによってもたらされるパフォーマンスへの影響を減らすことができます。
