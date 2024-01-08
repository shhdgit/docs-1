---
title: TiCDC Alert Rules
summary: Learn about TiCDC alert rules and how to handle the alerts.
---

# TiCDC アラートルール {#ticdc-alert-rules}

このドキュメントでは、TiCDCのアラートルールとそれに対応する解決策について説明します。深刻度のレベルは、降順で **Critical**、**Warning** です。

## Critical alerts {#critical-alerts}

このセクションでは、深刻なアラートとその解決策について紹介します。

### `cdc_checkpoint_high_delay` {#cdc-checkpoint-high-delay}

深刻なアラートでは、異常な監視メトリクスに注意を払う必要があります。

- アラートルール：

  (time() - ticdc\_owner\_checkpoint\_ts / 1000) > 600

- 説明：

  レプリケーションタスクが10分以上遅れています。

- 解決策：

  [TiCDC レプリケーションの中断を処理する](/ticdc/troubleshoot-ticdc.md#how-do-i-handle-replication-interruptions)を参照してください。

### `cdc_resolvedts_high_delay` {#cdc-resolvedts-high-delay}

- アラートルール：

  (time() - ticdc\_owner\_resolved\_ts / 1000) > 300

- 説明：

  レプリケーションタスクの Resolved TS が5分以上遅れています。

- 解決策：

  [TiCDC レプリケーションの中断を処理する](/ticdc/troubleshoot-ticdc.md#how-do-i-handle-replication-interruptions)を参照してください。

### `ticdc_processor_exit_with_error_count` {#ticdc-processor-exit-with-error-count}

- アラートルール：

  `changes(ticdc_processor_exit_with_error_count[1m]) > 0`

- 説明：

  レプリケーションタスクがエラーを報告して終了しました。

- 解決策：

  [TiCDC レプリケーションの中断を処理する](/ticdc/troubleshoot-ticdc.md#how-do-i-handle-replication-interruptions)を参照してください。

## Warning alerts {#warning-alerts}

警告アラートは、問題やエラーのリマインダーです。

### `cdc_multiple_owners` {#cdc-multiple-owners}

- アラートルール：

  `sum(rate(ticdc_owner_ownership_counter[30s])) >= 2`

- 説明：

  TiCDC クラスタに複数の所有者がいます。

- 解決策：

  ルート原因を特定するために TiCDC ログを収集してください。

### `cdc_sink_flush_duration_time_more_than_10s` {#cdc-sink-flush-duration-time-more-than-10s}

- アラートルール：

  `histogram_quantile(0.9, rate(ticdc_sink_txn_worker_flush_duration[1m])) > 10`

- 説明：

  レプリケーションタスクが下流データベースにデータを書き込むのに10秒以上かかります。

- 解決策：

  下流データベースに問題があるかどうかを確認してください。

### `cdc_processor_checkpoint_tso_no_change_for_1m` {#cdc-processor-checkpoint-tso-no-change-for-1m}

- アラートルール：

  `changes(ticdc_processor_checkpoint_ts[1m]) < 1`

- 説明：

  レプリケーションタスクが1分以上進んでいません。

- 解決策：

  [TiCDC レプリケーションの中断を処理する](/ticdc/troubleshoot-ticdc.md#how-do-i-handle-replication-interruptions)を参照してください。

### `ticdc_puller_entry_sorter_sort_bucket` {#ticdc-puller-entry-sorter-sort-bucket}

- アラートルール：

  `histogram_quantile(0.9, rate(ticdc_puller_entry_sorter_sort_bucket{}[1m])) > 1`

- 説明：

  TiCDC プーラーエントリーソーターの遅延が大きすぎます。

- 解決策：

  ルート原因を特定するために TiCDC ログを収集してください。

### `ticdc_puller_entry_sorter_merge_bucket` {#ticdc-puller-entry-sorter-merge-bucket}

- アラートルール：

  `histogram_quantile(0.9, rate(ticdc_puller_entry_sorter_merge_bucket{}[1m])) > 1`

- 説明：

  TiCDC プーラーエントリーソーターマージの遅延が大きすぎます。

- 解決策：

  ルート原因を特定するために TiCDC ログを収集してください。

### `tikv_cdc_min_resolved_ts_no_change_for_1m` {#tikv-cdc-min-resolved-ts-no-change-for-1m}

- アラートルール：

  `changes(tikv_cdc_min_resolved_ts[1m]) < 1 and ON (instance) tikv_cdc_region_resolve_status{status="resolved"} > 0`

- 説明：

  TiKV CDC の最小 Resolved TS 1 が1分以上進んでいません。

- 解決策：

  ルート原因を特定するために TiKV ログを収集してください。

### `tikv_cdc_scan_duration_seconds_more_than_10min` {#tikv-cdc-scan-duration-seconds-more-than-10min}

- アラートルール：

  `histogram_quantile(0.9, rate(tikv_cdc_scan_duration_seconds_bucket{}[1m])) > 600`

- 説明：

  TiKV CDC モジュールが増分レプリケーションを10分以上スキャンしました。

- 解決策：

  ルート原因を特定するために TiCDC 監視メトリクスと TiKV ログを収集してください。

### `ticdc_sink_mysql_execution_error` {#ticdc-sink-mysql-execution-error}

- アラートルール：

  `changes(ticdc_sink_mysql_execution_error[1m]) > 0`

- 説明：

  レプリケーションタスクが下流の MySQL にデータを書き込む際にエラーが発生しました。

- 解決策：

  多くの可能性が考えられます。[TiCDC のトラブルシューティング](/ticdc/troubleshoot-ticdc.md)を参照してください。

### `ticdc_memory_abnormal` {#ticdc-memory-abnormal}

- アラートルール：

  `go_memstats_heap_alloc_bytes{job="ticdc"} > 1e+10`

- 説明：

  TiCDC ヒープメモリの使用量が10 GiB を超えています。

- 解決策：

  ルート原因を特定するために TiCDC ログを収集してください。
