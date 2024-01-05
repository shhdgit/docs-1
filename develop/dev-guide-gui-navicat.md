---
title: Connect to TiDB with Navicat
summary: Learn how to connect to TiDB using Navicat.
---

# Navicatを使用してTiDBに接続する {#connect-to-tidb-with-navicat}

TiDBはMySQL互換のデータベースであり、[Navicat](https://www.navicat.com)はデータベースユーザー向けのGUIツールセットです。このチュートリアルでは、[Navicat for MySQL](https://www.navicat.com/en/products/navicat-for-mysql)ツールを使用してTiDBに接続します。

> **警告:**
>
> - NavicatはMySQL互換性を持つためTiDBに接続できますが、TiDBを完全にサポートしていません。TiDBをMySQLとして扱うため、使用中にいくつかの問題が発生する可能性があります。[Navicatユーザー管理の互換性](https://github.com/pingcap/tidb/issues/45154)に関する既知の問題があります。NavicatとTiDBの互換性の詳細については、[TiDB GitHubの問題ページ](https://github.com/pingcap/tidb/issues?q=is%3Aissue+navicat+is%3Aopen)を参照してください。
> - [DataGrip](/develop/dev-guide-gui-datagrip.md)、[DBeaver](/develop/dev-guide-gui-dbeaver.md)、および[VS Code SQLTools](/develop/dev-guide-gui-vscode-sqltools.md)など、公式にTiDBをサポートしている他のGUIツールの使用を推奨します。TiDBが完全にサポートするGUIツールの完全なリストについては、[TiDBがサポートするサードパーティツール](/develop/dev-guide-third-party-support.md#gui)を参照してください。

このチュートリアルでは、Navicatを使用してTiDBクラスタに接続する方法を学ぶことができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です:

- [Navicat for MySQL](https://www.navicat.com/en/download/navicat-for-mysql) **16.3.2** またはそれ以降のバージョン。
- Navicat for MySQLの有料アカウント。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)を作成するために従ってください。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を作成するために従ってください。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

TiDBに接続するには、選択したTiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタ名をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致することを確認してください。

   - **Endpoint Type**が`Public`に設定されていること。
   - **Branch**が`main`に設定されていること。
   - **Connect With**が`Navicat`に設定されていること。
   - **Operating System**が環境に一致していること。

4. **Generate Password**をクリックしてランダムなパスワードを作成します。

   > **ヒント:**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成できます。

5. Navicat for MySQLを起動し、左上隅の**Connection**をクリックし、ドロップダウンリストから**MySQL**を選択します。

   ![Navicat: 新しい接続の追加](/media/develop/navicat-add-new-connection.jpg)

6. \*\*New Connection (MySQL)\*\*ダイアログで、次の接続パラメータを構成します:

   - **Connection Name**: この接続に意味のある名前を付けます。
   - **Host**: TiDB Cloud接続ダイアログから`HOST`パラメータを入力します。
   - **Port**: TiDB Cloud接続ダイアログから`PORT`パラメータを入力します。
   - **User Name**: TiDB Cloud接続ダイアログから`USERNAME`パラメータを入力します。
   - **Password**: TiDB Serverlessクラスタのパスワードを入力します。

   ![Navicat: TiDB Serverlessの接続構成一般パネル](/media/develop/navicat-connection-config-serverless-general.png)

7. **SSL**タブをクリックし、**Use SSL**、**Use authentication**、および**Verify server certificate against CA**のチェックボックスを選択します。次に、TiDB Cloud接続ダイアログから`CA`ファイルを**CA Certificate**フィールドに選択します。

   ![Navicat: TiDB Serverlessの接続構成SSLパネル](/media/develop/navicat-connection-config-serverless-ssl.png)

8. **Test Connection**をクリックしてTiDB Serverlessクラスタへの接続を検証します。

9. 接続テストが成功した場合、**Connection Successful**メッセージが表示されます。接続構成を完了するために**Save**をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタ名をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. **Download CA cert**をクリックしてCAファイルをダウンロードします。

5. Navicat for MySQLを起動し、左上隅の**Connection**をクリックし、ドロップダウンリストから**MySQL**を選択します。

   ![Navicat: 新しい接続の追加](/media/develop/navicat-add-new-connection.jpg)

6. \*\*New Connection (MySQL)\*\*ダイアログで、次の接続パラメータを構成します:

   - **Connection Name**: この接続に意味のある名前を付けます。
   - **Host**: TiDB Cloud接続ダイアログから`host`パラメータを入力します。
   - **Port**: TiDB Cloud接続ダイアログから`port`パラメータを入力します。
   - **User Name**: TiDB Cloud接続ダイアログから`user`パラメータを入力します。
   - **Password**: TiDB Dedicatedクラスタのパスワードを入力します。

   ![Navicat: TiDB Dedicatedの接続構成一般パネル](/media/develop/navicat-connection-config-dedicated-general.png)

7. **SSL**タブをクリックし、**Use SSL**、**Use authentication**、および**Verify server certificate against CA**のチェックボックスを選択します。次に、ステップ4でダウンロードしたCAファイルを**CA Certificate**フィールドに選択します。

   ![Navicat: TiDB Dedicatedの接続構成SSLパネル](/media/develop/navicat-connection-config-dedicated-ssl.jpg)

8. TiDB Dedicatedクラスタへの接続を検証するために**Test Connection**をクリックします。

9. 接続テストが成功した場合、**Connection Successful**メッセージが表示されます。接続構成を完了するために**Save**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. Navicat for MySQLを起動し、左上隅の**Connection**をクリックし、ドロップダウンリストから**MySQL**を選択します。

   ![Navicat: 新しい接続の追加](/media/develop/navicat-add-new-connection.jpg)

2. \*\*New Connection (MySQL)\*\*ダイアログで、次の接続パラメータを構成します:

   - **Connection Name**: この接続に意味のある名前を付けます。
   - **Host**: TiDB Self-HostedクラスタのIPアドレスまたはドメイン名を入力します。
   - **Port**: TiDB Self-Hostedクラスタのポート番号を入力します。
   - **User Name**: TiDBに接続するために使用するユーザー名を入力します。
   - **Password**: TiDBに接続するために使用するパスワードを入力します。

   ![Navicat: self-hosted TiDBの接続構成一般パネル](/media/develop/navicat-connection-config-self-hosted-general.png)

3. **Test Connection**をクリックしてTiDB Self-Hostedクラスタへの接続を検証します。

4. 接続テストが成功した場合、**Connection Successful**メッセージが表示されます。接続構成を完了するために**Save**をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

- [開発者ガイド](/develop/dev-guide-overview.md)の章を読んで、TiDBアプリケーション開発のベストプラクティスを学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などを学びます。
- [TiDB開発者コース](https://www.pingcap.com/education/)を受講し、試験に合格して[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/tidb-cloud/tidb-cloud-support.md)してください。

</CustomContent>
