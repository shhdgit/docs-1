---
title: TiDB Monitoring API
summary: Learn the API of TiDB monitoring services.
---

# TiDBモニタリングAPI {#tidb-monitoring-api}

TiDBクラスターの状態を監視するために、次の種類のインターフェースを使用できます。

- [ステータスインターフェースを使用する](#use-the-status-interface): このインターフェースは、HTTPインターフェースを使用してコンポーネント情報を取得します。このインターフェースを使用すると、現在のTiDBサーバーの[実行状態](#running-status)とテーブルの[ストレージ情報](#storage-information)を取得できます。
- [メトリクスインターフェースを使用する](#use-the-metrics-interface): このインターフェースは、Prometheusを使用してコンポーネントのさまざまな操作の詳細情報を記録し、Grafanaを使用してこれらのメトリクスを表示します。

## ステータスインターフェースを使用する {#use-the-status-interface}

ステータスインターフェースは、TiDBクラスター内の特定のコンポーネントの基本情報を監視します。また、Keepaliveメッセージの監視インターフェースとしても機能します。さらに、PD（Placement Driver）のステータスインターフェースは、TiKVクラスター全体の詳細情報を取得できます。

### TiDBサーバー {#tidb-server}

- TiDB APIアドレス: `http://${host}:${port}`
- デフォルトポート: `10080`

### 実行状態 {#running-status}

次の例では、`http://${host}:${port}/status`を使用して、TiDBサーバーの現在の状態を取得し、サーバーが稼働しているかどうかを判断します。結果は**JSON**形式で返されます。

```bash
curl http://127.0.0.1:10080/status
{
    connections: 0,  # TiDBサーバーに接続しているクライアントの現在の数。
    version: "5.7.25-TiDB-v7.1.3",  # TiDBのバージョン番号。
    git_hash: "778c3f4a5a716880bcd1d71b257c8165685f0d70"  # 現在のTiDBコードのGitハッシュ。
}
```

#### ストレージ情報 {#storage-information}

次の例では、`http://${host}:${port}/schema_storage/${db}/${table}`を使用して、特定のデータテーブルのストレージ情報を取得します。結果は**JSON**形式で返されます。

```bash
curl http://127.0.0.1:10080/schema_storage/mysql/stats_histograms
```

```
{
    "table_schema": "mysql",
    "table_name": "stats_histograms",
    "table_rows": 0,
    "avg_row_length": 0,
    "data_length": 0,
    "max_data_length": 0,
    "index_length": 0,
    "data_free": 0
}
```

```bash
curl http://127.0.0.1:10080/schema_storage/test
```

```
[
    {
        "table_schema": "test",
        "table_name": "test",
        "table_rows": 0,
        "avg_row_length": 0,
        "data_length": 0,
        "max_data_length": 0,
        "index_length": 0,
        "data_free": 0
    }
]
```

### PDサーバー {#pd-server}

- PD APIアドレス: `http://${host}:${port}/pd/api/v1/${api_name}`
- デフォルトポート: `2379`
- API名の詳細については、[PD APIドキュメント](https://download.pingcap.com/pd-api-v1.html)を参照してください。

PDインターフェースは、すべてのTiKVサーバーの状態と負荷分散に関する情報を提供します。単一ノードのTiKVクラスターの情報については、次の例を参照してください。

```bash
curl http://127.0.0.1:2379/pd/api/v1/stores
{
  "count": 1,  # TiKVノードの数。
  "stores": [  # TiKVノードのリスト。
    # 単一のTiKVノードの詳細について。
    {
      "store": {
        "id": 1,
        "address": "127.0.0.1:20160",
        "version": "3.0.0-beta",
        "state_name": "Up"
      },
      "status": {
        "capacity": "20 GiB",  # 合計容量。
        "available": "16 GiB",  # 利用可能な容量。
        "leader_count": 17,
        "leader_weight": 1,
        "leader_score": 17,
        "leader_size": 17,
        "region_count": 17,
        "region_weight": 1,
        "region_score": 17,
        "region_size": 17,
        "start_ts": "2019-03-21T14:09:32+08:00",  # 開始タイムスタンプ。
        "last_heartbeat_ts": "2019-03-21T14:14:22.961171958+08:00",  # 最後のハートビートのタイムスタンプ。
        "uptime": "4m50.961171958s"
      }
    }
  ]
```

## メトリクスインターフェースを使用する {#use-the-metrics-interface}

メトリクスインターフェースは、TiDBクラスター全体の状態とパフォーマンスを監視します。

- 他のデプロイ方法を使用する場合は、このインターフェースを使用する前に、[PrometheusとGrafanaをデプロイ](/deploy-monitoring-services.md)してください。

PrometheusとGrafanaが正常にデプロイされたら、[Grafanaを構成](/deploy-monitoring-services.md#configure-grafana)してください。"
