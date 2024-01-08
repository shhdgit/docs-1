---
title: Connect to TiDB with JetBrains DataGrip
summary: Learn how to connect to TiDB using JetBrains DataGrip. This tutorial also applies to the Database Tools and SQL plugin available in other JetBrains IDEs, such as IntelliJ, PhpStorm, and PyCharm.
---

# JetBrains DataGripでTiDBに接続する {#connect-to-tidb-with-jetbrains-datagrip}

TiDBはMySQL互換のデータベースであり、[JetBrains DataGrip](https://www.jetbrains.com/help/datagrip/getting-started.html)はデータベースとSQLのための強力な統合開発環境（IDE）です。このチュートリアルでは、DataGripを使用してTiDBクラスタに接続する手順について説明します。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

DataGripを次の2つの方法で使用できます：

- [DataGrip IDE](https://www.jetbrains.com/datagrip/download)としてスタンドアロンツールとして。
- IntelliJ、PhpStorm、およびPyCharmなどのJetBrains IDEでの[Database Tools and SQLプラグイン](https://www.jetbrains.com/help/idea/relational-databases.html)として。

このチュートリアルは、主にスタンドアロンのDataGrip IDEに焦点を当てています。JetBrains IDEでのDatabase Tools and SQLプラグインを使用してTiDBに接続する手順は類似しています。また、JetBrains IDEからTiDBに接続する際の参考として、このドキュメントの手順に従うこともできます。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です：

- [DataGrip **2023.2.1** 以降](https://www.jetbrains.com/datagrip/download/)または非コミュニティ版の[JetBrains](https://www.jetbrains.com/) IDE。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます：**

- （推奨）[TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます：**

- （推奨）[TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

選択したTiDBデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致することを確認します。

   - **Endpoint Type**が`Public`に設定されていること
   - **Branch**が`main`に設定されていること
   - **Connect With**が`DataGrip`に設定されていること
   - **Operating System**が環境に一致していること

4. **Generate Password**をクリックしてランダムなパスワードを作成します。

   > **Tip:**
   >
   > 以前にパスワードを作成している場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成できます。

5. DataGripを起動し、接続を管理するプロジェクトを作成します。

   ![DataGripでプロジェクトを作成](/media/develop/datagrip-create-project.jpg)

6. 新しく作成したプロジェクトで、**Database Explorer**パネルの左上隅の\*\*+\*\*をクリックし、**Data Source** > **Other** > **TiDB**を選択します。

   ![DataGripでデータソースを選択](/media/develop/datagrip-data-source-select.jpg)

7. TiDB Cloud接続ダイアログから接続文字列をコピーし、**URL**フィールドに貼り付けます。残りのパラメータは自動的に入力されます。例を以下に示します：

   ![TiDB ServerlessのURLフィールドを構成](/media/develop/datagrip-url-paste.jpg)

   **Download missing driver files**の警告が表示された場合は、**Download**をクリックしてドライバファイルを取得します。

8. **Test Connection**をクリックしてTiDB Serverlessクラスタへの接続を検証します。

   ![TiDB Serverlessクラスタへの接続をテスト](/media/develop/datagrip-test-connection.jpg)

9. 接続構成を保存するために**OK**をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、次に**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. DataGripを起動し、接続を管理するプロジェクトを作成します。

   ![DataGripでプロジェクトを作成](/media/develop/datagrip-create-project.jpg)

5. 新しく作成したプロジェクトで、**Database Explorer**パネルの左上隅の\*\*+\*\*をクリックし、**Data Source** > **Other** > **TiDB**を選択します。

   ![DataGripでデータソースを選択](/media/develop/datagrip-data-source-select.jpg)

6. 適切な接続文字列を**Data Source and Drivers**ウィンドウにコピーして貼り付けます。DataGripフィールドとTiDB Dedicated接続文字列のマッピングは次のとおりです：

   | DataGripフィールド | TiDB Dedicated接続文字列 |
   | ------------- | ------------------- |
   | Host          | `{host}`            |
   | Port          | `{port}`            |
   | User          | `{user}`            |
   | Password      | `{password}`        |

   例を以下に示します：

   ![TiDB Dedicatedの接続パラメータを構成](/media/develop/datagrip-dedicated-connect.jpg)

7. **SSH/SSL**タブをクリックし、**Use SSL**チェックボックスを選択し、**CA file**フィールドにCA証明書のパスを入力します。

   **Download missing driver files**の警告が表示された場合は、**Download**をクリックしてドライバファイルを取得します。

8. **Advanced**タブをクリックし、**enabledTLSProtocols**パラメータを見つけてその値を`TLSv1.2,TLSv1.3`に設定します。

   ![TiDB DedicatedのTLSを構成](/media/develop/datagrip-dedicated-advanced.jpg)

9. **Test Connection**をクリックしてTiDB Dedicatedクラスタへの接続を検証します。

   ![TiDB Dedicatedクラスタへの接続をテスト](/media/develop/datagrip-dedicated-test-connection.jpg)

10. 接続構成を保存するために**OK**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. DataGripを起動し、接続を管理するプロジェクトを作成します。

   ![DataGripでプロジェクトを作成](/media/develop/datagrip-create-project.jpg)

2. 新しく作成したプロジェクトで、**Database Explorer**パネルの左上隅の\*\*+\*\*をクリックし、**Data Source** > **Other** > **TiDB**を選択します。

   ![DataGripでデータソースを選択](/media/develop/datagrip-data-source-select.jpg)

3. 次の接続パラメータを構成します：

   - **Host**: TiDB Self-HostedクラスタのIPアドレスまたはドメイン名。
   - **Port**: TiDB Self-Hostedクラスタのポート番号。
   - **User**: TiDB Self-Hostedクラスタに接続するために使用するユーザー名。
   - **Password**: ユーザーのパスワード。

   例を以下に示します：

   ![TiDB Self-Hostedの接続パラメータを構成](/media/develop/datagrip-self-hosted-connect.jpg)

   **Download missing driver files**の警告が表示された場合は、**Download**をクリックしてドライバファイルを取得します。

4. **Test Connection**をクリックしてTiDB Self-Hostedクラスタへの接続を検証します。

   ![TiDB Self-Hostedクラスタへの接続をテスト](/media/develop/datagrip-self-hosted-test-connection.jpg)

5. 接続構成を保存するために**OK**をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

- [DataGripのドキュメント](https://www.jetbrains.com/help/datagrip/getting-started.html)からDataGripのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じてTiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などです。
- [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
