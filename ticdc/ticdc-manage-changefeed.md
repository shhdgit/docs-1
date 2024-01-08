---
title: Manage Changefeeds
summary: Learn how to manage TiCDC changefeeds.
---

# チェンジフィードの管理 {#manage-changefeeds}

このドキュメントでは、TiCDCコマンドラインツール`cdc cli`を使用してTiCDCチェンジフィードを作成および管理する方法について説明します。また、TiCDCのHTTPインターフェースを介してチェンジフィードを管理することもできます。詳細については、[TiCDC OpenAPI](/ticdc/ticdc-open-api.md)を参照してください。

## レプリケーションタスクの作成 {#create-a-replication-task}

次のコマンドを実行してレプリケーションタスクを作成します：

```shell
cdc cli changefeed create --server=http://10.0.10.25:8300 --sink-uri="mysql://root:123456@127.0.0.1:3306/" --changefeed-id="simple-replication-task"
```

```shell
Create changefeed successfully!
ID: simple-replication-task
Info: {"upstream_id":7178706266519722477,"namespace":"default","id":"simple-replication-task","sink_uri":"mysql://root:xxxxx@127.0.0.1:4000/?time-zone=","create_time":"2023-12-21T15:05:46.679218+08:00","start_ts":438156275634929669,"engine":"unified","config":{"case_sensitive":false,"enable_old_value":true,"force_replicate":false,"ignore_ineligible_table":false,"check_gc_safe_point":true,"enable_sync_point":true,"bdr_mode":false,"sync_point_interval":30000000000,"sync_point_retention":3600000000000,"filter":{"rules":["test.*"],"event_filters":null},"mounter":{"worker_num":16},"sink":{"protocol":"","schema_registry":"","csv":{"delimiter":",","quote":"\"","null":"\\N","include_commit_ts":false},"column_selectors":null,"transaction_atomicity":"none","encoder_concurrency":16,"terminator":"\r\n","date_separator":"none","enable_partition_separator":false},"consistent":{"level":"none","max_log_size":64,"flush_interval":2000,"storage":""}},"state":"normal","creator_version":"v7.1.3"}
```

## レプリケーションタスクリストをクエリする {#query-the-replication-task-list}

次のコマンドを実行して、レプリケーションタスクリストをクエリします：

```shell
cdc cli changefeed list --server=http://10.0.10.25:8300
```

```shell
[{
    "id": "simple-replication-task",
    "summary": {
      "state": "normal",
      "tso": 417886179132964865,
      "checkpoint": "2020-07-07 16:07:44.881",
      "error": null
    }
}]
```

- `checkpoint`は、TiCDCがこの時点より前にデータを下流にレプリケートしたことを示します。
- `state`は、レプリケーションタスクの状態を示します。
  - `normal`: レプリケーションタスクは通常実行されます。
  - `stopped`: レプリケーションタスクは停止されています（手動で一時停止）。
  - `error`: レプリケーションタスクはエラーによって停止されています。
  - `removed`: レプリケーションタスクは削除されています。この状態のタスクは、`--all`オプションを指定した場合にのみ表示されます。このオプションが指定されていない場合は、`changefeed query`コマンドを実行してこれらのタスクを表示します。
  - `finished`: レプリケーションタスクは完了しています（データが`target-ts`にレプリケートされています）。この状態のタスクは、`--all`オプションを指定した場合にのみ表示されます。このオプションが指定されていない場合は、`changefeed query`コマンドを実行してこれらのタスクを表示します。

## 特定のレプリケーションタスクをクエリする {#query-a-specific-replication-task}

特定のレプリケーションタスクをクエリするには、`changefeed query`コマンドを実行します。クエリ結果には、タスク情報とタスクの状態が含まれます。`--simple`または`-s`引数を指定して、基本的なレプリケーション状態とチェックポイント情報のみを含むクエリ結果を簡略化することができます。この引数を指定しない場合は、詳細なタスク構成、レプリケーション状態、およびレプリケーションテーブル情報が出力されます。

```shell
cdc cli changefeed query -s --server=http://10.0.10.25:8300 --changefeed-id=simple-replication-task
```

```shell
{
 "state": "normal",
 "tso": 419035700154597378,
 "checkpoint": "2020-08-27 10:12:19.579",
 "error": null
}
```

