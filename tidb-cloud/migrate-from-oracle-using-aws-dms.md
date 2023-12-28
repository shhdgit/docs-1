---
title:  Migrate from Amazon RDS for Oracle to TiDB Cloud Using AWS DMS
summary: Learn how to migrate data from Amazon RDS for Oracle into TiDB Serverless using AWS Database Migration Service (AWS DMS).
---

# AWS DMSを使用してAmazon RDS for OracleからTiDB Cloudに移行する {#migrate-from-amazon-rds-for-oracle-to-tidb-cloud-using-aws-dms}

このドキュメントでは、AWS Database Migration Service（AWS DMS）を使用して、Amazon RDS for Oracleから[TiDB Serverless](https://tidbcloud.com/console/clusters/create-cluster)にデータを移行する手順の例を説明します。

TiDB CloudとAWS DMSについて詳しく学びたい場合は、次を参照してください。

- [TiDB Cloud](https://docs.pingcap.com/tidbcloud/)
- [TiDB Developer Guide](https://docs.pingcap.com/tidbcloud/dev-guide-overview)
- [AWS DMSドキュメント](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.html)

## AWS DMSを使用する理由 {#why-use-aws-dms}

AWS DMSは、リレーショナルデータベース、データウェアハウス、NoSQLデータベース、その他のタイプのデータストアを移行できるクラウドサービスです。

PostgreSQL、Oracle、SQL Serverなどの異種データベースからTiDB Cloudにデータを移行する場合は、AWS DMSを使用することをお勧めします。

## デプロイアーキテクチャ {#deployment-architecture}

大まかに言うと、次の手順に従います。

1. ソースのAmazon RDS for Oracleをセットアップします。
2. ターゲットの[TiDB Serverless](https://tidbcloud.com/console/clusters/create-cluster)をセットアップします。
3. AWS DMSを使用してデータ移行（フルロード）をセットアップします。

次の図は、高レベルのアーキテクチャを示しています。

![アーキテクチャ](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-0.png)

## 前提条件 {#prerequisites}

開始する前に、次の前提条件を読んでください。

- [AWS DMSの前提条件](/tidb-cloud/migrate-from-mysql-using-aws-dms.md#prerequisites)
- [AWS Cloudアカウント](https://aws.amazon.com)
- [TiDB Cloudアカウント](https://tidbcloud.com)
- [DBeaver](https://dbeaver.io/)

次に、AWS DMSを使用してAmazon RDS for OracleからTiDB Cloudにデータを移行する方法を学びます。

## ステップ1. VPCを作成する {#step-1-create-a-vpc}

[AWSコンソール](https://console.aws.amazon.com/vpc/home#vpcs:)にログインし、AWS VPCを作成します。後でこのVPCにOracle RDSとDMSインスタンスを作成する必要があります。

VPCを作成する方法については、[VPCの作成](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html#Create-VPC)を参照してください。

![VPCを作成する](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-1.png)

## ステップ2. Oracle DBインスタンスを作成する {#step-2-create-an-oracle-db-instance}

作成したVPCにOracle DBインスタンスを作成し、パスワードを覚えておき、パブリックアクセスを許可します。AWS Schema Conversion Toolを使用するには、パブリックアクセスを有効にする必要があります。本番環境でパブリックアクセスを許可することはお勧めしません。

Oracle DBインスタンスを作成する方法については、[Oracle DBインスタンスの作成とOracle DBインスタンス上のデータベースへの接続](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.Oracle.html)を参照してください。

![Oracle RDSを作成する](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-2.png)

## ステップ3. Oracleでテーブルデータを準備する {#step-3-prepare-the-table-data-in-oracle}

次のスクリプトを使用して、github\_eventsテーブルに10000行のデータを作成してポピュレートします。[GH Archive](https://gharchive.org/)からgithubイベントデータセットをダウンロードできます。10000行のデータが含まれています。次のSQLスクリプトを使用してOracleで実行します。

- [table\_schema\_oracle.sql](https://github.com/pingcap-inc/tidb-integration-script/blob/main/aws-dms/oracle_table_schema.sql)
- [oracle\_data.sql](https://github.com/pingcap-inc/tidb-integration-script/blob/main/aws-dms/oracle_data.sql)

SQLスクリプトの実行が完了したら、Oracleでデータを確認します。次の例では、[DBeaver](https://dbeaver.io/)を使用してデータをクエリします。

![Oracle RDSのデータ](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-3.png)

## ステップ4. TiDB Serverlessクラスタを作成する {#step-4-create-a-tidb-serverless-cluster}

1. [TiDB Cloudコンソール](https://tidbcloud.com/console/clusters)にログインします。

2. [TiDB Serverlessクラスターを作成します](/tidb-cloud/tidb-cloud-quickstart.md)。

3. [**クラスター**](https://tidbcloud.com/console/clusters)ページで、対象のクラスター名をクリックして、その概要ページに移動します。

4. 右上隅にある**接続**をクリックします。

5. **パスワードを生成**をクリックして、パスワードを生成し、生成されたパスワードをコピーします。

6. 好みの接続方法とオペレーティングシステムを選択し、表示された接続文字列を使用してクラスターに接続します。

## ステップ5. AWS DMSレプリケーションインスタンスを作成する {#step-5-create-an-aws-dms-replication-instance}

1. AWS DMSコンソールの[レプリケーションインスタンス](https://console.aws.amazon.com/dms/v2/home#replicationInstances)ページに移動し、対応するリージョンに切り替えます。

2. `dms.t3.large`を使用してVPC内にAWS DMSレプリケーションインスタンスを作成します。

   ![Create AWS DMS Instance](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-8.png)

## ステップ6. DMSエンドポイントを作成する {#step-6-create-dms-endpoints}

1. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、左側のペインで`Endpoints`メニューアイテムをクリックします。

2. OracleソースエンドポイントとTiDBターゲットエンドポイントを作成します。

   次のスクリーンショットは、ソースエンドポイントの構成を示しています。

   ![Create AWS DMS Source endpoint](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-9.png)

   次のスクリーンショットは、ターゲットエンドポイントの構成を示しています。

   ![Create AWS DMS Target endpoint](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-10.png)

## ステップ7. スキーマを移行する {#step-7-migrate-the-schema}

この例では、スキーマの定義が単純であるため、AWS DMSがスキーマを自動的に処理します。

AWS Schema Conversion Toolを使用してスキーマを移行することを決定した場合は、[AWS SCTのインストール](https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP_Installing.html#CHAP_Installing.Procedure)を参照してください。

詳細については、[AWS SCTを使用してソーススキーマをターゲットデータベースに移行する](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.SCT.html)を参照してください。

## ステップ8. データベース移行タスクを作成する {#step-8-create-a-database-migration-task}

1. AWS DMSコンソールで、[データ移行タスク](https://console.aws.amazon.com/dms/v2/home#tasks)ページに移動します。リージョンに切り替えます。その後、ウィンドウの右上隅にある**タスクを作成**をクリックします。

   ![Create task](/media/tidb-cloud/aws-dms-to-tidb-cloud-create-task.png)

2. データベース移行タスクを作成し、**選択ルール**を指定します。

   ![Create AWS DMS migration task](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-11.png)

   ![AWS DMS migration task selection rules](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-12.png)

3. タスクを作成し、開始し、タスクが完了するまで待ちます。

4. **テーブル統計**をクリックしてテーブルを確認します。スキーマ名は`ADMIN`です。

   ![Check AWS DMS migration task](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-13.png)

## ステップ9. ダウンストリームのTiDBクラスターでデータを確認する {#step-9-check-data-in-the-downstream-tidb-cluster}

[TiDB Serverlessクラスター](https://tidbcloud.com/console/clusters/create-cluster)に接続し、`admin.github_event`テーブルのデータを確認します。次のスクリーンショットに示すように、DMSは`github_events`テーブルと10000行のデータを正常に移行しました。

![Check Data In TiDB](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-14.png)

## 要約 {#summary}

AWS DMSを使用すると、このドキュメントの例に従って、上流のAWS RDSデータベースからデータを正常に移行できます。

移行中に問題や失敗が発生した場合は、[CloudWatch](https://console.aws.amazon.com/cloudwatch/home)でログ情報を確認して問題をトラブルシューティングできます。

![Troubleshooting](/media/tidb-cloud/aws-dms-to-tidb-cloud-troubleshooting.png)

## 関連情報 {#see-also}

- [AWS DMSを使用したMySQL互換データベースからの移行](/tidb-cloud/migrate-from-mysql-using-aws-dms.md)
