---
title: TiCDC OpenAPI v2
summary: Learn how to use the OpenAPI v2 interface to manage the cluster status and data replication.
---

# TiCDC OpenAPI v2 {#ticdc-openapi-v2}

<!-- markdownlint-disable MD024 -->

TiCDCは、TiCDCクラスターのクエリと操作のためのOpenAPI機能を提供します。OpenAPI機能は、[`cdc cli`ツール](/ticdc/ticdc-manage-changefeed.md)のサブセットです。

> **Note:**
>
> TiCDC OpenAPI v1は将来的に削除されます。TiCDC OpenAPI v2の使用を推奨します。

TiCDCクラスターで次のメンテナンス操作を実行するためにAPIを使用できます。

- [TiCDCノードのステータス情報を取得](#get-the-status-information-of-a-ticdc-node)
- [TiCDCクラスターのヘルスステータスを確認](#check-the-health-status-of-a-ticdc-cluster)
- [レプリケーションタスクを作成](#create-a-replication-task)
- [レプリケーションタスクを削除](#remove-a-replication-task)
- [レプリケーション構成を更新](#update-the-replication-configuration)
- [レプリケーションタスクリストをクエリ](#query-the-replication-task-list)
- [特定のレプリケーションタスクをクエリ](#query-a-specific-replication-task)
- [レプリケーションタスクを一時停止](#pause-a-replication-task)
- [レプリケーションタスクを再開](#resume-a-replication-task)
- [レプリケーションサブタスクリストをクエリ](#query-the-replication-subtask-list)
- [特定のレプリケーションサブタスクをクエリ](#query-a-specific-replication-subtask)
- [TiCDCサービスプロセスリストをクエリ](#query-the-ticdc-service-process-list)
- [所有者ノードを追放](#evict-an-owner-node)
- [TiCDCサーバーのログレベルを動的に調整](#dynamically-adjust-the-log-level-of-the-ticdc-server)

すべてのAPIのリクエストボディと返される値はJSON形式です。成功したリクエストは`200 OK`メッセージを返します。次のセクションでは、APIの具体的な使用方法について説明します。

次の例では、TiCDCサーバーのリッスンIPアドレスは`127.0.0.1`で、ポートは`8300`です。TiCDCサーバーを起動する際にTiCDCにバインドされたIPアドレスとポートを`--addr=ip:port`で指定できます。

## APIエラーメッセージテンプレート {#api-error-message-template}

APIリクエストを送信した後、エラーが発生した場合、返されるエラーメッセージは次の形式です。

```json
{
    "error_msg": "",
    "error_code": ""
}
```

上記のJSON出力では、`error_msg`がエラーメッセージを説明し、`error_code`が対応するエラーコードを示しています。

## APIリストインターフェースの戻り値形式 {#return-format-of-the-api-list-interface}

APIリクエストがリソースのリスト（例えば、すべての`Captures`のリスト）を返す場合、TiCDCの戻り値形式は以下のようになります：

```json
{
  "total": 2,
  "items": [
    {
      "id": "d2912e63-3349-447c-90ba-wwww",
      "is_owner": true,
      "address": "127.0.0.1:8300"
    },
    {
      "id": "d2912e63-3349-447c-90ba-xxxx",
      "is_owner": false,
      "address": "127.0.0.1:8302"
    }
  ]
}
```

上記の例では：

- `total`: リソースの合計数を示します。
- `items`: このリクエストによって返されるすべてのリソースを含む配列です。配列のすべての要素は同じリソースです。

## TiCDCノードのステータス情報を取得する {#get-the-status-information-of-a-ticdc-node}

このAPIは同期インターフェースです。リクエストが成功した場合、対応するノードのステータス情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/status`

### 例 {#example}

以下のリクエストは、IPアドレスが`127.0.0.1`でポート番号が`8300`であるTiCDCノードのステータス情報を取得します。

```shell
curl -X GET http://127.0.0.1:8300/api/v2/status
```

```json
{
  "version": "v7.1.3",
  "git_hash": "10413bded1bdb2850aa6d7b94eb375102e9c44dc",
  "id": "d2912e63-3349-447c-90ba-72a4e04b5e9e",
  "pid": 1447,
  "is_owner": true,
  "liveness": 0
}
```

上記の出力のパラメータは次のように説明されています：

- `version`: TiCDCの現在のバージョン番号。
- `git_hash`: Gitハッシュ値。
- `id`: ノードのキャプチャID。
- `pid`: ノードのキャプチャプロセスID（PID）。
- `is_owner`: ノードが所有者であるかどうかを示します。
- `liveness`: このノードがライブであるかどうか。 `0` は通常を意味します。 `1` はノードが `graceful shutdown` 状態にあることを意味します。

## TiCDCクラスターのヘルスステータスを確認する {#check-the-health-status-of-a-ticdc-cluster}

このAPIは同期インターフェースです。クラスターが健康であれば、`200 OK` が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/health`

### 例 {#example}

```shell
curl -X GET http://127.0.0.1:8300/api/v2/health
```

クラスターが健康である場合、応答は `200 OK` と空のJSONオブジェクトです。

```json
{}
```

クラスターが健全でない場合、応答はエラーメッセージを含むJSONオブジェクトです。

## レプリケーションタスクの作成 {#create-a-replication-task}

このインターフェースは、レプリケーションタスクをTiCDCに送信するために使用されます。リクエストが成功した場合、`200 OK`が返されます。返された結果は、サーバーがコマンドを実行することに同意することを意味しますが、コマンドが成功裏に実行されることを保証するものではありません。

### リクエストURI {#request-uri}

`POST /api/v2/changefeeds`

### パラメータの説明 {#parameter-descriptions}

```json
{
  "changefeed_id": "string",
  "replica_config": {
    "bdr_mode": true,
    "case_sensitive": false,
    "check_gc_safe_point": true,
    "consistent": {
      "flush_interval": 0,
      "level": "string",
      "max_log_size": 0,
      "storage": "string"
    },
    "enable_old_value": true,
    "enable_sync_point": true,
    "filter": {
      "do_dbs": [
        "string"
      ],
      "do_tables": [
        {
          "database_name": "string",
          "table_name": "string"
        }
      ],
      "event_filters": [
        {
          "ignore_delete_value_expr": "string",
          "ignore_event": [
            "string"
          ],
          "ignore_insert_value_expr": "string",
          "ignore_sql": [
            "string"
          ],
          "ignore_update_new_value_expr": "string",
          "ignore_update_old_value_expr": "string",
          "matcher": [
            "string"
          ]
        }
      ],
      "ignore_dbs": [
        "string"
      ],
      "ignore_tables": [
        {
          "database_name": "string",
          "table_name": "string"
        }
      ],
      "ignore_txn_start_ts": [
        0
      ],
      "rules": [
        "string"
      ]
    },
    "force_replicate": true,
    "ignore_ineligible_table": true,
    "memory_quota": 0,
    "mounter": {
      "worker_num": 0
    },
    "sink": {
      "column_selectors": [
        {
          "columns": [
            "string"
          ],
          "matcher": [
            "string"
          ]
        }
      ],
      "csv": {
        "delimiter": "string",
        "include_commit_ts": true,
        "null": "string",
        "quote": "string"
      },
      "date_separator": "string",
      "dispatchers": [
        {
          "matcher": [
            "string"
          ],
          "partition": "string",
          "topic": "string"
        }
      ],
      "enable_partition_separator": true,
      "encoder_concurrency": 0,
      "protocol": "string",
      "schema_registry": "string",
      "terminator": "string",
      "transaction_atomicity": "string"
    },
    "sync_point_interval": "string",
    "sync_point_retention": "string"
  },
  "sink_uri": "string",
  "start_ts": 0,
  "target_ts": 0
}
```

以下にパラメータが記載されています：

| パラメータ名           | 説明                                                                                                             |
| :--------------- | :------------------------------------------------------------------------------------------------------------- |
| `changefeed_id`  | `STRING` タイプ。レプリケーションタスクのIDです。（オプション）                                                                          |
| `replica_config` | レプリケーションタスクの構成パラメータです。（オプション）                                                                                  |
| **`sink_uri`**   | `STRING` タイプ。レプリケーションタスクの下流アドレスです。（**必須**）                                                                     |
| `start_ts`       | `UINT64` タイプ。changefeedの開始TSOを指定します。TiCDCクラスターはこのTSOからデータを取得し始めます。デフォルト値は現在時刻です。（オプション）                        |
| `target_ts`      | `UINT64` タイプ。changefeedのターゲットTSOを指定します。TiCDCクラスターはこのTSOに到達するとデータの取得を停止します。デフォルト値は空であり、TiCDCは自動的に停止しません。（オプション） |

`changefeed_id`、`start_ts`、`target_ts`、`sink_uri`の意味とフォーマットは、[cdc cliを使用してレプリケーションタスクを作成する](/ticdc/ticdc-manage-changefeed.md#create-a-replication-task)ドキュメントで説明されているものと同じです。これらのパラメータの詳細な説明については、そのドキュメントを参照してください。`sink_uri`で証明書パスを指定する場合は、対応するTiCDCサーバーに対応する証明書をアップロードしていることを確認してください。

`replica_config`パラメータの説明は以下の通りです。

| パラメータ名                    | 説明                                                                                                                                     |
| :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------- |
| `bdr_mode`                | `BOOLEAN` タイプ。[双方向レプリケーション](/ticdc/ticdc-bidirectional-replication.md)を有効にするかどうかを決定します。デフォルト値は `false` です。（オプション）                      |
| `case_sensitive`          | `BOOLEAN` タイプ。テーブル名のフィルタリング時に大文字と小文字を区別するかどうかを決定します。v6.5.6およびv7.1.3から、デフォルト値は `true` から `false` に変更されます。（オプション）                        |
| `check_gc_safe_point`     | `BOOLEAN` タイプ。レプリケーションタスクの開始時刻がGC時刻よりも前かどうかをチェックするかどうかを決定します。デフォルト値は `true` です。（オプション）                                                 |
| `consistent`              | redoログの構成パラメータです。（オプション）                                                                                                               |
| `enable_old_value`        | `BOOLEAN` タイプ。古い値（つまり、更新前の値）を出力するかどうかを決定します。デフォルト値は `true` です。（オプション）                                                                  |
| `enable_sync_point`       | `BOOLEAN` タイプ。`sync point` を有効にするかどうかを決定します。（オプション）                                                                                    |
| `filter`                  | `filter`の構成パラメータです。（オプション）                                                                                                             |
| `force_replicate`         | `BOOLEAN` タイプ。デフォルト値は `false` です。`true` に設定すると、レプリケーションタスクは一意なインデックスを持たないテーブルを強制的にレプリケートします。（オプション）                                    |
| `ignore_ineligible_table` | `BOOLEAN` タイプ。デフォルト値は `false` です。`true` に設定すると、レプリケーションタスクはレプリケートできないテーブルを無視します。（オプション）                                                |
| `memory_quota`            | `UINT64` タイプ。レプリケーションタスクのメモリクォータです。（オプション）                                                                                             |
| `mounter`                 | `mounter`の構成パラメータです。（オプション）                                                                                                            |
| `sink`                    | `sink`の構成パラメータです。（オプション）                                                                                                               |
| `sync_point_interval`     | `STRING` タイプ。`sync point` 機能が有効になっている場合、このパラメータは上流と下流のスナップショットを同期する間隔を指定します。デフォルト値は `10m` で、最小値は `30s` です。（オプション）                      |
| `sync_point_retention`    | `STRING` タイプ。`sync point` 機能が有効になっている場合、このパラメータはSyncpointでデータが下流テーブルに保持される期間を指定します。この期間を超過すると、データがクリーンアップされます。デフォルト値は `24h` です。（オプション） |

`consistent`パラメータは以下の通りです：

| パラメータ名                | 説明                                                                                                |
| :-------------------- | :------------------------------------------------------------------------------------------------ |
| `flush_interval`      | `UINT64` タイプ。redoログファイルをフラッシュする間隔です。（オプション）                                                       |
| `level`               | `STRING` タイプ。レプリケートされたデータの一貫性レベルです。（オプション）                                                        |
| `max_log_size`        | `UINT64` タイプ。redoログの最大値です。（オプション）                                                                 |
| `storage`             | `STRING` タイプ。ストレージの宛先アドレスです。（オプション）                                                               |
| `use_file_backend`    | `BOOL` タイプ。redoログをローカルファイルに保存するかどうかを指定します。（オプション）                                                 |
| `encoding_worker_num` | `INT` タイプ。redoモジュールのエンコードおよびデコードワーカーの数です。（オプション）                                                  |
| `flush_worker_num`    | `INT` タイプ。redoモジュールのフラッシュワーカーの数です。（オプション）                                                         |
| `compression`         | `STRING` タイプ。redoログファイルを圧縮する動作です。利用可能なオプションは `""` および `"lz4"` です。デフォルト値は `""` で、圧縮は行われません。（オプション） |
| `flush_concurrency`   | `INT` タイプ。単一ファイルのアップロードの並行性です。デフォルト値は `1` で、並行性は無効です。（オプション）                                      |

`filter`パラメータは以下の通りです：

| パラメータ名                | 説明                                                                                                                          |
| :-------------------- | :-------------------------------------------------------------------------------------------------------------------------- |
| `do_dbs`              | `STRING ARRAY` タイプ。レプリケートするデータベースです。（オプション）                                                                                 |
| `do_tables`           | レプリケートするテーブルです。（オプション）                                                                                                      |
| `ignore_dbs`          | `STRING ARRAY` タイプ。無視するデータベースです。（オプション）                                                                                     |
| `ignore_tables`       | 無視するテーブルです。（オプション）                                                                                                          |
| `event_filters`       | イベントをフィルタリングする構成です。（オプション）                                                                                                  |
| `ignore_txn_start_ts` | `UINT64 ARRAY` タイプ。これを指定すると、`start_ts` を指定するトランザクションが無視されます。例：`[1, 2]`。（オプション）                                              |
| `rules`               | `STRING ARRAY` タイプ。テーブルスキーマのフィルタリングルールです。例：`['foo*.*', 'bar*.*']`。詳細については、[Table Filter](/table-filter.md)を参照してください。（オプション） |

`filter.event_filters`パラメータは以下の通りです。詳細については、[Changefeed Log Filters](/ticdc/ticdc-filter.md)を参照してください。

| パラメータ名                         | 説明                                                                                                                    |
| :----------------------------- | :-------------------------------------------------------------------------------------------------------------------- |
| `ignore_delete_value_expr`     | `STRING ARRAY` タイプ。例：`"name = 'john'"` は、`name = 'john'` 条件を含むDELETE DMLステートメントをフィルタリングします。（オプション）                    |
| `ignore_event`                 | `STRING ARRAY` タイプ。例：`["insert"]` は、INSERTイベントをフィルタリングします。（オプション）                                                     |
| `ignore_insert_value_expr`     | `STRING ARRAY` タイプ。例：`"id >= 100"` は、`id >= 100` 条件に一致するINSERT DMLステートメントをフィルタリングします。（オプション）                          |
| `ignore_sql`                   | `STRING ARRAY` タイプ。例：`["^drop", "add column"]` は、`DROP` で始まるまたは `ADD COLUMN` を含むDDLステートメントをフィルタリングします。（オプション）         |
| `ignore_update_new_value_expr` | `STRING ARRAY` タイプ。例：`"gender = 'male'"` は、新しい値が `gender = 'male'` であるUPDATE DMLステートメントをフィルタリングします。（オプション）            |
| `ignore_update_old_value_expr` | `STRING ARRAY` タイプ。例：`"age < 18"` は、古い値が `age < 18` であるUPDATE DMLステートメントをフィルタリングします。（オプション）                           |
| `matcher`                      | `STRING ARRAY` タイプ。許可リストとして機能します。例：`["test.worker"]` は、フィルタールールが `test` データベースの `worker` テーブルにのみ適用されることを意味します。（オプション） |

`mounter`パラメータは以下の通りです：

| パラメータ名       | 説明                                                                                     |
| :----------- | :------------------------------------------------------------------------------------- |
| `worker_num` | `INT` タイプ。Mounterスレッドの数です。MounterはTiKVからのデータ出力をデコードするために使用されます。デフォルト値は `16` です。（オプション） |

`sink`パラメータは以下の通りです：

| Parameter name                | Description                                                                                                                         |
| :---------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| `column_selectors`            | カラムセレクターの構成。 (オプション)                                                                                                                |
| `csv`                         | CSVの構成。 (オプション)                                                                                                                     |
| `date_separator`              | `STRING` タイプ。ファイルディレクトリの日付区切りタイプを示します。値のオプションは `none`, `year`, `month`, および `day` です。 `none` はデフォルト値で、日付が分割されていないことを意味します。 (オプション)  |
| `dispatchers`                 | イベントディスパッチングの構成配列。 (オプション)                                                                                                          |
| `encoder_concurrency`         | `INT` タイプ。MQシンク内のエンコーダースレッドの数。デフォルト値は `16` です。 (オプション)                                                                              |
| `protocol`                    | `STRING` タイプ。MQシンクの場合、メッセージのプロトコル形式を指定できます。現在サポートされているプロトコルは次のとおりです: `canal-json`, `open-protocol`, `canal`, `avro`, および `maxwell`。 |
| `schema_registry`             | `STRING` タイプ。スキーマレジストリのアドレス。 (オプション)                                                                                                |
| `terminator`                  | `STRING` タイプ。ターミネーターは、2つのデータ変更イベントを分離するために使用されます。デフォルト値はnullで、`"\r\n"` がターミネーターとして使用されます。 (オプション)                                   |
| `transaction_atomicity`       | `STRING` タイプ。トランザクションのアトミックレベル。 (オプション)                                                                                             |
| `only_output_updated_columns` | `BOOLEAN` タイプ。`canal-json` または `open-protocol` プロトコルを使用するMQシンクの場合、変更されたカラムのみを出力するかどうかを指定できます。デフォルト値は `false` です。 (オプション)            |
| `cloud_storage_config`        | ストレージシンクの構成。 (オプション)                                                                                                                |

`sink.column_selectors` は配列です。パラメータは次のように記述されます:

| Parameter name | Description                                           |
| :------------- | :---------------------------------------------------- |
| `columns`      | `STRING ARRAY` タイプ。カラム配列。                             |
| `matcher`      | `STRING ARRAY` タイプ。マッチャー構成。フィルタールールと同じマッチング構文を持っています。 |

`sink.csv` パラメータは次のように記述されます:

| Parameter name           | Description                                                                         |
| :----------------------- | :---------------------------------------------------------------------------------- |
| `delimiter`              | `STRING` タイプ。CSVファイル内のフィールドを区切るために使用される文字。値はASCII文字でなければならず、デフォルト値は `,` です。         |
| `include_commit_ts`      | `BOOLEAN` タイプ。CSV行にcommit-tsを含めるかどうか。デフォルト値は `false` です。                            |
| `null`                   | `STRING` タイプ。CSVカラムがnullの場合に表示される文字。デフォルト値は `\N` です。                                |
| `quote`                  | `STRING` タイプ。CSVファイル内のフィールドを囲むために使用される引用符。値が空の場合、引用符は使用されません。デフォルト値は `"` です。        |
| `binary_encoding_method` | `STRING` タイプ。バイナリデータのエンコーディング方法。`"base64"` または `"hex"` になります。デフォルト値は `"base64"` です。 |

`sink.dispatchers`: MQタイプのシンクの場合、このパラメータを使用してイベントディスパッチャーを構成できます。次のディスパッチャーがサポートされています: `default`, `ts`, `rowid`, および `table`。ディスパッチャールールは次のとおりです:

- `default`: when multiple unique indexes (including the primary key) exist, events are dispatched in the table mode. When only one unique index (or the primary key) exists, events are dispatched in the rowid mode. If the Old Value feature is enabled, events are dispatched in the table mode.
- `ts`: uses the commitTs of the row change to create the hash value and dispatch events.
- `rowid`: uses the name and value of the selected HandleKey column to create the hash value and dispatch events.
- `table`: uses the schema name of the table and the table name to create the hash value and dispatch events.

`sink.dispatchers` is an array. The parameters are described as follows:

| Parameter name | Description                                                                   |
| :------------- | :---------------------------------------------------------------------------- |
| `matcher`      | `STRING ARRAY` type. It has the same matching syntax as the filter rule does. |
| `partition`    | `STRING` type. The target partition for dispatching events.                   |
| `topic`        | `STRING` type. The target topic for dispatching events.                       |

`sink.cloud_storage_config`  parameters are described as follows:

| Parameter name           | Description                                                                                                                                                                                                     |
| :----------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `worker_count`           | `INT` type. The concurrency for saving data changes to the downstream cloud storage.                                                                                                                            |
| `flush_interval`         | `STRING` type. The interval for saving data changes to the downstream cloud storage.                                                                                                                            |
| `file_size`              | `INT` type. A data change file is saved to the cloud storage when the number of bytes in this file exceeds the value of this parameter.                                                                         |
| `file_expiration_days`   | `INT` type. The duration to retain files, which takes effect only when `date-separator` is configured as `day`.                                                                                                 |
| `file_cleanup_cron_spec` | `STRING` type. The running cycle of the scheduled cleanup task, compatible with the crontab configuration, with a format of `<Second> <Minute> <Hour> <Day of the month> <Month> <Day of the week (Optional)>`. |
| `flush_concurrency`      | `INT` type. The concurrency for uploading a single file.                                                                                                                                                        |

### 例 {#example}

次のリクエストは、IDが`test5`で、`sink_uri`が`blackhome://`のレプリケーションタスクを作成します。

```shell
curl -X POST -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v2/changefeeds -d '{"changefeed_id":"test5","sink_uri":"blackhole://"}'
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

### Response body format {#response-body-format}

```json
{
  "admin_job_type": 0,
  "checkpoint_time": "string",
  "checkpoint_ts": 0,
  "config": {
    "bdr_mode": true,
    "case_sensitive": false,
    "check_gc_safe_point": true,
    "consistent": {
      "flush_interval": 0,
      "level": "string",
      "max_log_size": 0,
      "storage": "string"
    },
    "enable_old_value": true,
    "enable_sync_point": true,
    "filter": {
      "do_dbs": [
        "string"
      ],
      "do_tables": [
        {
          "database_name": "string",
          "table_name": "string"
        }
      ],
      "event_filters": [
        {
          "ignore_delete_value_expr": "string",
          "ignore_event": [
            "string"
          ],
          "ignore_insert_value_expr": "string",
          "ignore_sql": [
            "string"
          ],
          "ignore_update_new_value_expr": "string",
          "ignore_update_old_value_expr": "string",
          "matcher": [
            "string"
          ]
        }
      ],
      "ignore_dbs": [
        "string"
      ],
      "ignore_tables": [
        {
          "database_name": "string",
          "table_name": "string"
        }
      ],
      "ignore_txn_start_ts": [
        0
      ],
      "rules": [
        "string"
      ]
    },
    "force_replicate": true,
    "ignore_ineligible_table": true,
    "memory_quota": 0,
    "mounter": {
      "worker_num": 0
    },
    "sink": {
      "column_selectors": [
        {
          "columns": [
            "string"
          ],
          "matcher": [
            "string"
          ]
        }
      ],
      "csv": {
        "delimiter": "string",
        "include_commit_ts": true,
        "null": "string",
        "quote": "string"
      },
      "date_separator": "string",
      "dispatchers": [
        {
          "matcher": [
            "string"
          ],
          "partition": "string",
          "topic": "string"
        }
      ],
      "enable_partition_separator": true,
      "encoder_concurrency": 0,
      "protocol": "string",
      "schema_registry": "string",
      "terminator": "string",
      "transaction_atomicity": "string"
    },
    "sync_point_interval": "string",
    "sync_point_retention": "string"
  },
  "create_time": "string",
  "creator_version": "string",
  "error": {
    "addr": "string",
    "code": "string",
    "message": "string"
  },
  "id": "string",
  "resolved_ts": 0,
  "sink_uri": "string",
  "start_ts": 0,
  "state": "string",
  "target_ts": 0,
  "task_status": [
    {
      "capture_id": "string",
      "table_ids": [
        0
      ]
    }
  ]
}
```

以下にパラメータが記載されています：

| パラメータ名            | 説明                                                                                      |
| :---------------- | :-------------------------------------------------------------------------------------- |
| `admin_job_type`  | `INTEGER` タイプ。管理ジョブのタイプ。                                                                |
| `checkpoint_time` | `STRING` タイプ。レプリケーションタスクの現在のチェックポイントのフォーマットされた時間。                                       |
| `checkpoint_ts`   | `STRING` タイプ。レプリケーションタスクの現在のチェックポイントの TSO。                                              |
| `config`          | レプリケーションタスクの構成。構造と意味は、レプリケーションタスクの作成時の `replica_config` 構成と同じです。                        |
| `create_time`     | `STRING` タイプ。レプリケーションタスクが作成された時間。                                                       |
| `creator_version` | `STRING` タイプ。レプリケーションタスクが作成された TiCDC バージョン。                                             |
| `error`           | レプリケーションタスクのエラー。                                                                        |
| `id`              | `STRING` タイプ。レプリケーションタスク ID。                                                            |
| `resolved_ts`     | `UINT64` タイプ。レプリケーションタスクの resolved ts。                                                  |
| `sink_uri`        | `STRING` タイプ。レプリケーションタスクの sink URI。                                                     |
| `start_ts`        | `UINT64` タイプ。レプリケーションタスクの start ts。                                                     |
| `state`           | `STRING` タイプ。レプリケーションタスクのステータス。`normal`、`stopped`、`error`、`failed`、または`finished` になります。 |
| `target_ts`       | `UINT64` タイプ。レプリケーションタスクの target ts。                                                    |
| `task_status`     | レプリケーションタスクのディスパッチの詳細なステータス。                                                            |

`task_status` パラメータは以下のように記載されています：

| パラメータ名       | 説明                                              |
| :----------- | :---------------------------------------------- |
| `capture_id` | `STRING` タイプ。キャプチャ ID。                          |
| `table_ids`  | `UINT64 ARRAY` タイプ。このキャプチャでレプリケートされているテーブルの ID。 |

`error` パラメータは以下のように記載されています：

| パラメータ名    | 説明                       |
| :-------- | :----------------------- |
| `addr`    | `STRING` タイプ。キャプチャのアドレス。 |
| `code`    | `STRING` タイプ。エラーコード。     |
| `message` | `STRING` タイプ。エラーの詳細。     |

## レプリケーションタスクの削除 {#remove-a-replication-task}

この API は、レプリケーションタスクを削除するための冪等なインターフェースです（つまり、初期の適用を超えて結果を変更することなく複数回適用できます）。リクエストが成功した場合、`200 OK` が返されます。返された結果は、サーバーがコマンドを実行することに同意するだけであり、コマンドが成功裏に実行されることを保証するものではありません。

### リクエスト URI {#request-uri}

`DELETE /api/v2/changefeeds/{changefeed_id}`

### パラメータの説明 {#parameter-descriptions}

#### パスパラメータ {#path-parameters}

| パラメータ名          | 説明                               |
| :-------------- | :------------------------------- |
| `changefeed_id` | 削除するレプリケーションタスク（changefeed）の ID。 |

### 例 {#example}

以下のリクエストは、ID が `test1` のレプリケーションタスクを削除します。

```shell
curl -X DELETE http://127.0.0.1:8300/api/v2/changefeeds/test1
```

リクエストが成功した場合、`200 OK` が返されます。 リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーション構成の更新 {#update-the-replication-configuration}

このAPIは、レプリケーションタスクを更新するために使用されます。 リクエストが成功した場合、`200 OK` が返されます。 返された結果は、サーバーがコマンドを実行することに同意することを意味しますが、コマンドが成功裏に実行されることを保証するものではありません。

Changefeed構成を変更するには、`レプリケーションタスクを一時停止 -> 構成を変更 -> レプリケーションタスクを再開` の手順に従います。

### リクエストURI {#request-uri}

`PUT /api/v2/changefeeds/{changefeed_id}`

### パラメータの説明 {#parameter-descriptions}

#### パスパラメータ {#path-parameters}

| パラメータ名          | 説明                              |
| :-------------- | :------------------------------ |
| `changefeed_id` | 更新するレプリケーションタスク（changefeed）のID。 |

#### リクエストボディのパラメータ {#parameters-for-the-request-body}

```json
{
  "replica_config": {
    "bdr_mode": true,
    "case_sensitive": false,
    "check_gc_safe_point": true,
    "consistent": {
      "flush_interval": 0,
      "level": "string",
      "max_log_size": 0,
      "storage": "string"
    },
    "enable_old_value": true,
    "enable_sync_point": true,
    "filter": {
      "do_dbs": [
        "string"
      ],
      "do_tables": [
        {
          "database_name": "string",
          "table_name": "string"
        }
      ],
      "event_filters": [
        {
          "ignore_delete_value_expr": "string",
          "ignore_event": [
            "string"
          ],
          "ignore_insert_value_expr": "string",
          "ignore_sql": [
            "string"
          ],
          "ignore_update_new_value_expr": "string",
          "ignore_update_old_value_expr": "string",
          "matcher": [
            "string"
          ]
        }
      ],
      "ignore_dbs": [
        "string"
      ],
      "ignore_tables": [
        {
          "database_name": "string",
          "table_name": "string"
        }
      ],
      "ignore_txn_start_ts": [
        0
      ],
      "rules": [
        "string"
      ]
    },
    "force_replicate": true,
    "ignore_ineligible_table": true,
    "memory_quota": 0,
    "mounter": {
      "worker_num": 0
    },
    "sink": {
      "column_selectors": [
        {
          "columns": [
            "string"
          ],
          "matcher": [
            "string"
          ]
        }
      ],
      "csv": {
        "delimiter": "string",
        "include_commit_ts": true,
        "null": "string",
        "quote": "string"
      },
      "date_separator": "string",
      "dispatchers": [
        {
          "matcher": [
            "string"
          ],
          "partition": "string",
          "topic": "string"
        }
      ],
      "enable_partition_separator": true,
      "encoder_concurrency": 0,
      "protocol": "string",
      "schema_registry": "string",
      "terminator": "string",
      "transaction_atomicity": "string"
    },
    "sync_point_interval": "string",
    "sync_point_retention": "string"
  },
  "sink_uri": "string",
  "target_ts": 0
}
```

現在、APIを介して変更できるのは次の構成のみです。

| パラメータ名           | 説明                                             |
| :--------------- | :--------------------------------------------- |
| `target_ts`      | `UINT64` タイプ。changefeedのターゲットTSOを指定します。(オプション) |
| `sink_uri`       | `STRING` タイプ。レプリケーションタスクのダウンストリームアドレス。(オプション)  |
| `replica_config` | sinkの構成パラメータ。完全でなければなりません。(オプション)              |

上記のパラメータの意味は、[レプリケーションタスクの作成](#create-a-replication-task)セクションと同じです。詳細については、そのセクションを参照してください。

### 例 {#example}

以下のリクエストは、IDが`test1`のレプリケーションタスクの`target_ts`を`32`に更新します。

```shell
 curl -X PUT -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v2/changefeeds/test1 -d '{"target_ts":32}'
```

リクエストが成功した場合、 `200 OK` が返されます。 リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。 JSON応答本体の意味は、[レプリケーションタスクの作成](#create-a-replication-task) セクションと同じです。 詳細は、そのセクションを参照してください。

## レプリケーションタスクリストのクエリ {#query-the-replication-task-list}

このAPIは同期インターフェースです。 リクエストが成功した場合、TiCDCクラスターのすべてのレプリケーションタスク（changefeed）の基本情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/changefeeds`

### パラメータの説明 {#parameter-descriptions}

#### クエリパラメータ {#query-parameter}

| パラメータ名  | 説明                                                      |
| :------ | :------------------------------------------------------ |
| `state` | このパラメータが指定されている場合、指定された状態のレプリケーションタスクの情報が返されます。 (オプション) |

`state` の値オプションは、 `all`, `normal`, `stopped`, `error`, `failed`, および `finished` です。

このパラメータが指定されていない場合、 `normal`, `stopped`, または `failed` 状態のレプリケーションタスクの基本情報がデフォルトで返されます。

### 例 {#example}

次のリクエストは、 `normal` 状態のすべてのレプリケーションタスクの基本情報をクエリします。

```shell
curl -X GET http://127.0.0.1:8300/api/v2/changefeeds?state=normal
```

```json
{
  "total": 2,
  "items": [
    {
      "id": "test",
      "state": "normal",
      "checkpoint_tso": 439749918821711874,
      "checkpoint_time": "2023-02-27 23:46:52.888",
      "error": null
    },
    {
      "id": "test2",
      "state": "normal",
      "checkpoint_tso": 439749918821711874,
      "checkpoint_time": "2023-02-27 23:46:52.888",
      "error": null
    }
  ]
}
```

上記の返された結果のパラメータは次のように説明されています。

- `id`: レプリケーションタスクのID。
- `state`: レプリケーションタスクの現在の[state](/ticdc/ticdc-changefeed-overview.md#changefeed-state-transfer)。
- `checkpoint_tso`: レプリケーションタスクの現在のチェックポイントのTSO。
- `checkpoint_time`: レプリケーションタスクの現在のチェックポイントのフォーマットされた時間。
- `error`: レプリケーションタスクのエラー情報。

## 特定のレプリケーションタスクをクエリする {#query-a-specific-replication-task}

このAPIは同期インターフェースです。リクエストが成功した場合、指定されたレプリケーションタスク（changefeed）の詳細情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/changefeeds/{changefeed_id}`

### パラメータの説明 {#parameter-description}

#### パスパラメータ {#path-parameter}

| パラメータ名          | 説明                               |
| :-------------- | :------------------------------- |
| `changefeed_id` | クエリするレプリケーションタスク（changefeed）のID。 |

### 例 {#example}

以下のリクエストはIDが`test1`のレプリケーションタスクの詳細情報をクエリします。

```shell
curl -X GET http://127.0.0.1:8300/api/v2/changefeeds/test1
```

JSONレスポンスボディの意味は、[レプリケーションタスクの作成](#create-a-replication-task)セクションと同じです。詳細は、そのセクションを参照してください。

## レプリケーションタスクを一時停止する {#pause-a-replication-task}

このAPIは、レプリケーションタスクを一時停止します。リクエストが成功した場合、`200 OK`が返されます。返された結果は、サーバーがコマンドを実行することに同意することを意味しますが、コマンドが成功裏に実行されることを保証するものではありません。

### リクエストURI {#request-uri}

`POST /api/v2/changefeeds/{changefeed_id}/pause`

### パラメータの説明 {#parameter-description}

#### パスパラメータ {#path-parameter}

| パラメータ名          | 説明                                |
| :-------------- | :-------------------------------- |
| `changefeed_id` | 一時停止するレプリケーションタスク（changefeed）のID。 |

### 例 {#example}

以下のリクエストは、IDが`test1`のレプリケーションタスクを一時停止します。

```shell
curl -X POST http://127.0.0.1:8300/api/v2/changefeeds/test1/pause
```

リクエストが成功した場合、`200 OK` が返されます。 リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーションタスクを再開する {#resume-a-replication-task}

このAPIはレプリケーションタスクを再開します。 リクエストが成功した場合、`200 OK` が返されます。 返された結果は、サーバーがコマンドを実行することに同意することを意味しますが、コマンドが成功裏に実行されることを保証するものではありません。

### リクエストURI {#request-uri}

`POST /api/v2/changefeeds/{changefeed_id}/resume`

### パラメータの説明 {#parameter-description}

#### パスパラメータ {#path-parameter}

| パラメータ名          | 説明                              |
| :-------------- | :------------------------------ |
| `changefeed_id` | 再開するレプリケーションタスク（changefeed）のID。 |

#### リクエストボディのパラメータ {#parameters-for-the-request-body}

```json
{
  "overwrite_checkpoint_ts": 0
}
```

| Parameter name            | Description                                                              |
| :------------------------ | :----------------------------------------------------------------------- |
| `overwrite_checkpoint_ts` | `UINT64` type. サーバーのレプリケーションタスク（changefeed）を再開する際に、チェックポイントTSOを再割り当てします。 |

### Example {#example}

The following request resumes the replication task with the ID `test1`.

```shell
curl -X POST http://127.0.0.1:8300/api/v2/changefeeds/test1/resume -d '{}'
```

もしリクエストが成功した場合、`200 OK` が返されます。 もしリクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーションサブタスクリストをクエリする {#query-the-replication-subtask-list}

このAPIは同期インターフェースです。 もしリクエストが成功した場合、すべてのレプリケーションサブタスク（`processor`）の基本情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/processors`

### 例 {#example}

```shell
curl -X GET http://127.0.0.1:8300/api/v2/processors
```

```json
{
  "total": 3,
  "items": [
    {
      "changefeed_id": "test2",
      "capture_id": "d2912e63-3349-447c-90ba-72a4e04b5e9e"
    },
    {
      "changefeed_id": "test1",
      "capture_id": "d2912e63-3349-447c-90ba-72a4e04b5e9e"
    },
    {
      "changefeed_id": "test",
      "capture_id": "d2912e63-3349-447c-90ba-72a4e04b5e9e"
    }
  ]
}
```

パラメータは以下のように記述されています：

- `changefeed_id`：changefeed ID。
- `capture_id`：capture ID。

## 特定のレプリケーションサブタスクをクエリする {#query-a-specific-replication-subtask}

このAPIは同期インターフェースです。リクエストが成功した場合、指定されたレプリケーションサブタスク（`processor`）の詳細情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/processors/{changefeed_id}/{capture_id}`

### パラメータの説明 {#parameter-descriptions}

#### パスパラメータ {#path-parameters}

| パラメータ名          | 説明                                |
| :-------------- | :-------------------------------- |
| `changefeed_id` | クエリするレプリケーションサブタスクのchangefeed ID。 |
| `capture_id`    | クエリするレプリケーションサブタスクのcapture ID。    |

### 例 {#example}

以下のリクエストは、`changefeed_id`が`test`で`capture_id`が`561c3784-77f0-4863-ad52-65a3436db6af`であるサブタスクの詳細情報をクエリします。サブタスクは`changefeed_id`と`capture_id`で識別できます。

```shell
curl -X GET http://127.0.0.1:8300/api/v2/processors/test/561c3784-77f0-4863-ad52-65a3436db6af
```

```json
{
  "table_ids": [
    80
  ]
}
```

パラメータは以下のように説明されています：

- `table_ids`: このキャプチャでレプリケートするテーブルID。

## TiCDCサービスプロセスリストのクエリ {#query-the-ticdc-service-process-list}

このAPIは同期インターフェースです。リクエストが成功した場合、すべてのレプリケーションプロセス（`capture`）の基本情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/captures`

### 例 {#example}

```shell
curl -X GET http://127.0.0.1:8300/api/v2/captures
```

```json
{
  "total": 1,
  "items": [
    {
      "id": "d2912e63-3349-447c-90ba-72a4e04b5e9e",
      "is_owner": true,
      "address": "127.0.0.1:8300"
    }
  ]
}
```

パラメータは以下のように記述されています：

- `id`: キャプチャID。
- `is_owner`: キャプチャが所有者かどうか。
- `address`: キャプチャのアドレス。

## オーナーノードの追放 {#evict-an-owner-node}

このAPIは非同期インターフェースです。リクエストが成功した場合、`200 OK`が返されます。返された結果は、サーバーがコマンドを実行することに同意することを意味しますが、コマンドが成功裏に実行されることを保証するものではありません。

### リクエストURI {#request-uri}

`POST /api/v2/owner/resign`

### 例 {#example}

以下のリクエストは、TiCDCの現在のオーナーノードを追放し、新しいオーナーノードを生成するための新しい選挙をトリガーします。

```shell
curl -X POST http://127.0.0.1:8300/api/v2/owner/resign
```

リクエストが成功した場合、 `200 OK` が返されます。 リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## TiCDCサーバーのログレベルを動的に調整する {#dynamically-adjust-the-log-level-of-the-ticdc-server}

このAPIは同期インターフェースです。 リクエストが成功した場合、 `200 OK` が返されます。

### リクエストURI {#request-uri}

`POST /api/v2/log`

### リクエストパラメータ {#request-parameter}

#### リクエスト本文のパラメータ {#parameter-for-the-request-body}

| パラメータ名      | 説明          |
| :---------- | :---------- |
| `log_level` | 設定したいログレベル。 |

`log_level` は [zapが提供するログレベル](https://godoc.org/go.uber.org/zap#UnmarshalText) をサポートしています: "debug", "info", "warn", "error", "dpanic", "panic", および "fatal".

### 例 {#example}

```shell
curl -X POST -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v2/log -d '{"log_level":"debug"}'
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。
