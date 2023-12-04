---
title: Connect to TiDB with Navicat
summary: Learn how to connect to TiDB using Navicat.
---

# Navicatを使用してTiDBに接続する {#connect-to-tidb-with-navicat}

TiDBはMySQL互換のデータベースであり、[Navicat](https://www.navicat.com)はデータベースユーザー向けのGUIツールセットです。このチュートリアルでは、[Navicat for MySQL](https://www.navicat.com/en/products/navicat-for-mysql)を使用してTiDBに接続します。

> **注意:**
>
> -   NavicatはMySQL互換性を持つためTiDBに接続することができますが、TiDBを完全にサポートしていません。TiDBをMySQLとして扱うため、使用中にいくつかの問題が発生する可能性があります。[Navicatユーザー管理の互換性](https://github.com/pingcap/tidb/issues/45154)に関する既知の問題があります。NavicatとTiDBの互換性に関する詳細な問題については、[TiDB GitHubの問題ページ](https://github.com/pingcap/tidb/issues?q=is%3Aissue+navicat+is%3Aopen)を参照してください。
> -   TiDBを公式にサポートしている他のGUIツールを使用することをお勧めします。例えば、[DataGrip](/develop/dev-guide-gui-datagrip.md)、[DBeaver](/develop/dev-guide-gui-dbeaver.md)、[VS Code SQLTools](/develop/dev-guide-gui-vscode-sqltools.md)などです。TiDBが完全にサポートしているGUIツールの完全なリストについては、[TiDBがサポートするサードパーティツール](/develop/dev-guide-third-party-support.md#gui)を参照してください。

このチュートリアルでは、Navicatを使用してTiDBクラスターに接続する方法を学ぶことができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です:

-   [Navicat for MySQL](https://www.navicat.com/en/download/navicat-for-mysql)のバージョン**16.3.2**またはそれ以降。
-   有料のNavicat for MySQLアカウント。
-   TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターを持っていない場合は、次のように作成できます:**

-   (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
-   [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターを持っていない場合は、次のように作成できます:**

-   (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
-   [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

TiDBのデプロイオプションに応じてTiDBクラスターに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1.  [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスターの名前をクリックして概要ページに移動します。

2.  右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成がオペレーティング環境に合致していることを確認します。

    -   **エンドポイントタイプ** が `Public` に設定されていること。
    -   **接続方法** が `General` に設定されていること。
    -   **オペレーティングシステム** が環境に合致していること。

4.  ランダムなパスワードを作成するために**パスワードを作成**をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合、元のパスワードを使用するか、**パスワードをリセット**をクリックして新しいパスワードを生成することができます。

5.  Navicat for MySQL を起動し、左上隅の**接続**をクリックし、ドロップダウンリストから**MySQL**を選択します。

    ![Navicat: 新しい接続を追加](/media/develop/navicat-add-new-connection.jpg)

6.  \*\*新しい接続（MySQL）\*\*ダイアログで、以下の接続パラメータを構成します。

    -   **接続名**: この接続に意味のある名前を付けます。
    -   **ホスト**: TiDB Cloud 接続ダイアログから `host` パラメータを入力します。
    -   **ポート**: TiDB Cloud 接続ダイアログから `port` パラメータを入力します。
    -   **ユーザー名**: TiDB Cloud 接続ダイアログから `user` パラメータを入力します。
    -   **パスワード**: TiDB Serverless クラスターのパスワードを入力します。

    ![Navicat: TiDB Serverless の接続構成一般パネル](/media/develop/navicat-connection-config-serverless-general.png)

7.  **SSL** タブをクリックし、**SSL を使用**、**認証を使用**、**CA 証明書を検証する**のチェックボックスを選択します。その後、TiDB Cloud 接続ダイアログから `ssl_ca` ファイルを**CA 証明書**フィールドに選択します。

    ![Navicat: TiDB Serverless の接続構成 SSL パネル](/media/develop/navicat-connection-config-serverless-ssl.png)

8.  **接続をテスト**して、TiDB Serverless クラスターへの接続を検証します。

9.  接続テストが成功した場合、**接続が成功しました**メッセージが表示されます。接続構成を完了するために**保存**をクリックします。

</div>
<div label="TiDB Dedicated">

1.  [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスターの名前をクリックして概要ページに移動します。

2.  右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3.  **どこからでもアクセスを許可**をクリックします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated 標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4.  **CA 証明書をダウンロード**をクリックして、CA ファイルをダウンロードします。

5.  Navicat for MySQL を起動し、左上隅の**接続**をクリックし、ドロップダウンリストから**MySQL**を選択します。

    ![Navicat: 新しい接続を追加](/media/develop/navicat-add-new-connection.jpg)

6.  \*\*新しい接続（MySQL）\*\*ダイアログで、以下の接続パラメータを構成します。

    -   **接続名**: この接続に意味のある名前を付けます。
    -   **ホスト**: TiDB Cloud 接続ダイアログから `host` パラメータを入力します。
    -   **ポート**: TiDB Cloud 接続ダイアログから `port` パラメータを入力します。
    -   **ユーザー名**: TiDB Cloud 接続ダイアログから `user` パラメータを入力します。
    -   **パスワード**: TiDB Dedicated クラスターのパスワードを入力します。

    ![Navicat: TiDB Dedicated の接続構成一般パネル](/media/develop/navicat-connection-config-dedicated-general.png)

7.  **SSL** タブをクリックし、**SSL を使用**、**認証を使用**、**CA 証明書を検証する**のチェックボックスを選択します。その後、ステップ 4 でダウンロードしたCAファイルを**CA 証明書**フィールドに選択します。

    ![Navicat: TiDB Dedicated の接続構成 SSL パネル](/media/develop/navicat-connection-config-dedicated-ssl.jpg)

8.  **接続をテスト**して、TiDB Dedicated クラスターへの接続を検証します。

9.  接続テストが成功した場合、**接続が成功しました**メッセージが表示されます。接続構成を完了するために**保存**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1.  Navicat for MySQLを起動し、左上隅の**Connection**をクリックし、ドロップダウンリストから**MySQL**を選択します。

    ![Navicat: 新しい接続を追加](/media/develop/navicat-add-new-connection.jpg)

2.  **New Connection (MySQL)** ダイアログで、以下の接続パラメータを設定します:

    -   **Connection Name**: この接続に意味のある名前を付けます。
    -   **Host**: TiDB Self-HostedクラスターのIPアドレスまたはドメイン名を入力します。
    -   **Port**: TiDB Self-Hostedクラスターのポート番号を入力します。
    -   **User Name**: TiDBに接続するためのユーザー名を入力します。
    -   **Password**: TiDBに接続するためのパスワードを入力します。

    ![Navicat: self-hosted TiDBの接続構成一般パネルを設定](/media/develop/navicat-connection-config-self-hosted-general.png)

3.  **Test Connection** をクリックして、TiDB Self-Hostedクラスターへの接続を検証します。

4.  接続テストが成功した場合、**Connection Successful** メッセージが表示されます。接続構成を完了するために **Save** をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

-   [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例: [データの挿入](/develop/dev-guide-insert-data.md), [データの更新](/develop/dev-guide-update-data.md), [データの削除](/develop/dev-guide-delete-data.md), [単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md), [トランザクション](/develop/dev-guide-transaction-overview.md), および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
-   [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/tidb-cloud/tidb-cloud-support.md)してください。

</CustomContent>
