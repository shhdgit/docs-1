---
title: TiDB Backend Task Distributed Execution Framework
summary: Learn the use cases, limitations, usage, and implementation principles of the TiDB backend task distributed execution framework.
---

# TiDB バックエンドタスク分散実行フレームワーク {#tidb-backend-task-distributed-execution-framework}

> **Warning:**
>
> この機能は実験的な機能です。本番環境で使用することはお勧めしません。

<CustomContent platform="tidb-cloud">

> **Note:**
>
> 現在、この機能は TiDB Dedicated クラスターにのみ適用されます。TiDB Serverless クラスターでは使用できません。

</CustomContent>

TiDB は、優れた拡張性と弾力性を持つ計算とストレージの分離アーキテクチャを採用しています。v7.1.0 から、TiDB はバックエンドタスクの分散実行フレームワークを導入し、分散アーキテクチャのリソース利点をさらに活用します。このフレームワークの目標は、すべてのバックエンドタスクの統一されたスケジューリングと分散実行を実装し、全体および個々のバックエンドタスクの統一されたリソース管理機能を提供することで、ユーザーのリソース使用に対する期待によりよく応えることです。

このドキュメントでは、TiDB バックエンドタスク分散実行フレームワークのユースケース、制限、使用方法、および実装原則について説明します。

> **Note:**
>
> このフレームワークは、SQL クエリの分散実行をサポートしていません。

## ユースケースと制限 {#use-cases-and-limitations}

データベース管理システムでは、コアのトランザクション処理（TP）および分析処理（AP）のワークロードに加えて、DDL 操作、Load Data、TTL、Analyze、および Backup/Restore などの重要なタスクがあります。これらのバックエンドタスクは、データベースオブジェクト（テーブル）内の大量のデータを処理する必要があるため、通常、次の特性を持ちます。

- スキーマまたはデータベースオブジェクト（テーブル）内のすべてのデータを処理する必要があります。
- 定期的に実行する必要がある場合がありますが、頻度は低いです。
- リソースが適切に制御されないと、TP および AP タスクに影響を与え、データベースサービスの品質を低下させる可能性があります。

TiDB バックエンドタスク分散実行フレームワークを有効にすると、上記の問題を解決でき、次の3つの利点があります。

- フレームワークは高い拡張性、高い可用性、高いパフォーマンスの統一された機能を提供します。
- フレームワークはバックエンドタスクの分散実行をサポートし、TiDB クラスター全体の利用可能な計算リソースを柔軟にスケジュールできるため、TiDB クラスター内の計算リソースをより効果的に利用できます。
- フレームワークは、全体および個々のバックエンドタスクの統一されたリソース使用と管理機能を提供します。

現在、TiDB バックエンドタスク分散実行フレームワークは、`ADD INDEX` ステートメントの分散実行のみをサポートしています。つまり、インデックスを作成するための DDL ステートメントです。たとえば、次の SQL ステートメントがサポートされています。

```sql
ALTER TABLE t1 ADD INDEX idx1(c1);
CREATE INDEX idx1 ON table t1(c1);
```

現在、TiDB Self-Hosted では、DXF は `ADD INDEX` ステートメントの分散実行をサポートしています。

- `ADD INDEX` は、インデックスを作成するための DDL ステートメントです。たとえば：

  ```sql
  ALTER TABLE t1 ADD INDEX idx1(c1);
  CREATE INDEX idx1 ON table t1(c1);
  ```

## 制限 {#limitation}

- DXF は一度に 1 つの `ADD INDEX` タスクの分散実行のみをスケジュールできます。現在の `ADD INDEX` 分散タスクが完了する前に新しい `ADD INDEX` タスクが提出されると、新しいタスクはトランザクションを介して実行されます。
- DXF を使用して `TIMESTAMP` データ型の列にインデックスを追加することはサポートされていません。これは、インデックスとデータの不整合を引き起こす可能性があるためです。

## 前提条件 {#prerequisites}

分散フレームワークを使用する前に、[Fast Online DDL](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630) モードを有効にする必要があります。

<CustomContent platform="tidb">

