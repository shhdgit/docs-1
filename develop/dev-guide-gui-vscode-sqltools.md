---
title: Connect to TiDB with Visual Studio Code
summary: Learn how to connect to TiDB using Visual Studio Code or GitHub Codespaces.
---

# Visual Studio CodeでTiDBに接続する {#connect-to-tidb-with-visual-studio-code}

TiDBはMySQL互換のデータベースであり、[Visual Studio Code (VS Code)](https://code.visualstudio.com/)は軽量でありながら強力なソースコードエディタです。このチュートリアルでは、[SQLTools](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools)拡張機能を使用し、TiDBを[公式ドライバー](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)としてサポートしています。

このチュートリアルでは、Visual Studio Codeを使用してTiDBクラスターに接続する方法を学ぶことができます。

> **Note:**
>
> - このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedに対応しています。
> - このチュートリアルは、[GitHub Codespaces](https://github.com/features/codespaces)、[Visual Studio Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)、[Visual Studio Code WSL](https://code.visualstudio.com/docs/remote/wsl)などのVisual Studio Codeリモート開発環境でも動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Visual Studio Code](https://code.visualstudio.com/#alt-downloads) **1.72.0**以降のバージョン。
- Visual Studio Code用の[SQLTools MySQL/MariaDB/TiDB](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)拡張機能。インストールするには、次のいずれかの方法を使用できます。
  - <a href="vscode:extension/mtxr.sqltools-driver-mysql">このリンク</a>をクリックしてVS Codeを起動し、拡張機能を直接インストールします。
  - [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)に移動し、**インストール**をクリックします。
- TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスターを作成する](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターをデプロイする](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターをデプロイする](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスターを作成する](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターをデプロイする](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターをデプロイする](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

選択したTiDBデプロイメントオプションに応じて、TiDBクラスターに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が、オペレーティング環境に一致していることを確認します。

   - **Endpoint Type** が `Public` に設定されていること。

   - **Branch** が `main` に設定されていること。

   - **Connect With** が `VS Code` に設定されていること。

   - **Operating System** が環境に一致していること。

   > **Tip:**
   >
   > VS Code がリモート開発環境で実行されている場合は、リストからリモートオペレーティングシステムを選択します。たとえば、Windows Subsystem for Linux (WSL) を使用している場合は、対応する Linux ディストリビューションに切り替えます。GitHub Codespaces を使用している場合は、必要ありません。

4. ランダムなパスワードを作成するには、**Generate Password** をクリックします。

   > **Tip:**
   >
   > 以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、**Reset Password** をクリックして新しいパスワードを生成できます。

5. VS Code を起動し、ナビゲーションペインで **SQLTools** 拡張機能を選択します。**CONNECTIONS** セクションの下で、**Add New Connection** をクリックし、データベースドライバとして **TiDB** を選択します。

   ![VS Code SQLTools: add new connection](/media/develop/vsc-sqltools-add-new-connection.jpg)

6. 設定ペインで、次の接続パラメータを設定します。

   - **Connection name**: この接続に意味のある名前を付けます。
   - **Connection group**: (オプション) この接続グループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。
   - **Connect using**: **Server and Port** を選択します。
   - **Server Address**: TiDB Cloud 接続ダイアログの `HOST` パラメータを入力します。
   - **Port**: TiDB Cloud 接続ダイアログの `PORT` パラメータを入力します。
   - **Database**: 接続したいデータベースを入力します。
   - **Username**: TiDB Cloud 接続ダイアログの `USERNAME` パラメータを入力します。
   - **Password mode**: **SQLTools Driver Credentials** を選択します。
   - **MySQL driver specific options** エリアで、次のパラメータを設定します。

     - **Authentication Protocol**: **default** を選択します。
     - **SSL**: **Enabled** を選択します。TiDB Serverless では、安全な接続が必要です。**SSL Options (node.TLSSocket)** エリアで、**Certificate Authority (CA) Certificate File** フィールドを TiDB Cloud 接続ダイアログの `CA` パラメータに設定します。

       > **Note:**
       >
       > Windows または GitHub Codespaces で実行している場合は、**SSL** を空白のままにしておくことができます。SQLTools は、デフォルトで Let's Encrypt によって管理されているよく知られた CA を信頼します。詳細については、[TiDB Serverless ルート証明書の管理](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters#root-certificate-management)を参照してください。

   ![VS Code SQLTools: configure connection settings for TiDB Serverless](/media/develop/vsc-sqltools-connection-config-serverless.jpg)

7. **TEST CONNECTION** をクリックして、TiDB Serverless クラスタへの接続を検証します。

   1. ポップアップウィンドウで **Allow** をクリックします。
   2. **SQLTools Driver Credentials** ダイアログで、ステップ 4 で作成したパスワードを入力します。

      ![VS Code SQLTools: enter password to connect to TiDB Serverless](/media/develop/vsc-sqltools-password.jpg)

8. 接続テストが成功した場合、**Successfully connected!** メッセージが表示されます。接続構成を保存するには、**SAVE CONNECTION** をクリックします。

9. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、その概要ページに移動します。

10. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

11. **Allow Access from Anywhere** をクリックします。

    接続文字列を取得する詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

12. VS Code を起動し、ナビゲーションペインで **SQLTools** 拡張機能を選択します。**CONNECTIONS** セクションの下で、**Add New Connection** をクリックし、データベースドライバとして **TiDB** を選択します。

    ![VS Code SQLTools: add new connection](/media/develop/vsc-sqltools-add-new-connection.jpg)

13. 設定ペインで、次の接続パラメータを設定します。

    - **Connection name**: この接続に意味のある名前を付けます。
    - **Connection group**: (オプション) この接続グループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。
    - **Connect using**: **Server and Port** を選択します。
    - **Server Address**: TiDB Cloud 接続ダイアログの `host` パラメータを入力します。
    - **Port**: TiDB Cloud 接続ダイアログの `port` パラメータを入力します。
    - **Database**: 接続したいデータベースを入力します。
    - **Username**: TiDB Cloud 接続ダイアログの `user` パラメータを入力します。
    - **Password mode**: **SQLTools Driver Credentials** を選択します。
    - **MySQL driver specific options** エリアで、次のパラメータを設定します。

      - **Authentication Protocol**: **default** を選択します。
      - **SSL**: **Disabled** を選択します。

    ![VS Code SQLTools: configure connection settings for TiDB Dedicated](/media/develop/vsc-sqltools-connection-config-dedicated.jpg)

14. **TEST CONNECTION** をクリックして、TiDB Dedicated クラスタへの接続を検証します。

    1. ポップアップウィンドウで **Allow** をクリックします。
    2. **SQLTools Driver Credentials** ダイアログで、TiDB Dedicated クラスタのパスワードを入力します。

    ![VS Code SQLTools: enter password to connect to TiDB Dedicated](/media/develop/vsc-sqltools-password.jpg)

15. 接続テストが成功した場合、**Successfully connected!** メッセージが表示されます。接続設定を保存するには、**SAVE CONNECTION** をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. VS Code を起動し、ナビゲーションペインで **SQLTools** 拡張機能を選択します。**CONNECTIONS** セクションの下で、**Add New Connection** をクリックし、データベースドライバとして **TiDB** を選択します。

   ![VS Code SQLTools: add new connection](/media/develop/vsc-sqltools-add-new-connection.jpg)

2. 設定ペインで、次の接続パラメータを設定します。

   - **Connection name**: この接続に意味のある名前を付けます。

   - **Connection group**: (オプション) この接続グループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。

   - **Connect using**: **Server and Port** を選択します。

   - **Server Address**: TiDB Self-Hosted クラスタの IP アドレスまたはドメイン名を入力します。

   - **Port**: TiDB Self-Hosted クラスタのポート番号を入力します。

   - **Database**: 接続したいデータベースを入力します。

   - **Username**: TiDB Self-Hosted クラスタに接続するために使用するユーザー名を入力します。

   - **Password mode**:

     - パスワードが空の場合、**Use empty password** を選択します。
     - それ以外の場合、**SQLTools Driver Credentials** を選択します。

   - **MySQL driver specific options** エリアで、次のパラメータを設定します。

     - **Authentication Protocol**: **default** を選択します。
     - **SSL**: **Disabled** を選択します。

   ![VS Code SQLTools: configure connection settings for TiDB Self-Hosted](/media/develop/vsc-sqltools-connection-config-self-hosted.jpg)

3. **TEST CONNECTION** をクリックして、TiDB Self-Hosted クラスタへの接続を検証します。

   パスワードが空でない場合、ポップアップウィンドウで **Allow** をクリックし、TiDB Self-Hosted クラスタのパスワードを入力します。

   ![VS Code SQLTools: enter password to connect to TiDB Self-Hosted](/media/develop/vsc-sqltools-password.jpg)

4. 接続テストが成功した場合、**Successfully connected!** メッセージが表示されます。接続設定を保存するには、**SAVE CONNECTION** をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

- [Visual Studio Codeのドキュメント](https://code.visualstudio.com/docs)から、Visual Studio Codeのさらなる使用方法を学びましょう。
- [SQLToolsのドキュメント](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools)と[SQLToolsのGitHubリポジトリ](https://github.com/mtxr/vscode-sqltools)から、VS Code SQLTools拡張機能のさらなる使用方法を学びましょう。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びましょう。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定資格](https://www.pingcap.com/education/certification/)を取得しましょう。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
