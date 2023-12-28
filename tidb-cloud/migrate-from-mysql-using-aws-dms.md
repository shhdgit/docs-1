---
title: Migrate from MySQL-Compatible Databases to TiDB Cloud Using AWS DMS
summary: Learn how to migrate data from MySQL-compatible databases to TiDB Cloud using AWS Database Migration Service (AWS DMS).
---

# MySQL互換データベースからTiDB Cloudへの移行方法（AWS DMSを使用） {#migrate-from-mysql-compatible-databases-to-tidb-cloud-using-aws-dms}

異種データベース（PostgreSQL、Oracle、SQL Serverなど）をTiDB Cloudに移行する場合、AWS Database Migration Service（AWS DMS）を使用することをお勧めします。

AWS DMSは、リレーショナルデータベース、データウェアハウス、NoSQLデータベース、その他のデータストアを簡単に移行できるクラウドサービスです。AWS DMSを使用してデータをTiDB Cloudに移行できます。

このドキュメントでは、Amazon RDSを例として、AWS DMSを使用してデータをTiDB Cloudに移行する方法を説明します。この手順は、自己ホスト型のMySQLデータベースやAmazon AuroraからTiDB Cloudにデータを移行する場合にも適用されます。

この例では、データソースはAmazon RDSであり、データの宛先はTiDB CloudのTiDB Dedicatedクラスターです。上流および下流のデータベースは同じリージョンにあります。

## 前提条件 {#prerequisites}

移行を開始する前に、次の内容を読んでおく必要があります。

