---
title: mysql Schema
summary: Learn about the TiDB system tables.
---

# `mysql`スキーマ {#mysql-schema}

`mysql`スキーマには、TiDBシステムテーブルが含まれています。設計は、`mysql.user`などのテーブルが直接編集できるMySQLの`mysql`スキーマに類似しています。また、MySQLの拡張機能であるいくつかのテーブルも含まれています。

## 権限システムテーブル {#grant-system-tables}

これらのシステムテーブルには、ユーザーアカウントとその権限に関する権限情報が含まれています。

- `user`: ユーザーアカウント、グローバル権限、およびその他の非権限列
- `db`: データベースレベルの権限
- `tables_priv`: テーブルレベルの権限
- `columns_priv`: 列レベルの権限
- `password_history`: パスワード変更履歴
- `default_roles`: ユーザーのデフォルトの役割
- `global_grants`: 動的権限
- `global_priv`: 証明書に基づく認証情報
- `role_edges`: 役割間の関係

## クラスタステータスシステムテーブル {#cluster-status-system-tables}

- `tidb`テーブルには、TiDBに関するいくつかのグローバル情報が含まれています。

  - `bootstrapped`: TiDBクラスタが初期化されたかどうか。この値は読み取り専用であり、変更できません。
  - `tidb_server_version`: 初期化時のTiDBのバージョン情報。この値は読み取り専用であり、変更できません。
  - `system_tz`: TiDBのシステムタイムゾーン。
  - `new_collation_enabled`: TiDBが[照合順序の新しいフレームワーク](/character-set-and-collation.md#new-framework-for-collations)を有効にしているかどうか。この値は読み取り専用であり、変更できません。

## サーバーサイドヘルプシステムテーブル {#server-side-help-system-tables}

現在、`help_topic`はNULLです。

## 統計システムテーブル {#statistics-system-tables}

- `stats_buckets`: 統計のバケット
- `stats_histograms`: 統計のヒストグラム
- `stats_top_n`: 統計のTopN
- `stats_meta`: テーブルのメタ情報、行の総数や更新された行など
- `stats_extended`: 列間の順序相関など、拡張統計
- `stats_feedback`: 統計のクエリフィードバック
- `stats_fm_sketch`: 統計列のFMSketch分布のヒストグラム
- `analyze_options`: 各テーブルのデフォルトの`analyze`オプション
- `column_stats_usage`: 列統計の使用状況
- `schema_index_usage`: インデックスの使用状況
- `analyze_jobs`: 過去7日間の統計収集タスクと履歴タスクレコードの進行中の統計収集タスク

## 実行計画関連のシステムテーブル {#execution-plan-related-system-tables}

- `bind_info`: 実行計画のバインディング情報
- `capture_plan_baselines_blacklist`: 実行計画の自動バインディングのブロックリスト

## GCワーカーシステムテーブル {#gc-worker-system-tables}

> **Note:**
>
> GCワーカーシステムテーブルは、TiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

- `gc_delete_range`: 削除されるKV範囲
- `gc_delete_range_done`: 削除されたKV範囲

## キャッシュされたテーブルに関連するシステムテーブル {#system-tables-related-to-cached-tables}

- `table_cache_meta`は、キャッシュされたテーブルのメタデータを格納します。

## TTLに関連するシステムテーブル {#ttl-related-system-tables}

> **Note:**
>
> TTLに関連するシステムテーブルは、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。

- `mysql.tidb_ttl_table_status`は、以前に実行されたTTLジョブとすべてのTTLテーブルの進行中のTTLジョブを示します
- `mysql.tidb_ttl_task`は、現在進行中のTTLサブタスクです
- `mysql.tidb_ttl_job_history`は、過去90日間のTTLタスクの実行履歴です

## メタデータロックに関連するシステムテーブル {#system-tables-related-to-metadata-locks}

- `tidb_mdl_view`：メタデータロックのビュー。現在ブロックされているDDLステートメントに関する情報を表示するために使用できます
- `tidb_mdl_info`：ノード間でメタデータロックを同期するためにTiDBが内部的に使用する

## その他のシステムテーブル {#miscellaneous-system-tables}

> **Note:**
>
> `tidb`、`expr_pushdown_blacklist`、`opt_rule_blacklist`、および`table_cache_meta`システムテーブルは、TiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

- `GLOBAL_VARIABLES`: グローバルシステム変数テーブル
- `expr_pushdown_blacklist`: 式プッシュダウンのブロックリスト
- `opt_rule_blacklist`: 論理最適化ルールのブロックリスト
- `tidb_timers`: 内部タイマーのメタデータ
