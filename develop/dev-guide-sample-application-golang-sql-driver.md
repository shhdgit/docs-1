---
title: Connect to TiDB with Go-MySQL-Driver
summary: Learn how to connect to TiDB using Go-MySQL-Driver. This tutorial gives Golang sample code snippets that work with TiDB using Go-MySQL-Driver.
---

# Go-MySQL-Driverを使用してTiDBに接続する {#connect-to-tidb-with-go-mysql-driver}

TiDBはMySQL互換のデータベースであり、[Go-MySQL-Driver](https://github.com/go-sql-driver/mysql)は[database/sql](https://pkg.go.dev/database/sql)インターフェースのためのMySQL実装です。

このチュートリアルでは、TiDBとGo-MySQL-Driverを使用して次のタスクを実行する方法を学ぶことができます:

-   環境をセットアップする。
-   Go-MySQL-Driverを使用してTiDBクラスタに接続する。
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

**TiDBクラスタを持っていない場合、以下のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合、以下のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-golang-sql-driver-quickstart.git
cd tidb-golang-sql-driver-quickstart
```

### ステップ2：接続情報を構成する {#step-2-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDBデプロイメントオプションに応じて行います。

<SimpleTab>
<div label="TiDBサーバーレス">

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象クラスタの名前をクリックして概要ページに移動します。

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
    > もし以前にパスワードを作成している場合は、元のパスワードを使用するか、新しいパスワードを生成するために **Reset Password** をクリックすることができます。

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

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象クラスタの名前をクリックして概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  **Allow Access from Anywhere** をクリックし、その後 **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

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

    接続パラメータでプレースホルダ `{}` を置き換え、`USE_SSL` を `false` に設定してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3.  `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ3：コードを実行して結果を確認する {#step-3-run-the-code-and-check-the-result}

1.  次のコマンドを実行して、サンプルコードを実行します：

    ```shell
    make
    ```

2.  出力が一致しているかを確認するために [Expected-Output.txt](https://github.com/tidb-samples/tidb-golang-sql-driver-quickstart/blob/main/Expected-Output.txt) を確認してください。

## サンプルコードスニペット {#sample-code-snippets}

あなた自身のアプリケーション開発を完了するために、以下のサンプルコードスニペットを参照してください。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-golang-sql-driver-quickstart](https://github.com/tidb-samples/tidb-golang-sql-driver-quickstart) リポジトリをチェックしてください。

### TiDBへの接続 {#connect-to-tidb}

```golang
func openDB(driverName string, runnable func(db *sql.DB)) {
    dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&tls=%s",
        ${tidb_user}, ${tidb_password}, ${tidb_host}, ${tidb_port}, ${tidb_db_name}, ${use_ssl})
    db, err := sql.Open(driverName, dsn)
    if err != nil {
        panic(err)
    }
    defer db.Close()

    runnable(db)
}
```

この機能を使用する際には、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`をTiDBクラスターの実際の値で置き換える必要があります。TiDB Serverlessでは安全な接続が必要です。したがって、`${use_ssl}`の値を`true`に設定する必要があります。

### データの挿入 {#insert-data}

```golang
openDB("mysql", func(db *sql.DB) {
    insertSQL = "INSERT INTO player (id, coins, goods) VALUES (?, ?, ?)"
    _, err := db.Exec(insertSQL, "id", 1, 1)

    if err != nil {
        panic(err)
    }
})
```

[データを挿入](/develop/dev-guide-insert-data.md) に関する詳細は、詳細情報を参照してください。

### データのクエリ {#query-data}

// translated content end

```golang
openDB("mysql", func(db *sql.DB) {
    selectSQL = "SELECT id, coins, goods FROM player WHERE id = ?"
    rows, err := db.Query(selectSQL, "id")
    if err != nil {
        panic(err)
    }

    // This line is extremely important!
    defer rows.Close()

    id, coins, goods := "", 0, 0
    if rows.Next() {
        err = rows.Scan(&id, &coins, &goods)
        if err == nil {
            fmt.Printf("player id: %s, coins: %d, goods: %d\n", id, coins, goods)
        }
    }
})
```

[クエリデータ](/develop/dev-guide-get-data-from-single-table.md)を参照して、詳細情報をご覧ください。

### データを更新 {#update-data}

```golang
openDB("mysql", func(db *sql.DB) {
    updateSQL = "UPDATE player set goods = goods + ?, coins = coins + ? WHERE id = ?"
    _, err := db.Exec(updateSQL, 1, -1, "id")

    if err != nil {
        panic(err)
    }
})
```

[データの更新](/develop/dev-guide-update-data.md)に関する詳細は、こちらを参照してください。

### データの削除 {#delete-data}

```golang
openDB("mysql", func(db *sql.DB) {
    deleteSQL = "DELETE FROM player WHERE id=?"
    _, err := db.Exec(deleteSQL, "id")

    if err != nil {
        panic(err)
    }
})
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート {#useful-notes}

### ドライバーまたはORMフレームワークを使用しますか？ {#using-driver-or-orm-framework}

Golangドライバーはデータベースへの低レベルなアクセスを提供しますが、開発者が次のことを行う必要があります：

-   手動でデータベース接続を確立および解除する。
-   データベーストランザクションを手動で管理する。
-   データ行をデータオブジェクトに手動でマッピングする。

複雑なSQL文を書く必要がない限り、[GORM](/develop/dev-guide-sample-application-golang-gorm.md)などのORMフレームワークを使用することをお勧めします。これにより、次のことができます：

-   接続とトランザクションの管理における[定型コード](https://en.wikipedia.org/wiki/Boilerplate_code)を削減する。
-   複数のSQL文の代わりにデータオブジェクトでデータを操作する。

## 次のステップ {#next-steps}

-   [Go-MySQL-Driverのドキュメント](https://github.com/go-sql-driver/mysql/blob/master/README.md)でGo-MySQL-Driverのさらなる使用法を学ぶ。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章でTiDBアプリケーション開発のベストプラクティスを学ぶ。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
-   プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通して学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得する。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
