---
title: Connect to TiDB with Visual Studio Code
summary: Learn how to connect to TiDB using Visual Studio Code or GitHub Codespaces.
---

# Visual Studio Codeを使用してTiDBに接続する {#connect-to-tidb-with-visual-studio-code}

TiDBはMySQL互換のデータベースであり、[Visual Studio Code (VS Code)](https://code.visualstudio.com/)は軽量でありながら強力なソースコードエディタです。このチュートリアルでは、[SQLTools](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools)拡張機能を使用して、TiDBを[公式ドライバ](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)としてサポートしています。

このチュートリアルでは、Visual Studio Codeを使用してTiDBクラスタに接続する方法を学ぶことができます。

> **注意:**
>
> -   このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと互換性があります。
> -   このチュートリアルは、[GitHub Codespaces](https://github.com/features/codespaces)、[Visual Studio Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)、[Visual Studio Code WSL](https://code.visualstudio.com/docs/remote/wsl)などのVisual Studio Code Remote Development環境でも動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、以下が必要です:

-   [Visual Studio Code](https://code.visualstudio.com/#alt-downloads) **1.72.0** またはそれ以降のバージョン。
-   Visual Studio Code用の[SQLTools MySQL/MariaDB/TiDB](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)拡張機能。インストールするには、以下のいずれかの方法を使用できます:
    -   [このリンク](vscode:extension/mtxr.sqltools-driver-mysql)をクリックして、VS Codeを起動し、拡張機能を直接インストールします。
    -   [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)に移動し、**インストール**をクリックします。
-   TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、以下のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、以下のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

TiDBのデプロイオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1.  [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスターの名前をクリックして、概要ページに移動します。

2.  右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成がオペレーティング環境に一致していることを確認します。

    -   **エンドポイントタイプ** が `Public` に設定されていること。

    -   **ブランチ** が `main` に設定されていること。

    -   **接続方法** が `VS Code` に設定されていること。

    -   **オペレーティングシステム** が環境に一致していること。

    > **ヒント:**
    >
    > もしVS Codeがリモート開発環境で実行されている場合は、リストからリモートオペレーティングシステムを選択してください。例えば、Windows Subsystem for Linux (WSL) を使用している場合は、対応するLinuxディストリビューションに切り替えてください。GitHub Codespacesを使用している場合は、これは必要ありません。

4.  ランダムなパスワードを作成するために**パスワードを生成**をクリックします。

    > **ヒント:**
    >
    > もし以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、**パスワードをリセット**をクリックして新しいパスワードを生成することができます。

5.  VS Codeを起動し、ナビゲーションペインで**SQLTools**拡張機能を選択します。**CONNECTIONS**セクションの下で、**新しい接続を追加**をクリックし、データベースドライバとして**TiDB**を選択します。

    ![VS Code SQLTools: 新しい接続を追加](/media/develop/vsc-sqltools-add-new-connection.jpg)

6.  設定ペインで、以下の接続パラメータを構成します。

    -   **接続名**: この接続に意味のある名前を付けます。
    -   **接続グループ**: (任意) この接続グループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。
    -   **使用して接続**: **サーバーとポート**を選択します。
    -   **サーバーアドレス**: TiDB Cloud接続ダイアログからの`HOST`パラメータを入力します。
    -   **ポート**: TiDB Cloud接続ダイアログからの`PORT`パラメータを入力します。
    -   **データベース**: 接続したいデータベースを入力します。
    -   **ユーザー名**: TiDB Cloud接続ダイアログからの`USERNAME`パラメータを入力します。
    -   **パスワードモード**: **SQLToolsドライバー資格情報**を選択します。
    -   **MySQLドライバー固有のオプション**エリアで、以下のパラメータを構成します。

        -   **認証プロトコル**: **デフォルト**を選択します。
        -   **SSL**: **有効**を選択します。TiDB Serverlessでは安全な接続が必要です。**SSLオプション (node.TLSSocket)**エリアで、TiDB Cloud接続ダイアログからの`CA`パラメータを**証明書機関（CA）証明書ファイル**フィールドに構成します。

            > **注意:**
            >
            > もしWindowsやGitHub Codespacesで実行している場合は、**SSL**を空白のままにしておくことができます。デフォルトではSQLToolsはLet's Encryptによって管理されたよく知られたCAを信頼しています。詳細については、[TiDB Serverlessルート証明書管理](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters#root-certificate-management)を参照してください。

    ![VS Code SQLTools: TiDB Serverlessの接続設定を構成](/media/develop/vsc-sqltools-connection-config-serverless.jpg)

7.  **接続をテスト**をクリックして、TiDB Serverlessクラスターへの接続を検証します。

    1.  ポップアップウィンドウで**許可**をクリックします。
    2.  **SQLToolsドライバー資格情報**ダイアログで、ステップ4で作成したパスワードを入力します。

        ![VS Code SQLTools: TiDB Serverlessに接続するためのパスワードを入力](/media/develop/vsc-sqltools-password.jpg)

8.  接続テストが成功した場合、**接続が正常に完了しました！**メッセージが表示されます。接続構成を保存するために**接続を保存**をクリックします。

9.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

10. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

11. **どこからでもアクセスを許可**をクリックします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated 標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

12. VS Codeを起動し、ナビゲーションペインで**SQLTools**拡張機能を選択します。**CONNECTIONS**セクションの下で、**新しい接続を追加**をクリックし、データベースドライバとして**TiDB**を選択します。

    ![VS Code SQLTools: 新しい接続を追加](/media/develop/vsc-sqltools-add-new-connection.jpg)

13. 設定ペインで、以下の接続パラメータを構成します：

    -   **接続名**：この接続に意味のある名前を付けます。
    -   **接続グループ**：（オプション）この接続のグループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。
    -   **使用して接続**：**サーバーとポート**を選択します。
    -   **サーバーアドレス**：TiDB Cloud接続ダイアログから`host`パラメータを入力します。
    -   **ポート**：TiDB Cloud接続ダイアログから`port`パラメータを入力します。
    -   **データベース**：接続したいデータベースを入力します。
    -   **ユーザー名**：TiDB Cloud接続ダイアログから`user`パラメータを入力します。
    -   **パスワードモード**：**SQLToolsドライバ資格情報**を選択します。
    -   **MySQLドライバ固有のオプション**エリアで、以下のパラメータを構成します：

        -   **認証プロトコル**：**デフォルト**を選択します。
        -   **SSL**：**無効**を選択します。

    ![VS Code SQLTools: TiDB Dedicatedの接続設定を構成](/media/develop/vsc-sqltools-connection-config-dedicated.jpg)

14. **接続をテスト**をクリックして、TiDB Dedicatedクラスタへの接続を検証します。

    1.  ポップアップウィンドウで**許可**をクリックします。
    2.  **SQLToolsドライバ資格情報**ダイアログで、TiDB Dedicatedクラスタのパスワードを入力します。

    ![VS Code SQLTools: TiDB Dedicatedに接続するためのパスワードを入力](/media/develop/vsc-sqltools-password.jpg)

15. 接続テストが成功した場合、\*\*正常に接続しました！\*\*メッセージが表示されます。接続構成を保存するには、**接続を保存**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1.  VS Codeを起動し、ナビゲーションペインで**SQLTools**拡張機能を選択します。**CONNECTIONS**セクションの下で、**新しい接続を追加**をクリックし、データベースドライバとして**TiDB**を選択します。

    ![VS Code SQLTools: 新しい接続を追加](/media/develop/vsc-sqltools-add-new-connection.jpg)

2.  設定ペインで、以下の接続パラメータを構成します：

    -   **接続名**：この接続に意味のある名前を付けます。

    -   **接続グループ**：（オプション）この接続のグループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。

    -   **使用して接続**：**サーバーとポート**を選択します。

    -   **サーバーアドレス**：TiDB Self-HostedクラスタのIPアドレスまたはドメイン名を入力します。

    -   **ポート**：TiDB Self-Hostedクラスタのポート番号を入力します。

    -   **データベース**：接続したいデータベースを入力します。

    -   **ユーザー名**：TiDB Self-Hostedクラスタに接続するために使用するユーザー名を入力します。

    -   **パスワードモード**：

        -   パスワードが空の場合は、**空のパスワードを使用**を選択します。
        -   それ以外の場合は、**SQLToolsドライバ資格情報**を選択します。

    -   **MySQLドライバ固有のオプション**エリアで、以下のパラメータを構成します：

        -   **認証プロトコル**：**デフォルト**を選択します。
        -   **SSL**：**無効**を選択します。

    ![VS Code SQLTools: TiDB Self-Hostedの接続設定を構成](/media/develop/vsc-sqltools-connection-config-self-hosted.jpg)

3.  **接続をテスト**をクリックして、TiDB Self-Hostedクラスタへの接続を検証します。

    パスワードが空でない場合は、ポップアップウィンドウで**許可**をクリックし、その後、TiDB Self-Hostedクラスタのパスワードを入力します。

    ![VS Code SQLTools: TiDB Self-Hostedに接続するためのパスワードを入力](/media/develop/vsc-sqltools-password.jpg)

4.  接続テストが成功した場合、\*\*正常に接続しました！\*\*メッセージが表示されます。接続構成を保存するには、**接続を保存**をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

-   [Visual Studio Codeのドキュメント](https://code.visualstudio.com/docs)から、Visual Studio Codeのさらなる使用法を学びます。
-   [SQLToolsのドキュメント](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools)および[GitHubリポジトリ](https://github.com/mtxr/vscode-sqltools)から、VS Code SQLTools拡張機能のさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
-   [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
