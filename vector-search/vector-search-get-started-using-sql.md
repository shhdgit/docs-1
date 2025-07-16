---
title: 使用 SQL 快速入门向量搜索
summary: 了解如何在 TiDB 中使用 SQL 语句快速开始向量搜索，以支持你的生成式 AI 应用。
---

# 使用 SQL 快速入门向量搜索

TiDB 扩展了 MySQL 语法，支持 [Vector Search](/vector-search/vector-search-overview.md)，引入了新的 [Vector data types](/vector-search/vector-search-data-types.md) 和多个 [vector functions](/vector-search/vector-search-functions-and-operators.md)。

本教程演示了如何仅通过 SQL 语句开始使用 TiDB 向量搜索。你将学习如何使用 [MySQL command-line client](https://dev.mysql.com/doc/refman/8.4/en/mysql.html) 完成以下操作：

- 连接到你的 TiDB 集群。
- 创建向量表。
- 存储向量嵌入。
- 执行向量搜索查询。

<CustomContent platform="tidb">

> **Warning:**
>
> 向量搜索功能处于实验阶段。不建议在生产环境中使用此功能。此功能可能在不提前通知的情况下进行更改。如发现 bug，可以在 GitHub 上提交 [issue](https://github.com/pingcap/tidb/issues)。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> 向量搜索功能处于测试版，可能会在不提前通知的情况下进行更改。如发现 bug，可以在 GitHub 上提交 [issue](https://github.com/pingcap/tidb/issues)。

</CustomContent>

> **Note:**
>
> 向量搜索功能在 TiDB Self-Managed、[{{{ .starter }}}](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-cloud-serverless) 和 [TiDB Cloud Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-cloud-dedicated) 上均可用。对于 TiDB Self-Managed 和 TiDB Cloud Dedicated，TiDB 版本必须为 v8.4.0 或更高（建议 v8.5.0 或更高版本）。

## 前提条件

完成本教程，你需要：

- 在你的机器上安装 [MySQL command-line client](https://dev.mysql.com/doc/refman/8.4/en/mysql.html)（MySQL CLI）。
- 一个 TiDB 集群。

<CustomContent platform="tidb">

**如果你还没有 TiDB 集群，可以按照以下方式创建：**

- 参考 [部署本地测试 TiDB 集群](/quick-start-with-tidb.md#deploy-a-local-test-cluster) 或 [部署生产环境 TiDB 集群](/production-deployment-using-tiup.md) 来创建本地集群。
- 参考 [创建 {{{ .starter }}} 集群](/develop/dev-guide-build-cluster-in-cloud.md) 来创建你自己的 TiDB Cloud 集群。

</CustomContent>
<CustomContent platform="tidb-cloud">

**如果你还没有 TiDB 集群，可以按照以下方式创建：**

- （推荐）参考 [创建 {{{ .starter }}} 集群](/develop/dev-guide-build-cluster-in-cloud.md) 来创建你自己的 TiDB Cloud 集群。
- 参考 [部署本地测试 TiDB 集群](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster) 或 [部署生产环境 TiDB 集群](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) 来创建版本为 v8.4.0 或更高的本地集群。

</CustomContent>

## 开始操作

### 第一步。连接到 TiDB 集群

根据你选择的 TiDB 部署方式，连接到你的 TiDB 集群。

<SimpleTab>
<div label="{{{ .starter }}}">

1. 进入 [**Clusters**](https://tidbcloud.com/console/clusters) 页面，然后点击目标集群的名称，进入其概览页面。

2. 点击右上角的 **Connect**。会显示连接对话框。

3. 在连接对话框中，从 **Connect With** 下拉列表选择 **MySQL CLI**，并保持 **Connection Type** 的默认设置为 **Public**。

4. 如果还没有设置密码，点击 **Generate Password** 生成一个随机密码。

5. 复制连接命令，并粘贴到你的终端中。以下是 macOS 的示例：

    ```bash
    mysql -u '<prefix>.root' -h '<host>' -P 4000 -D 'test' --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p'<password>'
    ```

</div>
<div label="TiDB Self-Managed">

在你的 TiDB Self-Managed 集群启动后，在终端中执行你的集群连接命令。

以下是 macOS 的示例连接命令：

```bash
mysql --comments --host 127.0.0.1 --port 4000 -u root
```

</div>

</SimpleTab>

### 第二步。创建向量表

在创建表时，可以通过指定 `VECTOR` 数据类型，将某一列定义为 [vector](/vector-search/vector-search-overview.md#vector-embedding) 列。

例如，要创建一个名为 `embedded_documents`，包含一个三维 `VECTOR` 列的表，使用你的 MySQL CLI 执行以下 SQL 语句：

```sql
USE test;
CREATE TABLE embedded_documents (
    id        INT       PRIMARY KEY,
    -- 存储文档原始内容的列
    document  TEXT,
    -- 存储文档向量表示的列
    embedding VECTOR(3)
);
```

预期输出如下：

```text
Query OK, 0 rows affected (0.27 sec)
```

### 第三步。插入向量嵌入

向 `embedded_documents` 表中插入三个文档及其 [vector embeddings](/vector-search/vector-search-overview.md#vector-embedding)：

```sql
INSERT INTO embedded_documents
VALUES
    (1, 'dog', '[1,2,1]'),
    (2, 'fish', '[1,2,4]'),
    (3, 'tree', '[1,0,0]');
```

预期输出如下：

```
Query OK, 3 rows affected (0.15 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

> **Note**
>
> 这个示例简化了向量嵌入的维度，只使用了 3 维向量进行演示。
>
> 在实际应用中，[embedding models](/vector-search/vector-search-overview.md#embedding-model) 通常会生成数百或数千维的向量嵌入。

### 第四步。查询向量表

为了验证文档是否正确插入，可以查询 `embedded_documents` 表：

```sql
SELECT * FROM embedded_documents;
```

预期输出如下：

```sql
+----+----------+-----------+
| id | document | embedding |
+----+----------+-----------+
|  1 | dog      | [1,2,1]   |
|  2 | fish     | [1,2,4]   |
|  3 | tree     | [1,0,0]   |
+----+----------+-----------+
3 rows in set (0.15 sec)
```

### 第五步。执行向量搜索查询

类似全文搜索，用户在使用向量搜索时会向应用提供搜索词。

在本例中，搜索词为 “a swimming animal”，其对应的向量嵌入假设为 `[1,2,3]`。在实际应用中，你需要使用嵌入模型将用户的搜索词转换为向量嵌入。

执行以下 SQL 语句，TiDB 将通过计算并排序表中向量嵌入的余弦距离 (`vec_cosine_distance`) 来识别与 `[1,2,3]` 最接近的前三个文档。

```sql
SELECT id, document, vec_cosine_distance(embedding, '[1,2,3]') AS distance
FROM embedded_documents
ORDER BY distance
LIMIT 3;
```

预期输出如下：

```plain
+----+----------+---------------------+
| id | document | distance            |
+----+----------+---------------------+
|  2 | fish     | 0.00853986601633272 |
|  1 | dog      | 0.12712843905603044 |
|  3 | tree     |  0.7327387580875756 |
+----+----------+---------------------+
3 rows in set (0.15 sec)
```

搜索结果中的三个词条按照与查询向量的距离排序：距离越小，相关性越高。

因此，根据输出，最有可能的“会游泳的动物”是鱼，或者是擅长游泳的狗。