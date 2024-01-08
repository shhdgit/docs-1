---
title: Connect to TiDB with DBeaver
summary: Learn how to connect to TiDB using DBeaver Community.
---

# DBeaverを使用してTiDBに接続する {#connect-to-tidb-with-dbeaver}

TiDBはMySQL互換のデータベースであり、[DBeaver Community](https://dbeaver.io/download/)は、開発者、データベース管理者、アナリスト、およびデータを扱うすべての人向けの無料のクロスプラットフォームデータベースツールです。

このチュートリアルでは、DBeaver Communityを使用してTiDBクラスタに接続する方法を学ぶことができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [DBeaver Community **23.0.3** 以上](https://dbeaver.io/download/)
- TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- ローカルクラスタを作成するには、[ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従います。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- ローカルクラスタを作成するには、[ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従います。

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
   - **Connect With**が`DBeaver`に設定されていること
   - **Operating System**が環境に一致していること

4. ランダムなパスワードを作成するには、**Generate Password**をクリックします。

   > **Tip:**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成できます。

5. DBeaverを起動し、左上隅の**New Database Connection**をクリックします。**データベースに接続**ダイアログで、リストから**TiDB**を選択し、**Next**をクリックします。

   ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

6. TiDB Cloud接続ダイアログから接続文字列をコピーします。DBeaverで、**URL**を**Connect by**に選択し、接続文字列を**URL**フィールドに貼り付けます。

7. **Authentication (Database Native)**セクションで、**Username**と**Password**を入力します。例は次のとおりです:

   ![TiDB Serverlessの接続設定を構成](/media/develop/dbeaver-connection-settings-serverless.jpg)

8. TiDB Serverlessクラスタへの接続を検証するには、**Test Connection**をクリックします。

   **Download driver files**ダイアログが表示された場合は、ドライバファイルを取得するには**Download**をクリックします。

   接続テストが成功した場合、**Connection test**ダイアログが次のように表示されます。**OK**をクリックして閉じます。

   ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

9. 接続構成を保存するには、**Finish**をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. DBeaverを起動し、左上隅の**New Database Connection**をクリックします。**データベースに接続**ダイアログで、リストから**TiDB**を選択し、**Next**をクリックします。

   ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

5. 適切な接続文字列をDBeaver接続パネルにコピーして貼り付けます。DBeaverフィールドとTiDB Dedicated接続文字列のマッピングは次のとおりです:

   | DBeaverフィールド | TiDB Dedicated接続文字列 |
   | ------------ | ------------------- |
   | Server Host  | `{host}`            |
   | Port         | `{port}`            |
   | Username     | `{user}`            |
   | Password     | `{password}`        |

   例は次のとおりです:

   ![TiDB Dedicatedの接続設定を構成](/media/develop/dbeaver-connection-settings-dedicated.jpg)

6. TiDB Dedicatedクラスタへの接続を検証するには、**Test Connection**をクリックします。

   **Download driver files**ダイアログが表示された場合は、ドライバファイルを取得するには**Download**をクリックします。

   接続テストが成功した場合、**Connection test**ダイアログが次のように表示されます。**OK**をクリックして閉じます。

   ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

7. 接続構成を保存するには、**Finish**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. DBeaverを起動し、左上隅の**New Database Connection**をクリックします。**データベースに接続**ダイアログで、リストから**TiDB**を選択し、**Next**をクリックします。

   ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

2. 次の接続パラメータを構成します:

   - **Server Host**: TiDB Self-HostedクラスタのIPアドレスまたはドメイン名。
   - **Port**: TiDB Self-Hostedクラスタのポート番号。
   - **Username**: TiDB Self-Hostedクラスタに接続するために使用するユーザー名。
   - **Password**: ユーザー名のパスワード。

   例は次のとおりです:

   ![TiDB Self-Hostedの接続設定を構成](/media/develop/dbeaver-connection-settings-self-hosted.jpg)

3. TiDB Self-Hostedクラスタへの接続を検証するには、**Test Connection**をクリックします。

   **Download driver files**ダイアログが表示された場合は、ドライバファイルを取得するには**Download**をクリックします。

   接続テストが成功した場合、**Connection test**ダイアログが次のように表示されます。**OK**をクリックして閉じます。

   ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

4. 接続構成を保存するには、**Finish**をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

- [DBeaverのドキュメント](https://github.com/dbeaver/dbeaver/wiki)からDBeaverのさらなる使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章で、TiDBアプリケーション開発のベストプラクティスを学びます。例: [データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
