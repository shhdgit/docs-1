---
title: Integrate Data with Confluent Cloud and Snowflake
summary: Learn how to stream TiDB data to Confluent Cloud, Snowflake, ksqlDB, and SQL Server.
---

# Confluent CloudとSnowflakeを統合する {#integrate-data-with-confluent-cloud-and-snowflake}

Confluentは、強力なデータ統合機能を提供するApache Kafka互換のストリーミングデータプラットフォームです。このプラットフォームでは、ノンストップのリアルタイムストリーミングデータにアクセスし、保存し、管理することができます。

TiDB v6.1.0から、TiCDCはAvro形式でConfluentに増分データをレプリケートすることをサポートしています。このドキュメントでは、[TiCDC](/ticdc/ticdc-overview.md)を使用してTiDBの増分データをConfluentにレプリケートし、さらにConfluent Cloudを介してSnowflake、ksqlDB、およびSQL Serverにデータをレプリケートする方法について紹介します。このドキュメントの構成は次のとおりです。

1. TiDBクラスターをTiCDCを含めて迅速にデプロイします。
2. TiDBからConfluent Cloudにデータをレプリケートするchangefeedを作成します。
3. Confluent CloudからSnowflake、ksqlDB、およびSQL Serverにデータをレプリケートするコネクタを作成します。
4. go-tpcを使用してTiDBにデータを書き込み、Snowflake、ksqlDB、およびSQL Serverでデータの変更を観察します。

上記の手順はラボ環境で実行されます。これらの手順を参照して、本番環境でクラスターをデプロイすることもできます。

## 増分データをConfluent Cloudにレプリケートする {#replicate-incremental-data-to-confluent-cloud}

### ステップ1. 環境をセットアップする {#step-1-set-up-the-environment}

