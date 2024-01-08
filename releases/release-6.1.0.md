---
title: TiDB 6.1.0 Release Notes
summary: Learn about the new features, compatibility changes, improvements, and bug fixes in TiDB 6.1.0.
---

# TiDB 6.1.0 リリースノート {#tidb-6-1-0-release-notes}

リリース日: 2022年6月13日

TiDB バージョン: 6.1.0

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.1/quick-start-with-tidb) | [本番展開](https://docs.pingcap.com/tidb/v6.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.1.0#version-list)

6.1.0 では、主な新機能や改善点は以下の通りです:

- List パーティショニングと List COLUMNS パーティショニングが GA になり、MySQL 5.7 と互換性があります
- TiFlash パーティションテーブル (動的プルーニング) が GA になりました
- MySQL と互換性のあるユーザーレベルのロック管理をサポート
- トランザクション非対応の DML ステートメントをサポートします (`DELETE` のみサポート)
- TiFlash がオンデマンドデータのコンパクションをサポート
- MPP がウィンドウ関数フレームワークを導入
- TiCDC が Avro 経由で Kafka に変更ログをレプリケートするサポート
- TiCDC がレプリケーション中に大規模トランザクションを分割するサポートを追加し、大規模トランザクションによるレプリケーションレイテンシーを大幅に低減
- シャーディングされたテーブルのマージと移行のための楽観的モードが GA になりました

## 新機能 {#new-features}

### SQL {#sql}

- List パーティショニングと List COLUMNS パーティショニングが GA になりました。両方とも MySQL 5.7 と互換性があります。

  ユーザードキュメント: [List partitioning](/partitioned-table.md#list-partitioning), [List COLUMNS partitioning](/partitioned-table.md#list-columns-partitioning)

- TiFlash がコンパクトコマンドを開始することをサポートします (実験的)

  TiFlash v6.1.0 では、既存のバックグラウンドコンパクションメカニズムに基づいて物理データを手動でコンパクト化する `ALTER TABLE ... COMPACT` ステートメントを導入しました。このステートメントを使用すると、既存のフォーマットのデータを更新し、適切なタイミングでいつでも読み書きパフォーマンスを向上させることができます。v6.1.0 にクラスターをアップグレードした後、このステートメントを実行してデータをコンパクト化することをお勧めします。このステートメントは標準の SQL 構文の拡張であり、そのため MySQL クライアントと互換性があります。通常、TiFlash のアップグレード以外のシナリオでは、このステートメントを使用する必要はありません。

  [ユーザードキュメント](/sql-statements/sql-statement-alter-table-compact.md), [#4145](https://github.com/pingcap/tiflash/issues/4145)

- TiFlash がウィンドウ関数フレームワークを実装し、以下のウィンドウ関数をサポートします:

  - `RANK()`
  - `DENSE_RANK()`
  - `ROW_NUMBER()`

  [ユーザードキュメント](/tiflash/tiflash-supported-pushdown-calculations.md), [#33072](https://github.com/pingcap/tidb/issues/33072)

### 可観測性 {#observability}

- Continuous Profiling が ARM アーキテクチャと TiFlash をサポート

  [ユーザードキュメント](/dashboard/continuous-profiling.md)

- Grafana がパフォーマンス概要ダッシュボードを追加し、システムレベルの全体的なパフォーマンス診断のためのエントリを提供

  TiDB ビジュアライズモニタリングコンポーネント Grafana の新しいダッシュボードであるパフォーマンス概要は、システムレベルの全体的なパフォーマンス診断のためのエントリを提供します。トップダウンのパフォーマンス分析手法に従い、パフォーマンス概要ダッシュボードはデータベース時間の分解に基づいて TiDB パフォーマンスメトリクスを再構成し、これらのメトリクスを異なる色で表示します。これらの色を確認することで、一目でシステム全体のパフォーマンスボトルネックを特定でき、パフォーマンス診断時間を大幅に短縮し、パフォーマンス分析と診断を簡素化できます。

  [ユーザードキュメント](/performance-tuning-overview.md)

### パフォーマンス {#performance}

- カスタマイズされたリージョンサイズをサポート

  v6.1.0 から、[`coprocessor.region-split-size`](/tikv-configuration-file.md#region-split-size) を構成してリージョンをより大きなサイズに設定できます。これにより、リージョンの数を効果的に減らし、リージョンを管理しやすくし、クラスターのパフォーマンスと安定性を向上させることができます。

  [ユーザードキュメント](/tune-region-performance.md#use-region-split-size-to-adjust-region-size), [#11515](https://github.com/tikv/tikv/issues/11515)

- 並列性を高めるためにバケットを使用するサポート (実験的)

  リージョンをより大きなサイズに設定した後も、クエリの並列性をさらに向上させるために、TiDB はバケットという概念を導入しました。バケットをクエリ単位として使用することで、リージョンをより大きなサイズに設定した場合の並列クエリパフォーマンスを最適化できます。バケットをクエリ単位として使用することで、ホットスポットリージョンのサイズを動的に調整してスケジューリング効率と負荷分散を確保できます。この機能は現在実験的です。本番環境で使用することはお勧めしません。

  [ユーザードキュメント](/tune-region-performance.md#use-bucket-to-increase-concurrency), [#11515](https://github.com/tikv/tikv/issues/11515)

- デフォルトのログストレージエンジンとして Raft Engine を使用するサポート

  v6.1.0 以降、TiDB はログのデフォルトストレージエンジンとして Raft Engine を使用します。RocksDB と比較して、Raft Engine は TiKV の I/O 書き込みトラフィックを最大 40%、CPU 使用率を 10% 減少させ、特定の負荷下で前景スループットを約 5% 向上させ、テールレイテンシを 20% 減少させることができます。

  [ユーザードキュメント](/tikv-configuration-file.md#raft-engine), [#95](https://github.com/tikv/raft-engine/issues/95)

- ジョイン順序ヒント構文をサポート

  - `LEADING` ヒントは、オプティマイザに指定された順序をジョイン操作のプレフィックスとして使用するように促します。良好なジョインのプレフィックスは、ジョインの初期段階でデータ量を迅速に減らし、クエリパフォーマンスを向上させることができます。
  - `STRAIGHT_JOIN` ヒントは、`FROM` 句のテーブルの順序と一致する順序でテーブルをジョインするようにオプティマイザに促します。

  これにより、テーブルジョインの順序を修正する方法が提供されます。ヒントの適切な使用は、SQL パフォーマンスとクラスターの安定性を効果的に向上させることができます。

  ユーザードキュメント: [`LEADING`](/optimizer-hints.md#leadingt1_name--tl_name-), [`STRAIGHT_JOIN`](/optimizer-hints.md#straight_join), [#29932](https://github.com/pingcap/tidb/issues/29932)

- TiFlash がさらに 4 つの関数をサポート

  - `FROM_DAYS`
  - `TO_DAYS`
  - `TO_SECONDS`
  - `WEEKOFYEAR`

  [ユーザードキュメント](/tiflash/tiflash-supported-pushdown-calculations.md), [#4679](https://github.com/pingcap/tiflash/issues/4679), [#4678](https://github.com/pingcap/tiflash/issues/4678), [#4677](https://github.com/pingcap/tiflash/issues/4677)

- TiFlash がパーティションテーブルを動的プルーニングモードでサポート

  OLAP シナリオでのパフォーマンスを向上させるために、パーティションテーブルの動的プルーニングモードがサポートされています。TiDB が v6.0.0 より前のバージョンからアップグレードされた場合、既存のパーティションテーブルの統計情報を手動で更新することをお勧めします。これにより、パフォーマンスを最大化できます (v6.1.0 へのアップグレード後に新しいインストールまたは新しいパーティションを作成した場合は必要ありません)。

  ユーザードキュメント: [MPP モードでのパーティションテーブルへのアクセス](/tiflash/use-tiflash-mpp-mode.md#access-partitioned-tables-in-the-mpp-mode), [動的プルーニングモード](/partitioned-table.md#dynamic-pruning-mode), [#3873](https://github.com/pingcap/tiflash/issues/3873)

### 安定性 {#stability}

- SSTの破損からの自動リカバリ

  RocksDBがバックグラウンドで破損したSSTファイルを検出すると、TiKVは影響を受けるPeerをスケジュールし、他のレプリカを使用してデータを回復しようとします。`background-error-recovery-window`パラメータを使用して、リカバリの最大許容時間を設定できます。リカバリ操作が時間枠内に完了しない場合、TiKVはパニックに陥ります。この機能により、自動的に検出および回復可能な破損したストレージを回復するため、クラスタの安定性が向上します。

  [ユーザードキュメント](/tikv-configuration-file.md#background-error-recovery-window-new-in-v610), [#10578](https://github.com/tikv/tikv/issues/10578)

- 非トランザクションDMLステートメントのサポート

  大規模なデータ処理のシナリオでは、大規模なトランザクションを持つ単一のSQLステートメントは、クラスタの安定性とパフォーマンスに悪影響を与える可能性があります。v6.1.0以降、TiDBは`DELETE`ステートメントを複数のステートメントに分割してバッチ処理する構文をサポートしています。分割されたステートメントはトランザクションの原子性と分離性を犠牲にしますが、クラスタの安定性を大幅に向上させます。詳細な構文については、[`BATCH`](/sql-statements/sql-statement-batch.md)を参照してください。

  [ユーザードキュメント](/non-transactional-dml.md)

- TiDBは最大GC待ち時間の設定をサポート

  TiDBのトランザクションはMulti-Version Concurrency Control（MVCC）メカニズムを採用しています。新しく書き込まれたデータが古いデータを上書きすると、古いデータは置き換えられず、両方のバージョンのデータが保存されます。古いデータはガベージコレクション（GC）タスクによって定期的にクリーンアップされ、これによりストレージスペースを回収してクラスタのパフォーマンスと安定性を向上させます。GCはデフォルトで10分ごとにトリガーされます。長時間実行されるトランザクションが対応する履歴データにアクセスできるようにするために、実行中のトランザクションがある場合、GCタスクは遅延されます。GCタスクが無期限に遅延されないようにするために、TiDBはシステム変数[`tidb_gc_max_wait_time`](/system-variables.md#tidb_gc_max_wait_time-new-in-v610)を導入して、GCタスクの最大遅延時間を制御します。変数のデフォルト値は24時間です。この機能により、GC待ち時間と長時間実行されるトランザクションとの関係を制御でき、クラスタの安定性が向上します。

  [ユーザードキュメント](/system-variables.md#tidb_gc_max_wait_time-new-in-v610)

- TiDBは自動統計情報収集タスクの最大実行時間の設定をサポート

  データベースは統計情報を収集することで、データの分布を効果的に理解し、合理的な実行計画を生成し、SQL実行の効率を向上させることができます。TiDBは定期的にバックグラウンドで頻繁に変更されるデータオブジェクトの統計情報を収集します。ただし、統計情報の収集はクラスタのリソースを消費し、ビジネスピーク時に安定した運用に影響を与える可能性があります。

  v6.1.0から、TiDBはバックグラウンド統計情報収集の最大実行時間を制御するために[`tidb_max_auto_analyze_time`](/system-variables.md#tidb_max_auto_analyze_time-new-in-v610)を導入しました。デフォルトでは、最大実行時間は12時間です。アプリケーションがリソースボトルネックに遭遇しない場合は、この変数を変更しないことをお勧めします。

  [ユーザードキュメント](/system-variables.md)

### 利便性 {#ease-of-use}

- 複数のレプリカが失われた場合のワンストップオンラインデータリカバリのサポート

  TiDB v6.1.0以前では、マシンの障害により複数のリージョンレプリカが失われた場合、ユーザーはすべてのTiKVサーバーを停止し、TiKV Controlを使用してTiKVを1つずつ回復する必要がありました。TiDB v6.1.0以降、リカバリプロセスは完全に自動化され、TiKVを停止する必要はなく、他のアプリケーションに影響を与えません。リカバリプロセスはPD Controlを使用してトリガーでき、よりユーザーフレンドリーな概要情報を提供します。

  [ユーザードキュメント](/online-unsafe-recovery.md), [#10483](https://github.com/tikv/tikv/issues/10483)

- 履歴統計情報収集タスクの表示のサポート

  `SHOW ANALYZE STATUS`ステートメントを使用してクラスタレベルの統計情報収集タスクを表示できます。TiDB v6.1.0以前では、`SHOW ANALYZE STATUS`ステートメントはインスタンスレベルのタスクのみを表示し、TiDBの再起動後に履歴タスクレコードがクリアされます。そのため、履歴統計情報収集の時間と詳細を表示できませんでした。TiDB v6.1.0からは、統計情報収集タスクの履歴レコードが永続化され、クラスタの再起動後にクエリパフォーマンスの問題のトラブルシューティングの参照情報を提供します。

  [ユーザードキュメント](/sql-statements/sql-statement-show-analyze-status.md)

- TiDB、TiKV、およびTiFlashの設定の動的変更のサポート

  以前のTiDBバージョンでは、構成項目を変更した後、変更を有効にするためにクラスタを再起動する必要がありました。これによりオンラインサービスが中断される可能性があります。この問題に対処するために、TiDB v6.1.0では動的構成機能が導入され、クラスタを再起動せずにパラメータの変更を検証できるようになりました。具体的な最適化は次のとおりです。

  - 一部のTiDB構成項目をシステム変数に変換し、動的に変更および永続化できるようにします。なお、変換後、元の構成項目は非推奨となります。変換された構成項目の詳細なリストについては、[Configuration file parameters](#configuration-file-parameters)を参照してください。
  - 一部のTiKVパラメータのオンライン設定をサポートします。パラメータの詳細なリストについては、[Others](#others)を参照してください。
  - TiFlash構成項目`max_threads`をシステム変数`tidb_max_tiflash_threads`に変換し、構成を動的に変更および永続化できるようにします。なお、変換後も元の構成項目は残ります。

  以前のバージョンからv6.1.0にアップグレードされたクラスタ（オンラインおよびオフラインのアップグレードを含む）の場合は、次の点に注意してください。

  - アップグレード前の構成ファイルで指定された構成項目がすでに存在する場合、TiDBはアップグレードプロセス中に自動的に構成された項目の値を対応するシステム変数の値に更新します。これにより、アップグレード後、システムの動作はパラメータの最適化に影響されません。
  - 上記の自動更新はアップグレード中に1度だけ行われます。アップグレード後、非推奨の構成項目はもはや有効ではありません。

  この機能により、パラメータを動的に変更し、検証および永続化できるようになり、システムを再起動してサービスを中断する必要がなくなります。これにより、日常のメンテナンスが容易になります。

  [ユーザードキュメント](/dynamic-config.md)

- グローバルなクエリまたは接続の終了のサポート

  `enable-global-kill`構成を使用してグローバルキル機能を制御できます（デフォルトで有効）。

  TiDB v6.1.0以前では、操作が多くのリソースを消費し、クラスタの安定性に問題を引き起こす場合、ターゲットのTiDBインスタンスに接続して`KILL TIDB ${id};`コマンドを実行してターゲットの接続と操作を終了する必要がありました。多くのTiDBインスタンスの場合、この方法は使用しにくく、誤った操作を行いやすいです。v6.1.0から、`enable-global-kill`構成が導入され、デフォルトで有効になりました。この構成を使用して、クライアントとTiDBの間にプロキシがある場合でも、誤って他のクエリやセッションを誤って終了させる心配をせずに、任意のTiDBインスタンスでキルコマンドを実行して指定された接続と操作を終了できます。現在、TiDBはCtrl+Cを使用してクエリやセッションを終了することはサポートしていません。

  [ユーザードキュメント](/tidb-configuration-file.md#enable-global-kill-new-in-v610), [#8854](https://github.com/pingcap/tidb/issues/8854)

- TiKV API V2（実験的）

  v6.1.0以前、TiKVがRaw Key Valueストレージとして使用される場合、TiKVはクライアントから渡された生データの基本的なKey Valueの読み書き機能のみを提供していました。

  TiKV API V2は、新しいRaw Key Valueストレージフォーマットとアクセスインターフェースを提供します。これには次のものが含まれます。

  - データはMVCCで保存され、データの変更タイムスタンプが記録されます。この機能はChange Data Captureおよび増分バックアップとリストアの実装の基盤を築きます。
  - データは異なる使用に応じてスコープが設定され、単一のTiDBクラスタ、Transactional KV、RawKVアプリケーションの共存をサポートします。

  <Warning>

  基礎ストレージフォーマットの大幅な変更のため、API V2を有効にした後は、TiKVクラスタをv6.1.0より前のバージョンにロールバックすることはできません。TiKVのダウングレードはデータの破損を引き起こす可能性があります。

  </Warning>

  [ユーザードキュメント](/tikv-configuration-file.md#api-version-new-in-v610), [#11745](https://github.com/tikv/tikv/issues/11745)

### MySQL互換性 {#mysql-compatibility}

- MySQLのユーザーレベルロック管理との互換性のサポート

  ユーザーレベルロックは、MySQLが組み込み関数を通じて提供するユーザー名付きロック管理システムです。ロック関数は、ロックのブロック、待機などのロック管理機能を提供できます。ユーザーレベルロックは、Rails、Elixir、EctoなどのORMフレームワークでも広く使用されています。v6.1.0以降、TiDBはMySQL互換のユーザーレベルロック管理をサポートし、`GET_LOCK`、`RELEASE_LOCK`、`RELEASE_ALL_LOCKS`関数をサポートしています。

  [ユーザードキュメント](/functions-and-operators/locking-functions.md), [#14994](https://github.com/pingcap/tidb/issues/14994)

### データ移行 {#data-migration}

- シャーディングされたテーブルのマージと移行の楽観的モードがGAになりました

  DMは、シャーディングされたテーブルからのデータのマージと移行のためのシナリオテストを大量に追加し、これにより、日常的な使用シナリオの90%をカバーしています。悲観的モードと比較して、楽観的モードはよりシンプルで効率的に使用できます。使用上の注意をよく理解した後は、楽観的モードをできるだけ使用することをお勧めします。

  [ユーザードキュメント](/dm/feature-shard-merge-optimistic.md#restrictions)

- DM WebUIは、指定されたパラメータに従ってタスクを開始することをサポートしています

  移行タスクを開始する際に、開始時間と安全モードの期間を指定できます。これは、多くのソースを持つ増分移行タスクを作成する際に特に便利であり、各ソースに特定のbinlog開始位置を指定する必要がなくなります。

  [ユーザードキュメント](/dm/dm-webui-guide.md), [#5442](https://github.com/pingcap/tiflow/issues/5442)

### TiDBデータ共有サブスクリプション {#tidb-data-share-subscription}

- TiDBは、さまざまなサードパーティデータエコシステムとのデータ共有をサポートしています

  - TiCDCは、Avro形式のKafkaにTiDBの増分データを送信することをサポートし、Confluentを介してKSQLやSnowflakeなどのサードパーティとデータを共有できます。

    [ユーザードキュメント](/ticdc/ticdc-avro-protocol.md), [#5338](https://github.com/pingcap/tiflow/issues/5338)

  - TiCDCは、テーブルごとに異なるKafkaトピックに増分データをディスパッチすることをサポートし、Canal-json形式と組み合わせることで、Flinkと直接データを共有できます。

    [ユーザードキュメント](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink), [#4423](https://github.com/pingcap/tiflow/issues/4423)

  - TiCDCは、SASL GSSAPI認証タイプをサポートし、Kafkaを使用したSASL認証の例を追加しました。

    [ユーザードキュメント](/ticdc/ticdc-sink-to-kafka.md#ticdc-uses-the-authentication-and-authorization-of-kafka), [#4423](https://github.com/pingcap/tiflow/issues/4423)

- TiCDCは、`charset=GBK`テーブルのレプリケーションをサポートしています。

  [ユーザードキュメント](/character-set-gbk.md#component-compatibility), [#4806](https://github.com/pingcap/tiflow/issues/4806)

## 互換性の変更 {#compatibility-changes}

### システム変数 {#system-variables}

| 変数名                                                                                                                           | 変更タイプ | 説明                                                                                                                                                       |
| ----------------------------------------------------------------------------------------------------------------------------- | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`tidb_enable_list_partition`](/system-variables.md#tidb_enable_list_partition-new-in-v50)                                    | 変更    | デフォルト値が`OFF`から`ON`に変更されました。                                                                                                                              |
| [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)                                                           | 変更    | この変数はGLOBALスコープが追加され、変数値がクラスタに永続化されます。                                                                                                                   |
| [`tidb_query_log_max_len`](/system-variables.md#tidb_query_log_max_len)                                                       | 変更    | 変数スコープがINSTANCEからGLOBALに変更されました。変数値がクラスタに永続化され、値の範囲が`[0, 1073741824]`に変更されました。                                                                           |
| [`require_secure_transport`](/system-variables.md#require_secure_transport-new-in-v610)                                       | 新規追加  | この設定は以前は`tidb.toml`オプション（`security.require-secure-transport`）でしたが、TiDB v6.1.0からシステム変数に変更されました。                                                           |
| [`tidb_committer_concurrency`](/system-variables.md#tidb_committer_concurrency-new-in-v610)                                   | 新規追加  | この設定は以前は`tidb.toml`オプション（`performance.committer-concurrency`）でしたが、TiDB v6.1.0からシステム変数に変更されました。                                                           |
| [`tidb_enable_auto_analyze`](/system-variables.md#tidb_enable_auto_analyze-new-in-v610)                                       | 新規追加  | この設定は以前は`tidb.toml`オプション（`run-auto-analyze`）でしたが、TiDB v6.1.0からシステム変数に変更されました。                                                                            |
| [`tidb_enable_new_only_full_group_by_check`](/system-variables.md#tidb_enable_new_only_full_group_by_check-new-in-v610)       | 新規追加  | この変数は、TiDBが`ONLY_FULL_GROUP_BY`チェックを実行する際の動作を制御します。                                                                                                      |
| [`tidb_enable_outer_join_reorder`](/system-variables.md#tidb_enable_outer_join_reorder-new-in-v610)                           | 新規追加  | v6.1.0以降、TiDBのJoin ReorderアルゴリズムはOuter Joinをサポートしています。この変数はサポート動作を制御し、デフォルト値は`ON`です。                                                                     |
| [`tidb_enable_prepared_plan_cache`](/system-variables.md#tidb_enable_prepared_plan_cache-new-in-v610)                         | 新規追加  | この設定は以前は`tidb.toml`オプション（`prepared-plan-cache.enabled`）でしたが、TiDB v6.1.0からシステム変数に変更されました。                                                                 |
| [`tidb_gc_max_wait_time`](/system-variables.md#tidb_gc_max_wait_time-new-in-v610)                                             | 新規追加  | この変数は、GCセーフポイントが未コミットトランザクションによってブロックされる最大時間を設定するために使用されます。                                                                                              |
| [tidb\_max\_auto\_analyze\_time](/system-variables.md#tidb_max_auto_analyze_time-new-in-v610)                                 | 新規追加  | この変数は、自動解析の最大実行時間を指定するために使用されます。                                                                                                                         |
| [`tidb_max_tiflash_threads`](/system-variables.md#tidb_max_tiflash_threads-new-in-v610)                                       | 新規追加  | この変数は、TiFlashがリクエストを実行するための最大並行性を設定するために使用されます。                                                                                                          |
| [`tidb_mem_oom_action`](/system-variables.md#tidb_mem_oom_action-new-in-v610)                                                 | 新規追加  | この設定は以前は`tidb.toml`オプション（`oom-action`）でしたが、TiDB v6.1.0からシステム変数に変更されました。                                                                                  |
| [`tidb_mem_quota_analyze`](/system-variables.md#tidb_mem_quota_analyze-new-in-v610)                                           | 新規追加  | この変数は、TiDBが統計情報を更新する際の最大メモリ使用量を制御します。これには、ユーザによって手動で実行される[`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)と、TiDBバックグラウンドでの自動解析タスクが含まれます。 |
| [`tidb_nontransactional_ignore_error`](/system-variables.md#tidb_nontransactional_ignore_error-new-in-v610)                   | 新規追加  | この変数は、非トランザクショナルDMLステートメントでエラーが発生した場合にエラーを即座に返すかどうかを指定します。                                                                                               |
| [`tidb_prepared_plan_cache_memory_guard_ratio`](/system-variables.md#tidb_prepared_plan_cache_memory_guard_ratio-new-in-v610) | 新規追加  | この設定は以前は`tidb.toml`オプション（`prepared-plan-cache.memory-guard-ratio`）でしたが、TiDB v6.1.0からシステム変数に変更されました。                                                      |
| [`tidb_prepared_plan_cache_size`](/system-variables.md#tidb_prepared_plan_cache_size-new-in-v610)                             | 新規追加  | この設定は以前は`tidb.toml`オプション（`prepared-plan-cache.capacity`）でしたが、TiDB v6.1.0からシステム変数に変更されました。                                                                |
| [`tidb_stats_cache_mem_quota`](/system-variables.md#tidb_stats_cache_mem_quota-new-in-v610)                                   | 新規追加  | この変数は、TiDB統計情報キャッシュのメモリクォータを設定します。                                                                                                                       |

### Configuration file parameters {#configuration-file-parameters}

| Configuration file | Configuration                                                                                                                                                                                          | Change type | Description                                                                                                                                         |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| TiDB               | `committer-concurrency`                                                                                                                                                                                | 削除          | システム変数 `tidb_committer_concurrency` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は、対応するシステム変数を変更する必要があります。                                                   |
| TiDB               | `lower-case-table-names`                                                                                                                                                                               | 削除          | 現在、TiDBは `lower_case_table_name=2` のみをサポートしています。別の値が設定されている場合、クラスタがv6.1.0にアップグレードされた後、その値は失われます。                                                    |
| TiDB               | `mem-quota-query`                                                                                                                                                                                      | 削除          | システム変数 `tidb_mem_quota_query` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は、対応するシステム変数を変更する必要があります。                                                         |
| TiDB               | `oom-action`                                                                                                                                                                                           | 削除          | システム変数 `tidb_mem_oom_action` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は、対応するシステム変数を変更する必要があります。                                                          |
| TiDB               | `prepared-plan-cache.capacity`                                                                                                                                                                         | 削除          | システム変数 `tidb_prepared_plan_cache_size` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は、対応するシステム変数を変更する必要があります。                                                |
| TiDB               | `prepared-plan-cache.enabled`                                                                                                                                                                          | 削除          | システム変数 `tidb_enable_prepared_plan_cache` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は、対応するシステム変数を変更する必要があります。                                              |
| TiDB               | `query-log-max-len`                                                                                                                                                                                    | 削除          | システム変数 `tidb_query_log_max_len` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は、対志するシステム変数を変更する必要があります。                                                       |
| TiDB               | `require-secure-transport`                                                                                                                                                                             | 削除          | システム変数 `require_secure_transport` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は、対応するシステム変数を変更する必要があります。                                                     |
| TiDB               | `run-auto-analyze`                                                                                                                                                                                     | 削除          | システム変数 `tidb_enable_auto_analyze` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は、対応するシステム変数を変更する必要があります。                                                     |
| TiDB               | [`enable-global-kill`](/tidb-configuration-file.md#enable-global-kill-new-in-v610)                                                                                                                     | 新規追加        | グローバルキル（インスタンス間のクエリまたは接続の終了）機能を有効にするかどうかを制御します。値が `true` の場合、`KILL` および `KILL TIDB` ステートメントの両方がクエリまたはインスタンス間の接続を終了できるため、誤ってクエリまたは接続を終了する心配はありません。   |
| TiDB               | [`enable-stats-cache-mem-quota`](/tidb-configuration-file.md#enable-stats-cache-mem-quota-new-in-v610)                                                                                                 | 新規追加        | 統計キャッシュのメモリクォータを有効にするかどうかを制御します。                                                                                                                    |
| TiKV               | [`raft-engine.enable`](/tikv-configuration-file.md#enable-1)                                                                                                                                           | 変更          | デフォルト値が `FALSE` から `TRUE` に変更されました。                                                                                                                 |
| TiKV               | [`region-max-keys`](/tikv-configuration-file.md#region-max-keys)                                                                                                                                       | 変更          | デフォルト値が1440000から `region-split-keys / 2 * 3` に変更されました。                                                                                              |
| TiKV               | [`region-max-size`](/tikv-configuration-file.md#region-max-size)                                                                                                                                       | 変更          | デフォルト値が144 MBから `region-split-size / 2 * 3` に変更されました。                                                                                               |
| TiKV               | [`coprocessor.enable-region-bucket`](/tikv-configuration-file.md#enable-region-bucket-new-in-v610)                                                                                                     | 新規追加        | Regionをバケットと呼ばれるより小さな範囲に分割するかどうかを決定します。                                                                                                             |
| TiKV               | [`coprocessor.region-bucket-size`](/tikv-configuration-file.md#region-bucket-size-new-in-v610)                                                                                                         | 新規追加        | `enable-region-bucket` がtrueの場合のバケットのサイズです。                                                                                                         |
| TiKV               | [`causal-ts.renew-batch-min-size`](/tikv-configuration-file.md#renew-batch-min-size)                                                                                                                   | 新規追加        | ローカルにキャッシュされたタイムスタンプの最小数です。                                                                                                                         |
| TiKV               | [`causal-ts.renew-interval`](/tikv-configuration-file.md#renew-interval)                                                                                                                               | 新規追加        | ローカルにキャッシュされたタイムスタンプが更新される間隔です。                                                                                                                     |
| TiKV               | [`max-snapshot-file-raw-size`](/tikv-configuration-file.md#max-snapshot-file-raw-size-new-in-v610)                                                                                                     | 新規追加        | スナップショットファイルのサイズがこの値を超えると、複数のファイルに分割されます。                                                                                                           |
| TiKV               | [`raft-engine.memory-limit`](/tikv-configuration-file.md#memory-limit)                                                                                                                                 | 新規追加        | Raft Engineのメモリ使用量の制限を指定します。                                                                                                                        |
| TiKV               | [`storage.background-error-recovery-window`](/tikv-configuration-file.md#background-error-recovery-window-new-in-v610)                                                                                 | 新規追加        | RocksDBが回復可能なバックグラウンドエラーを検出した後に許可される最大回復時間です。                                                                                                       |
| TiKV               | [`storage.api-version`](/tikv-configuration-file.md#api-version-new-in-v610)                                                                                                                           | 新規追加        | TiKVが生のキー値ストアとして機能する場合に使用されるストレージフォーマットおよびインターフェースバージョンを指定します。                                                                                      |
| PD                 | [`schedule.max-store-preparing-time`](/pd-configuration-file.md#max-store-preparing-time-new-in-v610)                                                                                                  | 新規追加        | ストアがオンラインになるまでの最大待機時間を制御します。                                                                                                                        |
| TiCDC              | [`enable-tls`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka)                                                                                                                             | 新規追加        | ダウンストリームのKafkaインスタンスに接続する際にTLSを使用するかどうかを指定します。                                                                                                      |
| TiCDC              | `sasl-gssapi-user`<br/>`sasl-gssapi-password`<br/>`sasl-gssapi-auth-type`<br/>`sasl-gssapi-service-name`<br/>`sasl-gssapi-realm`<br/>`sasl-gssapi-key-tab-path`<br/>`sasl-gssapi-kerberos-config-path` | 新規追加        | KafkaのSASL/GSSAPI認証をサポートするために使用されます。詳細については、[Configure sink URI with `kafka`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka)を参照してください。 |
| TiCDC              | [`avro-decimal-handling-mode`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka)<br/>[`avro-bigint-unsigned-handling-mode`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka)      | 新規追加        | Avroフォーマットの出力詳細を決定します。                                                                                                                              |
| TiCDC              | [`dispatchers.topic`](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink)                                                                             | 新規追加        | TiCDCがインクリメンタルデータを異なるKafkaトピックにどのようにディスパッチするかを制御します。                                                                                                |
| TiCDC              | [`dispatchers.partition`](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink)                                                                         | 新規追加        | `dispatchers.partition` は `dispatchers.dispatcher` のエイリアスです。TiCDCがインクリメンタルデータをKafkaパーティションにどのようにディスパッチするかを制御します。                                    |
| TiCDC              | [`schema-registry`](/ticdc/ticdc-sink-to-kafka.md#integrate-ticdc-with-kafka-connect-confluent-platform)                                                                                               | 新規追加        | Avroスキーマを保存するスキーマレジストリのエンドポイントを指定します。                                                                                                               |
| DM                 | `dmctl start-relay` コマンドの `worker`                                                                                                                                                                     | 削除          | このパラメータは使用をお勧めしません。より簡単な実装を提供します。                                                                                                                   |
| DM                 | ソース構成ファイルの `relay-dir`                                                                                                                                                                                 | 削除          | ワーカー構成ファイル内の同じ構成項目に置き換えられました。                                                                                                                       |
| DM                 | タスク構成ファイルの `is-sharding`                                                                                                                                                                               | 削除          | `shard-mode` 構成項目に置き換えられました。                                                                                                                        |
| DM                 | タスク構成ファイルの `auto-fix-gtid`                                                                                                                                                                             | 削除          | v5.xで非推奨とされ、v6.1.0で正式に削除されました。                                                                                                                      |
| DM                 | ソース構成ファイルの `meta-dir` および `charset`                                                                                                                                                                    | 削除          | v5.xで非推奨とされ、v6.1.0で正式に削除されました。                                                                                                                      |

### その他 {#others}

- デフォルトで Prepared Plan Cache を有効にする

  新しいクラスタでは、`Prepare` / `Execute` リクエストの実行計画をキャッシュするために Prepared Plan Cache がデフォルトで有効になります。後続の実行では、クエリプランの最適化をスキップできるため、パフォーマンスが向上します。アップグレードされたクラスタは、構成ファイルから構成を継承します。新しいクラスタでは、新しいデフォルト値が使用されます。つまり、Prepared Plan Cache がデフォルトで有効になり、各セッションで最大100のプランをキャッシュできます（`capacity=100`）。この機能のメモリ消費については、[Prepared Plan Cache のメモリ管理](/sql-prepared-plan-cache.md#memory-management-of-prepared-plan-cache)を参照してください。

- TiDB v6.1.0 より前では、`SHOW ANALYZE STATUS` はインスタンスレベルのタスクを表示し、タスクレコードは TiDB の再起動後にクリアされます。TiDB v6.1.0 以降、`SHOW ANALYZE STATUS` はクラスタレベルのタスクを表示し、タスクレコードは再起動後も保持されます。`tidb_analyze_version = 2` の場合、`Job_info` 列に `analyze option` 情報が追加されます。

- TiKV の損傷した SST ファイルは、TiKV プロセスがパニックを引き起こす可能性があります。TiDB v6.1.0 より前では、損傷した SST ファイルは、TiKV が直ちにパニックを引き起こしました。TiDB v6.1.0 以降、SST ファイルが損傷してから1時間後に TiKV プロセスがパニックを引き起こします。

- 次の TiKV 構成項目は、[値を動的に変更](/dynamic-config.md#modify-tikv-configuration-dynamically)することができます。

  - `raftstore.raft-entry-max-size`
  - `quota.foreground-cpu-time`
  - `quota.foreground-write-bandwidth`
  - `quota.foreground-read-bandwidth`
  - `quota.max-delay-duration`
  - `server.grpc-memory-pool-quota`
  - `server.max-grpc-send-msg-len`
  - `server.raft-msg-max-batch-size`

- v6.1.0 では、一部の構成ファイルパラメータがシステム変数に変換されます。以前のバージョンからアップグレードされた v6.1.0 クラスタ（オンラインおよびオフラインのアップグレードを含む）の場合は、次の点に注意してください。

  - アップグレード前に構成ファイルで指定された構成項目がすでに存在する場合、TiDB はアップグレードプロセス中に構成された項目の値を対応するシステム変数の値に自動的に更新します。このようにして、アップグレード後にパラメータの最適化によりシステムの動作が変わらないようにします。
  - 上記の自動更新は、アップグレード中に1度だけ行われます。アップグレード後、非推奨の構成項目はもはや有効ではありません。

- DM WebUI からダッシュボードページが削除されました。

- `dispatchers.topic` および `dispatchers.partition` が有効になっている場合、TiCDC を v6.1.0 より前のバージョンにダウングレードすることはできません。

- Avro プロトコルを使用する TiCDC Changefeed は、v6.1.0 より前のバージョンにダウングレードすることはできません。

## 改善点 {#improvements}

- TiDB

  - `UnionScanRead` オペレータのパフォーマンスを向上させる [#32433](https://github.com/pingcap/tidb/issues/32433)
  - `EXPLAIN` の出力でタスクタイプの表示を改善する（MPP タスクタイプを追加） [#33332](https://github.com/pingcap/tidb/issues/33332)
  - 列のデフォルト値として `rand()` を使用することをサポートする [#10377](https://github.com/pingcap/tidb/issues/10377)
  - 列のデフォルト値として `uuid()` を使用することをサポートする [#33870](https://github.com/pingcap/tidb/issues/33870)
  - 列の文字セットを `latin1` から `utf8`/`utf8mb4` に変更することをサポートする [#34008](https://github.com/pingcap/tidb/issues/34008)

- TiKV

  - インメモリ悲観的ロックを使用する場合の CDC の古い値のヒット率を向上させる [#12279](https://github.com/tikv/tikv/issues/12279)
  - 利用できない Raftstore を検出するためのヘルスチェックを改善し、TiKV クライアントがリージョンキャッシュをタイムリーに更新できるようにする [#12398](https://github.com/tikv/tikv/issues/12398)
  - Raft Engine でメモリ制限を設定することをサポートする [#12255](https://github.com/tikv/tikv/issues/12255)
  - TiKV は損傷した SST ファイルを自動的に検出および削除して、製品の可用性を向上させる [#10578](https://github.com/tikv/tikv/issues/10578)
  - CDC が RawKV をサポートする [#11965](https://github.com/tikv/tikv/issues/11965)
  - 大きなスナップショットファイルを複数のファイルに分割することをサポートする [#11595](https://github.com/tikv/tikv/issues/11595)
  - スナップショットのガベージコレクションを Raftstore からバックグラウンドスレッドに移動して、Raftstore メッセージループがブロックされるのを防ぐ [#11966](https://github.com/tikv/tikv/issues/11966)
  - 最大メッセージ長（`max-grpc-send-msg-len`）と gPRC メッセージの最大バッチサイズ（`raft-msg-max-batch-size`）の動的設定をサポートする [#12334](https://github.com/tikv/tikv/issues/12334)
  - Raft を介したオンラインの安全でないリカバリプランの実行をサポートする [#10483](https://github.com/tikv/tikv/issues/10483)

- PD
  - リージョンラベルのための TTL（有効期限）をサポートする [#4694](https://github.com/tikv/pd/issues/4694)
  - リージョンバケットをサポートする [#4668](https://github.com/tikv/pd/issues/4668)
  - デフォルトで Swagger サーバーのコンパイルを無効にする [#4932](https://github.com/tikv/pd/issues/4932)

- TiFlash

  - 集約演算子のメモリ計算を最適化し、マージフェーズでより効率的なアルゴリズムを使用する [#4451](https://github.com/pingcap/tiflash/issues/4451)

- ツール

  - バックアップ＆リストア（BR）

    - 空のデータベースのバックアップとリストアをサポートする [#33866](https://github.com/pingcap/tidb/issues/33866)

  - TiDB Lightning

    - スキャッタリングリージョンプロセスの安定性を向上させるために、スキャッタリングリージョンをバッチモードに最適化する [#33618](https://github.com/pingcap/tidb/issues/33618)

  - TiCDC

    - レプリケーション中に大きなトランザクションを分割することをサポートし、大きなトランザクションによるレプリケーションのレイテンシーを大幅に低減する [#5280](https://github.com/pingcap/tiflow/issues/5280)

## バグ修正 {#bug-fixes}

- TiDB

  - `in` 関数が `bit` 型のデータを処理する際に発生する可能性のある panic の問題を修正 [#33070](https://github.com/pingcap/tidb/issues/33070)
  - `UnionScan` オペレーターが順序を維持できないため、クエリ結果が間違っている問題を修正 [#33175](https://github.com/pingcap/tidb/issues/33175)
  - 特定のケースで Merge Join オペレーターが誤った結果を取得する問題を修正 [#33042](https://github.com/pingcap/tidb/issues/33042)
  - 動的プルーニングモードで `index join` の結果が誤る可能性がある問題を修正 [#33231](https://github.com/pingcap/tidb/issues/33231)
  - パーティションテーブルの一部のパーティションが削除されたときにデータがガベージコレクションされない可能性がある問題を修正 [#33620](https://github.com/pingcap/tidb/issues/33620)
  - クラスターの PD ノードが置き換えられた後、一部の DDL ステートメントが一定期間スタックする可能性がある問題を修正 [#33908](https://github.com/pingcap/tidb/issues/33908)
  - `INFORMATION_SCHEMA.CLUSTER_SLOW_QUERY` テーブルがクエリされたときに TiDB サーバーがメモリ不足になる可能性がある問題を修正。Grafana ダッシュボードで遅いクエリをチェックするとこの問題が発生する可能性があります [#33893](https://github.com/pingcap/tidb/issues/33893)
  - システム変数 `max_allowed_packet` が効果を発揮しない問題を修正 [#31422](https://github.com/pingcap/tidb/issues/31422)
  - TopSQL モジュールでのメモリリークの問題を修正 [#34525](https://github.com/pingcap/tidb/issues/34525) [#34502](https://github.com/pingcap/tidb/issues/34502)
  - PointGet プランで Plan Cache が誤る可能性がある問題を修正 [#32371](https://github.com/pingcap/tidb/issues/32371)
  - RC 分離レベルで Plan Cache が開始されたときにクエリ結果が誤る可能性がある問題を修正 [#34447](https://github.com/pingcap/tidb/issues/34447)

- TiKV

  - TiKV インスタンスがオフラインになると Raft ログの遅延が増加する問題を修正 [#12161](https://github.com/tikv/tikv/issues/12161)
  - マージするターゲットリージョンが無効であるため、TiKV が予期せず panic しピアを破壊する問題を修正 [#12232](https://github.com/tikv/tikv/issues/12232)
  - v5.3.1 または v5.4.0 から v6.0.0 以降のバージョンにアップグレードする際に TiKV が `failed to load_latest_options` エラーを報告する問題を修正 [#12269](https://github.com/tikv/tikv/issues/12269)
  - メモリリソースが不足しているときに Raft ログを追加することによって OOM が発生する問題を修正 [#11379](https://github.com/tikv/tikv/issues/11379)
  - ピアを破壊するときとバッチ分割リージョンの間の競合によって TiKV が panic する問題を修正 [#12368](https://github.com/tikv/tikv/issues/12368)
  - `stats_monitor` がデッドループに陥ると TiKV のメモリ使用量が急増する問題を修正 [#12416](https://github.com/tikv/tikv/issues/12416)
  - Follower Read を使用すると TiKV が `invalid store ID 0` エラーを報告する問題を修正 [#12478](https://github.com/tikv/tikv/issues/12478)

- PD

  - `not leader` の間違ったステータスコードを修正 [#4797](https://github.com/tikv/pd/issues/4797)
  - 一部の特殊なケースで TSO フォールバックのバグを修正 [#4884](https://github.com/tikv/pd/issues/4884)
  - PD リーダーの移行後に削除された墓石ストアが再び表示される問題を修正 [#4941](https://github.com/tikv/pd/issues/4941)
  - PD リーダーの移行後にスケジューリングをすぐに開始できない問題を修正 [#4769](https://github.com/tikv/pd/issues/4769)

- TiDB Dashboard

  - Top SQL 機能が有効になる前に実行されていた SQL ステートメントの CPU オーバーヘッドを収集できないバグを修正 [#33859](https://github.com/pingcap/tidb/issues/33859)

- TiFlash

  - 大量の INSERT および DELETE 操作の後にデータの不整合が発生する可能性がある問題を修正 [#4956](https://github.com/pingcap/tiflash/issues/4956)

- Tools

  - TiCDC

    - DDL スキーマをバッファリングする方法を最適化することで過剰なメモリ使用量を修正 [#1386](https://github.com/pingcap/tiflow/issues/1386)
    - 特別な増分スキャンシナリオで発生するデータ損失を修正 [#5468](https://github.com/pingcap/tiflow/issues/5468)

  - TiDB データ移行 (DM)

    - `start-time` のタイムゾーンの問題を修正し、DM の動作を下流のタイムゾーンから上流のタイムゾーンに変更 [#5271](https://github.com/pingcap/tiflow/issues/5471)
    - タスクが自動的に再開された後に DM がより多くのディスクスペースを占有する問題を修正 [#3734](https://github.com/pingcap/tiflow/issues/3734) [#5344](https://github.com/pingcap/tiflow/issues/5344)
    - チェックポイントのフラッシュによって失敗した行のデータがスキップされる問題を修正 [#5279](https://github.com/pingcap/tiflow/issues/5279)
    - 下流でフィルタリングされた DDL を手動で実行するとタスクの再開に失敗する可能性がある問題を修正 [#5272](https://github.com/pingcap/tiflow/issues/5272)
    - `case-sensitive: true` が設定されていない場合に大文字のテーブルがレプリケートされない問題を修正 [#5255](https://github.com/pingcap/tiflow/issues/5255)
    - `SHOW CREATE TABLE` ステートメントで返されるインデックスでプライマリキーが最初でない場合に DM ワーカーが panic する問題を修正 [#5159](https://github.com/pingcap/tiflow/issues/5159)
    - GTID が有効になっている場合やタスクが自動的に再開される場合に CPU 使用率が増加し、大量のログが出力される問題を修正 [#5063](https://github.com/pingcap/tiflow/issues/5063)
    - DM WebUI でのオフラインオプションおよびその他の使用上の問題を修正 [#4993](https://github.com/pingcap/tiflow/issues/4993)
    - 上流で GTID が空の場合に増分タスクが開始できない問題を修正 [#3731](https://github.com/pingcap/tiflow/issues/3731)
    - 空の構成が dm-master を panic させる可能性がある問題を修正 [#3732](https://github.com/pingcap/tiflow/issues/3732)

  - TiDB Lightning

    - ローカルディスクリソースとクラスターの可用性をチェックしない問題を修正 [#34213](https://github.com/pingcap/tidb/issues/34213)
    - スキーマのルーティングが正しくない問題を修正 [#33381](https://github.com/pingcap/tidb/issues/33381)
    - TiDB Lightning が panic したときに PD 構成が正しく復元されない問題を修正 [#31733](https://github.com/pingcap/tidb/issues/31733)
    - `auto_increment` 列の範囲外データによってローカルバックエンドのインポートが失敗する問題を修正 [#29737](https://github.com/pingcap/tidb/issues/27937)
    - `auto_random` または `auto_increment` 列が null の場合にローカルバックエンドのインポートが失敗する問題を修正 [#34208](https://github.com/pingcap/tidb/issues/34208)
