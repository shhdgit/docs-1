---
title: TiDB Backend Task Distributed Execution Framework
summary: Learn the use cases, limitations, usage, and implementation principles of the TiDB backend task distributed execution framework.
---

# TiDB バックエンドタスク分散実行フレームワーク {#tidb-backend-task-distributed-execution-framework}

> **警告:**
>
> この機能は実験的な機能です。本番環境で使用することはお勧めしません。

<CustomContent platform="tidb-cloud">

> **注意:**
>
> 現在、この機能は TiDB Dedicated クラスターにのみ適用されます。TiDB Serverless クラスターでは使用できません。

</CustomContent>

TiDB は、優れたスケーラビリティと弾力性を備えた計算ストレージ分離アーキテクチャを採用しています。v7.1.0 から、TiDB はバックエンドタスク分散実行フレームワークを導入し、分散アーキテクチャのリソース優位性をさらに活用します。このフレームワークの目的は、すべてのバックエンドタスクの統一スケジューリングと分散実行を実現し、全体および個々のバックエンドタスクの統一リソース管理機能を提供することで、ユーザーのリソース使用の期待に応えることです。

このドキュメントでは、TiDB バックエンドタスク分散実行フレームワークの使用例、制限事項、使用方法、および実装原理について説明します。

> **注意:**
>
> このフレームワークは SQL クエリの分散実行をサポートしていません。

## 使用例と制限事項 {#use-cases-and-limitations}

データベース管理システムでは、コアのトランザクション処理 (TP) および分析処理 (AP) のワークロードに加えて、DDL 操作、Load Data、TTL、Analyze、および Backup/Restore などの重要なタスクがあります。これらのバックエンドタスクは、データベースオブジェクト (テーブル) の大量のデータを処理する必要があるため、通常次の特性を持ちます。

- スキーマまたはデータベースオブジェクト (テーブル) のすべてのデータを処理する必要があります。
- 定期的に実行する必要がある場合がありますが、頻度は低いです。
- リソースが適切に制御されない場合、TP および AP タスクに影響を与え、データベースサービスの品質を低下させる可能性があります。

TiDB バックエンドタスク分散実行フレームワークを有効にすると、上記の問題を解決でき、次の 3 つの利点があります。

- フレームワークは、高いスケーラビリティ、高い可用性、および高いパフォーマンスの統一機能を提供します。
- フレームワークはバックエンドタスクの分散実行をサポートし、TiDB クラスター全体の利用可能な計算リソースを柔軟にスケジュールできるため、TiDB クラスターの計算リソースをより効率的に利用できます。
- フレームワークは、全体および個々のバックエンドタスクの統一リソース使用と管理機能を提供します。

現在、TiDB バックエンドタスク分散実行フレームワークは、`ADD INDEX` ステートメント、つまりインデックスを作成するための DDL ステートメントの分散実行のみをサポートしています。たとえば、次の SQL ステートメントがサポートされます。

```sql
ALTER TABLE t1 ADD INDEX idx1(c1);
CREATE INDEX idx1 ON table t1(c1);
```

現在、TiDB Self-Hostedでは、DXFは`ADD INDEX`ステートメントの分散実行をサポートしています。

- `ADD INDEX`は、インデックスを作成するためのDDLステートメントです。例えば：

  ```sql
  ALTER TABLE t1 ADD INDEX idx1(c1);
  CREATE INDEX idx1 ON table t1(c1);
  ```

## 制限事項 {#limitation}

- DXFは、一度に1つの`ADD INDEX`タスクの分散実行のみをスケジュールできます。現在の`ADD INDEX`分散タスクが完了する前に新しい`ADD INDEX`タスクが送信された場合、新しいタスクはトランザクションを介して実行されます。
- DXFを使用して`TIMESTAMP`データ型の列にインデックスを追加することはサポートされていません。これは、インデックスとデータの間の不整合を引き起こす可能性があるためです。

## 前提条件 {#prerequisites}

分散フレームワークを使用する前に、[Fast Online DDL](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)モードを有効にする必要があります。

<CustomContent platform="tidb">

