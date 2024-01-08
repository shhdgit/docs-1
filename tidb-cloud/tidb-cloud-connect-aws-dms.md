---
title: Connect AWS DMS to TiDB Cloud clusters
summary: Learn how to migrate data from or into TiDB Cloud using AWS Database Migration Service (AWS DMS).
---

# TiDB CloudクラスターにAWS DMSを接続する {#connect-aws-dms-to-tidb-cloud-clusters}

[AWS Database Migration Service (AWS DMS)](https://aws.amazon.com/dms/)は、リレーショナルデータベース、データウェアハウス、NoSQLデータベース、およびその他のタイプのデータストアを移行できるクラウドサービスです。AWS DMSを使用して、TiDB Cloudクラスターからデータを移行したり、TiDB Cloudクラスターにデータを移行したりできます。このドキュメントでは、AWS DMSをTiDB Cloudクラスターに接続する方法について説明します。

## 前提条件 {#prerequisites}

### 十分なアクセス権を持つAWSアカウント {#an-aws-account-with-enough-access}

DMS関連のリソースを管理するために十分なアクセス権を持つAWSアカウントが必要です。持っていない場合は、次のAWSドキュメントを参照してください。

- [AWSアカウントにサインアップ](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.SettingUp.html#sign-up-for-aws)
- [AWS Database Migration Serviceのアイデンティティとアクセス管理](https://docs.aws.amazon.com/dms/latest/userguide/security-iam.html)

### TiDB CloudアカウントとTiDBクラスター {#a-tidb-cloud-account-and-a-tidb-cluster}

TiDB CloudアカウントとTiDB ServerlessまたはTiDB Dedicatedクラスターを持っていることが期待されます。持っていない場合は、次のドキュメントを参照して作成してください。

- [TiDB Serverlessクラスターを作成](/tidb-cloud/create-tidb-cluster-serverless.md)
- [TiDB Dedicatedクラスターを作成](/tidb-cloud/create-tidb-cluster.md)

## ネットワークの構成 {#configure-network}

DMSリソースを作成する前に、DMSがTiDB Cloudクラスターと通信できるようにネットワークを適切に構成する必要があります。AWSに不慣れな場合は、AWSサポートに連絡してください。以下は、参考のためのいくつかの可能な構成を提供します。

<SimpleTab>

<div label="TiDB Serverless">

TiDB Serverlessの場合、クライアントはパブリックエンドポイントまたはプライベートエンドポイントを介してクラスターに接続できます。

- [パブリックエンドポイント経由でTiDB Serverlessクラスターに接続](/tidb-cloud/connect-via-standard-connection-serverless.md)するには、次のいずれかを行って、DMSレプリケーションインスタンスがインターネットにアクセスできるようにしてください。

  - レプリケーションインスタンスをパブリックサブネットにデプロイし、**Public accessible**を有効にします。詳細については、[インターネットアクセスの構成](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#vpc-igw-internet-access)を参照してください。

  - レプリケーションインスタンスをプライベートサブネットにデプロイし、プライベートサブネットのトラフィックをパブリックサブネットにルーティングします。この場合、少なくとも3つのサブネットが必要で、2つのプライベートサブネットと1つのパブリックサブネットが必要です。2つのプライベートサブネットは、レプリケーションインスタンスが存在するサブネットグループを形成します。その後、パブリックサブネットにNATゲートウェイを作成し、2つのプライベートサブネットのトラフィックをNATゲートウェイにルーティングする必要があります。詳細については、[プライベートサブネットからインターネットへのアクセス](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-scenarios.html#public-nat-internet-access)を参照してください。

- プライベートエンドポイント経由でTiDB Serverlessクラスターに接続するには、まず[プライベートエンドポイントを設定](/tidb-cloud/set-up-private-endpoint-connections-serverless.md)し、次にレプリケーションインスタンスをプライベートサブネットにデプロイします。

</div>

<div label="TiDB Dedicated">

TiDB Dedicatedの場合、クライアントはパブリックエンドポイント、プライベートエンドポイント、またはVPCピアリングを介してクラスターに接続できます。

- [パブリックエンドポイント経由でTiDB Dedicatedクラスターに接続](/tidb-cloud/connect-via-standard-connection.md)するには、次のいずれかを行って、DMSレプリケーションインスタンスがインターネットにアクセスできるようにしてください。さらに、レプリケーションインスタンスまたはNATゲートウェイのパブリックIPアドレスをクラスターの[IPアクセスリスト](/tidb-cloud/configure-ip-access-list.md)に追加する必要があります。

  - レプリケーションインスタンスをパブリックサブネットにデプロイし、**Public accessible**を有効にします。詳細については、[インターネットアクセスの構成](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#vpc-igw-internet-access)を参照してください。

  - レプリケーションインスタンスをプライベートサブネットにデプロイし、プライベートサブネットのトラフィックをパブリックサブネットにルーティングします。この場合、少なくとも3つのサブネットが必要で、2つのプライベートサブネットと1つのパブリックサブネットが必要です。2つのプライベートサブネットは、レプリケーションインスタンスが存在するサブネットグループを形成します。その後、パブリックサブネットにNATゲートウェイを作成し、2つのプライベートサブネットのトラフィックをNATゲートウェイにルーティングする必要があります。詳細については、[プライベートサブネットからインターネットへのアクセス](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-scenarios.html#public-nat-internet-access)を参照してください。

- プライベートエンドポイント経由でTiDB Dedicatedクラスターに接続するには、まず[プライベートエンドポイントを設定](/tidb-cloud/set-up-private-endpoint-connections.md)し、次にレプリケーションインスタンスをプライベートサブネットにデプロイします。

- VPCピアリング経由でTiDB Dedicatedクラスターに接続するには、まず[VPCピアリング接続を設定](/tidb-cloud/set-up-vpc-peering-connections.md)し、次にレプリケーションインスタンスをプライベートサブネットにデプロイします。

</div>
</SimpleTab>

## AWS DMSレプリケーションインスタンスの作成 {#create-an-aws-dms-replication-instance}

1. AWS DMSコンソールで、[**Replication instances**](https://console.aws.amazon.com/dms/v2/home#replicationInstances)ページに移動し、対応するリージョンに切り替えます。AWS DMSとTiDB Cloudで同じリージョンを使用することをお勧めします。

   ![レプリケーションインスタンスの作成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-connect-replication-instances.png)

2. **Create replication instance**をクリックします。

3. インスタンス名、ARN、および説明を入力します。

4. **Instance configuration**セクションで、インスタンスを構成します。
   - **Instance class**: 適切なインスタンスクラスを選択します。詳細については、[レプリケーションインスタンスタイプの選択](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_ReplicationInstance.Types.html)を参照してください。
   - **Engine version**: デフォルトの構成を維持します。
   - **High Availability**: ビジネスニーズに基づいて**Multi-AZ**または**Single-AZ**を選択します。

5. \*\*Allocated storage (GiB)\*\*フィールドでストレージを構成します。

6. 接続とセキュリティを構成します。ネットワーク構成については、[前のセクション](#configure-network)を参照してください。

   - **Network type - new**: **IPv4**を選択します。
   - **Virtual private cloud (VPC) for IPv4**: 必要なVPCを選択します。
   - **Replication subnet group**: レプリケーションインスタンス用のサブネットグループを選択します。
   - **Public accessible**: ネットワーク構成に基づいて設定します。

   ![接続とセキュリティ](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-connect-connectivity-security.png)

7. 必要に応じて**Advanced settings**、**Maintenance**、および**Tags**セクションを構成し、**Create replication instance**をクリックしてインスタンスの作成を完了します。

> **Note:**
>
> AWS DMSはサーバーレスレプリケーションもサポートしています。詳細な手順については、[サーバーレスレプリケーションの作成](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Serverless.Components.html#CHAP_Serverless.create)を参照してください。レプリケーションインスタンスとは異なり、AWS DMSサーバーレスレプリケーションには**Public accessible**オプションは提供されません。

## TiDB Cloud DMSエンドポイントの作成 {#create-tidb-cloud-dms-endpoints}

接続については、TiDB Cloudクラスターをソースまたはターゲットとして使用する手順は類似していますが、DMSにはソースとターゲットの異なるデータベース設定要件があります。詳細については、[MySQLをソースとして使用](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.MySQL.html)または[MySQLをターゲットとして使用](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.MySQL.html)を参照してください。ソースとしてTiDB Cloudクラスターを使用する場合、TiDBはMySQLのbinlogをサポートしていないため、既存のデータのみを**移行**できます。

1. AWS DMSコンソールで、[**エンドポイント**](https://console.aws.amazon.com/dms/v2/home#endpointList)ページに移動し、対応するリージョンに切り替えます。

   ![エンドポイントの作成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-connect-create-endpoint.png)

2. **エンドポイントの作成**をクリックして、ターゲットデータベースのエンドポイントを作成します。

3. **エンドポイントタイプ**セクションで、**ソースエンドポイント**または**ターゲットエンドポイント**を選択します。

4. **エンドポイント構成**セクションで、**エンドポイント識別子**とARNフィールドに入力します。その後、**ソースエンジン**または**ターゲットエンジン**として**MySQL**を選択します。

5. **エンドポイントデータベースへのアクセス**フィールドで、**アクセス情報を手動で提供**チェックボックスを選択し、次のようにクラスター情報を入力します。

    <SimpleTab>

    <div label="TiDB Serverless">

   - **サーバー名**: TiDB Serverlessクラスターの`HOST`。
   - **ポート**: TiDB Serverlessクラスターの`PORT`。
   - **ユーザー名**: マイグレーション用のTiDB Serverlessクラスターのユーザー。DMSの要件を満たしていることを確認してください。
   - **パスワード**: TiDB Serverlessクラスターユーザーのパスワード。
   - **セキュアソケットレイヤー（SSL）モード**: パブリックエンドポイント経由で接続する場合、トランスポートセキュリティを確保するためにモードを**verify-full**に設定することを強くお勧めします。プライベートエンドポイント経由で接続する場合、モードを**none**に設定できます。
   - (オプション) **CA証明書**: [ISRG Root X1証明書](https://letsencrypt.org/certs/isrgrootx1.pem)を使用してください。詳細については、[TiDB ServerlessへのTLS接続](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。

    </div>

    <div label="TiDB Dedicated">

   - **サーバー名**: TiDB Dedicatedクラスターの`HOST`。
   - **ポート**: TiDB Dedicatedクラスターの`PORT`。
   - **ユーザー名**: マイグレーション用のTiDB Dedicatedクラスターのユーザー。DMSの要件を満たしていることを確認してください。
   - **パスワード**: TiDB Dedicatedクラスターユーザーのパスワード。
   - **セキュアソケットレイヤー（SSL）モード**: パブリックエンドポイント経由で接続する場合、トランスポートセキュリティを確保するためにモードを**verify-full**に設定することを強くお勧めします。プライベートエンドポイント経由で接続する場合、モードを**none**に設定できます。
   - (オプション) **CA証明書**: [TLS connections to TiDB Dedicated](/tidb-cloud/tidb-cloud-tls-connect-to-dedicated.md)に従ってCA証明書を取得してください。

    </div>
    </SimpleTab>

   ![アクセス情報を手動で提供](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-connect-configure-endpoint.png)

6. **エンドポイントをターゲットエンドポイントとして作成**する場合は、**エンドポイント設定**セクションを展開し、**エンドポイント接続属性を使用**チェックボックスを選択し、**追加の接続属性**を`Initstmt=SET FOREIGN_KEY_CHECKS=0;`に設定します。

7. 必要に応じて**KMSキー**および**タグ**セクションを構成します。インスタンスの作成を完了するには、**エンドポイントの作成**をクリックしてください。"
