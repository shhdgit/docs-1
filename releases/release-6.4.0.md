---
title: TiDB 6.4.0 Release Notes
---

# TiDB 6.4.0 リリースノート {#tidb-6-4-0-release-notes}

リリース日: 2022年11月17日

TiDB バージョン: 6.4.0-DMR

> **Note:**
>
> TiDB 6.4.0-DMR のドキュメントは[アーカイブ](https://docs-archive.pingcap.com/tidb/v6.4/)されました。PingCAP では、TiDB データベースの[最新のLTSバージョン](https://docs.pingcap.com/tidb/stable)を使用することをお勧めします。

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.4/quick-start-with-tidb) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.4.0#version-list)

v6.4.0-DMR では、主な新機能と改善点は以下の通りです:

- [`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-cluster.md)を使用して、クラスタを特定の時点に復元するサポート (実験的)。
- TiDB インスタンスの[グローバルメモリ使用量](/configure-memory-usage.md)を追跡するサポート (実験的)。
- [リニアハッシュパーティショニング構文](/partitioned-table.md#how-tidb-handles-linear-hash-partitions)と互換性がある。
- 高性能でグローバルに単調増加する[`AUTO_INCREMENT`](/auto-increment.md#mysql-compatibility-mode)をサポート (実験的)。
- [JSONタイプ](/data-type-json.md)の配列データの範囲選択をサポート。
- ディスク障害やI/Oのスタックなどの極端な状況での障害復旧を加速。
- テーブルの結合順序を決定するための[ダイナミックプランニングアルゴリズム](/join-reorder.md#example-the-dynamic-programming-algorithm-of-join-reorder)を追加。
- 相関サブクエリのデコレーションを実行するかどうかを制御するための新しいオプティマイザヒント`NO_DECORRELATE`を導入。
- [クラスタ診断](/dashboard/dashboard-diagnostics-access.md)機能がGAになりました。
- TiFlash が[安定性のための暗号化](/encryption-at-rest.md#tiflash)のためにSM4アルゴリズムをサポート。
- SQLステートメントを使用して、指定されたテーブルのTiFlashレプリカを即座に[コンパクト化](/sql-statements/sql-statement-alter-table-compact.md#compact-tiflash-replicas-of-specified-partitions-in-a-table)するサポート。
- EBSボリュームスナップショットを使用して、TiDBクラスタを[バックアップ](https://docs.pingcap.com/tidb-in-kubernetes/v1.4/backup-to-aws-s3-by-snapshot)するサポート。
- DM が[上流データソース情報を下流マージテーブルの拡張列に書き込む](/dm/dm-table-routing.md#extract-table-schema-and-source-information-and-write-into-the-merged-table)サポート。

## 新機能 {#new-features}

### SQL {#sql}

- SQLステートメントを使用して、指定されたテーブルのTiFlashレプリカを即座にコンパクト化するサポート [#5315](https://github.com/pingcap/tiflash/issues/5315) @[hehechen](https://github.com/hehechen)

  v6.2.0以降、TiDBは[即座に物理データをコンパクト化](/sql-statements/sql-statement-alter-table-compact.md#alter-table--compact)する機能をサポートしています。TiFlashのフルテーブルレプリカで物理データを即座にコンパクト化するためのSQLステートメントを手動で実行することで、ストレージスペースを削減し、クエリのパフォーマンスを向上させることができます。v6.4.0では、TiFlashレプリカデータのコンパクト化の粒度を細かくし、指定されたテーブルのTiFlashレプリカを即座にコンパクト化することをサポートしています。

  SQLステートメント`ALTER TABLE table_name COMPACT [PARTITION PartitionNameList] [engine_type REPLICA]`を実行することで、指定されたテーブルのTiFlashレプリカを即座にコンパクト化することができます。

  詳細については、[ユーザードキュメント](/sql-statements/sql-statement-alter-table-compact.md#compact-tiflash-replicas-of-specified-partitions-in-a-table)を参照してください。

- `FLASHBACK CLUSTER TO TIMESTAMP`を使用して、クラスタを特定の時点に復元するサポート (実験的) [#37197](https://github.com/pingcap/tidb/issues/37197) [#13303](https://github.com/tikv/tikv/issues/13303) @[Defined2014](https://github.com/Defined2014) @[bb7133](https://github.com/bb7133) @[JmPotato](https://github.com/JmPotato) @[Connor1996](https://github.com/Connor1996) @[HuSharp](https://github.com/HuSharp) @[CalvinNeo](https://github.com/CalvinNeo)

  `FLASHBACK CLUSTER TO TIMESTAMP`構文を使用して、ガベージコレクション（GC）寿命内でクラスタを特定の時点に迅速に復元できます。この機能により、DMLの誤操作を簡単かつ迅速に元に戻すことができます。例えば、`WHERE`句のない`DELETE`を誤って実行した後、数分で元のクラスタを復元するためにこの構文を使用できます。この機能はデータベースバックアップに依存せず、データの変更が行われた正確な時刻を特定するために異なる時点でデータをロールバックすることをサポートします。`FLASHBACK CLUSTER TO TIMESTAMP`はデータベースバックアップを置き換えることはできません。

  `FLASHBACK CLUSTER TO TIMESTAMP`を実行する前に、TiCDCなどのツールで実行されているPITRとレプリケーションタスクを一時停止し、`FLASHBACK`が完了した後に再開する必要があります。そうしないと、レプリケーションタスクが失敗する可能性があります。

  詳細については、[ユーザードキュメント](/sql-statements/sql-statement-flashback-cluster.md)を参照してください。

- `FLASHBACK DATABASE`を使用して、削除されたデータベースを復元するサポート [#20463](https://github.com/pingcap/tidb/issues/20463) @[erwadba](https://github.com/erwadba)

  `FLASHBACK DATABASE`を使用することで、ガベージコレクション（GC）寿命内で`DROP`によって削除されたデータベースとそのデータを復元することができます。この機能は外部ツールに依存しません。SQLステートメントを使用して、データとメタデータを迅速に復元することができます。

  詳細については、[ユーザードキュメント](/sql-statements/sql-statement-flashback-database.md)を参照してください。

### セキュリティ {#security}

- TiFlash が[安定性のための暗号化](/encryption-at-rest.md#tiflash)のためにSM4アルゴリズムをサポート [#5953](https://github.com/pingcap/tiflash/issues/5953) @[lidezhu](https://github.com/lidezhu)

  TiFlashの安定性のための暗号化にSM4アルゴリズムを追加しました。安定性のための暗号化を構成する際に、`tiflash-learner.toml`構成ファイルで`data-encryption-method`の値を`sm4-ctr`に設定することで、SM4暗号化機能を有効にすることができます。

  詳細については、[ユーザードキュメント](/encryption-at-rest.md#tiflash)を参照してください。

### 観測性 {#observability}

- クラスタ診断がGAになりました [#1438](https://github.com/pingcap/tidb-dashboard/issues/1438) @[Hawkson-jee](https://github.com/Hawkson-jee)

  TiDB Dashboardの[クラスタ診断](/dashboard/dashboard-diagnostics-access.md)機能は、指定された時間範囲内でクラスタに存在する可能性のある問題を診断し、診断結果とクラスタ関連の負荷監視情報を[診断レポート](/dashboard/dashboard-diagnostics-report.md)の形でまとめます。この診断レポートはWebページの形式であり、オフラインでページを閲覧し、ブラウザからページを保存してリンクを共有することができます。

  診断レポートにより、クラスタの基本的な健康情報、負荷、コンポーネントの状態、時間消費、構成などを素早く把握することができます。クラスタに一般的な問題がある場合、[診断情報](/dashboard/dashboard-diagnostics-report.md#diagnostic-information)の結果で原因を特定することができます。

### パフォーマンス {#performance}

- コプロセッサータスクの並行適応メカニズムを導入 [#37724](https://github.com/pingcap/tidb/issues/37724) @[you06](https://github.com/you06)

  コプロセッサータスクの数が増加すると、TiKVの処理速度に基づいて、TiDBは自動的に並行性を増加させます（[`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency)の値を調整）して、コプロセッサータスクのキューを減らし、それによってレイテンシーを低減します。

