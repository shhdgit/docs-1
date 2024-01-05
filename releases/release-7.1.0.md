---
title: TiDB 7.1.0 Release Notes
summary: Learn about the new features, compatibility changes, improvements, and bug fixes in TiDB 7.1.0.
---

# TiDB 7.1.0 リリースノート {#tidb-7-1-0-release-notes}

リリース日: 2023年5月31日

TiDB バージョン: 7.1.0

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v7.1/quick-start-with-tidb) | [本番展開](https://docs.pingcap.com/tidb/v7.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v7.1.0#version-list)

TiDB 7.1.0 はロングタームサポートリリース（LTS）です。

前の LTS 6.5.0 と比較して、7.1.0 には [6.6.0-DMR](/releases/release-6.6.0.md)、[7.0.0-DMR](/releases/release-7.0.0.md) でリリースされた新機能、改善、バグ修正が含まれています。さらに、以下の主な機能と改善が導入されています:

<table>
<thead>
  <tr>
    <th>カテゴリ</th>
    <th>機能</th>
    <th>説明</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="4">拡張性とパフォーマンス</td>
    <td>TiFlash は<a href="https://docs.pingcap.com/tidb/v7.1/tiflash-disaggregated-and-s3" target="_blank">分離されたストレージとコンピュートアーキテクチャおよび S3 共有ストレージ</a>をサポート（実験的、v7.0.0 で導入）</td>
    <td>TiFlash はクラウドネイティブアーキテクチャをオプションとして導入します:
      <ul>
        <li>TiFlash のコンピュートとストレージを分離し、弾力的な HTAP リソース利用のマイルストーンとなります。</li>
        <li>低コストで共有ストレージを提供する S3 ベースのストレージエンジンを導入します。</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>TiKV は<a href="https://docs.pingcap.com/tidb/v7.1/system-variables#tidb_store_batch_size" target="_blank">データリクエストのバッチ集約をサポート</a>（v6.6.0 で導入）</td>
    <td>この強化により、TiKV のバッチ取得操作の総 RPC 数が大幅に減少します。データが高度に分散しており gRPC スレッドプールのリソースが不足している状況では、コプロセッサリクエストのバッチ処理によりパフォーマンスが50%以上向上することがあります。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/troubleshoot-hot-spot-issues#scatter-read-hotspots" target="_blank">負荷ベースのレプリカ読み取り</a></td>
    <td>リードホットスポットのシナリオでは、TiDB はホットスポットの TiKV ノードの読み取りリクエストをそのレプリカにリダイレクトできます。この機能により、リードホットスポットが効率的に分散され、クラスタリソースの利用が最適化されます。負荷ベースのレプリカ読み取りをトリガーする閾値を制御するには、システム変数<a href="https://docs.pingcap.com/tidb/v7.1/system-variables#tidb_load_based_replica_read_threshold-new-in-v700" target="_blank"><code>tidb_load_based_replica_read_threshold</code></a>を調整できます。</td>
  </tr>
  <tr>
      <td>TiKV は<a href="https://docs.pingcap.com/tidb/v7.1/partitioned-raft-kv" target="_blank">パーティション化された Raft KV ストレージエンジン</a>をサポート（実験的）</td>
    <td>TiKV は新しい世代のストレージエンジン、パーティション化された Raft KV を導入します。各データリージョンに専用の RocksDB インスタンスを許可することで、クラスタのストレージ容量を TB レベルから PB レベルに拡張し、より安定した書き込みレイテンシと強力な拡張性を提供できます。</td>
    </tr>
  <tr>
    <td rowspan="2">信頼性と可用性</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/tidb-resource-control" target="_blank">リソースグループによるリソース制御</a>（GA）</td>
   <td>リソース管理をリソースグループに基づいてサポートし、同じクラスタ内の異なるワークロードに対してリソースを割り当て、分離します。この機能により、マルチアプリケーションクラスタの安定性が大幅に向上し、マルチテナンシーの基盤が構築されます。v7.1.0 では、この機能により、実際のワークロードまたはハードウェア展開に基づいてシステム容量を見積もる能力が導入されます。</td>
  </tr>
  <tr>
    <td>TiFlash は<a href="https://docs.pingcap.com/tidb/v7.1/tiflash-spill-disk" target="_blank">ディスクへのスピル</a>をサポート（v7.0.0 で導入）</td>
    <td>TiFlash は中間結果のディスクへのスピルをサポートし、集計、ソート、ハッシュ結合などのデータ集約型操作における OOM を緩和します。</td>
  </tr>
  <tr>
    <td rowspan="3">SQL</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/sql-statement-create-index#multi-valued-indexes" target="_blank">マルチバリューインデックス</a>（GA）</td>
    <td>MySQL 互換のマルチバリューインデックスをサポートし、JSON タイプを強化して MySQL 8.0 との互換性を向上させます。この機能により、マルチバリューカラムのメンバーシップチェックの効率が向上します。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/time-to-live" target="_blank">行レベル TTL</a>（v7.0.0 でGA）</td>
    <td>データベースサイズの管理をサポートし、一定の年齢のデータを自動的に期限切れにすることでパフォーマンスを向上させます。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/generated-columns" target="_blank">生成されたカラム</a>（GA）</td>
    <td>生成されたカラムの値は、カラム定義内の SQL 式によってリアルタイムに計算されます。この機能により、一部のアプリケーションロジックをデータベースレベルに移動し、クエリの効率が向上します。</td>
  </tr>
  <tr>
    <td rowspan="2">セキュリティ</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/security-compatibility-with-mysql" target="_blank">LDAP 認証</a></td>
    <td>TiDB は LDAP 認証をサポートし、<a href="https://dev.mysql.com/doc/refman/8.0/en/ldap-pluggable-authentication.html" target="_blank">MySQL 8.0</a> と互換性があります。</td>
  </tr>
  <tr>
    <td><a href="https://static.pingcap.com/files/2023/09/18204824/TiDB-Database-Auditing-User-Guide1.pdf" target="_blank">監査ログの強化</a>（<a href="https://www.pingcap.com/tidb-enterprise" target="_blank">エンタープライズエディション</a>のみ）</td>
    <td>TiDB エンタープライズエディションは、データベース監査機能を強化します。より細かいイベントフィルタリングコントロール、ユーザーフレンドリーなフィルタ設定、JSON 形式の新しいファイル出力形式、監査ログのライフサイクル管理により、システムの監査容量が大幅に向上します。</td>
  </tr>
</tbody>
</table>

## 機能の詳細 {#feature-details}

### パフォーマンス {#performance}

- Enhance the Partitioned Raft KV storage engine (experimental) [#11515](https://github.com/tikv/tikv/issues/11515) [#12842](https://github.com/tikv/tikv/issues/12842) @[busyjay](https://github.com/busyjay) @[tonyxuqqi](https://github.com/tonyxuqqi) @[tabokie](https://github.com/tabokie) @[bufferflies](https://github.com/bufferflies) @[5kbpers](https://github.com/5kbpers) @[SpadeA-Tang](https://github.com/SpadeA-Tang) @[nolouch](https://github.com/nolouch)

  TiDB v6.6.0 introduces the Partitioned Raft KV storage engine as an experimental feature, which uses multiple RocksDB instances to store TiKV Region data, and the data of each Region is independently stored in a separate RocksDB instance. The new storage engine can better control the number and level of files in the RocksDB instance, achieve physical isolation of data operations between Regions, and support stably managing more data. Compared with the original TiKV storage engine, using the Partitioned Raft KV storage engine can achieve about twice the write throughput and reduce the elastic scaling time by about 4/5 under the same hardware conditions and mixed read and write scenarios.

  In TiDB v7.1.0, the Partitioned Raft KV storage engine supports tools such as TiDB Lightning, BR, and TiCDC.

  Currently, this feature is experimental and not recommended for use in production environments. You can only use this engine in a newly created cluster and you cannot directly upgrade from the original TiKV storage engine.

  For more information, see [documentation](/partitioned-raft-kv.md).

- TiFlash supports late materialization (GA) [#5829](https://github.com/pingcap/tiflash/issues/5829) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)

  In v7.0.0, late materialization was introduced in TiFlash as an experimental feature for optimizing query performance. This feature is disabled by default (the [`tidb_opt_enable_late_materialization`](/system-variables.md#tidb_opt_enable_late_materialization-new-in-v700) system variable defaults to `OFF`). When processing a `SELECT` statement with filter conditions (`WHERE` clause), TiFlash reads all the data from the columns required by the query, and then filters and aggregates the data based on the query conditions. When Late materialization is enabled, TiDB supports pushing down part of the filter conditions to the TableScan operator. That is, TiFlash first scans the column data related to the filter conditions that are pushed down to the TableScan operator, filters the rows that meet the condition, and then scans the other column data of these rows for further calculation, thereby reducing IO scans and computations of data processing.

  Starting from v7.1.0, the TiFlash late materialization feature is generally available and enabled by default (the [`tidb_opt_enable_late_materialization`](/system-variables.md#tidb_opt_enable_late_materialization-new-in-v700) system variable defaults to `ON`). The TiDB optimizer decides which filters to be pushed down to the TableScan operator based on the statistics and the filter conditions of the query.

  For more information, see [documentation](/tiflash/tiflash-late-materialization.md).

- TiFlash supports automatically choosing an MPP Join algorithm according to the overhead of network transmission [#7084](https://github.com/pingcap/tiflash/issues/7084) @[solotzg](https://github.com/solotzg)

  The TiFlash MPP mode supports multiple Join algorithms. Before v7.1.0, TiDB determines whether the MPP mode uses the Broadcast Hash Join algorithm based on the [`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50) and [`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50) variables and the actual data volume.

  In v7.1.0, TiDB introduces the [`tidb_prefer_broadcast_join_by_exchange_data_size`](/system-variables.md#tidb_prefer_broadcast_join_by_exchange_data_size-new-in-v710) variable, which controls whether to choose the MPP Join algorithm based on the minimum overhead of network transmission. This variable is disabled by default, indicating that the default algorithm selection method remains the same as that before v7.1.0. You can set the variable to `ON` to enable it. When it is enabled, you no longer need to manually adjust the [`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50) and [`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50) variables (both variables does not take effect at this time), TiDB automatically estimates the threshold of network transmission by different Join algorithms, and then chooses the algorithm with the smallest overhead overall, thus reducing network traffic and improving MPP query performance.

  For more information, see [documentation](/tiflash/use-tiflash-mpp-mode.md#algorithm-support-for-the-mpp-mode).

- Support load-based replica read to mitigate read hotspots [#14151](https://github.com/tikv/tikv/issues/14151) @[sticnarf](https://github.com/sticnarf) @[you06](https://github.com/you06)

  In a read hotspot scenario, the hotspot TiKV node cannot process read requests in time, resulting in the read requests queuing. However, not all TiKV resources are exhausted at this time. To reduce latency, TiDB v7.1.0 introduces the load-based replica read feature, which allows TiDB to read data from other TiKV nodes without queuing on the hotspot TiKV node. You can control the queue length of read requests using the [`tidb_load_based_replica_read_threshold`](/system-variables.md#tidb_load_based_replica_read_threshold-new-in-v700) system variable. When the estimated queue time of the leader node exceeds this threshold, TiDB prioritizes reading data from follower nodes. This feature can improve read throughput by 70% to 200% in a read hotspot scenario compared to not scattering read hotspots.

  For more information, see [documentation](/troubleshoot-hot-spot-issues.md#scatter-read-hotspots).

- Enhance the capability of caching execution plans for non-prepared statements (experimental) [#36598](https://github.com/pingcap/tidb/issues/36598) @[qw4990](https://github.com/qw4990)

  TiDB v7.0.0 introduces non-prepared plan cache as an experimental feature to improve the load capacity of concurrent OLTP. In v7.1.0, TiDB enhances this feature and supports caching more SQL statements.

  To improve memory utilization, TiDB v7.1.0 merges the cache pools of non-prepared and prepared plan caches. You can control the cache size using the system variable [`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710). The [`tidb_prepared_plan_cache_size`](/system-variables.md#tidb_prepared_plan_cache_size-new-in-v610) and [`tidb_non_prepared_plan_cache_size`](/system-variables.md#tidb_non_prepared_plan_cache_size) system variables are deprecated.

  To maintain forward compatibility, when you upgrade from an earlier version to v7.1.0 or later versions, the cache size `tidb_session_plan_cache_size` remains the same value as `tidb_prepared_plan_cache_size`, and [`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache) remains the setting before the upgrade. After sufficient performance testing, you can enable non-prepared plan cache using `tidb_enable_non_prepared_plan_cache`. For a newly created cluster, non-prepared plan cache is enabled by default.

  Non-prepared plan cache does not support DML statements by default. To remove this restriction, you can set the [`tidb_enable_non_prepared_plan_cache_for_dml`](/system-variables.md#tidb_enable_non_prepared_plan_cache_for_dml-new-in-v710) system variable to `ON`.

  For more information, see [documentation](/sql-non-prepared-plan-cache.md).

- Support the DDL distributed parallel execution framework (experimental) [#41495](https://github.com/pingcap/tidb/issues/41495) @[benjamin2037](https://github.com/benjamin2037)

  Before TiDB v7.1.0, only one TiDB node can serve as the DDL owner and execute DDL tasks at the same time. Starting from TiDB v7.1.0, in the new distributed parallel execution framework, multiple TiDB nodes can execute the same DDL task in parallel, thus better utilizing the resources of the TiDB cluster and significantly improving the performance of DDL. In addition, you can linearly improve the performance of DDL by adding more TiDB nodes. Note that this feature is currently experimental and only supports `ADD INDEX` operations.

  To use the distributed framework, set the value of [`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710) to `ON`:

  ```sql
  SET GLOBAL tidb_enable_dist_task = ON;
  ```

  For more information, see [documentation](/tidb-distributed-execution-framework.md).

### 信頼性 {#reliability}

- リソース制御が一般に利用可能（GA）になりました [#38825](https://github.com/pingcap/tidb/issues/38825) @[nolouch](https://github.com/nolouch) @[BornChanger](https://github.com/BornChanger) @[glorv](https://github.com/glorv) @[tiancaiamao](https://github.com/tiancaiamao) @[Connor1996](https://github.com/Connor1996) @[JmPotato](https://github.com/JmPotato) @[hnes](https://github.com/hnes) @[CabinfeverB](https://github.com/CabinfeverB) @[HuSharp](https://github.com/HuSharp)

  TiDBは、リソースグループに基づいたリソース制御機能を強化し、v7.1.0でGAになりました。この機能により、TiDBクラスターのリソース利用効率とパフォーマンスが大幅に向上します。リソース制御機能の導入は、TiDBの画期的な進歩です。分散データベースクラスターを複数の論理ユニットに分割し、異なるデータベースユーザーを対応するリソースグループにマッピングし、必要に応じて各リソースグループのクォータを設定できます。クラスターリソースが限られている場合、同じリソースグループのセッションで使用されるすべてのリソースはクォータに制限されます。このように、リソースグループが過剰に消費されても、他のリソースグループのセッションに影響を与えません。

  この機能により、異なるシステムからの複数の小規模および中規模のアプリケーションを単一のTiDBクラスターに組み合わせることができます。アプリケーションのワークロードが大きくなっても、他のアプリケーションの正常な動作に影響を与えません。システムのワークロードが低い場合、クォータを超えてもビジーなアプリケーションに必要なシステムリソースを割り当てることができ、リソースの最大利用が可能です。また、リソース制御機能の合理的な使用は、クラスターの数を減らし、運用および保守の難しさを軽減し、管理コストを節約できます。

  TiDB v7.1.0では、この機能により、実際のワークロードまたはハードウェア展開に基づいてシステム容量を推定する機能が導入されます。推定機能により、容量計画のためのより正確な参照が提供され、企業レベルのシナリオの安定性ニーズを満たすためにTiDBリソースの割り当てをよりよく管理できます。

  ユーザーエクスペリエンスを向上させるために、TiDBダッシュボードは[リソースマネージャーページ](/dashboard/dashboard-resource-manager.md)を提供します。このページでリソースグループの構成を表示し、クラスター容量を視覚的に推定して、合理的なリソース割り当てを容易にすることができます。

  詳細については、[ドキュメント](/tidb-resource-control.md)を参照してください。

- フォールトトレランスと自動回復能力を向上するために、Fast Online DDLのチェックポイントメカニズムをサポート [#42164](https://github.com/pingcap/tidb/issues/42164) @[tangenta](https://github.com/tangenta)

  TiDB v7.1.0では、[Fast Online DDL](/ddl-introduction.md)のチェックポイントメカニズムが導入され、Fast Online DDLのフォールトトレランスと自動回復能力が大幅に向上します。たとえTiDBオーナーノードが障害のために再起動または変更された場合でも、TiDBは定期的に更新されるチェックポイントから進行状況を回復できるため、DDLの実行がより安定し効率的になります。

  詳細については、[ドキュメント](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)を参照してください。

- バックアップ＆リストアがチェックポイントリストアをサポート [#42339](https://github.com/pingcap/tidb/issues/42339) @[Leavrth](https://github.com/Leavrth)

  スナップショットリストアまたはログリストアは、ディスクの枯渇やノードのクラッシュなどの回復可能なエラーによって中断されることがあります。TiDB v7.1.0以前では、中断前のリカバリ進行状況はエラーが解消された後も無効になり、リストアを最初からやり直す必要がありました。大規模なクラスターの場合、これにはかなりの追加コストがかかります。

  TiDB v7.1.0から、バックアップ＆リストア（BR）はチェックポイントリストア機能を導入し、中断されたリストアを継続できるようになりました。この機能により、中断されたリストアのほとんどのリカバリ進行状況を保持できます。

  詳細については、[ドキュメント](/br/br-checkpoint-restore.md)を参照してください。

- 統計の読み込み戦略を最適化 [#42160](https://github.com/pingcap/tidb/issues/42160) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

  TiDB v7.1.0では、実験的な機能として軽量統計の初期化が導入されました。軽量統計の初期化により、起動時に読み込む必要がある統計の数が大幅に減少し、統計の読み込み速度が向上します。この機能により、TiDBの安定性が複雑なランタイム環境で向上し、TiDBノードの再起動時の全体的なサービスへの影響が軽減されます。この機能を有効にするには、パラメータ[`lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710)を`true`に設定できます。

  TiDBの起動中に、初期統計が完全に読み込まれる前に実行されるSQLステートメントは、サブ最適な実行計画を持つ可能性があり、パフォーマンスの問題を引き起こすことがあります。そのような問題を回避するために、TiDB v7.1.0では、設定パラメータ[`force-init-stats`](/tidb-configuration-file.md#force-init-stats-new-in-v710)が導入されました。このオプションを使用すると、TiDBが起動中に統計の初期化が完了した後にのみサービスを提供するかどうかを制御できます。このパラメータはデフォルトで無効になっています。

  詳細については、[ドキュメント](/statistics.md#load-statistics)を参照してください。

- TiCDCは、単一行データのデータ整合性検証機能をサポート [#8718](https://github.com/pingcap/tiflow/issues/8718) [#42747](https://github.com/pingcap/tidb/issues/42747) @[3AceShowHand](https://github.com/3AceShowHand) @[zyguan](https://github.com/zyguan)

  v7.1.0から、TiCDCはデータ整合性検証機能を導入し、チェックサムアルゴリズムを使用して単一行データの整合性を検証します。この機能により、TiDBからデータを書き込み、TiCDCを介してレプリケートし、Kafkaクラスターに書き込む過程でエラーが発生していないかを検証できます。データ整合性検証機能は、Kafkaをダウンストリームとして使用するチェンジフィードのみをサポートし、現在はAvroプロトコルのみをサポートしています。

  詳細については、[ドキュメント](/ticdc/ticdc-integrity-check.md)を参照してください。

- TiCDCはDDLレプリケーション操作を最適化 [#8686](https://github.com/pingcap/tiflow/issues/8686) @[hi-rustin](https://github.com/hi-rustin)

  v7.1.0以前では、大規模なテーブルのすべての行に影響を与えるDDL操作（列の追加または削除など）を実行すると、TiCDCのレプリケーションレイテンシが大幅に増加しました。v7.1.0から、TiCDCはこのレプリケーション操作を最適化し、DDL操作がダウンストリームレイテンシに与える影響を軽減します。

  詳細については、[ドキュメント](/ticdc/ticdc-faq.md#does-ticdc-replicate-data-changes-caused-by-lossy-ddl-operations-to-the-downstream)を参照してください。

- TiDB Lightningの安定性を向上させ、TiBレベルのデータをインポートする際の4つの構成項目を追加 [#43510](https://github.com/pingcap/tidb/issues/43510) [#43657](https://github.com/pingcap/tidb/issues/43657) @[D3Hunter](https://github.com/D3Hunter) @[lance6716](https://github.com/lance6716)

  v7.1.0から、TiDB Lightningは、TiBレベルのデータをインポートする際の安定性を向上するために、4つの構成項目を追加しました。

  - `tikv-importer.region-split-batch-size`は、リージョンをバッチで分割する際のリージョン数を制御します。デフォルト値は`4096`です。
  - `tikv-importer.region-split-concurrency`は、リージョンを分割する際の並行性を制御します。デフォルト値はCPUコア数です。
  - `tikv-importer.region-check-backoff-limit`は、リージョンの分割および散布操作後にリージョンがオンラインになるまでのリトライ回数を制御します。デフォルト値は`1800`で、最大リトライ間隔は2秒です。リージョンがリトライ間隔の間にオンラインになった場合、リトライ回数は増加しません。
  - `tikv-importer.pause-pd-scheduler-scope`は、TiDB LightningがPDスケジューリングを一時停止するスコープを制御します。値のオプションは`"table"`および`"global"`です。デフォルト値は`"table"`です。v6.1.0より前のTiDBバージョンでは、データインポート中にグローバルスケジューリングを一時停止する`"global"`オプションのみを構成できます。v6.1.0以降では、`"table"`オプションがサポートされ、ターゲットテーブルデータを格納するリージョンのみでスケジューリングが一時停止されます。大容量のデータを扱うシナリオでは、この構成項目を`"global"`に設定することをお勧めします。

  詳細については、[ドキュメント](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

### SQL {#sql}

- サポートをして、`INSERT INTO SELECT` ステートメントを使用して TiFlash クエリ結果を保存する (GA) [#37515](https://github.com/pingcap/tidb/issues/37515) @[gengliqi](https://github.com/gengliqi)

  v6.5.0 から、TiDB は `INSERT INTO SELECT` ステートメントの `SELECT` 句（解析クエリ）を TiFlash にプッシュダウンすることをサポートしています。これにより、TiFlash クエリ結果を `INSERT INTO` で指定された TiDB テーブルに簡単に保存し、結果のキャッシュ（つまり、結果のマテリアリゼーション）を行うことができます。

  v7.1.0 では、この機能が一般的に利用可能になりました。`INSERT INTO SELECT` ステートメントの `SELECT` 句の実行中に、オプティマイザは [SQL モード](/sql-mode.md) と TiFlash レプリカのコスト見積もりに基づいて、クエリを TiFlash にプッシュダウンするかどうかを知能的に決定できます。そのため、実験フェーズで導入された `tidb_enable_tiflash_read_for_write_stmt` システム変数は廃止されました。TiDB は `INSERT INTO SELECT` ステートメントの計算ルールが `STRICT SQL Mode` の要件を満たさないため、現在のセッションの [SQL モード](/sql-mode.md) が厳格でない場合にのみ、`INSERT INTO SELECT` ステートメントの `SELECT` 句を TiFlash にプッシュダウンすることを許可しています。つまり、`sql_mode` の値に `STRICT_TRANS_TABLES` と `STRICT_ALL_TABLES` が含まれていないことを意味します。

  詳細については、[ドキュメント](/tiflash/tiflash-results-materialization.md) を参照してください。

- MySQL 互換の多値インデックスが一般的に利用可能になりました (GA) [#39592](https://github.com/pingcap/tidb/issues/39592) @[xiongjiwei](https://github.com/xiongjiwei) @[qw4990](https://github.com/qw4990) @[YangKeao](https://github.com/YangKeao)

  JSON カラム内の配列の値をフィルタリングすることは一般的な操作ですが、通常のインデックスではそのような操作の高速化に役立ちません。配列に多値インデックスを作成すると、フィルタリングのパフォーマンスが大幅に向上します。JSON カラム内の配列に多値インデックスがある場合、`MEMBER OF()`、`JSON_CONTAINS()`、`JSON_OVERLAPS()` 関数で検索条件をフィルタリングするために、多値インデックスを使用できます。これにより、I/O 消費が減少し、操作速度が向上します。

  v7.1.0 では、多値インデックス機能が一般的に利用可能になりました (GA)。より完全なデータ型をサポートし、TiDB ツールと互換性があります。本番環境で JSON 配列の検索操作を高速化するために、多値インデックスを使用できます。

  詳細については、[ドキュメント](/sql-statements/sql-statement-create-index.md#multi-valued-indexes) を参照してください。

- ハッシュおよびキーでパーティションされたテーブルのパーティション管理を改善しました [#42728](https://github.com/pingcap/tidb/issues/42728) @[mjonss](https://github.com/mjonss)

  v7.1.0 より前では、TiDB のハッシュおよびキーでパーティションされたテーブルは `TRUNCATE PARTITION` パーティション管理ステートメントのみをサポートしていました。v7.1.0 からは、ハッシュおよびキーでパーティションされたテーブルは `ADD PARTITION` および `COALESCE PARTITION` パーティション管理ステートメントもサポートします。そのため、必要に応じてハッシュおよびキーでパーティションされたテーブルのパーティション数を柔軟に調整できます。たとえば、`ADD PARTITION` ステートメントでパーティション数を増やしたり、`COALESCE PARTITION` ステートメントでパーティション数を減らしたりできます。

  詳細については、[ドキュメント](/partitioned-table.md#manage-hash-and-key-partitions) を参照してください。

- Range INTERVAL パーティショニングの構文が一般的に利用可能になりました (GA) [#35683](https://github.com/pingcap/tidb/issues/35683) @[mjonss](https://github.com/mjonss)

  Range INTERVAL パーティショニングの構文（v6.3.0 で導入）が GA になりました。この構文を使用すると、すべてのパーティションを列挙せずに、所望の間隔で Range パーティショニングを定義できます。これにより、Range パーティショニングの DDL ステートメントの長さが大幅に短縮されます。この構文は、元の Range パーティショニングと同等です。

  詳細については、[ドキュメント](/partitioned-table.md#range-interval-partitioning) を参照してください。

- 生成列が一般的に利用可能になりました (GA) @[bb7133](https://github.com/bb7133)

  生成列はデータベースの貴重な機能です。テーブルを作成する際、ユーザーによって明示的に挿入または更新されるのではなく、テーブル内の他の列の値に基づいて列の値が計算されるように定義できます。この生成列は、仮想列または格納列のいずれかになります。TiDB は以前のバージョンから MySQL 互換の生成列をサポートしており、この機能は v7.1.0 で一般的に利用可能になりました。

  生成列を使用すると、MySQL 互換性が向上し、MySQL からの移行プロセスが簡素化されます。また、データのメンテナンスの複雑さが低減され、データの整合性とクエリの効率が向上します。

  詳細については、[ドキュメント](/generated-columns.md) を参照してください。

### DB operations {#db-operations}

- 手動で DDL 操作をキャンセルせずにスムーズなクラスタのアップグレードをサポート (実験的) [#39751](https://github.com/pingcap/tidb/issues/39751) @[zimulala](https://github.com/zimulala)

  TiDB v7.1.0 より前では、クラスタをアップグレードするには、アップグレード前に実行中またはキューに入れられた DDL タスクを手動でキャンセルする必要があり、アップグレード後にそれらを追加する必要がありました。

  スムーズなアップグレード体験を提供するために、TiDB v7.1.0 では、自動的に DDL タスクを一時停止および再開する機能がサポートされています。v7.1.0 からは、アップグレード前に DDL タスクを手動でキャンセルする必要がなくなります。TiDB はアップグレード前に実行中またはキューに入れられたユーザー DDL タスクを自動的に一時停止し、ローリングアップグレード後にこれらのタスクを再開します。これにより、TiDB クラスタのアップグレードが容易になります。

  詳細については、[ドキュメント](/smooth-upgrade-tidb.md) を参照してください。

### Observability {#observability}

- オプティマイザの診断情報を強化しました [#43122](https://github.com/pingcap/tidb/issues/43122) @[time-and-fate](https://github.com/time-and-fate)

  十分な情報を取得することは、SQL パフォーマンス診断の鍵です。v7.1.0 では、TiDB はさまざまな診断ツールにオプティマイザのランタイム情報を追加し、実行計画の選択方法に関するより良い洞察を提供し、SQL パフォーマンスの問題のトラブルシューティングを支援します。新しい情報には次のものが含まれます。

  - [`PLAN REPLAYER`](/sql-plan-replayer.md) の出力に `debug_trace.json`。
  - [`EXPLAIN`](/explain-walkthrough.md) の出力の `operator info` に部分的な統計詳細。
  - [遅いクエリ](/identify-slow-queries.md) の `Stats` フィールドに部分的な統計詳細。

  詳細については、[現場情報を保存および復元するために `PLAN REPLAYER` を使用](/sql-plan-replayer.md)、[`EXPLAIN` の解説](/explain-walkthrough.md)、および[遅いクエリの特定](/identify-slow-queries.md) を参照してください。

### セキュリティ {#security}

- TiFlashシステムテーブル情報のクエリに使用されるインターフェースを置換する [#6941](https://github.com/pingcap/tiflash/issues/6941) @[flowbehappy](https://github.com/flowbehappy)

  v7.1.0から、TiDBのために[`INFORMATION_SCHEMA.TIFLASH_TABLES`](/information-schema/information-schema-tiflash-tables.md)と[`INFORMATION_SCHEMA.TIFLASH_SEGMENTS`](/information-schema/information-schema-tiflash-segments.md)システムテーブルのクエリサービスを提供する際、TiFlashはHTTPポートの代わりにgRPCポートを使用し、これによりHTTPサービスのセキュリティリスクを回避します。

- LDAP認証のサポート [#43580](https://github.com/pingcap/tidb/issues/43580) @[YangKeao](https://github.com/YangKeao)

  v7.1.0から、TiDBはLDAP認証をサポートし、`authentication_ldap_sasl`と`authentication_ldap_simple`の2つの認証プラグインを提供します。

  詳細については、[ドキュメント](/security-compatibility-with-mysql.md)を参照してください。

- データベース監査機能の強化（エンタープライズエディション）

  v7.1.0では、TiDBエンタープライズエディションはデータベース監査機能を強化し、容量を大幅に拡張し、ユーザーエクスペリエンスを向上させ、企業のデータベースセキュリティコンプライアンスのニーズに対応します。

  - より細かい監査イベントの定義とより細かい監査設定のために「フィルター」と「ルール」の概念を導入します。
  - よりユーザーフレンドリーな構成方法を提供するために、JSON形式でルールの定義をサポートします。
  - 自動ログローテーションとスペース管理機能を追加し、保持時間とログサイズの2つの次元でログローテーションを構成することをサポートします。
  - TEXTおよびJSON形式で監査ログを出力することをサポートし、サードパーティツールとの簡単な統合を容易にします。
  - 監査ログのマスキングをサポートします。セキュリティを強化するために、すべてのリテラルを置換できます。

  データベース監査はTiDBエンタープライズエディションの重要な機能です。この機能は、企業がデータセキュリティとコンプライアンスを確保するための強力な監視および監査ツールを提供します。この機能は、企業のマネージャーがデータベース操作のソースと影響を追跡し、違法なデータの盗難や改ざんを防ぐのに役立ちます。さらに、データベース監査は、企業がさまざまな規制およびコンプライアンス要件を満たすのにも役立ち、法的および倫理的なコンプライアンスを確保します。この機能は、企業情報セキュリティにおいて重要な応用価値を持っています。

  詳細については、[ユーザーガイド](https://static.pingcap.com/files/2023/09/18204824/TiDB-Database-Auditing-User-Guide1.pdf)を参照してください。この機能はTiDBエンタープライズエディションに含まれています。この機能を使用するには、[TiDBエンタープライズ](https://www.pingcap.com/tidb-enterprise)ページにアクセスしてTiDBエンタープライズエディションを入手してください。

## 互換性の変更 {#compatibility-changes}

> **Note:**
>
> このセクションでは、v7.0.0から現在のバージョン（v7.1.0）にアップグレードする際に知っておく必要がある互換性の変更を提供します。v6.6.0またはそれ以前のバージョンから現在のバージョンにアップグレードする場合、中間バージョンで導入された互換性の変更も確認する必要があります。

### 動作の変更 {#behavior-changes}

- セキュリティを向上させるために、TiFlashはHTTPサービスポート（デフォルト`8123`）を非推奨とし、代わりにgRPCポートを使用します

  TiFlashをv7.1.0にアップグレードした場合、TiDBをv7.1.0にアップグレードする際、TiDBはTiFlashシステムテーブル（[`INFORMATION_SCHEMA.TIFLASH_TABLES`](/information-schema/information-schema-tiflash-tables.md)および[`INFORMATION_SCHEMA.TIFLASH_SEGMENTS`](/information-schema/information-schema-tiflash-segments.md)）を読み取ることができません。

- TiDB v6.2.0からv7.0.0のバージョンのTiDB Lightningは、TiDBクラスターバージョンに基づいてグローバルスケジューリングを一時停止するかどうかを決定します。TiDBクラスターバージョンがv6.1.0以上の場合、スケジューリングは対象テーブルデータを保存するリージョンに対してのみ一時停止され、対象テーブルのインポートが完了した後に再開されます。他のバージョンの場合、TiDB Lightningはグローバルスケジューリングを一時停止します。TiDB v7.1.0からは、[`pause-pd-scheduler-scope`](/tidb-lightning/tidb-lightning-configuration.md)を構成することで、グローバルスケジューリングを一時停止するかどうかを制御できます。デフォルトでは、TiDB Lightningは対象テーブルデータを保存するリージョンに対してスケジューリングを一時停止します。対象クラスターバージョンがv6.1.0未満の場合、エラーが発生します。この場合、パラメータの値を「global」に変更して再試行できます。

- TiDB v7.1.0で[`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-cluster.md)を使用すると、一部のリージョンがFLASHBACKプロセスの完了後もFLASHBACKプロセスに残る場合があります。v7.1.0ではこの機能の使用を避けることをお勧めします。詳細については、issue [#44292](https://github.com/pingcap/tidb/issues/44292)を参照してください。この問題に遭遇した場合は、[TiDBスナップショットバックアップとリストア](/br/br-snapshot-guide.md)機能を使用してデータを復元できます。"

### システム変数 {#system-variables}

| Variable name                                                                                                                           | Change type | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| --------------------------------------------------------------------------------------------------------------------------------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`tidb_enable_tiflash_read_for_write_stmt`](/system-variables.md#tidb_enable_tiflash_read_for_write_stmt-new-in-v630)                   | Deprecated  | Changes the default value from `OFF` to `ON`. When [`tidb_allow_mpp = ON`](/system-variables.md#tidb_allow_mpp-new-in-v50), the optimizer intelligently decides whether to push a query down to TiFlash based on the [SQL mode](/sql-mode.md) and the cost estimates of the TiFlash replica.                                                                                                                                                                                                                                                                                                 |
| [`tidb_non_prepared_plan_cache_size`](/system-variables.md#tidb_non_prepared_plan_cache_size)                                           | Deprecated  | Starting from v7.1.0, this system variable is deprecated. You can use [`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710) to control the maximum number of plans that can be cached.                                                                                                                                                                                                                                                                                                                                                             |
| [`tidb_prepared_plan_cache_size`](/system-variables.md#tidb_prepared_plan_cache_size-new-in-v610)                                       | Deprecated  | Starting from v7.1.0, this system variable is deprecated. You can use [`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710) to control the maximum number of plans that can be cached.                                                                                                                                                                                                                                                                                                                                                             |
| `tidb_ddl_distribute_reorg`                                                                                                             | Deleted     | This variable is renamed to [`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710).                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| [`default_authentication_plugin`](/system-variables.md#default_authentication_plugin)                                                   | Modified    | Introduces two new value options: `authentication_ldap_sasl` and `authentication_ldap_simple`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| [`tidb_load_based_replica_read_threshold`](/system-variables.md#tidb_load_based_replica_read_threshold-new-in-v700)                     | Modified    | Takes effect starting from v7.1.0 and controls the threshold for triggering load-based replica read. Changes the default value from `"0s"` to `"1s"` after further tests.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| [`tidb_opt_enable_late_materialization`](/system-variables.md#tidb_opt_enable_late_materialization-new-in-v700)                         | Modified    | Changes the default value from `OFF` to `ON`, meaning that the TiFlash late materialization feature is enabled by default.                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| [`authentication_ldap_sasl_auth_method_name`](/system-variables.md#authentication_ldap_sasl_auth_method_name-new-in-v710)               | Newly added | Specifies the authentication method name in LDAP SASL authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| [`authentication_ldap_sasl_bind_base_dn`](/system-variables.md#authentication_ldap_sasl_bind_base_dn-new-in-v710)                       | Newly added | Limits the search scope within the search tree in LDAP SASL authentication. If a user is created without `AS ...` clause, TiDB automatically searches the `dn` in LDAP server according to the user name.                                                                                                                                                                                                                                                                                                                                                                                    |
| [`authentication_ldap_sasl_bind_root_dn`](/system-variables.md#authentication_ldap_sasl_bind_root_dn-new-in-v710)                       | Newly added | Specifies the `dn` used to login to the LDAP server to search users in LDAP SASL authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| [`authentication_ldap_sasl_bind_root_pwd`](/system-variables.md#authentication_ldap_sasl_bind_root_pwd-new-in-v710)                     | Newly added | Specifies the password used to login to the LDAP server to search users in LDAP SASL authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| [`authentication_ldap_sasl_ca_path`](/system-variables.md#authentication_ldap_sasl_ca_path-new-in-v710)                                 | Newly added | Specifies the absolute path of the certificate authority file for StartTLS connections in LDAP SASL authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| [`authentication_ldap_sasl_init_pool_size`](/system-variables.md#authentication_ldap_sasl_init_pool_size-new-in-v710)                   | Newly added | Specifies the initial connections in the connection pool to the LDAP server in LDAP SASL authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| [`authentication_ldap_sasl_max_pool_size`](/system-variables.md#authentication_ldap_sasl_max_pool_size-new-in-v710)                     | Newly added | Specifies the maximum connections in the connection pool to the LDAP server in LDAP SASL authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| [`authentication_ldap_sasl_server_host`](/system-variables.md#authentication_ldap_sasl_server_host-new-in-v710)                         | Newly added | Specifies the LDAP server host in LDAP SASL authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| [`authentication_ldap_sasl_server_port`](/system-variables.md#authentication_ldap_sasl_server_port-new-in-v710)                         | Newly added | Specifies the LDAP server TCP/IP port number in LDAP SASL authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| [`authentication_ldap_sasl_tls`](/system-variables.md#authentication_ldap_sasl_tls-new-in-v710)                                         | Newly added | Specifies whether connections by the plugin to the LDAP server are protected with StartTLS in LDAP SASL authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| [`authentication_ldap_simple_auth_method_name`](/system-variables.md#authentication_ldap_simple_auth_method_name-new-in-v710)           | Newly added | Specifies the authentication method name in LDAP simple authentication. It only supports `SIMPLE`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| [`authentication_ldap_simple_bind_base_dn`](/system-variables.md#authentication_ldap_simple_bind_base_dn-new-in-v710)                   | Newly added | Limits the search scope within the search tree in LDAP simple authentication. If a user is created without `AS ...` clause, TiDB will automatically search the `dn` in LDAP server according to the user name.                                                                                                                                                                                                                                                                                                                                                                               |
| [`authentication_ldap_simple_bind_root_dn`](/system-variables.md#authentication_ldap_simple_bind_root_dn-new-in-v710)                   | Newly added | Specifies the `dn` used to login to the LDAP server to search users in LDAP simple authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| [`authentication_ldap_simple_bind_root_pwd`](/system-variables.md#authentication_ldap_simple_bind_root_pwd-new-in-v710)                 | Newly added | Specifies the password used to login to the LDAP server to search users in LDAP simple authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| [`authentication_ldap_simple_ca_path`](/system-variables.md#authentication_ldap_simple_ca_path-new-in-v710)                             | Newly added | Specifies the absolute path of the certificate authority file for StartTLS connections in LDAP simple authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| [`authentication_ldap_simple_init_pool_size`](/system-variables.md#authentication_ldap_simple_init_pool_size-new-in-v710)               | Newly added | Specifies the initial connections in the connection pool to the LDAP server in LDAP simple authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| [`authentication_ldap_simple_max_pool_size`](/system-variables.md#authentication_ldap_simple_max_pool_size-new-in-v710)                 | Newly added | Specifies the maximum connections in the connection pool to the LDAP server in LDAP simple authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| [`authentication_ldap_simple_server_host`](/system-variables.md#authentication_ldap_simple_server_host-new-in-v710)                     | Newly added | Specifies the LDAP server host in LDAP simple authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| [`authentication_ldap_simple_server_port`](/system-variables.md#authentication_ldap_simple_server_port-new-in-v710)                     | Newly added | Specifies the LDAP server TCP/IP port number in LDAP simple authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| [`authentication_ldap_simple_tls`](/system-variables.md#authentication_ldap_simple_tls-new-in-v710)                                     | Newly added | Specifies whether connections by the plugin to the LDAP server are protected with StartTLS in LDAP simple authentication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| [`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710)                                                       | Newly added | Controls whether to enable the distributed execution framework. After enabling distributed execution, DDL, import, and other supported backend tasks will be jointly completed by multiple TiDB nodes in the cluster. This variable was renamed from `tidb_ddl_distribute_reorg`.                                                                                                                                                                                                                                                                                                            |
| [`tidb_enable_non_prepared_plan_cache_for_dml`](/system-variables.md#tidb_enable_non_prepared_plan_cache_for_dml-new-in-v710)           | Newly added | Controls whether to enable the [Non-prepared plan cache](/sql-non-prepared-plan-cache.md) feature for DML statements.                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| [`tidb_enable_row_level_checksum`](/system-variables.md#tidb_enable_row_level_checksum-new-in-v710)                                     | Newly added | Controls whether to enable the TiCDC data integrity validation for single-row data feature.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| [`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v710)                                                         | Newly added | This variable provides more fine-grained control over the optimizer and helps to prevent performance regression after upgrading caused by behavior changes in the optimizer.                                                                                                                                                                                                                                                                                                                                                                                                                 |
| [`tidb_plan_cache_invalidation_on_fresh_stats`](/system-variables.md#tidb_plan_cache_invalidation_on_fresh_stats-new-in-v710)           | Newly added | Controls whether to invalidate the plan cache automatically when statistics on related tables are updated.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| [`tidb_plan_cache_max_plan_size`](/system-variables.md#tidb_plan_cache_max_plan_size-new-in-v710)                                       | Newly added | Controls the maximum size of a plan that can be cached in prepared or non-prepared plan cache.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| [`tidb_prefer_broadcast_join_by_exchange_data_size`](/system-variables.md#tidb_prefer_broadcast_join_by_exchange_data_size-new-in-v710) | Newly added | Controls whether to use the algorithm with the minimum overhead of network transmission. If this variable is enabled, TiDB estimates the size of the data to be exchanged in the network using `Broadcast Hash Join` and `Shuffled Hash Join` respectively, and then chooses the one with the smaller size. [`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50) and [`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50) will not take effect after this variable is enabled. |
| [`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710)                                         | Newly added | Controls the maximum number of plans that can be cached. Prepared plan cache and non-prepared plan cache share the same cache.                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

### 設定ファイルパラメータ {#configuration-file-parameters}

| 設定ファイル         | 設定パラメータ                                                                                                                        | 変更タイプ | 説明                                                                                                                                                                  |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------ | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TiDB           | [`performance.force-init-stats`](/tidb-configuration-file.md#force-init-stats-new-in-v710)                                     | 新規追加  | TiDBの起動中に統計の初期化が完了するのを待つかどうかを制御します。                                                                                                                                 |
| TiDB           | [`performance.lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710)                                       | 新規追加  | TiDBの起動中に軽量統計の初期化を使用するかどうかを制御します。                                                                                                                                   |
| TiDB           | [`log.timeout`](/tidb-configuration-file.md#timeout-new-in-v710)                                                               | 新規追加  | TiDBでのログ書き込み操作のタイムアウトを設定します。デフォルト値は`0`で、タイムアウトが設定されていないことを意味します。                                                                                                    |
| TiKV           | [`region-compact-min-redundant-rows`](/tikv-configuration-file.md#region-compact-min-redundant-rows-new-in-v710)               | 新規追加  | RocksDBのコンパクションをトリガーするために必要な冗長MVCC行の数を設定します。デフォルト値は`50000`です。                                                                                                       |
| TiKV           | [`region-compact-redundant-rows-percent`](/tikv-configuration-file.md#region-compact-redundant-rows-percent-new-in-v710)       | 新規追加  | RocksDBのコンパクションをトリガーするために必要な冗長MVCC行の割合を設定します。デフォルト値は`20`です。                                                                                                         |
| TiKV           | [`split.byte-threshold`](/tikv-configuration-file.md#byte-threshold-new-in-v50)                                                | 変更済み  | [`region-split-size`](/tikv-configuration-file.md#region-split-size)が4 GB以上の場合、デフォルト値を`30MiB`から`100MiB`に変更します。                                                      |
| TiKV           | [`split.qps-threshold`](/tikv-configuration-file.md#qps-threshold)                                                             | 変更済み  | [`region-split-size`](/tikv-configuration-file.md#region-split-size)が4 GB以上の場合、デフォルト値を`3000`から`7000`に変更します。                                                         |
| TiKV           | [`split.region-cpu-overload-threshold-ratio`](/tikv-configuration-file.md#region-cpu-overload-threshold-ratio-new-in-v620)     | 変更済み  | [`region-split-size`](/tikv-configuration-file.md#region-split-size)が4 GB以上の場合、デフォルト値を`0.25`から`0.75`に変更します。                                                         |
| TiKV           | [`region-compact-check-step`](/tikv-configuration-file.md#region-compact-check-step)                                           | 変更済み  | パーティション化されたRaft KVが有効になっている場合(`storage.engine="partitioned-raft-kv"`)、デフォルト値を`100`から`5`に変更します。                                                                      |
| PD             | [`store-limit-version`](/pd-configuration-file.md#store-limit-version-new-in-v710)                                             | 新規追加  | ストア制限のモードを制御します。値のオプションは`"v1"`と`"v2"`です。                                                                                                                            |
| PD             | [`schedule.enable-diagnostic`](/pd-configuration-file.md#enable-diagnostic-new-in-v630)                                        | 変更済み  | スケジューラの診断機能がデフォルトで有効になるように、デフォルト値を`false`から`true`に変更します。                                                                                                            |
| TiFlash        | `http_port`                                                                                                                    | 削除済み  | HTTPサービスポート（デフォルト`8123`）を非推奨にします。                                                                                                                                   |
| TiDB Lightning | [`tikv-importer.pause-pd-scheduler-scope`](/tidb-lightning/tidb-lightning-configuration.md)                                    | 新規追加  | TiDB LightningがPDスケジューリングを一時停止する範囲を制御します。デフォルト値は`"table"`で、値のオプションは`"global"`と`"table"`です。                                                                          |
| TiDB Lightning | [`tikv-importer.region-check-backoff-limit`](/tidb-lightning/tidb-lightning-configuration.md)                                  | 新規追加  | 分割および散布操作後にRegionがオンラインになるまで待機するリトライ回数を制御します。デフォルト値は`1800`です。最大リトライ間隔は2秒です。リトライ回数は、リトライ間にいずれかのRegionがオンラインになった場合には増加しません。                                           |
| TiDB Lightning | [`tikv-importer.region-split-batch-size`](/tidb-lightning/tidb-lightning-configuration.md)                                     | 新規追加  | バッチでRegionを分割する際のRegion数を制御します。デフォルト値は`4096`です。                                                                                                                     |
| TiDB Lightning | [`tikv-importer.region-split-concurrency`](/tidb-lightning/tidb-lightning-configuration.md)                                    | 新規追加  | Regionを分割する際の並行性を制御します。デフォルト値はCPUコア数です。                                                                                                                             |
| TiCDC          | [`insecure-skip-verify`](/ticdc/ticdc-sink-to-kafka.md)                                                                        | 新規追加  | TLSが有効になっている場合に、Kafkaへデータをレプリケートするシナリオで認証アルゴリズムが設定されるかどうかを制御します。                                                                                                    |
| TiCDC          | [`integrity.corruption-handle-level`](/ticdc/ticdc-changefeed-config.md#cli-and-configuration-parameters-of-ticdc-changefeeds) | 新規追加  | 単一行データのチェックサム検証に失敗した場合のChangefeedのログレベルを指定します。デフォルト値は`"warn"`です。値のオプションは`"warn"`と`"error"`です。                                                                       |
| TiCDC          | [`integrity.integrity-check-level`](/ticdc/ticdc-changefeed-config.md#cli-and-configuration-parameters-of-ticdc-changefeeds)   | 新規追加  | 単一行データのチェックサム検証を有効にするかどうかを制御します。デフォルト値は`"none"`で、機能を無効にします。                                                                                                         |
| TiCDC          | [`sink.only-output-updated-columns`](/ticdc/ticdc-changefeed-config.md#cli-and-configuration-parameters-of-ticdc-changefeeds)  | 新規追加  | 更新された列のみを出力するかどうかを制御します。デフォルト値は`false`です。                                                                                                                           |
| TiCDC          | [`sink.enable-partition-separator`](/ticdc/ticdc-changefeed-config.md#cli-and-configuration-parameters-of-ticdc-changefeeds)   | 変更済み  | 追加のテスト後、デフォルト値を`false`から`true`に変更します。これにより、テーブルのパーティションがデフォルトで別のディレクトリに保存されるようになります。パーティション化されたテーブルのストレージサービスへのレプリケーション中にデータ損失の潜在的な問題を回避するために、値を`true`に保つことをお勧めします。 |

## 改善 {#improvements}

- TiDB

  - `SHOW INDEX`のCardinality列で対応する列の異なる値の数を表示する[#42227](https://github.com/pingcap/tidb/issues/42227) @[winoros](https://github.com/winoros)
  - `SQL_NO_CACHE`を使用して、TTLスキャンクエリがTiKVブロックキャッシュに影響を与えないようにする[#43206](https://github.com/pingcap/tidb/issues/43206) @[lcwangchao](https://github.com/lcwangchao)
  - `MAX_EXECUTION_TIME`に関連するエラーメッセージを改善して、MySQLと互換性のあるものにする[#43031](https://github.com/pingcap/tidb/issues/43031) @[dveeden](https://github.com/dveeden)
  - IndexLookUpでパーティションテーブルでMergeSort演算子を使用するサポート[#26166](https://github.com/pingcap/tidb/issues/26166) @[Defined2014](https://github.com/Defined2014)
  - `caching_sha2_password`を改良して、MySQLと互換性のあるものにする[#43576](https://github.com/pingcap/tidb/issues/43576) @[asjdf](https://github.com/asjdf)

- TiKV

  - パーティション化されたRaft KVを使用する際の分割操作が書き込みQPSに与える影響を減らす[#14447](https://github.com/tikv/tikv/issues/14447) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
  - パーティション化されたRaft KVを使用する際のスナップショットが占有するスペースを最適化する[#14581](https://github.com/tikv/tikv/issues/14581) @[bufferflies](https://github.com/bufferflies)
  - TiKVでリクエスト処理の各段階の詳細な時間情報を提供する[#12362](https://github.com/tikv/tikv/issues/12362) @[cfzjywxk](https://github.com/cfzjywxk)
  - ログバックアップでPDをメタストアとして使用する[#13867](https://github.com/tikv/tikv/issues/13867) @[YuJuncen](https://github.com/YuJuncen)

- PD

  - スナップショットの実行の詳細に基づいて自動的にストア制限のサイズを調整するコントローラーを追加する。このコントローラーを有効にするには、`store-limit-version`を`v2`に設定する。有効になると、スケーリングインまたはスケーリングアウトの速度を制御するためにストア制限の構成を手動で調整する必要はありません[#6147](https://github.com/tikv/pd/issues/6147) @[bufferflies](https://github.com/bufferflies)
  - 不安定な負荷を持つリージョンの頻繁なスケジューリングを避けるために、履歴的な負荷情報を追加する[#6297](https://github.com/tikv/pd/issues/6297) @[bufferflies](https://github.com/bufferflies)
  - リーダーのヘルスチェックメカニズムを追加する。etcdリーダーが存在するPDサーバーがリーダーに選出されない場合、PDは積極的にetcdリーダーを切り替えて、PDリーダーが利用可能であることを確認する[#6403](https://github.com/tikv/pd/issues/6403) @[nolouch](https://github.com/nolouch)

- TiFlash

  - 分離されたストレージとコンピュートアーキテクチャでのTiFlashのパフォーマンスと安定性を向上させる[#6882](https://github.com/pingcap/tiflash/issues/6882) @[JaySon-Huang](https://github.com/JaySon-Huang) @[breezewish](https://github.com/breezewish) @[JinheLin](https://github.com/JinheLin)
  - Semi JoinまたはAnti Semi Joinでのクエリパフォーマンスを最適化するために、小さいテーブルをビルドサイドとして選択するサポートを追加する[#7280](https://github.com/pingcap/tiflash/issues/7280) @[yibin87](https://github.com/yibin87)
  - デフォルトの構成でBRおよびTiDB LightningからTiFlashへのデータインポートのパフォーマンスを向上させる[#7272](https://github.com/pingcap/tiflash/issues/7272) @[breezewish](https://github.com/breezewish)

- ツール

  - バックアップ＆リストア（BR）

    - ログバックアップ中にTiKV構成項目`log-backup.max-flush-interval`を変更するサポートを追加する[#14433](https://github.com/tikv/tikv/issues/14433) @[joccau](https://github.com/joccau)

  - TiCDC

    - DDLイベントが発生した場合のディレクトリ構造を最適化する[#8890](https://github.com/pingcap/tiflow/issues/8890) @[CharlesCheung96](https://github.com/CharlesCheung96)
    - TiCDCレプリケーションタスクが失敗した場合に、上流のGC TLSの設定方法を最適化する[#8403](https://github.com/pingcap/tiflow/issues/8403) @[charleszheng44](https://github.com/charleszheng44)
    - Kafka-on-Pulsar下流にデータをレプリケートするサポートを追加する[#8892](https://github.com/pingcap/tiflow/issues/8892) @[hi-rustin](https://github.com/hi-rustin)
    - データをKafkaにレプリケートする際に、更新後に変更された列のみをレプリケートするためにオープンプロトコルプロトコルを使用するサポートを追加する[#8706](https://github.com/pingcap/tiflow/issues/8706) @[sdojjy](https://github.com/sdojjy)
    - 下流の失敗やその他のシナリオでのTiCDCのエラーハンドリングを最適化する[#8657](https://github.com/pingcap/tiflow/issues/8657) @[hicqu](https://github.com/hicqu)
    - TLSを有効にするシナリオで認証アルゴリズムを設定するかどうかを制御する構成項目`insecure-skip-verify`を追加する[#8867](https://github.com/pingcap/tiflow/issues/8867) @[hi-rustin](https://github.com/hi-rustin)

  - TiDB Lightning

    - データのインポート時に不均一なリージョン分布に関連する事前チェック項目の重大度レベルを`Critical`から`Warn`に変更して、ユーザーがデータをインポートするのをブロックしないようにする[#42836](https://github.com/pingcap/tidb/issues/42836) @[okJiang](https://github.com/okJiang)
    - データインポート中に`unknown RPC`エラーが発生した場合のリトライメカニズムを追加する[#43291](https://github.com/pingcap/tidb/issues/43291) @[D3Hunter](https://github.com/D3Hunter)
    - リージョンジョブのリトライメカニズムを強化する[#43682](https://github.com/pingcap/tidb/issues/43682) @[lance6716](https://github.com/lance6716)

## バグ修正 {#bug-fixes}

- TiDB

  - Fix the issue that there is no prompt about manually executing `ANALYZE TABLE` after reorganizing partitions [#42183](https://github.com/pingcap/tidb/issues/42183) @[CbcWestwolf](https://github.com/CbcWestwolf)
  - Fix the issue of missing table names in the `ADMIN SHOW DDL JOBS` result when a `DROP TABLE` operation is being executed [#42268](https://github.com/pingcap/tidb/issues/42268) @[tiancaiamao](https://github.com/tiancaiamao)
  - Fix the issue that `Ignore Event Per Minute` and `Stats Cache LRU Cost` charts might not be displayed normally in the Grafana monitoring panel [#42562](https://github.com/pingcap/tidb/issues/42562) @[pingandb](https://github.com/pingandb)
  - Fix the issue that the `ORDINAL_POSITION` column returns incorrect results when querying the `INFORMATION_SCHEMA.COLUMNS` table [#43379](https://github.com/pingcap/tidb/issues/43379) @[bb7133](https://github.com/bb7133)
  - Fix the case sensitivity issue in some columns of the permission table [#41048](https://github.com/pingcap/tidb/issues/41048) @[bb7133](https://github.com/bb7133)
  - Fix the issue that after a new column is added in the cache table, the value is `NULL` instead of the default value of the column [#42928](https://github.com/pingcap/tidb/issues/42928) @[lqs](https://github.com/lqs)
  - Fix the issue that CTE results are incorrect when pushing down predicates [#43645](https://github.com/pingcap/tidb/issues/43645) @[winoros](https://github.com/winoros)
  - Fix the issue of DDL retry caused by write conflict when executing `TRUNCATE TABLE` for partitioned tables with many partitions and TiFlash replicas [#42940](https://github.com/pingcap/tidb/issues/42940) @[mjonss](https://github.com/mjonss)
  - Fix the issue that there is no warning when using `SUBPARTITION` in creating partitioned tables [#41198](https://github.com/pingcap/tidb/issues/41198) [#41200](https://github.com/pingcap/tidb/issues/41200) @[mjonss](https://github.com/mjonss)
  - Fix the incompatibility issue with MySQL when dealing with value overflow issues in generated columns [#40066](https://github.com/pingcap/tidb/issues/40066) @[jiyfhust](https://github.com/jiyfhust)
  - Fix the issue that `REORGANIZE PARTITION` cannot be concurrently executed with other DDL operations [#42442](https://github.com/pingcap/tidb/issues/42442) @[bb7133](https://github.com/bb7133)
  - Fix the issue that canceling the partition reorganization task in DDL might cause subsequent DDL operations to fail [#42448](https://github.com/pingcap/tidb/issues/42448) @[lcwangchao](https://github.com/lcwangchao)
  - Fix the issue that assertions on delete operations are incorrect under certain conditions [#42426](https://github.com/pingcap/tidb/issues/42426) @[tiancaiamao](https://github.com/tiancaiamao)
  - Fix the issue that TiDB server cannot start due to an error in reading the cgroup information with the error message "can't read file memory.stat from cgroup v1: open /sys/memory.stat no such file or directory" [#42659](https://github.com/pingcap/tidb/issues/42659) @[hawkingrei](https://github.com/hawkingrei)
  - Fix the `Duplicate Key` issue that occurs when updating the partition key of a row on a partitioned table with a global index [#42312](https://github.com/pingcap/tidb/issues/42312) @[L-maple](https://github.com/L-maple)
  - Fix the issue that the `Scan Worker Time By Phase` chart in the TTL monitoring panel does not display data [#42515](https://github.com/pingcap/tidb/issues/42515) @[lcwangchao](https://github.com/lcwangchao)
  - Fix the issue that some queries on partitioned tables with a global index return incorrect results [#41991](https://github.com/pingcap/tidb/issues/41991) [#42065](https://github.com/pingcap/tidb/issues/42065) @[L-maple](https://github.com/L-maple)
  - Fix the issue of displaying some error logs during the process of reorganizing a partitioned table [#42180](https://github.com/pingcap/tidb/issues/42180) @[mjonss](https://github.com/mjonss)
  - Fix the issue that the data length in the `QUERY` column of the `INFORMATION_SCHEMA.DDL_JOBS` table might exceed the column definition [#42440](https://github.com/pingcap/tidb/issues/42440) @[tiancaiamao](https://github.com/tiancaiamao)
  - Fix the issue that the `INFORMATION_SCHEMA.CLUSTER_HARDWARE` table might display incorrect values in containers [#42851](https://github.com/pingcap/tidb/issues/42851) @[hawkingrei](https://github.com/hawkingrei)
  - Fix the issue that an incorrect result is returned when you query a partitioned table using `ORDER BY` + `LIMIT` [#43158](https://github.com/pingcap/tidb/issues/43158) @[Defined2014](https://github.com/Defined2014)
  - Fix the issue of multiple DDL tasks running simultaneously using the ingest method [#42903](https://github.com/pingcap/tidb/issues/42903) @[tangenta](https://github.com/tangenta)
  - Fix the wrong value returned when querying a partitioned table using `Limit` [#24636](https://github.com/pingcap/tidb/issues/24636)
  - Fix the issue of displaying the incorrect TiDB address in IPv6 environment [#43260](https://github.com/pingcap/tidb/issues/43260) @[nexustar](https://github.com/nexustar)
  - Fix the issue of displaying incorrect values for system variables `tidb_enable_tiflash_read_for_write_stmt` and `tidb_enable_exchange_partition` [#43281](https://github.com/pingcap/tidb/issues/43281) @[gengliqi](https://github.com/gengliqi)
  - Fix the issue that when `tidb_scatter_region` is enabled, Region does not automatically split after a partition is truncated [#43174](https://github.com/pingcap/tidb/issues/43174) [#43028](https://github.com/pingcap/tidb/issues/43028) @[jiyfhust](https://github.com/jiyfhust)
  - Add checks on the tables with generated columns and report errors for unsupported DDL operations on these columns [#38988](https://github.com/pingcap/tidb/issues/38988) [#24321](https://github.com/pingcap/tidb/issues/24321) @[tiancaiamao](https://github.com/tiancaiamao)
  - Fix the issue that the error message is incorrect in certain type conversion errors [#41730](https://github.com/pingcap/tidb/issues/41730) @[hawkingrei](https://github.com/hawkingrei)
  - Fix the issue that after a TiDB node is normally shutdown, DDL tasks triggered on this node will be canceled [#43854](https://github.com/pingcap/tidb/issues/43854) @[zimulala](https://github.com/zimulala)
  - Fix the issue that when the PD member address changes, allocating ID for the `AUTO_INCREMENT` column will be blocked for a long time [#42643](https://github.com/pingcap/tidb/issues/42643) @[tiancaiamao](https://github.com/tiancaiamao)
  - Fix the issue of reporting the `GC lifetime is shorter than transaction duration` error during DDL execution [#40074](https://github.com/pingcap/tidb/issues/40074) @[tangenta](https://github.com/tangenta)
  - Fix the issue that metadata locks unexpectedly block the DDL execution [#43755](https://github.com/pingcap/tidb/issues/43755) @[wjhuang2016](https://github.com/wjhuang2016)
  - Fix the issue that the cluster cannot query some system views in IPv6 environment [#43286](https://github.com/pingcap/tidb/issues/43286) @[Defined2014](https://github.com/Defined2014)
  - Fix the issue of not finding the partition during inner join in dynamic pruning mode [#43686](https://github.com/pingcap/tidb/issues/43686) @[mjonss](https://github.com/mjonss)
  - Fix the issue that TiDB reports syntax errors when analyzing tables [#43392](https://github.com/pingcap/tidb/issues/43392) @[guo-shaoge](https://github.com/guo-shaoge)
  - Fix the issue that TiCDC might lose some row changes during table renaming [#43338](https://github.com/pingcap/tidb/issues/43338) @[tangenta](https://github.com/tangenta)
  - Fix the issue that TiDB server crashes when the client uses cursor reads [#38116](https://github.com/pingcap/tidb/issues/38116) @[YangKeao](https://github.com/YangKeao)
  - Fix the issue that `ADMIN SHOW DDL JOBS LIMIT` returns incorrect results [#42298](https://github.com/pingcap/tidb/issues/42298) @[CbcWestwolf](https://github.com/CbcWestwolf)
  - Fix the TiDB panic issue that occurs when querying union views and temporary tables with `UNION` [#42563](https://github.com/pingcap/tidb/issues/42563) @[lcwangchao](https://github.com/lcwangchao)
  - Fix the issue that renaming tables does not take effect when committing multiple statements in a transaction [#39664](https://github.com/pingcap/tidb/issues/39664) @[tiancaiamao](https://github.com/tiancaiamao)
  - Fix the incompatibility issue between the behavior of prepared plan cache and non-prepared plan cache during time conversion [#42439](https://github.com/pingcap/tidb/issues/42439) @[qw4990](https://github.com/qw4990)
  - Fix the wrong results caused by plan cache for Decimal type [#43311](https://github.com/pingcap/tidb/issues/43311) @[qw4990](https://github.com/qw4990)
  - Fix the TiDB panic issue in null-aware anti join (NAAJ) due to the wrong field type check [#42459](https://github.com/pingcap/tidb/issues/42459) @[AilinKid](https://github.com/AilinKid)
  - Fix the issue that DML execution failures in pessimistic transactions at the RC isolation level might cause inconsistency between data and indexes [#43294](https://github.com/pingcap/tidb/issues/43294) @[ekexium](https://github.com/ekexium)
  - Fix the issue that in some extreme cases, when the first statement of a pessimistic transaction is retried, resolving locks on this transaction might affect transaction correctness [#42937](https://github.com/pingcap/tidb/issues/42937) @[MyonKeminta](https://github.com/MyonKeminta)
  - Fix the issue that in some rare cases, residual pessimistic locks of pessimistic transactions might affect data correctness when GC resolves locks [#43243](https://github.com/pingcap/tidb/issues/43243) @[MyonKeminta](https://github.com/MyonKeminta)
  - Fix the issue that the `LOCK` to `PUT` optimization leads to duplicate data being returned in specific queries [#28011](https://github.com/pingcap/tidb/issues/28011) @[zyguan](https://github.com/zyguan)
  - Fix the issue that when data is changed, the locking behavior of the unique index is not consistent with that when the data is unchanged [#36438](https://github.com/pingcap/tidb/issues/36438) @[zyguan](https://github.com/zyguan)

- TiKV

  - Fix the issue that when you enable `tidb_pessimistic_txn_fair_locking`, in some extreme cases, expired requests caused by failed RPC retries might affect data correctness during the resolve lock operation [#14551](https://github.com/tikv/tikv/issues/14551) @[MyonKeminta](https://github.com/MyonKeminta)
  - Fix the issue that when you enable `tidb_pessimistic_txn_fair_locking`, in some extreme cases, expired requests caused by failed RPC retries might cause transaction conflicts to be ignored, thus affecting transaction consistency [#14311](https://github.com/tikv/tikv/issues/14311) @[MyonKeminta](https://github.com/MyonKeminta)
  - Fix the issue that encryption key ID conflict might cause the deletion of the old keys [#14585](https://github.com/tikv/tikv/issues/14585) @[tabokie](https://github.com/tabokie)
  - Fix the performance degradation issue caused by accumulated lock records when a cluster is upgraded from a previous version to v6.5 or later versions [#14780](https://github.com/tikv/tikv/issues/14780) @[MyonKeminta](https://github.com/MyonKeminta)
  - Fix the issue that the `raft entry is too large` error occurs during the PITR recovery process [#14313](https://github.com/tikv/tikv/issues/14313) @[YuJuncen](https://github.com/YuJuncen)
  - Fix the issue that TiKV panics during the PITR recovery process due to `log_batch` exceeding 2 GB [#13848](https://github.com/tikv/tikv/issues/13848) @[YuJuncen](https://github.com/YuJuncen)

- PD

  - Fix the issue that the number of `low space store` in the PD monitoring panel is abnormal after TiKV panics [#6252](https://github.com/tikv/pd/issues/6252) @[HuSharp](https://github.com/HuSharp)
  - Fix the issue that Region Health monitoring data is deleted after PD leader switch [#6366](https://github.com/tikv/pd/issues/6366) @[iosmanthus](https://github.com/iosmanthus)
  - Fix the issue that the rule checker cannot repair unhealthy Regions with the `schedule=deny` label [#6426](https://github.com/tikv/pd/issues/6426) @[nolouch](https://github.com/nolouch)
  - Fix the issue that some existing labels are lost after TiKV or TiFlash restarts [#6467](https://github.com/tikv/pd/issues/6467) @[JmPotato](https://github.com/JmPotato)
  - Fix the issue that the replication status cannot be switched when there are learner nodes in the replication mode [#14704](https://github.com/tikv/tikv/issues/14704) @[nolouch](https://github.com/nolouch)

- TiFlash

  - Fix the issue that querying data in the `TIMESTAMP` or `TIME` type returns errors after enabling late materialization [#7455](https://github.com/pingcap/tiflash/issues/7455) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)
  - Fix the issue that large update transactions might cause TiFlash to repeatedly report errors and restart [#7316](https://github.com/pingcap/tiflash/issues/7316) @[JaySon-Huang](https://github.com/JaySon-Huang)

- Tools

  - Backup & Restore (BR)

    - Fix the issue of backup slowdown when a TiKV node crashes in a cluster [#42973](https://github.com/pingcap/tidb/issues/42973) @[YuJuncen](https://github.com/YuJuncen)
    - Fix the issue of inaccurate error messages caused by a backup failure in some cases [#43236](https://github.com/pingcap/tidb/issues/43236) @[YuJuncen](https://github.com/YuJuncen)

  - TiCDC

    - Fix the issue of TiCDC time zone setting [#8798](https://github.com/pingcap/tiflow/issues/8798) @[hi-rustin](https://github.com/hi-rustin)
    - Fix the issue that TiCDC cannot automatically recover when PD address or leader fails [#8812](https://github.com/pingcap/tiflow/issues/8812) [#8877](https://github.com/pingcap/tiflow/issues/8877) @[asddongmen](https://github.com/asddongmen)
    - Fix the issue that checkpoint lag increases when one of the upstream TiKV nodes crashes [#8858](https://github.com/pingcap/tiflow/issues/8858) @[hicqu](https://github.com/hicqu)
    - Fix the issue that when replicating data to object storage, the `EXCHANGE PARTITION` operation in the upstream cannot be properly replicated to the downstream [#8914](https://github.com/pingcap/tiflow/issues/8914) @[CharlesCheung96](https://github.com/CharlesCheung96)
    - Fix the OOM issue caused by excessive memory usage of the sorter component in some special scenarios [#8974](https://github.com/pingcap/tiflow/issues/8974) @[hicqu](https://github.com/hicqu)
    - Fix the TiCDC node panic that occurs when the downstream Kafka sinks are rolling restarted [#9023](https://github.com/pingcap/tiflow/issues/9023) @[asddongmen](https://github.com/asddongmen)

  - TiDB Data Migration (DM)

    - Fix the issue that latin1 data might be corrupted during replication [#7028](https://github.com/pingcap/tiflow/issues/7028) @[lance6716](https://github.com/lance6716)

  - TiDB Dumpling

    - Fix the issue that the `UNSIGNED INTEGER` type primary key cannot be used for splitting chunks [#42620](https://github.com/pingcap/tidb/issues/42620) @[lichunzhu](https://github.com/lichunzhu)
    - Fix the issue that TiDB Dumpling might panic when `--output-file-template` is incorrectly set [#42391](https://github.com/pingcap/tidb/issues/42391) @[lichunzhu](https://github.com/lichunzhu)

  - TiDB Binlog

    - Fix the issue that an error might occur when encountering a failed DDL statement [#1228](https://github.com/pingcap/tidb-binlog/issues/1228) @[okJiang](https://github.com/okJiang)

  - TiDB Lightning

    - Fix the performance degradation issue during data import [#42456](https://github.com/pingcap/tidb/issues/42456) @[lance6716](https://github.com/lance6716)
    - Fix the issue of `write to tikv with no leader returned` when importing a large amount of data [#43055](https://github.com/pingcap/tidb/issues/43055) @[lance6716](https://github.com/lance6716)
    - Fix the issue of excessive `keys within region is empty, skip doIngest` logs during data import [#43197](https://github.com/pingcap/tidb/issues/43197) @[D3Hunter](https://github.com/D3Hunter)
    - Fix the issue that panic might occur during partial write [#43363](https://github.com/pingcap/tidb/issues/43363) @[lance6716](https://github.com/lance6716)
    - Fix the issue that OOM might occur when importing a wide table [#43728](https://github.com/pingcap/tidb/issues/43728) @[D3Hunter](https://github.com/D3Hunter)
    - Fix the issue of missing data in the TiDB Lightning Grafana dashboard [#43357](https://github.com/pingcap/tidb/issues/43357) @[lichunzhu](https://github.com/lichunzhu)
    - Fix the import failure due to incorrect setting of `keyspace-name` [#43684](https://github.com/pingcap/tidb/issues/43684) @[zeminzhou](https://github.com/zeminzhou)
    - Fix the issue that data import might be skipped during range partial write in some cases [#43768](https://github.com/pingcap/tidb/issues/43768) @[lance6716](https://github.com/lance6716)

## パフォーマンステスト {#performance-test}

TiDB v7.1.0のパフォーマンスについては、TiDB Dedicatedクラスターの[TPC-Cパフォーマンステストレポート](https://docs.pingcap.com/tidbcloud/v7.1.0-performance-benchmarking-with-tpcc)および[Sysbenchパフォーマンステストレポート](https://docs.pingcap.com/tidbcloud/v7.1.0-performance-benchmarking-with-sysbench)を参照してください。

## 貢献者 {#contributors}

TiDBコミュニティから以下の貢献者に感謝します：

- [blacktear23](https://github.com/blacktear23)
- [ethercflow](https://github.com/ethercflow)
- [hihihuhu](https://github.com/hihihuhu)
- [jiyfhust](https://github.com/jiyfhust)
- [L-maple](https://github.com/L-maple)
- [lqs](https://github.com/lqs)
- [pingandb](https://github.com/pingandb)
- [yorkhellen](https://github.com/yorkhellen)
- [yujiarista](https://github.com/yujiarista)（初めての貢献者）