1. Fast Online DDL に関連する次のシステム変数を調整します。

   - [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630): Fast Online DDL モードを有効にします。TiDB v6.5.0 以降ではデフォルトで有効になっています。
   - [`tidb_ddl_disk_quota`](/system-variables.md#tidb_ddl_disk_quota-new-in-v630): Fast Online DDL モードで使用できるローカルディスクの最大クォータを制御します。

2. Fast Online DDL に関連する次の構成項目を調整します。

   - [`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630): Fast Online DDL モードで使用できるローカルディスクパスを指定します。

> **Note:**
>
> TiDB を v6.5.0 以降にアップグレードする前に、TiDB の [`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630) パスが正しく SSD ディスクにマウントされているかどうかを確認することをお勧めします。TiDB を実行するオペレーティングシステムユーザーがこのディレクトリに対して読み書き権限を持っていることを確認してください。そうしないと、DDL 操作に予測不可能な問題が発生する可能性があります。このパスは TiDB の構成項目であり、TiDB を再起動した後に有効になります。したがって、アップグレード前にこの構成項目を設定することで、別の再起動を回避できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

Fast Online DDL に関連する次のシステム変数を調整します。

- [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630): Fast Online DDL モードを有効にします。TiDB v6.5.0 以降ではデフォルトで有効になっています。
- [`tidb_ddl_disk_quota`](/system-variables.md#tidb_ddl_disk_quota-new-in-v630): Fast Online DDL モードで使用できるローカルディスクの最大クォータを制御します。

</CustomContent>

## 使用方法 {#usage}

1. 分散フレームワークを有効にするには、[`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710) の値を `ON` に設定します。

   ```sql
   SET GLOBAL tidb_enable_dist_task = ON;
   ```

   バックエンドタスクが実行されている場合、フレームワークでサポートされている DDL ステートメントは分散して実行されます。

2. 分散 DDL タスクの実行に影響を与える可能性のある次のシステム変数を必要に応じて調整します。

   - [`tidb_ddl_reorg_worker_cnt`](/system-variables.md#tidb_ddl_reorg_worker_cnt): デフォルト値は `4` を使用します。推奨される最大値は `16` です。
   - [`tidb_ddl_reorg_priority`](/system-variables.md#tidb_ddl_reorg_priority)
   - [`tidb_ddl_error_count_limit`](/system-variables.md#tidb_ddl_error_count_limit)
   - [`tidb_ddl_reorg_batch_size`](/system-variables.md#tidb_ddl_reorg_batch_size): デフォルト値を使用します。推奨される最大値は `1024` です。

> **Tip:**
>
> `ADD INDEX` ステートメントの分散実行には、`tidb_ddl_reorg_worker_cnt` のみを設定する必要があります。

## 実装原則 {#implementation-principles}

TiDB バックエンドタスク分散実行フレームワークのアーキテクチャは次のとおりです。

![TiDB バックエンドタスク分散実行フレームワークのアーキテクチャ](/media/dist-task/dist-task-architect.jpg)

前述の図に示すように、分散フレームワークでのバックエンドタスクの実行は、主に次のモジュールによって処理されます。

- Dispatcher: 各タスクの分散実行プランを生成し、実行プロセスを管理し、タスクのステータスを変換し、ランタイムタスク情報を収集およびフィードバックします。
- Scheduler: 分散タスクの実行を TiDB ノード間で複製し、バックエンドタスクの実行効率を向上させます。
- Subtask Executor: 分散サブタスクの実際の実行者。また、Subtask Executor はサブタスクの実行ステータスをスケジューラに返し、スケジューラはサブタスクの実行ステータスを統一的に更新します。
- リソースプール: 上記のモジュールの計算リソースをプール化することで、リソースの使用と管理を数量化する基盤を提供します。

## 関連情報 {#see-also}

<CustomContent platform="tidb">

- [DDL ステートメントの実行原則とベストプラクティス](/ddl-introduction.md)

</CustomContent>
<CustomContent platform="tidb-cloud">

- [DDL ステートメントの実行原則とベストプラクティス](https://docs.pingcap.com/tidb/stable/ddl-introduction)

</CustomContent>
