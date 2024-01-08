---
title: Troubleshoot TiDB OOM Issues
summary: Learn how to diagnose and resolve TiDB OOM (Out of Memory) issues.
---

# TiDB OOMのトラブルシューティング {#troubleshoot-tidb-oom-issues}

このドキュメントでは、TiDB OOM（メモリ不足）の問題をトラブルシューティングする方法について、現象、原因、解決策、および診断情報について説明します。

## 典型的なOOM現象 {#typical-oom-phenomena}

以下は、いくつかの典型的なOOM現象です。

- クライアント側が次のエラーを報告します：`SQL error, errno = 2013, state = 'HY000': Lost connection to MySQL server during query`。

- Grafanaダッシュボードには次の情報が表示されます：
  - **TiDB** > **サーバー** > **メモリ使用量** では、`process/heapInUse`メトリックが増加し続け、しきい値に達した後に突然ゼロになることがあります。
  - **TiDB** > **サーバー** > **稼働時間** が突然ゼロになります。
  - **TiDB-Runtime** > **メモリ使用量** では、`estimate-inuse`メトリックが増加し続けます。

- `tidb.log`をチェックすると、次のログエントリが見つかります：
  - OOMに関する警告：`[WARN] [memory_usage_alarm.go:139] ["tidb-server has the risk of OOM because of memory usage exceeds alarm ratio. Running SQLs and heap profile will be recorded in record path"]`。詳細については、[`memory-usage-alarm-ratio`](/system-variables.md#tidb_memory_usage_alarm_ratio)を参照してください。
  - 再起動に関するログエントリ：`[INFO] [printer.go:33] ["Welcome to TiDB."]`。

## 全体的なトラブルシューティングプロセス {#overall-troubleshooting-process}

OOMの問題をトラブルシューティングする際は、次のプロセスに従います。

1. OOMの問題かどうかを確認します。

   次のコマンドを実行して、オペレーティングシステムのログを確認します。問題が発生したタイミングで`oom-killer`のログがあれば、OOMの問題であることを確認できます。

   ```shell
   dmesg -T | grep tidb-server
   ```

   次は、`oom-killer`を含むログの例です：

   ```shell
   ......
   Mar 14 16:55:03 localhost kernel: tidb-server invoked oom-killer: gfp_mask=0x201da, order=0, oom_score_adj=0
   Mar 14 16:55:03 localhost kernel: tidb-server cpuset=/ mems_allowed=0
   Mar 14 16:55:03 localhost kernel: CPU: 14 PID: 21966 Comm: tidb-server Kdump: loaded Not tainted 3.10.0-1160.el7.x86_64 #1
   Mar 14 16:55:03 localhost kernel: Hardware name: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.14.0-0-g155821a1990b-prebuilt.qemu.org 04/01/2014
   ......
   Mar 14 16:55:03 localhost kernel: Out of memory: Kill process 21945 (tidb-server) score 956 or sacrifice child
   Mar 14 16:55:03 localhost kernel: Killed process 21945 (tidb-server), UID 1000, total-vm:33027492kB, anon-rss:31303276kB, file-rss:0kB, shmem-rss:0kB
   Mar 14 16:55:07 localhost systemd: tidb-4000.service: main process exited, code=killed, status=9/KILL
   ......

   ```
2. OOMの問題であることを確認した後、デプロイまたはデータベースによってOOMが引き起こされたかどうかをさらに調査できます。

   - OOMがデプロイの問題によるものであれば、リソース構成とハイブリッドデプロイメントの影響を調査する必要があります。
   - OOMがデータベースの問題によるものであれば、次のような可能性があります：
     - TiDBが大規模なデータトラフィックを処理している場合、大規模なクエリ、大規模な書き込み、およびデータのインポートがあります。
     - TiDBが高並行シナリオであり、複数のSQLステートメントが同時にリソースを消費するか、オペレータの並行性が高い場合があります。
     - TiDBにメモリリークがあり、リソースが解放されていない場合があります。

   具体的なトラブルシューティング方法については、次のセクションを参照してください。

## 典型的な原因と解決策 {#typical-causes-and-solutions}

OOMの問題は通常、次のような理由で引き起こされます：

- [デプロイの問題](#deployment-issues)
- [データベースの問題](#database-issues)
- [クライアント側の問題](#client-side-issues)

### デプロイの問題 {#deployment-issues}

適切でないデプロイによるOOMの原因として次のものがあります：

- オペレーティングシステムのメモリ容量が小さすぎる。
- TiUP構成の[`resource_control`](/tiup/tiup-cluster-topology-reference.md#global)が適切でない。
- ハイブリッドデプロイメントの場合（つまり、TiDBと他のアプリケーションが同じサーバーにデプロイされている場合）、リソースが不足して`oom-killer`によってTiDBが誤って終了されることがあります。

### データベースの問題 {#database-issues}

このセクションでは、データベースの問題によるOOMの原因と解決策について説明します。

> **Note:**
>
> [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)を構成している場合、次のエラーが発生します：`ERROR 1105 (HY000): Out Of Memory Quota![conn_id=54]`。これはデータベースのメモリ使用量制御の挙動によるものです。これは正常な挙動です。

#### SQLステートメントの実行によるメモリ消費が過剰 {#executing-sql-statements-consumes-too-much-memory}

OOMの問題の異なる原因に応じて、SQLステートメントのメモリ使用量を減らすために次の対策を取ることができます。

- SQLの実行計画が最適でない場合、たとえば適切なインデックスがない、統計情報が古い、またはオプティマイザのバグがある場合、誤ったSQLの実行計画が選択される可能性があります。その結果、膨大な中間結果セットがメモリに蓄積されます。この場合、次の対策を検討してください：
  - 適切なインデックスを追加します。
  - 実行演算子のために[ディスクスピル](/configure-memory-usage.md#disk-spill)機能を使用します。
  - テーブル間のJOIN順序を調整します。
  - ヒントを使用してSQLステートメントを最適化します。

- 一部の演算子や関数がストレージレベルにプッシュダウンされないため、膨大な中間結果セットが蓄積されます。この場合、SQLステートメントを改善するか、ヒントを使用して最適化し、プッシュダウンをサポートする関数や演算子を使用する必要があります。

- 実行計画に`HashAgg`演算子が含まれています。HashAggは複数のスレッドで並行して実行され、より高速ですが、より多くのメモリを消費します。代わりに`STREAM_AGG()`を使用できます。

- 同時に読み取るリージョンの数を減らしたり、演算子の並行性を減らしてメモリ問題を回避するために、次のシステム変数を調整します：
  - [`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency)
  - [`tidb_index_serial_scan_concurrency`](/system-variables.md#tidb_index_serial_scan_concurrency)
  - [`tidb_executor_concurrency`](/system-variables.md#tidb_executor_concurrency-new-in-v50)

- 問題が発生する時間点近くでセッションの並行性が高すぎる場合、TiDBクラスターをスケールアウトして、より多くのTiDBノードを追加することを検討してください。

#### 大規模なトランザクションまたは大規模な書き込みによるメモリ消費が過剰 {#large-transactions-or-large-writes-consume-too-much-memory}

メモリ容量を計画する必要があります。トランザクションが実行されると、TiDBプロセスのメモリ使用量は、トランザクションサイズと比較して2倍から3倍以上にスケールアップします。

単一の大規模なトランザクションを複数の小さなトランザクションに分割することができます。

#### 統計情報の収集およびロードプロセスによるメモリ消費が過剰 {#the-process-of-collecting-and-loading-statistical-information-consumes-too-much-memory}

TiDBノードは起動後に統計情報をメモリにロードする必要があります。統計情報を収集する際にTiDBはメモリを消費します。次の方法でメモリ使用量を制御できます：

- サンプリング率を指定し、特定の列の統計情報のみを収集し、`ANALYZE`の並行性を減らします。
- TiDB v6.1.0以降、システム変数[`tidb_stats_cache_mem_quota`](/system-variables.md#tidb_stats_cache_mem_quota-new-in-v610)を使用して統計情報のメモリ使用量を制御できます。
- TiDB v6.1.0以降、システム変数[`tidb_mem_quota_analyze`](/system-variables.md#tidb_mem_quota_analyze-new-in-v610)を使用して、TiDBが統計情報を更新する際の最大メモリ使用量を制御できます。

詳細については、[統計情報の概要](/statistics.md)を参照してください。

#### プリペアドステートメントの過剰な使用 {#prepared-statements-are-overused}

クライアント側が継続的にプリペアドステートメントを作成し、`deallocate prepare stmt`を実行しないため、メモリ消費が継続的に増加し、最終的にTiDB OOMが発生します。プリペアドステートメントが占有するメモリは、セッションが閉じられるまで解放されません。これは特に長時間の接続セッションにとって重要です。

問題を解決するために、次の対策を検討してください：

- セッションライフサイクルを調整します。
- 接続プールの[`wait_timeout`および`max_execution_time`](/develop/dev-guide-connection-parameters.md#timeout-related-parameters)を調整します。
- システム変数[`max_prepared_stmt_count`](/system-variables.md#max_prepared_stmt_count)を使用して、セッション内のプリペアドステートメントの最大数を制御します。

#### `tidb_enable_rate_limit_action` is not configured properly {#tidb-enable-rate-limit-action-is-not-configured-properly}

システム変数[`tidb_enable_rate_limit_action`](/system-variables.md#tidb_enable_rate_limit_action)は、SQLステートメントがデータのみを読み取る場合にメモリ使用量を効果的に制御します。この変数が有効になっており、計算操作（結合や集計操作など）が必要な場合、メモリ使用量は[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)の制御下にない可能性があり、OOMのリスクが高まります。

このシステム変数を無効にすることをお勧めします。TiDB v6.3.0以降、このシステム変数はデフォルトで無効になっています。

### クライアント側の問題 {#client-side-issues}

クライアント側でOOMが発生した場合、次のことを調査してください。

- **Grafana TiDB Details** > **Server** > **Client Data Traffic**でトレンドと速度を確認し、ネットワークのブロックがあるかどうかを確認します。
- JDBC構成パラメータの誤った設定によるアプリケーションのOOMがあるかどうかを確認します。たとえば、ストリーミング読み取り用の`defaultFetchSize`パラメータが誤って構成されていると、クライアント側でデータが大量に蓄積される可能性があります。

## OOMの問題をトラブルシューティングするために収集する診断情報 {#diagnostic-information-to-be-collected-to-troubleshoot-oom-issues}

OOMの問題の原因を特定するには、次の情報を収集する必要があります。

- オペレーティングシステムのメモリ関連の構成を収集します：
  - TiUP構成：`resource_control.memory_limit`
  - オペレーティングシステムの構成：
    - メモリ情報：`cat /proc/meminfo`
    - カーネルパラメータ：`vm.overcommit_memory`
  - NUMA情報：
    - `numactl --hardware`
    - `numactl --show`

- データベースのバージョン情報とメモリ関連の構成を収集します：
  - TiDBバージョン
  - `tidb_mem_quota_query`
  - `memory-usage-alarm-ratio`
  - `mem-quota-query`
  - `oom-action`
  - `tidb_enable_rate_limit_action`
  - `tidb_server_memory_limit`
  - `oom-use-tmp-storage`
  - `tmp-storage-path`
  - `tmp-storage-quota`
  - `tidb_analyze_version`

- GrafanaダッシュボードのTiDBメモリの日次使用状況を確認します：**TiDB** > **Server** > **Memory Usage**。

- より多くのメモリを消費するSQLステートメントを確認します。

  - TiDBダッシュボードでSQLステートメントの分析、遅いクエリ、およびメモリ使用状況を確認します。
  - `INFORMATION_SCHEMA`の`SLOW_QUERY`および`CLUSTER_SLOW_QUERY`を確認します。
  - 各TiDBノードの`tidb_slow_query.log`を確認します。
  - `grep "expensive_query" tidb.log`を実行して、対応するログエントリを確認します。
  - `EXPLAIN ANALYZE`を実行して、演算子のメモリ使用量を確認します。
  - `SELECT * FROM information_schema.processlist;`を実行して、`MEM`列の値を確認します。

- メモリ使用量が高い場合にTiDBプロファイル情報を収集するには、次のコマンドを実行します：

  ```shell
  curl -G http://{TiDBIP}:10080/debug/zip?seconds=10" > profile.zip
  ```

- `tidb-server has the risk of OOM`を検索して、TiDB Serverが収集したアラートファイルのパスを確認します。以下は例です：

  ```shell
  ["tidb-server has the risk of OOM because of memory usage exceeds alarm ratio. Running SQLs and heap profile will be recorded in record path"] ["is tidb_server_memory_limit set"=false] ["system memory total"=14388137984] ["system memory usage"=11897434112] ["tidb-server memory usage"=11223572312] [memory-usage-alarm-ratio=0.8] ["record path"="/tmp/0_tidb/MC4wLjAuMDo0MDAwLzAuMC4wLjA6MTAwODA=/tmp-storage/record"]
  ```

## 関連情報 {#see-also}

- [TiDB Memory Control](/configure-memory-usage.md)
- [Tune TiKV Memory Parameter Performance](/tune-tikv-memory-performance.md)
