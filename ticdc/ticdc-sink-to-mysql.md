---
title: Replicate Data to MySQL-compatible Databases
summary: Learn how to replicate data to TiDB or MySQL using TiCDC.
---

# MySQL互換のデータベースにデータをレプリケートする {#replicate-data-to-mysql-compatible-databases}

このドキュメントでは、TiCDCを使用して、インクリメンタルデータを下流のTiDBデータベースやその他のMySQL互換のデータベースにレプリケートする方法について説明します。また、災害シナリオでの最終的な整合性レプリケーション機能の使用方法も紹介します。

## レプリケーションタスクの作成 {#create-a-replication-task}

次のコマンドを実行して、レプリケーションタスクを作成します：

```shell
cdc cli changefeed create \
    --server=http://10.0.10.25:8300 \
    --sink-uri="mysql://root:123456@127.0.0.1:3306/" \
    --changefeed-id="simple-replication-task"
```

```shell
Create changefeed successfully!
ID: simple-replication-task
Info: {"sink-uri":"mysql://root:123456@127.0.0.1:3306/","opts":{},"create-time":"2023-12-21T22:04:08.103600025+08:00","start-ts":415241823337054209,"target-ts":0,"admin-job-type":0,"sort-engine":"unified","sort-dir":".","config":{"case-sensitive":false,"filter":{"rules":["*.*"],"ignore-txn-start-ts":null,"ddl-allow-list":null},"mounter":{"worker-num":16},"sink":{"dispatchers":null},"scheduler":{"type":"table-number","polling-time":-1}},"state":"normal","history":null,"error":null}
```

