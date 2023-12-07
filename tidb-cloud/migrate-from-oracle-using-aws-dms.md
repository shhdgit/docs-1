---
title:  Migrate from Amazon RDS for Oracle to TiDB Cloud Using AWS DMS
summary: Learn how to migrate data from Amazon RDS for Oracle into TiDB Serverless using AWS Database Migration Service (AWS DMS).
---

# Amazon RDS for Oracle から TiDB Cloud への移行を AWS DMS を使用して行う方法 {#migrate-from-amazon-rds-for-oracle-to-tidb-cloud-using-aws-dms}

このドキュメントでは、Amazon RDS for Oracle から [TiDB Serverless](https://tidbcloud.com/console/clusters/create-cluster) へのデータ移行を AWS Database Migration Service (AWS DMS) を使用して段階的に説明します。

TiDB Cloud や AWS DMS について詳しく知りたい場合は、以下を参照してください:

-   [TiDB Cloud](https://docs.pingcap.com/tidbcloud/)
-   [TiDB Developer Guide](https://docs.pingcap.com/tidbcloud/dev-guide-overview)
-   [AWS DMS Documentation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.html)

## AWS DMS の利用理由 {#why-use-aws-dms}

AWS DMS は、リレーショナルデータベース、データウェアハウス、NoSQL データベース、およびその他のデータストアの移行を可能にするクラウドサービスです。

PostgreSQL、Oracle、SQL Server などの異種データベースから TiDB Cloud にデータを移行したい場合は、AWS DMS を使用することをお勧めします。

## デプロイメントアーキテクチャ {#deployment-architecture}

高レベルでは、以下の手順に従います:

1.  ソースとなる Amazon RDS for Oracle をセットアップします。
2.  ターゲットとなる [TiDB Serverless](https://tidbcloud.com/console/clusters/create-cluster) をセットアップします。
3.  AWS DMS を使用してデータ移行（フルロード）をセットアップします。

以下の図は、高レベルのアーキテクチャを示しています。

![Architecture](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-0.png)

## 前提条件 {#prerequisites}

始める前に、以下の前提条件を確認してください:

-   [AWS DMS の前提条件](/tidb-cloud/migrate-from-mysql-using-aws-dms.md#prerequisites)
-   [AWS クラウドアカウント](https://aws.amazon.com)
-   [TiDB Cloud アカウント](https://tidbcloud.com)
-   [DBeaver](https://dbeaver.io/)

次に、Amazon RDS for Oracle から TiDB Cloud へのデータ移行に AWS DMS を使用する方法を学びます。

## ステップ 1. VPC の作成 {#step-1-create-a-vpc}

[AWS コンソール](https://console.aws.amazon.com/vpc/home#vpcs:) にログインし、AWS VPC を作成します。後でこの VPC 内に Oracle RDS や DMS インスタンスを作成する必要があります。

VPC の作成方法については、[VPC の作成](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html#Create-VPC) を参照してください。

![Create VPC](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-1.png)

## ステップ 2. Oracle DB インスタンスの作成 {#step-2-create-an-oracle-db-instance}

作成した VPC 内で Oracle DB インスタンスを作成し、パスワードを覚えておく必要があります。また、パブリックアクセスを許可する必要があります。AWS Schema Conversion Tool を使用するためにパブリックアクセスを有効にする必要があります。なお、本番環境でのパブリックアクセスの許可は推奨されません。

Oracle DB インスタンスの作成方法については、[Oracle DB インスタンスの作成およびデータベースへの接続](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.Oracle.html) を参照してください。

![Create Oracle RDS](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-2.png)

## ステップ 3. Oracle でテーブルデータの準備 {#step-3-prepare-the-table-data-in-oracle}

以下のスクリプトを使用して、github\_events テーブルに10000行のデータを作成してポピュレートします。GitHub イベントのデータセットを使用し、[GH Archive](https://gharchive.org/) からダウンロードできます。10000行のデータが含まれています。以下の SQL スクリプトを使用して Oracle で実行してください。

-   [table\_schema\_oracle.sql](https://github.com/pingcap-inc/tidb-integration-script/blob/main/aws-dms/oracle_table_schema.sql)
-   [oracle\_data.sql](https://github.com/pingcap-inc/tidb-integration-script/blob/main/aws-dms/oracle_data.sql)

SQL スクリプトの実行が完了したら、Oracle のデータを確認してください。以下の例では、[DBeaver](https://dbeaver.io/) を使用してデータをクエリしています:

![Oracle RDS Data](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-3.png)

## ステップ 4. TiDB Serverless クラスターの作成 {#step-4-create-a-tidb-serverless-cluster}

1.  [TiDB Cloudコンソール](https://tidbcloud.com/console/clusters)にログインします。

2.  [TiDB Serverlessクラスター](/tidb-cloud/tidb-cloud-quickstart.md)を作成します。

3.  [**クラスター**](https://tidbcloud.com/console/clusters) ページで、対象のクラスター名をクリックして概要ページに移動します。

4.  右上隅にある **Connect** をクリックします。

5.  **Generate Password** をクリックしてパスワードを生成し、生成されたパスワードをコピーします。

6.  好みの接続方法とオペレーティングシステムを選択し、表示された接続文字列を使用してクラスターに接続します。

## ステップ5. AWS DMSレプリケーションインスタンスを作成する {#step-5-create-an-aws-dms-replication-instance}

1.  AWS DMSコンソールの[レプリケーションインスタンス](https://console.aws.amazon.com/dms/v2/home#replicationInstances) ページに移動し、対応するリージョンに切り替えます。

2.  VPC内で `dms.t3.large` を使用してAWS DMSレプリケーションインスタンスを作成します。

    ![AWS DMSインスタンスの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-8.png)

## ステップ6. DMSエンドポイントを作成する {#step-6-create-dms-endpoints}

1.  [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home) で、左ペインの `Endpoints` メニューアイテムをクリックします。

2.  OracleソースエンドポイントとTiDBターゲットエンドポイントを作成します。

    次のスクリーンショットは、ソースエンドポイントの構成を示しています。

    ![AWS DMSソースエンドポイントの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-9.png)

    次のスクリーンショットは、ターゲットエンドポイントの構成を示しています。

    ![AWS DMSターゲットエンドポイントの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-10.png)

## ステップ7. スキーマを移行する {#step-7-migrate-the-schema}

この例では、スキーマ定義が単純であるため、AWS DMSが自動的にスキーマを処理します。

AWS Schema Conversion Toolを使用してスキーマを移行する場合は、[AWS SCTのインストール](https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP_Installing.html#CHAP_Installing.Procedure)を参照してください。

詳細については、[AWS SCTを使用してソーススキーマをターゲットデータベースに移行する](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.SCT.html)を参照してください。

## ステップ8. データベース移行タスクを作成する {#step-8-create-a-database-migration-task}

1.  AWS DMSコンソールで、[データ移行タスク](https://console.aws.amazon.com/dms/v2/home#tasks) ページに移動します。リージョンを切り替えた後、ウィンドウの右上隅にある **Create task** をクリックします。

    ![タスクの作成](/media/tidb-cloud/aws-dms-to-tidb-cloud-create-task.png)

2.  データベース移行タスクを作成し、**Selection rules** を指定します。

    ![AWS DMS移行タスクの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-11.png)

    ![AWS DMS移行タスクの選択ルール](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-12.png)

3.  タスクを作成し、開始し、タスクが完了するのを待ちます。

4.  **Table statistics** をクリックしてテーブルを確認します。スキーマ名は `ADMIN` です。

    ![AWS DMS移行タスクの確認](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-13.png)

## ステップ9. 下流のTiDBクラスターでデータを確認する {#step-9-check-data-in-the-downstream-tidb-cluster}

[TiDB Serverlessクラスター](https://tidbcloud.com/console/clusters/create-cluster) に接続し、`admin.github_event` テーブルのデータを確認します。次のスクリーンショットに示すように、DMSは `github_events` テーブルと10000行のデータを正常に移行しました。

![TiDBでデータを確認](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-14.png)

## 要約 {#summary}

AWS DMSを使用すると、このドキュメントの例に従って、どのような上流AWS RDSデータベースからもデータを正常に移行できます。

移行中に問題や失敗が発生した場合は、問題をトラブルシューティングするために[CloudWatch](https://console.aws.amazon.com/cloudwatch/home)でログ情報を確認できます。

![トラブルシューティング](/media/tidb-cloud/aws-dms-to-tidb-cloud-troubleshooting.png)

## 関連項目 {#see-also}

-   [AWS DMSを使用したMySQL互換データベースからの移行](/tidb-cloud/migrate-from-mysql-using-aws-dms.md)
