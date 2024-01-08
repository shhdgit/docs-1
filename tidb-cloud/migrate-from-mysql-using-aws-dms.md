---
title: Migrate from MySQL-Compatible Databases to TiDB Cloud Using AWS DMS
summary: Learn how to migrate data from MySQL-compatible databases to TiDB Cloud using AWS Database Migration Service (AWS DMS).
---

# MySQL互換データベースをAWS DMSを使用してTiDB Cloudに移行する {#migrate-from-mysql-compatible-databases-to-tidb-cloud-using-aws-dms}

異種データベース（PostgreSQL、Oracle、SQL Serverなど）をTiDB Cloudに移行する場合、AWS Database Migration Service（AWS DMS）を使用することをお勧めします。

AWS DMSは、リレーショナルデータベース、データウェアハウス、NoSQLデータベース、およびその他のタイプのデータストアを簡単に移行できるクラウドサービスです。AWS DMSを使用してデータをTiDB Cloudに移行できます。

このドキュメントでは、Amazon RDSを使用して、AWS DMSを使用してTiDB Cloudにデータを移行する方法を示します。この手順は、セルフホスト型のMySQLデータベースまたはAmazon AuroraからTiDB Cloudにデータを移行する場合にも適用されます。

この例では、データソースはAmazon RDSであり、データの送信先はTiDB CloudのTiDB Dedicatedクラスターです。上流および下流のデータベースは同じリージョンにあります。

## 前提条件 {#prerequisites}

移行を開始する前に、次の内容を確認してください。