1. Fast Online DDLに関連する次のシステム変数を調整します。

   - [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)：Fast Online DDLモードを有効にするために使用されます。TiDB v6.5.0以降ではデフォルトで有効になっています。
   - [`tidb_ddl_disk_quota`](/system-variables.md#tidb_ddl_disk_quota-new-in-v630)：Fast Online DDLモードで使用できるローカルディスクの最大クォータを制御するために使用されます。

2. Fast Online DDLに関連する次の構成項目を調整します。

   - [`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)：Fast Online DDLモードで使用できるローカルディスクパスを指定します。

> **Note:**
>
> TiDBをv6.5.0以降にアップグレードする前に、TiDBの[`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)パスが正しくSSDディスクにマウントされているかどうかを確認することをお勧めします。TiDBを実行するオペレーティングシステムのユーザーがこのディレクトリに対して読み書き権限を持っていることを確認してください。そうしないと、DDL操作に予測不可能な問題が発生する可能性があります。このパスはTiDBの構成項目であり、TiDBが再起動された後に有効になります。そのため、アップグレード前にこの構成項目を事前に設定することで、別の再起動を回避できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

Fast Online DDLに関連する次のシステム変数を調整します。

- [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)：Fast Online DDLモードを有効にするために使用されます。TiDB v6.5.0以降ではデフォルトで有効になっています。
- [`tidb_ddl_disk_quota`](/system-variables.md#tidb_ddl_disk_quota-new-in-v630)：Fast Online DDLモードで使用できるローカルディスクの最大クォータを制御するために使用されます。

</CustomContent>

## 使用方法 {#usage}

1. 分散フレームワークを有効にするには、[`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710)の値を`ON`に設定します。

   ```sql
   SET GLOBAL tidb_enable_dist_task = ON;
   ```

   バックエンドタスクが実行されている場合、フレームワークでサポートされているDDLステートメントは分散方式で実行されます。

2. 次のシステム変数を調整します。これらの変数はDDLタスクの分散実行に影響する可能性があります。

   - [`tidb_ddl_reorg_worker_cnt`](/system-variables.md#tidb_ddl_reorg_worker_cnt)：デフォルト値`4`を使用します。推奨される最大値は`16`です。
   - [`tidb_ddl_reorg_priority`](/system-variables.md#tidb_ddl_reorg_priority)
   - [`tidb_ddl_error_count_limit`](/system-variables.md#tidb_ddl_error_count_limit)
   - [`tidb_ddl_reorg_batch_size`](/system-variables.md#tidb_ddl_reorg_batch_size)：デフォルト値を使用します。推奨される最大値は`1024`です。

> **Tip:**
>
> `ADD INDEX`ステートメントの分散実行には、`tidb_ddl_reorg_worker_cnt`のみを設定する必要があります。

## 実装原理 {#implementation-principles}

TiDBバックエンドタスク分散実行フレームワークのアーキテクチャは次のとおりです。

![TiDBバックエンドタスク分散実行フレームワークのアーキテクチャ](/media/dist-task/dist-task-architect.jpg)

前の図に示すように、分散フレームワークでバックエンドタスクの実行は主に次のモジュールによって処理されます：

- Dispatcher: 各タスクの分散実行計画を生成し、実行プロセスを管理し、タスクの状態を変換し、ランタイムタスク情報を収集してフィードバックします。
- Scheduler: TiDBノード間で分散タスクの実行を複製し、バックエンドタスクの実行効率を向上させます。
- Subtask Executor: 分散サブタスクの実際の実行者です。また、Subtask Executorはサブタスクの実行状況をSchedulerに返し、Schedulerはサブタスクの実行状況を統一的に更新します。
- リソースプール: 上記モジュールの計算リソースをプールすることで、リソース使用量の定量化と管理の基盤を提供します。

## 関連情報 {#see-also}

<CustomContent platform="tidb">

- [DDLステートメントの実行原理とベストプラクティス](/ddl-introduction.md)

</CustomContent>
<CustomContent platform="tidb-cloud">

- [DDLステートメントの実行原理とベストプラクティス](https://docs.pingcap.com/tidb/stable/ddl-introduction)

</CustomContent>