- テーブルの結合順序を決定するための動的計画アルゴリズムを追加 [#37825](https://github.com/pingcap/tidb/issues/37825) @[winoros](https://github.com/winoros)

  以前のバージョンでは、TiDBはテーブルの結合順序を決定するために貪欲アルゴリズムを使用していました。v6.4.0では、TiDBオプティマイザーは[動的計画アルゴリズム](/join-reorder.md#example-the-dynamic-programming-algorithm-of-join-reorder)を導入しました。動的計画アルゴリズムは、貪欲アルゴリズムよりも多くの可能な結合順序を列挙できるため、より良い実行計画を見つける可能性を高め、一部のシナリオでSQLの実行効率を向上させます。

  動的プログラミングアルゴリズムはより多くの時間を消費するため、TiDB Join Reorderアルゴリズムの選択は[`tidb_opt_join_reorder_threshold`](/system-variables.md#tidb_opt_join_reorder_threshold)変数によって制御されます。Join Reorderに参加するノードの数がこの閾値を超える場合、TiDBは貪欲アルゴリズムを使用します。それ以外の場合、TiDBは動的プログラミングアルゴリズムを使用します。

  詳細については、[ユーザードキュメント](/join-reorder.md)を参照してください。

- プレフィックスインデックスがNULL値のフィルタリングをサポート [#21145](https://github.com/pingcap/tidb/issues/21145) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

  この機能は、プレフィックスインデックスの最適化です。テーブルの列にプレフィックスインデックスがある場合、SQLステートメントの中でその列の`IS NULL`または`IS NOT NULL`条件をプレフィックスで直接フィルタリングできます。これにより、この場合のテーブル検索を回避し、SQLの実行パフォーマンスを向上させます。

  詳細については、[ユーザードキュメント](/system-variables.md#tidb_opt_prefix_index_single_scan-new-in-v640)を参照してください。

- TiDBチャンク再利用メカニズムを強化 [#38606](https://github.com/pingcap/tidb/issues/38606) @[keeplearning20221](https://github.com/keeplearning20221)

  以前のバージョンでは、TiDBは`writechunk`関数でのみチャンクを再利用していました。TiDB v6.4.0では、チャンク再利用メカニズムをExecutor内のオペレーターに拡張しました。チャンクを再利用することで、TiDBは頻繁にメモリ解放を要求する必要がなくなり、一部のシナリオでSQLクエリがより効率的に実行されます。システム変数[`tidb_enable_reuse_chunk`](/system-variables.md#tidb_enable_reuse_chunk-new-in-v640)を使用して、チャンクオブジェクトの再利用を制御できます（デフォルトで有効）。

  詳細については、[ユーザードキュメント](/system-variables.md#tidb_enable_reuse_chunk-new-in-v640)を参照してください。

- 相関サブクエリのdecorrelationを実行するかどうかを制御する新しいオプティマイザーヒント`NO_DECORRELATE`を導入 [#37789](https://github.com/pingcap/tidb/issues/37789) @[time-and-fate](https://github.com/time-and-fate)

  デフォルトでは、TiDBは常に相関サブクエリをdecorrelationするように書き換えますが、これは通常、実行効率を向上させます。ただし、一部のシナリオではdecorrelationが実行効率を低下させます。v6.4.0では、TiDBは指定されたクエリブロックのdecorrelationを実行しないようにするためのオプティマイザーヒント`NO_DECORRELATE`を導入し、一部のシナリオでクエリのパフォーマンスを向上させます。

  詳細については、[ユーザードキュメント](/optimizer-hints.md#no_decorrelate)を参照してください。

- パーティションテーブルの統計収集のパフォーマンスを向上 [#37977](https://github.com/pingcap/tidb/issues/37977) @[Yisaer](https://github.com/Yisaer)

  v6.4.0では、TiDBはパーティションテーブルの統計収集戦略を最適化しました。システム変数[`tidb_auto_analyze_partition_batch_size`](/system-variables.md#tidb_auto_analyze_partition_batch_size-new-in-v640)を使用して、パーティションテーブルの統計を並行して収集するための並行性を設定し、収集と解析時間を短縮できます。

### 安定性 {#stability}

- ディスク障害やI/Oのスタックなどの極端な状況での障害回復を加速 [#13648](https://github.com/tikv/tikv/issues/13648) @[LykxSassinator](https://github.com/LykxSassinator)

  企業ユーザーにとって、データベースの可用性は最も重要なメトリックの1つです。しかし、複雑なハードウェア環境では、障害の迅速な検出と回復が常にデータベースの可用性の課題の1つでした。v6.4.0では、TiDBはTiKVノードの状態検出メカニズムを完全に最適化しました。ディスク障害やI/Oのスタックなどの極端な状況でも、TiDBはノードの状態を迅速に報告し、リーダー選出を事前に起動するためのアクティブなウェイクアップメカニズムを使用して、クラスタの自己修復を加速します。この最適化により、ディスク障害の場合にクラスタの回復時間を約50%短縮できます。

- TiDBメモリ使用量のグローバル制御を導入（実験的） [#37816](https://github.com/pingcap/tidb/issues/37816) @[wshwsh12](https://github.com/wshwsh12)

  v6.4.0では、TiDBはTiDBインスタンスのグローバルメモリ使用量を追跡する実験的な機能としてグローバルメモリ使用量の制御を導入しました。システム変数[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)を使用して、グローバルメモリ使用量の上限を設定できます。メモリ使用量が閾値に達すると、TiDBはより多くの空きメモリを回収して解放しようとします。メモリ使用量が閾値を超えると、TiDBは最もメモリを使用するSQL操作を特定してキャンセルし、過剰なメモリ使用によるシステムの問題を回避します。

  TiDBインスタンスのメモリ消費に潜在的なリスクがある場合、TiDBは事前に診断情報を収集し、指定されたディレクトリに書き込んで問題の診断を容易にします。同時に、TiDBはメモリ使用量と操作履歴を表示するシステムテーブルビュー[`INFORMATION_SCHEMA.MEMORY_USAGE`](/information-schema/information-schema-memory-usage.md)および[`INFORMATION_SCHEMA.MEMORY_USAGE_OPS_HISTORY`](/information-schema/information-schema-memory-usage-ops-history.md)を提供し、メモリ使用量をよりよく理解するのに役立ちます。

  グローバルメモリ制御はTiDBメモリ管理の画期的な進展です。インスタンスのグローバルビューを導入し、メモリの体系的な管理を採用することで、より多くの重要なシナリオでデータベースの安定性とサービスの可用性を大幅に向上させることができます。

  詳細については、[ユーザードキュメント](/configure-memory-usage.md)を参照してください。

- レンジビルディングオプティマイザーのメモリ使用量を制御 [#37176](https://github.com/pingcap/tidb/issues/37176) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

  v6.4.0では、システム変数[`tidb_opt_range_max_size`](/system-variables.md#tidb_opt_range_max_size-new-in-v640)が導入され、レンジをビルドするオプティマイザーの最大メモリ使用量を制限します。メモリ使用量が制限を超えると、オプティマイザーはより粗い範囲を構築する代わりに、より正確な範囲を構築することを減らしてメモリ消費を減らします。SQLステートメントに多くの`IN`条件がある場合、この最適化によりコンパイルのメモリ使用量を大幅に削減し、システムの安定性を確保できます。

  詳細については、[ユーザードキュメント](/system-variables.md#tidb_opt_range_max_size-new-in-v640)を参照してください。

- 同期的に統計情報をロードする機能をサポート（GA） [#37434](https://github.com/pingcap/tidb/issues/37434) @[chrysan](https://github.com/chrysan)

  TiDB v6.4.0では、同期的に統計情報をロードする機能をデフォルトで有効にしました。この機能により、SQLステートメントを実行する際に大規模な統計情報（ヒストグラム、TopN、Count-Min Sketch統計など）をメモリに同期的にロードできるため、SQLの最適化のための統計情報の完全性が向上します。

  詳細については、[ユーザードキュメント](/system-variables.md#tidb_stats_load_sync_wait-new-in-v540)を参照してください。

- 軽量トランザクション書き込みの応答時間に対するバッチ書き込みリクエストの影響を低減 [#13313](https://github.com/tikv/tikv/issues/13313) @[glorv](https://github.com/glorv)

  一部のシステムのビジネスロジックでは、定期的なバッチDMLタスクが必要ですが、これらのバッチ書き込みタスクの処理はオンライントランザクションのレイテンシーを増加させます。v6.3.0では、TiKVはハイブリッドワークロードシナリオでの読み込みリクエストのスケジューリングを最適化し、[`readpool.unified.auto-adjust-pool-size`](/tikv-configuration-file.md#auto-adjust-pool-size-new-in-v630)構成項目を有効にして、すべての読み込みリクエストのためのUnifyReadPoolスレッドプールのサイズを自動的に調整できます。v6.4.0では、TiKVは動的に書き込みリクエストを識別し、1回のポーリングでFSM（有限状態機械）に対して書き込むことができる最大バイト数を制御し、バッチ書き込みリクエストがトランザクション書き込みの応答時間に与える影響を低減します。"

### 利便性 {#ease-of-use}

- TiKV API V2が一般利用可能（GA）になりました [#11745](https://github.com/tikv/tikv/issues/11745) @[pingyu](https://github.com/pingyu)

  v6.1.0以前、TiKVはクライアントから渡された生データのみを提供し、基本的なキー値の読み書き機能のみを提供していました。また、異なるコーディング方法とスコープのデータ範囲のため、TiDB、Transactional KV、およびRawKVは同時に同じTiKVクラスターで使用できませんでした。そのため、この場合は複数のクラスターが必要であり、それによりマシンと展開コストが増加します。

  TiKV API V2は新しいRawKVストレージ形式とアクセスインターフェースを提供し、以下の利点をもたらします：

  - データを変更タイムスタンプでMVCCに保存し、これに基づいてChange Data Capture（CDC）が実装されます。この機能は実験的であり、[TiKV-CDC](https://github.com/tikv/migration/blob/main/cdc/README.md)で詳細に説明されています。
  - データは異なる使用方法に応じてスコープが設定され、API V2は単一クラスターでTiDB、Transactional KV、およびRawKVアプリケーションの共存をサポートします。
  - Key Spaceフィールドを予約して、マルチテナンシーなどの機能をサポートします。

  TiKV API V2を有効にするには、TiKV構成ファイルの`[storage]`セクションで`api-version = 2`を設定します。

  詳細については、[ユーザードキュメント](/tikv-configuration-file.md#api-version-new-in-v610)を参照してください。

- TiFlashデータレプリケーションの進捗の精度を向上させました [#4902](https://github.com/pingcap/tiflash/issues/4902) @[hehechen](https://github.com/hehechen)

  TiDBでは、`INFORMATION_SCHEMA.TIFLASH_REPLICA`テーブルの`PROGRESS`フィールドは、TiKVからTiFlashレプリカへのデータレプリケーションの進捗を示すために使用されます。以前のTiDBバージョンでは、`PROCESS`フィールドはTiFlashレプリカの作成中のデータレプリケーションの進捗のみを提供していました。TiFlashレプリカが作成された後、TiKVの対応するテーブルに新しいデータがインポートされると、このフィールドは新しいデータのTiKVからTiFlashへのレプリケーション進捗を示すように更新されませんでした。

  v6.4.0では、TiDBはTiFlashレプリカのデータレプリケーション進捗の更新メカニズムを改善しました。TiFlashレプリカが作成された後、TiKVの対応するテーブルに新しいデータがインポートされると、[`INFORMATION_SCHEMA.TIFLASH_REPLICA`](/information-schema/information-schema-tiflash-replica.md)テーブルの`PROGRESS`値が更新され、新しいデータのTiKVからTiFlashへの実際のレプリケーション進捗が表示されます。この改善により、TiFlashデータレプリケーションの実際の進捗を簡単に表示できるようになります。

  詳細については、[ユーザードキュメント](/information-schema/information-schema-tiflash-replica.md)を参照してください。

### MySQL互換性 {#mysql-compatibility}

- 線形ハッシュパーティショニング構文と互換性があります [#38450](https://github.com/pingcap/tidb/issues/38450) @[mjonss](https://github.com/mjonss)

  以前のバージョンでは、TiDBはハッシュ、範囲、およびリストパーティショニングをサポートしていました。v6.4.0から、TiDBは[MySQL線形ハッシュパーティショニング](https://dev.mysql.com/doc/refman/5.7/en/partitioning-linear-hash.html)の構文とも互換性があります。

  TiDBでは、既存のMySQL線形ハッシュパーティションのDDLステートメントを直接実行でき、TiDBは対応するハッシュパーティションテーブルを作成します（TiDB内に線形ハッシュパーティションはありません）。また、既存のMySQL線形ハッシュパーティションのDMLステートメントを直接実行でき、TiDBは対応するTiDBハッシュパーティションのクエリ結果を通常通り返します。この機能により、TiDBの構文がMySQL線形ハッシュパーティションと互換性があり、MySQLベースのアプリケーションをTiDBにシームレスに移行できます。

  パーティションの数が2の累乗である場合、TiDBのハッシュパーティションテーブルの行はMySQLの線形ハッシュパーティションテーブルの行と同じように分散されます。それ以外の場合、これらの行の分散はMySQLとは異なります。

  詳細については、[ユーザードキュメント](/partitioned-table.md#how-tidb-handles-linear-hash-partitions)を参照してください。

- 高性能でグローバルに単調増加する`AUTO_INCREMENT`をサポートします（実験的） [#38442](https://github.com/pingcap/tidb/issues/38442) @[tiancaiamao](https://github.com/tiancaiamao)

  TiDB v6.4.0では、`AUTO_INCREMENT`のMySQL互換モードが導入されました。このモードは、すべてのTiDBインスタンスでIDが単調に増加することを保証する中央集権型の自動インクリメントID割り当てサービスを導入します。この機能により、自動インクリメントIDでクエリ結果を簡単にソートできるようになります。MySQL互換モードを使用するには、テーブルを作成する際に`AUTO_ID_CACHE`を`1`に設定する必要があります。以下は例です：

  ```sql
  CREATE TABLE t (a INT AUTO_INCREMENT PRIMARY KEY) AUTO_ID_CACHE = 1;
  ```

  詳細については、[ユーザードキュメント](/auto-increment.md#mysql-compatibility-mode)を参照してください。

- JSONタイプの配列データの範囲選択をサポートします [#13644](https://github.com/tikv/tikv/issues/13644) @[YangKeao](https://github.com/YangKeao)

  v6.4.0から、TiDBではMySQL互換の[範囲選択構文](https://dev.mysql.com/doc/refman/8.0/en/json.html#json-paths)を使用できるようになります。

  - `to`キーワードを使用して、配列要素の開始位置と終了位置を指定し、配列内の連続する範囲の要素を選択できます。`0`を使用して、配列内の最初の要素の位置を指定できます。たとえば、`$[0 to 2]`を使用すると、配列の最初の3つの要素を選択できます。

  - `last`キーワードを使用して、配列内の最後の要素の位置を指定でき、これにより右から左に位置を設定できます。たとえば、`$[last-2 to last]`を使用すると、配列の最後の3つの要素を選択できます。

  この機能により、SQLステートメントの記述プロセスが簡素化され、JSONタイプの互換性がさらに向上し、MySQLアプリケーションをTiDBに移行する難しさが軽減されます。

- データベースユーザーの追加の説明をサポートします [#38172](https://github.com/pingcap/tidb/issues/38172) @[CbcWestwolf](https://github.com/CbcWestwolf)

  TiDB v6.4では、[`CREATE USER`](/sql-statements/sql-statement-create-user.md)または[`ALTER USER`](/sql-statements/sql-statement-alter-user.md)を使用して、データベースユーザーに追加の説明を追加できます。TiDBでは2つの説明形式を提供しています。`COMMENT`を使用してテキストコメントを追加し、`ATTRIBUTE`を使用してJSON形式の構造化された属性セットを追加できます。

  さらに、TiDB v6.4.0では、ユーザーコメントとユーザー属性の情報を表示できる[`USER_ATTRIBUTES`](/information-schema/information-schema-user-attributes.md)テーブルが追加されました。

  ```sql
  CREATE USER 'newuser1'@'%' COMMENT 'This user is created only for test';
  CREATE USER 'newuser2'@'%' ATTRIBUTE '{"email": "user@pingcap.com"}';
  SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES;
  ```

  ```sql
  +-----------+------+---------------------------------------------------+
  | USER      | HOST | ATTRIBUTE                                         |
  +-----------+------+---------------------------------------------------+
  | newuser1  | %    | {"comment": "This user is created only for test"} |
  | newuser1  | %    | {"email": "user@pingcap.com"}                     |
  +-----------+------+---------------------------------------------------+
  2 rows in set (0.00 sec)
  ```

  この機能により、TiDBのMySQL構文との互換性が向上し、MySQLエコシステムのツールやプラットフォームにTiDBを統合することが容易になります。

### バックアップとリストア {#backup-and-restore}

- EBSボリュームスナップショットを使用してTiDBクラスターのバックアップをサポートします [#33849](https://github.com/pingcap/tidb/issues/33849) @[fengou1](https://github.com/fengou1)

  TiDBクラスターがEKS上に展開され、AWS EBSボリュームを使用しており、TiDBクラスターデータをバックアップする際に以下の要件がある場合、TiDB Operatorを使用してデータをボリュームスナップショットとメタデータでAWS S3にバックアップできます：

  - バックアップの影響を最小限に抑える。たとえば、QPSとトランザクションレイテンシに対する影響を5%未満に抑え、クラスターのCPUとメモリを占有しない。
  - 短時間でデータをバックアップおよびリストアする。たとえば、1時間以内にバックアップを完了し、2時間でデータをリストアする。

  詳細については、[ユーザードキュメント](https://docs.pingcap.com/tidb-in-kubernetes/v1.4/backup-to-aws-s3-by-snapshot)を参照してください。

### データ移行 {#data-migration}

- DMは、アップストリームデータソース情報をダウンストリームのマージされたテーブルの拡張列に書き込むことをサポートします [#37797](https://github.com/pingcap/tidb/issues/37797) @[lichunzhu](https://github.com/lichunzhu)

  アップストリームからTiDBへのシャーディングされたスキーマとテーブルのマージ時に、DMタスクの構成時にターゲットテーブルにいくつかのフィールド（拡張列）を手動で追加し、その値を指定できます。たとえば、アップストリームのシャーディングされたスキーマとテーブルの名前を拡張列に指定した場合、DMによってダウンストリームに書き込まれるデータにはスキーマ名とテーブル名が含まれます。ダウンストリームのデータが異常に見える場合、この機能を使用してターゲットテーブル内のデータソース情報（スキーマ名やテーブル名など）を素早く特定できます。

  詳細については、[テーブル、スキーマ、およびソース情報を抽出してマージされたテーブルに書き込む](/dm/dm-table-routing.md#extract-table-schema-and-source-information-and-write-into-the-merged-table)を参照してください。

- DMは、プリチェックメカニズムを最適化し、いくつかの必須チェック項目をオプション項目に変更しました [#7333](https://github.com/pingcap/tiflow/issues/7333) @[lichunzhu](https://github.com/lichunzhu)

  データ移行タスクをスムーズに実行するために、DMはタスク開始時に自動的に[プリチェック](/dm/dm-precheck.md)をトリガーし、チェック結果を返します。プリチェックが合格した後にのみ、DMは移行を開始します。

  v6.4.0では、DMは以下の3つのチェック項目を必須からオプションに変更し、プリチェックの合格率を向上させました。

  - アップストリームのテーブルがTiDBと互換性のない文字セットを使用していないかどうかをチェックします。
  - アップストリームのテーブルにプライマリキー制約またはユニークキー制約があるかどうかをチェックします。
  - アップストリームデータベースのデータベースID `server_id` がプライマリセカンダリ構成で指定されているかどうかをチェックします。

- DMは、増分移行タスクのオプションパラメータとしてbinlogの位置とGTIDの設定をサポートします [#7393](https://github.com/pingcap/tiflow/issues/7393) @[GMHDBJD](https://github.com/GMHDBJD)

  v6.4.0以降、binlogの位置やGTIDを指定せずに直接増分移行を実行できます。DMは、タスク開始後にアップストリームで生成されたbinlogファイルを自動的に取得し、これらの増分データをダウンストリームに移行します。これにより、ユーザーは煩雑な理解や複雑な構成から解放されます。

  詳細については、[DM Advanced Task Configuration File](/dm/task-configuration-file-full.md)を参照してください。

- DMは、移行タスクのためにより多くのステータスインジケータを追加しました [#7343](https://github.com/pingcap/tiflow/issues/7343) @[okJiang](https://github.com/okJiang)

  v6.4.0では、DMは移行タスクのためにより多くのパフォーマンスおよび進行状況のインジケータを追加し、移行のパフォーマンスと進行状況を直感的に理解し、トラブルシューティングの参考情報を提供します。

  - データのインポートおよびエクスポートのパフォーマンスを示すステータスインジケータ（バイト/秒）を追加します。
  - ダウンストリームデータベースへのデータ書き込みのパフォーマンスインジケータをTPSからRPS（行/秒）に名称変更します。
  - DMフル移行タスクのデータエクスポート進行状況を示す進行状況インジケータを追加します。

  これらのインジケータについての詳細については、[TiDBデータ移行でのタスクステータスのクエリ](/dm/dm-query-status.md)を参照してください。

### TiDBデータ共有サブスクリプション {#tidb-data-share-subscription}

- TiCDCは、`3.2.0`バージョンのKafkaへのデータレプリケーションをサポートします [#7191](https://github.com/pingcap/tiflow/issues/7191) @[3AceShowHand](https://github.com/3AceShowHand)

  v6.4.0から、TiCDCは`3.2.0`バージョンおよびそれ以前の[Kafkaへのデータレプリケーション](/replicate-data-to-kafka.md)をサポートします。

## 互換性の変更 {#compatibility-changes}

### システム変数 {#system-variables}

| 変数名                                                                                                                                 | 変更タイプ | 説明                                                                                                                                                                                                                                 |
| ----------------------------------------------------------------------------------------------------------------------------------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630)         | 変更    | GLOBALスコープを削除し、[`pessimistic-txn.constraint-check-in-place-pessimistic`](/tidb-configuration-file.md#constraint-check-in-place-pessimistic-new-in-v640)構成項目を使用してデフォルト値を変更できるようにします。この変数は、TiDBが悲観的トランザクションで一意の制約をチェックするタイミングを制御します。 |
| [`tidb_ddl_flashback_concurrency`](/system-variables.md#tidb_ddl_flashback_concurrency-new-in-v630)                                 | 変更    | v6.4.0から有効で、[`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-cluster.md)の並行性を制御します。デフォルト値は`64`です。                                                                                                        |
| [`tidb_enable_clustered_index`](/system-variables.md#tidb_enable_clustered_index-new-in-v50)                                        | 変更    | デフォルト値を`INT_ONLY`から`ON`に変更し、プライマリキーがデフォルトでクラスタ化インデックスとして作成されるようにします。                                                                                                                                                               |
| [`tidb_enable_paging`](/system-variables.md#tidb_enable_paging-new-in-v540)                                                         | 変更    | デフォルト値を`OFF`から`ON`に変更し、デフォルトでコプロセッサリクエストを送信するためのページング方法を使用するようにします。                                                                                                                                                                |
| [`tidb_enable_prepared_plan_cache`](/system-variables.md#tidb_enable_prepared_plan_cache-new-in-v610)                               | 変更    | SESSIONスコープを追加します。この変数は、[プリペアドプランキャッシュ](/sql-prepared-plan-cache.md)を有効にするかどうかを制御します。                                                                                                                                              |
| [`tidb_memory_usage_alarm_ratio`](/system-variables.md#tidb_memory_usage_alarm_ratio)                                               | 変更    | デフォルト値を`0.8`から`0.7`に変更します。この変数は、tidb-serverメモリアラームをトリガーするメモリ使用率の割合を制御します。                                                                                                                                                          |
| [`tidb_opt_agg_push_down`](/system-variables.md#tidb_opt_agg_push_down)                                                             | 変更    | GLOBALスコープを追加します。この変数は、オプティマイザが集約関数をJoin、Projection、およびUnionAllの前の位置にプッシュダウンする最適化操作を実行するかどうかを制御します。                                                                                                                                |
| [`tidb_prepared_plan_cache_size`](/system-variables.md#tidb_prepared_plan_cache_size-new-in-v610)                                   | 変更    | SESSIONスコープを追加します。この変数は、セッションでキャッシュできるプランの最大数を制御します。                                                                                                                                                                               |
| [`tidb_stats_load_sync_wait`](/system-variables.md#tidb_stats_load_sync_wait-new-in-v540)                                           | 変更    | デフォルト値を`0`から`100`に変更し、SQL実行が同期的に完全な列統計を読み込むために最大で100ミリ秒待機できるようにします。                                                                                                                                                                |
| [`tidb_stats_load_pseudo_timeout`](/system-variables.md#tidb_stats_load_pseudo_timeout-new-in-v540)                                 | 変更    | デフォルト値を`OFF`から`ON`に変更し、同期的に完全な列統計を読み込むタイムアウトに達した後、SQL最適化が擬似統計を再度使用するようにします。                                                                                                                                                        |
| [`last_sql_use_alloc`](/system-variables.md#last_sql_use_alloc-new-in-v640)                                                         | 新規追加  | 前のステートメントがキャッシュされたチャンクオブジェクト（チャンク割り当て）を使用するかどうかを示します。この変数は読み取り専用で、デフォルト値は`OFF`です。                                                                                                                                                  |
| [`tidb_auto_analyze_partition_batch_size`](/system-variables.md#tidb_auto_analyze_partition_batch_size-new-in-v640)                 | 新規追加  | パーティションテーブルを分析する際にTiDBが[自動的に分析](/statistics.md#automatic-update)できるパーティションの数を指定します（つまり、パーティションテーブルの統計情報を自動的に収集します）。デフォルト値は`1`です。                                                                                                   |
| [`tidb_enable_external_ts_read`](/system-variables.md#tidb_enable_external_ts_read-new-in-v640)                                     | 新規追加  | TiDBが[`tidb_external_ts`](/system-variables.md#tidb_external_ts-new-in-v640)で指定されたタイムスタンプでデータを読み取るかどうかを制御します。デフォルト値は`OFF`です。                                                                                                       |
| [`tidb_enable_gogc_tuner`](/system-variables.md#tidb_enable_gogc_tuner-new-in-v640)                                                 | 新規追加  | GOGC Tunerを有効にするかどうかを制御します。デフォルト値は`ON`です。                                                                                                                                                                                          |
| [`tidb_enable_reuse_chunk`](/system-variables.md#tidb_enable_reuse_chunk-new-in-v640)                                               | 新規追加  | TiDBがチャンクオブジェクトキャッシュを有効にするかどうかを制御します。デフォルト値は`ON`で、TiDBはキャッシュされたチャンクオブジェクトを使用し、キャッシュされたオブジェクトがない場合にのみシステムからリクエストします。値が`OFF`の場合、TiDBは直接システムからチャンクオブジェクトを要求します。                                                                       |
| [`tidb_enable_prepared_plan_cache_memory_monitor`](/system-variables.md#tidb_enable_prepared_plan_cache_memory_monitor-new-in-v640) | 新規追加  | プリペアドプランキャッシュにキャッシュされた実行プランによって消費されるメモリをカウントするかどうかを制御します。デフォルト値は`ON`です。                                                                                                                                                            |
| [`tidb_external_ts`](/system-variables.md#tidb_external_ts-new-in-v640)                                                             | 新規追加  | デフォルト値は`0`です。[`tidb_enable_external_ts_read`](/system-variables.md#tidb_enable_external_ts_read-new-in-v640)が`ON`に設定されている場合、TiDBはこの変数で指定されたタイムスタンプでデータを読み取ります。                                                                     |
| [`tidb_gogc_tuner_threshold`](/system-variables.md#tidb_gogc_tuner_threshold-new-in-v640)                                           | 新規追加  | GOGCの最大メモリ閾値を指定します。メモリがこの閾値を超えると、GOGC Tunerは動作を停止します。デフォルト値は`0.6`です。                                                                                                                                                               |
| [`tidb_memory_usage_alarm_keep_record_num`](/system-variables.md#tidb_memory_usage_alarm_keep_record_num-new-in-v640)               | 新規追加  | tidb-serverメモリ使用量がメモリアラーム閾値を超えてアラームをトリガーし、TiDBはデフォルトで直近の5回のアラームで生成されたステータスファイルのみを保持します。この数をこの変数で調整できます。                                                                                                                            |
| [`tidb_opt_prefix_index_single_scan`](/system-variables.md#tidb_opt_prefix_index_single_scan-new-in-v640)                           | 新規追加  | TiDBオプティマイザが一部のフィルタ条件をプレフィックスインデックスにプッシュダウンして不要なテーブル検索を回避し、クエリのパフォーマンスを向上させるかどうかを制御します。デフォルト値は`ON`です。                                                                                                                              |
| [`tidb_opt_range_max_size`](/system-variables.md#tidb_opt_range_max_size-new-in-v640)                                               | 新規追加  | オプティマイザがスキャン範囲を構築するためのメモリ使用量の上限を指定します。デフォルト値は`67108864`（64 MiB）です。                                                                                                                                                                 |
| [`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)                                             | 新規追加  | オプティマイザがスキャン範囲を構築するためのメモリ使用量の上限を制御します（実験的）。デフォルト値は`0`で、メモリ制限はありません。                                                                                                                                                                |
| [`tidb_server_memory_limit_gc_trigger`](/system-variables.md#tidb_server_memory_limit_gc_trigger-new-in-v640)                       | 新規追加  | TiDBがGCをトリガしようとするしきい値を制御します（実験的）。デフォルト値は`70%`です。                                                                                                                                                                                   |
| [`tidb_server_memory_limit_sess_min_size`](/system-variables.md#tidb_server_memory_limit_sess_min_size-new-in-v640)                 | 新規追加  | メモリ制限を有効にした後、TiDBは現在のインスタンスで最もメモリを使用するSQLステートメントを終了します。この変数は、終了するSQLステートメントの最小メモリ使用量を指定します。デフォルト値は`134217728`（128 MiB）です。                                                                                                          |

### 設定ファイルパラメータ {#configuration-file-parameters}

| 設定ファイル  | 設定パラメータ                                                                                                                                   | 変更タイプ | 説明                                                                                                                                                              |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TiDB    | `tidb_memory_usage_alarm_ratio`                                                                                                           | 削除    | この設定項目はもはや有効ではありません。                                                                                                                                            |
| TiDB    | `memory-usage-alarm-ratio`                                                                                                                | 削除    | システム変数[`tidb_memory_usage_alarm_ratio`](/system-variables.md#tidb_memory_usage_alarm_ratio)に置き換えられました。この設定項目がv6.4.0より前のTiDBバージョンで構成されている場合、アップグレード後には効果がありません。  |
| TiDB    | [`pessimistic-txn.constraint-check-in-place-pessimistic`](/tidb-configuration-file.md#constraint-check-in-place-pessimistic-new-in-v640)  | 新規追加  | システム変数[`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630)のデフォルト値を制御します。デフォルト値は`true`です。 |
| TiDB    | [`tidb-max-reuse-chunk`](/tidb-configuration-file.md#tidb-max-reuse-chunk-new-in-v640)                                                    | 新規追加  | チャンク割り当ての最大キャッシュされたチャンクオブジェクトを制御します。デフォルト値は`64`です。                                                                                                              |
| TiDB    | [`tidb-max-reuse-column`](/tidb-configuration-file.md#tidb-max-reuse-column-new-in-v640)                                                  | 新規追加  | チャンク割り当ての最大キャッシュされた列オブジェクトを制御します。デフォルト値は`256`です。                                                                                                                |
| TiKV    | [`cdc.raw-min-ts-outlier-threshold`](https://docs.pingcap.com/tidb/v6.2/tikv-configuration-file#raw-min-ts-outlier-threshold-new-in-v620) | 廃止    | この設定項目はもはや有効ではありません。                                                                                                                                            |
| TiKV    | [`causal-ts.alloc-ahead-buffer`](/tikv-configuration-file.md#alloc-ahead-buffer-new-in-v640)                                              | 新規追加  | 事前に割り当てられたTSOキャッシュサイズ（期間で指定）を制御します。デフォルト値は`3s`です。                                                                                                               |
| TiKV    | [`causal-ts.renew-batch-max-size`](/tikv-configuration-file.md#renew-batch-max-size-new-in-v640)                                          | 新規追加  | タイムスタンプリクエスト内のTSOの最大数を制御します。デフォルト値は`8192`です。                                                                                                                    |
| TiKV    | [`raftstore.apply-yield-write-size`](/tikv-configuration-file.md#apply-yield-write-size-new-in-v640)                                      | 新規追加  | Applyスレッドが1回のポーリングで1つのFSM（有限状態機械）に書き込むことができる最大バイト数を制御します。デフォルト値は`32KiB`です。これはソフトリミットです。                                                                         |
| PD      | [`tso-update-physical-interval`](/pd-configuration-file.md#tso-update-physical-interval)                                                  | 新規追加  | v6.4.0から有効になり、PDがTSOの物理時間を更新する間隔を制御します。デフォルト値は`50ms`です。                                                                                                         |
| TiFlash | [`data-encryption-method`](/tiflash/tiflash-configuration.md#configure-the-tiflash-learnertoml-file)                                      | 変更    | 新しい値オプション`sm4-ctr`を導入します。この設定項目が`sm4-ctr`に設定されている場合、データはストレージ前にSM4を使用して暗号化されます。                                                                                 |
| DM      | [`routes.route-rule-1.extract-table`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)                      | 新規追加  | シャーディングシナリオでオプションです。シャードされたテーブルのソース情報を抽出するために使用されます。抽出された情報は、データソースを識別するためにダウンストリームのマージされたテーブルに書き込まれます。このパラメータが構成されている場合、事前にダウンストリームでマージされたテーブルを手動で作成する必要があります。 |
| DM      | [`routes.route-rule-1.extract-schema`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)                     | 新規追加  | シャーディングシナリオでオプションです。シャードされたスキーマのソース情報を抽出するために使用されます。抽出された情報は、データソースを識別するためにダウンストリームのマージされたテーブルに書き込まれます。このパラメータが構成されている場合、事前にダウンストリームでマージされたテーブルを手動で作成する必要があります。 |
| DM      | [`routes.route-rule-1.extract-source`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)                     | 新規追加  | シャーディングシナリオでオプションです。ソースインスタンス情報を抽出するために使用されます。抽出された情報は、データソースを識別するためにダウンストリームのマージされたテーブルに書き込まれます。このパラメータが構成されている場合、事前にダウンストリームでマージされたテーブルを手動で作成する必要があります。       |
| TiCDC   | [`transaction-atomicity`](/ticdc/ticdc-sink-to-mysql.md#configure-sink-uri-for-mysql-or-tidb)                                             | 変更    | デフォルト値を`table`から`none`に変更します。この変更により、レプリケーションの遅延とOOMリスクが低減されます。また、TiCDCは今後、すべてのトランザクションではなく、一部のトランザクション（単一トランザクションのサイズが1024行を超える）のみを分割します。                      |

### その他 {#others}

- v6.4.0から、`mysql.user`テーブルには2つの新しい列`User_attributes`と`Token_issuer`が追加されます。以前のTiDBバージョンのバックアップデータから`mysql`スキーマのシステムテーブルを[復元](/br/br-snapshot-guide.md#restore-tables-in-the-mysql-schema)すると、BRは`mysql.user`テーブルの`column count mismatch`エラーを報告します。`mysql`スキーマのシステムテーブルを復元しない場合、このエラーは報告されません。
- Dumplingエクスポートファイルの[形式に一致するファイル](/dumpling-overview.md#format-of-exported-files)で、非圧縮形式で終わるファイル（`test-schema-create.sql.origin`や`test.table-schema.sql.origin`など）について、TiDB Lightningが処理する方法が変更されました。v6.4.0より前は、インポートするファイルにこのようなファイルが含まれている場合、TiDB Lightningはこれらのファイルのインポートをスキップします。v6.4.0からは、TiDB Lightningはこのようなファイルがサポートされていない圧縮形式を使用していると見なし、インポートタスクが失敗します。
- v6.4.0から、`SYSTEM_VARIABLES_ADMIN`または`SUPER`権限を持つチェンジフィードのみがTiCDC Syncpoint機能を使用できます。"

## 改善 {#improvements}

- TiDB

  - noop変数`lc_messages`の変更を許可する[#38231](https://github.com/pingcap/tidb/issues/38231) @[djshow832](https://github.com/djshow832)
  - クラスタ化された複合インデックスの最初の列として`AUTO_RANDOM`列をサポートする[#38572](https://github.com/pingcap/tidb/issues/38572) @[tangenta](https://github.com/tangenta)
  - 内部トランザクションのリトライで悲観的トランザクションを使用して、リトライの失敗を避け、時間の消費を減らす[#38136](https://github.com/pingcap/tidb/issues/38136) @[jackysp](https://github.com/jackysp)

- TiKV

  - 新しい設定項目`apply-yield-write-size`を追加して、Applyスレッドが1回のポーリングで1つの有限状態マシンに書き込むことができる最大バイト数を制御し、Applyスレッドが大量のデータを書き込む際のRaftstoreの混雑を緩和する[#13313](https://github.com/tikv/tikv/issues/13313) @[glorv](https://github.com/glorv)
  - リージョンのリーダーの移行前にエントリーキャッシュをウォームアップして、リーダーの移行プロセス中のQPSの揺れを避ける[#13060](https://github.com/tikv/tikv/issues/13060) @[cosven](https://github.com/cosven)
  - `json_constains`オペレータをCoprocessorにプッシュダウンするサポート[#13592](https://github.com/tikv/tikv/issues/13592) @[lizhenhuan](https://github.com/lizhenhuan)
  - `CausalTsProvider`の非同期関数を追加して、一部のシナリオでのフラッシュパフォーマンスを改善する[#13428](https://github.com/tikv/tikv/issues/13428) @[zeminzhou](https://github.com/zeminzhou)

- PD

  - ホットリージョンスケジューラのv2アルゴリズムがGAになりました。いくつかのシナリオでは、v2アルゴリズムは構成された両方のディメンションでのバランスをより良く達成し、無効なスケジューリングを減らすことができます[#5021](https://github.com/tikv/pd/issues/5021) @[HundunDM](https://github.com/hundundm)
  - オペレータステップのタイムアウトメカニズムを最適化して、早すぎるタイムアウトを避ける[#5596](https://github.com/tikv/pd/issues/5596) @[bufferflies](https://github.com/bufferflies)
  - 大規模クラスタでのスケジューラのパフォーマンスを改善する[#5473](https://github.com/tikv/pd/issues/5473) @[bufferflies](https://github.com/bufferflies)
  - PDが提供しない外部タイムスタンプの使用をサポートする[#5637](https://github.com/tikv/pd/issues/5637) @[lhy1024](https://github.com/lhy1024)

- TiFlash

  - TiFlash MPPエラーハンドリングロジックをリファクタリングして、MPPの安定性をさらに向上させる[#5095](https://github.com/pingcap/tiflash/issues/5095) @[windtalker](https://github.com/windtalker)
  - TiFlash計算プロセスのソートを最適化し、JoinとAggregationのキーハンドリングを最適化する[#5294](https://github.com/pingcap/tiflash/issues/5294) @[solotzg](https://github.com/solotzg)
  - デコードのメモリ使用量を最適化し、余分な転送列を削除してJoinのパフォーマンスを改善する[#6157](https://github.com/pingcap/tiflash/issues/6157) @[yibin87](https://github.com/yibin87)

- ツール

  - TiDBダッシュボード

    - モニタリングページでTiFlashメトリクスを表示するサポートと、そのページでのメトリクスの表示を最適化する[#1440](https://github.com/pingcap/tidb-dashboard/issues/1440) @[YiniXu9506](https://github.com/YiniXu9506)
    - 遅いクエリリストとSQLステートメントリストの結果の行数を表示する[#1443](https://github.com/pingcap/tidb-dashboard/issues/1443) @[baurine](https://github.com/baurine)
    - Alertmanagerが存在しない場合にDashboardがAlertmanagerエラーを報告しないように最適化する[#1444](https://github.com/pingcap/tidb-dashboard/issues/1444) @[baurine](https://github.com/baurine)

  - バックアップとリストア（BR）

    - メタデータの読み込みメカニズムを改善し、PITR中のメモリ使用量を大幅に削減するために、メタデータは必要な場合にのみメモリに読み込まれる[#38404](https://github.com/pingcap/tidb/issues/38404) @[YuJuncen](https://github.com/YuJuncen)

  - TiCDC

    - 交換パーティションDDLステートメントのレプリケーションをサポートする[#639](https://github.com/pingcap/tiflow/issues/639) @[asddongmen](https://github.com/asddongmen)
    - MQシンクモジュールの非バッチ送信パフォーマンスを改善する[#7353](https://github.com/pingcap/tiflow/issues/7353) @[hi-rustin](https://github.com/hi-rustin)
    - テーブルに大量のリージョンがある場合のTiCDCプラーラのパフォーマンスを改善する[#7078](https://github.com/pingcap/tiflow/issues/7078) [#7281](https://github.com/pingcap/tiflow/issues/7281) @[sdojjy](https://github.com/sdojjy)
    - Syncpointが有効な場合、ダウンストリームTiDBで外部TSリードを使用して履歴データを読み取るサポートを追加する[#7419](https://github.com/pingcap/tiflow/issues/7419) @[asddongmen](https://github.com/asddongmen)
    - レプリケーションの安定性を向上させるために、トランザクションの分割を有効にし、デフォルトでsafeModeを無効にする[#7505](https://github.com/pingcap/tiflow/issues/7505) @[asddongmen](https://github.com/asddongmen)

  - TiDBデータ移行（DM）

    - dmctlから無用な`operate-source update`コマンドを削除する[#7246](https://github.com/pingcap/tiflow/issues/7246) @[buchuitoudegou](https://github.com/buchuitoudegou)
    - DMフルインポートが失敗する問題を修正する。上流データベースがTiDBでサポートされていないDDLステートメントを使用する場合は、事前にTiDBでサポートされているDDLステートメントを使用してターゲットテーブルのスキーマを手動で作成して、インポートが成功するようにする[#37984](https://github.com/pingcap/tidb/issues/37984) @[lance6716](https://github.com/lance6716)

  - TiDB Lightning

    - スキーマファイルのスキャンを高速化するためのファイルスキャンロジックを最適化する[#38598](https://github.com/pingcap/tidb/issues/38598) @[dsdashun](https://github.com/dsdashun)

## バグ修正 {#bug-fixes}

- TiDB

  - Fix the potential issue of index inconsistency that occurs after you create a new index [#38165](https://github.com/pingcap/tidb/issues/38165) @[tangenta](https://github.com/tangenta)
  - Fix a permission issue of the `INFORMATION_SCHEMA.TIKV_REGION_STATUS` table [#38407](https://github.com/pingcap/tidb/issues/38407) @[CbcWestwolf](https://github.com/CbcWestwolf)
  - Fix the issue that the `grantor` field is missing in the `mysql.tables_priv` table [#38293](https://github.com/pingcap/tidb/issues/38293) @[CbcWestwolf](https://github.com/CbcWestwolf)
  - Fix the issue that the join result of common table expressions might be wrong [#38170](https://github.com/pingcap/tidb/issues/38170) @[wjhuang2016](https://github.com/wjhuang2016)
  - Fix the issue that the union result of common table expressions might be wrong [#37928](https://github.com/pingcap/tidb/issues/37928) @[YangKeao](https://github.com/YangKeao)
  - Fix the issue that the information in the **transaction region num** monitoring panel is incorrect [#38139](https://github.com/pingcap/tidb/issues/38139) @[jackysp](https://github.com/jackysp)
  - Fix the issue that the system variable [`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630) might affect internal transactions. The variable scope is modified to SESSION. [#38766](https://github.com/pingcap/tidb/issues/38766) @[ekexium](https://github.com/ekexium)
  - Fix the issue that conditions in a query are mistakenly pushed down to projections [#35623](https://github.com/pingcap/tidb/issues/35623) @[Reminiscent](https://github.com/Reminiscent)
  - Fix the issue that the wrong `isNullRejected` check results for `AND` and `OR` cause wrong query results [#38304](https://github.com/pingcap/tidb/issues/38304) @[Yisaer](https://github.com/Yisaer)
  - Fix the issue that `ORDER BY` in `GROUP_CONCAT` is not considered when the outer join is eliminated, which causes wrong query results [#18216](https://github.com/pingcap/tidb/issues/18216) @[winoros](https://github.com/winoros)
  - Fix the issue of the wrong query result that occurs when the mistakenly pushed-down conditions are discarded by Join Reorder [#38736](https://github.com/pingcap/tidb/issues/38736) @[winoros](https://github.com/winoros)

- TiKV

  - Fix the issue that TiDB fails to start on Gitpod when there are multiple `cgroup` and `mountinfo` records [#13660](https://github.com/tikv/tikv/issues/13660) @[tabokie](https://github.com/tabokie)
  - Fix the wrong expression of a TiKV metric `tikv_gc_compaction_filtered` [#13537](https://github.com/tikv/tikv/issues/13537) @[Defined2014](https://github.com/Defined2014)
  - Fix the performance issue caused by the abnormal `delete_files_in_range` [#13534](https://github.com/tikv/tikv/issues/13534) @[tabokie](https://github.com/tabokie)
  - Fix abnormal Region competition caused by expired lease during snapshot acquisition [#13553](https://github.com/tikv/tikv/issues/13553) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
  - Fix errors occurred when `FLASHBACK` fails in the first batch [#13672](https://github.com/tikv/tikv/issues/13672) [#13704](https://github.com/tikv/tikv/issues/13704) [#13723](https://github.com/tikv/tikv/issues/13723) @[HuSharp](https://github.com/HuSharp)

- PD

  - Fix inaccurate Stream timeout and accelerate leader switchover [#5207](https://github.com/tikv/pd/issues/5207) @[CabinfeverB](https://github.com/CabinfeverB)

- TiFlash

  - Fix the OOM issue due to oversized WAL files that occurs when PageStorage GC does not clear the Page deletion marker properly [#6163](https://github.com/pingcap/tiflash/issues/6163) @[JaySon-Huang](https://github.com/JaySon-Huang)

- Tools

  - TiDB Dashboard

    - Fix the TiDB OOM issue when querying execution plans of certain complex SQL statements [#1386](https://github.com/pingcap/tidb-dashboard/issues/1386) @[baurine](https://github.com/baurine)
    - Fix the issue that the Top SQL switch might not take effect when NgMonitoring loses the connection to the PD nodes [#164](https://github.com/pingcap/ng-monitoring/issues/164) @[zhongzc](https://github.com/zhongzc)

  - Backup & Restore (BR)

    - Fix the restoration failure issue caused by PD leader switch during the restoration process [#36910](https://github.com/pingcap/tidb/issues/36910) @[MoCuishle28](https://github.com/MoCuishle28)
    - Fix the issue that the log backup task cannot be paused [#38250](https://github.com/pingcap/tidb/issues/38250) @[joccau](https://github.com/joccau)
    - Fix the issue that when BR deletes log backup data, it mistakenly deletes data that should not be deleted [#38939](https://github.com/pingcap/tidb/issues/38939) @[Leavrth](https://github.com/leavrth)
    - Fix the issue that BR fails to delete data when deleting the log backup data stored in Azure Blob Storage or Google Cloud Storage for the first time [#38229](https://github.com/pingcap/tidb/issues/38229) @[Leavrth](https://github.com/leavrth)

  - TiCDC

    - Fix the issue that `sasl-password` in the `changefeed query` result is not masked [#7182](https://github.com/pingcap/tiflow/issues/7182) @[dveeden](https://github.com/dveeden)
    - Fix the issue that TiCDC might become unavailable when too many operations in an etcd transaction are committed [#7131](https://github.com/pingcap/tiflow/issues/7131) @[asddongmen](https://github.com/asddongmen)
    - Fix the issue that redo logs might be deleted incorrectly [#6413](https://github.com/pingcap/tiflow/issues/6413) @[asddongmen](https://github.com/asddongmen)
    - Fix performance regression when replicating wide tables in Kafka Sink V2 [#7344](https://github.com/pingcap/tiflow/issues/7344) @[hi-rustin](https://github.com/hi-rustin)
    - Fix the issue that checkpoint ts might be advanced incorrectly [#7274](https://github.com/pingcap/tiflow/issues/7274) @[hi-rustin](https://github.com/hi-rustin)
    - Fix the issue that too many logs are printed due to improper log level of the mounter module [#7235](https://github.com/pingcap/tiflow/issues/7235) @[hi-rustin](https://github.com/hi-rustin)
    - Fix the issue that a TiCDC cluster might have two owners [#4051](https://github.com/pingcap/tiflow/issues/4051) @[asddongmen](https://github.com/asddongmen)

  - TiDB Data Migration (DM)

    - Fix the issue that DM WebUI generates the wrong `allow-list` parameter [#7096](https://github.com/pingcap/tiflow/issues/7096) @[zoubingwu](https://github.com/zoubingwu)
    - Fix the issue that a DM-worker has a certain probability of triggering data race when it starts or stops [#6401](https://github.com/pingcap/tiflow/issues/6401) @[liumengya94](https://github.com/liumengya94)
    - Fix the issue that when DM replicates an `UPDATE` or `DELETE` statement but the corresponding row data does not exist, DM silently ignores the event [#6383](https://github.com/pingcap/tiflow/issues/6383) @[GMHDBJD](https://github.com/GMHDBJD)
    - Fix the issue that the `secondsBehindMaster` field is not displayed after you run the `query-status` command [#7189](https://github.com/pingcap/tiflow/issues/7189) @[GMHDBJD](https://github.com/GMHDBJD)
    - Fix the issue that updating the checkpoint may trigger a large transaction [#5010](https://github.com/pingcap/tiflow/issues/5010) @[lance6716](https://github.com/lance6716)
    - Fix the issue that in full task mode, when a task enters the sync stage and fails immediately, DM may lose upstream table schema information [#7159](https://github.com/pingcap/tiflow/issues/7159) @[lance6716](https://github.com/lance6716)
    - Fix the issue that deadlock may be triggered when the consistency check is enabled [#7241](https://github.com/pingcap/tiflow/issues/7241) @[buchuitoudegou](https://github.com/buchuitoudegou)
    - Fix the issue that task precheck requires the `SELECT` privilege for the `INFORMATION_SCHEMA` table [#7317](https://github.com/pingcap/tiflow/issues/7317) @[lance6716](https://github.com/lance6716)
    - Fix the issue that an empty TLS configuration causes an error [#7384](https://github.com/pingcap/tiflow/issues/7384) @[liumengya94](https://github.com/liumengya94)

  - TiDB Lightning

    - Fix the import performance degradation when importing the Apache Parquet files to the target tables that contain the string type columns in the`binary` encoding format [#38351](https://github.com/pingcap/tidb/issues/38351) @[dsdashun](https://github.com/dsdashun)

  - TiDB Dumpling

    - Fix the issue that Dumpling might time out when exporting a lot of tables [#36549](https://github.com/pingcap/tidb/issues/36549) @[lance6716](https://github.com/lance6716)
    - Fix lock errors reported when consistency lock is enabled but the upstream has no target table [#38683](https://github.com/pingcap/tidb/issues/38683) @[lance6716](https://github.com/lance6716)

## 貢献者 {#contributors}

TiDBコミュニティから以下の貢献者に感謝したいと思います：

- [645775992](https://github.com/645775992)
- [An-DJ](https://github.com/An-DJ)
- [AndrewDi](https://github.com/AndrewDi)
- [erwadba](https://github.com/erwadba)
- [fuzhe1989](https://github.com/fuzhe1989)
- [goldwind-ting](https://github.com/goldwind-ting) (初めての貢献者)
- [h3n4l](https://github.com/h3n4l)
- [igxlin](https://github.com/igxlin) (初めての貢献者)
- [ihcsim](https://github.com/ihcsim)
- [JigaoLuo](https://github.com/JigaoLuo)
- [morgo](https://github.com/morgo)
- [Ranxy](https://github.com/Ranxy)
- [shenqidebaozi](https://github.com/shenqidebaozi) (初めての貢献者)
- [taofengliu](https://github.com/taofengliu) (初めての貢献者)
- [TszKitLo40](https://github.com/TszKitLo40)
- [wxbty](https://github.com/wxbty) (初めての貢献者)
- [zgcbj](https://github.com/zgcbj)
