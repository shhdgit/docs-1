---
title: Periodically Delete Data Using TTL (Time to Live)
summary: Time to live (TTL) is a feature that allows you to manage TiDB data lifetime at the row level. In this document, you can learn how to use TTL to automatically expire and delete old data.
---

# 定期的にTTL（Time to Live）を使用して期限切れのデータを削除する {#periodically-delete-expired-data-using-ttl-time-to-live}

Time to live（TTL）は、TiDBデータの寿命を行レベルで管理する機能です。TTL属性を持つテーブルでは、TiDBは自動的にデータの寿命をチェックし、行レベルで期限切れのデータを削除します。この機能は、いくつかのシナリオで効果的にストレージスペースを節約し、パフォーマンスを向上させることができます。

以下は、TTLの一般的な使用シナリオです。

- 定期的に検証コードや短縮URLを削除します。
- 不要な過去の注文を定期的に削除します。
- 計算の中間結果を自動的に削除します。

TTLは、ユーザーがオンラインの読み取りおよび書き込みワークロードに影響を与えることなく、定期的かつ適時に不要なデータをクリーンアップするのに役立ちます。TTLは、テーブルの単位でデータを並列に削除するため、異なるTiDBノードに異なるジョブを同時にディスパッチします。TTLは、すべての期限切れのデータがすぐに削除されることを保証しません。つまり、データが期限切れになっても、バックグラウンドのTTLジョブによってそのデータが削除されるまで、クライアントはそのデータを一定時間読み取る可能性があります。

## 構文 {#syntax}

TTL属性を[`CREATE TABLE`](/sql-statements/sql-statement-create-table.md)または[`ALTER TABLE`](/sql-statements/sql-statement-alter-table.md)ステートメントを使用してテーブルに構成できます。

### TTL属性を持つテーブルを作成する {#create-a-table-with-a-ttl-attribute}

- TTL属性を持つテーブルを作成します。

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP
  ) TTL = `created_at` + INTERVAL 3 MONTH;
  ```

  上記の例では、テーブル`t1`を作成し、データの作成時間を示すTTLタイムスタンプ列として`created_at`を指定しています。また、`INTERVAL 3 MONTH`を使用して、テーブル内の行が許可される最長の生存期間を3ヶ月に設定しています。この値を超えるデータは後で削除されます。

- `TTL_ENABLE`属性を設定して、期限切れのデータのクリーンアップ機能を有効または無効にします。

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP
  ) TTL = `created_at` + INTERVAL 3 MONTH TTL_ENABLE = 'OFF';
  ```

  `TTL_ENABLE`が`OFF`に設定されている場合、他のTTLオプションが設定されていても、TiDBはこのテーブルの期限切れのデータを自動的にクリーンアップしません。TTL属性を持つテーブルでは、`TTL_ENABLE`はデフォルトで`ON`になります。

- MySQLと互換性を持たせるために、コメントを使用してTTL属性を設定できます。

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP
  ) /*T![ttl] TTL = `created_at` + INTERVAL 3 MONTH TTL_ENABLE = 'OFF'*/;
  ```

  TiDBでは、テーブルのTTL属性を使用するか、コメントを使用してTTLを構成するかは同等です。MySQLでは、コメントは無視され、通常のテーブルが作成されます。

### テーブルのTTL属性を変更する {#modify-the-ttl-attribute-of-a-table}

- テーブルのTTL属性を変更します。

  ```sql
  ALTER TABLE t1 TTL = `created_at` + INTERVAL 1 MONTH;
  ```

  上記のステートメントを使用して、既存のTTL属性を持つテーブルを変更するか、TTL属性のないテーブルにTTL属性を追加することができます。

- TTL属性を持つテーブルの`TTL_ENABLE`の値を変更します。

  ```sql
  ALTER TABLE t1 TTL_ENABLE = 'OFF';
  ```

- テーブルのすべてのTTL属性を削除するには：

  ```sql
  ALTER TABLE t1 REMOVE TTL;
  ```

### TTLとデータ型のデフォルト値 {#ttl-and-the-default-values-of-data-types}

TTLを[データ型のデフォルト値](/data-type-default-values.md)と一緒に使用できます。以下は、2つの一般的な使用例です。

- `DEFAULT CURRENT_TIMESTAMP`を使用して、列のデフォルト値を現在の作成時間として指定し、この列をTTLタイムスタンプ列として使用します。3ヶ月前に作成されたレコードは期限切れです。

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  ) TTL = `created_at` + INTERVAL 3 MONTH;
  ```

