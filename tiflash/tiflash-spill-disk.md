---
title: TiFlash Spill to Disk
summary: 了解 TiFlash 如何将数据溢出到磁盘以及如何自定义溢出行为。
---

# TiFlash Spill to Disk

本文介绍了 TiFlash 在计算过程中如何将数据溢出到磁盘。

从 v7.0.0 版本开始，TiFlash 支持将中间结果溢出到磁盘，以缓解内存压力。支持的操作符包括：

* 具有等值连接条件的 Hash Join 操作符
* 具有 `GROUP BY` 键的 Hash Aggregation 操作符
* TopN 操作符，以及窗口函数中的 Sort 操作符

## 触发溢出

TiFlash 提供了两种触发将数据溢出到磁盘的机制。

* 操作符级别溢出：通过为每个操作符指定数据溢出阈值，可以控制何时将该操作符的中间结果溢出到磁盘。
* 查询级别溢出：通过指定 TiFlash 节点上的最大内存使用量以及溢出所占的内存比例，可以根据需要控制在查询中支持的操作符何时将数据溢出到磁盘。

### 操作符级别溢出

从 v7.0.0 版本开始，TiFlash 支持在操作符级别自动溢出。你可以使用以下系统变量控制每个操作符的溢出阈值。当操作符的内存使用超过阈值时，TiFlash 会触发该操作符的溢出。

