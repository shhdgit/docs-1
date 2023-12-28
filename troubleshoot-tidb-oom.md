---
title: Troubleshoot TiDB OOM Issues
summary: Learn how to diagnose and resolve TiDB OOM (Out of Memory) issues.
---

# TiDB OOMのトラブルシューティング {#troubleshoot-tidb-oom-issues}

このドキュメントでは、TiDB OOM（メモリ不足）の問題をトラブルシューティングする方法について説明します。現象、原因、解決策、および診断情報について説明します。

## 典型的なOOMの現象 {#typical-oom-phenomena}

以下は、典型的なOOMの現象のいくつかです。

- クライアント側で次のエラーが報告される：`SQL error, errno = 2013, state = 'HY000': Lost connection to MySQL server during query`。

- Grafanaダッシュボードに次のように表示されます：
  - **TiDB** > **Server** > **Memory Usage** において、`process/heapInUse`メトリックが上昇し、しきい値に達した後に突然ゼロになります。
  - **TiDB** > **Server** > **Uptime** が突然ゼロになります。
  - **TiDB-Runtime** > **Memory Usage** において、`estimate-inuse`メトリックが上昇し続けます。

- `tidb.log`をチェックすると、次のログエントリが見つかります：
  - OOMに関するアラーム：`[WARN] [memory_usage_alarm.go:139] ["tidb-server has the risk of OOM because of memory usage exceeds alarm ratio. Running SQLs and heap profile will be recorded in record path"]`。詳細については、[`memory-usage-alarm-ratio`](/system-variables.md#tidb_memory_usage_alarm_ratio)を参照してください。
  - 再起動に関するログエントリ：`[INFO] [printer.go:33] ["Welcome to TiDB."]`。

## 全体的なトラブルシューティングプロセス {#overall-troubleshooting-process}

OOMの問題をトラブルシューティングする場合は、次のプロセスに従ってください。

1. OOMの問題かどうかを確認します。

   次のコマンドを実行して、オペレーティングシステムのログをチェックします。問題が発生した時刻の近くに`oom-killer`ログがある場合、OOMの問題であることを確認できます。

   ```shell
   dmesg -T | grep tidb-server
   ```

   次は、`oom-killer`を含むログの例です。

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

2. OOMの問題であることが確認できたら、デプロイメントまたはデータベースによってOOMが引き起こされたかどうかをさらに調査できます。

   - OOMがデプロイメントの問題によって引き起こされた場合、リソース構成とハイブリッドデプロイメントの影響を調査する必要があります。
   - OOMがデータベースの問題によって引き起こされた場合、次のいずれかの原因が考えられます。
     - TiDBが大量のデータトラフィックを処理している場合、大きなクエリ、大きな書き込み、およびデータのインポートなど。
     - TiDBが高並行性のシナリオで、複数のSQLステートメントが同時にリソースを消費したり、オペレータの並行性が高い場合。
     - TiDBにメモリリークがあり、リソースが解放されない場合。

   具体的なトラブルシューティング方法については、次のセクションを参照してください。

## 典型的な原因と解決策 {#typical-causes-and-solutions}

OOMの問題は、通常次のような原因によって引き起こされます。

- [デプロイメントの問題](#デプロイメントの問題)
- [データベースの問題](#データベースの問題)
- [クライアント側の問題](#クライアント側の問題)

### デプロイメントの問題 {#deployment-issues}

不適切なデプロイメントによってOOMが引き起こされる原因は次のとおりです。

- オペレーティングシステムのメモリ容量が小さすぎる。
- TiUPの構成[`resource_control`](/tiup/tiup-cluster-topology-reference.md#global)が適切でない。
- ハイブリッドデプロイメントの場合（TiDBと他のアプリケーションが同じサーバーにデプロイされていることを意味します）、リソースが不足して`oom-killer`によってTiDBが誤ってキルされることがあります。

### データベースの問題 {#database-issues}

このセクションでは、データベースの問題によって引き起こされるOOMの原因と解決策について説明します。

> **注意：**
>
> [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)を設定している場合、エラーが発生します：`ERROR 1105 (HY000): Out Of Memory Quota![conn_id=54]`。これは、データベースのメモリ使用量の制御動作によるものです。これは正常な動作です。

#### SQLステートメントの実行によるメモリ消費が大きすぎる {#executing-sql-statements-consumes-too-much-memory}

SQLの実行計画が最適でない場合、適切なインデックスがない、古い統計情報がある、またはオプティマイザのバグがあるため、間違ったSQLの実行計画が選択される可能性があります。その結果、巨大な中間結果セットがメモリに蓄積されます。この場合、次の対策を検討してください。

- 適切なインデックスを追加します。

- 実行演算子に[ディスクスピル](/configure-memory-usage.md#disk-spill)機能を使用します。

- テーブル間のJOIN順序を調整します。

- ヒントを使用してSQLステートメントを最適化します。

- 一部の演算子や関数はストレージレベルにプッシュダウンすることができないため、巨大な中間結果セットが蓄積されます。この場合、SQLステートメントを改良するか、ヒントを使用して最適化し、プッシュダウンをサポートする関数や演算子を使用する必要があります。

- 実行計画に演算子HashAggが含まれています。HashAggは複数のスレッドで並行実行されるため、より高速ですが、より多くのメモリを消費します。代わりに、`STREAM_AGG()`を使用できます。

- 同時に読み取るリージョンの数を減らすか、演算子の並行性を減らすことで、高い並行性によって引き起こされるメモリ問題を回避します。対応するシステム変数には次のものがあります。
  - [`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency)
  - [`tidb_index_serial_scan_concurrency`](/system-variables.md#tidb_index_serial_scan_concurrency)
  - [`tidb_executor_concurrency`](/system-variables.md#tidb_executor_concurrency-new-in-v50)

- 問題が発生する時点の近くでセッションの並行性が高すぎる場合、TiDBクラスターをスケールアウトして、より多くのTiDBノードを追加することを検討してください。

#### 大きなトランザクションまたは大きな書き込みによるメモリ消費が大きすぎる {#large-transactions-or-large-writes-consume-too-much-memory}

メモリ容量を計画する必要があります。トランザクションが実行されると、TiDBプロセスのメモリ使用量はトランザクションのサイズと比較してスケールアップし、トランザクションのサイズの2〜3倍以上になります。

単一の大きなトランザクションを複数の小さなトランザクションに分割することができます。

#### 統計情報の収集とロードによるメモリ消費が大きすぎる {#the-process-of-collecting-and-loading-statistical-information-consumes-too-much-memory}

TiDBノードは起動後、統計情報をメモリにロードする必要があります。TiDBは統計情報を収集するときにメモリを消費します。次の方法でメモリ使用量を制御できます。

- サンプリング率を指定し、特定の列のみの統計情報を収集し、`ANALYZE`の並行性を減らします。
- TiDB v6.1.0以降、システム変数[`tidb_stats_cache_mem_quota`](/system-variables.md#tidb_stats_cache_mem_quota-new-in-v610)を使用して、統計情報のメモリ使用量を制御できます。
- TiDB v6.1.0以降、システム変数[`tidb_mem_quota_analyze`](/system-variables.md#tidb_mem_quota_analyze-new-in-v610)を使用して、TiDBが統計情報を更新する際の最大メモリ使用量を制御できます。

詳細については、[統計情報の概要](/statistics.md)を参照してください。

#### プリペアドステートメントが過剰に使用されています {#prepared-statements-are-overused}

クライアント側はプリペアドステートメントを作成し続けますが、[`deallocate prepare stmt`](/sql-prepared-plan-cache.md#ignore-the-com_stmt_close-command-and-the-deallocate-prepare-statement)を実行しません。そのため、メモリ消費が継続的に増加し、最終的にTiDB OOMが発生します。その理由は、プリペアドステートメントが占有するメモリがセッションが閉じられるまで解放されないためです。これは、長時間の接続セッションにとって特に重要です。

問題を解決するには、次の対策を検討してください。

- セッションのライフサイクルを調整します。
- 接続プールの[`wait_timeout`と`max_execution_time`](/develop/dev-guide-connection-parameters.md#timeout-related-parameters)を調整します。
- システム変数[`max_prepared_stmt_count`](/system-variables.md#max_prepared_stmt_count)を使用して、セッション内のプリペアドステートメントの最大数を制御します。

#### `tidb_enable_rate_limit_action`が適切に設定されていません {#tidb-enable-rate-limit-action-is-not-configured-properly}

システム変数[`tidb_enable_rate_limit_action`](/system-variables.md#tidb_enable_rate_limit_action)は、SQLステートメントがデータのみを読み取る場合にメモリ使用量を効果的に制御します。この変数が有効になっていて、計算操作（JOINや集計操作など）が必要な場合、メモリ使用量は[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)の制御下にない可能性があり、OOMのリスクが高まります。

このシステム変数を無効にすることをお勧めします。TiDB v6.3.0以降、このシステム変数はデフォルトで無効になっています。

### クライアント側の問題 {#client-side-issues}

クライアント側でOOMが発生した場合は、次のことを調査してください。

- **Grafana TiDB Details** > **Server** > **Client Data Traffic**でトレンドと速度を確認し、ネットワークのブロックがあるかどうかを確認します。
- アプリケーションのOOMが、間違ったJDBC設定パラメータによって引き起こされているかどうかを確認します。たとえば、ストリーミングリード用の`defaultFetchSize`パラメータが誤って設定されていると、クライアント側にデータが大量に蓄積されることがあります。

## OOM問題のトラブルシューティングに収集する診断情報 {#diagnostic-information-to-be-collected-to-troubleshoot-oom-issues}

OOM問題の原因を特定するには、次の情報を収集する必要があります。

- オペレーティングシステムのメモリ関連の設定を収集します。
  - TiUPの設定：`resource_control.memory_limit`
  - オペレーティングシステムの設定：
    - メモリ情報：`cat /proc/meminfo`
    - カーネルパラメータ：`vm.overcommit_memory`
  - NUMA情報：
    - `numactl --hardware`
    - `numactl --show`

- データベースのバージョン情報とメモリ関連の設定を収集します。
  - TiDBのバージョン
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

- メモリを消費するSQLステートメントを確認します。

  - TiDBダッシュボードでSQLステートメントの分析、遅いクエリ、およびメモリ使用状況を表示します。
  - `INFORMATION_SCHEMA`の`SLOW_QUERY`と`CLUSTER_SLOW_QUERY`を確認します。
  - 各TiDBノードの`tidb_slow_query.log`を確認します。
  - `grep "expensive_query" tidb.log`を実行して、対応するログエントリを確認します。
  - `EXPLAIN ANALYZE`を実行して、オペレータのメモリ使用状況を確認します。
  - `SELECT * FROM information_schema.processlist;`を実行して、`MEM`列の値を確認します。

- メモリ使用量が高い場合にTiDBプロファイル情報を収集するには、次のコマンドを実行します。

  ```shell
  curl -G http://{TiDBIP}:10080/debug/zip?seconds=10" > profile.zip
  ```

- `grep "tidb-server has the risk of OOM" tidb.log`を実行して、TiDB Serverが収集したアラートファイルのパスを確認します。次は、例の出力です。

  ```shell
  ["tidb-server has the risk of OOM because of memory usage exceeds alarm ratio. Running SQLs and heap profile will be recorded in record path"] ["is tidb_server_memory_limit set"=false] ["system memory total"=14388137984] ["system memory usage"=11897434112] ["tidb-server memory usage"=11223572312] [memory-usage-alarm-ratio=0.8] ["record path"="/tmp/0_tidb/MC4wLjAuMDo0MDAwLzAuMC4wLjA6MTAwODA=/tmp-storage/record"]
  ```

## 関連情報 {#see-also}

- [TiDBメモリ制御](/configure-memory-usage.md)
- [TiKVメモリパラメータのパフォーマンスを調整する](/tune-tikv-memory-performance.md)
