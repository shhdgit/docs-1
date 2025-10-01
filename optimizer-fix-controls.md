---
title: Optimizer Fix Controls
summary: 了解 Optimizer Fix Controls 特性，以及如何使用 `tidb_opt_fix_control` 以更细粒度地控制 TiDB 优化器。
---

# Optimizer Fix Controls

随着产品的迭代演进，TiDB 优化器的行为也在不断变化，从而生成更合理的执行计划。然而，在某些特定场景下，新的行为可能会导致意外的结果。例如：

- 某些行为的效果依赖于特定场景，对大多数场景带来提升的变更，可能会对其他场景造成回退。
- 有时，行为细节的变化与其后果之间的关系非常复杂，对某一行为的改进可能会导致整体回退。

因此，TiDB 提供了 Optimizer Fix Controls 特性，允许你通过为一组修复项设置值，对 TiDB 优化器的行为进行细粒度的控制。本文档介绍了 Optimizer Fix Controls 特性及其使用方法，并列出了 TiDB 当前支持的所有 Optimizer Fix Controls 修复项。

## `tidb_opt_fix_control` 简介

从 v6.5.3 和 v7.1.0 开始，TiDB 提供了 [`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v653-and-v710) 系统变量，以更细粒度地控制优化器的行为。

每个修复项都是用于调整 TiDB 优化器某一特定行为的控制项。它由一个数字表示，该数字对应一个包含行为变更技术细节的 GitHub Issue。例如，对于修复项 `44262`，你可以在 [Issue 44262](https://github.com/pingcap/tidb/issues/44262) 中查看其控制内容。

[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v653-and-v710) 系统变量可以接受多个修复项作为一个值，使用逗号（`,`）分隔。格式为 `"<#issue1>:<value1>,<#issue2>:<value2>,...,<#issueN>:<valueN>"`，其中 `<#issueN>` 是修复项编号。例如：

```sql
SET SESSION tidb_opt_fix_control = '44262:ON,44389:ON';
```

## Optimizer Fix Controls 参考

### [`33031`](https://github.com/pingcap/tidb/issues/33031) <span class="version-mark">New in v8.0.0</span>

- 默认值：`OFF`
- 可选值：`ON`，`OFF`
- 此变量控制是否允许分区表使用计划缓存。如果设置为 `ON`，则 [Prepared statement plan cache](/sql-prepared-plan-cache.md) 和 [Non-prepared statement plan cache](/sql-non-prepared-plan-cache.md) 都不会对 [分区表](/partitioned-table.md) 生效。

### [`44262`](https://github.com/pingcap/tidb/issues/44262) <span class="version-mark">New in v6.5.3 and v7.2.0</span>

- 默认值：`OFF`
- 可选值：`ON`，`OFF`
- 此变量控制当 [全局统计信息](/statistics.md#collect-statistics-of-partitioned-tables-in-dynamic-pruning-mode) 缺失时，是否允许使用 [动态裁剪模式](/partitioned-table.md#dynamic-pruning-mode) 访问分区表。

### [`44389`](https://github.com/pingcap/tidb/issues/44389) <span class="version-mark">New in v6.5.3 and v7.2.0</span>

- 默认值：`OFF`
- 可选值：`ON`，`OFF`
- 对于如 `c = 10 and (a = 'xx' or (a = 'kk' and b = 1))` 这样的过滤条件，此变量控制是否尝试为 `IndexRangeScan` 构建更全面的扫描范围。

### [`44823`](https://github.com/pingcap/tidb/issues/44823) <span class="version-mark">New in v7.3.0</span>

- 默认值：`200`
- 可选值：`[0, 2147483647]`
- 为了节省内存，Plan Cache 不会缓存参数数量超过该变量指定值的查询。`0` 表示不限制。

### [`44830`](https://github.com/pingcap/tidb/issues/44830) <span class="version-mark">New in v6.5.7 and v7.3.0</span>

- 默认值：`OFF`
- 可选值：`ON`，`OFF`
- 此变量控制 Plan Cache 是否允许缓存物理优化阶段生成的包含 `PointGet` 算子的执行计划。

### [`44855`](https://github.com/pingcap/tidb/issues/44855) <span class="version-mark">New in v6.5.4 and v7.3.0</span>

- 默认值：`OFF`
- 可选值：`ON`，`OFF`
- 在某些场景下，当 `IndexJoin` 算子的 `Probe` 端包含 `Selection` 算子时，TiDB 会严重高估 `IndexScan` 的行数。这可能导致选择了次优的查询计划而不是 `IndexJoin`。
- 为缓解该问题，TiDB 引入了一项改进。但由于存在查询计划回退的风险，该改进默认关闭。
- 此变量控制是否启用上述改进。

### [`45132`](https://github.com/pingcap/tidb/issues/45132) <span class="version-mark">New in v7.4.0</span>

- 默认值：`1000`
- 可选值：`[0, 2147483647]`
- 此变量设置优化器启发式选择访问路径的阈值。如果某个访问路径（如 `Index_A`）的估算行数远小于其他访问路径（默认 `1000` 倍），优化器会跳过成本比较，直接选择 `Index_A`。
- `0` 表示禁用该启发式策略。

### [`45798`](https://github.com/pingcap/tidb/issues/45798) <span class="version-mark">New in v7.5.0</span>

- 默认值：`ON`
- 可选值：`ON`，`OFF`
- 此变量控制 Plan Cache 是否允许缓存访问 [生成列](/generated-columns.md) 的执行计划。

### [`46177`](https://github.com/pingcap/tidb/issues/46177) <span class="version-mark">New in v6.5.6, v7.1.3 and v7.5.0</span>

- 默认值：`ON`。v8.5.0 之前的默认值为 `OFF`。
- 可选值：`ON`，`OFF`
- 此变量控制在查询优化过程中，优化器在找到未强制的计划后，是否继续探索强制计划。

### [`47400`](https://github.com/pingcap/tidb/issues/47400) <span class="version-mark">New in v8.4.0</span>

- 默认值：`ON`
- 可选值：`ON`，`OFF`
- 由于在查询计划中准确估算每一步合格行数存在挑战，优化器可能会对 `estRows` 估算出较小的值。此变量控制是否限制 `estRows` 的最小值。
- `ON`：将 `estRows` 的最小值限制为 1，这是 v8.4.0 引入的新行为，并与 Oracle、Db2 等其他数据库保持一致。
- `OFF`：不限制最小行数估算，行为与 v8.4.0 之前版本一致，此时 `estRows` 可能为 0。

### [`52592`](https://github.com/pingcap/tidb/issues/52592) <span class="version-mark">New in v8.4.0</span>

- 默认值：`OFF`
- 可选值：`ON`，`OFF`
- 此变量控制是否禁用查询执行中的 `Point Get` 和 `Batch Point Get` 算子。默认值 `OFF` 表示允许使用 `Point Get` 和 `Batch Point Get` 执行查询。如果设置为 `ON`，优化器会禁用 `Point Get` 和 `Batch Point Get`，强制选择 Coprocessor 执行查询。
- `Point Get` 和 `Batch Point Get` 不支持列裁剪（即无法只返回部分列），因此在某些场景下，其执行效率可能低于 Coprocessor，将该变量设置为 `ON` 可以提升查询性能。以下场景推荐将该变量设置为 `ON`：

    - 宽表，包含大量列，但只查询少量列。
    - 表中包含大体积 JSON 字段，但查询时不涉及该 JSON 字段，或只查询 JSON 字段的一小部分。

### [`52869`](https://github.com/pingcap/tidb/issues/52869) <span class="version-mark">New in v8.1.0</span>

- 默认值：`OFF`
- 可选值：`ON`，`OFF`
- 如 [Explain Statements Using Index Merge](/explain-index-merge.md#examples) 的 **Note** 所述，如果优化器可以为查询计划选择单一索引扫描方式（非全表扫描），则不会自动使用 index merge。
- 你可以通过启用该修复项移除此限制。移除该限制后，优化器可以在更多查询中自动选择 index merge，但也可能导致优化器忽略最优的执行计划。因此，建议在实际使用场景中充分测试后再移除此限制，以确保不会引发性能回退。

### [`54337`](https://github.com/pingcap/tidb/issues/54337) <span class="version-mark">New in v8.3.0</span>

- 默认值：`OFF`
- 可选值：`ON`，`OFF`
- 目前，TiDB 优化器在推导每个子句为范围列表的复杂合取条件的索引范围时存在一定限制。通过应用通用范围交集可以解决该问题。
- 你可以通过启用该修复项移除此限制，使优化器能够处理复杂的范围交集。但对于合取条件数量较多（超过 10 个）的情况，优化时间可能会略有增加。

### [`56318`](https://github.com/pingcap/tidb/issues/56318)

> **Note:**
>
> 仅适用于 [TiDB Cloud Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-cloud-serverless)。

- 默认值：`ON`
- 可选值：`ON`，`OFF`
- 此变量控制是否避免在 `ORDER BY` 语句中对复杂表达式进行两次计算。