* [`tidb_max_bytes_before_tiflash_external_group_by`](/system-variables.md#tidb_max_bytes_before_tiflash_external_group_by-new-in-v700)
* [`tidb_max_bytes_before_tiflash_external_join`](/system-variables.md#tidb_max_bytes_before_tiflash_external_join-new-in-v700)
* [`tidb_max_bytes_before_tiflash_external_sort`](/system-variables.md#tidb_max_bytes_before_tiflash_external_sort-new-in-v700)

#### 示例

此示例构造一个消耗大量内存的 SQL 语句，演示 Hash Aggregation 操作符的溢出。

1. 准备环境。创建一个包含 2 个节点的 TiFlash 集群，并导入 TPCH-100 数据。
2. 执行以下语句。这些语句未限制具有 `GROUP BY` 键的 Hash Aggregation 操作符的内存使用。

    ```sql
    SET tidb_max_bytes_before_tiflash_external_group_by = 0;
    SELECT
      l_orderkey,
      MAX(L_COMMENT),
      MAX(L_SHIPMODE),
      MAX(L_SHIPINSTRUCT),
      MAX(L_SHIPDATE),
      MAX(L_EXTENDEDPRICE)
    FROM lineitem
    GROUP BY l_orderkey
    HAVING SUM(l_quantity) > 314;
    ```

3. 从 TiFlash 的日志中可以看到，该查询在单个 TiFlash 节点上需要消耗 29.55 GiB 的内存：

    ```
    [DEBUG] [MemoryTracker.cpp:69] ["Peak memory usage (total): 29.55 GiB."] [source=MemoryTracker] [thread_id=468]
    ```

4. 执行以下语句。这条语句将具有 `GROUP BY` 键的 Hash Aggregation 操作符的内存使用限制为 10737418240（10 GiB）。

    ```sql
    SET tidb_max_bytes_before_tiflash_external_group_by = 10737418240;
    SELECT
      l_orderkey,
      MAX(L_COMMENT),
      MAX(L_SHIPMODE),
      MAX(L_SHIPINSTRUCT),
      MAX(L_SHIPDATE),
      MAX(L_EXTENDEDPRICE)
    FROM lineitem
    GROUP BY l_orderkey
    HAVING SUM(l_quantity) > 314;
    ```

5. 从 TiFlash 的日志中可以看到，通过配置 `tidb_max_bytes_before_tiflash_external_group_by`，TiFlash 触发了中间结果的溢出，显著减少了查询的内存使用。

    ```
    [DEBUG] [MemoryTracker.cpp:69] ["Peak memory usage (total): 12.80 GiB."] [source=MemoryTracker] [thread_id=110]
    ```

### 查询级别溢出

从 v7.4.0 版本开始，TiFlash 支持在查询级别自动溢出。你可以使用以下系统变量控制此功能：

* [`tiflash_mem_quota_query_per_node`](/system-variables.md#tiflash_mem_quota_query_per_node-new-in-v740)：限制在 TiFlash 节点上单个查询的最大内存使用量。
* [`tiflash_query_spill_ratio`](/system-variables.md#tiflash_query_spill_ratio-new-in-v740)：控制触发数据溢出的内存比例。

如果 `tiflash_mem_quota_query_per_node` 和 `tiflash_query_spill_ratio` 均设置为大于 0 的值，当查询的内存使用超过 `tiflash_mem_quota_query_per_node * tiflash_query_spill_ratio` 时，TiFlash 会自动触发支持的操作符的溢出。

#### 示例

此示例构造一个消耗大量内存的 SQL 语句，演示查询级别溢出。

1. 准备环境。创建一个包含 2 个节点的 TiFlash 集群，并导入 TPCH-100 数据。

2. 执行以下语句。这些语句未限制查询的内存使用或具有 `GROUP BY` 键的 Hash Aggregation 操作符的内存使用。

    ```sql
    SET tidb_max_bytes_before_tiflash_external_group_by = 0;
    SET tiflash_mem_quota_query_per_node = 0;
    SET tiflash_query_spill_ratio = 0;
    SELECT
      l_orderkey,
      MAX(L_COMMENT),
      MAX(L_SHIPMODE),
      MAX(L_SHIPINSTRUCT),
      MAX(L_SHIPDATE),
      MAX(L_EXTENDEDPRICE)
    FROM lineitem
    GROUP BY l_orderkey
    HAVING SUM(l_quantity) > 314;
    ```

3. 从 TiFlash 的日志中可以看到，该查询在单个 TiFlash 节点上消耗了 29.55 GiB 的内存：

    ```
    [DEBUG] [MemoryTracker.cpp:69] ["Peak memory usage (total): 29.55 GiB."] [source=MemoryTracker] [thread_id=468]
    ```

4. 执行以下语句。这些语句将限制在 TiFlash 节点上的最大查询内存为 5 GiB。

    ```sql
    SET tiflash_mem_quota_query_per_node = 5368709120;
    SET tiflash_query_spill_ratio = 0.7;
    SELECT
      l_orderkey,
      MAX(L_COMMENT),
      MAX(L_SHIPMODE),
      MAX(L_SHIPINSTRUCT),
      MAX(L_SHIPDATE),
      MAX(L_EXTENDEDPRICE)
    FROM lineitem
    GROUP BY l_orderkey
    HAVING SUM(l_quantity) > 314;
    ```

5. 从 TiFlash 的日志中可以看到，通过配置查询级别溢出，TiFlash 触发了中间结果的溢出，显著减少了查询的内存使用。

    ```
    [DEBUG] [MemoryTracker.cpp:101] ["Peak memory usage (for query): 3.94 GiB."] [source=MemoryTracker] [thread_id=1547]
    ```

## 注意事项

* 当 Hash Aggregation 操作符没有 `GROUP BY` 键时，不支持溢出。即使 Hash Aggregation 操作符包含去重聚合函数，也不支持溢出。
* 目前，操作符级别溢出的阈值是为每个操作符单独计算的。对于包含两个 Hash Aggregation 操作符的查询，如果未配置查询级别溢出且聚合操作符的阈值设置为 10 GiB，则这两个 Hash Aggregation 操作符只有在各自的内存使用超过 10 GiB 时才会溢出。
* 目前，Hash Aggregation 操作符和 TopN/Sort 操作符在恢复阶段采用合并聚合和合并排序算法。因此，这两个操作符只会触发一次溢出。如果内存需求非常高，且在恢复阶段的内存使用仍超过阈值，则不会再次触发溢出。
* 目前，Hash Join 操作符采用基于分区的溢出策略。如果在恢复阶段的内存使用仍超过阈值，则会再次触发溢出。但为了控制溢出的规模，溢出轮次限制为三次。如果在第三次溢出后，内存使用仍超过阈值，则不会再次触发溢出。
* 当配置查询级别溢出（即 [`tiflash_mem_quota_query_per_node`](/system-variables.md#tiflash_mem_quota_query_per_node-new-in-v740) 和 [`tiflash_query_spill_ratio`](/system-variables.md#tiflash_query_spill_ratio-new-in-v740) 均大于 0）时，TiFlash 会忽略单个操作符的溢出阈值，自动根据查询级别的阈值触发相关操作符的溢出。
* 即使配置了查询级别溢出，如果查询中使用的操作符都不支持溢出，该查询的中间计算结果仍无法溢出到磁盘。在这种情况下，当该查询的内存使用超过相关阈值时，TiFlash 会返回错误并终止查询。
* 即使配置了查询级别溢出，且查询包含支持溢出的操作符，若在以下任一场景中超出内存阈值，查询仍可能返回错误：
    - 查询中的其他非溢出操作符占用过多内存。
    - 溢出操作符未能及时将数据溢出到磁盘。

  为了应对溢出操作符未能及时溢出到磁盘的情况，你可以尝试将 [`tiflash_query_spill_ratio`](/system-variables.md#tiflash_query_spill_ratio-new-in-v740) 降低，以避免内存阈值错误。