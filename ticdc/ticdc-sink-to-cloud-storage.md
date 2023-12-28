---
title: Replicate Data to Storage Services
summary: Learn how to replicate data to storage services using TiCDC, and learn about the storage path of the replicated data.
---

# ストレージサービスへのデータのレプリケーション {#replicate-data-to-storage-services}

TiDB v6.5.0から、TiCDCはAmazon S3、GCS、Azure Blob Storage、およびNFSを含むストレージサービスに行の変更イベントを保存することができるようになりました。このドキュメントでは、TiCDCを使用して変更データをこれらのストレージサービスにレプリケートする方法と、データがどのように保存されるかについて説明します。このドキュメントの構成は以下の通りです：

- [ストレージサービスへのデータのレプリケーション方法](#ストレージサービスへのデータのレプリケーション).
- [ストレージサービスにおけるデータの保存方法](#ストレージパスの構造).

## ストレージサービスへのデータのレプリケーション方法 {#replicate-change-data-to-storage-services}

次のコマンドを実行して、変更フィードタスクを作成します：

```shell
cdc cli changefeed create \
    --server=http://10.0.10.25:8300 \
    --sink-uri="s3://logbucket/storage_test?protocol=canal-json" \
    --changefeed-id="simple-replication-task"
```

出力は以下の通りです：

```shell
Info: {"upstream_id":7171388873935111376,"namespace":"default","id":"simple-replication-task","sink_uri":"s3://logbucket/storage_test?protocol=canal-json","create_time":"2023-12-21T18:52:05.566016967+08:00","start_ts":437706850431664129,"engine":"unified","config":{"case_sensitive":false,"enable_old_value":true,"force_replicate":false,"ignore_ineligible_table":false,"check_gc_safe_point":true,"enable_sync_point":false,"sync_point_interval":600000000000,"sync_point_retention":86400000000000,"filter":{"rules":["*.*"],"event_filters":null},"mounter":{"worker_num":16},"sink":{"protocol":"canal-json","schema_registry":"","csv":{"delimiter":",","quote":"\"","null":"\\N","include_commit_ts":false},"column_selectors":null,"transaction_atomicity":"none","encoder_concurrency":16,"terminator":"\r\n","date_separator":"none","enable_partition_separator":false},"consistent":{"level":"none","max_log_size":64,"flush_interval":2000,"storage":""}},"state":"normal","creator_version":"v7.1.3"}
```

- `--server`: TiCDCクラスター内の任意のTiCDCサーバーのアドレス。
- `--changefeed-id`: changefeedのID。フォーマットは`^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$`の正規表現に一致する必要があります。このIDが指定されていない場合、TiCDCは自動的にUUID（バージョン4のフォーマット）をIDとして生成します。
- `--sink-uri`: changefeedの下流アドレス。詳細については、[Sink URIの設定](#configure-sink-uri)を参照してください。
- `--start-ts`: changefeedの開始TSO。TiCDCはこのTSOからデータを取得し始めます。デフォルト値は現在時刻です。
- `--target-ts`: changefeedの終了TSO。TiCDCはこのTSOまでデータを取得し、自動的にデータの取得を停止します。デフォルト値は空で、TiCDCは自動的にデータの取得を停止しません。
- `--config`: changefeedの設定ファイル。詳細については、[TiCDC changefeedの設定パラメーター](/ticdc/ticdc-changefeed-config.md)を参照してください。

## Sink URIの設定 {#configure-sink-uri}

このセクションでは、Amazon S3、GCS、Azure Blob Storage、およびNFSなどのストレージサービスのSink URIを設定する方法について説明します。Sink URIは、TiCDCのターゲットシステムの接続情報を指定するために使用されます。フォーマットは以下のとおりです：

```shell
[scheme]://[host]/[path]?[query_parameters]
```

URIの`[query_parameters]`には、以下のパラメータを設定できます：

| パラメーター                  | 説明                                                                                                                                                                                                                                | デフォルト値     | 値の範囲                   |
| :---------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------- | :--------------------- |
| `worker-count`          | ダウンストリームでクラウドストレージにデータ変更を保存するための並行性。                                                                                                                                                                                              | `16`       | `[1, 512]`             |
| `flush-interval`        | ダウンストリームでクラウドストレージにデータ変更を保存するための間隔。                                                                                                                                                                                               | `5s`       | `[2s, 10m]`            |
| `file-size`             | データ変更ファイルは、バイト数がこのパラメーターの値を超える場合にクラウドストレージに保存されます。                                                                                                                                                                                | `67108864` | `[1048576, 536870912]` |
| `protocol`              | ダウンストリームに送信されるメッセージのプロトコル形式。                                                                                                                                                                                                      | N/A        | `canal-json`および`csv`   |
| `enable-tidb-extension` | `protocol`が`canal-json`に設定され、`enable-tidb-extension`が`true`に設定されている場合、TiCDCは[WATERMARKイベント](/ticdc/ticdc-canal-json.md#watermark-event)を送信し、[TiDB拡張フィールド](/ticdc/ticdc-canal-json.md#tidb-extension-field)をCanal-JSONメッセージに追加します。 | `false`    | `false`および`true`       |

> **Note:**
>
> `flush-interval`または`file-size`のいずれかが要件を満たすと、データ変更ファイルがダウンストリームに保存されます。
> `protocol`パラメーターは必須です。TiCDCがchangefeedを作成する際にこのパラメーターを受信しない場合、`CDC:ErrSinkUnknownProtocol`エラーが返されます。

### 外部ストレージのシンクURIを設定する {#configure-sink-uri-for-external-storage}

以下はAmazon S3の例です：

```shell
--sink-uri="s3://bucket/prefix?protocol=canal-json"
```

以下はGCSの設定の例です：

```shell
--sink-uri="gcs://bucket/prefix?protocol=canal-json"
```

以下はAzure Blob Storageの設定例です：

```shell
--sink-uri="azure://bucket/prefix?protocol=canal-json"
```

> **ヒント：**
>
> TiCDCにおけるAmazon S3、GCS、およびAzure Blob StorageのURIパラメーターの詳細については、[外部ストレージサービスのURIフォーマット](/external-storage-uri.md)を参照してください。

### NFSのシンクURIを構成する {#configure-sink-uri-for-nfs}

以下はNFSの例設定です：

```shell
--sink-uri="file:///my-directory/prefix?protocol=canal-json"
```

## ストレージパスの構造 {#storage-path-structure}

このセクションでは、データ変更レコード、メタデータ、およびDDLイベントのストレージパスの構造について説明します。

### データ変更レコード {#data-change-records}

データ変更レコードは、以下のパスに保存されます：

```shell
{scheme}://{prefix}/{schema}/{table}/{table-version-separator}/{partition-separator}/{date-separator}/CDC{num}.{extension}
```

- `scheme`：ストレージタイプを指定します。例えば、`s3`、`gcs`、`azure`、または`file`です。
- `prefix`：ユーザー定義の親ディレクトリを指定します。例えば、<code>s3://**bucket/bbb/ccc**</code>です。
- `schema`：スキーマ名を指定します。例えば、<code>s3://bucket/bbb/ccc/**test**</code>です。
- `table`：テーブル名を指定します。例えば、<code>s3://bucket/bbb/ccc/test/**table1**</code>です。
- `table-version-separator`：テーブルバージョンでパスを区切るセパレータを指定します。例えば、<code>s3://bucket/bbb/ccc/test/table1/**9999**</code>です。
- `partition-separator`：テーブルパーティションでパスを区切るセパレータを指定します。例えば、<code>s3://bucket/bbb/ccc/test/table1/9999/**20**</code>です。
- `date-separator`：トランザクションのコミット日付でファイルを分類します。デフォルト値は`day`です。値のオプションは次のとおりです：
  - `none`：`date-separator`はありません。例えば、`test.table1`バージョンが`9999`のすべてのファイルは`s3://bucket/bbb/ccc/test/table1/9999`に保存されます。
  - `year`：セパレータはトランザクションのコミット日付の年です。例えば、<code>s3://bucket/bbb/ccc/test/table1/9999/**2022**</code>です。
  - `month`：セパレータはトランザクションのコミット日付の年と月です。例えば、<code>s3://bucket/bbb/ccc/test/table1/9999/**2022-01**</code>です。
  - `day`：セパレータはトランザクションのコミット日付の年、月、および日です。例えば、<code>s3://bucket/bbb/ccc/test/table1/9999/**2022-01-02**</code>です。
- `num`：データ変更を記録するファイルのシリアル番号を保存します。例えば、<code>s3://bucket/bbb/ccc/test/table1/9999/2022-01-02/CDC**000005**.csv</code>です。
- `extension`：ファイルの拡張子を指定します。TiDB v6.5.0では、CSVおよびCanal-JSON形式がサポートされています。

> **Note:**
>
> テーブルバージョンは、アップストリームテーブルでDDL操作が実行された後にのみ変更され、新しいテーブルバージョンはアップストリームTiDBがDDLの実行を完了したTSOです。ただし、テーブルバージョンの変更はテーブルスキーマの変更を意味するものではありません。例えば、列にコメントを追加しても、スキーマファイルの内容は変更されません。

### インデックスファイル {#index-files}

インデックスファイルは、間違って書き込まれたデータを防ぐために使用されます。データ変更レコードと同じパスに保存されます。

```shell
{scheme}://{prefix}/{schema}/{table}/{table-version-separator}/{partition-separator}/{date-separator}/meta/CDC.index
```

現在のディレクトリで使用されている最大のファイル名を記録するインデックスファイルです。例えば：

    CDC000005.csv

この例では、このディレクトリ内の `CDC000001.csv` から `CDC000004.csv` までのファイルが占有されています。TiCDC クラスターでテーブルスケジューリングやノードの再起動が発生した場合、新しいノードはインデックスファイルを読み取り、`CDC000005.csv` が占有されているかどうかを判断します。占有されていない場合、新しいノードは `CDC000005.csv` からファイルを書き込みます。占有されている場合、他のノードが書き込んだデータを上書きすることを防ぐために、`CDC000006.csv` から書き込みを開始します。

### メタデータ {#metadata}

メタデータは以下のパスに保存されます：

```shell
{protocol}://{prefix}/metadata
```

メタデータはJSON形式のファイルです。例えば：

```json
{
    "checkpoint-ts":433305438660591626
}
```

- `checkpoint-ts`：`commit-ts`が`checkpoint-ts`より小さいトランザクションは、ダウンストリームのターゲットストレージに書き込まれます。

### DDLイベント {#ddl-events}

### テーブルレベルのDDLイベント {#ddl-events-at-the-table-level}

上流テーブルのDDLイベントにより、テーブルバージョンが変更されると、TiCDCは自動的に以下の処理を行います：

- データ変更レコードを書き込むための新しいパスに切り替えます。例えば、`test.table1`のバージョンが`441349361156227074`に変更された場合、TiCDCは` s3://bucket/bbb/ccc/test/table1/441349361156227074/2022-01-02/`パスに切り替えてデータ変更レコードを書き込みます。
- テーブルスキーマ情報を保存するためのスキーマファイルを以下のパスに生成します：

  ```shell
  {scheme}://{prefix}/{schema}/{table}/meta/schema_{table-version}_{hash}.json
  ```

  例えば、`schema_441349361156227074_3131721815.json`スキーマファイルの場合、このファイルに含まれるテーブルスキーマ情報は以下の通りです：

```json
{
    "Table":"table1",
    "Schema":"test",
    "Version":1,
    "TableVersion":441349361156227074,
    "Query":"ALTER TABLE test.table1 ADD OfficeLocation blob(20)",
    "Type":5,
    "TableColumns":[
        {
            "ColumnName":"Id",
            "ColumnType":"INT",
            "ColumnNullable":"false",
            "ColumnIsPk":"true"
        },
        {
            "ColumnName":"LastName",
            "ColumnType":"CHAR",
            "ColumnLength":"20"
        },
        {
            "ColumnName":"FirstName",
            "ColumnType":"VARCHAR",
            "ColumnLength":"30"
        },
        {
            "ColumnName":"HireDate",
            "ColumnType":"DATETIME"
        },
        {
            "ColumnName":"OfficeLocation",
            "ColumnType":"BLOB",
            "ColumnLength":"20"
        }
    ],
    "TableColumnsTotal":"5"
}
```

- `Table`: テーブル名。
- `Schema`: スキーマ名。
- `Version`: ストレージシンクのプロトコルバージョン。
- `TableVersion`: テーブルバージョン。
- `Query`: DDLステートメント。
- `Type`: DDLタイプ。
- `TableColumns`: 1つ以上のマップの配列で、それぞれがソーステーブルのカラムを記述する。
  - `ColumnName`: カラム名。
  - `ColumnType`: カラムタイプ。詳細については、[データタイプ](#data-type)を参照してください。
  - `ColumnLength`: カラムの長さ。詳細については、[データタイプ](#data-type)を参照してください。
  - `ColumnPrecision`: カラムの精度。詳細については、[データタイプ](#data-type)を参照してください。
  - `ColumnScale`: 小数点以下の桁数（スケール）。詳細については、[データタイプ](#data-type)を参照してください。
  - `ColumnNullable`: このオプションの値が`true`の場合、カラムはNULLになる可能性があります。
  - `ColumnIsPk`: このオプションの値が`true`の場合、カラムは主キーの一部です。
- `TableColumnsTotal`: `TableColumns`配列のサイズ。

### データベースレベルのDDLイベント {#ddl-events-at-the-database-level}

アップストリームデータベースでデータベースレベルのDDLイベントが実行されると、TiCDCは自動的に次のパスにスキーマファイルを生成してデータベースのスキーマ情報を保存します：

```shell
{scheme}://{prefix}/{schema}/meta/schema_{table-version}_{hash}.json
```

このファイルでは、`schema_441349361156227000_3131721815.json`というスキーマファイルを例にとって、このファイルに含まれるデータベースのスキーマ情報は以下の通りです:

```json
{
  "Table": "",
  "Schema": "schema1",
  "Version": 1,
  "TableVersion": 441349361156227000,
  "Query": "CREATE DATABASE `schema1`",
  "Type": 1,
  "TableColumns": null,
  "TableColumnsTotal": 0
}
```

### データ型 {#data-type}

このセクションでは、`schema_{table-version}_{hash}.json`ファイル（以下、以下のセクションでは「スキーマファイル」と呼ぶ）で使用されるデータ型について説明します。データ型は`T(M[, D])`と定義されます。詳細については、[データ型](/data-type-overview.md)を参照してください。

#### 整数型 {#integer-types}

TiDBの整数型は、`IT[(M)] [UNSIGNED]`と定義されます。ここで、

- `IT`は整数型であり、`TINYINT`、`SMALLINT`、`MEDIUMINT`、`INT`、`BIGINT`、または`BIT`になります。
- `M`は型の表示幅です。

スキーマファイルでは、整数型は次のように定義されます：

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{IT} [UNSIGNED]",
    "ColumnPrecision":"{M}"
}
```

#### 10進数のタイプ {#decimal-types}

TiDBの10進数のタイプは、`DT[(M,D)][UNSIGNED]`と定義されています。

- `DT`は浮動小数点型で、`FLOAT`、`DOUBLE`、`DECIMAL`、または`NUMERIC`になります。
- `M`はデータ型の精度、または総桁数です。
- `D`は小数点以下の桁数です。

スキーマファイルでは、10進数のタイプは以下のように定義されています：

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{DT} [UNSIGNED]",
    "ColumnPrecision":"{M}",
    "ColumnScale":"{D}"
}
```

#### 日付と時刻のタイプ {#date-and-time-types}

TiDBの日付タイプは、`DT`と定義されています。

- `DT`は日付タイプであり、`DATE`または`YEAR`になります。

スキーマファイルでは、日付タイプは以下のように定義されています：

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{DT}"
}
```

TiDBの時間型は、`TT[(M)]`と定義されています。

- `TT`は時間型であり、`TIME`、`DATETIME`、または`TIMESTAMP`になります。
- `M`は秒の精度を0から6の範囲で表します。

スキーマファイルでは、時間型は以下のように定義されています：

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{TT}",
    "ColumnScale":"{M}"
}
```

#### 文字列タイプ {#string-types}

TiDBの文字列タイプは、`ST[(M)]`と定義されています。

- `ST`は文字列タイプで、`CHAR`、`VARCHAR`、`TEXT`、`BINARY`、`BLOB`、または`JSON`になります。
- `M`は文字列の最大長です。

スキーマファイルでは、文字列タイプは以下のように定義されています：

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{ST}",
    "ColumnLength":"{M}"
}
```

#### EnumとSetタイプ {#enum-and-set-types}

スキーマファイルで、EnumとSetタイプは以下のように定義されます：

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{ENUM/SET}",
}
```
