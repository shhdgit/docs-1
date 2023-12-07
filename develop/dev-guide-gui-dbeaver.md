---
title: Connect to TiDB with DBeaver
summary: Learn how to connect to TiDB using DBeaver Community.
---

# DBeaverを使用してTiDBに接続する {#connect-to-tidb-with-dbeaver}

TiDBはMySQL互換のデータベースであり、[DBeaver Community](https://dbeaver.io/download/)は開発者、データベース管理者、アナリスト、およびデータを扱うすべての人向けの無料のクロスプラットフォームデータベースツールです。

このチュートリアルでは、DBeaver Communityを使用してTiDBクラスタに接続する方法を学ぶことができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です:

-   [DBeaver Community **23.0.3** 以上](https://dbeaver.io/download/)
-   TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)を参照して、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を参照して、ローカルクラスタを作成します。

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
    -   **Connect With**が`DBeaver`に設定されていること
    -   **Operating System**が環境に一致していること

4.  ランダムなパスワードを作成するには、**Generate Password**をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために**Reset Password**をクリックできます。

5.  DBeaverを起動し、左上隅の**New Database Connection**をクリックします。**Connect to a database**ダイアログで、リストから**TiDB**を選択し、**Next**をクリックします。

    ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

6.  TiDB Cloud接続ダイアログから接続文字列をコピーします。DBeaverで、**URL**を**Connect by**に選択し、接続文字列を**URL**フィールドに貼り付けます。

7.  **Authentication (Database Native)**セクションで、**Username**と**Password**を入力します。例:

    ![TiDB Serverlessの接続設定を構成する](/media/develop/dbeaver-connection-settings-serverless.jpg)

8.  TiDB Serverlessクラスタへの接続を検証するには、**Test Connection**をクリックします。

    **Download driver files**ダイアログが表示された場合は、ドライバファイルを取得するために**Download**をクリックします。

    接続テストが成功した場合、以下のように**Connection test**ダイアログが表示されます。**OK**をクリックして閉じます。

    ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

9.  接続構成を保存するには、**Finish**をクリックします。

</div>
<div label="TiDB Dedicated">

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2.  右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3.  **どこからでもアクセスを許可**をクリックし、次に**TiDB クラスタ CA をダウンロード**をクリックして CA 証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated 標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4.  DBeaver を起動し、左上隅の**新しいデータベース接続**をクリックします。**データベースに接続**ダイアログで、リストから**TiDB**を選択し、次に**次へ**をクリックします。

    ![DBeaver でデータベースを選択](/media/develop/dbeaver-select-database.jpg)

5.  適切な接続文字列を DBeaver 接続パネルにコピーして貼り付けます。DBeaver のフィールドと TiDB Dedicated 接続文字列の対応は次のとおりです:

    | DBeaver のフィールド | TiDB Dedicated 接続文字列 |
    | -------------- | -------------------- |
    | サーバーホスト        | `{host}`             |
    | ポート            | `{port}`             |
    | ユーザー名          | `{user}`             |
    | パスワード          | `{password}`         |

    例:

    ![TiDB Dedicated の接続設定を構成](/media/develop/dbeaver-connection-settings-dedicated.jpg)

6.  **接続をテスト**をクリックして、TiDB Dedicated クラスタへの接続を検証します。

    **ドライバファイルをダウンロード**ダイアログが表示された場合は、ドライバファイルを取得するために**ダウンロード**をクリックします。

    接続テストが成功した場合、**接続テスト**ダイアログが次のように表示されます。**OK**をクリックして閉じます。

    ![接続テストの結果](/media/develop/dbeaver-connection-test.jpg)

7.  接続構成を保存するには、**完了**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1.  DBeaver を起動し、左上隅の**新しいデータベース接続**をクリックします。**データベースに接続**ダイアログで、リストから**TiDB**を選択し、次に**次へ**をクリックします。

    ![DBeaver でデータベースを選択](/media/develop/dbeaver-select-database.jpg)

2.  次の接続パラメータを構成します:

    -   **サーバーホスト**: TiDB Self-Hosted クラスタの IP アドレスまたはドメイン名。
    -   **ポート**: TiDB Self-Hosted クラスタのポート番号。
    -   **ユーザー名**: TiDB Self-Hosted クラスタに接続するためのユーザー名。
    -   **パスワード**: ユーザー名のパスワード。

    例:

    ![TiDB Self-Hosted の接続設定を構成](/media/develop/dbeaver-connection-settings-self-hosted.jpg)

3.  **接続をテスト**をクリックして、TiDB Self-Hosted クラスタへの接続を検証します。

    **ドライバファイルをダウンロード**ダイアログが表示された場合は、ドライバファイルを取得するために**ダウンロード**をクリックします。

    接続テストが成功した場合、**接続テスト**ダイアログが次のように表示されます。**OK**をクリックして閉じます。

    ![接続テストの結果](/media/develop/dbeaver-connection-test.jpg)

4.  接続構成を保存するには、**完了**をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

-   [DBeaver のドキュメント](https://github.com/dbeaver/dbeaver/wiki)から DBeaver のさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDB アプリケーション開発のベストプラクティスを学びます。例: [データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
-   プロフェッショナルな[TiDB 開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDB 認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