- `--server`: TiCDCクラスター内の任意のTiCDCサーバーのアドレス。
- `--changefeed-id`: レプリケーションタスクのID。フォーマットは `^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$` の正規表現に一致する必要があります。このIDが指定されていない場合、TiCDCは自動的にUUID（バージョン4フォーマット）をIDとして生成します。
- `--sink-uri`: レプリケーションタスクの下流アドレス。詳細については、[MySQL/TiDB用のsink URIを設定する](#configure-sink-uri-for-mysql-or-tidb)を参照してください。
- `--start-ts`: changefeedの開始TSOを指定します。このTSOから、TiCDCクラスターはデータの取得を開始します。デフォルト値は現在時刻です。
- `--target-ts`: changefeedの終了TSOを指定します。このTSOまで、TiCDCクラスターはデータの取得を停止します。デフォルト値は空で、TiCDCは自動的にデータの取得を停止しません。
- `--config`: changefeedの設定ファイルを指定します。詳細については、[TiCDC Changefeed Configuration Parameters](/ticdc/ticdc-changefeed-config.md)を参照してください。

## MySQL/TiDB用のsink URIを設定する {#configure-sink-uri-for-mysql-or-tidb}

Sink URIは、TiCDCターゲットシステムの接続情報を指定するために使用されます。フォーマットは次のとおりです：

    [scheme]://[userinfo@][host]:[port][/path]?[query_parameters]

> **Note:**
>
> `/path`はMySQLのシンクには使用されません。

MySQLのサンプルコンフィグレーション：

```shell
--sink-uri="mysql://root:123456@127.0.0.1:3306"
```

以下は、MySQLまたはTiDBに設定できるシンクURIパラメータとパラメータ値の説明です：

| パラメータ/パラメータ値            | 説明                                                                                                                                                                                                                                      |
| :---------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `root`                  | ダウンストリームデータベースのユーザー名です。                                                                                                                                                                                                                 |
| `123456`                | ダウンストリームデータベースのパスワードです（Base64でエンコードすることができます）。                                                                                                                                                                                          |
| `127.0.0.1`             | ダウンストリームデータベースのIPアドレスです。                                                                                                                                                                                                                |
| `3306`                  | ダウンストリームデータのポートです。                                                                                                                                                                                                                      |
| `worker-count`          | ダウンストリームに並行して実行できるSQLステートメントの数です（オプション、デフォルトは`16`）。                                                                                                                                                                                     |
| `cache-prep-stmts`      | ダウンストリームでSQLを実行する際にプリペアドステートメントを使用するかどうかを制御し、クライアント側でプリペアドステートメントキャッシュを有効にします（オプション、デフォルトは`true`）。                                                                                                                                      |
| `max-txn-row`           | ダウンストリームに実行できるトランザクションバッチのサイズです（オプション、デフォルトは`256`）。                                                                                                                                                                                     |
| `ssl-ca`                | ダウンストリームのMySQLインスタンスに接続するために必要なCA証明書ファイルのパスです（オプション）。                                                                                                                                                                                   |
| `ssl-cert`              | ダウンストリームのMySQLインスタンスに接続するために必要な証明書ファイルのパスです（オプション）。                                                                                                                                                                                     |
| `ssl-key`               | ダウンストリームのMySQLインスタンスに接続するために必要な証明書キーファイルのパスです（オプション）。                                                                                                                                                                                   |
| `time-zone`             | ダウンストリームのMySQLインスタンスに接続する際に使用するタイムゾーンです。v4.0.8以降有効です。これはオプションのパラメータです。このパラメータが指定されていない場合、TiCDCサービスプロセスのタイムゾーンが使用されます。このパラメータが`time-zone=""`のように空の値に設定されている場合、TiCDCがダウンストリームのMySQLインスタンスに接続する際にタイムゾーンが指定されず、ダウンストリームのデフォルトタイムゾーンが使用されます。 |
| `transaction-atomicity` | トランザクションのアトミック性レベルです。これはオプションのパラメータで、デフォルト値は`none`です。値が`table`の場合、TiCDCは単一テーブルトランザクションのアトミック性を保証します。値が`none`の場合、TiCDCは単一テーブルトランザクションを分割します。                                                                                             |

シンクURIでデータベースのパスワードをBase64でエンコードするには、次のコマンドを使用してください：

```shell
echo -n '123456' | base64   # '123456' is the password to be encoded.
```

エンコードされたパスワードは `MTIzNDU2` です：

```shell
MTIzNDU2
```

> **Note:**
>
> シンクURIに`! * ' ( ) ; : @ & = + $ , / ? % # [ ]`などの特殊文字が含まれる場合、特殊文字をエスケープする必要があります。例えば、[URIエンコーダー](https://www.urlencoder.org/)でエスケープすることができます。

## 災害シナリオにおける最終的に一貫したレプリケーション {#eventually-consistent-replication-in-disaster-scenarios}

v6.1.1から、この機能はGAになりました。v5.3.0から、TiCDCはアップストリームのTiDBクラスターからオブジェクトストレージまたはダウンストリームクラスターのNFSにインクリメンタルデータをバックアップすることができます。アップストリームクラスターが災害に遭遇して利用できなくなった場合、TiCDCはダウンストリームデータを最近の最終的に一貫した状態に復元することができます。これはTiCDCが提供する最終的に一貫したレプリケーション機能です。この機能により、アプリケーションを素早くダウンストリームクラスターに切り替えることができ、長時間のダウンタイムを回避し、サービスの連続性を向上させることができます。

現在、TiCDCはTiDBクラスターから別のTiDBクラスターまたはMySQL互換のデータベースシステム（Aurora、MySQL、MariaDBを含む）にインクリメンタルデータをレプリケートすることができます。アップストリームクラスターがクラッシュした場合、TiCDCはダウンストリームクラスターのデータを5分以内に復元することができます。これは、TiCDCがクラッシュ前にデータを正常にレプリケートし、レプリケーションラグが小さい場合に限ります。最大で10秒のデータ損失が許容され、つまり、RTO <= 5分、P95 RPO <= 10秒です。

TiCDCのレプリケーションラグは、次のシナリオで増加します。

- TPSが短時間で大幅に増加する。
- アップストリームで大きなまたは長いトランザクションが発生する。
- アップストリームのTiKVまたはTiCDCクラスターがリロードまたはアップグレードされる。
- `add index`などの時間のかかるDDLステートメントがアップストリームで実行される。
- PDが積極的なスケジューリング戦略で構成されているため、リージョンリーダーの頻繁な転送、またはリージョンのマージまたは分割が頻繁に発生する。

> **Note:**
>
> v6.1.1から、TiCDCの最終的に一貫したレプリケーション機能はAmazon S3互換のオブジェクトストレージをサポートしています。v6.1.4から、この機能はGCSおよびAzure互換のオブジェクトストレージをサポートしています。

### 前提条件 {#prerequisites}

- TiCDCのリアルタイムインクリメンタルデータバックアップファイルを保存するための高可用性のオブジェクトストレージまたはNFSを準備する。これらのファイルは、アップストリームで災害が発生した場合にアクセスできるようになります。
- 災害シナリオで最終的に一貫性を持つ必要があるchangefeedにこの機能を有効にする。これを有効にするには、changefeedの設定ファイルに次の設定を追加することができます。

```toml
[consistent]
# Consistency level. Options include:
# - none: the default value. In a non-disaster scenario, eventual consistency is only guaranteed if and only if finished-ts is specified.
# - eventual: Uses redo log to guarantee eventual consistency in case of the primary cluster disasters.
level = "eventual"

# Individual redo log file size, in MiB. By default, it's 64. It is recommended to be no more than 128.
max-log-size = 64

# The interval for flushing or uploading redo logs to Amazon S3, in milliseconds. It is recommended that this configuration be equal to or greater than 2000.
flush-interval = 2000

# The path under which redo log backup is stored. The scheme can be nfs (NFS directory), or Amazon S3, GCS, and Azure (uploaded to object storage).
storage = "$SCHEME://logbucket/test-changefeed?endpoint=http://$ENDPOINT/"
```

### ディザスターリカバリー {#disaster-recovery}

プライマリクラスターで災害が発生した場合、`cdc redo`コマンドを実行してセカンダリクラスターで手動でリカバリーする必要があります。リカバリーのプロセスは以下の通りです。

1. TiCDCプロセスがすべて終了していることを確認します。これは、データリカバリー中にプライマリクラスターがサービスを再開するのを防ぎ、TiCDCがデータ同期を再開するのを防ぐためです。
2. データリカバリーのためにcdcバイナリを使用します。次のコマンドを実行します：

```shell
cdc redo apply --tmp-dir="/tmp/cdc/redo/apply" \
    --storage="s3://logbucket/test-changefeed?endpoint=http://10.0.10.25:24927/" \
    --sink-uri="mysql://normal:123456@10.0.10.55:3306/"
```

このコマンドでは：

- `tmp-dir`：TiCDCの増分データバックアップファイルをダウンロードするための一時ディレクトリを指定します。
- `storage`：TiCDCの増分データバックアップファイルを保存するアドレスを指定します。オブジェクトストレージのURIまたはNFSディレクトリのいずれかです。
- `sink-uri`：データを復元するためのセカンダリクラスターのアドレスを指定します。スキームは`mysql`のみです。
