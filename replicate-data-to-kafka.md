---
title: Integrate Data with Apache Kafka and Apache Flink
summary: Learn how to replicate TiDB data to Apache Kafka and Apache Flink using TiCDC.
---

# Apache KafkaとApache Flinkでデータを統合する {#integrate-data-with-apache-kafka-and-apache-flink}

このドキュメントでは、[TiCDC](/ticdc/ticdc-overview.md)を使用してTiDBのデータをApache KafkaとApache Flinkにレプリケートする方法について説明します。このドキュメントの構成は以下の通りです。

1. TiCDCが含まれたTiDBクラスターを迅速にデプロイし、KafkaクラスターとFlinkクラスターを作成します。
2. TiDBからKafkaにデータをレプリケートするchangefeedを作成します。
3. go-tpcを使用してTiDBにデータを書き込みます。
4. Kafkaコンソールコンシューマーでデータを観察し、指定したKafkaトピックにデータがレプリケートされていることを確認します。
5. (オプション) Flinkクラスターを構成してKafkaデータを消費する。

上記の手順はラボ環境で実行されます。これらの手順を参考にして、本番環境でクラスターをデプロイすることもできます。

## ステップ1. 環境をセットアップする {#step-1-set-up-the-environment}

1. TiCDCが含まれたTiDBクラスターをデプロイします。

   ラボ環境またはテスト環境では、TiUP Playgroundを使用して迅速にTiCDCが含まれたTiDBクラスターをデプロイできます。

   ```shell
   tiup playground --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
   # クラスターのステータスを表示する
   tiup status
   ```

   TiUPがまだインストールされていない場合は、[TiUPのインストール](/tiup/tiup-overview.md#install-tiup)を参照してください。本番環境では、[TiCDCのデプロイ](/ticdc/deploy-ticdc.md)の指示に従ってTiCDCをデプロイできます。

2. Kafkaクラスターを作成します。

   - ラボ環境：[Apache Kakfaクイックスタート](https://kafka.apache.org/quickstart)を参照して、Kafkaクラスターを起動します。
   - 本番環境：[本番環境でのKafkaの実行](https://docs.confluent.io/platform/current/kafka/deployment.html)を参照して、Kafka本番クラスターをデプロイします。

3. (オプション) Flinkクラスターを作成します。

   - ラボ環境：[Apache Flinkの最初のステップ](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/try-flink/local_installation/)を参照して、Flinkクラスターを起動します。
   - 本番環境：[Apache Kafkaのデプロイ](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/deployment/overview/)を参照して、Flink本番クラスターをデプロイします。

## ステップ2. Kafka changefeedを作成する {#step-2-create-a-kafka-changefeed}

1. チェンジフィードの設定ファイルを作成します。

   Flinkの要件により、各テーブルの増分データは独立したトピックに送信され、プライマリキーの値に基づいて各イベントにパーティションが割り当てられる必要があります。そのため、次の内容を持つチェンジフィードの設定ファイル`changefeed.conf`を作成する必要があります。

       [sink]
       dispatchers = [
       {matcher = ['*.*'], topic = "tidb_{schema}_{table}", partition="index-value"},
       ]

   設定ファイルの`dispatchers`の詳細な説明については、[Kafka SinkのTopicおよびPartitionディスパッチャーのルールをカスタマイズする](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink)を参照してください。

2. Kafkaに増分データをレプリケートするためのチェンジフィードを作成します。

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://127.0.0.1:9092/kafka-topic-name?protocol=canal-json" --changefeed-id="kafka-changefeed" --config="changefeed.conf"
   ```

   - チェンジフィードが正常に作成された場合、チェンジフィードの情報（チェンジフィードIDなど）が以下のように表示されます。

     ```shell
     チェンジフィードの作成に成功しました！
     ID: kafka-changefeed
     Info: {... changfeed info json struct ...}
     ```

   - コマンドを実行しても結果が返されない場合は、コマンドを実行するサーバーとsink URIで指定されたKafkaマシンのネットワーク接続を確認してください。

   本番環境では、Kafkaクラスターには複数のブローカーノードがあります。そのため、複数のブローカーのアドレスをsink URIに追加することができます。これにより、Kafkaクラスターへの安定したアクセスが保証されます。Kafkaクラスターがダウンしても、チェンジフィードは動作し続けます。KafkaクラスターにIPアドレスがそれぞれ127.0.0.1:9092、127.0.0.2:9092、127.0.0.3:9092の3つのブローカーノードがあると仮定します。次のsink URIを使用してチェンジフィードを作成できます。

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://127.0.0.1:9092,127.0.0.2:9092,127.0.0.3:9092/kafka-topic-name?protocol=canal-json&partition-num=3&replication-factor=1&max-message-bytes=1048576" --config="changefeed.conf"
   ```

3. チェンジフィードを作成した後、次のコマンドを実行してチェンジフィードのステータスを確認します。

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed list --server="http://127.0.0.1:8300"
   ```

   チェンジフィードを管理するには、[TiCDCチェンジフィードの管理](/ticdc/ticdc-manage-changefeed.md)を参照してください。

## ステップ3. データを書き込んで変更ログを生成する {#step-3-write-data-to-generate-change-logs}

前のステップが完了したら、TiCDCはTiDBクラスターの増分データの変更ログをKafkaに送信します。このセクションでは、TiDBにデータを書き込んで変更ログを生成する方法について説明します。

1. サービスのワークロードをシミュレートします。

   ラボ環境で変更ログを生成するには、go-tpcを使用してTiDBクラスターにデータを書き込むことができます。具体的には、次のコマンドを実行してTiUP benchを使用して`tpcc`データベースを作成し、この新しいデータベースにデータを書き込むことができます。

   ```shell
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 prepare
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 run --time 300s
   ```

   go-tpcの詳細については、[TiDBでTPC-Cテストを実行する方法](/benchmark/benchmark-tidb-using-tpcc.md)を参照してください。

2. Kafkaトピックでデータを消費します。

   チェンジフィードが正常に動作すると、データがKafkaトピックに書き込まれます。`kafka-console-consumer.sh`を実行します。データが正常にKafkaトピックに書き込まれていることがわかります。

   ```shell
   ./bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --from-beginning --topic `${topic-name}`
   ```

TiDBデータベースの増分データは現在Kafkaに正常にレプリケートされています。次に、Flinkを使用してKafkaデータを消費することができます。または、特定のサービスシナリオのために独自のKafkaコンシューマクライアントを開発することもできます。

## (オプション)ステップ4. Flinkを設定してKafkaデータを消費する {#optional-step-4-configure-flink-to-consume-kafka-data}

1. Flink Kafkaコネクタをインストールします。

   Flinkエコシステムでは、Flink Kafkaコネクタを使用してKafkaデータを消費し、Flinkにデータを出力します。ただし、Flink Kafkaコネクタは自動的にインストールされません。使用するには、Flink Kafkaコネクタとその依存関係をFlinkインストールディレクトリに追加する必要があります。具体的には、次のjarファイルをFlinkインストールディレクトリの`lib`ディレクトリにダウンロードします。すでにFlinkクラスタを実行している場合は、新しいプラグインをロードするために再起動します。

   - [flink-connector-kafka-1.15.0.jar](https://repo.maven.apache.org/maven2/org/apache/flink/flink-connector-kafka/1.15.0/flink-connector-kafka-1.15.0.jar)
   - [flink-sql-connector-kafka-1.15.0.jar](https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-kafka/1.15.0/flink-sql-connector-kafka-1.15.0.jar)
   - [kafka-clients-3.2.0.jar](https://repo.maven.apache.org/maven2/org/apache/kafka/kafka-clients/3.2.0/kafka-clients-3.2.0.jar)

2. テーブルを作成します。

   Flinkがインストールされているディレクトリで、次のコマンドを実行してFlink SQLクライアントを起動します。

   ```shell
   [root@flink flink-1.15.0]# ./bin/sql-client.sh
   ```

   次に、次のコマンドを実行して、`tpcc_orders`という名前のテーブルを作成します。

   ```sql
   CREATE TABLE tpcc_orders (
       o_id INTEGER,
       o_d_id INTEGER,
       o_w_id INTEGER,
       o_c_id INTEGER,
       o_entry_d STRING,
       o_carrier_id INTEGER,
       o_ol_cnt INTEGER,
       o_all_local INTEGER
   ) WITH (
   'connector' = 'kafka',
   'topic' = 'tidb_tpcc_orders',
   'properties.bootstrap.servers' = '127.0.0.1:9092',
   'properties.group.id' = 'testGroup',
   'format' = 'canal-json',
   'scan.startup.mode' = 'earliest-offset',
   'properties.auto.offset.reset' = 'earliest'
   )
   ```

   `topic`と`properties.bootstrap.servers`を環境の実際の値に置き換えます。

3. テーブルのデータをクエリします。

   次のコマンドを実行して、`tpcc_orders`テーブルのデータをクエリします。

   ```sql
   SELECT * FROM tpcc_orders;
   ```

   このコマンドが実行されると、次の図に示すように、テーブルに新しいデータがあることがわかります。

   ![SQLクエリ結果](/media/integrate/sql-query-result.png)

Kafkaとのデータ統合は完了です。