- ソースデータベースがAmazon RDSまたはAmazon Auroraの場合、`binlog_format`パラメータを`ROW`に設定する必要があります。データベースがデフォルトのパラメータグループを使用している場合、`binlog_format`パラメータはデフォルトで`MIXED`になり、変更できません。この場合、`newset`などの新しいパラメータグループを[作成](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.Prerequisites.html#CHAP_GettingStarted.Prerequisites.params)し、その`binlog_format`を`ROW`に設定する必要があります。その後、[デフォルトのパラメータグループを変更](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithDBInstanceParamGroups.html#USER_WorkingWithParamGroups.Modifying)して`newset`を使用します。パラメータグループを変更すると、データベースが再起動されます。
- ソースデータベースがTiDBと互換性のある照合順序を使用していることを確認してください。TiDBのutf8mb4文字セットのデフォルトの照合順序は`utf8mb4_bin`です。しかし、MySQL 8.0では、デフォルトの照合順序は`utf8mb4_0900_ai_ci`です。上流のMySQLがデフォルトの照合順序を使用している場合、TiDBは`utf8mb4_0900_ai_ci`と互換性がないため、AWS DMSはターゲットテーブルをTiDBに作成できず、データを移行できません。この問題を解決するには、移行前にソースデータベースの照合順序を`utf8mb4_bin`に変更する必要があります。TiDBでサポートされている文字セットと照合順序の完全なリストについては、[文字セットと照合順序](https://docs.pingcap.com/tidb/stable/character-set-and-collation)を参照してください。
- TiDBには、デフォルトで次のシステムデータベースが含まれています：`INFORMATION_SCHEMA`、`PERFORMANCE_SCHEMA`、`mysql`、`sys`、および`test`。AWS DMSの移行タスクを作成するときは、これらのシステムデータベースをフィルタリングして、デフォルトの`%`を使用して移行オブジェクトを選択する代わりに、特定のデータベース名とテーブル名を入力する必要があります。そうしないと、AWS DMSはソースデータベースからこれらのシステムデータベースをターゲットのTiDBに移行しようとし、タスクが失敗します。この問題を回避するために、特定のデータベース名とテーブル名を入力することをお勧めします。
- AWS DMSのパブリックおよびプライベートネットワークIPアドレスを、ソースおよびターゲットのデータベースのIPアクセスリストに追加してください。そうしないと、ネットワーク接続が一部のシナリオで失敗する可能性があります。
- [VPCピアリング](/tidb-cloud/set-up-vpc-peering-connections.md#set-up-vpc-peering-on-aws)または[プライベートエンドポイント接続](/tidb-cloud/set-up-private-endpoint-connections.md)を使用して、AWS DMSとTiDBクラスターを接続してください。
- AWS DMSとTiDBクラスターで同じリージョンを使用することをお勧めします。これにより、データ書き込みのパフォーマンスが向上します。
- AWS DMS `dms.t3.large`（2 vCPU、8 GiBメモリ）またはそれ以上のインスタンスクラスを使用することをお勧めします。小さいインスタンスクラスを使用すると、メモリ不足（OOM）エラーが発生する可能性があります。
- AWS DMSは、ターゲットデータベースに`awsdms_control`データベースを自動的に作成します。

## 制限事項 {#limitation}

- AWS DMSは`DROP TABLE`のレプリケーションをサポートしていません。
- AWS DMSは、テーブルの作成や主キーの作成など、基本的なスキーマ移行をサポートしています。ただし、AWS DMSは自動的に二次インデックス、外部キー、またはユーザーアカウントをTiDB Cloudに作成しません。必要に応じて、これらのオブジェクトを手動でTiDBに作成する必要があります。詳細については、[AWS Database Migration Serviceの移行計画](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.html#CHAP_SettingUp.MigrationPlanning)を参照してください。

## ステップ1. AWS DMSレプリケーションインスタンスを作成する {#step-1-create-an-aws-dms-replication-instance}

1. AWS DMSコンソールの[レプリケーションインスタンス](https://console.aws.amazon.com/dms/v2/home#replicationInstances)ページに移動し、対応するリージョンに切り替えます。AWS DMSはTiDB Cloudと同じリージョンを使用することをお勧めします。このドキュメントでは、上流および下流のデータベースとDMSインスタンスはすべて**us-west-2**リージョンにあります。

2. **レプリケーションインスタンスの作成**をクリックします。

   ![レプリケーションインスタンスの作成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-create-instance.png)

3. インスタンス名、ARN、および説明を入力します。

4. インスタンスの構成を入力します：
   - **インスタンスクラス**：適切なインスタンスクラスを選択します。パフォーマンスを向上させるために、`dms.t3.large`またはそれ以上のインスタンスクラスを使用することをお勧めします。
   - **エンジンバージョン**：デフォルトの構成を使用します。
   - **マルチAZ**：ビジネスニーズに基づいて**Single-AZ**または**Multi-AZ**を選択します。

5. \*\*割り当てられたストレージ（GiB）\*\*フィールドでストレージを構成します。デフォルトの構成を使用します。

6. 接続とセキュリティを構成します。
   - **ネットワークタイプ - 新規**：**IPv4**を選択します。
   - **IPv4用の仮想プライベートクラウド（VPC）**：必要なVPCを選択します。ネットワーク構成を簡略化するために、上流データベースと同じVPCを使用することをお勧めします。
   - **レプリケーションサブネットグループ**：レプリケーションインスタンス用のサブネットグループを選択します。
   - **パブリックアクセス可能**：デフォルトの構成を使用します。

7. 必要に応じて**詳細設定**、**メンテナンス**、および**タグ**を構成します。**レプリケーションインスタンスの作成**をクリックして、インスタンスの作成を完了します。

## ステップ2. ソースデータベースエンドポイントの作成 {#step-2-create-the-source-database-endpoint}

1. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、作成したばかりのレプリケーションインスタンスをクリックします。次のスクリーンショットに示すように、パブリックおよびプライベートネットワークのIPアドレスをコピーします。

   ![パブリックおよびプライベートネットワークのIPアドレスをコピーする](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-copy-ip.png)

2. Amazon RDSのセキュリティグループルールを構成します。この例では、AWS DMSインスタンスのパブリックおよびプライベートIPアドレスをセキュリティグループに追加します。

   ![セキュリティグループルールを構成する](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-rules.png)

3. **エンドポイントの作成**をクリックして、ソースデータベースエンドポイントを作成します。

   ![エンドポイントの作成をクリックする](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-endpoint.png)

4. この例では、**RDS DBインスタンスを選択**をクリックし、ソースRDSインスタンスを選択します。ソースデータベースが自己ホストされたMySQLの場合は、このステップをスキップし、次のステップで情報を入力します。

   ![RDS DBインスタンスを選択する](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-select-rds.png)

5. 次の情報を入力します：

   - **エンドポイント識別子**：ソースエンドポイントのラベルを作成し、後続のタスク構成で識別できるようにします。
   - **説明的なAmazon Resource Name（ARN）-オプション**：デフォルトのDMS ARNのためのフレンドリーな名前を作成します。
   - **ソースエンジン**：**MySQL**を選択します。
   - **エンドポイントデータベースへのアクセス**：**アクセス情報を手動で提供**を選択します。
   - **サーバー名**：データプロバイダーのデータサーバーの名前を入力します。データベースコンソールからコピーできます。上流がAmazon RDSまたはAmazon Auroraの場合、名前は自動的に入力されます。ドメイン名がない自己ホストされたMySQLの場合は、IPアドレスを入力できます。
   - ソースデータベースの**ポート**、**ユーザー名**、および**パスワード**を入力します。
   - **Secure Socket Layer（SSL）モード**：必要に応じてSSLモードを有効にできます。

   ![エンドポイントの構成を入力する](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-endpoint-config.png)

6. **エンドポイント設定**、**KMSキー**、および**タグ**にはデフォルト値を使用します。**エンドポイント接続のテスト（オプション）**セクションでは、ネットワーク構成を簡略化するために、ソースデータベースと同じVPCを選択することをお勧めします。対応するレプリケーションインスタンスを選択し、**テストを実行**をクリックします。ステータスは**成功**である必要があります。

7. **エンドポイントの作成**をクリックします。

   ![エンドポイントの作成をクリックする](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-connection.png)

## ステップ3. ターゲットデータベースエンドポイントの作成 {#step-3-create-the-target-database-endpoint}

1. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、作成したレプリケーションインスタンスをクリックします。次のスクリーンショットに示すように、パブリックおよびプライベートネットワークのIPアドレスをコピーします。

   ![パブリックおよびプライベートネットワークのIPアドレスをコピーする](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-copy-ip.png)

2. TiDB Cloudコンソールで、[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックし、右上隅の**接続**をクリックして、TiDB Cloudデータベース接続情報を取得します。

3. ダイアログの**ステップ1：トラフィックフィルタを作成**の下で、**編集**をクリックし、AWS DMSコンソールからコピーしたパブリックおよびプライベートネットワークのIPアドレスを入力し、**フィルタを更新**をクリックします。同時に、AWS DMSがTiDBクラスタに接続できない可能性があるため、TiDBクラスタのトラフィックフィルタにAWS DMSレプリケーションインスタンスのパブリックIPアドレスとプライベートIPアドレスを追加することをお勧めします。

4. **CA証明書をダウンロード**をクリックして、CA証明書をダウンロードします。ダイアログの**ステップ3：SQLクライアントで接続**の下で、接続文字列の`-u`、`-h`、および`-P`情報を後で使用するためにメモしておきます。

5. ダイアログの**VPCピアリング**タブをクリックし、**ステップ1：VPCを設定**の下で**追加**をクリックして、TiDBクラスタとAWS DMSの間にVPCピアリング接続を作成します。

6. 対応する情報を設定します。[VPCピアリング接続の設定](/tidb-cloud/set-up-vpc-peering-connections.md)を参照してください。

7. TiDBクラスタのターゲットエンドポイントを設定します。

   - **エンドポイントタイプ**：**ターゲットエンドポイント**を選択します。
   - **エンドポイント識別子**：エンドポイントの名前を入力します。
   - **説明的なAmazonリソース名（ARN）-オプション**：デフォルトのDMS ARNのためのフレンドリーな名前を作成します。
   - **ターゲットエンジン**：**MySQL**を選択します。

   ![ターゲットエンドポイントを設定する](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-target-endpoint.png)

8. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、**エンドポイントを作成**をクリックして、ターゲットデータベースエンドポイントを作成し、次の情報を設定します。

   - **サーバー名**：TiDBクラスタのホスト名を入力します。これは、記録した`-h`情報です。
   - **ポート**：TiDBクラスタのポートを入力します。これは、記録した`-P`情報です。TiDBクラスタのデフォルトポートは4000です。
   - **ユーザー名**：TiDBクラスタのユーザー名を入力します。これは、記録した`-u`情報です。
   - **パスワード**：TiDBクラスタのパスワードを入力します。
   - **Secure Socket Layer（SSL）モード**：**Verify-ca**を選択します。
   - **新しいCA証明書を追加**をクリックして、前のステップでTiDB CloudコンソールからダウンロードしたCAファイルをインポートします。

   ![ターゲットエンドポイント情報を入力する](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-target-endpoint2.png)

9. CAファイルをインポートします。

   ![CAをアップロードする](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-upload-ca.png)

10. **エンドポイント設定**、**KMSキー**、および**タグ**のデフォルト値を使用します。**エンドポイント接続をテスト（オプション）**セクションで、ソースデータベースと同じVPCを選択します。対応するレプリケーションインスタンスを選択し、**テストを実行**をクリックします。ステータスは**成功**である必要があります。

11. **エンドポイントを作成**をクリックします。

    ![エンドポイントを作成する](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-target-endpoint3.png)

## ステップ4. データベース移行タスクを作成する {#step-4-create-a-database-migration-task}

1. AWS DMSコンソールに移動し、[データ移行タスク](https://console.aws.amazon.com/dms/v2/home#tasks)ページに移動します。リージョンを切り替え、ウィンドウの右上隅にある**タスクの作成**をクリックします。

   ![タスクの作成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-create-task.png)

2. 次の情報を設定します：

   - **タスク識別子**：タスクの名前を入力します。覚えやすい名前を使用することをお勧めします。
   - **説明的なAmazon Resource Name (ARN) - オプション**：デフォルトのDMS ARNのためのフレンドリーな名前を作成します。
   - **レプリケーションインスタンス**：作成したAWS DMSインスタンスを選択します。
   - **ソースデータベースエンドポイント**：作成したソースデータベースエンドポイントを選択します。
   - **ターゲットデータベースエンドポイント**：作成したターゲットデータベースエンドポイントを選択します。
   - **移行タイプ**：必要に応じて移行タイプを選択します。この例では、**既存のデータを移行し、継続的な変更をレプリケートする**を選択します。

   ![タスクの設定](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-task-config.png)

3. 次の情報を設定します：

   - **編集モード**：**ウィザード**を選択します。
   - **ソーストランザクションのカスタムCDC停止モード**：デフォルトの設定を使用します。
   - **ターゲットテーブルの準備モード**：**何もしない**または必要に応じて他のオプションを選択します。この例では、**何もしない**を選択します。
   - **完全なロードが完了したらタスクを停止する**：デフォルトの設定を使用します。
   - **レプリケーションでLOBカラムを含める**：**制限付きLOBモード**を選択します。
   - **(KB)の最大LOBサイズ**：デフォルト値**32**を使用します。
   - **検証をオンにする**：必要に応じて選択します。
   - **タスクログ**：将来のトラブルシューティングのために**CloudWatchログをオンにする**を選択します。関連する設定にはデフォルト値を使用します。

   ![タスクの設定](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-task-settings.png)

4. **テーブルマッピング**セクションで、移行するデータベースを指定します。

   スキーマ名はAmazon RDSインスタンスのデータベース名です。**ソース名**のデフォルト値は「%」で、Amazon RDSのすべてのデータベースがTiDBに移行されます。これにより、Amazon RDSのシステムデータベースである`mysql`や`sys`がTiDBクラスターに移行され、タスクが失敗する可能性があります。そのため、特定のデータベース名を入力するか、すべてのシステムデータベースをフィルタリングすることをお勧めします。例えば、次のスクリーンショットの設定に従って、`franktest`という名前のデータベースとそのデータベースのすべてのテーブルのみが移行されます。

   ![テーブルマッピング](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-table-mappings.png)

5. 右下の**タスクの作成**をクリックします。

6. [データ移行タスク](https://console.aws.amazon.com/dms/v2/home#tasks)ページに戻り、リージョンを切り替えます。タスクのステータスと進捗状況が表示されます。

   ![タスクのステータス](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-task-status.png)

移行中に問題や失敗が発生した場合は、[CloudWatch](https://console.aws.amazon.com/cloudwatch/home)でログ情報を確認して問題をトラブルシューティングできます。

![トラブルシューティング](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-troubleshooting.png)

## 関連情報 {#see-also}

- MySQL互換のデータベース（Aurora MySQLやAmazon Relational Database Service（RDS）など）からTiDB Cloudに移行する場合は、[TiDB Cloudでのデータ移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を使用することをお勧めします。

- Amazon RDS for OracleからTiDB ServerlessにAWS DMSを使用して移行する場合は、[Amazon RDS for OracleからTiDB ServerlessにAWS DMSを使用して移行する](/tidb-cloud/migrate-from-oracle-using-aws-dms.md)を参照してください。
