---
title: Replicate Data to Kafka
summary: Learn how to replicate data to Apache Kafka using TiCDC.
---

# Apache Kafkaへのデータレプリケーション {#replicate-data-to-kafka}

このドキュメントでは、TiCDCを使用して、Apache Kafkaに増分データをレプリケートする変更フィードを作成する方法について説明します。

## レプリケーションタスクの作成 {#create-a-replication-task}

次のコマンドを実行して、レプリケーションタスクを作成します：

```shell
cdc cli changefeed create \
    --server=http://10.0.10.25:8300 \
    --sink-uri="kafka://127.0.0.1:9092/topic-name?protocol=canal-json&kafka-version=2.4.0&partition-num=6&max-message-bytes=67108864&replication-factor=1" \
    --changefeed-id="simple-replication-task"
```

```shell
Create changefeed successfully!
ID: simple-replication-task
Info: {"sink-uri":"kafka://127.0.0.1:9092/topic-name?protocol=canal-json&kafka-version=2.4.0&partition-num=6&max-message-bytes=67108864&replication-factor=1","opts":{},"create-time":"2023-12-21T22:04:08.103600025+08:00","start-ts":415241823337054209,"target-ts":0,"admin-job-type":0,"sort-engine":"unified","sort-dir":".","config":{"case-sensitive":false,"filter":{"rules":["*.*"],"ignore-txn-start-ts":null,"ddl-allow-list":null},"mounter":{"worker-num":16},"sink":{"dispatchers":null},"scheduler":{"type":"table-number","polling-time":-1}},"state":"normal","history":null,"error":null}
```