- ソースデータベースがAmazon RDSまたはAmazon Auroraの場合、`binlog_format`パラメータを`ROW`に設定する必要があります。データベースがデフォルトのパラメータグループを使用している場合、`binlog_format`パラメータはデフォルトで`MIXED`になっており、変更できません。この場合、`newset`などの新しいパラメータグループを作成し、その`binlog_format`を`ROW`に設定する必要があります。その後、デフォルトのパラメータグループを`newset`に変更します。パラメータグループを変更するとデータベースが再起動します。
- ソースデータベースがTiDBと互換性のある照合順序を使用していることを確認してください。TiDBのutf8mb4文字セットのデフォルト照合順序は`utf8mb4_bin`です。しかし、MySQL 8.0では、デフォルトの照合順序は`utf8mb4_0900_ai_ci`です。上流のMySQLがデフォルトの照合順序を使用している場合、TiDBが`utf8mb4_0900_ai_ci`と互換性がないため、AWS DMSはターゲットのテーブルをTiDBに作成できず、データを移行できません。この問題を解決するには、移行前にソースデータベースの照合順序を`utf8mb4_bin`に変更する必要があります。TiDBでサポートされている文字セットと照合順序の完全なリストについては、[Character Set and Collation](https://docs.pingcap.com/tidb/stable/character-set-and-collation)を参照してください。
- TiDBにはデフォルトで次のシステムデータベースが含まれています：`INFORMATION_SCHEMA`、`PERFORMANCE_SCHEMA`、`mysql`、`sys`、`test`。AWS DMS移行タスクを作成する際に、これらのシステムデータベースをフィルタリングして、デフォルトの`%`を使用せずに移行オブジェクトを選択する必要があります。そうしないと、AWS DMSはこれらのシステムデータベースをソースデータベースからターゲットのTiDBに移行しようとし、タスクが失敗します。この問題を回避するためには、特定のデータベース名とテーブル名を入力することをお勧めします。
- AWS DMSの公開およびプライベートネットワークIPアドレスをソースおよびターゲットデータベースのIPアクセスリストに追加してください。そうしないと、特定のシナリオでネットワーク接続が失敗する可能性があります。
- [VPC Peerings](/tidb-cloud/set-up-vpc-peering-connections.md#set-up-vpc-peering-on-aws)または[Private Endpoint connections](/tidb-cloud/set-up-private-endpoint-connections.md)を使用して、AWS DMSとTiDBクラスターを接続してください。
- AWS DMSとTiDBクラスターで同じリージョンを使用することをお勧めします。これにより、データ書き込みパフォーマンスが向上します。
- AWS DMS `dms.t3.large`（2 vCPUおよび8 GiBメモリ）またはそれ以上のインスタンスクラスを使用することをお勧めします。小さなインスタンスクラスはメモリ不足（OOM）エラーを引き起こす可能性があります。
- AWS DMSは、ターゲットデータベースに自動的に`awsdms_control`データベースを作成します。

## 制限事項 {#limitation}

- AWS DMSは`DROP TABLE`のレプリケーションをサポートしていません。
- AWS DMSは、テーブルとプライマリキーの作成を含む基本的なスキーマ移行をサポートしています。ただし、AWS DMSは、TiDB Cloudで二次インデックス、外部キー、またはユーザーアカウントを自動的に作成しません。必要に応じて、これらのオブジェクトを手動でTiDBに作成する必要があります。詳細については、[Migration planning for AWS Database Migration Service](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.html#CHAP_SettingUp.MigrationPlanning)を参照してください。

## ステップ1. AWS DMSレプリケーションインスタンスを作成する {#step-1-create-an-aws-dms-replication-instance}

1. AWS DMSコンソールの[Replication instances](https://console.aws.amazon.com/dms/v2/home#replicationInstances)ページに移動し、対応するリージョンに切り替えます。AWS DMSとTiDB Cloudで同じリージョンを使用することをお勧めします。このドキュメントでは、上流および下流のデータベースとDMSインスタンスはすべて**us-west-2**リージョンにあります。

2. **Create replication instance**をクリックします。

   ![Create replication instance](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-create-instance.png)

3. インスタンス名、ARN、および説明を入力します。

4. インスタンス構成を入力します：
   - **Instance class**: 適切なインスタンスクラスを選択します。パフォーマンスを向上させるために、`dms.t3.large`またはそれ以上のインスタンスクラスを使用することをお勧めします。
   - **Engine version**: デフォルトの構成を使用します。
   - **Multi-AZ**: ビジネスニーズに基づいて**Single-AZ**または**Multi-AZ**を選択します。

5. \*\*Allocated storage (GiB)\*\*フィールドでストレージを構成します。デフォルトの構成を使用します。

6. 接続とセキュリティを構成します。
   - **Network type - new**: **IPv4**を選択します。
   - **Virtual private cloud (VPC) for IPv4**: 必要なVPCを選択します。ネットワーク構成を簡素化するために、上流データベースと同じVPCを使用することをお勧めします。
   - **Replication subnet group**: レプリケーションインスタンス用のサブネットグループを選択します。
   - **Public accessible**: デフォルトの構成を使用します。

7. 必要に応じて**Advanced settings**、**Maintenance**、および**Tags**を構成します。インスタンスの作成を完了するには、**Create replication instance**をクリックします。

## ステップ2. ソースデータベースエンドポイントを作成する {#step-2-create-the-source-database-endpoint}

1. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、作成したばかりのレプリケーションインスタンスをクリックします。次のスクリーンショットに示すように、公開およびプライベートネットワークIPアドレスをコピーします。

   ![Copy the public and private network IP addresses](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-copy-ip.png)

2. Amazon RDSのセキュリティグループルールを構成します。この例では、AWS DMSインスタンスの公開およびプライベートIPアドレスをセキュリティグループに追加します。

   ![Configure the security group rules](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-rules.png)

3. **Create endpoint**をクリックして、ソースデータベースエンドポイントを作成します。

   ![Click Create endpoint](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-endpoint.png)

4. この例では、**Select RDS DB instance**をクリックして、ソースRDSインスタンスを選択します。ソースデータベースがセルフホスト型のMySQLの場合は、このステップをスキップし、次のステップで情報を入力します。

   ![Select RDS DB instance](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-select-rds.png)

5. 次の情報を構成します：

   - **Endpoint identifier**: ソースエンドポイントのラベルを作成して、後続のタスク構成で識別できるようにします。
   - **Descriptive Amazon Resource Name (ARN) - optional**: デフォルトのDMS ARNのフレンドリーな名前を作成します。
   - **Source engine**: **MySQL**を選択します。
   - **Access to endpoint database**: **Provide access information manually**を選択します。
   - **Server name**: データプロバイダーのデータサーバーの名前を入力します。データベースコンソールからコピーできます。上流がAmazon RDSまたはAmazon Auroraの場合、名前は自動的に入力されます。ドメイン名のないセルフホスト型のMySQLの場合は、IPアドレスを入力します。
   - ソースデータベースの**Port**、**Username**、および**Password**を入力します。
   - **Secure Socket Layer (SSL) mode**: 必要に応じてSSLモードを有効にします。

   ![Fill in the endpoint configurations](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-endpoint-config.png)

6. **Endpoint settings**、**KMS key**、および**Tags**にはデフォルト値を使用します。**Test endpoint connection (optional)**セクションでは、ネットワーク構成を簡素化するために、ソースデータベースと同じVPCを選択することをお勧めします。対応するレプリケーションインスタンスを選択し、**Run test**をクリックします。ステータスは**successful**である必要があります。

7. **Create endpoint**をクリックします。

   ![Click Create endpoint](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-connection.png)

## ステップ3. ターゲットデータベースエンドポイントを作成する {#step-3-create-the-target-database-endpoint}

1. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、作成したレプリケーションインスタンスをクリックします。次のスクリーンショットに示すように、パブリックおよびプライベートネットワークのIPアドレスをコピーします。

   ![パブリックおよびプライベートネットワークのIPアドレスをコピー](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-copy-ip.png)

2. TiDB Cloudコンソールで、[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックし、右上隅の**Connect**をクリックして、TiDB Cloudデータベース接続情報を取得します。

3. ダイアログの**Step 1: トラフィックフィルタの作成**の下で、**編集**をクリックし、AWS DMSコンソールからコピーしたパブリックおよびプライベートネットワークのIPアドレスを入力し、**Update Filter**をクリックします。同時に、AWS DMSのパブリックIPアドレスとプライベートIPアドレスをTiDBクラスタのトラフィックフィルタに追加することをお勧めします。そうしないと、いくつかのシナリオでAWS DMSがTiDBクラスタに接続できない場合があります。

4. **Download CA cert**をクリックしてCA証明書をダウンロードします。ダイアログの**Step 3: SQLクライアントで接続**の下で、後で使用するために接続文字列の`-u`、`-h`、および`-P`情報をメモしておきます。

5. ダイアログの**VPC Peering**タブをクリックし、**Step 1: VPCの設定**の下で**Add**をクリックして、TiDBクラスタとAWS DMSのためのVPC Peering接続を作成します。

6. 対応する情報を構成します。[VPC Peering接続の設定](/tidb-cloud/set-up-vpc-peering-connections.md)を参照してください。

7. TiDBクラスタのターゲットエンドポイントを構成します。

   - **エンドポイントタイプ**: **ターゲットエンドポイント**を選択します。
   - **エンドポイント識別子**: エンドポイントの名前を入力します。
   - **説明的なAmazonリソース名（ARN）- オプション**: デフォルトのDMS ARNのためのフレンドリーな名前を作成します。
   - **ターゲットエンジン**: **MySQL**を選択します。

   ![ターゲットエンドポイントの構成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-target-endpoint.png)

8. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、**Create endpoint**をクリックして、ターゲットデータベースエンドポイントを作成し、次の情報を構成します。

   - **サーバー名**: TiDBクラスタのホスト名を入力します。これは、記録した`-h`情報です。
   - **ポート**: TiDBクラスタのポートを入力します。これは、記録した`-P`情報です。TiDBクラスタのデフォルトポートは4000です。
   - **ユーザー名**: TiDBクラスタのユーザー名を入力します。これは、記録した`-u`情報です。
   - **パスワード**: TiDBクラスタのパスワードを入力します。
   - **Secure Socket Layer (SSL)モード**: **Verify-ca**を選択します。
   - **Add new CA certificate**をクリックして、前の手順でTiDB CloudコンソールからダウンロードしたCAファイルをインポートします。

   ![ターゲットエンドポイント情報の入力](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-target-endpoint2.png)

9. CAファイルをインポートします。

   ![CAをアップロード](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-upload-ca.png)

10. **エンドポイント設定**、**KMSキー**、および**タグ**のデフォルト値を使用します。**エンドポイント接続のテスト（オプション）**セクションで、ソースデータベースと同じVPCを選択します。対応するレプリケーションインスタンスを選択し、**Run test**をクリックします。ステータスは**successful**である必要があります。

11. **Create endpoint**をクリックします。

    ![Create endpointをクリック](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-target-endpoint3.png)

## ステップ4. データベース移行タスクを作成する {#step-4-create-a-database-migration-task}

1. AWS DMSコンソールで、[データ移行タスク](https://console.aws.amazon.com/dms/v2/home#tasks)ページに移動します。リージョンを切り替えます。その後、ウィンドウの右上隅にある**Create task**をクリックします。

   ![タスクの作成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-create-task.png)

2. 次の情報を構成します:

   - **タスク識別子**: タスクの名前を入力します。覚えやすい名前を使用することをお勧めします。
   - **説明的なAmazonリソース名（ARN）- オプション**: デフォルトのDMS ARNのためのフレンドリーな名前を作成します。
   - **レプリケーションインスタンス**: 作成したAWS DMSインスタンスを選択します。
   - **ソースデータベースエンドポイント**: 作成したソースデータベースエンドポイントを選択します。
   - **ターゲットデータベースエンドポイント**: 作成したターゲットデータベースエンドポイントを選択します。
   - **移行タイプ**: 必要に応じて移行タイプを選択します。この例では、**既存データの移行と進行中の変更のレプリケーション**を選択します。

   ![タスクの構成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-task-config.png)

3. 次の情報を構成します:

   - **編集モード**: **ウィザード**を選択します。
   - **ソーストランザクションのためのカスタムCDC停止モード**: デフォルトの設定を使用します。
   - **ターゲットテーブルの準備モード**: **何もしない**または必要な他のオプションを選択します。この例では、**何もしない**を選択します。
   - **フルロードが完了した後にタスクを停止**: デフォルトの設定を使用します。
   - **レプリケーションでLOB列を含める**: **制限付きLOBモード**を選択します。
   - **LOBの最大サイズ（KB）**: デフォルト値の**32**を使用します。
   - **検証をオンにする**: 必要に応じて選択します。
   - **タスクログ**: 将来のトラブルシューティングのために**CloudWatchログをオンにする**を選択します。関連する構成にはデフォルト設定を使用します。

   ![タスクの設定](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-task-settings.png)

4. **テーブルマッピング**セクションで、移行するデータベースを指定します。

   スキーマ名はAmazon RDSインスタンスのデータベース名です。**ソース名**のデフォルト値は「%」で、これはAmazon RDSのすべてのデータベースがTiDBに移行されることを意味します。これにより、Amazon RDSの`mysql`や`sys`などのシステムデータベースがTiDBクラスタに移行され、タスクの失敗を引き起こす可能性があります。したがって、特定のデータベース名を入力するか、すべてのシステムデータベースをフィルタリングすることをお勧めします。例えば、次のスクリーンショットの設定に従って、`franktest`という名前のデータベースとそのデータベース内のすべてのテーブルのみが移行されます。

   ![テーブルマッピング](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-table-mappings.png)

5. 右下隅にある**Create task**をクリックします。

6. [データ移行タスク](https://console.aws.amazon.com/dms/v2/home#tasks)ページに戻ります。リージョンを切り替えます。タスクのステータスと進捗状況を確認できます。

   ![タスクのステータス](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-task-status.png)

移行中に問題や失敗が発生した場合は、問題をトラブルシューティングするために[CloudWatch](https://console.aws.amazon.com/cloudwatch/home)でログ情報を確認できます。

![トラブルシューティング](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-troubleshooting.png)

## 関連項目 {#see-also}

- AWS DMSをTiDB ServerlessやTiDB Dedicatedに接続する方法について詳しく知りたい場合は、[AWS DMSをTiDB Cloudクラスタに接続](/tidb-cloud/tidb-cloud-connect-aws-dms.md)を参照してください。

- Aurora MySQLやAmazon Relational Database Service（RDS）などのMySQL互換データベースからTiDB Cloudに移行したい場合は、[TiDB Cloudでのデータ移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を使用することをお勧めします。

- Amazon RDS for OracleからAWS DMSを使用してTiDB Serverlessに移行したい場合は、[Amazon RDS for OracleからAWS DMSを使用してTiDB Serverlessに移行](/tidb-cloud/migrate-from-oracle-using-aws-dms.md)を参照してください。
