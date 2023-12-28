---
title: Connect to TiDB with DBeaver
summary: Learn how to connect to TiDB using DBeaver Community.
---

# DBeaverを使用してTiDBに接続する {#connect-to-tidb-with-dbeaver}

TiDBはMySQL互換のデータベースであり、[DBeaver Community](https://dbeaver.io/download/)は開発者、データベース管理者、アナリスト、およびデータを扱うすべての人々のための無料のクロスプラットフォームデータベースツールです。

このチュートリアルでは、DBeaver Communityを使用してTiDBクラスターに接続する方法を学ぶことができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedに対応しています。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [DBeaver Community **23.0.3**以上](https://dbeaver.io/download/)
- TiDBクラスター

<CustomContent platform="tidb">

**TiDBクラスターをお持ちでない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターをお持ちでない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

TiDBに接続するには、選択したTiDBデプロイオプションに応じてTiDBクラスターに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスターの名前をクリックして、その概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が操作環境に一致することを確認します。

   - **Endpoint Type**が`Public`に設定されていること
   - **Branch**が`main`に設定されていること
   - **Connect With**が`DBeaver`に設定されていること
   - **Operating System**が環境に一致していること

4. ランダムなパスワードを作成するには、**Generate Password**をクリックします。

   > **Tip:**
   >
   > 以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成できます。

5. DBeaverを起動し、左上隅の**New Database Connection**をクリックします。**Connect to a database**ダイアログで、リストから**TiDB**を選択し、**Next**をクリックします。

   ![DBeaverでデータベースを選択する](/media/develop/dbeaver-select-database.jpg)

6. TiDB Cloud接続ダイアログから接続文字列をコピーします。DBeaverで、**Connect by**の**URL**を選択し、接続文字列を**URL**フィールドに貼り付けます。

7. **Authentication (Database Native)**セクションで、**Username**と**Password**を入力します。例は次のとおりです。

   ![TiDB Serverlessの接続設定を構成する](/media/develop/dbeaver-connection-settings-serverless.jpg)

8. TiDB Serverlessクラスターへの接続を検証するには、**Test Connection**をクリックします。

   **Download driver files**ダイアログが表示される場合は、ドライバーファイルを取得するために**Download**をクリックします。

   ![ドライバーファイルをダウンロードする](/media/develop/dbeaver-download-driver.jpg)

   接続テストが成功した場合、次のように**Connection test**ダイアログが表示されます。**OK**をクリックして閉じます。

   ![接続テストの結果](/media/develop/dbeaver-connection-test.jpg)

9. 接続設定を保存するには、**Finish**をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックし、**Download TiDB cluster CA** をクリックして CA 証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated 標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. DBeaver を起動し、左上隅の **New Database Connection** をクリックします。**Connect to a database** ダイアログで、リストから **TiDB** を選択し、**Next** をクリックします。

   ![DBeaver でデータベースを選択する](/media/develop/dbeaver-select-database.jpg)

5. 適切な接続文字列を DBeaver 接続パネルにコピーして貼り付けます。DBeaver フィールドと TiDB Dedicated 接続文字列のマッピングは次のとおりです。

   | DBeaver フィールド | TiDB Dedicated 接続文字列 |
   | ------------- | -------------------- |
   | Server Host   | `{host}`             |
   | Port          | `{port}`             |
   | Username      | `{user}`             |
   | Password      | `{password}`         |

   例は次のとおりです。

   ![TiDB Dedicated の接続設定を構成する](/media/develop/dbeaver-connection-settings-dedicated.jpg)

6. **Test Connection** をクリックして、TiDB Dedicated クラスタへの接続を検証します。

   **Download driver files** ダイアログが表示された場合は、ドライバーファイルを取得するために **Download** をクリックします。

   ![ドライバーファイルをダウンロードする](/media/develop/dbeaver-download-driver.jpg)

   接続テストが成功した場合、次のように **Connection test** ダイアログが表示されます。**OK** をクリックして閉じます。

   ![接続テストの結果](/media/develop/dbeaver-connection-test.jpg)

7. **Finish** をクリックして接続設定を保存します。

</div>
<div label="TiDB Self-Hosted">

1. DBeaver を起動し、左上隅の **New Database Connection** をクリックします。**Connect to a database** ダイアログで、リストから **TiDB** を選択し、**Next** をクリックします。

   ![DBeaver でデータベースを選択する](/media/develop/dbeaver-select-database.jpg)

2. 次の接続パラメータを構成します。

   - **Server Host**: TiDB Self-Hosted クラスタの IP アドレスまたはドメイン名。
   - **Port**: TiDB Self-Hosted クラスタのポート番号。
   - **Username**: TiDB Self-Hosted クラスタに接続するために使用するユーザー名。
   - **Password**: ユーザー名のパスワード。

   例は次のとおりです。

   ![TiDB Self-Hosted の接続設定を構成する](/media/develop/dbeaver-connection-settings-self-hosted.jpg)

3. **Test Connection** をクリックして、TiDB Self-Hosted クラスタへの接続を検証します。

   **Download driver files** ダイアログが表示された場合は、ドライバーファイルを取得するために **Download** をクリックします。

   ![ドライバーファイルをダウンロードする](/media/develop/dbeaver-download-driver.jpg)

   接続テストが成功した場合、次のように **Connection test** ダイアログが表示されます。**OK** をクリックして閉じます。

   ![接続テストの結果](/media/develop/dbeaver-connection-test.jpg)

4. **Finish** をクリックして接続設定を保存します。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

- [DBeaver のドキュメント](https://github.com/dbeaver/dbeaver/wiki)を参照して、DBeaver の使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章、例えば [データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータの取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および [SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)を通じて、TiDB アプリケーション開発のベストプラクティスを学びます。
- 専門の [TiDB 開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後、[TiDB 認定資格](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
