---
title: Connect to TiDB with MySQL Workbench
summary: Learn how to connect to TiDB using MySQL Workbench.
---

# MySQL Workbenchを使用してTiDBに接続する {#connect-to-tidb-with-mysql-workbench}

TiDBはMySQL互換のデータベースであり、[MySQL Workbench](https://www.mysql.com/products/workbench/)はMySQLデータベースユーザー向けのGUIツールセットです。

> **警告:**
>
> -   TiDBがMySQL互換であるため、MySQL Workbenchを使用してTiDBに接続することはできますが、MySQL WorkbenchはTiDBを完全にサポートしていません。TiDBをMySQLとして扱うため、使用中に問題が発生する可能性があります。
> -   [DataGrip](/develop/dev-guide-gui-datagrip.md)、[DBeaver](/develop/dev-guide-gui-dbeaver.md)、および[VS Code SQLTools](/develop/dev-guide-gui-vscode-sqltools.md)など、TiDBを公式にサポートする他のGUIツールを使用することをお勧めします。TiDBが完全にサポートするGUIツールの完全なリストについては、[TiDBがサポートするサードパーティツール](/develop/dev-guide-third-party-support.md#gui)を参照してください。

このチュートリアルでは、MySQL Workbenchを使用してTiDBクラスターに接続する方法を学ぶことができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、以下が必要です:

-   [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) **8.0.31** またはそれ以降のバージョン。
-   TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターを持っていない場合は、以下のように作成できます:**

-   (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
-   [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターを持っていない場合は、以下のように作成できます:**

-   (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
-   [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

TiDBのデプロイオプションに応じてTiDBクラスターに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1.  [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスターの名前をクリックして概要ページに移動します。

2.  右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成がオペレーティング環境に一致していることを確認します。

    -   **エンドポイントタイプ**が`Public`に設定されていること。
    -   **ブランチ**が`main`に設定されていること。
    -   **接続先**が`MySQL Workbench`に設定されていること。
    -   **オペレーティングシステム**が環境に一致していること。

4.  ランダムなパスワードを作成するために**パスワードを生成**をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合、元のパスワードを使用するか、**パスワードをリセット**をクリックして新しいパスワードを生成することができます。

5.  MySQL Workbenchを起動し、**MySQL Connections**タイトルの近くに\*\*+\*\*をクリックします。

    ![MySQL Workbench: 新しい接続を追加](/media/develop/navicat-add-new-connection.png)

6.  **新しい接続の設定**ダイアログで、以下の接続パラメータを構成します。

    -   **接続名**: この接続に意味のある名前を付けます。
    -   **ホスト名**: TiDB Cloud接続ダイアログから`HOST`パラメータを入力します。
    -   **ポート**: TiDB Cloud接続ダイアログから`PORT`パラメータを入力します。
    -   **ユーザー名**: TiDB Cloud接続ダイアログから`USERNAME`パラメータを入力します。
    -   **パスワード**: **キーチェーンに保存...**をクリックし、TiDB Serverlessクラスターのパスワードを入力し、パスワードを保存するために**OK**をクリックします。

        ![MySQL Workbench: TiDB Serverlessのパスワードをキーチェーンに保存](/media/develop/mysql-workbench-store-password-in-keychain.png)

    以下の図は接続パラメータの例を示しています:

    ![MySQL Workbench: TiDB Serverlessの接続設定を構成](/media/develop/mysql-workbench-connection-config-serverless-parameters.png)

7.  TiDB Serverlessクラスターへの接続を検証するために**接続をテスト**をクリックします。

8.  接続テストが成功した場合、**MySQL接続が正常に確立されました**メッセージが表示されます。接続構成を保存するために**OK**をクリックします。

</div>
<div label="TiDB Dedicated">

1.  [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスターの名前をクリックして概要ページに移動します。

2.  右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3.  **どこからでもアクセスを許可**をクリックします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4.  MySQL Workbenchを起動し、**MySQL Connections**タイトルの近くに\*\*+\*\*をクリックします。

    ![MySQL Workbench: 新しい接続を追加](/media/develop/navicat-add-new-connection.png)

5.  **新しい接続の設定**ダイアログで、以下の接続パラメータを構成します。

    -   **接続名**: この接続に意味のある名前を付けます。
    -   **ホスト名**: TiDB Cloud接続ダイアログから`host`パラメータを入力します。
    -   **ポート**: TiDB Cloud接続ダイアログから`port`パラメータを入力します。
    -   **ユーザー名**: TiDB Cloud接続ダイアログから`user`パラメータを入力します。
    -   **パスワード**: **キーチェーンに保存...**をクリックし、TiDB Dedicatedクラスターのパスワードを入力し、パスワードを保存するために**OK**をクリックします。

        ![MySQL Workbench: TiDB Dedicatedのパスワードをキーチェーンに保存](/media/develop/mysql-workbench-store-dedicated-password-in-keychain.png)

    以下の図は接続パラメータの例を示しています:

    ![MySQL Workbench: TiDB Dedicatedの接続設定を構成](/media/develop/mysql-workbench-connection-config-dedicated-parameters.png)

6.  TiDB Dedicatedクラスターへの接続を検証するために**接続をテスト**をクリックします。

7.  接続テストが成功した場合、**MySQL接続が正常に確立されました**メッセージが表示されます。接続構成を保存するために**OK**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1.  MySQL Workbenchを起動し、**MySQL Connections**タイトルの近くの\*\*+\*\*をクリックします。

    ![MySQL Workbench: 新しい接続を追加](/media/develop/navicat-add-new-connection.png)

2.  **新しい接続の設定**ダイアログで、以下の接続パラメータを設定します:

    -   **接続名**: この接続に意味のある名前を付けます。
    -   **ホスト名**: TiDB Self-HostedクラスターのIPアドレスまたはドメイン名を入力します。
    -   **ポート**: TiDB Self-Hostedクラスターのポート番号を入力します。
    -   **ユーザー名**: TiDBに接続するために使用するユーザー名を入力します。
    -   **パスワード**: **Keychainに保存...**をクリックし、TiDBクラスターに接続するためのパスワードを入力し、その後**OK**をクリックしてパスワードを保存します。

        ![MySQL Workbench: TiDB Self-HostedのパスワードをKeychainに保存](/media/develop/mysql-workbench-store-self-hosted-password-in-keychain.png)

    以下の図は、接続パラメータの例を示しています:

    ![MySQL Workbench: TiDB Self-Hostedの接続設定を構成する](/media/develop/mysql-workbench-connection-config-self-hosted-parameters.png)

3.  **接続をテスト**をクリックして、TiDB Self-Hostedクラスターへの接続を検証します。

4.  接続テストが成功した場合、**MySQL接続が正常に確立されました**メッセージが表示されます。接続構成を保存するには**OK**をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

-   [MySQL Workbenchのドキュメント](https://dev.mysql.com/doc/workbench/en/)からMySQL Workbenchのさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例: [データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
-   [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/tidb-cloud/tidb-cloud-support.md)してください。

</CustomContent>
