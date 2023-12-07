---
title: Connect to TiDB with JetBrains DataGrip
summary: Learn how to connect to TiDB using JetBrains DataGrip. This tutorial also applies to the Database Tools and SQL plugin available in other JetBrains IDEs, such as IntelliJ, PhpStorm, and PyCharm.
---

# JetBrains DataGripを使用してTiDBに接続する {#connect-to-tidb-with-jetbrains-datagrip}

TiDBはMySQL互換のデータベースであり、[JetBrains DataGrip](https://www.jetbrains.com/help/datagrip/getting-started.html)はデータベースとSQLのための強力な統合開発環境（IDE）です。このチュートリアルでは、DataGripを使用してTiDBクラスタに接続する手順について説明します。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと互換性があります。

DataGripを以下の2つの方法で使用できます：

-   [DataGrip IDE](https://www.jetbrains.com/datagrip/download)としてスタンドアロンツールとして。
-   IntelliJ、PhpStorm、PyCharmなどのJetBrains IDEでの[Database Tools and SQLプラグイン](https://www.jetbrains.com/help/idea/relational-databases.html)として。

このチュートリアルは主にスタンドアロンのDataGrip IDEに焦点を当てています。JetBrains IDE内のDatabase Tools and SQLプラグインを使用してTiDBに接続する手順は類似しています。また、JetBrains IDEからTiDBに接続する際の参考として、このドキュメントの手順に従うこともできます。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、以下が必要です：

-   [DataGrip **2023.2.1** 以降](https://www.jetbrains.com/datagrip/download/)または非コミュニティ版の[JetBrains](https://www.jetbrains.com/) IDE。
-   TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、以下のように作成できます：**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、以下のように作成できます：**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

TiDBのデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1.  [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2.  右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成が操作環境に一致することを確認します。

    -   **Endpoint Type**が`Public`に設定されていること
    -   **Branch**が`main`に設定されていること
    -   **Connect With**が`DataGrip`に設定されていること
    -   **Operating System**が環境に一致していること

4.  **Generate Password**をクリックしてランダムなパスワードを作成します。

    > **ヒント:**
    >
    > 以前にパスワードを作成している場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成できます。

5.  DataGripを起動し、接続を管理するためのプロジェクトを作成します。

    ![DataGripでプロジェクトを作成](/media/develop/datagrip-create-project.jpg)

6.  新しく作成したプロジェクトで、**Database Explorer**パネルの左上隅の\*\*+\*\*をクリックし、**Data Source** > **Other** > **TiDB**を選択します。

    ![DataGripでデータソースを選択](/media/develop/datagrip-data-source-select.jpg)

7.  TiDB Cloud接続ダイアログから接続文字列をコピーし、**URL**フィールドに貼り付けます。残りのパラメータは自動的に入力されます。例として以下のようになります：

    ![TiDB ServerlessのURLフィールドを構成](/media/develop/datagrip-url-paste.jpg)

    **Download missing driver files**の警告が表示された場合は、**Download**をクリックしてドライバファイルを取得します。

8.  **Test Connection**をクリックしてTiDB Serverlessクラスタへの接続を検証します。

    ![TiDB Serverlessクラスタへの接続をテスト](/media/develop/datagrip-test-connection.jpg)

9.  接続構成を保存するために**OK**をクリックします。

</div>
<div label="TiDB Dedicated">

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2.  右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3.  **どこからでもアクセスを許可**をクリックし、次に**TiDB クラスタ CA をダウンロード**をクリックして CA 証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated 標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4.  DataGrip を起動し、接続を管理するプロジェクトを作成します。

    ![DataGrip でプロジェクトを作成](/media/develop/datagrip-create-project.jpg)

5.  新しく作成したプロジェクトで、**データベースエクスプローラ**パネルの左上隅にある\*\*+\*\*をクリックし、**データソース** > **その他** > **TiDB**を選択します。

    ![DataGrip でデータソースを選択](/media/develop/datagrip-data-source-select.jpg)

6.  適切な接続文字列を**Data Source and Drivers**ウィンドウにコピーして貼り付けます。DataGrip のフィールドと TiDB Dedicated 接続文字列のマッピングは次のとおりです:

    | DataGrip フィールド | TiDB Dedicated 接続文字列 |
    | -------------- | -------------------- |
    | ホスト            | `{host}`             |
    | ポート            | `{port}`             |
    | ユーザー           | `{user}`             |
    | パスワード          | `{password}`         |

    例:

    ![TiDB Dedicated の接続パラメータを構成](/media/develop/datagrip-dedicated-connect.jpg)

7.  **SSH/SSL**タブをクリックし、**SSL を使用**チェックボックスを選択し、**CA ファイル**フィールドに CA 証明書のパスを入力します。

    ![TiDB Dedicated の CA を構成](/media/develop/datagrip-dedicated-ssl.jpg)

    **ドライバファイルのダウンロードが必要です**という警告が表示された場合は、**ダウンロード**をクリックしてドライバファイルを取得します。

8.  **詳細**タブをクリックし、**enabledTLSProtocols**パラメータを見つけて、その値を `TLSv1.2,TLSv1.3` に設定します。

    ![TiDB Dedicated の TLS を構成](/media/develop/datagrip-dedicated-advanced.jpg)

9.  **接続をテスト**をクリックして、TiDB Dedicated クラスタへの接続を検証します。

    ![TiDB Dedicated クラスタへの接続をテスト](/media/develop/datagrip-dedicated-test-connection.jpg)

10. **OK**をクリックして接続構成を保存します。

</div>
<div label="TiDB Self-Hosted">

1.  DataGrip を起動し、接続を管理するプロジェクトを作成します。

    ![DataGrip でプロジェクトを作成](/media/develop/datagrip-create-project.jpg)

2.  新しく作成したプロジェクトで、**データベースエクスプローラ**パネルの左上隅にある\*\*+\*\*をクリックし、**データソース** > **その他** > **TiDB**を選択します。

    ![DataGrip でデータソースを選択](/media/develop/datagrip-data-source-select.jpg)

3.  次の接続パラメータを構成します:

    -   **ホスト**: TiDB Self-Hosted クラスタの IP アドレスまたはドメイン名。
    -   **ポート**: TiDB Self-Hosted クラスタのポート番号。
    -   **ユーザー**: TiDB Self-Hosted クラスタに接続するためのユーザー名。
    -   **パスワード**: ユーザーのパスワード。

    例:

    ![TiDB Self-Hosted の接続パラメータを構成](/media/develop/datagrip-self-hosted-connect.jpg)

    **ドライバファイルのダウンロードが必要です**という警告が表示された場合は、**ダウンロード**をクリックしてドライバファイルを取得します。

4.  **接続をテスト**をクリックして、TiDB Self-Hosted クラスタへの接続を検証します。

    ![TiDB Self-Hosted クラスタへの接続をテスト](/media/develop/datagrip-self-hosted-test-connection.jpg)

5.  **OK**をクリックして接続構成を保存します。

</div>
</SimpleTab>

## 次の手順 {#next-steps}

-   [DataGripのドキュメント](https://www.jetbrains.com/help/datagrip/getting-started.html)を参照して、DataGripのさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などです。
-   [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
