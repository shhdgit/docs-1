---
title: Connect to TiDB with DBeaver
summary: Learn how to connect to TiDB using DBeaver Community.
---

# DBeaverでTiDBに接続する {#connect-to-tidb-with-dbeaver}

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

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成します。
- ローカルクラスタを作成するには、[ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)を参照してください。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成します。
- ローカルクラスタを作成するには、[ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を参照してください。

</CustomContent>

## TiDBに接続する {#connect-to-tidb}

選択したTiDBデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致することを確認してください。

   - **エンドポイントタイプ**が`Public`に設定されていること
   - **ブランチ**が`main`に設定されていること
   - **接続先**が`DBeaver`に設定されていること
   - **オペレーティングシステム**が環境に一致していること

4. ランダムなパスワードを作成するには、**パスワードを生成**をクリックします。

   > **ヒント:**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**パスワードをリセット**をクリックして新しいパスワードを生成できます。

5. DBeaverを起動し、左上隅の**新しいデータベース接続**をクリックします。**データベースに接続**ダイアログで、リストから**TiDB**を選択し、**次へ**をクリックします。

   ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

6. TiDB Cloud接続ダイアログから接続文字列をコピーします。DBeaverで、**URL**を**Connect by**に選択し、接続文字列を**URL**フィールドに貼り付けます。

7. **認証（データベースネイティブ）**セクションで、**ユーザー名**と**パスワード**を入力します。例は次のとおりです:

   ![TiDB Serverlessの接続設定を構成](/media/develop/dbeaver-connection-settings-serverless.jpg)

8. TiDB Serverlessクラスタへの接続を検証するには、**接続のテスト**をクリックします。

   **ドライバーファイルのダウンロード**ダイアログが表示された場合は、**ダウンロード**をクリックしてドライバーファイルを取得します。

   接続テストが成功した場合、次のように**接続テスト**ダイアログが表示されます。**OK**をクリックして閉じます。

   ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

9. 接続構成を保存するには、**完了**をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. **どこからでもアクセスを許可**をクリックし、次に**TiDBクラスタCAをダウンロード**をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. DBeaverを起動し、左上隅の**新しいデータベース接続**をクリックします。**データベースに接続**ダイアログで、リストから**TiDB**を選択し、**次へ**をクリックします。

   ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

5. 適切な接続文字列をDBeaver接続パネルにコピーして貼り付けます。DBeaverフィールドとTiDB Dedicated接続文字列のマッピングは次のとおりです:

   | DBeaverフィールド | TiDB Dedicated接続文字列 |
   | ------------ | ------------------- |
   | サーバーホスト      | `{host}`            |
   | ポート          | `{port}`            |
   | ユーザー名        | `{user}`            |
   | パスワード        | `{password}`        |

   例は次のとおりです:

   ![TiDB Dedicatedの接続設定を構成](/media/develop/dbeaver-connection-settings-dedicated.jpg)

6. TiDB Dedicatedクラスタへの接続を検証するには、**接続のテスト**をクリックします。

   **ドライバーファイルのダウンロード**ダイアログが表示された場合は、**ダウンロード**をクリックしてドライバーファイルを取得します。

   接続テストが成功した場合、次のように**接続テスト**ダイアログが表示されます。**OK**をクリックして閉じます。

   ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

7. 接続構成を保存するには、**完了**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. DBeaverを起動し、左上隅の**新しいデータベース接続**をクリックします。**データベースに接続**ダイアログで、リストから**TiDB**を選択し、**次へ**をクリックします。

   ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

2. 次の接続パラメータを構成します:

   - **サーバーホスト**: TiDB Self-HostedクラスタのIPアドレスまたはドメイン名。
   - **ポート**: TiDB Self-Hostedクラスタのポート番号。
   - **ユーザー名**: TiDB Self-Hostedクラスタに接続するために使用するユーザー名。
   - **パスワード**: ユーザー名のパスワード。

   例は次のとおりです:

   ![TiDB Self-Hostedの接続設定を構成](/media/develop/dbeaver-connection-settings-self-hosted.jpg)

3. TiDB Self-Hostedクラスタへの接続を検証するには、**接続のテスト**をクリックします。

   **ドライバーファイルのダウンロード**ダイアログが表示された場合は、**ダウンロード**をクリックしてドライバーファイルを取得します。

   接続テストが成功した場合、次のように**接続テスト**ダイアログが表示されます。**OK**をクリックして閉じます。

   ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

4. 接続構成を保存するには、**完了**をクリックします。

</div>
</SimpleTab>

## 次のステップ {#next-steps}

- [DBeaverのドキュメント](https://github.com/dbeaver/dbeaver/wiki)からDBeaverのさらなる使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例: [データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