前のコマンドと結果を比較すると：

- `state` は現在の変更フィードのレプリケーション状態です。各状態は `changefeed list` の状態と一致している必要があります。
- `tso` は現在の変更フィードで下流に正常にレプリケートされた最大トランザクション TSO を表します。
- `checkpoint` は現在の変更フィードで下流に正常にレプリケートされた最大トランザクション TSO の対応する時間を表します。
- `error` は現在の変更フィードでエラーが発生したかどうかを記録します。

```shell
cdc cli changefeed query --server=http://10.0.10.25:8300 --changefeed-id=simple-replication-task
```

```shell
{
  "info": {
    "sink-uri": "mysql://127.0.0.1:3306/?max-txn-row=20\u0026worker-number=4",
    "opts": {},
    "create-time": "2020-08-27T10:33:41.687983832+08:00",
    "start-ts": 419036036249681921,
    "target-ts": 0,
    "admin-job-type": 0,
    "sort-engine": "unified",
    "sort-dir": ".",
    "config": {
      "case-sensitive": false,
      "enable-old-value": false,
      "filter": {
        "rules": [
          "*.*"
        ],
        "ignore-txn-start-ts": null,
        "ddl-allow-list": null
      },
      "mounter": {
        "worker-num": 16
      },
      "sink": {
        "dispatchers": null,
      },
      "scheduler": {
        "type": "table-number",
        "polling-time": -1
      }
    },
    "state": "normal",
    "history": null,
    "error": null
  },
  "status": {
    "resolved-ts": 419036036249681921,
    "checkpoint-ts": 419036036249681921,
    "admin-job-type": 0
  },
  "count": 0,
  "task-status": [
    {
      "capture-id": "97173367-75dc-490c-ae2d-4e990f90da0f",
      "status": {
        "tables": {
          "47": {
            "start-ts": 419036036249681921
          }
        },
        "operation": null,
        "admin-job-type": 0
      }
    }
  ]
}
```

前のコマンドと結果を比較すると：

- `info`は、問い合わせられた変更フィードのレプリケーション構成です。
- `status`は、問い合わせられた変更フィードのレプリケーション状態です。
  - `resolved-ts`：現在の変更フィード内の最大トランザクション`TS`です。この`TS`はTiKVからTiCDCに正常に送信されていることに注意してください。
  - `checkpoint-ts`：現在の`changefeed`内の最大トランザクション`TS`です。この`TS`はダウンストリームに正常に書き込まれていることに注意してください。
  - `admin-job-type`：変更フィードの状態：
    - `0`：状態は正常です。
    - `1`：タスクが一時停止されています。タスクが一時停止されると、すべてのレプリケートされた`processor`が終了します。タスクの構成とレプリケーションの状態は保持されるため、`checkpiont-ts`からタスクを再開できます。
    - `2`：タスクが再開されています。レプリケーションタスクは`checkpoint-ts`から再開します。
    - `3`：タスクが削除されています。タスクが削除されると、すべてのレプリケートされた`processor`が終了し、レプリケーションタスクの構成情報がクリアされます。後でのクエリのためにレプリケーションの状態のみが保持されます。
- `task-status`は、問い合わせられた変更フィード内の各レプリケーションサブタスクの状態を示します。

## レプリケーションタスクを一時停止する {#pause-a-replication-task}

次のコマンドを実行して、レプリケーションタスクを一時停止します：

```shell
cdc cli changefeed pause --server=http://10.0.10.25:8300 --changefeed-id simple-replication-task
```

先行するコマンドで：

- `--changefeed-id=uuid` は、一時停止したレプリケーションタスクに対応するチェンジフィードのIDを表します。

## レプリケーションタスクを再開する {#resume-a-replication-task}

次のコマンドを実行して、一時停止したレプリケーションタスクを再開します：

```shell
cdc cli changefeed resume --server=http://10.0.10.25:8300 --changefeed-id simple-replication-task
```

