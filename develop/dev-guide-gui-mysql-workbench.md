---
title: Connect to TiDB with MySQL Workbench
summary: Learn how to connect to TiDB using MySQL Workbench.
---

# MySQL Workbenchを使用してTiDBに接続する {#connect-to-tidb-with-mysql-workbench}

TiDBはMySQL互換のデータベースであり、[MySQL Workbench](https://www.mysql.com/products/workbench/)はMySQLデータベースユーザー向けのGUIツールセットです。

> **Warning:**
>
> - TiDBはMySQL互換性があるため、MySQL Workbenchを使用してTiDBに接続できますが、MySQL WorkbenchはTiDBを完全にサポートしていません。TiDBをMySQLとして扱うため、使用中にいくつかの問題が発生する可能性があります。
> - [DataGrip](/develop/dev-guide-gui-datagrip.md)、[DBeaver](/develop/dev-guide-gui-dbeaver.md)、および[VS Code SQLTools](/develop/dev-guide-gui-vscode-sqltools.md)など、TiDBを公式にサポートしている他のGUIツールを使用することをお勧めします。TiDBが完全にサポートしているGUIツールの完全なリストについては、[TiDBがサポートするサードパーティツール](/develop/dev-guide-third-party-support.md#gui)を参照してください。

このチュートリアルでは、MySQL Workbenchを使用してTiDBクラスターに接続する方法を学ぶことができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) **8.0.31** 以降のバージョン。
- TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターがない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターがない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

選択したTiDBデプロイオプションに応じてTiDBクラスターに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスターの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致することを確認してください。

   - **Endpoint Type**が`Public`に設定されていること。
   - **Branch**が`main`に設定されていること。
   - **Connect With**が`MySQL Workbench`に設定されていること。
   - **Operating System**が環境に一致していること。

4. **Generate Password**をクリックしてランダムなパスワードを作成します。

   > **Tip:**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成できます。

5. MySQL Workbenchを起動し、**MySQL Connections**タイトルの近くに\*\*+\*\*をクリックします。

   ![MySQL Workbench: 新しい接続を追加](/media/develop/navicat-add-new-connection.png)

6. **Setup New Connection**ダイアログで、次の接続パラメータを構成します。

   - **Connection Name**: この接続に意味のある名前を付けます。
   - **Hostname**: TiDB Cloud接続ダイアログから`HOST`パラメータを入力します。
   - **Port**: TiDB Cloud接続ダイアログから`PORT`パラメータを入力します。
   - **Username**: TiDB Cloud接続ダイアログから`USERNAME`パラメータを入力します。
   - **Password**: TiDB Serverlessクラスターのパスワードを入力し、**OK**をクリックしてパスワードを保存します。

     ![MySQL Workbench: TiDB Serverlessのパスワードをキーチェーンに保存](/media/develop/mysql-workbench-store-password-in-keychain.png)

   次の図は、接続パラメータの例を示しています。

   ![MySQL Workbench: TiDB Serverlessの接続設定を構成](/media/develop/mysql-workbench-connection-config-serverless-parameters.png)

7. **Test Connection**をクリックしてTiDB Serverlessクラスターへの接続を検証します。

8. 接続テストが成功した場合、**Successfully made the MySQL connection**メッセージが表示されます。接続構成を保存するには、**OK**をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスターの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. MySQL Workbenchを起動し、**MySQL Connections**タイトルの近くに\*\*+\*\*をクリックします。

   ![MySQL Workbench: 新しい接続を追加](/media/develop/navicat-add-new-connection.png)

5. **Setup New Connection**ダイアログで、次の接続パラメータを構成します。

   - **Connection Name**: この接続に意味のある名前を付けます。
   - **Hostname**: TiDB Cloud接続ダイアログから`host`パラメータを入力します。
   - **Port**: TiDB Cloud接続ダイアログから`port`パラメータを入力します。
   - **Username**: TiDB Cloud接続ダイアログから`user`パラメータを入力します。
   - **Password**: TiDB Dedicatedクラスターのパスワードを入力し、**OK**をクリックしてパスワードを保存します。

     ![MySQL Workbench: TiDB Dedicatedのパスワードをキーチェーンに保存](/media/develop/mysql-workbench-store-dedicated-password-in-keychain.png)

   次の図は、接続パラメータの例を示しています。

   ![MySQL Workbench: TiDB Dedicatedの接続設定を構成](/media/develop/mysql-workbench-connection-config-dedicated-parameters.png)

6. **Test Connection**をクリックしてTiDB Dedicatedクラスターへの接続を検証します。

7. 接続テストが成功した場合、**Successfully made the MySQL connection**メッセージが表示されます。接続構成を保存するには、**OK**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. MySQL Workbenchを起動し、**MySQL Connections**タイトルの近くに\*\*+\*\*をクリックします。

   ![MySQL Workbench: 新しい接続を追加](/media/develop/navicat-add-new-connection.png)

2. **Setup New Connection**ダイアログで、次の接続パラメータを構成します。

   - **Connection Name**: この接続に意味のある名前を付けます。
   - **Hostname**: TiDB Self-HostedクラスターのIPアドレスまたはドメイン名を入力します。
   - **Port**: TiDB Self-Hostedクラスターのポート番号を入力します。
   - **Username**: TiDBに接続するために使用するユーザー名を入力します。
   - **Password**: TiDBクラスターに接続するために使用するパスワードを入力し、**OK**をクリックしてパスワードを保存します。

     ![MySQL Workbench: TiDB Self-Hostedのパスワードをキーチェーンに保存](/media/develop/mysql-workbench-store-self-hosted-password-in-keychain.png)

   次の図は、接続パラメータの例を示しています。

   ![MySQL Workbench: TiDB Self-Hostedの接続設定を構成](/media/develop/mysql-workbench-connection-config-self-hosted-parameters.png)

3. **Test Connection**をクリックしてTiDB Self-Hostedクラスターへの接続を検証します。

4. 接続テストが成功した場合、**Successfully made the MySQL connection**メッセージが表示されます。接続構成を保存するには、**OK**をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

- [MySQL Workbenchのドキュメント](https://dev.mysql.com/doc/workbench/en/)からMySQL Workbenchのさらなる使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例: [データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

<CustomContent platform="TiDB Serverless">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="TiDB Cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/tidb-cloud/tidb-cloud-support.md)してください。

</CustomContent>
