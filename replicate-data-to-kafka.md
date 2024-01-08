---
title: Integrate Data with Apache Kafka and Apache Flink
summary: Learn how to replicate TiDB data to Apache Kafka and Apache Flink using TiCDC.
---

# Apache KafkaとApache Flinkでデータを統合 {#integrate-data-with-apache-kafka-and-apache-flink}

このドキュメントでは、[TiCDC](/ticdc/ticdc-overview.md)を使用してTiDBデータをApache KafkaとApache Flinkにレプリケートする方法について説明します。このドキュメントの構成は次のとおりです。

1. TiCDCを含むTiDBクラスターを迅速にデプロイし、KafkaクラスターとFlinkクラスターを作成します。
2. TiDBからKafkaへデータをレプリケートするchangefeedを作成します。
3. go-tpcを使用してTiDBにデータを書き込みます。
4. Kafkaのコンソールコンシューマーでデータを確認し、指定されたKafkaトピックにデータがレプリケートされていることを確認します。
5. （オプション）FlinkクラスターをKafkaデータを消費するように構成します。

上記の手順はラボ環境で実行されます。これらの手順を参照して、本番環境でクラスターをデプロイすることもできます。

## ステップ1. 環境をセットアップする {#step-1-set-up-the-environment}

1. TiCDCを含むTiDBクラスターをデプロイします。

   ラボ環境またはテスト環境では、TiUP Playgroundを使用してTiCDCを含むTiDBクラスターを迅速にデプロイできます。

   ```shell
   tiup playground --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
   # クラスターステータスを表示
   tiup status
   ```

   TiUPがまだインストールされていない場合は、[TiUPのインストール](/tiup/tiup-overview.md#install-tiup)を参照してください。本番環境では、[TiCDCのデプロイ](/ticdc/deploy-ticdc.md)に従ってTiCDCをデプロイできます。

2. Kafkaクラスターを作成します。

   - ラボ環境：[Apache Kafkaクイックスタート](https://kafka.apache.org/quickstart)を参照してKafkaクラスターを起動します。
   - 本番環境：[本番環境でのKafkaの実行](https://docs.confluent.io/platform/current/kafka/deployment.html)を参照してKafka本番クラスターをデプロイします。

3. （オプション）Flinkクラスターを作成します。

   - ラボ環境：[Apache Flinkの最初のステップ](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/try-flink/local_installation/)を参照してFlinkクラスターを起動します。
   - 本番環境：[Apache Kafkaのデプロイ](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/deployment/overview/)を参照してFlink本番クラスターをデプロイします。

## ステップ2. Kafka changefeedを作成する {#step-2-create-a-kafka-changefeed}

1. changefeed構成ファイルを作成します。

   Flinkで必要とされるように、各テーブルの増分データは独立したトピックに送信され、プライマリキーの値に基づいて各イベントにパーティションがディスパッチされる必要があります。したがって、次の内容を持つ`changefeed.conf`というchangefeed構成ファイルを作成する必要があります。

   ```
   [sink]
   dispatchers = [
   {matcher = ['*.*'], topic = "tidb_{schema}_{table}", partition="index-value"},
   ]
   ```

   構成ファイルの`dispatchers`の詳細については、[Kafka SinkのTopicおよびPartitionディスパッチャのルールをカスタマイズする](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink)を参照してください。

2. Kafkaへの増分データのレプリケートのためのchangefeedを作成します：

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://127.0.0.1:9092/kafka-topic-name?protocol=canal-json" --changefeed-id="kafka-changefeed" --config="changefeed.conf"
   ```

   - changefeedが正常に作成された場合、changefeed IDなどのchangefeed情報が以下のように表示されます：

     ```shell
     changefeedの作成に成功しました！
     ID: kafka-changefeed
     Info: {... changfeed info json struct ...}
     ```

   - コマンドを実行しても結果が返されない場合は、コマンドを実行するサーバーとsink URIで指定されたKafkaマシンとのネットワーク接続を確認してください。

   本番環境では、Kafkaクラスターには複数のブローカーノードがあります。したがって、複数のブローカーのアドレスをsink URIに追加できます。これにより、Kafkaクラスターへの安定したアクセスが確保されます。Kafkaクラスターがダウンしても、changefeedは引き続き動作します。Kafkaクラスターに3つのブローカーノードがあり、それぞれのIPアドレスが127.0.0.1:9092、127.0.0.2:9092、127.0.0.3:9092であると仮定します。次のsink URIを使用してchangefeedを作成できます。

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://127.0.0.1:9092,127.0.0.2:9092,127.0.0.3:9092/kafka-topic-name?protocol=canal-json&partition-num=3&replication-factor=1&max-message-bytes=1048576" --config="changefeed.conf"
   ```

3. changefeedを作成した後、次のコマンドを実行してchangefeedのステータスを確認します：

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed list --server="http://127.0.0.1:8300"
   ```

   changefeedの管理については、[TiCDC Changefeedの管理](/ticdc/ticdc-manage-changefeed.md)を参照してください。

## ステップ3. 変更ログを生成するためのデータを書き込む {#step-3-write-data-to-generate-change-logs}

前の手順が完了した後、TiCDCはTiDBクラスターの増分データの変更ログをKafkaに送信します。このセクションでは、TiDBにデータを書き込んで変更ログを生成する方法について説明します。

1. サービスのワークロードをシミュレートします。

   ラボ環境で変更ログを生成するために、go-tpcを使用してTiDBクラスターにデータを書き込むことができます。具体的には、次のコマンドを実行して、TiUP benchを使用して`tpcc`データベースを作成し、この新しいデータベースにデータを書き込みます。

   ```shell
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 prepare
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 run --time 300s
   ```

   go-tpcの詳細については、[TiDBでTPC-Cテストを実行する方法](/benchmark/benchmark-tidb-using-tpcc.md)を参照してください。

2. Kafkaトピックでデータを消費します。

   changefeedが正常に動作している場合、データはKafkaトピックに書き込まれます。`kafka-console-consumer.sh`を実行します。そうすると、データがKafkaトピックに正常に書き込まれていることが確認できます。

   ```shell
   ./bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --from-beginning --topic `${topic-name}`
   ```

この時点で、TiDBデータベースの増分データがKafkaに正常にレプリケートされています。次に、Kafkaデータを消費するためにFlinkを使用するか、特定のサービスシナリオのためにKafkaコンシューマクライアントを開発することができます。

## (Optional) Step 4. Configure Flink to consume Kafka data {#optional-step-4-configure-flink-to-consume-kafka-data}

1. Install a Flink Kafka connector.

   Flinkエコシステムでは、Flink Kafkaコネクタを使用してKafkaデータを消費し、データをFlinkに出力します。ただし、Flink Kafkaコネクタは自動的にインストールされません。使用するには、Flinkをインストールした後、Flinkインストールディレクトリの`lib`ディレクトリにFlink Kafkaコネクタとその依存関係を追加する必要があります。具体的には、次のjarファイルをFlinkインストールディレクトリの`lib`ディレクトリにダウンロードします。Flinkクラスタを既に実行している場合は、新しいプラグインをロードするために再起動する必要があります。

   - [flink-connector-kafka-1.15.0.jar](https://repo.maven.apache.org/maven2/org/apache/flink/flink-connector-kafka/1.15.0/flink-connector-kafka-1.15.0.jar)
   - [flink-sql-connector-kafka-1.15.0.jar](https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-kafka/1.15.0/flink-sql-connector-kafka-1.15.0.jar)
   - [kafka-clients-3.2.0.jar](https://repo.maven.apache.org/maven2/org/apache/kafka/kafka-clients/3.2.0/kafka-clients-3.2.0.jar)

2. Create a table.

   Flinkがインストールされているディレクトリで、次のコマンドを実行してFlink SQLクライアントを起動します。

   ```shell
   [root@flink flink-1.15.0]# ./bin/sql-client.sh
   ```

   次に、次のコマンドを実行して`tpcc_orders`という名前のテーブルを作成します。

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

3. Query data of the table.

   `tpcc_orders`テーブルのデータをクエリするには、次のコマンドを実行します。

   ```sql
   SELECT * FROM tpcc_orders;
   ```

   このコマンドを実行すると、次の図に示すように、テーブルに新しいデータがあることがわかります。

   ![SQL query result](/media/integrate/sql-query-result.png)

Kafkaとのデータ統合が完了しました。
