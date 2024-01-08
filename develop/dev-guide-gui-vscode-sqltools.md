---
title: Connect to TiDB with Visual Studio Code
summary: Learn how to connect to TiDB using Visual Studio Code or GitHub Codespaces.
---

# Visual Studio CodeでTiDBに接続する {#connect-to-tidb-with-visual-studio-code}

TiDBはMySQL互換のデータベースであり、[Visual Studio Code (VS Code)](https://code.visualstudio.com/)は軽量でありながら強力なソースコードエディタです。このチュートリアルでは、TiDBを[公式ドライバー](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)としてサポートする[SQLTools](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools)拡張機能を使用します。

このチュートリアルでは、Visual Studio Codeを使用してTiDBクラスタに接続する方法を学ぶことができます。

> **Note:**
>
> - このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと互換性があります。
> - このチュートリアルは、[GitHub Codespaces](https://github.com/features/codespaces)、[Visual Studio Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)、[Visual Studio Code WSL](https://code.visualstudio.com/docs/remote/wsl)などのVisual Studio Codeリモート開発環境でも機能します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です：

- [Visual Studio Code](https://code.visualstudio.com/#alt-downloads) **1.72.0** 以降のバージョン。
- Visual Studio Code用の[SQLTools MySQL/MariaDB/TiDB](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)拡張機能。インストールするには、次のいずれかの方法を使用できます：
  - [このリンク](vscode:extension/mtxr.sqltools-driver-mysql)をクリックして、VS Codeを起動し、拡張機能を直接インストールします。
  - [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)に移動し、**Install**をクリックします。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます：**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます：**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

選択したTiDBデプロイメントオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が、操作環境に一致していることを確認します。

   - **Endpoint Type** が `Public` に設定されていること。

   - **Branch** が `main` に設定されていること。

   - **Connect With** が `VS Code` に設定されていること。

   - **Operating System** が環境に一致していること。

   > **ヒント:**
   >
   > もしVS Codeがリモート開発環境で実行されている場合は、リストからリモートオペレーティングシステムを選択します。例えば、Windows Subsystem for Linux (WSL)を使用している場合は、対応するLinuxディストリビューションに切り替えます。これはGitHub Codespacesを使用している場合は必要ありません。

4. ランダムなパスワードを作成するために **Generate Password** をクリックします。

   > **ヒント:**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために **Reset Password** をクリックできます。

5. VS Codeを起動し、ナビゲーションペインで **SQLTools** 拡張機能を選択します。**CONNECTIONS** セクションで **Add New Connection** をクリックし、データベースドライバとして **TiDB** を選択します。

   ![VS Code SQLTools: add new connection](/media/develop/vsc-sqltools-add-new-connection.jpg)

6. 設定ペインで、以下の接続パラメータを構成します:

   - **Connection name**: この接続に意味のある名前を付けます。
   - **Connection group**: (オプション) この接続グループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。
   - **Connect using**: **Server and Port** を選択します。
   - **Server Address**: TiDB Cloud接続ダイアログから `HOST` パラメータを入力します。
   - **Port**: TiDB Cloud接続ダイアログから `PORT` パラメータを入力します。
   - **Database**: 接続したいデータベースを入力します。
   - **Username**: TiDB Cloud接続ダイアログから `USERNAME` パラメータを入力します。
   - **Password mode**: **SQLTools Driver Credentials** を選択します。
   - **MySQL driver specific options** エリアで、以下のパラメータを構成します:

     - **Authentication Protocol**: **default** を選択します。
     - **SSL**: **Enabled** を選択します。TiDB Serverlessでは安全な接続が必要です。**SSL Options (node.TLSSocket)** エリアで、**Certificate Authority (CA) Certificate File** フィールドをTiDB Cloud接続ダイアログからの `CA` パラメータに構成します。

       > **Note:**
       >
       > WindowsまたはGitHub Codespacesで実行している場合は、**SSL** を空白のままにしておくことができます。デフォルトでは、SQLToolsはLet's Encryptによって管理されるよく知られたCAを信頼します。詳細については、[TiDB Serverlessルート証明書管理](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters#root-certificate-management)を参照してください。

   ![VS Code SQLTools: configure connection settings for TiDB Serverless](/media/develop/vsc-sqltools-connection-config-serverless.jpg)

7. **TEST CONNECTION** をクリックして、TiDB Serverlessクラスタへの接続を検証します。

   1. ポップアップウィンドウで **Allow** をクリックします。
   2. **SQLTools Driver Credentials** ダイアログで、ステップ4で作成したパスワードを入力します。

      ![VS Code SQLTools: enter password to connect to TiDB Serverless](/media/develop/vsc-sqltools-password.jpg)

8. 接続テストが成功した場合、**Successfully connected!** メッセージが表示されます。接続構成を保存するために **SAVE CONNECTION** をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. VS Codeを起動し、ナビゲーションペインで **SQLTools** 拡張機能を選択します。**CONNECTIONS** セクションで **Add New Connection** をクリックし、データベースドライバとして **TiDB** を選択します。

   ![VS Code SQLTools: add new connection](/media/develop/vsc-sqltools-add-new-connection.jpg)

5. 設定ペインで、以下の接続パラメータを構成します:

   - **Connection name**: この接続に意味のある名前を付けます。
   - **Connection group**: (オプション) この接続グループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。
   - **Connect using**: **Server and Port** を選択します。
   - **Server Address**: TiDB Cloud接続ダイアログから `host` パラメータを入力します。
   - **Port**: TiDB Cloud接続ダイアログから `port` パラメータを入力します。
   - **Database**: 接続したいデータベースを入力します。
   - **Username**: TiDB Cloud接続ダイアログから `user` パラメータを入力します。
   - **Password mode**: **SQLTools Driver Credentials** を選択します。
   - **MySQL driver specific options** エリアで、以下のパラメータを構成します:

     - **Authentication Protocol**: **default** を選択します。
     - **SSL**: **Disabled** を選択します。

   ![VS Code SQLTools: configure connection settings for TiDB Dedicated](/media/develop/vsc-sqltools-connection-config-dedicated.jpg)

6. **TEST CONNECTION** をクリックして、TiDB Dedicatedクラスタへの接続を検証します。

   1. ポップアップウィンドウで **Allow** をクリックします。
   2. **SQLTools Driver Credentials** ダイアログで、TiDB Dedicatedクラスタのパスワードを入力します。

   ![VS Code SQLTools: enter password to connect to TiDB Dedicated](/media/develop/vsc-sqltools-password.jpg)

7. 接続テストが成功した場合、**Successfully connected!** メッセージが表示されます。接続構成を保存するために **SAVE CONNECTION** をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. VS Codeを起動し、ナビゲーションペインで **SQLTools** 拡張機能を選択します。**CONNECTIONS** セクションで **Add New Connection** をクリックし、データベースドライバとして **TiDB** を選択します。

   ![VS Code SQLTools: add new connection](/media/develop/vsc-sqltools-add-new-connection.jpg)

2. 設定ペインで、以下の接続パラメータを構成します:

   - **Connection name**: この接続に意味のある名前を付けます。

   - **Connection group**: (オプション) この接続グループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。

   - **Connect using**: **Server and Port** を選択します。

   - **Server Address**: TiDB Self-HostedクラスタのIPアドレスまたはドメイン名を入力します。

   - **Port**: TiDB Self-Hostedクラスタのポート番号を入力します。

   - **Database**: 接続したいデータベースを入力します。

   - **Username**: TiDB Self-Hostedクラスタに接続するために使用するユーザー名を入力します。

   - **Password mode**:

     - パスワードが空の場合は、**Use empty password** を選択します。
     - それ以外の場合は、**SQLTools Driver Credentials** を選択します。

   - **MySQL driver specific options** エリアで、以下のパラメータを構成します:

     - **Authentication Protocol**: **default** を選択します。
     - **SSL**: **Disabled** を選択します。

   ![VS Code SQLTools: configure connection settings for TiDB Self-Hosted](/media/develop/vsc-sqltools-connection-config-self-hosted.jpg)

3. **TEST CONNECTION** をクリックして、TiDB Self-Hostedクラスタへの接続を検証します。

   パスワードが空でない場合は、ポップアップウィンドウで **Allow** をクリックし、TiDB Self-Hostedクラスタのパスワードを入力します。

   ![VS Code SQLTools: enter password to connect to TiDB Self-Hosted](/media/develop/vsc-sqltools-password.jpg)

4. 接続テストが成功した場合、**Successfully connected!** メッセージが表示されます。接続構成を保存するために **SAVE CONNECTION** をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

- [Visual Studio Codeのドキュメント](https://code.visualstudio.com/docs)からVisual Studio Codeのさらなる使用法を学びます。
- [SQLTools拡張機能のドキュメント](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools)およびSQLToolsの[GitHubリポジトリ](https://github.com/mtxr/vscode-sqltools)から、SQLTools拡張機能のさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問してください。または、[サポートチケットを作成](https://support.pingcap.com/)してください。
