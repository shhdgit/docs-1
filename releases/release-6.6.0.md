---
title: TiDB 6.6.0 Release Notes
summary: Learn about the new features, compatibility changes, improvements, and bug fixes in TiDB 6.6.0.
---

# TiDB 6.6.0 リリースノート {#tidb-6-6-0-release-notes}

リリース日: 2023年2月20日

TiDB バージョン: 6.6.0-[DMR](/releases/versioning.md#development-milestone-releases)

> **Note:**
>
> TiDB 6.6.0-DMR のドキュメントは[アーカイブ](https://docs-archive.pingcap.com/tidb/v6.6/)されました。PingCAPは、TiDBデータベースの[最新のLTSバージョン](https://docs.pingcap.com/tidb/stable)を使用することをお勧めします。

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.6/quick-start-with-tidb) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.6.0#version-list)

v6.6.0-DMR では、主な新機能と改善点は以下の通りです:

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
    <td rowspan="3">拡張性とパフォーマンス<br /></td>
    <td>TiKVは<a href="https://docs.pingcap.com/tidb/v6.6/partitioned-raft-kv" target="_blank">Partitioned Raft KV storage engine</a> (実験的)</td>
    <td>TiKVはPartitioned Raft KVストレージエンジンを導入し、各リージョンが独立したRocksDBインスタンスを使用し、クラスタのストレージ容量をTBからPBに簡単に拡張し、より安定した書き込みレイテンシーと強力な拡張性を提供できます。</td>
  </tr>
  <tr>
    <td>TiKVは<a href="https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_store_batch_size" target="_blank">バッチ集約データリクエスト</a>をサポート</td>
    <td>この強化により、TiKVバッチ取得操作の総RPC数が大幅に削減されます。データが高度に分散しており、gRPCスレッドプールのリソースが不足している状況では、バッチコプロセッサリクエストを使用することで、パフォーマンスが50%以上向上することがあります。</td>
  </tr>
  <tr>
    <td>TiFlashは<a href="https://docs.pingcap.com/tidb/v6.6/stale-read" target="_blank">Stale Read</a>と<a href="https://docs.pingcap.com/tidb/v6.6/explain-mpp#mpp-version-and-exchange-data-compression" target="_blank">圧縮交換</a>をサポート</td>
    <td>TiFlashはステイル読み取り機能をサポートし、リアルタイム要件が制限されていないシナリオでのクエリパフォーマンスを向上させます。また、データの圧縮をサポートし、並列データ交換の効率を向上させ、全体的なTPC-Hパフォーマンスが10%向上し、ネットワーク使用量を50%以上節約できます。</td>
  </tr>
  <tr>
    <td rowspan="2">信頼性と可用性<br /></td>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/tidb-resource-control" target="_blank">リソース制御</a> (実験的)</td>
    <td>実際のニーズに基づいてデータベースユーザーを対応するリソースグループにマッピングし、各リソースグループに対してクォータを設定するリソース管理をサポートします。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/sql-plan-management#create-a-binding-according-to-a-historical-execution-plan" target="_blank">履歴SQLバインディング</a>をサポート</td>
    <td>履歴実行計画をバインドし、TiDBダッシュボードで迅速に実行計画をバインドすることをサポートします。</td>
  </tr>
  <tr>
    <td rowspan="2">SQL機能<br /></td>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/foreign-key" target="_blank">外部キー</a> (実験的)</td>
    <td>MySQL互換の外部キー制約をサポートし、データの整合性を維持し、データ品質を向上させます。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/sql-statement-create-index#multi-valued-indexes" target="_blank">多値インデックス</a> (実験的)</td>
    <td>MySQL互換の多値インデックスを導入し、JSONタイプを強化して、MySQL 8.0との互換性を向上させます。</td>
  </tr>
  <tr>
    <td>DB操作と可観測性<br /></td>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/dm-precheck#check-items-for-physical-import" target="_blank">DMは物理インポートをサポート</a> (実験的)</td>
    <td>TiDBデータ移行（DM）は、TiDB Lightningの物理インポートモードを統合し、フルデータ移行のパフォーマンスを最大10倍向上させます。</td>
  </tr>
</tbody>
</table>

## 機能の詳細 {#feature-details}

### 拡張性 {#scalability}

- Partitioned Raft KVストレージエンジンをサポート (実験的) [#11515](https://github.com/tikv/tikv/issues/11515) [#12842](https://github.com/tikv/tikv/issues/12842) @[busyjay](https://github.com/busyjay) @[tonyxuqqi](https://github.com/tonyxuqqi) @[tabokie](https://github.com/tabokie) @[bufferflies](https://github.com/bufferflies) @[5kbpers](https://github.com/5kbpers) @[SpadeA-Tang](https://github.com/SpadeA-Tang) @[nolouch](https://github.com/nolouch)

  TiDB v6.6.0以前では、TiKVのRaftベースのストレージエンジンは、TiKVインスタンスのすべての「リージョン」のデータを1つのRocksDBインスタンスに格納していました。より大規模なクラスタをより安定してサポートするために、TiDB v6.6.0からは、複数のRocksDBインスタンスを使用してTiKVリージョンデータを格納する新しいTiKVストレージエンジンが導入されました。各リージョンのデータは独立したRocksDBインスタンスに個別に格納されます。新しいエンジンでは、RocksDBインスタンス内のファイルの数とレベルをよりよく制御し、リージョン間のデータ操作を物理的に分離し、より多くのデータを安定して管理できます。これは、TiKVがパーティションを介して複数のRocksDBインスタンスを管理していると見なすことができます。この機能の主な利点は、より良い書き込みパフォーマンス、より速いスケーリング、および同じハードウェアでサポートされるデータの大容量です。また、より大規模なクラスタスケールをサポートできます。

  現在、この機能は実験的であり、本番環境での使用は推奨されていません。

  詳細については、[ドキュメント](/partitioned-raft-kv.md)を参照してください。

- DDL操作の分散並列実行フレームワークをサポート (実験的) [#37125](https://github.com/pingcap/tidb/issues/37125) @[zimulala](https://github.com/zimulala)

  以前のバージョンでは、TiDBクラスタ全体の1つのTiDBインスタンスのみがDDLオーナーとしてスキーマ変更タスクを処理することが許可されていました。大規模なテーブルのDDL操作のDDL並行性をさらに向上させるために、TiDB v6.6.0では、DDLの分散並列実行フレームワークが導入されました。これにより、クラスタ内のすべてのTiDBインスタンスが同じタスクの`StateWriteReorganization`フェーズを同時に実行できるようになり、DDLの実行を高速化できます。この機能は、システム変数[`tidb_ddl_distribute_reorg`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_ddl_distribute_reorg-new-in-v660)によって制御され、現在は`Add Index`操作のみをサポートしています。

### パフォーマンス {#performance}

- 悲観的ロックキューの安定したウェイクアップモデルをサポート [#13298](https://github.com/tikv/tikv/issues/13298) @[MyonKeminta](https://github.com/MyonKeminta)

  アプリケーションが頻繁に単一ポイントの悲観的ロックの競合に遭遇した場合、既存のウェイクアップメカニズムではトランザクションがロックを取得する時間を保証できず、高いロングテールのレイテンシーやロックの取得タイムアウトを引き起こします。v6.6.0から、システム変数[`tidb_pessimistic_txn_aggressive_locking`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_pessimistic_txn_aggressive_locking-new-in-v660)の値を`ON`に設定することで、悲観的ロックのための安定したウェイクアップモデルを有効にできます。このウェイクアップモデルでは、キューのウェイクアップシーケンスを厳密に制御して、無効なウェイクアップによるリソースの浪費を避けることができます。深刻なロックの競合があるシナリオでは、安定したウェイクアップモデルにより、ロングテールのレイテンシーやP99の応答時間を削減できます。

  テストによると、これによりテールレイテンシーが40〜60%削減されます。

  詳細については、[ドキュメント](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_pessimistic_txn_aggressive_locking-new-in-v660)を参照してください。

- バッチ集約データリクエスト [#39361](https://github.com/pingcap/tidb/issues/39361) @[cfzjywxk](https://github.com/cfzjywxk) @[you06](https://github.com/you06)

  TiDBがTiKVにデータリクエストを送信すると、TiDBはデータが配置されているリージョンに応じてリクエストを異なるサブタスクにコンパイルし、各サブタスクは単一のリージョンのリクエストのみを処理します。アクセスするデータが非常に分散している場合、データのサイズが大きくなくても、多くのサブタスクが生成され、それにより多くのRPCリクエストが生成され、余分な時間がかかります。v6.6.0から、TiDBは同じTiKVインスタンスに送信されるデータリクエストを部分的にマージすることをサポートし、サブタスクの数とRPCリクエストのオーバーヘッドを削減します。データが高度に分散しており、gRPCスレッドプールリソースが不足している場合、リクエストのバッチ処理によりパフォーマンスが50%以上向上することがあります。

  この機能はデフォルトで有効です。リクエストのバッチサイズは、システム変数[`tidb_store_batch_size`](/system-variables.md#tidb_store_batch_size)を使用して設定できます。

- `LIMIT`句の制限を解除 [#40219](https://github.com/pingcap/tidb/issues/40219) @[fzzf678](https://github.com/fzzf678)

  v6.6.0から、TiDBプランキャッシュは`LIMIT`パラメータとして変数を持つ実行プランをキャッシュすることをサポートします。たとえば、`LIMIT ?`または`LIMIT 10, ?`などです。この機能により、より多くのSQLステートメントがプランキャッシュの恩恵を受けることができ、実行効率が向上します。現在、セキュリティ上の理由から、TiDBは`?`が10000を超えない実行プランのキャッシュのみをサポートしています。

  詳細については、[ドキュメント](/sql-prepared-plan-cache.md)を参照してください。

- TiFlashは圧縮を使用したデータ交換をサポート [#6620](https://github.com/pingcap/tiflash/issues/6620) @[solotzg](https://github.com/solotzg)

  複数のノードでの計算をサポートするために、TiFlashエンジンは異なるノード間でデータを交換する必要があります。交換するデータのサイズが非常に大きい場合、データ交換のパフォーマンスが全体の計算効率に影響を与える可能性があります。v6.6.0では、TiFlashエンジンは必要に応じて交換するデータを圧縮する圧縮メカニズムを導入し、その後交換を行うことで、データ交換の効率を向上させます。

  詳細については、[ドキュメント](/explain-mpp.md#mpp-version-and-exchange-data-compression)を参照してください。

- TiFlashはステイル読み取り機能をサポート [#4483](https://github.com/pingcap/tiflash/issues/4483) @[hehechen](https://github.com/hehechen)

  ステイル読み取り機能はv5.1.1以降一般利用可能（GA）となっており、特定のタイムスタンプでまたは指定された時間範囲内で過去のデータを読み取ることができます。ステイル読み取りは、ローカルのTiKVレプリカからデータを読み取ることで、読み取りのレイテンシーを削減し、クエリのパフォーマンスを向上させることができます。v6.6.0以前、TiFlashはステイル読み取りをサポートしていませんでした。テーブルにTiFlashレプリカがある場合でも、ステイル読み取りはそのTiKVレプリカのみを読み取ることができました。

  v6.6.0から、TiFlashはステイル読み取り機能をサポートします。テーブルの過去のデータを[`AS OF TIMESTAMP`](/as-of-timestamp.md)構文または[`tidb_read_staleness`](/tidb-read-staleness.md)システム変数を使用してクエリする場合、テーブルにTiFlashレプリカがある場合、オプティマイザは対応するデータをTiFlashレプリカから読み取ることを選択できるようになり、クエリのパフォーマンスをさらに向上させることができます。

  詳細については、[ドキュメント](/stale-read.md)を参照してください。

- `regexp_replace`文字列関数をTiFlashにプッシュダウンする機能をサポート [#6115](https://github.com/pingcap/tiflash/issues/6115) @[xzhangxian1008](https://github.com/xzhangxian1008)

### 信頼性 {#reliability}

- リソースグループに基づいたリソース制御をサポート（実験的）[#38825](https://github.com/pingcap/tidb/issues/38825) @[nolouch](https://github.com/nolouch) @[BornChanger](https://github.com/BornChanger) @[glorv](https://github.com/glorv) @[tiancaiamao](https://github.com/tiancaiamao) @[Connor1996](https://github.com/Connor1996) @[JmPotato](https://github.com/JmPotato) @[hnes](https://github.com/hnes) @[CabinfeverB](https://github.com/CabinfeverB) @[HuSharp](https://github.com/HuSharp)

  これで、TiDBクラスターにリソースグループを作成し、異なるデータベースユーザーを対応するリソースグループにバインドし、実際のニーズに応じて各リソースグループにクォータを設定できます。クラスターリソースが限られている場合、同じリソースグループ内のセッションによって使用されるすべてのリソースはクォータに制限されます。このようにして、リソースグループが過度に消費されても、他のリソースグループのセッションに影響を与えません。TiDBは、Grafanaダッシュボード上でリソースの実際の使用状況のビューを提供し、リソースをより合理的に割り当てるのを支援します。

  リソース制御機能の導入は、TiDBにとっての画期的な出来事です。これにより、分散データベースクラスターを複数の論理ユニットに分割できます。個々のユニットがリソースを過度に使用しても、他のユニットが必要とするリソースを圧迫することはありません。

  この機能を使用すると、次のことができます。

  - 異なるシステムからの複数の小規模および中規模のアプリケーションを1つのTiDBクラスターに組み合わせることができます。アプリケーションのワークロードが大きくなっても、他のアプリケーションの正常な動作に影響を与えません。システムのワークロードが低い場合、設定された読み取りおよび書き込みクォータを超えても、ビジーなアプリケーションに必要なシステムリソースを割り当てることができ、リソースの最大利用を実現できます。
  - すべてのテスト環境を1つのTiDBクラスターに組み合わせるか、より多くのリソースを消費するバッチタスクを1つのリソースグループにグループ化することができます。これにより、ハードウェアの利用率が向上し、運用コストが削減される一方で、重要なアプリケーションが常に必要なリソースを取得できるようになります。

  また、リソース制御機能の合理的な使用は、クラスターの数を減らし、運用および保守の難しさを緩和し、管理コストを節約できます。

  v6.6では、リソース制御を有効にするには、TiDBのグローバル変数[`tidb_enable_resource_control`](/system-variables.md#tidb_enable_resource_control-new-in-v660)とTiKVの構成項目[`resource-control.enabled`](/tikv-configuration-file.md#resource-control)の両方を有効にする必要があります。現在、サポートされているクォータメソッドは、"[リクエストユニット（RU）](/tidb-resource-control.md#what-is-request-unit-ru)"に基づいています。RUは、CPUやIOなどのシステムリソースのTiDBの統一された抽象単位です。

  詳細については、[ドキュメント](/tidb-resource-control.md)を参照してください。

- 履歴実行計画のバインディングがGAになりました[#39199](https://github.com/pingcap/tidb/issues/39199) @[fzzf678](https://github.com/fzzf678)

  v6.5.0では、TiDBは[`CREATE [GLOBAL | SESSION] BINDING`](/sql-statements/sql-statement-create-binding.md)ステートメントのバインディングターゲットを拡張し、履歴実行計画に従ってバインディングを作成することをサポートしています。v6.6.0では、この機能がGAになりました。実行計画の選択は、現在のTiDBノードに限定されません。任意のTiDBノードで生成された任意の履歴実行計画を[SQLバインディング](/sql-statements/sql-statement-create-binding.md)のターゲットとして選択できます。これにより、機能の利便性がさらに向上します。

  詳細については、[ドキュメント](/sql-plan-management.md#create-a-binding-according-to-a-historical-execution-plan)を参照してください。

- いくつかの最適化ヒントを追加しました[#39964](https://github.com/pingcap/tidb/issues/39964) @[Reminiscent](https://github.com/Reminiscent)

  TiDBはv6.6.0でいくつかの最適化ヒントを追加し、`LIMIT`操作の実行計画選択を制御します。

  - [`ORDER_INDEX()`](/optimizer-hints.md#order_indext1_name-idx1_name--idx2_name-)：指定されたインデックスを使用し、データの読み取り時にインデックスの順序を保持し、`Limit + IndexScan(keep order: true)`に類似した計画を生成するようにオプティマイザに指示します。
  - [`NO_ORDER_INDEX()`](/optimizer-hints.md#no_order_indext1_name-idx1_name--idx2_name-)：指定されたインデックスを使用し、データの読み取り時にインデックスの順序を保持しないようにオプティマイザに指示し、`TopN + IndexScan(keep order: false)`に類似した計画を生成します。

  継続的に最適化ヒントを導入することで、ユーザーにより多くの介入方法を提供し、SQLパフォーマンスの問題を解決し、全体的なパフォーマンスの安定性を向上させます。

- DDL操作のリソース使用の動的管理をサポート（実験的）[#38025](https://github.com/pingcap/tidb/issues/38025) @[hawkingrei](https://github.com/hawkingrei)

  TiDB v6.6.0では、DDL操作のリソース管理を導入し、これらの操作のCPU使用率を自動的に制御することで、オンラインアプリケーションへのDDL変更の影響を軽減します。この機能は、[DDL分散並列実行フレームワーク](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_ddl_distribute_reorg-new-in-v660)が有効になっている場合にのみ有効です。

### 可用性 {#availability}

- [SQLの配置ルール](/placement-rules-in-sql.md)に`SURVIVAL_PREFERENCE`を設定するサポートを追加しました[#38605](https://github.com/pingcap/tidb/issues/38605) @[nolouch](https://github.com/nolouch)

  `SURVIVAL_PREFERENCES`は、データの災害耐久性を高めるためのデータの生存優先設定を提供します。`SURVIVAL_PREFERENCE`を指定することで、次のことを制御できます。

  - クラウドリージョン全体にわたって展開されたTiDBクラスターの場合、クラウドリージョンが障害が発生した場合、指定されたデータベースまたはテーブルは別のクラウドリージョンで生存できます。
  - 単一のクラウドリージョンに展開されたTiDBクラスターの場合、可用性ゾーンが障害が発生した場合、指定されたデータベースまたはテーブルは別の可用性ゾーンで生存できます。

  詳細については、[ドキュメント](/placement-rules-in-sql.md#survival-preferences)を参照してください。

- `FLASHBACK CLUSTER TO TIMESTAMP`ステートメントを使用したDDL操作のロールバックをサポートしました[#14088](https://github.com/tikv/tikv/pull/14088) @[Defined2014](https://github.com/Defined2014) @[JmPotato](https://github.com/JmPotato)

  [`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-cluster.md)ステートメントは、ガベージコレクション（GC）ライフタイム内の指定された時点までクラスターを復元することをサポートし、TiDB v6.6.0では、この機能がDDL操作のロールバックをサポートするようになりました。これにより、クラスター上でのDMLまたはDDLの誤操作を素早く元に戻し、数分でクラスターをロールバックし、タイムライン上でクラスターを複数回ロールバックして、特定のデータ変更がいつ発生したかを判断することができます。

  詳細については、[ドキュメント](/sql-statements/sql-statement-flashback-cluster.md)を参照してください。

### SQL {#sql}

- MySQL互換の外部キー制約をサポート（実験的）[#18209](https://github.com/pingcap/tidb/issues/18209) @[crazycs520](https://github.com/crazycs520)

  TiDB v6.6.0では、MySQLと互換性のある外部キー制約機能を導入しました。この機能は、テーブル内またはテーブル間の参照、制約の検証、およびカスケード操作をサポートします。この機能により、アプリケーションをTiDBに移行し、データの整合性を維持し、データ品質を向上し、データモデリングを容易にすることができます。

  詳細については、[ドキュメント](/foreign-key.md)を参照してください。

- MySQL互換の多値インデックスをサポート（実験的）[#39592](https://github.com/pingcap/tidb/issues/39592) @[xiongjiwei](https://github.com/xiongjiwei) @[qw4990](https://github.com/qw4990)

  TiDBはv6.6.0でMySQL互換の多値インデックスを導入しました。JSONカラム内の配列の値をフィルタリングすることは一般的な操作ですが、通常のインデックスではこのような操作の高速化に役立ちません。JSONカラムの配列に多値インデックスを作成すると、フィルタリングパフォーマンスが大幅に向上します。JSONカラム内の配列に多値インデックスがある場合、`MEMBER OF()`、`JSON_CONTAINS()`、`JSON_OVERLAPS()`関数を使用してフィルタリング条件をフィルタリングでき、I/O消費を大幅に削減し、操作速度を向上させることができます。

  多値インデックスの導入により、TiDBのJSONデータ型へのサポートがさらに強化され、MySQL 8.0との互換性が向上します。

  詳細については、[ドキュメント](/sql-statements/sql-statement-create-index.md#multi-valued-indexes)を参照してください。

### DB operations {#db-operations}

- サポートを提供する：リソースを消費するタスクのための読み取り専用ストレージノードの構成 @[v01dstar](https://github.com/v01dstar)

  本番環境では、一部の読み取り専用操作が定期的に多くのリソースを消費し、バックアップや大規模なデータの読み取りと解析など、クラスタ全体のパフォーマンスに影響を与えることがあります。TiDB v6.6.0では、読み取り専用ストレージノードをリソースを消費する読み取り専用タスクのために構成することで、オンラインアプリケーションへの影響を軽減することができます。現在、TiDB、TiSpark、BRは読み取り専用ストレージノードからデータを読み取ることをサポートしています。システム変数 `tidb_replica_read`、TiSparkの構成項目 `spark.tispark.replica_read`、またはbrコマンドライン引数 `--replica-read-label` を使用して、データの読み取り先を指定することで、クラスタのパフォーマンスの安定性を確保できます。

  詳細については、[ドキュメント](/best-practices/readonly-nodes.md) を参照してください。

- `store-io-pool-size` を動的に変更するサポート [#13964](https://github.com/tikv/tikv/issues/13964) @[LykxSassinator](https://github.com/LykxSassinator)

  TiKVの構成項目 [`raftstore.store-io-pool-size`](/tikv-configuration-file.md#store-io-pool-size-new-in-v530) は、Raft I/O タスクを処理するスレッドの許容可能な数を指定します。これは、TiKVのパフォーマンスを調整する際に調整できます。v6.6.0以前では、この構成項目は動的に変更できませんでした。v6.6.0からは、サーバーを再起動することなくこの構成を変更できるようになり、より柔軟なパフォーマンス調整が可能です。

  詳細については、[ドキュメント](/dynamic-config.md) を参照してください。

- TiDBクラスタの初期化時に実行されるSQLスクリプトを指定するサポート [#35624](https://github.com/pingcap/tidb/issues/35624) @[morgo](https://github.com/morgo)

  TiDBクラスタを初めて起動する際に、コマンドラインパラメータ `--initialize-sql-file` を構成することで、実行されるSQLスクリプトを指定できます。システム変数の値を変更したり、ユーザーを作成したり、権限を付与するなどの操作が必要な場合に、この機能を使用できます。

  詳細については、[ドキュメント](/tidb-configuration-file.md#initialize-sql-file-new-in-v660) を参照してください。

- TiDBデータ移行（DM）は、フル移行のためにTiDB Lightningの物理インポートモードを統合し、パフォーマンスを最大10倍向上させる（実験的） @[lance6716](https://github.com/lance6716)

  v6.6.0では、DMのフル移行機能がTiDB Lightningの物理インポートモードと統合され、DMがフルデータ移行のパフォーマンスを最大10倍向上させることができます。これにより、大容量データのシナリオでの移行時間を大幅に短縮できます。

  v6.6.0以前では、大容量データのシナリオでは、TiDB Lightningの物理インポートタスクを別途構成して高速なフルデータ移行を行い、その後にDMを使用して増分データ移行を行う必要がありました。これは複雑な構成でした。v6.6.0からは、TiDB Lightningタスクを構成する必要なく、1つのDMタスクで大容量データを移行できます。

  詳細については、[ドキュメント](/dm/dm-precheck.md#check-items-for-physical-import) を参照してください。

- TiDB Lightningは、ソースファイルとターゲットテーブルの列名の不一致の問題に対処するために新しい構成パラメータ `"header-schema-match"` を追加しました @[dsdashun](https://github.com/dsdashun)

  v6.6.0では、TiDB Lightningに新しいプロファイルパラメータ `"header-schema-match"` が追加されました。デフォルト値は `true` で、これはソースCSVファイルの最初の行を列名として扱い、ターゲットテーブルと一致させることを意味します。CSVテーブルヘッダーのフィールド名がターゲットテーブルの列名と一致しない場合、この構成を `false` に設定できます。TiDB Lightningはエラーを無視し、ターゲットテーブルの列の順序でデータをインポートし続けます。

  詳細については、[ドキュメント](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) を参照してください。

- TiDB Lightningは、TiKVにキー値ペアを送信する際に圧縮転送を有効にすることをサポートします [#41163](https://github.com/pingcap/tidb/issues/41163) @[gozssky](https://github.com/gozssky)

  v6.6.0から、TiDB Lightningは、ローカルでエンコードおよびソートされたキー値ペアを圧縮してTiKVに送信することをサポートし、ネットワーク経由で転送されるデータ量を減らし、ネットワーク帯域幅のオーバーヘッドを低減します。この機能はデフォルトで無効です。有効にするには、TiDB Lightningの `compress-kv-pairs` 構成項目を `"gzip"` または `"gz"` に設定できます。

  詳細については、[ドキュメント](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) を参照してください。

- TiKV-CDCツールは現在GAであり、RawKVのデータ変更に対するサブスクリプションをサポートします [#48](https://github.com/tikv/migration/issues/48) @[zeminzhou](https://github.com/zeminzhou) @[haojinming](https://github.com/haojinming) @[pingyu](https://github.com/pingyu)

  TiKV-CDCは、TiKVクラスタのためのCDC（Change Data Capture）ツールです。TiKVとPDは、TiDBを使用せずにKVデータベースを構成できます。これをRawKVと呼びます。TiKV-CDCは、RawKVのデータ変更をサブスクライブし、リアルタイムでダウンストリームのTiKVクラスタにレプリケートすることができます。これにより、RawKVのクロスクラスタレプリケーションが可能になります。

  詳細については、[ドキュメント](https://tikv.org/docs/latest/concepts/explore-tikv-features/cdc/cdc/) を参照してください。

- TiCDCは、Kafka changefeeds上の単一のテーブルのスケーリングアウトと、複数のTiCDCノードにchangefeedを分散することをサポートします（実験的） [#7720](https://github.com/pingcap/tiflow/issues/7720) @[overvenus](https://github.com/overvenus)

  v6.6.0以前では、上流のテーブルが大量の書き込みを受け入れる場合、このテーブルのレプリケーション能力をスケーリングアウトすることができず、レプリケーションの遅延が増加することがありました。TiCDC v6.6.0からは、上流テーブルのchangefeedをKafkaシンクの複数のTiCDCノードに分散することができるようになり、単一のテーブルのレプリケーション能力をスケーリングアウトできます。

  詳細については、[ドキュメント](/ticdc/ticdc-sink-to-kafka.md#scale-out-the-load-of-a-single-large-table-to-multiple-ticdc-nodes) を参照してください。

- [GORM](https://github.com/go-gorm/gorm) はTiDB統合テストを追加しました。現在、TiDBはGORMがデフォルトでサポートするデータベースです。[#6014](https://github.com/go-gorm/gorm/pull/6014) @[Icemap](https://github.com/Icemap)

  - v1.4.6では、[GORM MySQLドライバ](https://github.com/go-gorm/mysql) がTiDBの `AUTO_RANDOM` 属性に適合しました [#104](https://github.com/go-gorm/mysql/pull/104)
  - v1.4.6では、[GORM MySQLドライバ](https://github.com/go-gorm/mysql) がTiDBに接続する際に、`Unique` フィールドの `Unique` 属性を `AutoMigrate` 中に変更できない問題を修正しました [#105](https://github.com/go-gorm/mysql/pull/105)
  - [GORMドキュメント](https://github.com/go-gorm/gorm.io) がTiDBをデフォルトのデータベースとして言及しています [#638](https://github.com/go-gorm/gorm.io/pull/638)

  詳細については、[GORMドキュメント](https://gorm.io/docs/index.html) を参照してください。

### 観測可能性 {#observability}

- TiDB v6.6.0は、ステートメント履歴からSQLバインディングを迅速に作成することをサポートします。これにより、TiDBダッシュボードでSQLステートメントを特定のプランに迅速にバインドできます。

  ユーザーフレンドリーなインターフェースを提供することで、この機能はTiDBでのプランのバインディングプロセスを簡素化し、操作の複雑さを減らし、効率とユーザーエクスペリエンスを向上させます。

  詳細については、[ドキュメント](/dashboard/dashboard-statement-details.md#fast-plan-binding)を参照してください。

- キャッシュ実行プランに警告を追加しました。実行プランをキャッシュできない場合、TiDBは警告で理由を示し、診断を容易にします。例：

  ```sql
  mysql> PREPARE st FROM 'SELECT * FROM t WHERE a<?';
  Query OK, 0 rows affected (0.00 sec)

  mysql> SET @a='1';
  Query OK, 0 rows affected (0.00 sec)

  mysql> EXECUTE st USING @a;
  Empty set, 1 warning (0.01 sec)

  mysql> SHOW WARNINGS;
  +---------+------+----------------------------------------------+
  | Level   | Code | Message                                      |
  +---------+------+----------------------------------------------+
  | Warning | 1105 | skip plan-cache: '1' may be converted to INT |
  +---------+------+----------------------------------------------+
  ```

  上記の例では、最適化プログラムが非INT型をINT型に変換し、パラメータの変更に伴って実行プランが変更される可能性があるため、TiDBはプランをキャッシュしません。

  詳細については、[ドキュメント](/sql-prepared-plan-cache.md#diagnostics-of-prepared-plan-cache)を参照してください。

- スロークエリログに`Warnings`フィールドを追加しました。このフィールドは、スロークエリの実行中に生成された警告を記録し、パフォーマンスの問題を診断するのに役立ちます。また、TiDBダッシュボードのスロークエリページで警告を表示できます。

  詳細については、[ドキュメント](/identify-slow-queries.md)を参照してください。

- SQL実行プランの生成を自動的にキャプチャする機能を追加しました。トラブルシューティング実行プランの問題の際、`PLAN REPLAYER`はシーンを保存し、診断の効率を向上させるのに役立ちます。ただし、一部のシナリオでは、いくつかの実行プランの生成が自由に再現できないため、診断作業がより困難になります。

  このような問題に対処するために、TiDB v6.6.0では、`PLAN REPLAYER`が自動キャプチャの機能を拡張しました。`PLAN REPLAYER CAPTURE`コマンドを使用すると、事前にターゲットSQLステートメントを登録し、同時にターゲット実行プランを指定できます。TiDBが登録されたターゲットに一致するSQLステートメントまたは実行プランを検出すると、自動的に`PLAN REPLAYER`情報を生成してパッケージ化します。実行プランが不安定な場合、この機能は診断の効率を向上させることができます。

  この機能を使用するには、[`tidb_enable_plan_replayer_capture`](/system-variables.md#tidb_enable_plan_replayer_capture)の値を`ON`に設定してください。

  詳細については、[ドキュメント](/sql-plan-replayer.md#use-plan-replayer-capture)を参照してください。

- ステートメントサマリーの永続化をサポートしました（実験的）。v6.6.0以前では、ステートメントサマリーデータはメモリに保持され、TiDBサーバーの再起動時に失われます。v6.6.0以降、TiDBはステートメントサマリーの永続化をサポートし、履歴データを定期的にディスクに書き込むことができます。同時に、システムテーブルのクエリ結果はメモリではなくディスクから派生します。TiDBが再起動した後も、すべての履歴データが利用可能です。

  詳細については、[ドキュメント](/statement-summary-tables.md#persist-statements-summary)を参照してください。

### セキュリティ {#security}

- TiFlashはTLS証明書の自動ローテーションをサポートします。v6.6.0では、TiDBはTiFlash TLS証明書の自動ローテーションをサポートします。コンポーネント間の暗号化データ転送を有効にしたTiDBクラスターでは、TiFlash TLS証明書の有効期限が切れて新しい証明書を再発行する必要がある場合、新しいTiFlash TLS証明書をTiDBクラスターを再起動せずに自動的にロードできます。また、TiDBクラスター内のコンポーネント間でTLS証明書をローテーションしても、TiDBクラスターの利用に影響を与えません。これにより、クラスターの高可用性が確保されます。

  詳細については、[ドキュメント](/enable-tls-between-components.md)を参照してください。

- TiDB LightningはAWS IAMロールキーとセッショントークンを使用してAmazon S3データにアクセスすることをサポートします。v6.6.0以前では、TiDB LightningはAWS IAM **ユーザーのアクセスキー**（各アクセスキーはアクセスキーIDとシークレットアクセスキーで構成されています）を使用してS3データにアクセスすることしかサポートしていませんでした。v6.6.0以降、TiDB LightningはAWS IAM **ロールのアクセスキー+セッショントークン**を使用してS3データにアクセスすることもサポートし、データのセキュリティを向上させます。

  詳細については、[ドキュメント](/tidb-lightning/tidb-lightning-data-source.md#import-data-from-amazon-s3)を参照してください。

### テレメトリ {#telemetry}

- 2023年2月20日から、[テレメトリ機能](/telemetry.md)は、新しいバージョンのTiDBとTiDBダッシュボード（v6.6.0を含む）ではデフォルトで無効になります。デフォルトのテレメトリ構成を使用している以前のバージョンからアップグレードする場合、アップグレード後にテレメトリ機能が無効になります。詳細なバージョンについては、[TiDBリリースタイムライン](/releases/release-timeline.md)を参照してください。

- v1.11.3から、新しくデプロイされたTiUPでは、テレメトリ機能がデフォルトで無効になります。以前のバージョンのTiUPからv1.11.3またはそれ以降のバージョンにアップグレードする場合、テレメトリ機能はアップグレード前と同じ状態になります。

## 互換性の変更 {#compatibility-changes}

> **Note:**
>
> このセクションでは、v6.5.0から現在のバージョン（v6.6.0）にアップグレードする際に知っておく必要がある互換性の変更を提供します。v6.4.0またはそれ以前のバージョンから現在のバージョンにアップグレードする場合、中間バージョンで導入された互換性の変更も確認する必要がある場合があります。

### MySQL互換性 {#mysql-compatibility}

- MySQL互換の外部キー制約をサポートします（実験的）。詳細については、このドキュメントの[SQL](#sql)セクションと[ドキュメント](/foreign-key.md)を参照してください。

- MySQL互換の多値インデックスをサポートします（実験的）。詳細については、このドキュメントの[SQL](#sql)セクションと[ドキュメント](/sql-statements/sql-statement-create-index.md#multi-valued-indexes)を参照してください。

### システム変数 {#system-variables}

| Variable name                                                                                                                                        | Change type | Description                                                                                                                                                                                                                                                                                                                                                                                             |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tidb_enable_amend_pessimistic_txn`                                                                                                                  | Deleted     | Starting from v6.5.0, this variable is deprecated. Starting from v6.6.0, this variable and the `AMEND TRANSACTION` feature are deleted. TiDB will use [meta lock](/metadata-lock.md) to avoid the `Information schema is changed` error.                                                                                                                                                                |
| `tidb_enable_concurrent_ddl`                                                                                                                         | Deleted     | This variable controls whether to allow TiDB to use concurrent DDL statements. When this variable is disabled, TiDB uses the old DDL execution framework, which provides limited support for concurrent DDL execution. Starting from v6.6.0, this variable is deleted and TiDB no longer supports the old DDL execution framework.                                                                      |
| `tidb_ttl_job_run_interval`                                                                                                                          | Deleted     | This variable is used to control the scheduling interval of TTL jobs in the background. Starting from v6.6.0, this variable is deleted, because TiDB provides the `TTL_JOB_INTERVAL` attribute for every table to control the TTL runtime, which is more flexible than `tidb_ttl_job_run_interval`.                                                                                                     |
| [`foreign_key_checks`](/system-variables.md#foreign_key_checks)                                                                                      | Modified    | This variable controls whether to enable the foreign key constraint check. The default value changes from `OFF` to `ON`, which means enabling the foreign key check by default.                                                                                                                                                                                                                         |
| [`tidb_enable_foreign_key`](/system-variables.md#tidb_enable_foreign_key-new-in-v630)                                                                | Modified    | This variable controls whether to enable the foreign key feature. The default value changes from `OFF` to `ON`, which means enabling foreign key by default.                                                                                                                                                                                                                                            |
| `tidb_enable_general_plan_cache`                                                                                                                     | Modified    | This variable controls whether to enable General Plan Cache. Starting from v6.6.0, this variable is renamed to [`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache).                                                                                                                                                                                       |
| [`tidb_enable_historical_stats`](/system-variables.md#tidb_enable_historical_stats)                                                                  | Modified    | This variable controls whether to enable historical statistics. The default value changes from `OFF` to `ON`, which means that historical statistics are enabled by default.                                                                                                                                                                                                                            |
| [`tidb_enable_telemetry`](/system-variables.md#tidb_enable_telemetry-new-in-v402)                                                                    | Modified    | The default value changes from `ON` to `OFF`, which means that telemetry is disabled by default in TiDB.                                                                                                                                                                                                                                                                                                |
| `tidb_general_plan_cache_size`                                                                                                                       | Modified    | This variable controls the maximum number of execution plans that can be cached by General Plan Cache. Starting from v6.6.0, this variable is renamed to [`tidb_non_prepared_plan_cache_size`](/system-variables.md#tidb_non_prepared_plan_cache_size).                                                                                                                                                 |
| [`tidb_replica_read`](/system-variables.md#tidb_replica_read-new-in-v40)                                                                             | Modified    | A new value option `learner` is added for this variable to specify the learner replicas with which TiDB reads data from read-only nodes.                                                                                                                                                                                                                                                                |
| [`tidb_replica_read`](/system-variables.md#tidb_replica_read-new-in-v40)                                                                             | Modified    | A new value option `prefer-leader` is added for this variable to improve the overall read availability of TiDB clusters. When this option is set, TiDB prefers to read from the leader replica. When the performance of the leader replica significantly decreases, TiDB automatically reads from follower replicas.                                                                                    |
| [`tidb_store_batch_size`](/system-variables.md#tidb_store_batch_size)                                                                                | Modified    | This variable controls the batch size of the Coprocessor Tasks of the `IndexLookUp` operator. `0` means to disable batch. Starting from v6.6.0, the default value is changed from `0` to `4`, which means 4 Coprocessor tasks will be batched into one task for each batch of requests.                                                                                                                 |
| [`mpp_exchange_compression_mode`](/system-variables.md#mpp_exchange_compression_mode-new-in-v660)                                                    | Newly added | This variable specifies the data compression mode of the MPP Exchange operator. It takes effect when TiDB selects the MPP execution plan with the version number `1`. The default value `UNSPECIFIED` means that TiDB automatically selects the `FAST` compression mode.                                                                                                                                |
| [`mpp_version`](/system-variables.md#mpp_version-new-in-v660)                                                                                        | Newly added | This variable specifies the version of the MPP execution plan. After a version is specified, TiDB selects the specified version of the MPP execution plan. The default value `UNSPECIFIED` means that TiDB automatically selects the latest version `1`.                                                                                                                                                |
| [`tidb_ddl_distribute_reorg`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_ddl_distribute_reorg-new-in-v660)                             | Newly added | This variable controls whether to enable distributed execution of the DDL reorg phase to accelerate this phase. The default value `OFF` means not to enable distributed execution of the DDL reorg phase by default. Currently, this variable takes effect only for `ADD INDEX`.                                                                                                                        |
| [`tidb_enable_historical_stats_for_capture`](/system-variables.md#tidb_enable_historical_stats_for_capture)                                          | Newly added | This variable controls whether the information captured by `PLAN REPLAYER CAPTURE` includes historical statistics by default. The default value `OFF` means that historical statistics are not included by default.                                                                                                                                                                                     |
| [`tidb_enable_plan_cache_for_param_limit`](/system-variables.md#tidb_enable_plan_cache_for_param_limit-new-in-v660)                                  | Newly added | This variable controls whether Prepared Plan Cache caches execution plans that contain `COUNT` after `Limit`. The default value is `ON`, which means Prepared Plan Cache supports caching such execution plans. Note that Prepared Plan Cache does not support caching execution plans with a `COUNT` condition that counts a number greater than 10000.                                                |
| [`tidb_enable_plan_replayer_capture`](/system-variables.md#tidb_enable_plan_replayer_capture)                                                        | Newly added | This variable controls whether to enable the [`PLAN REPLAYER CAPTURE` feature](/sql-plan-replayer.md#use-plan-replayer-capture-to-capture-target-plans). The default value `OFF` means to disable the `PLAN REPLAYER CAPTURE` feature.                                                                                                                                                                  |
| [`tidb_enable_resource_control`](/system-variables.md#tidb_enable_resource_control-new-in-v660)                                                      | Newly added | This variable controls whether to enable the resource control feature. The default value is `OFF`. When this variable is set to `ON`, the TiDB cluster supports resource isolation of applications based on resource groups.                                                                                                                                                                            |
| [`tidb_historical_stats_duration`](/system-variables.md#tidb_historical_stats_duration-new-in-v660)                                                  | Newly added | This variable controls how long the historical statistics are retained in storage. The default value is 7 days.                                                                                                                                                                                                                                                                                         |
| [`tidb_index_join_double_read_penalty_cost_rate`](/system-variables.md#tidb_index_join_double_read_penalty_cost_rate-new-in-v660)                    | Newly added | This variable controls whether to add some penalty cost to the selection of index join. The default value `0` means that this feature is disabled by default.                                                                                                                                                                                                                                           |
| [`tidb_pessimistic_txn_aggressive_locking`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_pessimistic_txn_aggressive_locking-new-in-v660) | Newly added | This variable controls whether to use enhanced pessimistic locking wake-up model for pessimistic transactions. The default value `OFF` means not to use such a wake-up model for pessimistic transactions by default.                                                                                                                                                                                   |
| [`tidb_stmt_summary_enable_persistent`](/system-variables.md#tidb_stmt_summary_enable_persistent-new-in-v660)                                        | Newly added | This variable is read-only. It controls whether to enable [statements summary persistence](/statement-summary-tables.md#persist-statements-summary). The value of this variable is the same as that of the configuration item [`tidb_stmt_summary_enable_persistent`](/tidb-configuration-file.md#tidb_stmt_summary_enable_persistent-new-in-v660).                                                     |
| [`tidb_stmt_summary_filename`](/system-variables.md#tidb_stmt_summary_filename-new-in-v660)                                                          | Newly added | This variable is read-only. It specifies the file to which persistent data is written when [statements summary persistence](/statement-summary-tables.md#persist-statements-summary) is enabled. The value of this variable is the same as that of the configuration item [`tidb_stmt_summary_filename`](/tidb-configuration-file.md#tidb_stmt_summary_filename-new-in-v660).                           |
| [`tidb_stmt_summary_file_max_backups`](/system-variables.md#tidb_stmt_summary_file_max_backups-new-in-v660)                                          | Newly added | This variable is read-only. It specifies the maximum number of data files that can be persisted when [statements summary persistence](/statement-summary-tables.md#persist-statements-summary) is enabled. The value of this variable is the same as that of the configuration item [`tidb_stmt_summary_file_max_backups`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_backups-new-in-v660). |
| [`tidb_stmt_summary_file_max_days`](/system-variables.md#tidb_stmt_summary_file_max_days-new-in-v660)                                                | Newly added | This variable is read-only. It specifies the maximum number of days to keep persistent data files when [statements summary persistence](/statement-summary-tables.md#persist-statements-summary) is enabled. The value of this variable is the same as that of the configuration item [`tidb_stmt_summary_file_max_days`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_days-new-in-v660).     |
| [`tidb_stmt_summary_file_max_size`](/system-variables.md#tidb_stmt_summary_file_max_size-new-in-v660)                                                | Newly added | This variable is read-only. It specifies the maximum size of a persistent data file when [statements summary persistence](/statement-summary-tables.md#persist-statements-summary) is enabled. The value of this variable is the same as that of the configuration item [`tidb_stmt_summary_file_max_size`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_size-new-in-v660).                   |

### 設定ファイルのパラメータ {#configuration-file-parameters}

| Configuration file  | Configuration parameter                                                                                                                                                                                                                                     | Change type | Description                                                                                                                                                                                                                                                                                                         |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TiKV                | `rocksdb.enable-statistics`                                                                                                                                                                                                                                 | Deleted     | This configuration item specifies whether to enable RocksDB statistics. Starting from v6.6.0, this item is deleted. RocksDB statistics are enabled for all clusters by default to help diagnostics. For details, see [#13942](https://github.com/tikv/tikv/pull/13942).                                             |
| TiKV                | `raftdb.enable-statistics`                                                                                                                                                                                                                                  | Deleted     | This configuration item specifies whether to enable Raft RocksDB statistics. Starting from v6.6.0, this item is deleted. Raft RocksDB statistics are enabled for all clusters by default to help diagnostics. For details, see [#13942](https://github.com/tikv/tikv/pull/13942).                                   |
| TiKV                | `storage.block-cache.shared`                                                                                                                                                                                                                                | Deleted     | Starting from v6.6.0, this configuration item is deleted, and the block cache is enabled by default and cannot be disabled. For details, see [#12936](https://github.com/tikv/tikv/issues/12936).                                                                                                                   |
| DM                  | `on-duplicate`                                                                                                                                                                                                                                              | Deleted     | This configuration item controls the methods to resolve conflicts during the full import phase. In v6.6.0, new configuration items `on-duplicate-logical` and `on-duplicate-physical` are introduced to replace `on-duplicate`.                                                                                     |
| TiDB                | [`enable-telemetry`](/tidb-configuration-file.md#enable-telemetry-new-in-v402)                                                                                                                                                                              | Modified    | Starting from v6.6.0, the default value changes from `true` to `false`, which means that telemetry is disabled by default in TiDB.                                                                                                                                                                                  |
| TiKV                | [`rocksdb.defaultcf.block-size`](/tikv-configuration-file.md#block-size) and [`rocksdb.writecf.block-size`](/tikv-configuration-file.md#block-size)                                                                                                         | Modified    | The default values change from `64K` to `32K`.                                                                                                                                                                                                                                                                      |
| TiKV                | [`rocksdb.defaultcf.block-cache-size`](/tikv-configuration-file.md#block-cache-size), [`rocksdb.writecf.block-cache-size`](/tikv-configuration-file.md#block-cache-size), [`rocksdb.lockcf.block-cache-size`](/tikv-configuration-file.md#block-cache-size) | Deprecated  | Starting from v6.6.0, these configuration items are deprecated. For details, see [#12936](https://github.com/tikv/tikv/issues/12936).                                                                                                                                                                               |
| PD                  | [`enable-telemetry`](/pd-configuration-file.md#enable-telemetry)                                                                                                                                                                                            | Modified    | Starting from v6.6.0, the default value changes from `true` to `false`, which means that telemetry is disabled by default in TiDB Dashboard.                                                                                                                                                                        |
| DM                  | [`import-mode`](/dm/task-configuration-file-full.md)                                                                                                                                                                                                        | Modified    | The possible values of this configuration item are changed from `"sql"` and `"loader"` to `"logical"` and `"physical"`. The default value is `"logical"`, which means using TiDB Lightning's logical import mode to import data.                                                                                    |
| TiFlash             | [`profile.default.max_memory_usage_for_all_queries`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                                                                                                      | Modified    | Specifies the memory usage limit for the generated intermediate data in all queries. Starting from v6.6.0, the default value changes from `0` to `0.8`, which means the limit is 80% of the total memory.                                                                                                           |
| TiCDC               | [`consistent.storage`](/ticdc/ticdc-sink-to-mysql.md#prerequisites)                                                                                                                                                                                         | Modified    | This configuration item specifies the path under which redo log backup is stored. Two more value options are added for `scheme`, GCS, and Azure.                                                                                                                                                                    |
| TiDB                | [`initialize-sql-file`](/tidb-configuration-file.md#initialize-sql-file-new-in-v660)                                                                                                                                                                        | Newly added | This configuration item specifies the SQL script to be executed when the TiDB cluster is started for the first time. The default value is empty.                                                                                                                                                                    |
| TiDB                | [`tidb_stmt_summary_enable_persistent`](/tidb-configuration-file.md#tidb_stmt_summary_enable_persistent-new-in-v660)                                                                                                                                        | Newly added | This configuration item controls whether to enable statements summary persistence. The default value is `false`, which means this feature is not enabled by default.                                                                                                                                                |
| TiDB                | [`tidb_stmt_summary_file_max_backups`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_backups-new-in-v660)                                                                                                                                          | Newly added | When statements summary persistence is enabled, this configuration specifies the maximum number of data files that can be persisted. `0` means no limit on the number of files.                                                                                                                                     |
| TiDB                | [`tidb_stmt_summary_file_max_days`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_days-new-in-v660)                                                                                                                                                | Newly added | When statements summary persistence is enabled, this configuration specifies the maximum number of days to keep persistent data files.                                                                                                                                                                              |
| TiDB                | [`tidb_stmt_summary_file_max_size`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_size-new-in-v660)                                                                                                                                                | Newly added | When statements summary persistence is enabled, this configuration specifies the maximum size of a persistent data file (in MiB).                                                                                                                                                                                   |
| TiDB                | [`tidb_stmt_summary_filename`](/tidb-configuration-file.md#tidb_stmt_summary_filename-new-in-v660)                                                                                                                                                          | Newly added | When statements summary persistence is enabled, this configuration specifies the file to which persistent data is written.                                                                                                                                                                                          |
| TiKV                | [`resource-control.enabled`](/tikv-configuration-file.md#resource-control)                                                                                                                                                                                  | Newly added | Whether to enable scheduling for user foreground read/write requests according to the Request Unit (RU) of the corresponding resource groups. The default value is `false`, which means to disable scheduling according to the RU of the corresponding resource groups.                                             |
| TiKV                | [`storage.engine`](/tikv-configuration-file.md#engine-new-in-v660)                                                                                                                                                                                          | Newly added | This configuration item specifies the type of the storage engine. Value options are `"raft-kv"` and `"partitioned-raft-kv"`. This configuration item can only be specified when creating a cluster and cannot be modified once being specified.                                                                     |
| TiKV                | [`rocksdb.write-buffer-flush-oldest-first`](/tikv-configuration-file.md#write-buffer-flush-oldest-first-new-in-v660)                                                                                                                                        | Newly added | This configuration item specifies the flush strategy used when the memory usage of `memtable` of the current RocksDB reaches the threshold.                                                                                                                                                                         |
| TiKV                | [`rocksdb.write-buffer-limit`](/tikv-configuration-file.md#write-buffer-limit-new-in-v660)                                                                                                                                                                  | Newly added | This configuration item specifies the limit on total memory used by `memtable` of all RocksDB instances in a single TiKV. The default value is 25% of the total machine memory.                                                                                                                                     |
| PD                  | [`pd-server.enable-gogc-tuner`](/pd-configuration-file.md#enable-gogc-tuner-new-in-v660)                                                                                                                                                                    | Newly added | This configuration item controls whether to enable the GOGC tuner, which is disabled by default.                                                                                                                                                                                                                    |
| PD                  | [`pd-server.gc-tuner-threshold`](/pd-configuration-file.md#gc-tuner-threshold-new-in-v660)                                                                                                                                                                  | Newly added | This configuration item specifies the maximum memory threshold ratio for tuning GOGC. The default value is `0.6`.                                                                                                                                                                                                   |
| PD                  | [`pd-server.server-memory-limit-gc-trigger`](/pd-configuration-file.md#server-memory-limit-gc-trigger-new-in-v660)                                                                                                                                          | Newly added | This configuration item specifies the threshold ratio at which PD tries to trigger GC. The default value is `0.7`.                                                                                                                                                                                                  |
| PD                  | [`pd-server.server-memory-limit`](/pd-configuration-file.md#server-memory-limit-new-in-v660)                                                                                                                                                                | Newly added | This configuration item specifies the memory limit ratio for a PD instance. The value `0` means no memory limit.                                                                                                                                                                                                    |
| TiCDC               | [`scheduler.region-per-span`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters)                                                                                                                                                        | Newly added | This configuration item controls whether to split a table into multiple replication ranges based on the number of Regions, and these ranges can be replicated by multiple TiCDC nodes. The default value is `50000`.                                                                                                |
| TiDB Lightning      | [`compress-kv-pairs`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)                                                                                                                                                                  | Newly added | This configuration item controls whether to enable compression when sending KV pairs to TiKV in the physical import mode. The default value is empty, meaning that the compression is not enabled.                                                                                                                  |
| DM                  | [`checksum-physical`](/dm/task-configuration-file-full.md)                                                                                                                                                                                                  | Newly added | This configuration item controls whether DM performs `ADMIN CHECKSUM TABLE <table>` for each table to verify data integrity after the import. The default value is `"required"`, which performs admin checksum after the import. If checksum fails, DM pauses the task and you need to manually handle the failure. |
| DM                  | [`disk-quota-physical`](/dm/task-configuration-file-full.md)                                                                                                                                                                                                | Newly added | This configuration item sets the disk quota. It corresponds to the [`disk-quota` configuration](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md#configure-disk-quota-new-in-v620) of TiDB Lightning.                                                                                                   |
| DM                  | [`on-duplicate-logical`](/dm/task-configuration-file-full.md)                                                                                                                                                                                               | Newly added | This configuration item controls how DM resolves conflicting data in the logical import mode. The default value is `"replace"`, which means using the new data to replace the existing data.                                                                                                                        |
| DM                  | [`on-duplicate-physical`](/dm/task-configuration-file-full.md)                                                                                                                                                                                              | Newly added | This configuration item controls how DM resolves conflicting data in the physical import mode. The default value is `"none"`, which means not resolving conflicting data. `"none"` has the best performance, but might lead to inconsistent data in the downstream database.                                        |
| DM                  | [`sorting-dir-physical`](/dm/task-configuration-file-full.md)                                                                                                                                                                                               | Newly added | This configuration item specifies the directory used for local KV sorting in the physical import mode. The default value is the same as the `dir` configuration.                                                                                                                                                    |
| sync-diff-inspector | [`skip-non-existing-table`](/sync-diff-inspector/sync-diff-inspector-overview.md#configuration-file-description)                                                                                                                                            | Newly added | This configuration item controls whether to skip checking upstream and downstream data consistency when tables in the downstream do not exist in the upstream.                                                                                                                                                      |
| TiSpark             | [`spark.tispark.replica_read`](/tispark-overview.md#tispark-configurations)                                                                                                                                                                                 | Newly added | This configuration item controls the type of replicas to be read. The value options are `leader`, `follower`, and `learner`.                                                                                                                                                                                        |
| TiSpark             | [`spark.tispark.replica_read.label`](/tispark-overview.md#tispark-configurations)                                                                                                                                                                           | Newly added | This configuration item is used to set labels for the target TiKV node.                                                                                                                                                                                                                                             |

### その他 {#others}

- [`store-io-pool-size`](/tikv-configuration-file.md#store-io-pool-size-new-in-v530)を動的に変更する機能をサポート。これにより、より柔軟なTiKVのパフォーマンスチューニングが可能になります。
- `LIMIT`句の制限を解除し、実行パフォーマンスを向上させます。
- v6.6.0から、BRはv6.1.0より前のクラスターにデータを復元することをサポートしません。
- v6.6.0から、TiDBはパーティションテーブルの列の型を変更することをサポートしなくなりました。これは潜在的な正確性の問題があるためです。

## 改善 {#improvements}

- TiDB

  - Improve the scheduling mechanism of TTL background cleaning tasks to allow the cleaning task of a single table to be split into several sub-tasks and scheduled to run on multiple TiDB nodes simultaneously [#40361](https://github.com/pingcap/tidb/issues/40361) @[YangKeao](https://github.com/YangKeao)
  - Optimize the column name display of the result returned by running multi-statements after setting a non-default delimiter [#39662](https://github.com/pingcap/tidb/issues/39662) @[mjonss](https://github.com/mjonss)
  - Optimize the execution efficiency of statements after warning messages are generated [#39702](https://github.com/pingcap/tidb/issues/39702) @[tiancaiamao](https://github.com/tiancaiamao)
  - Support distributed data backfill for `ADD INDEX` (experimental) [#37119](https://github.com/pingcap/tidb/issues/37119) @[zimulala](https://github.com/zimulala)
  - Support using `CURDATE()` as the default value of a column [#38356](https://github.com/pingcap/tidb/issues/38356) @[CbcWestwolf](https://github.com/CbcWestwolf)
  - `partial order prop push down` now supports the LIST-type partitioned tables [#40273](https://github.com/pingcap/tidb/issues/40273) @[winoros](https://github.com/winoros)
  - Add error messages for conflicts between optimizer hints and execution plan bindings [#40910](https://github.com/pingcap/tidb/issues/40910) @[Reminiscent](https://github.com/Reminiscent)
  - Optimize the plan cache strategy to avoid non-optimal plans when using plan cache in some scenarios [#40312](https://github.com/pingcap/tidb/pull/40312) [#40218](https://github.com/pingcap/tidb/pull/40218) [#40280](https://github.com/pingcap/tidb/pull/40280) [#41136](https://github.com/pingcap/tidb/pull/41136) [#40686](https://github.com/pingcap/tidb/pull/40686) @[qw4990](https://github.com/qw4990)
  - Clear expired region cache regularly to avoid memory leak and performance degradation [#40461](https://github.com/pingcap/tidb/issues/40461) @[sticnarf](https://github.com/sticnarf)
  - `MODIFY COLUMN` is not supported on partitioned tables [#39915](https://github.com/pingcap/tidb/issues/39915) @[wjhuang2016](https://github.com/wjhuang2016)
  - Disable renaming of columns that partition tables depend on [#40150](https://github.com/pingcap/tidb/issues/40150) @[mjonss](https://github.com/mjonss)
  - Refine the error message reported when a column that a partitioned table depends on is deleted [#38739](https://github.com/pingcap/tidb/issues/38739) @[jiyfhust](https://github.com/jiyfhust)
  - Add a mechanism that `FLASHBACK CLUSTER` retries when it fails to check the `min-resolved-ts` [#39836](https://github.com/pingcap/tidb/issues/39836) @[Defined2014](https://github.com/Defined2014)

- TiKV

  - Optimize the default values of some parameters in partitioned-raft-kv mode: the default value of the TiKV configuration item `storage.block-cache.capacity` is adjusted from 45% to 30%, and the default value of `region-split-size` is adjusted from `96MiB` adjusted to `10GiB`. When using raft-kv mode and `enable-region-bucket` is `true`, `region-split-size` is adjusted to 1 GiB by default. [#12842](https://github.com/tikv/tikv/issues/12842) @[tonyxuqqi](https://github.com/tonyxuqqi)
  - Support priority scheduling in Raftstore asynchronous writes [#13730](https://github.com/tikv/tikv/issues/13730) @[Connor1996](https://github.com/Connor1996)
  - Support starting TiKV on a CPU with less than 1 core [#13586](https://github.com/tikv/tikv/issues/13586) [#13752](https://github.com/tikv/tikv/issues/13752) [#14017](https://github.com/tikv/tikv/issues/14017) @[andreid-db](https://github.com/andreid-db)
  - Optimize the new detection mechanism of Raftstore slow score and add `evict-slow-trend-scheduler` [#14131](https://github.com/tikv/tikv/issues/14131) @[innerr](https://github.com/innerr)
  - Force the block cache of RocksDB to be shared and no longer support setting the block cache separately according to CF [#12936](https://github.com/tikv/tikv/issues/12936) @[busyjay](https://github.com/busyjay)

- PD

  - Support managing the global memory threshold to alleviate the OOM problem (experimental) [#5827](https://github.com/tikv/pd/issues/5827) @[hnes](https://github.com/hnes)
  - Add the GC Tuner to alleviate the GC pressure (experimental) [#5827](https://github.com/tikv/pd/issues/5827) @[hnes](https://github.com/hnes)
  - Add the `evict-slow-trend-scheduler` scheduler to detect and schedule abnormal nodes [#5808](https://github.com/tikv/pd/pull/5808) @[innerr](https://github.com/innerr)
  - Add the keyspace manager to manage keyspace [#5293](https://github.com/tikv/pd/issues/5293) @[AmoebaProtozoa](https://github.com/AmoebaProtozoa)

- TiFlash

  - Support an independent MVCC bitmap filter that decouples the MVCC filtering operations in the TiFlash data scanning process, which provides the foundation for future optimization of the data scanning process [#6296](https://github.com/pingcap/tiflash/issues/6296) @[JinheLin](https://github.com/JinheLin)
  - Reduce the memory usage of TiFlash by up to 30% when there is no query [#6589](https://github.com/pingcap/tiflash/pull/6589) @[hongyunyan](https://github.com/hongyunyan)

- Tools

  - Backup & Restore (BR)

    - Optimize the concurrency of downloading log backup files on the TiKV side to improve the performance of PITR recovery in regular scenarios [#14206](https://github.com/tikv/tikv/issues/14206) @[YuJuncen](https://github.com/YuJuncen)

  - TiCDC

    - Support batch `UPDATE` DML statements to improve TiCDC replication performance [#8084](https://github.com/pingcap/tiflow/issues/8084) @[amyangfei](https://github.com/amyangfei)
    - Implement MQ sink and MySQL sink in the asynchronous mode to improve the sink throughput [#5928](https://github.com/pingcap/tiflow/issues/5928) @[hicqu](https://github.com/hicqu) @[hi-rustin](https://github.com/hi-rustin)

  - TiDB Data Migration (DM)

    - Optimize DM alert rules and content [#7376](https://github.com/pingcap/tiflow/issues/7376) @[D3Hunter](https://github.com/D3Hunter)

      Previously, alerts similar to "DM\_XXX\_process\_exits\_with\_error" were raised whenever a related error occurred. But some alerts are caused by idle database connections, which can be recovered after reconnecting. To reduce these kinds of alerts, DM divides errors into two types: automatically recoverable errors and unrecoverable errors:

      - For an error that is automatically recoverable, DM reports the alert only if the error occurs more than 3 times within 2 minutes.
      - For an error that is not automatically recoverable, DM maintains the original behavior and reports the alert immediately.

    - Optimize relay performance by adding the async/batch relay writer [#4287](https://github.com/pingcap/tiflow/issues/4287) @[GMHDBJD](https://github.com/GMHDBJD)

  - TiDB Lightning

    - Physical Import Mode supports keyspace [#40531](https://github.com/pingcap/tidb/issues/40531) @[iosmanthus](https://github.com/iosmanthus)
    - Support setting the maximum number of conflicts by `lightning.max-error` [#40743](https://github.com/pingcap/tidb/issues/40743) @[dsdashun](https://github.com/dsdashun)
    - Support importing CSV data files with BOM headers [#40744](https://github.com/pingcap/tidb/issues/40744) @[dsdashun](https://github.com/dsdashun)
    - Optimize the processing logic when encountering TiKV flow-limiting errors and try other available regions instead [#40205](https://github.com/pingcap/tidb/issues/40205) @[lance6716](https://github.com/lance6716)
    - Disable checking the table foreign keys during import [#40027](https://github.com/pingcap/tidb/issues/40027) @[gozssky](https://github.com/gozssky)

  - Dumpling

    - Support exporting settings for foreign keys [#39913](https://github.com/pingcap/tidb/issues/39913) @[lichunzhu](https://github.com/lichunzhu)

  - sync-diff-inspector

    - Add a new parameter `skip-non-existing-table` to control whether to skip checking upstream and downstream data consistency when tables in the downstream do not exist in the upstream [#692](https://github.com/pingcap/tidb-tools/issues/692) @[lichunzhu](https://github.com/lichunzhu) @[liumengya94](https://github.com/liumengya94)

## バグ修正 {#bug-fixes}

- TiDB

  - Fix the issue that a statistics collection task fails due to an incorrect `datetime` value [#39336](https://github.com/pingcap/tidb/issues/39336) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
  - Fix the issue that `stats_meta` is not created following table creation [#38189](https://github.com/pingcap/tidb/issues/38189) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
  - Fix frequent write conflicts in transactions when performing DDL data backfill [#24427](https://github.com/pingcap/tidb/issues/24427) @[mjonss](https://github.com/mjonss)
  - Fix the issue that sometimes an index cannot be created for an empty table using ingest mode [#39641](https://github.com/pingcap/tidb/issues/39641) @[tangenta](https://github.com/tangenta)
  - Fix the issue that `wait_ts` in the slow query log is the same for different SQL statements within the same transaction [#39713](https://github.com/pingcap/tidb/issues/39713) @[TonsnakeLin](https://github.com/TonsnakeLin)
  - Fix the issue that the `Assertion Failed` error is reported when adding a column during the process of deleting a row record [#39570](https://github.com/pingcap/tidb/issues/39570) @[wjhuang2016](https://github.com/wjhuang2016)
  - Fix the issue that the `not a DDL owner` error is reported when modifying a column type [#39643](https://github.com/pingcap/tidb/issues/39643) @[zimulala](https://github.com/zimulala)
  - Fix the issue that no error is reported when inserting a row after exhaustion of the auto-increment values of the `AUTO_INCREMENT` column [#38950](https://github.com/pingcap/tidb/issues/38950) @[Dousir9](https://github.com/Dousir9)
  - Fix the issue that the `Unknown column` error is reported when creating an expression index [#39784](https://github.com/pingcap/tidb/issues/39784) @[Defined2014](https://github.com/Defined2014)
  - Fix the issue that data cannot be inserted into a renamed table when the generated expression includes the name of this table [#39826](https://github.com/pingcap/tidb/issues/39826) @[Defined2014](https://github.com/Defined2014)
  - Fix the issue that the `INSERT ignore` statement cannot fill in default values when the column is write-only [#40192](https://github.com/pingcap/tidb/issues/40192) @[YangKeao](https://github.com/YangKeao)
  - Fix the issue that resources are not released when disabling the resource management module [#40546](https://github.com/pingcap/tidb/issues/40546) @[zimulala](https://github.com/zimulala)
  - Fix the issue that TTL tasks cannot trigger statistics updates in time [#40109](https://github.com/pingcap/tidb/issues/40109) @[YangKeao](https://github.com/YangKeao)
  - Fix the issue that unexpected data is read because TiDB improperly handles `NULL` values when constructing key ranges [#40158](https://github.com/pingcap/tidb/issues/40158) @[tiancaiamao](https://github.com/tiancaiamao)
  - Fix the issue that illegal values are written to a table when the `MODIFT COLUMN` statement also changes the default value of a column [#40164](https://github.com/pingcap/tidb/issues/40164) @[wjhuang2016](https://github.com/wjhuang2016)
  - Fix the issue that the adding index operation is inefficient due to invalid Region cache when there are many Regions in a table [#38436](https://github.com/pingcap/tidb/issues/38436) @[tangenta](https://github.com/tangenta)
  - Fix data race occurred in allocating auto-increment IDs [#40584](https://github.com/pingcap/tidb/issues/40584) @[Dousir9](https://github.com/Dousir9)
  - Fix the issue that the implementation of the not operator in JSON is incompatible with the implementation in MySQL [#40683](https://github.com/pingcap/tidb/issues/40683) @[YangKeao](https://github.com/YangKeao)
  - Fix the issue that concurrent view might cause DDL operations to be blocked [#40352](https://github.com/pingcap/tidb/issues/40352) @[zeminzhou](https://github.com/zeminzhou)
  - Fix data inconsistency caused by concurrently executing DDL statements to modify columns of partitioned tables [#40620](https://github.com/pingcap/tidb/issues/40620) @[mjonss](https://github.com/mjonss) @[mjonss](https://github.com/mjonss)
  - Fix the issue that "Malformed packet" is reported when using `caching_sha2_password` for authentication without specifying a password [#40831](https://github.com/pingcap/tidb/issues/40831) @[dveeden](https://github.com/dveeden)
  - Fix the issue that a TTL task fails if the primary key of the table contains an `ENUM` column [#40456](https://github.com/pingcap/tidb/issues/40456) @[lcwangchao](https://github.com/lcwangchao)
  - Fix the issue that some DDL operations blocked by MDL cannot be queried in `mysql.tidb_mdl_view` [#40838](https://github.com/pingcap/tidb/issues/40838) @[YangKeao](https://github.com/YangKeao)
  - Fix the issue that data race might occur during DDL ingestion [#40970](https://github.com/pingcap/tidb/issues/40970) @[tangenta](https://github.com/tangenta)
  - Fix the issue that TTL tasks might delete some data incorrectly after the time zone changes [#41043](https://github.com/pingcap/tidb/issues/41043) @[lcwangchao](https://github.com/lcwangchao)
  - Fix the issue that `JSON_OBJECT` might report an error in some cases [#39806](https://github.com/pingcap/tidb/issues/39806) @[YangKeao](https://github.com/YangKeao)
  - Fix the issue that TiDB might deadlock during initialization [#40408](https://github.com/pingcap/tidb/issues/40408) @[Defined2014](https://github.com/Defined2014)
  - Fix the issue that the value of system variables might be incorrectly modified in some cases due to memory reuse [#40979](https://github.com/pingcap/tidb/issues/40979) @[lcwangchao](https://github.com/lcwangchao)
  - Fix the issue that data might be inconsistent with the index when a unique index is created in the ingest mode [#40464](https://github.com/pingcap/tidb/issues/40464) @[tangenta](https://github.com/tangenta)
  - Fix the issue that some truncate operations cannot be blocked by MDL when truncating the same table concurrently [#40484](https://github.com/pingcap/tidb/issues/40484) @[wjhuang2016](https://github.com/wjhuang2016)
  - Fix the issue that the `SHOW PRIVILEGES` statement returns an incomplete privilege list [#40591](https://github.com/pingcap/tidb/issues/40591) @[CbcWestwolf](https://github.com/CbcWestwolf)
  - Fix the issue that TiDB panics when adding a unique index [#40592](https://github.com/pingcap/tidb/issues/40592) @[tangenta](https://github.com/tangenta)
  - Fix the issue that executing the `ADMIN RECOVER` statement might cause the index data to be corrupted [#40430](https://github.com/pingcap/tidb/issues/40430) @[xiongjiwei](https://github.com/xiongjiwei)
  - Fix the issue that a query might fail when the queried table contains a `CAST` expression in the expression index [#40130](https://github.com/pingcap/tidb/issues/40130) @[xiongjiwei](https://github.com/xiongjiwei)
  - Fix the issue that a unique index might still produce duplicate data in some cases [#40217](https://github.com/pingcap/tidb/issues/40217) @[tangenta](https://github.com/tangenta)
  - Fix the PD OOM issue when there is a large number of Regions but the table ID cannot be pushed down when querying some virtual tables using `Prepare` or `Execute` [#39605](https://github.com/pingcap/tidb/issues/39605) @[djshow832](https://github.com/djshow832)
  - Fix the issue that data race might occur when an index is added [#40879](https://github.com/pingcap/tidb/issues/40879) @[tangenta](https://github.com/tangenta)
  - Fix the `can't find proper physical plan` issue caused by virtual columns [#41014](https://github.com/pingcap/tidb/issues/41014) @[AilinKid](https://github.com/AilinKid)
  - Fix the issue that TiDB cannot restart after global bindings are created for partition tables in dynamic trimming mode [#40368](https://github.com/pingcap/tidb/issues/40368) @[Yisaer](https://github.com/Yisaer)
  - Fix the issue that `auto analyze` causes graceful shutdown to take a long time [#40038](https://github.com/pingcap/tidb/issues/40038) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
  - Fix the panic of the TiDB server when the IndexMerge operator triggers memory limiting behaviors [#41036](https://github.com/pingcap/tidb/pull/41036) @[guo-shaoge](https://github.com/guo-shaoge)
  - Fix the issue that the `SELECT * FROM table_name LIMIT 1` query on partitioned tables is slow [#40741](https://github.com/pingcap/tidb/pull/40741) @[solotzg](https://github.com/solotzg)

- TiKV

  - Fix an error that occurs when casting the `const Enum` type to other types [#14156](https://github.com/tikv/tikv/issues/14156) @[wshwsh12](https://github.com/wshwsh12)
  - Fix the issue that Resolved TS causes higher network traffic [#14092](https://github.com/tikv/tikv/issues/14092) @[overvenus](https://github.com/overvenus)
  - Fix the data inconsistency issue caused by network failure between TiDB and TiKV during the execution of a DML after a failed pessimistic DML [#14038](https://github.com/tikv/tikv/issues/14038) @[MyonKeminta](https://github.com/MyonKeminta)

- PD

  - Fix the issue that the Region Scatter task generates redundant replicas unexpectedly [#5909](https://github.com/tikv/pd/issues/5909) @[HundunDM](https://github.com/HunDunDM)
  - Fix the issue that the Online Unsafe Recovery feature would get stuck and time out in `auto-detect` mode [#5753](https://github.com/tikv/pd/issues/5753) @[Connor1996](https://github.com/Connor1996)
  - Fix the issue that the execution `replace-down-peer` slows down under certain conditions [#5788](https://github.com/tikv/pd/issues/5788) @[HundunDM](https://github.com/HunDunDM)
  - Fix the PD OOM issue that occurs when the calls of `ReportMinResolvedTS` are too frequent [#5965](https://github.com/tikv/pd/issues/5965) @[HundunDM](https://github.com/HunDunDM)

- TiFlash

  - Fix the issue that querying TiFlash-related system tables might get stuck [#6745](https://github.com/pingcap/tiflash/pull/6745) @[lidezhu](https://github.com/lidezhu)
  - Fix the issue that semi-joins use excessive memory when calculating Cartesian products [#6730](https://github.com/pingcap/tiflash/issues/6730) @[gengliqi](https://github.com/gengliqi)
  - Fix the issue that the result of the division operation on the DECIMAL data type is not rounded [#6393](https://github.com/pingcap/tiflash/issues/6393) @[LittleFall](https://github.com/LittleFall)
  - Fix the issue that `start_ts` cannot uniquely identify an MPP query in TiFlash queries, which might cause an MPP query to be incorrectly canceled [#43426](https://github.com/pingcap/tidb/issues/43426) @[hehechen](https://github.com/hehechen)

- Tools

  - Backup & Restore (BR)

    - Fix the issue that when restoring log backup, hot Regions cause the restore to fail [#37207](https://github.com/pingcap/tidb/issues/37207) @[Leavrth](https://github.com/Leavrth)
    - Fix the issue that restoring data to a cluster on which the log backup is running causes the log backup file to be unrecoverable [#40797](https://github.com/pingcap/tidb/issues/40797) @[Leavrth](https://github.com/Leavrth)
    - Fix the issue that the PITR feature does not support CA-bundles [#38775](https://github.com/pingcap/tidb/issues/38775) @[YuJuncen](https://github.com/YuJuncen)
    - Fix the panic issue caused by duplicate temporary tables during recovery [#40797](https://github.com/pingcap/tidb/issues/40797) @[joccau](https://github.com/joccau)
    - Fix the issue that PITR does not support configuration changes for PD clusters [#14165](https://github.com/tikv/tikv/issues/14165) @[YuJuncen](https://github.com/YuJuncen)
    - Fix the issue that the connection failure between PD and tidb-server causes PITR backup progress not to advance [#41082](https://github.com/pingcap/tidb/issues/41082) @[YuJuncen](https://github.com/YuJuncen)
    - Fix the issue that TiKV cannot listen to PITR tasks due to the connection failure between PD and TiKV [#14159](https://github.com/tikv/tikv/issues/14159) @[YuJuncen](https://github.com/YuJuncen)
    - Fix the issue that the frequency of `resolve lock` is too high when there is no PITR backup task in the TiDB cluster [#40759](https://github.com/pingcap/tidb/issues/40759) @[joccau](https://github.com/joccau)
    - Fix the issue that when a PITR backup task is deleted, the residual backup data causes data inconsistency in new tasks [#40403](https://github.com/pingcap/tidb/issues/40403) @[joccau](https://github.com/joccau)

  - TiCDC

    - Fix the issue that `transaction_atomicity` and `protocol` cannot be updated via the configuration file [#7935](https://github.com/pingcap/tiflow/issues/7935) @[CharlesCheung96](https://github.com/CharlesCheung96)
    - Fix the issue that precheck is not performed on the storage path of redo log [#6335](https://github.com/pingcap/tiflow/issues/6335) @[CharlesCheung96](https://github.com/CharlesCheung96)
    - Fix the issue of insufficient duration that redo log can tolerate for S3 storage failure [#8089](https://github.com/pingcap/tiflow/issues/8089) @[CharlesCheung96](https://github.com/CharlesCheung96)
    - Fix the issue that changefeed might get stuck in special scenarios such as when scaling in or scaling out TiKV or TiCDC nodes [#8174](https://github.com/pingcap/tiflow/issues/8174) @[hicqu](https://github.com/hicqu)
    - Fix the issue of too high traffic among TiKV nodes [#14092](https://github.com/tikv/tikv/issues/14092) @[overvenus](https://github.com/overvenus)
    - Fix the performance issues of TiCDC in terms of CPU usage, memory control, and throughput when the pull-based sink is enabled [#8142](https://github.com/pingcap/tiflow/issues/8142) [#8157](https://github.com/pingcap/tiflow/issues/8157) [#8001](https://github.com/pingcap/tiflow/issues/8001) [#5928](https://github.com/pingcap/tiflow/issues/5928) @[hicqu](https://github.com/hicqu) @[hi-rustin](https://github.com/hi-rustin)

  - TiDB Data Migration (DM)

    - Fix the issue that the `binlog-schema delete` command fails to execute [#7373](https://github.com/pingcap/tiflow/issues/7373) @[liumengya94](https://github.com/liumengya94)
    - Fix the issue that the checkpoint does not advance when the last binlog is a skipped DDL [#8175](https://github.com/pingcap/tiflow/issues/8175) @[D3Hunter](https://github.com/D3Hunter)
    - Fix a bug that when the expression filters of both "update" and "non-update" types are specified in one table, all `UPDATE` statements are skipped [#7831](https://github.com/pingcap/tiflow/issues/7831) @[lance6716](https://github.com/lance6716)
    - Fix a bug that when only one of `update-old-value-expr` or `update-new-value-expr` is set for a table, the filter rule does not take effect or DM panics [#7774](https://github.com/pingcap/tiflow/issues/7774) @[lance6716](https://github.com/lance6716)

  - TiDB Lightning

    - Fix the issue that TiDB Lightning timeout hangs due to TiDB restart in some scenarios [#33714](https://github.com/pingcap/tidb/issues/33714) @[lichunzhu](https://github.com/lichunzhu)
    - Fix the issue that TiDB Lightning might incorrectly skip conflict resolution when all but the last TiDB Lightning instance encounters a local duplicate record during a parallel import [#40923](https://github.com/pingcap/tidb/issues/40923) @[lichunzhu](https://github.com/lichunzhu)
    - Fix the issue that precheck cannot accurately detect the presence of a running TiCDC in the target cluster [#41040](https://github.com/pingcap/tidb/issues/41040) @[lance6716](https://github.com/lance6716)
    - Fix the issue that TiDB Lightning panics in the split-region phase [#40934](https://github.com/pingcap/tidb/issues/40934) @[lance6716](https://github.com/lance6716)
    - Fix the issue that the conflict resolution logic (`duplicate-resolution`) might lead to inconsistent checksums [#40657](https://github.com/pingcap/tidb/issues/40657) @[gozssky](https://github.com/gozssky)
    - Fix a possible OOM problem when there is an unclosed delimiter in the data file [#40400](https://github.com/pingcap/tidb/issues/40400) @[buchuitoudegou](https://github.com/buchuitoudegou)
    - Fix the issue that the file offset in the error report exceeds the file size [#40034](https://github.com/pingcap/tidb/issues/40034) @[buchuitoudegou](https://github.com/buchuitoudegou)
    - Fix an issue with the new version of PDClient that might cause parallel import to fail [#40493](https://github.com/pingcap/tidb/issues/40493) @[AmoebaProtozoa](https://github.com/AmoebaProtozoa)
    - Fix the issue that TiDB Lightning prechecks cannot find dirty data left by previously failed imports [#39477](https://github.com/pingcap/tidb/issues/39477) @[dsdashun](https://github.com/dsdashun)

## Contributors {#contributors}

We would like to thank the following contributors from the TiDB community:

- [morgo](https://github.com/morgo)
- [jiyfhust](https://github.com/jiyfhust)
- [b41sh](https://github.com/b41sh)
- [sourcelliu](https://github.com/sourcelliu)
- [songzhibin97](https://github.com/songzhibin97)
- [mamil](https://github.com/mamil)
- [Dousir9](https://github.com/Dousir9)
- [hihihuhu](https://github.com/hihihuhu)
- [mychoxin](https://github.com/mychoxin)
- [xuning97](https://github.com/xuning97)
- [andreid-db](https://github.com/andreid-db)