1. TiCDCを含めたTiDBクラスターをデプロイします。

   ラボ環境やテスト環境では、TiUP Playgroundを使用してTiCDCを含めたTiDBクラスターを迅速にデプロイできます。

   ```shell
   tiup playground --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
   # クラスターステータスを表示
   tiup status
   ```

   TiUPがまだインストールされていない場合は、[TiUPのインストール](/tiup/tiup-overview.md#install-tiup)を参照してください。本番環境では、[TiCDCのデプロイ](/ticdc/deploy-ticdc.md)に従ってTiCDCをデプロイできます。

2. Confluent Cloudに登録し、Confluentクラスターを作成します。

   ベーシッククラスターを作成し、インターネット経由でアクセス可能にします。詳細については、[Confluent Cloudのクイックスタート](https://docs.confluent.io/cloud/current/get-started/index.html)を参照してください。

### ステップ2. アクセスキーペアを作成する {#step-2-create-an-access-key-pair}

1. クラスターAPIキーを作成します。

   [Confluent Cloud](https://confluent.cloud)にサインインします。**データ統合** > **APIキー** > **キーの作成**を選択します。表示される**APIキーのスコープを選択**ページで、**グローバルアクセス**を選択します。

   作成後、以下のようにキーペアファイルが生成されます。

   ```
   === Confluent Cloud API key: xxx-xxxxx ===

   API key:
   L5WWA4GK4NAT2EQV

   API secret:
   xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

   ブートストラップサーバー:
   xxx-xxxxx.ap-east-1.aws.confluent.cloud:9092
   ```

2. スキーマレジストリのエンドポイントを記録します。

   Confluent Cloudコンソールで、**スキーマレジストリ** > **APIエンドポイント**を選択します。スキーマレジストリのエンドポイントを記録します。以下は例です。

   ```
   https://yyy-yyyyy.us-east-2.aws.confluent.cloud
   ```

3. スキーマレジストリAPIキーを作成します。

   Confluent Cloudコンソールで、**スキーマレジストリ** > **API資格情報**を選択します。**編集**をクリックしてから**キーの作成**をクリックします。

   作成後、以下のようにキーペアファイルが生成されます。

   ```
   === Confluent Cloud API key: yyy-yyyyy ===
   API key:
   7NBH2CAFM2LMGTH7
   API secret:
   xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   ```

   また、Confluent CLIを使用してこの手順を実行することもできます。詳細については、[Confluent CLIをConfluent Cloudクラスターに接続](https://docs.confluent.io/confluent-cli/current/connect.html)を参照してください。

### ステップ3. Kafka changefeedを作成する {#step-3-create-a-kafka-changefeed}

1. changefeedの構成ファイルを作成します。

   AvroとConfluent Connectorの要件により、各テーブルの増分データは独立したトピックに送信され、プライマリキーの値に基づいて各イベントにパーティションが割り当てられる必要があります。したがって、次の内容で`changefeed.conf`という名前のchangefeedの構成ファイルを作成する必要があります。

   ```
   [sink]
   dispatchers = [
   {matcher = ['*.*'], topic = "tidb_{schema}_{table}", partition="index-value"},
   ]
   ```

   構成ファイルの`dispatchers`の詳細な説明については、[Kafka SinkのTopicおよびPartitionディスパッチャのルールをカスタマイズ](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink)を参照してください。

2. Confluent Cloudに増分データをレプリケートするchangefeedを作成します。

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://<broker_endpoint>/ticdc-meta?protocol=avro&replication-factor=3&enable-tls=true&auto-create-topic=true&sasl-mechanism=plain&sasl-user=<broker_api_key>&sasl-password=<broker_api_secret>" --schema-registry="https://<schema_registry_api_key>:<schema_registry_api_secret>@<schema_registry_endpoint>" --changefeed-id="confluent-changefeed" --config changefeed.conf
   ```

   以下のフィールドの値を[ステップ2. アクセスキーペアを作成する](#step-2-create-an-access-key-pair)で作成または記録した値に置き換える必要があります。

   - `<broker_endpoint>`
   - `<broker_api_key>`
   - `<broker_api_secret>`
   - `<schema_registry_api_key>`
   - `<schema_registry_api_secret>`
   - `<schema_registry_endpoint>`

   なお、`<schema_registry_api_secret>`の値を置き換える前に[HTML URLエンコーディングリファレンス](https://www.w3schools.com/tags/ref_urlencode.asp)に基づいてエンコードする必要があります。上記のフィールドをすべて置き換えた後、構成ファイルは以下のようになります。

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://xxx-xxxxx.ap-east-1.aws.confluent.cloud:9092/ticdc-meta?protocol=avro&replication-factor=3&enable-tls=true&auto-create-topic=true&sasl-mechanism=plain&sasl-user=L5WWA4GK4NAT2EQV&sasl-password=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" --schema-registry="https://7NBH2CAFM2LMGTH7:xxxxxxxxxxxxxxxxxx@yyy-yyyyy.us-east-2.aws.confluent.cloud" --changefeed-id="confluent-changefeed" --config changefeed.conf
   ```

   - changefeedを作成するコマンドを実行します。

     - changefeedが正常に作成された場合、changefeedの情報（changefeed IDなど）が表示されます。

       ```shell
       changefeedの作成に成功しました！
       ID: confluent-changefeed
       Info: {... changfeed info json struct ...}
       ```

     - コマンドを実行しても結果が返されない場合は、コマンドを実行するサーバーとConfluent Cloudの間のネットワーク接続を確認してください。詳細については、[Confluent Cloudへの接続のテスト](https://docs.confluent.io/cloud/current/networking/testing.html)を参照してください。

3. changefeedを作成した後、次のコマンドを実行してchangefeedのステータスを確認します。

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed list --server="http://127.0.0.1:8300"
   ```

   changefeedの管理については、[TiCDC Changefeedの管理](/ticdc/ticdc-manage-changefeed.md)を参照してください。

### ステップ4. データを書き込んで変更ログを生成する {#step-4-write-data-to-generate-change-logs}

前の手順が完了した後、TiCDCはTiDBクラスタの増分データの変更ログをConfluent Cloudに送信します。このセクションでは、TiDBにデータを書き込んで変更ログを生成する方法について説明します。

1. サービスのワークロードをシミュレートします。

   ラボ環境で変更ログを生成するには、go-tpcを使用してTiDBクラスタにデータを書き込むことができます。具体的には、次のコマンドを実行してTiDBクラスタに`tpcc`というデータベースを作成します。その後、TiUP benchを使用してこの新しいデータベースにデータを書き込みます。

   ```shell
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 prepare
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 run --time 300s
   ```

   go-tpcの詳細については、[TiDBでTPC-Cテストを実行する方法](/benchmark/benchmark-tidb-using-tpcc.md)を参照してください。

2. Confluent Cloudでデータを観察します。

   ![Confluent topics](/media/integrate/confluent-topics.png)

   Confluent Cloudコンソールで**Topics**をクリックします。ターゲットのトピックが作成され、データが受信されていることがわかります。この時点で、TiDBデータベースの増分データがConfluent Cloudに正常にレプリケートされています。

## Snowflakeとデータを統合する {#integrate-data-with-snowflake}

Snowflakeはクラウドネイティブのデータウェアハウスです。Confluentを使用すると、Snowflake Sink Connectorsを作成してTiDBの増分データをSnowflakeにレプリケートできます。

### 前提条件 {#prerequisites}

- Snowflakeクラスタを登録および作成していること。[Snowflakeのはじめ方](https://docs.snowflake.com/en/user-guide-getting-started.html)を参照してください。
- Snowflakeクラスタに接続する前に、そのためのプライベートキーを生成していること。[キーペア認証とキーペアのローテーション](https://docs.snowflake.com/en/user-guide/key-pair-auth.html)を参照してください。

### 統合手順 {#integration-procedure}

1. Snowflakeでデータベースとスキーマを作成します。

   Snowflakeコントロールコンソールで**Data** > **Database**を選択します。`TPCC`という名前のデータベースと`TiCDC`という名前のスキーマを作成します。

2. Confluent Cloudコンソールで**Data integration** > **Connectors** > **Snowflake Sink**を選択します。以下のページが表示されます。

   ![Add snowflake sink connector](/media/integrate/add-snowflake-sink-connector.png)

3. Snowflakeにレプリケートしたいトピックを選択し、次のページに移動します。

   ![Configuration](/media/integrate/configuration.png)

4. Snowflakeに接続するための認証情報を指定します。前の手順で作成した値を**Database name**と**Schema name**に入力します。次のページに移動します。

   ![Configuration](/media/integrate/configuration.png)

5. **Configuration**ページで、**Input Kafka record value format**と**Input Kafka record key format**の両方に`AVRO`を選択します。その後、**Continue**をクリックします。コネクタが作成され、ステータスが**Running**になるまで待ちます。これには数分かかる場合があります。

   ![Data preview](/media/integrate/data-preview.png)

6. Snowflakeコンソールで**Data** > **Database** > **TPCC** > **TiCDC**を選択します。TiDBの増分データがSnowflakeにレプリケートされていることがわかります。Snowflakeとのデータ統合が完了しました（前の図を参照）。ただし、Snowflakeのテーブル構造はTiDBと異なり、データは増分で挿入されます。ほとんどのシナリオでは、SnowflakeのデータはTiDBのデータのレプリカとして期待されますが、TiDBの変更ログを保存するのではなく、Snowflakeに挿入されます。この問題は次のセクションで解決されます。

### SnowflakeでTiDBテーブルのデータレプリカを作成する {#create-data-replicas-of-tidb-tables-in-snowflake}

前のセクションでは、TiDBの増分データの変更ログがSnowflakeにレプリケートされました。このセクションでは、SnowflakeのTASKおよびSTREAM機能を使用して、`INSERT`、`UPDATE`、`DELETE`のイベントタイプに応じてこれらの変更ログを処理し、上流と同じ構造のテーブルに書き込む方法について説明します。これにより、SnowflakeにTiDBテーブルのデータレプリカが作成されます。以下は`ITEM`テーブルの構造です。

```
CREATE TABLE `item` (
  `i_id` int(11) NOT NULL,
  `i_im_id` int(11) DEFAULT NULL,
  `i_name` varchar(24) DEFAULT NULL,
  `i_price` decimal(5,2) DEFAULT NULL,
  `i_data` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`i_id`)
);
```

Snowflakeでは、Confluent Snowflake Sink Connectorによって自動的に作成される`TIDB_TEST_ITEM`というテーブルがあります。テーブルの構造は以下の通りです：

```
create or replace TABLE TIDB_TEST_ITEM (
        RECORD_METADATA VARIANT,
        RECORD_CONTENT VARIANT
);
```

1. Snowflakeで、TiDBと同じ構造のテーブルを作成します：

   ```
   create or replace table TEST_ITEM (
       i_id INTEGER primary key,
       i_im_id INTEGER,
       i_name VARCHAR,
       i_price DECIMAL(36,2),
       i_data VARCHAR
   );
   ```

2. `TIDB_TEST_ITEM`のストリームを作成し、`append_only`を`true`に設定します。

   ```
   create or replace stream TEST_ITEM_STREAM on table TIDB_TEST_ITEM append_only=true;
   ```

   この方法で、作成されたストリームはリアルタイムで`INSERT`イベントのみをキャプチャします。具体的には、TiDBの`ITEM`に新しい変更ログが生成されると、その変更ログは`TIDB_TEST_ITEM`に挿入され、ストリームによってキャプチャされます。

3. ストリーム内のデータを処理します。イベントの種類に応じて、`TEST_ITEM`テーブル内のストリームデータを挿入、更新、または削除します。

   ```
   -- TEST_ITEMテーブルにデータをマージ
   merge into TEST_ITEM n
     using
         -- TEST_ITEM_STREAMをクエリ
         (SELECT RECORD_METADATA:key as k, RECORD_CONTENT:val as v from TEST_ITEM_STREAM) stm
         -- ストリームをテーブルにマッチさせる条件はi_idが等しいこと
         on k:i_id = n.i_id
     -- TEST_ITEMテーブルにi_idにマッチするレコードがあり、vが空の場合、このレコードを削除
     when matched and IS_NULL_VALUE(v) = true then
         delete

     -- TEST_ITEMテーブルにi_idにマッチするレコードがあり、vが空でない場合、このレコードを更新
     when matched and IS_NULL_VALUE(v) = false then
         update set n.i_data = v:i_data, n.i_im_id = v:i_im_id, n.i_name = v:i_name, n.i_price = v:i_price

     -- TEST_ITEMテーブルにi_idにマッチするレコードがない場合、このレコードを挿入
     when not matched then
         insert
             (i_data, i_id, i_im_id, i_name, i_price)
         values
             (v:i_data, v:i_id, v:i_im_id, v:i_name, v:i_price)
   ;
   ```

   上記の例では、Snowflakeの`MERGE INTO`ステートメントを使用して、ストリームとテーブルを特定の条件でマッチさせ、それに応じた操作（レコードの削除、更新、または挿入）を実行します。この例では、次の3つのシナリオに対して3つの`WHERE`句が使用されています：

   - ストリームとテーブルが一致し、ストリーム内のデータが空の場合、テーブル内のレコードを削除します。
   - ストリームとテーブルが一致し、ストリーム内のデータが空でない場合、テーブル内のレコードを更新します。
   - ストリームとテーブルが一致しない場合、テーブルにレコードを挿入します。

4. ステップ3のステートメントを定期的に実行して、データが常に最新であることを確認します。また、Snowflakeの`SCHEDULED TASK`機能を使用することもできます：

   ```
   -- MERGE INTOステートメントを定期的に実行するためのTASKを作成
   create or replace task STREAM_TO_ITEM
       warehouse = test
       -- 1分ごとにTASKを実行
       schedule = '1 minute'
   when
       -- TEST_ITEM_STREAMにデータがない場合、TASKをスキップ
       system$stream_has_data('TEST_ITEM_STREAM')
   as
   -- TEST_ITEMテーブルにデータをマージ。ステートメントは前述の例と同じです
   merge into TEST_ITEM n
     using
         (select RECORD_METADATA:key as k, RECORD_CONTENT:val as v from TEST_ITEM_STREAM) stm
         on k:i_id = n.i_id
     when matched and IS_NULL_VALUE(v) = true then
         delete
     when matched and IS_NULL_VALUE(v) = false then
         update set n.i_data = v:i_data, n.i_im_id = v:i_im_id, n.i_name = v:i_name, n.i_price = v:i_price
     when not matched then
         insert
             (i_data, i_id, i_im_id, i_name, i_price)
         values
             (v:i_data, v:i_id, v:i_im_id, v:i_name, v:i_price)
   ;
   ```

この時点で、特定のETL機能を備えたデータチャネルが確立されています。このデータチャネルを通じて、TiDBの増分データ変更ログをSnowflakeにレプリケートし、TiDBのデータレプリカを維持し、Snowflakeのデータを使用できます。

最後のステップは、定期的に`TIDB_TEST_ITEM`テーブル内の不要なデータをクリーンアップすることです。

```
-- Clean up the TIDB_TEST_ITEM table every two hours
create or replace task TRUNCATE_TIDB_TEST_ITEM
    warehouse = test
    schedule = '120 minute'
when
    system$stream_has_data('TIDB_TEST_ITEM')
as
    TRUNCATE table TIDB_TEST_ITEM;
```

## ksqlDBを使用したデータの統合 {#integrate-data-with-ksqldb}

ksqlDBは、ストリーム処理アプリケーション向けに特別に設計されたデータベースです。Confluent Cloud上でksqlDBクラスタを作成し、TiCDCによってレプリケートされた増分データにアクセスできます。

1. Confluent Cloudコンソールで**ksqlDB**を選択し、指示に従ってksqlDBクラスタを作成します。

   ksqlDBクラスタのステータスが**Running**になるまで待ちます。このプロセスには数分かかります。

2. ksqlDBエディタで、次のコマンドを実行して、`tidb_tpcc_orders`トピックにアクセスするストリームを作成します。

   ```sql
   CREATE STREAM orders (o_id INTEGER, o_d_id INTEGER, o_w_id INTEGER, o_c_id INTEGER, o_entry_d STRING, o_carrier_id INTEGER, o_ol_cnt INTEGER, o_all_local INTEGER) WITH (kafka_topic='tidb_tpcc_orders', partitions=3, value_format='AVRO');
   ```

3. 次のコマンドを実行して、orders STREAMデータを確認します。

   ```sql
   SELECT * FROM ORDERS EMIT CHANGES;
   ```

   ![Select from orders](/media/integrate/select-from-orders.png)

   前述の図に示されているように、増分データがksqlDBにレプリケートされていることがわかります。ksqlDBとのデータ統合が完了しました。

## SQL Serverを使用したデータの統合 {#integrate-data-with-sql-server}

Microsoft SQL Serverは、Microsoftによって開発されたリレーショナルデータベース管理システム（RDBMS）です。Confluentを使用すると、SQL Server Sink Connectorsを作成して、TiDBの増分データをSQL Serverにレプリケートできます。

1. SQL Serverに接続し、`tpcc`という名前のデータベースを作成します。

   ```shell
   [ec2-user@ip-172-1-1-1 bin]$ sqlcmd -S 10.61.43.14,1433 -U admin
   Password:
   1> create database tpcc
   2> go
   1> select name from master.dbo.sysdatabases
   2> go
   name
   ----------------------------------------------------------------------
   master
   tempdb
   model
   msdb
   rdsadmin
   tpcc
   (6 rows affected)
   ```

2. Confluent Cloudコンソールで、**Data integration** > **Connectors** > **Microsoft SQL Server Sink**を選択します。以下のページが表示されます。

   ![Topic selection](/media/integrate/topic-selection.png)

3. SQL Serverにレプリケートしたいトピックを選択し、次のページに移動します。

   ![Authentication](/media/integrate/authentication.png)

4. 接続情報と認証情報を入力し、次のページに移動します。

5. **Configuration**ページで、以下のフィールドを構成し、**Continue**をクリックします。

   | フィールド                           | 値           |
   | :------------------------------ | :---------- |
   | Input Kafka record value format | AVRO        |
   | Insert mode                     | UPSERT      |
   | Auto create table               | true        |
   | Auto add columns                | true        |
   | PK mode                         | record\_key |
   | Input Kafka record key format   | AVRO        |
   | Delete on null                  | true        |

6. 構成後、**Continue**をクリックします。コネクタのステータスが**Running**になるまで待ちます。これには数分かかる場合があります。

   ![Results](/media/integrate/results.png)

7. SQL Serverに接続し、データを確認します。前述の図に示されているように、増分データがSQL Serverにレプリケートされていることがわかります。SQL Serverとのデータ統合が完了しました。