- 列のデフォルト値を作成時間または最新の更新時間として指定し、この列をTTLタイムスタンプ列として使用します。3ヶ月間更新されていないレコードは期限切れです。

  ```sql
  CREATE TABLE t1 (
      id int PRIMARY KEY,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  ) TTL = `created_at` + INTERVAL 3 MONTH;
  ```

### TTLと生成された列 {#ttl-and-generated-columns}

TTLを[生成された列](/generated-columns.md)と一緒に使用して、複雑な有効期限切れルールを構成できます。例：

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

前述の文は、`expire_at` 列を TTL タイムスタンプ列として使用し、メッセージの種類に応じて有効期限を設定します。メッセージが画像の場合、5日で期限切れになります。それ以外の場合は、30日で期限切れになります。

[JSON タイプ](/data-type-json.md) と TTL を一緒に使用することができます。例えば：

```sql
CREATE TABLE orders (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    order_info JSON,
    created_at DATE AS (JSON_EXTRACT(order_info, '$.created_at')) VIRTUAL
) TTL = `created_at` + INTERVAL 3 month;
```

## TTL job {#ttl-job}

TTL属性を持つ各テーブルについて、TiDBは内部で期限切れのデータをクリーンアップするバックグラウンドジョブをスケジュールします。テーブルの`TTL_JOB_INTERVAL`属性を設定することで、これらのジョブの実行期間をカスタマイズできます。次の例では、テーブル`orders`のバックグラウンドクリーンアップジョブを24時間ごとに実行するように設定しています。

```sql
ALTER TABLE orders TTL_JOB_INTERVAL = '24h';
```

`TTL_JOB_INTERVAL` はデフォルトで `1h` に設定されています。

