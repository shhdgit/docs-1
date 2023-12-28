---
title: mysql Schema
summary: Learn about the TiDB system tables.
---

# `mysql`スキーマ {#mysql-schema}

`mysql`スキーマには、TiDBシステムテーブルが含まれています。デザインはMySQLの`mysql`スキーマと似ており、`mysql.user`などのテーブルを直接編集することができます。また、MySQLの拡張機能となるテーブルも含まれています。

## 権限システムテーブル {#grant-system-tables}

これらのシステムテーブルには、ユーザーアカウントとその権限に関する権限情報が含まれています。

- `user`：ユーザーアカウント、グローバル権限、およびその他の非権限列
- `db`：データベースレベルの権限
- `tables_priv`：テーブルレベルの権限
- `columns_priv`：列レベルの権限
- `password_history`：パスワード変更履歴
- `default_roles`：ユーザーのデフォルトロール
- `global_grants`：動的権限
- `global_priv`：証明書に基づく認証情報
- `role_edges`：ロール間の関係

## クラスターステータスシステムテーブル {#cluster-status-system-tables}

- `tidb`テーブルには、TiDBに関するいくつかのグローバル情報が含まれています：

  - `bootstrapped`：TiDBクラスターが初期化されたかどうか。この値は読み取り専用であり、変更することはできません。
  - `tidb_server_version`：TiDBが初期化されたときのバージョン情報。この値は読み取り専用であり、変更することはできません。
  - `system_tz`：TiDBのシステムタイムゾーン。
  - `new_collation_enabled`：TiDBが[照合順序の新しいフレームワーク](/character-set-and-collation.md#new-framework-for-collations)を有効にしているかどうか。この値は読み取り専用であり、変更することはできません。

## サーバーサイドヘルプシステムテーブル {#server-side-help-system-tables}

現在、`help_topic`はNULLです。

## 統計システムテーブル {#statistics-system-tables}

- `stats_buckets`：統計のバケット
- `stats_histograms`：統計のヒストグラム
- `stats_top_n`：統計のTopN
- `stats_meta`：テーブルのメタ情報、例えば総行数や更新された行数など
- `stats_extended`：拡張統計、例えば列間の順序相関など
- `stats_feedback`：統計のクエリフィードバック
- `stats_fm_sketch`：統計カラムのヒストグラムのFMSketch分布
- `analyze_options`：各テーブルのデフォルトの`analyze`オプション
- `column_stats_usage`：列統計の使用状況
- `schema_index_usage`：インデックスの使用状況
- `analyze_jobs`：過去7日間の統計収集タスクと履歴タスクレコード

## 実行計画に関連するシステムテーブル {#execution-plan-related-system-tables}

- `bind_info`：実行計画のバインディング情報
- `capture_plan_baselines_blacklist`：実行計画の自動バインディングのブロックリスト

## GCワーカーシステムテーブル {#gc-worker-system-tables}

> **Note:**
>
> GCワーカーシステムテーブルは、TiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

- `gc_delete_range`：削除するKV範囲
- `gc_delete_range_done`：削除されたKV範囲

## キャッシュされたテーブルに関連するシステムテーブル {#system-tables-related-to-cached-tables}

- `table_cache_meta`：キャッシュされたテーブルのメタデータを格納します。

## TTLに関連するシステムテーブル {#ttl-related-system-tables}

> **Note:**
>
> TTLに関連するシステムテーブルは、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

- `mysql.tidb_ttl_table_status`：すべてのTTLテーブルの以前に実行されたTTLジョブと現在実行中のTTLジョブ
- `mysql.tidb_ttl_task`：現在実行中のTTLサブタスク
- `mysql.tidb_ttl_job_history`：過去90日間のTTLタスクの実行履歴

## メタデータロックに関連するシステムテーブル {#system-tables-related-to-metadata-locks}

- `tidb_mdl_view`：メタデータロックのビュー。現在ブロックされているDDLステートメントに関する情報を表示するために使用できます。
- `tidb_mdl_info`：ノード間でメタデータロックを同期するためにTiDB内部で使用されます。

## その他のシステムテーブル {#miscellaneous-system-tables}

> **Note:**
>
> `tidb`、`expr_pushdown_blacklist`、`opt_rule_blacklist`、および`table_cache_meta`システムテーブルは、TiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

- `GLOBAL_VARIABLES`：グローバルシステム変数テーブル
- `expr_pushdown_blacklist`：式プッシュダウンのブロックリスト
- `opt_rule_blacklist`：論理最適化ルールのブロックリスト
- `tidb_timers`：内部タイマーのメタデータ
