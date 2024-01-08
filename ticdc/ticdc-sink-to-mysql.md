---
title: Replicate Data to MySQL-compatible Databases
summary: Learn how to replicate data to TiDB or MySQL using TiCDC.
---

# MySQL互換のデータベースにデータをレプリケートする {#replicate-data-to-mysql-compatible-databases}

このドキュメントでは、TiCDCを使用して、増分データを下流のTiDBデータベースまたは他のMySQL互換のデータベースにレプリケートする方法について説明します。また、災害シナリオでの最終的に整合性のあるレプリケーション機能の使用方法も紹介します。

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
- `--changefeed-id`: レプリケーションタスクのID。フォーマットは `^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$` 正規表現と一致する必要があります。このIDが指定されていない場合、TiCDCは自動的にUUID（バージョン4フォーマット）をIDとして生成します。
- `--sink-uri`: レプリケーションタスクのダウンストリームアドレス。詳細については、[MySQLまたはTiDBの`mysql`/`tidb`を使用してシンクURIを構成する](#configure-sink-uri-for-mysql-or-tidb)を参照してください。
- `--start-ts`: changefeedの開始TSOを指定します。このTSOから、TiCDCクラスターはデータの取得を開始します。デフォルト値は現在時刻です。
- `--target-ts`: changefeedの終了TSOを指定します。このTSOまで、TiCDCクラスターはデータの取得を停止します。デフォルト値は空で、これはTiCDCが自動的にデータの取得を停止しないことを意味します。
- `--config`: changefeedの構成ファイルを指定します。詳細については、[TiCDC Changefeed Configuration Parameters](/ticdc/ticdc-changefeed-config.md)を参照してください。

## MySQLまたはTiDBのためのシンクURIを構成する {#configure-sink-uri-for-mysql-or-tidb}

シンクURIは、TiCDCターゲットシステムの接続情報を指定するために使用されます。フォーマットは次のとおりです：

```
[scheme]://[userinfo@][host]:[port][/path]?[query_parameters]
```

> **Note:**
>
> `/path`はMySQLシンクには使用されません。

MySQLのサンプル構成：

```shell
--sink-uri="mysql://root:123456@127.0.0.1:3306"
```

以下は、MySQLまたはTiDBに構成できるシンクURIパラメータとパラメータ値の説明です。

| パラメータ/パラメータ値            | 説明                                                                                                                                                                                                                     |
| :---------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `root`                  | 下流データベースのユーザー名。                                                                                                                                                                                                        |
| `123456`                | 下流データベースのパスワード（Base64でエンコードすることができます）。                                                                                                                                                                                 |
| `127.0.0.1`             | 下流データベースのIPアドレス。                                                                                                                                                                                                       |
| `3306`                  | 下流データのポート。                                                                                                                                                                                                             |
| `worker-count`          | 下流へのSQLステートメントを並行して実行できる数（オプション、デフォルトは`16`）。                                                                                                                                                                           |
| `cache-prep-stmts`      | 下流でSQLを実行する際にプリペアドステートメントを使用し、クライアント側でプリペアドステートメントキャッシュを有効にするかどうかを制御します（オプション、デフォルトは`true`）。                                                                                                                           |
| `max-txn-row`           | 下流で実行できるトランザクションバッチのサイズ（オプション、デフォルトは`256`）。                                                                                                                                                                            |
| `ssl-ca`                | 下流のMySQLインスタンスに接続するために必要なCA証明書ファイルのパス（オプション）。                                                                                                                                                                          |
| `ssl-cert`              | 下流のMySQLインスタンスに接続するために必要な証明書ファイルのパス（オプション）。                                                                                                                                                                            |
| `ssl-key`               | 下流のMySQLインスタンスに接続するために必要な証明書キーファイルのパス（オプション）。                                                                                                                                                                          |
| `time-zone`             | 下流のMySQLインスタンスに接続する際に使用されるタイムゾーン。v4.0.8以降で有効です。これはオプションのパラメータです。このパラメータが指定されていない場合、TiCDCサービスプロセスのタイムゾーンが使用されます。このパラメータが`time-zone=""`のように空の値に設定されている場合、TiCDCが下流のMySQLインスタンスに接続する際にタイムゾーンが指定されず、下流のデフォルトのタイムゾーンが使用されます。 |
| `transaction-atomicity` | トランザクションの原子性レベル。これはオプションのパラメータで、デフォルト値は`none`です。値が`table`の場合、TiCDCは単一テーブルトランザクションの原子性を保証します。値が`none`の場合、TiCDCは単一テーブルトランザクションを分割します。                                                                                    |

シンクURIでデータベースのパスワードをBase64でエンコードするには、次のコマンドを使用します：

```shell
echo -n '123456' | base64   # '123456' is the password to be encoded.
```

エンコードされたパスワードは `MTIzNDU2` です：

```shell
MTIzNDU2
```

> **Note:**
>
> シンクURIに`! * ' ( ) ; : @ & = + $ , / ? % # [ ]`などの特殊文字が含まれる場合は、特殊文字をエスケープする必要があります。たとえば、[URIエンコーダー](https://www.urlencoder.org/)を使用します。

## 災害シナリオでの最終的な整合性レプリケーション {#eventually-consistent-replication-in-disaster-scenarios}

v6.1.1から、この機能はGAになります。v5.3.0から、TiCDCは上流のTiDBクラスタから増分データを下流クラスタのオブジェクトストレージまたはNFSにバックアップすることをサポートします。上流クラスタが災害に遭遇して利用できなくなった場合、TiCDCは下流のデータを最新の最終的に整合した状態に復元できます。これは、TiCDCが提供する最終的な整合性レプリケーション機能です。この機能により、アプリケーションを素早く下流クラスタに切り替えて、長時間のダウンタイムを回避し、サービスの連続性を向上させることができます。

現在、TiCDCはTiDBクラスタから別のTiDBクラスタまたはMySQL互換のデータベースシステム（Aurora、MySQL、MariaDBを含む）に増分データをレプリケートできます。上流クラスタがクラッシュした場合、TiCDCは下流クラスタのデータを5分以内に復元できます。前提条件として、TiCDCがクラッシュ前にデータを正常にレプリケートし、レプリケーション遅延が小さいことが条件です。最大で10秒のデータ損失が許容され、つまり、RTO <= 5分、P95 RPO <= 10秒です。

TiCDCのレプリケーション遅延は、次のシナリオで増加します。

- TPSが短時間で大幅に増加する。
- 上流で大規模または長時間のトランザクションが発生する。
- 上流のTiKVまたはTiCDCクラスタが再読み込みまたはアップグレードされる。
- `add index`などの時間のかかるDDLステートメントが上流で実行される。
- PDが積極的なスケジューリング戦略で構成されており、リージョンリーダーの頻繁な転送、またはリージョンの頻繁なマージまたは分割が発生している。

> **Note:**
>
> v6.1.1から、TiCDCの最終的な整合性レプリケーション機能はAmazon S3互換のオブジェクトストレージをサポートします。v6.1.4から、この機能はGCSおよびAzure互換のオブジェクトストレージをサポートします。

### 前提条件 {#prerequisites}

- TiCDCのリアルタイム増分データバックアップファイルを格納するための高可用性のオブジェクトストレージまたはNFSを準備します。これらのファイルは、上流で災害が発生した場合にアクセスできるようにする必要があります。
- 災害シナリオで最終的な整合性が必要なチェンジフィードにこの機能を有効にします。これを有効にするには、チェンジフィードの構成ファイルに次の構成を追加できます。

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

### Disaster recovery {#disaster-recovery}

プライマリクラスターで災害が発生した場合、`cdc redo`コマンドを実行してセカンダリクラスターで手動でリカバリする必要があります。リカバリプロセスは次のとおりです。

1. すべてのTiCDCプロセスが終了していることを確認します。これは、データリカバリ中にプライマリクラスターがサービスを再開するのを防ぎ、TiCDCがデータ同期を再開するのを防ぐためです。
2. データリカバリのためにcdcバイナリを使用します。次のコマンドを実行します：

```shell
cdc redo apply --tmp-dir="/tmp/cdc/redo/apply" \
    --storage="s3://logbucket/test-changefeed?endpoint=http://10.0.10.25:24927/" \
    --sink-uri="mysql://normal:123456@10.0.10.55:3306/"
```

このコマンドでは：

- `tmp-dir`: TiCDCの増分データバックアップファイルの一時ディレクトリを指定します。
- `storage`: TiCDCの増分データバックアップファイルを保存するアドレスを指定します。オブジェクトストレージのURIまたはNFSディレクトリのいずれかです。
- `sink-uri`: データを復元するためのセカンダリクラスターのアドレスを指定します。スキームは`mysql`のみです。
