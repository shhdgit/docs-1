---
title: Optimizer Fix Controls
summary: Learn about the Optimizer Fix Controls feature and how to use `tidb_opt_fix_control` to control the TiDB optimizer in a more fine-grained way.
---

# Optimizer Fix Controls {#optimizer-fix-controls}

製品が反復的に進化するにつれて、TiDBオプティマイザの動作が変わり、それによってより合理的な実行計画が生成されます。ただし、特定のシナリオでは、新しい動作が予期しない結果をもたらすことがあります。例：

- 一部の動作の効果は特定のシナリオに依存します。ほとんどのシナリオに改善をもたらす変更が、他のシナリオには回帰を引き起こす可能性があります。
- 時々、動作の詳細とその結果との関係は非常に複雑です。特定の動作の改善が全体的な回帰を引き起こす可能性があります。

そのため、TiDBはオプティマイザの動作を細かく制御するために、オプティマイザの修正コントロール機能を提供しています。このドキュメントでは、オプティマイザの修正コントロール機能とその使用方法、およびTiDBが現在サポートしているすべての修正をリストします。

## `tidb_opt_fix_control`の紹介 {#introduction-to-tidb-opt-fix-control}

v7.1.0から、TiDBは[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v657-and-v710)システム変数を提供し、オプティマイザの動作をより細かく制御できるようになりました。

各修正は、TiDBオプティマイザの特定の目的のために動作を調整するために使用される制御項目です。それは、動作変更の技術的な詳細を含むGitHub Issueに対応する番号で示されます。たとえば、修正`44262`の場合、[Issue 44262](https://github.com/pingcap/tidb/issues/44262)でそれが制御する内容を確認できます。

[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v657-and-v710)システム変数は、複数の修正を1つの値として受け入れ、コンマ（`,`）で区切っています。フォーマットは`"<#issue1>:<value1>,<#issue2>:<value2>,...,<#issueN>:<valueN>"`で、`<#issueN>`は修正番号です。たとえば：

```sql
SET SESSION tidb_opt_fix_control = '44262:ON,44389:ON';
```

## オプティマイザの修正コントロール参照 {#optimizer-fix-controls-reference}

### [`44262`](https://github.com/pingcap/tidb/issues/44262) <span class="version-mark">v7.1.1で新規</span> {#44262-https-github-com-pingcap-tidb-issues-44262-span-class-version-mark-new-in-v7-1-1-span}

- デフォルト値：`OFF`
- 可能な値：`ON`、`OFF`
- この変数は、[動的プルーニングモード](/partitioned-table.md#dynamic-pruning-mode)を使用して[GlobalStats](/statistics.md#collect-statistics-of-partitioned-tables-in-dynamic-pruning-mode)にアクセスするかどうかを制御します。

### [`44389`](https://github.com/pingcap/tidb/issues/44389) <span class="version-mark">v7.1.1で新規</span> {#44389-https-github-com-pingcap-tidb-issues-44389-span-class-version-mark-new-in-v7-1-1-span}

- デフォルト値：`OFF`
- 可能な値：`ON`、`OFF`
- `c = 10 and (a = 'xx' or (a = 'kk' and b = 1))`などのフィルタについて、この変数は`IndexRangeScan`のためにより包括的なスキャン範囲を構築しようとするかどうかを制御します。

### [`44823`](https://github.com/pingcap/tidb/issues/44823) <span class="version-mark">v7.1.1で新規</span> {#44823-https-github-com-pingcap-tidb-issues-44823-span-class-version-mark-new-in-v7-1-1-span}

- デフォルト値：`200`
- 可能な値：`[0, 2147483647]`
- メモリを節約するために、Plan Cacheは指定されたパラメータ数を超えるクエリをキャッシュしません。
- この変数は、最大パラメータ数の閾値を制御します。`0`は制限なしを意味します。

### [`44855`](https://github.com/pingcap/tidb/issues/44855) <span class="version-mark">v7.1.1で新規</span> {#44855-https-github-com-pingcap-tidb-issues-44855-span-class-version-mark-new-in-v7-1-1-span}

- デフォルト値：`OFF`
- 可能な値：`ON`、`OFF`
- 一部のシナリオでは、`IndexJoin`演算子の`Probe`側に`Selection`演算子が含まれる場合、TiDBは`IndexScan`の行数を過大評価することがあります。これにより、`IndexJoin`の代わりにサブ最適なクエリプランが選択される可能性があります。
- この問題を緩和するために、TiDBは改善を導入しました。ただし、潜在的なクエリプランのフォールバックリスクがあるため、この改善はデフォルトで無効になっています。
- この変数は、前述の改善を有効にするかどうかを制御します。