- `--サーバー`: The address of any TiCDC server in the TiCDC cluster.
- `--changefeed-id`: The ID of the replication task. The format must match the `^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$` regular expression. If this ID is not specified, TiCDC automatically generates a UUID (the version 4 format) as the ID.
- `--sink-uri`: The downstream address of the replication task. For details, see [Configure sink URI with `kafka`](#configure-sink-uri-for-kafka).
- `--start-ts`: Specifies the starting TSO of the changefeed. From this TSO, the TiCDC cluster starts pulling data. The default value is the current time.
- `--target-ts`: Specifies the ending TSO of the changefeed. To this TSO, the TiCDC cluster stops pulling data. The default value is empty, which means that TiCDC does not automatically stop pulling data.
- `--コンフィグレーション`: Specifies the changefeed configuration file. For details, see [TiCDC Changefeed Configuration Parameters](/ticdc/ticdc-changefeed-config.md).

## Configure sink URI for Kafka {#configure-sink-uri-for-kafka}

Sink URI is used to specify the connection information of the TiCDC target system. The format is as follows:

```shell
[scheme]://[userinfo@][host]:[port][/path]?[query_parameters]
```

サンプル構成：

```shell
--sink-uri="kafka://127.0.0.1:9092/topic-name?protocol=canal-json&kafka-version=2.4.0&partition-num=6&max-message-bytes=67108864&replication-factor=1"
```

以下は、Kafkaに構成できるシンクURIパラメータと値の説明です。

| パラメータ/パラメータの値                        | 説明                                                                                                                                                                                                                                                                                                                                                                       |
| :----------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `127.0.0.1`                          | 下流のKafkaサービスのIPアドレスです。                                                                                                                                                                                                                                                                                                                                                   |
| `9092`                               | 下流のKafkaのポートです。                                                                                                                                                                                                                                                                                                                                                          |
| `topic-name`                         | 変数。Kafkaのトピックの名前です。                                                                                                                                                                                                                                                                                                                                                      |
| `kafka-version`                      | 下流のKafkaのバージョンです（オプション、デフォルトは`2.4.0`です。現在、最初にサポートされるKafkaのバージョンは`0.11.0.2`で、最新のバージョンは`3.2.0`です。この値は、下流のKafkaの実際のバージョンと一致している必要があります）。                                                                                                                                                                                                                                    |
| `kafka-client-id`                    | レプリケーションタスクのKafkaクライアントIDを指定します（オプション、デフォルトは`TiCDC_sarama_producer_replication ID`です）。                                                                                                                                                                                                                                                                                   |
| `partition-num`                      | 下流のKafkaパーティションの数です（オプション。値は実際のパーティションの数よりも大きくてはいけません。それ以外の場合、レプリケーションタスクを正常に作成できません。デフォルトは`3`です）。                                                                                                                                                                                                                                                                       |
| `max-message-bytes`                  | 1回にKafkaブローカーに送信されるデータの最大サイズです（オプション、デフォルトは`10MB`です。v5.0.6とv4.0.6から、デフォルト値は`64MB`および`256MB`から`10MB`に変更されました）。                                                                                                                                                                                                                                                            |
| `replication-factor`                 | 保存できるKafkaメッセージのレプリカの数です（オプション、デフォルトは`1`です）。この値は、Kafkaの[`min.insync.replicas`](https://kafka.apache.org/33/documentation.html#brokerconfigs_min.insync.replicas)の値よりも大きくなければなりません。                                                                                                                                                                                       |
| `required-acks`                      | `Produce`リクエストで使用されるパラメータで、応答を受ける前にブローカーが受け取る必要があるレプリカの数を通知します。値のオプションは`0`（`NoResponse`：応答なし、`TCP ACK`のみが提供されます）、`1`（`WaitForLocal`：ローカルのコミットが正常に送信された後にのみ応答）、`-1`（`WaitForAll`：すべてのレプリカが正常にコミットされた後に応答。ブローカーの[`min.insync.replicas`](https://kafka.apache.org/33/documentation.html#brokerconfigs_min.insync.replicas)構成項目を使用して、最小限のレプリカ数を構成できます）（オプション、デフォルト値は`-1`です）。 |
| `compression`                        | メッセージの送信時に使用される圧縮アルゴリズムです（値のオプションは`none`、`lz4`、`gzip`、`snappy`、`zstd`で、デフォルトは`none`です）。Snappy圧縮ファイルは[公式Snappyフォーマット](https://github.com/google/snappy)である必要があることに注意してください。Snappy圧縮の他のバリアントはサポートされていません。                                                                                                                                                                  |
| `protocol`                           | メッセージがKafkaに出力されるプロトコルです。値のオプションは`canal-json`、`open-protocol`、`canal`、`avro`、`maxwell`です。                                                                                                                                                                                                                                                                                |
| `auto-create-topic`                  | `topic-name`がKafkaクラスタに存在しない場合、TiCDCが自動的にトピックを作成するかどうかを決定します（オプション、デフォルトは`true`です）。                                                                                                                                                                                                                                                                                      |
| `enable-tidb-extension`              | オプション。出力プロトコルが`canal-json`の場合、値が`true`の場合、TiCDCは[WATERMARKイベント](/ticdc/ticdc-canal-json.md#watermark-event)を送信し、Kafkaメッセージに[TiDB拡張フィールド](/ticdc/ticdc-canal-json.md#tidb-extension-field)を追加します。v6.1.0から、このパラメータは`avro`プロトコルにも適用されます。値が`true`の場合、TiCDCはKafkaメッセージに[3つのTiDB拡張フィールド](/ticdc/ticdc-avro-protocol.md#tidb-extension-fields)を追加します。                           |
| `max-batch-size`                     | v4.0.9で新しく追加されました。メッセージプロトコルが1つのKafkaメッセージに複数のデータ変更を出力できる場合、このパラメータは1つのKafkaメッセージの中のデータ変更の最大数を指定します。現在、Kafkaの`protocol`が`open-protocol`の場合のみ有効です（オプション、デフォルトは`16`です）。                                                                                                                                                                                                    |
| `enable-tls`                         | 下流のKafkaインスタンスに接続する際にTLSを使用するかどうか（オプション、デフォルトは`false`です）。                                                                                                                                                                                                                                                                                                                |
| `ca`                                 | 下流のKafkaインスタンスに接続するために必要なCA証明書ファイルのパス（オプション）。                                                                                                                                                                                                                                                                                                                            |
| `cert`                               | 下流のKafkaインスタンスに接続するために必要な証明書ファイルのパス（オプション）。                                                                                                                                                                                                                                                                                                                              |
| `key`                                | 下流のKafkaインスタンスに接続するために必要な証明書キーファイルのパス（オプション）。                                                                                                                                                                                                                                                                                                                            |
| `insecure-skip-verify`               | 下流のKafkaインスタンスに接続する際に証明書検証をスキップするかどうか（オプション、デフォルトは`false`です）。                                                                                                                                                                                                                                                                                                            |
| `sasl-user`                          | 下流のKafkaインスタンスに接続するために必要なSASL/PLAINまたはSASL/SCRAM認証のアイデンティティ（オプション）。                                                                                                                                                                                                                                                                                                      |
| `sasl-password`                      | 下流のKafkaインスタンスに接続するために必要なSASL/PLAINまたはSASL/SCRAM認証のパスワード（オプション）。特殊文字が含まれる場合は、URLエンコードする必要があります。                                                                                                                                                                                                                                                                          |
| `sasl-mechanism`                     | 下流のKafkaインスタンスに接続するために必要なSASL認証の名前です。値は`plain`、`scram-sha-256`、`scram-sha-512`、`gssapi`になります。                                                                                                                                                                                                                                                                            |
| `sasl-gssapi-auth-type`              | gssapi認証タイプです。値は`user`または`keytab`になります（オプション）。                                                                                                                                                                                                                                                                                                                           |
| `sasl-gssapi-keytab-path`            | gssapiキータブのパス（オプション）。                                                                                                                                                                                                                                                                                                                                                    |
| `sasl-gssapi-kerberos-config-path`   | gssapi kerberos構成パス（オプション）。                                                                                                                                                                                                                                                                                                                                              |
| `sasl-gssapi-service-name`           | gssapiサービス名（オプション）。                                                                                                                                                                                                                                                                                                                                                      |
| `sasl-gssapi-user`                   | gssapi認証のユーザー名（オプション）。                                                                                                                                                                                                                                                                                                                                                   |
| `sasl-gssapi-password`               | gssapi認証のパスワード（オプション）。特殊文字が含まれる場合は、URLエンコードする必要があります。                                                                                                                                                                                                                                                                                                                    |
| `sasl-gssapi-realm`                  | gssapiのレルム名（オプション）。                                                                                                                                                                                                                                                                                                                                                      |
| `sasl-gssapi-disable-pafxfast`       | gssapi PA-FX-FASTを無効にするかどうか（オプション）。                                                                                                                                                                                                                                                                                                                                      |
| `dial-timeout`                       | 下流のKafkaとの接続のタイムアウトです。デフォルト値は`10s`です。                                                                                                                                                                                                                                                                                                                                    |
| `read-timeout`                       | 下流のKafkaからの応答の取得のタイムアウトです。デフォルト値は`10s`です。                                                                                                                                                                                                                                                                                                                                |
| `write-timeout`                      | 下流のKafkaにリクエストを送信するタイムアウトです。デフォルト値は`10s`です。                                                                                                                                                                                                                                                                                                                              |
| `avro-decimal-handling-mode`         | `avro`プロトコルでのみ有効です。AvroがDECIMALフィールドをどのように処理するかを決定します。値は`string`または`precise`で、DECIMALフィールドを文字列または正確な浮動小数点数にマッピングすることを示します。                                                                                                                                                                                                                                               |
| `avro-bigint-unsigned-handling-mode` | `avro`プロトコルでのみ有効です。AvroがBIGINT UNSIGNEDフィールドをどのように処理するかを決定します。値は`string`または`long`で、BIGINT UNSIGNEDフィールドを64ビットの符号付き数または文字列にマッピングすることを示します。                                                                                                                                                                                                                                |

### ベストプラクティス {#best-practices}

- 独自のKafkaトピックを作成することをお勧めします。最低限、トピックがKafkaブローカーに送信できる1つのメッセージの最大データ量と、下流のKafkaパーティションの数を設定する必要があります。変更フィードを作成する場合、これら2つの設定はそれぞれ`max-message-bytes`と`partition-num`に対応します。
- まだ存在しないトピックで変更フィードを作成する場合、TiCDCは`partition-num`と`replication-factor`パラメータを使用してトピックを作成しようとします。これらのパラメータを明示的に指定することをお勧めします。
- ほとんどの場合、`canal-json`プロトコルを使用することをお勧めします。

> **Note:**
>
> `protocol`が`open-protocol`の場合、TiCDCは`max-message-bytes`を超えるメッセージを生成しないようにします。ただし、1つの変更が`max-message-bytes`を超える長さになる場合、静かな失敗を避けるために、TiCDCはこのメッセージを出力しようとし、ログに警告を出力します。

### TiCDCはKafkaの認証と認可を使用します {#ticdc-uses-the-authentication-and-authorization-of-kafka}

Kafka SASL認証を使用する場合の例は次のとおりです。

- SASL/PLAIN

  ```shell
  --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&sasl-user=alice-user&sasl-password=alice-secret&sasl-mechanism=plain"
  ```

- SASL/SCRAM

  SCRAM-SHA-256およびSCRAM-SHA-512はPLAINメソッドと類似しています。対応する認証メソッドとして`sasl-mechanism`を指定するだけです。

- SASL/GSSAPI

  SASL/GSSAPI `user` 認証:

  ```shell
  --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&sasl-mechanism=gssapi&sasl-gssapi-auth-type=user&sasl-gssapi-kerberos-config-path=/etc/krb5.conf&sasl-gssapi-service-name=kafka&sasl-gssapi-user=alice/for-kafka&sasl-gssapi-password=alice-secret&sasl-gssapi-realm=example.com"
  ```

  `sasl-gssapi-user`および`sasl-gssapi-realm`の値は、kerberosで指定された[principal](https://web.mit.edu/kerberos/krb5-1.5/krb5-1.5.4/doc/krb5-user/What-is-a-Kerberos-Principal_003f.html)に関連しています。たとえば、principalが`alice/for-kafka@example.com`に設定されている場合、`sasl-gssapi-user`および`sasl-gssapi-realm`はそれぞれ`alice/for-kafka`および`example.com`として指定されます。

  SASL/GSSAPI `keytab` 認証:

  ```shell
  --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&sasl-mechanism=gssapi&sasl-gssapi-auth-type=keytab&sasl-gssapi-kerberos-config-path=/etc/krb5.conf&sasl-gssapi-service-name=kafka&sasl-gssapi-user=alice/for-kafka&sasl-gssapi-keytab-path=/var/lib/secret/alice.key&sasl-gssapi-realm=example.com"
  ```

  SASL/GSSAPI認証メソッドの詳細については、[GSSAPIの構成](https://docs.confluent.io/platform/current/kafka/authentication_sasl/authentication_sasl_gssapi.html)を参照してください。

- TLS/SSL暗号化

  KafkaブローカーがTLS/SSL暗号化を有効にしている場合、`--sink-uri`に`-enable-tls=true`パラメータを追加する必要があります。自己署名証明書を使用する場合、`--sink-uri`で`ca`、`cert`、および`key`を指定する必要があります。

- ACL認可

  TiCDCが正常に機能するために必要な最小限の権限は次のとおりです。

  - Topic [リソースタイプ](https://docs.confluent.io/platform/current/kafka/authorization.html#resources)の`Create`、`Write`、および`Describe`権限。
  - Clusterリソースタイプの`DescribeConfigs`権限。

### Kafka Connect（Confluent Platform）とTiCDCを統合する {#integrate-ticdc-with-kafka-connect-confluent-platform}

Confluentが提供する[data connectors](https://docs.confluent.io/current/connect/managing/connectors.html)を使用して、リレーショナルデータベースや非リレーショナルデータベースにデータをストリーム配信するには、`avro`プロトコルを使用し、`schema-registry`に[Confluent Schema Registry](https://www.confluent.io/product/confluent-platform/data-compatibility/)のURLを指定する必要があります。

サンプル構成:

```shell
--sink-uri="kafka://127.0.0.1:9092/topic-name?&protocol=avro&replication-factor=3" --schema-registry="http://127.0.0.1:8081" --config changefeed_config.toml
```

```shell
[sink]
dispatchers = [
 {matcher = ['*.*'], topic = "tidb_{schema}_{table}"},
]
```

詳しい統合ガイドについては、[TiDBをConfluent Platformと統合するクイックスタートガイド](/ticdc/integrate-confluent-using-ticdc.md)を参照してください。

## Kafka SinkのTopicとPartitionディスパッチャのルールをカスタマイズする {#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink}

### マッチャールール {#matcher-rules}

次の`dispatchers`の構成を例に取る：

```toml
[sink]
dispatchers = [
  {matcher = ['test1.*', 'test2.*'], topic = "Topic expression 1", partition = "ts" },
  {matcher = ['test3.*', 'test4.*'], topic = "Topic expression 2", partition = "index-value" },
  {matcher = ['test1.*', 'test5.*'], topic = "Topic expression 3", partition = "table"},
  {matcher = ['test6.*'], partition = "ts"}
]
```

- マッチャールールに一致するテーブルについては、対応するトピック式で指定されたポリシーに従ってディスパッチされます。たとえば、`test3.aa` テーブルは "Topic expression 2" に従ってディスパッチされ、`test5.aa` テーブルは "Topic expression 3" に従ってディスパッチされます。
- 複数のマッチャールールに一致するテーブルについては、最初に一致したトピック式に従ってディスパッチされます。たとえば、`test1.aa` テーブルは "Topic expression 1" に従ってディスパッチされます。
- いずれのマッチャールールにも一致しないテーブルについては、`--sink-uri` で指定されたデフォルトトピックに対応するデータ変更イベントが送信されます。たとえば、`test10.aa` テーブルはデフォルトトピックに送信されます。
- マッチャールールに一致するテーブルでトピックディスパッチャが指定されていない場合、対応するデータ変更は `--sink-uri` で指定されたデフォルトトピックに送信されます。たとえば、`test6.aa` テーブルはデフォルトトピックに送信されます。

### トピックディスパッチャ {#topic-dispatchers}

トピック = "xxx" を使用してトピックディスパッチャを指定し、トピック式を使用して柔軟なトピックディスパッチポリシーを実装できます。トピックの総数は 1000 未満であることが推奨されます。

トピック式の形式は `[prefix]{schema}[middle][{table}][suffix]` です。

- `prefix`: オプションです。トピック名のプレフィックスを示します。
- `{schema}`: 必須です。スキーマ名に一致させるために使用されます。
- `middle`: オプションです。スキーマ名とテーブル名の間の区切り文字を示します。
- `{table}`: オプションです。テーブル名に一致させるために使用されます。
- `suffix`: オプションです。トピック名のサフィックスを示します。

`prefix`、`middle`、`suffix` には、`a-z`、`A-Z`、`0-9`、`.`、`_`、`-` のみを含めることができます。`{schema}` と `{table}` はどちらも小文字です。`{Schema}` や `{TABLE}` などのプレースホルダーは無効です。

いくつかの例:

- `matcher = ['test1.table1', 'test2.table2'], topic = "hello_{schema}_{table}"`
  - `test1.table1` に対応するデータ変更イベントは `hello_test1_table1` という名前のトピックに送信されます。
  - `test2.table2` に対応するデータ変更イベントは `hello_test2_table2` という名前のトピックに送信されます。
- `matcher = ['test3.*', 'test4.*'], topic = "hello_{schema}_world"`
  - `test3` のすべてのテーブルに対応するデータ変更イベントは `hello_test3_world` という名前のトピックに送信されます。
  - `test4` のすべてのテーブルに対応するデータ変更イベントは `hello_test4_world` という名前のトピックに送信されます。
- `matcher = ['*.*'], topic = "{schema}_{table}"`
  - TiCDC がリッスンするすべてのテーブルは、"schema\_table" ルールに従って別々のトピックにディスパッチされます。たとえば、`test.account` テーブルの場合、TiCDC はそのデータ変更ログを `test_account` という名前のトピックに送信します。

### DDL イベントのディスパッチ {#dispatch-ddl-events}

#### スキーマレベルの DDL {#schema-level-ddls}

特定のテーブルに関連しない DDL はスキーマレベルの DDL と呼ばれ、`create database` や `drop database` などが該当します。スキーマレベルの DDL に対応するイベントは、`--sink-uri` で指定されたデフォルトトピックに送信されます。

#### テーブルレベルの DDL {#table-level-ddls}

特定のテーブルに関連する DDL はテーブルレベルの DDL と呼ばれ、`alter table` や `create table` などが該当します。テーブルレベルの DDL に対応するイベントは、ディスパッチャの設定に従って対応するトピックに送信されます。

たとえば、`matcher = ['test.*'], topic = {schema}_{table}` のようなディスパッチャがある場合、DDL イベントは次のようにディスパッチされます:

- 単一のテーブルが DDL イベントに関与する場合、DDL イベントはそのまま対応するトピックに送信されます。たとえば、`drop table test.table1` のような DDL イベントの場合、そのイベントは `test_table1` という名前のトピックに送信されます。
- 複数のテーブルが DDL イベントに関与する場合（`rename table` / `drop table` / `drop view` などは複数のテーブルに関与する可能性があります）、DDL イベントは複数のイベントに分割され、対応するトピックに送信されます。たとえば、`rename table test.table1 to test.table10, test.table2 to test.table20` のような DDL イベントの場合、`rename table test.table1 to test.table10` というイベントは `test_table1` という名前のトピックに送信され、`rename table test.table2 to test.table20` というイベントは `test.table2` という名前のトピックに送信されます。

### パーティションディスパッチャ {#partition-dispatchers}

`partition = "xxx"` を使用してパーティションディスパッチャを指定できます。デフォルト、ts、index-value、table の 4 つのディスパッチャをサポートしています。ディスパッチャのルールは次のとおりです:

- default: 複数のユニークインデックス（プライマリキーを含む）が存在するか、古い値機能が有効になっている場合、イベントはテーブルモードでディスパッチされます。唯一のユニークインデックス（またはプライマリキー）が存在する場合、イベントは index-value モードでディスパッチされます。
- ts: 行の変更の commitTs をハッシュ化してイベントをディスパッチします。
- index-value: テーブルのプライマリキーまたはユニークインデックスの値をハッシュ化してイベントをディスパッチします。
- table: テーブルのスキーマ名とテーブル名をハッシュ化してイベントをディスパッチします。

> **Note:**
>
> v6.1.0 以降、構成の意味を明確にするために、パーティションディスパッチャを指定するために使用される構成は `dispatcher` から `partition` に変更され、`partition` は `dispatcher` のエイリアスとなりました。たとえば、次の 2 つのルールはまったく同じです。
>
> ```
> [sink]
> dispatchers = [
>    {matcher = ['*.*'], dispatcher = "ts"},
>    {matcher = ['*.*'], partition = "ts"},
> ]
> ```
>
> ただし、`dispatcher` と `partition` は同じルールには現れません。たとえば、次のルールは無効です。
>
> ```
> {matcher = ['*.*'], dispatcher = "ts", partition = "table"},
> ```

> **Warning:**
>
> [古い値機能](/ticdc/ticdc-manage-changefeed.md#output-the-historical-value-of-a-row-changed-event) が有効になっている場合（`enable-old-value = true`）、index-value ディスパッチャを使用すると、同じインデックス値を持つ行の変更の順序を保証できない場合があります。そのため、デフォルトディスパッチャを使用することをお勧めします。
>
> 詳細については、[TiCDC が古い値機能を有効にした場合、変更イベントの形式に何らかの変更が発生するのですか？](/ticdc/ticdc-faq.md#what-changes-occur-to-the-change-event-format-when-ticdc-enables-the-old-value-feature) を参照してください。

## 単一の大規模テーブルの負荷を複数の TiCDC ノードに分散させる {#scale-out-the-load-of-a-single-large-table-to-multiple-ticdc-nodes}

この機能は、単一の大規模テーブルのデータレプリケーション範囲をデータボリュームと分分毎の変更行数に応じて複数の範囲に分割し、各範囲でのデータボリュームと変更行数をほぼ同じにします。この機能は、これらの範囲を複数の TiCDC ノードに分散してレプリケーションし、複数の TiCDC ノードが同時に大規模な単一のテーブルをレプリケーションできるようにします。この機能は、次の 2 つの問題を解決できます:

- 単一の TiCDC ノードでは大規模な単一のテーブルを時間内にレプリケーションできない。
- TiCDC ノードが使用するリソース（CPU やメモリなど）が均等に分散されていない。

> **Warning:**
>
> TiCDC v7.0.0 では、大規模な単一のテーブルの負荷を Kafka changefeeds で分散することのみがサポートされています。

サンプル構成:

```toml
[scheduler]
# The default value is "false". You can set it to "true" to enable this feature.
enable-table-across-nodes = true
# When you enable this feature, it only takes effect for tables with the number of regions greater than the `region-threshold` value.
region-threshold = 100000
# When you enable this feature, it takes effect for tables with the number of rows modified per minute greater than the `write-key-threshold` value.
# Note:
# * The default value of `write-key-threshold` is 0, which means that the feature does not split the table replication range according to the number of rows modified in a table by default.
# * You can configure this parameter according to your cluster workload. For example, if it is configured as 30000, it means that the feature will split the replication range of a table when the number of modified rows per minute in the table exceeds 30000.
# * When `region-threshold` and `write-key-threshold` are configured at the same time:
#   TiCDC will check whether the number of modified rows is greater than `write-key-threshold` first.
#   If not, next check whether the number of Regions is greater than `region-threshold`.
write-key-threshold = 30000
```

次のSQLステートメントを使用して、テーブルが含むリージョンの数をクエリできます:

```sql
SELECT COUNT(*) FROM INFORMATION_SCHEMA.TIKV_REGION_STATUS WHERE DB_NAME="database1" AND TABLE_NAME="table1" AND IS_INDEX=0;
```

## Kafkaトピックの制限を超えるメッセージの処理 {#handle-messages-that-exceed-the-kafka-topic-limit}

Kafkaトピックは受信できるメッセージのサイズに制限を設定します。この制限は[`max.message.bytes`](https://kafka.apache.org/documentation/#topicconfigs_max.message.bytes)パラメータによって制御されます。TiCDC Kafkaシンクがこの制限を超えるデータを送信すると、チェンジフィードはエラーを報告し、データのレプリケーションを続行することができません。この問題を解決するために、TiCDCは以下の解決策を提供します。

### ハンドルキーのみを送信 {#send-handle-keys-only}

v7.1.2から、TiCDC Kafkaシンクは、メッセージのサイズが制限を超えた場合に、ハンドルキーのみを送信することをサポートします。これにより、メッセージのサイズを大幅に削減し、Kafkaトピックの制限を超えたことによるチェンジフィードのエラーやタスクの失敗を回避することができます。ハンドルキーとは以下のものを指します：

- レプリケートするテーブルに主キーがある場合、主キーがハンドルキーです。
- テーブルに主キーがなく、NOT NULLのユニークキーがある場合、NOT NULLのユニークキーがハンドルキーです。

現在、この機能は2つのエンコーディングプロトコルをサポートしています：Canal-JSONとOpen Protocol。Canal-JSONプロトコルを使用する場合、`sink-uri`で`enable-tidb-extension=true`を指定する必要があります。

サンプルのコンフィグレーションは以下の通りです：

```toml
[sink.kafka-config.large-message-handle]
# This configuration is introduced in v7.1.2.
# Empty by default, which means when the message size exceeds the limit, the changefeed fails.
# If this configuration is set to "handle-key-only", when the message size exceeds the limit, only the handle key is sent in the data field. If the message size still exceeds the limit, the changefeed fails.
large-message-handle-option = "handle-key-only"
```

### ハンドルキーのみを使用してメッセージを消費する {#consume-messages-with-handle-keys-only}

ハンドルキーのみを使用したメッセージ形式は次のとおりです：

```json
{
    "id": 0,
    "database": "test",
    "table": "tp_int",
    "pkNames": [
        "id"
    ],
    "isDdl": false,
    "type": "INSERT",
    "es": 1639633141221,
    "ts": 1639633142960,
    "sql": "",
    "sqlType": {
        "id": 4
    },
    "mysqlType": {
        "id": "int"
    },
    "data": [
        {
          "id": "2"
        }
    ],
    "old": null,
    "_tidb": {     // TiDB extension fields
        "commitTs": 163963314122145239,
        "onlyHandleKey": true
    }
}
```

Kafkaのコンシューマーがメッセージを受信すると、まず`onlyHandleKey`フィールドをチェックします。このフィールドが存在し、`true`である場合、メッセージには完全なデータのハンドルキーのみが含まれていることを意味します。この場合、完全なデータを取得するには、上流のTiDBをクエリし、[`tidb_snapshot`を使用して履歴データを読む](/read-historical-data.md)必要があります。

> **警告:**
>
> Kafkaコンシューマーがデータを処理し、TiDBをクエリする際、データはGCによって削除されている可能性があります。この状況を避けるために、TiDBクラスタの[GCライフタイムを変更](/system-variables.md#tidb_gc_life_time-new-in-v50)して、より大きな値にする必要があります。