TTL ジョブを実行すると、TiDB はリージョンを最小単位として、テーブルを最大 64 タスクに分割します。これらのタスクは分散して実行されます。システム変数 [`tidb_ttl_running_tasks`](/system-variables.md#tidb_ttl_running_tasks-new-in-v700) を設定することで、クラスタ全体で同時に実行される TTL タスクの数を制限することができます。ただし、すべての種類のテーブルの TTL ジョブをタスクに分割することはできません。どの種類のテーブルの TTL ジョブをタスクに分割できないかの詳細については、[Limitations](#limitations) セクションを参照してください。

TTL ジョブの実行を無効にするには、`TTL_ENABLE='OFF'` テーブルオプションを設定するだけでなく、[`tidb_ttl_job_enable`](/system-variables.md#tidb_ttl_job_enable-new-in-v650) グローバル変数を設定することで、クラスタ全体で TTL ジョブの実行を無効にすることもできます。

```sql
SET @@global.tidb_ttl_job_enable = OFF;
```

いくつかのシナリオでは、TTLジョブが特定の時間枠でのみ実行されるようにしたい場合があります。この場合、[`tidb_ttl_job_schedule_window_start_time`](/system-variables.md#tidb_ttl_job_schedule_window_start_time-new-in-v650)と[`tidb_ttl_job_schedule_window_end_time`](/system-variables.md#tidb_ttl_job_schedule_window_end_time-new-in-v650)のグローバル変数を設定して、時間枠を指定できます。例えば：

```sql
SET @@global.tidb_ttl_job_schedule_window_start_time = '01:00 +0000';
SET @@global.tidb_ttl_job_schedule_window_end_time = '05:00 +0000';
```

前述のステートメントにより、TTLジョブはデフォルトで`00:00 +0000`から`23:59 +0000`に設定され、いつでもジョブをスケジュールできます。

## 観測可能性 {#observability}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> このセクションはTiDB Self-Hostedにのみ適用されます。現在、TiDB CloudはTTLメトリクスを提供していません。

</CustomContent>

TiDBは定期的にTTLに関するランタイム情報を収集し、これらのメトリクスの視覚化されたチャートをGrafanaで提供します。GrafanaのTiDB -> TTLパネルでこれらのメトリクスを確認できます。

<CustomContent platform="tidb">

メトリクスの詳細については、[TiDB Monitoring Metrics](/grafana-tidb-dashboard.md)のTTLセクションを参照してください。

</CustomContent>

さらに、TTLジョブに関する詳細情報を取得するために、TiDBは3つのテーブルを提供しています。

- `mysql.tidb_ttl_table_status`テーブルには、以前に実行されたTTLジョブとすべてのTTLテーブルの実行中のTTLジョブに関する情報が含まれています

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

  `table_id`列はパーティションテーブルのIDであり、`parent_table_id`は`infomation_schema.tables`のIDと対応しています。テーブルがパーティションテーブルでない場合、2つのIDは同じです。

  列`{last, current}_job_{start_time, finish_time, ttl_expire}`は、それぞれ前回または現在の実行のTTLジョブの開始時間、終了時間、および有効期限を示しています。`last_job_summary`列には、前回のTTLタスクの実行状況が含まれており、行の総数、成功した行の数、失敗した行の数などが記載されています。

- `mysql.tidb_ttl_task`テーブルには、実行中のTTLサブタスクに関する情報が含まれています。TTLジョブは多くのサブタスクに分割され、このテーブルは現在実行中のサブタスクを記録します。

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

  `table_id`列はパーティションテーブルのIDであり、`parent_table_id`は`infomation_schema.tables`のIDと対応しています。`table_schema`、`table_name`、`partition_name`はそれぞれデータベース、テーブル名、およびパーティション名に対応します。`create_time`、`finish_time`、`ttl_expire`は、TTLタスクの作成時間、終了時間、および有効期限を示します。`expired_rows`および`deleted_rows`は、期限切れの行数と正常に削除された行数を示します。

## TiDBツールとの互換性 {#compatibility-with-tidb-tools}

TTLは他のTiDB移行、バックアップ、およびリカバリツールと使用できます。

| ツール名            | 最小サポートバージョン | 説明                                                                                                                                                  |
| --------------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| バックアップ＆リストア（BR） | v6.6.0      | BRを使用してデータをリストアした後、テーブルの`TTL_ENABLE`属性は`OFF`に設定されます。これにより、バックアップとリストア後に期限切れのデータがすぐに削除されるのを防ぎます。各テーブルで`TTL_ENABLE`属性を手動でオンにする必要があります。                |
| TiDB Lightning  | v6.6.0      | TiDB Lightningを使用してデータをインポートした後、インポートされたテーブルの`TTL_ENABLE`属性は`OFF`に設定されます。これにより、インポート後に期限切れのデータがすぐに削除されるのを防ぎます。各テーブルで`TTL_ENABLE`属性を手動でオンにする必要があります。 |
| TiCDC           | v7.0.0      | 下流の`TTL_ENABLE`属性は自動的に`OFF`に設定されます。上流のTTL削除は下流に同期されます。したがって、重複した削除を防ぐため、下流テーブルの`TTL_ENABLE`属性は強制的に`OFF`に設定されます。                                     |

## SQLとの互換性 {#compatibility-with-sql}

| 機能名                                                                         | 説明                                                                                                                                                      |
| :-------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`FLASHBACK TABLE`](/sql-statements/sql-statement-flashback-table.md)       | `FLASHBACK TABLE`はテーブルの`TTL_ENABLE`属性を`OFF`に設定します。これにより、フラッシュバック後に期限切れのデータがすぐに削除されるのを防ぎます。各テーブルで`TTL_ENABLE`属性を手動でオンにする必要があります。                         |
| [`FLASHBACK DATABASE`](/sql-statements/sql-statement-flashback-database.md) | `FLASHBACK DATABASE`はテーブルの`TTL_ENABLE`属性を`OFF`に設定し、`TTL_ENABLE`属性は変更されません。これにより、フラッシュバック後に期限切れのデータがすぐに削除されるのを防ぎます。各テーブルで`TTL_ENABLE`属性を手動でオンにする必要があります。 |
| [`FLASHBACK CLUSTER`](/sql-statements/sql-statement-flashback-cluster.md)   | `FLASHBACK CLUSTER`はシステム変数[`TIDB_TTL_JOB_ENABLE`](/system-variables.md#tidb_ttl_job_enable-new-in-v650)を`OFF`に設定し、`TTL_ENABLE`属性の値は変更されません。             |

## 制限事項 {#limitations}

現在、TTL機能には以下の制限事項があります。

- TTL属性は、ローカル一時テーブルおよびグローバル一時テーブルを含む一時テーブルに設定できません。
- TTL属性を持つテーブルは、他のテーブルによって外部キー制約の主テーブルとして参照されることはサポートされていません。
- 期限切れのデータがすぐに削除されることは保証されません。期限切れのデータが削除されるタイミングは、バックグラウンドのクリーンアップジョブのスケジューリング間隔とスケジューリングウィンドウに依存します。
- [クラスタ化インデックス](/clustered-indexes.md)を使用するテーブルの場合、主キーが整数型またはバイナリ文字列型でない場合、TTLジョブを複数のタスクに分割できません。これにより、TTLジョブが単一のTiDBノードで順次実行されることになります。テーブルに大量のデータが含まれる場合、TTLジョブの実行が遅くなる可能性があります。
- [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)ではTTLは利用できません。

## FAQs {#faqs}

<CustomContent platform="tidb">

- How can I determine whether the deletion is fast enough to keep the data size relatively stable?

  [Grafana `TiDB` ダッシュボード](/grafana-tidb-dashboard.md) では、`TTL Insert Rows Per Hour` パネルが前時間に挿入された合計行数を記録します。対応する `TTL Delete Rows Per Hour` は前時間に TTL タスクによって削除された合計行数を記録します。長い時間 `TTL Insert Rows Per Hour` が `TTL Delete Rows Per Hour` よりも高い場合、挿入の速度が削除の速度よりも高く、データの総量が増加することを意味します。例：

  ![insert fast example](/media/ttl/insert-fast.png)

  TTL は期限切れの行が即座に削除されることを保証しないため、現在挿入されている行が将来の TTL タスクで削除されることを保証しないため、短期間における TTL 削除の速度が挿入の速度よりも低い場合でも、TTL の速度が遅すぎるとは限りません。状況を考慮する必要があります。

- How can I determine whether the bottleneck of a TTL task is in scanning or deleting?

  `TTL Scan Worker Time By Phase` と `TTL Delete Worker Time By Phase` パネルを見てください。スキャンワーカーが大部分の時間を `dispatch` フェーズで待機しており、削除ワーカーが滅多に `idle` フェーズにならない場合、スキャンワーカーは削除ワーカーが削除を完了するのを待っています。この時点でクラスターリソースがまだ空いている場合、`tidb_ttl_delete_worker_count` を増やして削除ワーカーの数を増やすことを検討できます。例：

  ![scan fast example](/media/ttl/scan-fast.png)

  一方、スキャンワーカーが滅多に `dispatch` フェーズにならず、削除ワーカーが長い時間 `idle` フェーズにある場合、スキャンワーカーは比較的忙しい状態です。例：

  ![delete fast example](/media/ttl/delete-fast.png)

  TTL ジョブのスキャンと削除の割合は、マシン構成とデータ分布に関連しており、各瞬間の監視データは実行されている TTL ジョブを代表するものです。特定の瞬間に実行されている TTL ジョブとそのジョブの対応するテーブルを判断するために、`mysql.tidb_ttl_job_history` テーブルを読むことができます。

- How to configure `tidb_ttl_scan_worker_count` and `tidb_ttl_delete_worker_count` properly?

  1. TTL タスクのボトルネックがスキャンか削除かを判断する方法についての質問を参照して、`tidb_ttl_scan_worker_count` または `tidb_ttl_delete_worker_count` の値を増やすかどうかを検討してください。
  2. TiKV ノードの数が多い場合、`tidb_ttl_scan_worker_count` の値を増やすことで TTL タスクのワークロードをより均衡させることができます。

  TTL ワーカーが多すぎると多くの圧力をかけるため、TiDB の CPU レベルと TiKV のディスクおよび CPU 使用状況を総合的に評価する必要があります。異なるシナリオとニーズに応じて（TTL を可能な限り高速化する必要があるか、他のクエリに対する TTL の影響を減らす必要があるか）、`tidb_ttl_scan_worker_count` と `tidb_ttl_delete_worker_count` の値を調整して、TTL のスキャンと削除の速度を向上させるか、TTL タスクによってもたらされるパフォーマンスへの影響を減らすことができます。

</CustomContent>
<CustomContent platform="tidb-cloud">

- How to configure `tidb_ttl_scan_worker_count` and `tidb_ttl_delete_worker_count` properly?

  TiKV ノードの数が多い場合、`tidb_ttl_scan_worker_count` の値を増やすことで TTL タスクのワークロードをより均衡させることができます。

  しかし、TTL ワーカーが多すぎると多くの圧力をかけるため、TiDB の CPU レベルと TiKV のディスクおよび CPU 使用状況を総合的に評価する必要があります。異なるシナリオとニーズに応じて（TTL を可能な限り高速化する必要があるか、他のクエリに対する TTL の影響を減らす必要があるか）、`tidb_ttl_scan_worker_count` と `tidb_ttl_delete_worker_count` の値を調整して、TTL のスキャンと削除の速度を向上させるか、TTL タスクによってもたらされるパフォーマンスへの影響を減らすことができます。

</CustomContent>