- `--changefeed-id=uuid`は、再開したいレプリケーションタスクに対応するchangefeedのIDを表します。
- `--overwrite-checkpoint-ts`: v6.2.0から、レプリケーションタスクを再開する際の開始TSOを指定できます。TiCDCは指定されたTSOからデータを取得します。この引数は`now`または特定のTSO（たとえば434873584621453313など）を受け入れます。指定されたTSOは（GCセーフポイント、CurrentTSO]の範囲内でなければなりません。この引数が指定されていない場合、TiCDCはデフォルトで現在の`checkpoint-ts`からデータをレプリケートします。
- `--no-confirm`: レプリケーションが再開されると、関連情報を確認する必要はありません。デフォルトは`false`です。

> **Note:**
>
> - `--overwrite-checkpoint-ts`（`t2`）で指定されたTSOが、changefeedの現在のチェックポイントTSO（`t1`）よりも大きい場合、`t1`と` t2`の間のデータはダウンストリームにレプリケートされません。これによりデータが失われます。`cdc cli changefeed query`を実行することで`t1`を取得できます。
> - `--overwrite-checkpoint-ts`（`t2`）で指定されたTSOが、changefeedの現在のチェックポイントTSO（`t1`）よりも小さい場合、TiCDCは古い時点（`t2`）からデータを取得し、データの重複（たとえば、ダウンストリームがMQシンクの場合）を引き起こす可能性があります。

## レプリケーションタスクの削除 {#remove-a-replication-task}

次のコマンドを実行して、レプリケーションタスクを削除します：

```shell
cdc cli changefeed remove --server=http://10.0.10.25:8300 --changefeed-id simple-replication-task
```

先行するコマンドでは：

- `--changefeed-id=uuid` は、削除したいレプリケーションタスクに対応するchangefeedのIDを表します。

## タスク構成の更新 {#update-task-configuration}

TiCDCは、レプリケーションタスクの構成を変更することをサポートしています（動的ではありません）。 changefeedの構成を変更するには、タスクを一時停止し、構成を変更してからタスクを再開します。

```shell
cdc cli changefeed pause -c test-cf --server=http://10.0.10.25:8300
cdc cli changefeed update -c test-cf --server=http://10.0.10.25:8300 --sink-uri="mysql://127.0.0.1:3306/?max-txn-row=20&worker-number=8" --config=changefeed.toml
cdc cli changefeed resume -c test-cf --server=http://10.0.10.25:8300
```

現在、次の構成項目を変更できます。

- changefeedの`sink-uri`。
- changefeed構成ファイルとファイル内のすべての構成項目。
- changefeedの`target-ts`。

## レプリケーションサブタスク（`processor`）の処理ユニットを管理する {#manage-processing-units-of-replication-sub-tasks-processor}

- `processor`リストをクエリします：

  ```shell
  cdc cli processor list --server=http://10.0.10.25:8300
  ```

  ```shell
  [
          {
                  "id": "9f84ff74-abf9-407f-a6e2-56aa35b33888",
                  "capture-id": "b293999a-4168-4988-a4f4-35d9589b226b",
                  "changefeed-id": "simple-replication-task"
          }
  ]
  ```

- 特定のレプリケーションタスクのステータスに対応する特定のchangefeedをクエリします：

  ```shell
  cdc cli processor query --server=http://10.0.10.25:8300 --changefeed-id=simple-replication-task --capture-id=b293999a-4168-4988-a4f4-35d9589b226b
  ```

  ```shell
  {
    "status": {
      "tables": {
        "56": {    # 56 ID of the replication table, corresponding to tidb_table_id of a table in TiDB
          "start-ts": 417474117955485702
        }
      },
      "operation": null,
      "admin-job-type": 0
    },
    "position": {
      "checkpoint-ts": 417474143881789441,
      "resolved-ts": 417474143881789441,
      "count": 0
    }
  }
  ```

  前述のコマンドでは、以下が含まれます：

  - `status.tables`：各キー番号は、TiDBのテーブルの`tidb_table_id`に対応するレプリケーションテーブルのIDを表します。
  - `resolved-ts`：現在のprocessor内のソートされたデータの中で最大のTSO。
  - `checkpoint-ts`：現在のprocessorでダウンストリームに正常に書き込まれた最大のTSO。

## 行の変更イベントの過去の値を出力する {#output-the-historical-value-of-a-row-changed-event}

