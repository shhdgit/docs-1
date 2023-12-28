---
title: Connect to TiDB with Navicat
summary: Learn how to connect to TiDB using Navicat.
---

# Navicatを使用してTiDBに接続する {#connect-to-tidb-with-navicat}

TiDBはMySQL互換のデータベースであり、[Navicat](https://www.navicat.com)はデータベースユーザー向けのGUIツールセットです。このチュートリアルでは、[Navicat for MySQL](https://www.navicat.com/en/products/navicat-for-mysql)ツールを使用してTiDBに接続します。

> **注意:**
>
> - NavicatはMySQL互換であるため、TiDBに接続することができますが、TiDBを完全にサポートしていません。TiDBをMySQLとして扱うため、使用中にいくつかの問題が発生する可能性があります。[Navicatユーザー管理の互換性に関する既知の問題](https://github.com/pingcap/tidb/issues/45154)があります。NavicatとTiDBの互換性の問題については、[TiDB GitHubの問題ページ](https://github.com/pingcap/tidb/issues?q=is%3Aissue+navicat+is%3Aopen)を参照してください。
> - [DataGrip](/develop/dev-guide-gui-datagrip.md)、[DBeaver](/develop/dev-guide-gui-dbeaver.md)、[VS Code SQLTools](/develop/dev-guide-gui-vscode-sqltools.md)など、公式にTiDBをサポートしている他のGUIツールを使用することをお勧めします。TiDBが完全にサポートしているGUIツールの完全なリストについては、[TiDBがサポートするサードパーティツール](/develop/dev-guide-third-party-support.md#gui)を参照してください。

このチュートリアルでは、Navicatを使用してTiDBクラスターに接続する方法を学ぶことができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Navicat for MySQL](https://www.navicat.com/en/download/navicat-for-mysql) **16.3.2**以降のバージョン。
- Navicat for MySQLの有料アカウント。
- TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

TiDBに接続するには、選択したTiDBデプロイオプションに応じてTiDBクラスターに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が、オペレーティング環境に一致していることを確認します。

   - **Endpoint Type** が `Public` に設定されていること。
   - **Branch** が `main` に設定されていること。
   - **Connect With** が `Navicat` に設定されていること。
   - **Operating System** が環境に一致していること。

4. **Generate Password** をクリックして、ランダムなパスワードを作成します。

   > **ヒント:**
   >
   > 以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、新しいパスワードを生成するために **Reset Password** をクリックすることができます。

5. Navicat for MySQL を起動し、左上隅の **Connection** をクリックし、ドロップダウンリストから **MySQL** を選択します。

   ![Navicat: add new connection](/media/develop/navicat-add-new-connection.jpg)

6. **New Connection (MySQL)** ダイアログで、次の接続パラメータを設定します。

   - **Connection Name**: この接続に意味のある名前を付けます。
   - **Host**: TiDB Cloud 接続ダイアログから `HOST` パラメータを入力します。
   - **Port**: TiDB Cloud 接続ダイアログから `PORT` パラメータを入力します。
   - **User Name**: TiDB Cloud 接続ダイアログから `USERNAME` パラメータを入力します。
   - **Password**: TiDB Serverless クラスタのパスワードを入力します。

   ![Navicat: configure connection general panel for TiDB Serverless](/media/develop/navicat-connection-config-serverless-general.png)

7. **SSL** タブをクリックし、**Use SSL**、**Use authentication**、**Verify server certificate against CA** のチェックボックスを選択します。次に、TiDB Cloud 接続ダイアログからダウンロードした `CA` ファイルを **CA Certificate** フィールドに選択します。

   ![Navicat: configure connection SSL panel for TiDB Serverless](/media/develop/navicat-connection-config-serverless-ssl.png)

8. **Test Connection** をクリックして、TiDB Serverless クラスタへの接続を検証します。

9. 接続テストが成功した場合、**Connection Successful** メッセージが表示されます。接続構成を完了するために **Save** をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックします。

   接続文字列を取得する詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. **Download CA cert** をクリックして、CA ファイルをダウンロードします。

5. Navicat for MySQL を起動し、左上隅の **Connection** をクリックし、ドロップダウンリストから **MySQL** を選択します。

   ![Navicat: add new connection](/media/develop/navicat-add-new-connection.jpg)

6. **New Connection (MySQL)** ダイアログで、次の接続パラメータを設定します。

   - **Connection Name**: この接続に意味のある名前を付けます。
   - **Host**: TiDB Cloud 接続ダイアログから `host` パラメータを入力します。
   - **Port**: TiDB Cloud 接続ダイアログから `port` パラメータを入力します。
   - **User Name**: TiDB Cloud 接続ダイアログから `user` パラメータを入力します。
   - **Password**: TiDB Dedicated クラスタのパスワードを入力します。

   ![Navicat: configure connection general panel for TiDB Dedicated](/media/develop/navicat-connection-config-dedicated-general.png)

7. **SSL** タブをクリックし、**Use SSL**、**Use authentication**、**Verify server certificate against CA** のチェックボックスを選択します。次に、ステップ 4 でダウンロードした CA ファイルを **CA Certificate** フィールドに選択します。

   ![Navicat: configure connection SSL panel for TiDB Dedicated](/media/develop/navicat-connection-config-dedicated-ssl.jpg)

8. **Test Connection** をクリックして、TiDB Dedicated クラスタへの接続を検証します。

9. 接続テストが成功した場合、**Connection Successful** メッセージが表示されます。接続構成を完了するために **Save** をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. Navicat for MySQLを起動し、左上隅の**接続**をクリックし、ドロップダウンリストから**MySQL**を選択します。

   ![Navicat: 新しい接続を追加](/media/develop/navicat-add-new-connection.jpg)

2. **新しい接続 (MySQL)** ダイアログで、以下の接続パラメータを設定します:

   - **接続名**: この接続に意味のある名前を付けます。
   - **ホスト**: TiDB Self-HostedクラスターのIPアドレスまたはドメイン名を入力します。
   - **ポート**: TiDB Self-Hostedクラスターのポート番号を入力します。
   - **ユーザー名**: TiDBに接続するために使用するユーザー名を入力します。
   - **パスワード**: TiDBに接続するために使用するパスワードを入力します。

   ![Navicat: Self-Hosted TiDBの接続一般パネルを設定する](/media/develop/navicat-connection-config-self-hosted-general.png)

3. **接続をテスト**をクリックして、TiDB Self-Hostedクラスターへの接続を検証します。

4. 接続テストが成功した場合、**接続に成功しました**メッセージが表示されます。接続構成を完了するには、**保存**をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

- [デベロッパーガイド](/develop/dev-guide-overview.md)の章で、TiDBアプリケーション開発のベストプラクティスを学びます。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- 専門の[TiDBデベロッパーコース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後、[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/tidb-cloud/tidb-cloud-support.md)してください。

</CustomContent>
