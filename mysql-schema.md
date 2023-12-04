---
title: mysql Schema
summary: Learn about the TiDB system tables.
---

# `mysql` スキーマ {#mysql-schema}

`mysql` スキーマにはTiDBシステムテーブルが含まれています。デザインはMySQLの`mysql`スキーマに類似しており、`mysql.user`などのテーブルは直接編集することができます。また、MySQLの拡張機能としていくつかのテーブルも含まれています。

## 権限システムテーブル {#grant-system-tables}

これらのシステムテーブルには、ユーザーアカウントとその権限に関する権限情報が含まれています：

-   `user`: ユーザーアカウント、グローバル権限、およびその他の非権限列
-   `db`: データベースレベルの権限
-   `tables_priv`: テーブルレベルの権限
-   `columns_priv`: カラムレベルの権限
-   `password_history`: パスワード変更履歴
-   `default_roles`: ユーザーのデフォルトの役割
-   `global_grants`: 動的な権限
-   `global_priv`: 証明書に基づく認証情報
-   `role_edges`: 役割間の関係

## クラスターステータスシステムテーブル {#cluster-status-system-tables}

-   `tidb` テーブルにはTiDBに関するいくつかのグローバル情報が含まれています：

    -   `bootstrapped`: TiDBクラスターが初期化されたかどうか。この値は読み取り専用であり、変更することはできません。
    -   `tidb_server_version`: TiDBが初期化されたときのバージョン情報。この値は読み取り専用であり、変更することはできません。
    -   `system_tz`: TiDBのシステムタイムゾーン。
    -   `new_collation_enabled`: TiDBが[新しい照合フレームワーク](/character-set-and-collation.md#new-framework-for-collations)を有効にしているかどうか。この値は読み取り専用であり、変更することはできません。

## サーバーサイドヘルプシステムテーブル {#server-side-help-system-tables}

現在、`help_topic` はNULLです。

## 統計システムテーブル {#statistics-system-tables}

-   `stats_buckets`: 統計のバケット
-   `stats_histograms`: 統計のヒストグラム
-   `stats_top_n`: 統計のTopN
-   `stats_meta`: テーブルのメタ情報、行の総数や更新された行など
-   `stats_extended`: 列間の順序相関など、拡張統計
-   `stats_feedback`: 統計のクエリフィードバック
-   `stats_fm_sketch`: 統計列のヒストグラムのFMSketch分布
-   `analyze_options`: 各テーブルのデフォルトの`analyze`オプション
-   `column_stats_usage`: カラム統計の使用状況
-   `schema_index_usage`: インデックスの使用状況
-   `analyze_jobs`: 過去7日間の進行中の統計収集タスクと履歴タスクレコード

## 実行計画関連のシステムテーブル {#execution-plan-related-system-tables}

-   `bind_info`: 実行計画のバインディング情報
-   `capture_plan_baselines_blacklist`: 実行計画の自動バインディングのブロックリスト

## GCワーカーシステムテーブル {#gc-worker-system-tables}

> **注意:**
>
> GCワーカーシステムテーブルはTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

-   `gc_delete_range`: 削除されるKV範囲
-   `gc_delete_range_done`: 削除されたKV範囲

## キャッシュされたテーブルに関連するシステムテーブル {#system-tables-related-to-cached-tables}

-   `table_cache_meta` はキャッシュされたテーブルのメタデータを格納します。

## TTLに関連するシステムテーブル {#ttl-related-system-tables}

> **注意:**
>
> TTLに関連するシステムテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

-   `mysql.tidb_ttl_table_status` すべてのTTLテーブルの以前に実行されたTTLジョブと現在進行中のTTLジョブ
-   `mysql.tidb_ttl_task` 現在進行中のTTLサブタスク
-   `mysql.tidb_ttl_job_history` 過去90日間のTTLタスクの実行履歴

## メタデータロックに関連するシステムテーブル {#system-tables-related-to-metadata-locks}

-   `tidb_mdl_view`：メタデータロックのビュー。現在ブロックされているDDLステートメントに関する情報を表示するために使用できます
-   `tidb_mdl_info`：TiDB内部で使用され、ノード間でメタデータロックを同期するために使用されます

## その他のシステムテーブル {#miscellaneous-system-tables}

> **注意:**
>
> `tidb`、`expr_pushdown_blacklist`、`opt_rule_blacklist`、および`table_cache_meta`システムテーブルはTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

-   `GLOBAL_VARIABLES`: グローバルシステム変数テーブル
-   `expr_pushdown_blacklist`: 式プッシュダウンのブロックリスト
-   `opt_rule_blacklist`: 論理最適化ルールのブロックリスト
-   `table_cache_meta`: キャッシュされたテーブルのメタデータ