デフォルトの構成では、TiCDC Open ProtocolのRow Changed Eventは、変更前の値ではなく、変更された値のみがレプリケーションタスクで出力されます。したがって、出力値は、Row Changed Eventの過去の値としてTiCDC Open Protocolのコンシューマエンドで使用できません。

v4.0.5から、TiCDCは行の変更イベントの過去の値を出力する機能をサポートしています。この機能を有効にするには、ルートレベルのchangefeed構成ファイルで次の構成を指定します：

```toml
enable-old-value = true
```

この機能は、v5.0以降、デフォルトで有効になっています。この機能が有効になった後のTiCDCオープンプロトコルの出力形式については、[TiCDCオープンプロトコル - 行が変更されたイベント](/ticdc/ticdc-open-protocol.md#row-changed-event)を参照してください。

## 照合順序が有効になっている新しいフレームワークでテーブルをレプリケートする {#replicate-tables-with-the-new-framework-for-collations-enabled}

v4.0.15、v5.0.4、v5.1.1、v5.2.0から、TiCDCは[照合順序の新しいフレームワーク](/character-set-and-collation.md#new-framework-for-collations)が有効になっているテーブルをサポートしています。

## 有効なインデックスがないテーブルをレプリケートする {#replicate-tables-without-a-valid-index}

v4.0.8以降、TiCDCはタスク構成を変更することで有効なインデックスがないテーブルをレプリケートすることをサポートしています。この機能を有効にするには、changefeed構成ファイルで次のように設定してください：

```toml
enable-old-value = true
force-replicate = true
```

> **Warning:**
>
> バリッドなインデックスを持たないテーブルに対する`INSERT`や`REPLACE`などの操作は再入不可であり、データの冗長性のリスクがあります。TiCDCは、レプリケーションプロセス中にデータが少なくとも一度だけ分散されることを保証します。したがって、有効なインデックスを持たないテーブルをレプリケートするためにこの機能を有効にすると、データの冗長性が確実に発生します。データの冗長性を受け入れない場合は、`AUTO RANDOM`属性を持つプライマリキーカラムを追加するなど、有効なインデックスを追加することをお勧めします。

## 統一ソーター {#unified-sorter}

> **Note:**
>
> v6.0.0から、TiCDCはデフォルトでDBソーターエンジンを使用し、統一ソーターは使用しません。`sort engine`アイテムを構成しないことをお勧めします。

統一ソーターは、TiCDCのソートエンジンです。次のシナリオによって引き起こされるOOM問題を緩和することができます。

- TiCDCのデータレプリケーションタスクが長時間停止し、増分データが大量に蓄積され、レプリケートする必要がある場合。
- データレプリケーションタスクが早いタイムスタンプから開始されるため、大量の増分データをレプリケートする必要がある場合。

v4.0.13以降の`cdc cli`を使用して作成されたchangefeedについては、統一ソーターがデフォルトで有効になっています。v4.0.13より前に存在したchangefeedについては、以前の構成が使用されます。

changefeedで統一ソーター機能が有効になっているかどうかを確認するには、次の例のコマンドを実行できます（PDインスタンスのIPアドレスが`http://10.0.10.25:2379`であると仮定）。

```shell
cdc cli --server="http://10.0.10.25:8300" changefeed query --changefeed-id=simple-replication-task | grep 'sort-engine'
```

上記のコマンドの出力では、`sort-engine`の値が"unified"である場合、Unified Sorterがchangefeedで有効になっていることを意味します。

> **Note:**
>
> - サーバーが機械式ハードドライブや他のレイテンシーや帯域幅が制限されているストレージデバイスを使用している場合、Unified Sorterのパフォーマンスには大きな影響があります。
> - デフォルトでは、Unified Sorterは一時ファイルを保存するために`data_dir`を使用します。空きディスク容量が500 GiB以上であることを確認することをお勧めします。本番環境では、各ノードの空きディスク容量が（ビジネスが許可する最大の`checkpoint-ts`遅延）\*（ビジネスのピーク時のアップストリーム書き込みトラフィック）よりも大きいことをお勧めします。さらに、`changefeed`が作成された後に大量の履歴データをレプリケートする予定がある場合は、各ノードの空き容量がレプリケートされるデータの量よりも大きいことを確認してください。
