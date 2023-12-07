---
title: Connect to TiDB with GORM
summary: Learn how to connect to TiDB using GORM. This tutorial gives Golang sample code snippets that work with TiDB using GORM.
---

# GORMを使用してTiDBに接続する {#connect-to-tidb-with-gorm}

TiDBはMySQL互換のデータベースであり、[GORM](https://gorm.io/index.html)はGolang向けの人気のあるオープンソースのORMフレームワークです。GORMは`AUTO_RANDOM`などのTiDBの機能に適応し、[デフォルトのデータベースオプションとしてTiDBをサポート](https://gorm.io/docs/connecting_to_the_database.html#TiDB)しています。

このチュートリアルでは、TiDBとGORMを使用して次のタスクを達成する方法を学ぶことができます:

-   環境をセットアップする。
-   GORMを使用してTiDBクラスタに接続する。
-   アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、以下が必要です:

-   [Go](https://go.dev/) **1.20** 以上。
-   [Git](https://git-scm.com/downloads)。
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

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-golang-gorm-quickstart.git
cd tidb-golang-gorm-quickstart
```

### ステップ2：接続情報を構成する {#step-2-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDBデプロイメントオプションに応じて行います。

<SimpleTab>
<div label="TiDBサーバーレス">

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの名前をクリックして概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成が、操作環境に一致することを確認します。

    -   **エンドポイントタイプ** が `Public` に設定されていること

    -   **Branch** が `main` に設定されていること

    -   **Connect With** が `General` に設定されていること

    -   **Operating System** が環境に一致していること

    > **ヒント:**
    >
    > もしプログラムがWindows Subsystem for Linux (WSL)で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4.  **Generate Password** をクリックしてランダムなパスワードを作成します。

    > **ヒント:**
    >
    > もし以前にパスワードを作成している場合は、元のパスワードを使用するか、新しいパスワードを生成するために **Reset Password** をクリックします。

5.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

6.  対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例として以下のようになります：

    ```dotenv
    TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: xxxxxx.root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    USE_SSL='true'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えるようにしてください。

    TiDBサーバーレスでは安全な接続が必要です。そのため、`USE_SSL` の値を `true` に設定する必要があります。

7.  `.env` ファイルを保存します。

</div>
<div label="TiDB専用">

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの名前をクリックして概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  **どこからでもアクセスを許可** をクリックし、その後 **TiDBクラスタCAをダウンロード** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB専用標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

5.  対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例として以下のようになります：

    ```dotenv
    TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    USE_SSL='false'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えるようにしてください。

6.  `.env` ファイルを保存します。

</div>
<div label="TiDBセルフホスト">

1.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

2.  対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例として以下のようになります：

    ```dotenv
    TIDB_HOST='{host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    USE_SSL='false'
    ```

    接続パラメータをプレースホルダ `{}` で置き換え、`USE_SSL` を `false` に設定するようにしてください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3.  `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ3：コードを実行して結果を確認する {#step-3-run-the-code-and-check-the-result}

1.  次のコマンドを実行してサンプルコードを実行します：

    ```shell
    make
    ```

2.  出力が一致しているかを確認するために [Expected-Output.txt](https://github.com/tidb-samples/tidb-golang-gorm-quickstart/blob/main/Expected-Output.txt) を確認してください。

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-golang-gorm-quickstart](https://github.com/tidb-samples/tidb-golang-gorm-quickstart) リポジトリをチェックしてください。

### TiDBへの接続 {#connect-to-tidb}

```golang
func createDB() *gorm.DB {
    dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&tls=%s",
        ${tidb_user}, ${tidb_password}, ${tidb_host}, ${tidb_port}, ${tidb_db_name}, ${use_ssl})

    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info),
    })
    if err != nil {
        panic(err)
    }

    return db
}
```

この機能を使用する際には、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`をTiDBクラスターの実際の値で置き換える必要があります。TiDB Serverlessでは安全な接続が必要です。したがって、`${use_ssl}`の値を`true`に設定する必要があります。

### データの挿入 {#insert-data}

```golang
db.Create(&Player{ID: "id", Coins: 1, Goods: 1})
```

[データを挿入](/develop/dev-guide-insert-data.md) に関する詳細は、参照してください。

### データのクエリ {#query-data}

```golang
var queryPlayer Player
db.Find(&queryPlayer, "id = ?", "id")
```

詳細については、[クエリデータ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

```golang
db.Save(&Player{ID: "id", Coins: 100, Goods: 1})
```

[データの更新](/develop/dev-guide-update-data.md)に関する詳細は、こちらを参照してください。

### データの削除 {#delete-data}

```golang
db.Delete(&Player{ID: "id"})
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ {#next-steps}

-   [GORMのドキュメント](https://gorm.io/docs/index.html)および[GORMのTiDBセクションのドキュメント](https://gorm.io/docs/connecting_to_the_database.html#TiDB)から、GORMのさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
-   [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定資格](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
