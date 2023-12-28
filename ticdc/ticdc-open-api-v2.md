---
title: TiCDC OpenAPI v2
summary: Learn how to use the OpenAPI v2 interface to manage the cluster status and data replication.
---

# TiCDC OpenAPI v2 {#ticdc-openapi-v2}

<!-- markdownlint-disable MD024 -->

TiCDCは、TiCDCクラスターのクエリと操作を行うためのOpenAPI機能を提供します。OpenAPI機能は、[`cdc cli`ツール](/ticdc/ticdc-manage-changefeed.md)のサブセットです。

> **Note:**
>
> TiCDC OpenAPI v1は将来的に削除されます。TiCDC OpenAPI v2を使用することをお勧めします。

APIを使用して、TiCDCクラスターで次のメンテナンス操作を実行できます：

- [TiCDCノードのステータス情報を取得する](#get-the-status-information-of-a-ticdc-node)
- [TiCDCクラスターのヘルスステータスをチェックする](#check-the-health-status-of-a-ticdc-cluster)
- [レプリケーションタスクを作成する](#create-a-replication-task)
- [レプリケーションタスクを削除する](#remove-a-replication-task)
- [レプリケーション構成を更新する](#update-the-replication-configuration)
- [レプリケーションタスクリストをクエリする](#query-the-replication-task-list)
- [特定のレプリケーションタスクをクエリする](#query-a-specific-replication-task)
- [レプリケーションタスクを一時停止する](#pause-a-replication-task)
- [レプリケーションタスクを再開する](#resume-a-replication-task)
- [レプリケーションサブタスクリストをクエリする](#query-the-replication-subtask-list)
- [特定のレプリケーションサブタスクをクエリする](#query-a-specific-replication-subtask)
- [TiCDCサービスプロセスリストをクエリする](#query-the-ticdc-service-process-list)
- [オーナーノードを追い出す](#evict-an-owner-node)
- [TiCDCサーバーのログレベルを動的に調整する](#dynamically-adjust-the-log-level-of-the-ticdc-server)

すべてのAPIのリクエストボディと返される値はJSON形式です。成功したリクエストは`200 OK`メッセージを返します。次のセクションでは、APIの具体的な使用方法について説明します。

以下の例では、TiCDCサーバーのリスニングIPアドレスは`127.0.0.1`で、ポートは`8300`です。TiCDCにバインドされたIPアドレスとポートは、TiCDCサーバーを起動する際に`--addr=ip:port`で指定できます。

## APIエラーメッセージテンプレート {#api-error-message-template}

APIリクエストが送信された後、エラーが発生した場合、返されるエラーメッセージは次の形式です：

```json
{
    "error_msg": "",
    "error_code": ""
}
```

上記のJSON出力では、 `error_msg`はエラーメッセージを、 `error_code`は対応するエラーコードを表します。

## APIリストインターフェースの戻り値形式 {#return-format-of-the-api-list-interface}

APIリクエストがリソースのリスト（例：すべての `Captures`のリスト）を返す場合、TiCDCの戻り値形式は次のとおりです：

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

- `total`：リソースの総数を示します。
- `items`：このリクエストで返されるすべてのリソースを含む配列です。配列のすべての要素は同じリソースです。

## TiCDCノードのステータス情報を取得する {#get-the-status-information-of-a-ticdc-node}

このAPIは同期インターフェースです。リクエストが成功した場合、対応するノードのステータス情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/status`

### 例 {#example}

以下のリクエストは、IPアドレスが`127.0.0.1`でポート番号が`8300`のTiCDCノードのステータス情報を取得します。

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

上記の出力のパラメータは以下のように説明されています：

- `version`：TiCDCの現在のバージョン番号。
- `git_hash`：Gitのハッシュ値。
- `id`：ノードのキャプチャID。
- `pid`：ノードのキャプチャプロセスID（PID）。
- `is_owner`：ノードがオーナーであるかどうかを示します。
- `liveness`：このノードがライブであるかどうか。 `0`は正常を意味します。 `1`は、ノードが `graceful shutdown`状態にあることを意味します。

## TiCDCクラスターの健康状態をチェックする {#check-the-health-status-of-a-ticdc-cluster}

このAPIは同期インターフェースです。クラスターが健全であれば、 `200 OK`が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/health`

### 例 {#example}

```shell
curl -X GET http://127.0.0.1:8300/api/v2/health
```

クラスターが正常であれば、`200 OK`と空のJSONオブジェクトが返されます。

```json
{}
```

クラスターが健全でない場合、エラーメッセージを含むJSONオブジェクトが返されます。

## レプリケーションタスクの作成 {#create-a-replication-task}

このインターフェースは、TiCDCにレプリケーションタスクを送信するために使用されます。リクエストが成功した場合、`200 OK`が返されます。返された結果は、サーバーがコマンドを実行することに同意したことを意味するだけであり、コマンドが正常に実行されることを保証するものではありません。

### リクエストURI {#request-uri}

`POST /api/v2/changefeeds`

### パラメーターの説明 {#parameter-descriptions}

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

パラメータは以下のように説明されています：

| パラメータ名           | 説明                                                                                                      |
| :--------------- | :------------------------------------------------------------------------------------------------------ |
| `changefeed_id`  | `STRING`型。レプリケーションタスクのIDです。（オプション）                                                                      |
| `replica_config` | レプリケーションタスクの設定パラメータです。（オプション）                                                                           |
| **`sink_uri`**   | `STRING`型。レプリケーションタスクの下流アドレスです。（**必須**）                                                                 |
| `start_ts`       | `UINT64`型。チェンジフィードの開始TSOを指定します。TiCDCクラスターはこのTSOからデータを取得し始めます。デフォルト値は現在時刻です。（オプション）                      |
| `target_ts`      | `UINT64`型。チェンジフィードのターゲットTSOを指定します。TiCDCクラスターはこのTSOに到達するとデータの取得を停止します。デフォルト値は空で、TiCDCは自動的に停止しません。（オプション） |

`changefeed_id`、`start_ts`、`target_ts`、`sink_uri`の意味とフォーマットは、[Use `cdc cli` to create a replication task](/ticdc/ticdc-manage-changefeed.md#create-a-replication-task)ドキュメントで説明されているものと同じです。これらのパラメータの詳細な説明については、そのドキュメントを参照してください。`sink_uri`で証明書パスを指定する場合は、対応するTiCDCサーバーに対応する証明書をアップロードしていることを確認してください。

`replica_config`パラメータの説明は以下のとおりです。

| パラメータ名                    | 説明                                                                                                                                                         |
| :------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `bdr_mode`                | `BOOLEAN`型。[双方向レプリケーション](/ticdc/ticdc-bidirectional-replication.md)を有効にするかどうかを決定します。デフォルト値は`false`です。（オプション）                                               |
| `case_sensitive`          | `BOOLEAN`型。テーブル名をフィルタリングする際に大文字と小文字を区別するかどうかを決定します。v6.5.6およびv7.1.3以降、デフォルト値は`true`から`false`に変更されます。（オプション）                                                 |
| `check_gc_safe_point`     | `BOOLEAN`型。レプリケーションタスクの開始時刻がGC時刻よりも前であることを確認するかどうかを決定します。デフォルト値は`true`です。（オプション）                                                                           |
| `consistent`              | リドログの設定パラメータです。（オプション）                                                                                                                                     |
| `enable_old_value`        | `BOOLEAN`型。古い値（つまり、更新前の値）を出力するかどうかを決定します。デフォルト値は`true`です。（オプション）                                                                                           |
| `enable_sync_point`       | `BOOLEAN`型。`sync point`を有効にするかどうかを決定します。（オプション）                                                                                                            |
| `filter`                  | `filter`の設定パラメータです。（オプション）                                                                                                                                 |
| `force_replicate`         | `BOOLEAN`型。デフォルト値は`false`です。`true`に設定すると、レプリケーションタスクは一意のインデックスを持たないテーブルを強制的にレプリケートします。（オプション）                                                              |
| `ignore_ineligible_table` | `BOOLEAN`型。デフォルト値は`false`です。`true`に設定すると、レプリケーションタスクはレプリケートできないテーブルを無視します。（オプション）                                                                          |
| `memory_quota`            | `UINT64`型。レプリケーションタスクのメモリクォータです。（オプション）                                                                                                                    |
| `mounter`                 | `mounter`の設定パラメータです。（オプション）                                                                                                                                |
| `sink`                    | `sink`の設定パラメータです。（オプション）                                                                                                                                   |
| `sync_point_interval`     | `STRING`型。返される値は、`UINT64`型のナノ秒単位の時間です。`sync point`機能が有効になっている場合、このパラメータはSyncpointが上流と下流のスナップショットを整列する間隔を指定します。デフォルト値は`10m`で、最小値は`30s`です。（オプション）            |
| `sync_point_retention`    | `STRING`型。返される値は、`UINT64`型のナノ秒単位の時間です。`sync point`機能が有効になっている場合、このパラメータはSyncpointが下流テーブルでデータを保持する期間を指定します。この期間を超えると、データはクリーンアップされます。デフォルト値は`24h`です。（オプション） |

`consistent`パラメータの説明は以下のとおりです：

| パラメーター名               | 説明                                                                                            |
| :-------------------- | :-------------------------------------------------------------------------------------------- |
| `flush_interval`      | `UINT64` 型。リドゥログファイルをフラッシュする間隔です。 (オプション)                                                     |
| `level`               | `STRING` 型。レプリケートされたデータの整合性レベルです。 (オプション)                                                     |
| `max_log_size`        | `UINT64` 型。リドゥログの最大値です。 (オプション)                                                               |
| `storage`             | `STRING` 型。ストレージの宛先アドレスです。 (オプション)                                                            |
| `use_file_backend`    | `BOOL` 型。リドゥログをローカルファイルに保存するかどうかを指定します。 (オプション)                                               |
| `encoding_worker_num` | `INT` 型。リドゥモジュールのエンコードとデコードを行うワーカーの数です。 (オプション)                                               |
| `flush_worker_num`    | `INT` 型。リドゥモジュールのフラッシュを行うワーカーの数です。 (オプション)                                                    |
| `compression`         | `STRING` 型。リドゥログファイルを圧縮する動作です。使用可能なオプションは `""` と `"lz4"` です。デフォルト値は `""` で、圧縮は行われません。 (オプション) |
| `flush_concurrency`   | `INT` 型。単一ファイルをアップロードする際の並行性です。デフォルト値は `1` で、並行性は無効になります。 (オプション)                             |

`filter` パラメーターは以下のように説明されます:

| パラメーター名               | 説明                                                                                                                             |
| :-------------------- | :----------------------------------------------------------------------------------------------------------------------------- |
| `do_dbs`              | `STRING ARRAY` 型。レプリケートするデータベースです。 (オプション)                                                                                     |
| `do_tables`           | レプリケートするテーブルです。 (オプション)                                                                                                        |
| `ignore_dbs`          | `STRING ARRAY` 型。無視するデータベースです。 (オプション)                                                                                         |
| `ignore_tables`       | 無視するテーブルです。 (オプション)                                                                                                            |
| `event_filters`       | イベントをフィルタリングするための設定です。 (オプション)                                                                                                 |
| `ignore_txn_start_ts` | `UINT64 ARRAY` 型。`start_ts` を指定するトランザクションを無視します。例: `[1, 2]`。 (オプション)                                                           |
| `rules`               | `STRING ARRAY` 型。テーブルスキーマをフィルタリングするためのルールです。例: `['foo*.*', 'bar*.*']`。詳細については [テーブルフィルター](/table-filter.md) を参照してください。 (オプション) |

`filter.event_filters` パラメーターは以下のように説明されます。詳細については [Changefeed ログフィルター](/ticdc/ticdc-filter.md) を参照してください。

| パラメーター名                        | 説明                                                                                                                         |
| :----------------------------- | :------------------------------------------------------------------------------------------------------------------------- |
| `ignore_delete_value_expr`     | `STRING ARRAY` 型。例えば、`"name = 'john'"` は `name = 'john'` の条件を含む DELETE DML ステートメントをフィルタリングします。 (オプション)                     |
| `ignore_event`                 | `STRING ARRAY` 型。例えば、`["insert"]` は INSERT イベントをフィルタリングします。 (オプション)                                                        |
| `ignore_insert_value_expr`     | `STRING ARRAY` 型。例えば、`"id >= 100"` は `id >= 100` の条件に一致する INSERT DML ステートメントをフィルタリングします。 (オプション)                           |
| `ignore_sql`                   | `STRING ARRAY` 型。例えば、`["^drop", "add column"]` は `DROP` で始まるまたは `ADD COLUMN` を含む DDL ステートメントをフィルタリングします。 (オプション)           |
| `ignore_update_new_value_expr` | `STRING ARRAY` 型。例えば、`"gender = 'male'"` は新しい値が `gender = 'male'` の UPDATE DML ステートメントをフィルタリングします。 (オプション)                 |
| `ignore_update_old_value_expr` | `STRING ARRAY` 型。例えば、`"age < 18"` は古い値が `age < 18` の UPDATE DML ステートメントをフィルタリングします。 (オプション)                                |
| `matcher`                      | `STRING ARRAY` 型。allowlist として機能します。例えば、`["test.worker"]` は `test` データベースの `worker` テーブルにのみフィルタールールが適用されることを意味します。 (オプション) |

`mounter` パラメーターは以下のように説明されます:

| パラメーター名      | 説明                                                                                          |
| :----------- | :------------------------------------------------------------------------------------------ |
| `worker_num` | `INT` 型。Mounter スレッドの数です。Mounter は TiKV から出力されたデータをデコードするために使用されます。デフォルト値は `16` です。 (オプション) |

`sink` パラメーターは以下のように説明されます:

| パラメータ名                        | 説明                                                                                                                    |
| :---------------------------- | :-------------------------------------------------------------------------------------------------------------------- |
| `column_selectors`            | カラムセレクターの設定です。 (オプション)                                                                                                |
| `csv`                         | CSVの設定です。 (オプション)                                                                                                     |
| `date_separator`              | `STRING`型。ファイルディレクトリの日付区切りタイプを示します。値のオプションは、`none`、`year`、`month`、`day`です。`none`はデフォルト値で、日付が区切られていないことを意味します。 (オプション) |
| `dispatchers`                 | イベントディスパッチングのための設定配列です。 (オプション)                                                                                       |
| `encoder_concurrency`         | `INT`型。MQシンクのエンコーダースレッドの数です。デフォルト値は`16`です。 (オプション)                                                                    |
| `protocol`                    | `STRING`型。MQシンクの場合、メッセージのプロトコル形式を指定できます。現在サポートされているプロトコルは、`canal-json`、`open-protocol`、`canal`、`avro`、`maxwell`です。    |
| `schema_registry`             | `STRING`型。スキーマレジストリのアドレスです。 (オプション)                                                                                   |
| `terminator`                  | `STRING`型。2つのデータ変更イベントを区切るために使用されます。デフォルト値はnullで、`"\r\n"`が区切り文字として使用されます。 (オプション)                                     |
| `transaction_atomicity`       | `STRING`型。トランザクションのアトミックレベルです。 (オプション)                                                                                |
| `only_output_updated_columns` | `BOOLEAN`型。`canal-json`または`open-protocol`プロトコルを使用するMQシンクの場合、変更されたカラムのみを出力するかどうかを指定できます。デフォルト値は`false`です。 (オプション)      |
| `cloud_storage_config`        | ストレージシンクの設定です。 (オプション)                                                                                                |

`sink.column_selectors`は配列です。パラメータは以下のように説明されます:

| パラメータ名    | 説明                                                   |
| :-------- | :--------------------------------------------------- |
| `columns` | `STRING ARRAY`型。カラムの配列です。                            |
| `matcher` | `STRING ARRAY`型。マッチャーの設定です。フィルタールールと同じマッチング構文を使用します。 |

`sink.csv`のパラメータは以下のように説明されます:

| パラメータ名                   | 説明                                                                              |
| :----------------------- | :------------------------------------------------------------------------------ |
| `delimiter`              | `STRING`型。CSVファイルのフィールドを区切るために使用される文字です。値はASCII文字である必要があり、デフォルト値は`,`です。         |
| `include_commit_ts`      | `BOOLEAN`型。CSV行にcommit-tsを含めるかどうかを指定します。デフォルト値は`false`です。                       |
| `null`                   | `STRING`型。CSVカラムがnullの場合に表示される文字です。デフォルト値は`\N`です。                               |
| `quote`                  | `STRING`型。CSVファイルのフィールドを囲むために使用される引用符です。値が空の場合、引用符は使用されません。デフォルト値は`"`です。        |
| `binary_encoding_method` | `STRING`型。バイナリデータのエンコーディング方法です。`"base64"`または`"hex"`を指定できます。デフォルト値は`"base64"`です。 |

`sink.dispatchers`: MQタイプのシンクの場合、イベントディスパッチャーを設定するためにこのパラメータを使用できます。次のディスパッチャーがサポートされています: `default`、`ts`、`rowid`、`table`。ディスパッチャールールは次のとおりです:

- `default`: 複数のユニークインデックス(主キーを含む)が存在する場合、イベントはテーブルモードでディスパッチされます。ユニークインデックス(または主キー)が1つしか存在しない場合、イベントはrowidモードでディスパッチされます。古い値機能が有効になっている場合、イベントはテーブルモードでディスパッチされます。
- `ts`: 行の変更のcommitTsを使用してハッシュ値を作成し、イベントをディスパッチします。
- `rowid`: 選択したHandleKeyカラムの名前と値を使用してハッシュ値を作成し、イベントをディスパッチします。
- `table`: テーブルのスキーマ名とテーブル名を使用してハッシュ値を作成し、イベントをディスパッチします。

`sink.dispatchers`は配列です。パラメータは以下のように説明されます:

| パラメータ名      | 説明                                        |
| :---------- | :---------------------------------------- |
| `matcher`   | `STRING ARRAY`型。フィルタールールと同じマッチング構文を使用します。 |
| `partition` | `STRING`型。イベントをディスパッチするターゲットパーティションです。    |
| `topic`     | `STRING`型。イベントをディスパッチするターゲットトピックです。       |

`sink.cloud_storage_config`のパラメータは以下のように説明されます:

| パラメーター名                  | 説明                                                                                                          |
| :----------------------- | :---------------------------------------------------------------------------------------------------------- |
| `worker_count`           | `INT` 型。データ変更を下流のクラウドストレージに保存するための並行性。                                                                      |
| `flush_interval`         | `STRING` 型。データ変更を下流のクラウドストレージに保存するための間隔。                                                                    |
| `file_size`              | `INT` 型。このパラメーターの値を超えるバイト数のデータ変更ファイルがある場合、そのファイルはクラウドストレージに保存されます。                                          |
| `file_expiration_days`   | `INT` 型。`date-separator` が `day` に設定されている場合にのみ有効になる、ファイルを保持する期間。                                            |
| `file_cleanup_cron_spec` | `STRING` 型。スケジュールされたクリーンアップタスクの実行サイクルで、crontab の構成と互換性があり、フォーマットは `<秒> <分> <時> <月の日> <月> <週の日 (オプション)>` です。 |
| `flush_concurrency`      | `INT` 型。単一のファイルをアップロードするための並行性。                                                                             |

### 例 {#example}

次のリクエストは、ID が `test5` で `sink_uri` が `blackhome://` のレプリケーションタスクを作成します。

```shell
curl -X POST -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v2/changefeeds -d '{"changefeed_id":"test5","sink_uri":"blackhole://"}'
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

### レスポンスボディのフォーマット {#response-body-format}

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

パラメータは以下のように説明されています：

| パラメータ名            | 説明                                                                                 |
| :---------------- | :--------------------------------------------------------------------------------- |
| `admin_job_type`  | `INTEGER` 型。管理ジョブのタイプ。                                                             |
| `checkpoint_time` | `STRING` 型。レプリケーションタスクの現在のチェックポイントのフォーマットされた時刻。                                    |
| `checkpoint_ts`   | `STRING` 型。レプリケーションタスクの現在のチェックポイントの TSO。                                           |
| `config`          | レプリケーションタスクの構成。作成時の `replica_config` 構成と同じ構造と意味を持つ。                                |
| `create_time`     | `STRING` 型。レプリケーションタスクが作成された時刻。                                                    |
| `creator_version` | `STRING` 型。レプリケーションタスクが作成された時の TiCDC のバージョン。                                       |
| `error`           | レプリケーションタスクのエラー。                                                                   |
| `id`              | `STRING` 型。レプリケーションタスクの ID。                                                        |
| `resolved_ts`     | `UINT64` 型。レプリケーションタスクの resolved ts。                                               |
| `sink_uri`        | `STRING` 型。レプリケーションタスクの sink URI。                                                  |
| `start_ts`        | `UINT64` 型。レプリケーションタスクの start ts。                                                  |
| `state`           | `STRING` 型。レプリケーションタスクのステータス。`normal`、`stopped`、`error`、`failed`、`finished` のいずれか。 |
| `target_ts`       | `UINT64` 型。レプリケーションタスクの target ts。                                                 |
| `task_status`     | レプリケーションタスクのディスパッチの詳細ステータス。                                                        |

`task_status` パラメータは以下のように説明されています：

| パラメータ名       | 説明                                               |
| :----------- | :----------------------------------------------- |
| `capture_id` | `STRING` 型。キャプチャー ID。                            |
| `table_ids`  | `UINT64 ARRAY` 型。このキャプチャーでレプリケーションされているテーブルの ID。 |

`error` パラメータは以下のように説明されています：

| パラメータ名    | 説明                      |
| :-------- | :---------------------- |
| `addr`    | `STRING` 型。キャプチャーのアドレス。 |
| `code`    | `STRING` 型。エラーコード。      |
| `message` | `STRING` 型。エラーの詳細。      |

## レプリケーションタスクの削除 {#remove-a-replication-task}

この API は、レプリケーションタスクを削除するための冪等性のあるインターフェースです（つまり、初期の適用以降に結果が変化しないように複数回適用できます）。リクエストが成功した場合、`200 OK` が返されます。返される結果は、サーバーがコマンドを実行することに同意することを意味するだけで、コマンドが正常に実行されることを保証するものではありません。

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

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーション構成の更新 {#update-the-replication-configuration}

このAPIは、レプリケーションタスクを更新するために使用されます。リクエストが成功した場合、`200 OK`が返されます。返される結果は、サーバーがコマンドを実行することに同意することを意味するだけであり、コマンドが正常に実行されることを保証するものではありません。

チェンジフィードの構成を変更するには、`レプリケーションタスクを一時停止 -> 構成を変更 -> レプリケーションタスクを再開`の手順に従います。

### リクエストURI {#request-uri}

`PUT /api/v2/changefeeds/{changefeed_id}`

### パラメータの説明 {#parameter-descriptions}

#### パスパラメータ {#path-parameters}

| パラメータ名          | 説明                            |
| :-------------- | :---------------------------- |
| `changefeed_id` | 更新するレプリケーションタスク（チェンジフィード）のID。 |

#### リクエストボディのパラメータ {#parameters-for-the-request-body}

// The input content end

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

現在、APIを介して変更できるのは、次の設定のみです。

| パラメーター名          | 説明                                        |
| :--------------- | :---------------------------------------- |
| `target_ts`      | `UINT64`型。チェンジフィードのターゲットTSOを指定します。(オプション) |
| `sink_uri`       | `STRING`型。レプリケーションタスクの下流アドレスです。(オプション)    |
| `replica_config` | シンクの設定パラメーターです。完全である必要があります。(オプション)       |

上記のパラメーターの意味は、[レプリケーションタスクの作成](#create-a-replication-task)セクションと同じです。詳細は、そのセクションを参照してください。

### 例 {#example}

次のリクエストは、IDが`test1`のレプリケーションタスクの`target_ts`を`32`に更新します。

```shell
 curl -X PUT -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v2/changefeeds/test1 -d '{"target_ts":32}'
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。JSONレスポンスボディの意味は、[レプリケーションタスクの作成](#create-a-replication-task)セクションと同じです。詳細については、そのセクションを参照してください。

## レプリケーションタスクリストのクエリ {#query-the-replication-task-list}

このAPIは同期インターフェースです。リクエストが成功した場合、TiCDCクラスターのすべてのレプリケーションタスク（changefeed）の基本情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/changefeeds`

### パラメータの説明 {#parameter-descriptions}

#### クエリパラメータ {#query-parameter}

| パラメータ名  | 説明                                                      |
| :------ | :------------------------------------------------------ |
| `state` | このパラメータが指定された場合、この指定された状態のレプリケーションタスクの情報が返されます。 （オプション） |

`state`の値オプションは、`all`、`normal`、`stopped`、`error`、`failed`、`finished`です。

このパラメータが指定されていない場合、デフォルトで`normal`、`stopped`、または`failed`の状態のレプリケーションタスクの基本情報が返されます。

### 例 {#example}

次のリクエストは、`normal`状態のすべてのレプリケーションタスクの基本情報をクエリします。

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

上記の返された結果のパラメータは以下のように説明されています：

- `id`：レプリケーションタスクのID。
- `state`：レプリケーションタスクの現在の[state](/ticdc/ticdc-changefeed-overview.md#changefeed-state-transfer)。
- `checkpoint_tso`：レプリケーションタスクの現在のチェックポイントのTSO。
- `checkpoint_time`：レプリケーションタスクの現在のチェックポイントのフォーマットされた時間。
- `error`：レプリケーションタスクのエラー情報。

## 特定のレプリケーションタスクをクエリする {#query-a-specific-replication-task}

このAPIは同期インターフェースです。リクエストが成功すると、指定されたレプリケーションタスク（changefeed）の詳細情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/changefeeds/{changefeed_id}`

### パラメータの説明 {#parameter-description}

#### パスパラメータ {#path-parameter}

| パラメータ名          | 説明                               |
| :-------------- | :------------------------------- |
| `changefeed_id` | クエリするレプリケーションタスク（changefeed）のID。 |

### 例 {#example}

以下のリクエストは、IDが`test1`のレプリケーションタスクの詳細情報をクエリします。

```shell
curl -X GET http://127.0.0.1:8300/api/v2/changefeeds/test1
```

JSONレスポンスボディの意味は、[レプリケーションタスクの作成](#create-a-replication-task)セクションと同じです。詳細については、そのセクションを参照してください。

## レプリケーションタスクの一時停止 {#pause-a-replication-task}

このAPIは、レプリケーションタスクを一時停止します。リクエストが成功した場合、`200 OK`が返されます。返される結果は、サーバーがコマンドを実行することに同意することを意味するだけであり、コマンドが正常に実行されることを保証するものではありません。

### リクエストURI {#request-uri}

`POST /api/v2/changefeeds/{changefeed_id}/pause`

### パラメータの説明 {#parameter-description}

#### パスパラメータ {#path-parameter}

| パラメータ名          | 説明                                |
| :-------------- | :-------------------------------- |
| `changefeed_id` | 一時停止するレプリケーションタスク（changefeed）のID。 |

### 例 {#example}

次のリクエストは、IDが`test1`のレプリケーションタスクを一時停止します。

```shell
curl -X POST http://127.0.0.1:8300/api/v2/changefeeds/test1/pause
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーションタスクの再開 {#resume-a-replication-task}

このAPIはレプリケーションタスクを再開します。リクエストが成功した場合、`200 OK`が返されます。返される結果は、サーバーがコマンドを実行することに同意したことを意味するだけであり、コマンドが正常に実行されることを保証するものではありません。

### リクエストURI {#request-uri}

`POST /api/v2/changefeeds/{changefeed_id}/resume`

### パラメータの説明 {#parameter-description}

#### パスパラメータ {#path-parameter}

| パラメータ名          | 説明                              |
| :-------------- | :------------------------------ |
| `changefeed_id` | 再開するレプリケーションタスク（changefeed）のID。 |

#### リクエストボディのパラメータ {#parameters-for-the-request-body}

| パラメータ名 | 説明 |
| :----- | :- |
| なし     |    |

```json
{
  "overwrite_checkpoint_ts": 0
}
```

| パラメーター名                   | 説明                                                            |
| :------------------------ | :------------------------------------------------------------ |
| `overwrite_checkpoint_ts` | `UINT64`型。レプリケーションタスク（changefeed）を再開する際にチェックポイントTSOを再割り当てします。 |

### 例 {#example}

次のリクエストは、IDが`test1`のレプリケーションタスクを再開します。

```shell
curl -X POST http://127.0.0.1:8300/api/v2/changefeeds/test1/resume -d '{}'
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーションサブタスクリストのクエリ {#query-the-replication-subtask-list}

このAPIは同期インターフェースです。リクエストが成功した場合、すべてのレプリケーションサブタスク（`processor`）の基本情報が返されます。

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

パラメータは以下のように説明されています：

- `changefeed_id`：changefeedのID。
- `capture_id`：キャプチャーのID。

## 特定のレプリケーションサブタスクをクエリする {#query-a-specific-replication-subtask}

このAPIは同期インターフェースです。リクエストが成功すると、指定されたレプリケーションサブタスク（`processor`）の詳細情報が返されます。

### リクエストURI {#request-uri}

`GET /api/v2/processors/{changefeed_id}/{capture_id}`

### パラメータの説明 {#parameter-descriptions}

#### パスパラメータ {#path-parameters}

| パラメータ名          | 説明                                |
| :-------------- | :-------------------------------- |
| `changefeed_id` | クエリするレプリケーションサブタスクのchangefeedのID。 |
| `capture_id`    | クエリするレプリケーションサブタスクのキャプチャーのID。     |

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

パラメータは以下のように記述されています：

- `table_ids`：このキャプチャにレプリケートするテーブルID。

## TiCDCサービスのプロセスリストをクエリする {#query-the-ticdc-service-process-list}

このAPIは同期インターフェースです。リクエストが成功すると、すべてのレプリケーションプロセス（`capture`）の基本情報が返されます。

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

- `id`：キャプチャーID。
- `is_owner`：キャプチャーがオーナーであるかどうか。
- `address`：キャプチャーのアドレス。

## オーナーノードの追放 {#evict-an-owner-node}

このAPIは非同期インターフェースです。リクエストが成功した場合、`200 OK`が返されます。返された結果は、サーバーがコマンドを実行することに同意したことを意味するだけであり、コマンドが正常に実行されることを保証するものではありません。

### リクエストURI {#request-uri}

`POST /api/v2/owner/resign`

### 例 {#example}

以下のリクエストは、TiCDCの現在のオーナーノードを追放し、新しいオーナーノードを生成するための新しい選挙をトリガーします。

```shell
curl -X POST http://127.0.0.1:8300/api/v2/owner/resign
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## TiCDCサーバーのログレベルを動的に調整する {#dynamically-adjust-the-log-level-of-the-ticdc-server}

このAPIは同期インターフェースです。リクエストが成功した場合、`200 OK`が返されます。

### リクエストURI {#request-uri}

`POST /api/v2/log`

### リクエストパラメーター {#request-parameter}

#### リクエストボディのパラメーター {#parameter-for-the-request-body}

| パラメーター名     | 説明          |
| :---------- | :---------- |
| `log_level` | 設定したいログレベル。 |

`log_level`は[zapが提供するログレベル](https://godoc.org/go.uber.org/zap#UnmarshalText)をサポートしています: "debug", "info", "warn", "error", "dpanic" , "panic", and "fatal".

### 例 {#example}

```shell
curl -X POST -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v2/log -d '{"log_level":"debug"}'
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。
