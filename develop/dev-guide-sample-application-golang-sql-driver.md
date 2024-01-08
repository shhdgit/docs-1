---
title: Connect to TiDB with Go-MySQL-Driver
summary: Learn how to connect to TiDB using Go-MySQL-Driver. This tutorial gives Golang sample code snippets that work with TiDB using Go-MySQL-Driver.
---

# Go-MySQL-Driverを使用してTiDBに接続 {#connect-to-tidb-with-go-mysql-driver}

TiDBはMySQL互換のデータベースであり、[Go-MySQL-Driver](https://github.com/go-sql-driver/mysql)は[database/sql](https://pkg.go.dev/database/sql)インターフェース用のMySQL実装です。

このチュートリアルでは、TiDBとGo-MySQL-Driverを使用して次のタスクを実行する方法を学ぶことができます。

- 環境を設定します。
- Go-MySQL-Driverを使用してTiDBクラスタに接続します。
- アプリケーションをビルドして実行します。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと共に動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Go](https://go.dev/) **1.20** 以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md)して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタをデプロイ](/production-deployment-using-tiup.md)して、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md)して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)して、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続 {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1：サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-golang-sql-driver-quickstart.git
cd tidb-golang-sql-driver-quickstart
```

### ステップ2：接続情報を構成する {#step-2-configure-connection-information}

選択したTiDBデプロイメントオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致することを確認します。

   - **エンドポイントタイプ** が `Public` に設定されていること

   - **Branch** が `main` に設定されていること

   - **Connect With** が `General` に設定されていること

   - **オペレーティングシステム** が環境に一致していること

   > **ヒント:**
   >
   > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えます。

4. ランダムなパスワードを作成するには、**Generate Password** をクリックします。

   > **ヒント:**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**Reset Password** をクリックして新しいパスワードを生成できます。

5. 次のコマンドを実行して、`.env.example` をコピーして `.env` に名前を変更します。

   ```shell
   cp .env.example .env
   ```

6. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
   TIDB_PORT='4000'
   TIDB_USER='{user}'  # 例: xxxxxx.root
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   USE_SSL='true'
   ```

   接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えるようにしてください。

   TiDB Serverlessでは安全な接続が必要です。そのため、`USE_SSL` の値を `true` に設定する必要があります。

7. `.env` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **どこからでもアクセスを許可** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. 次のコマンドを実行して、`.env.example` をコピーして `.env` に名前を変更します。

   ```shell
   cp .env.example .env
   ```

5. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
   TIDB_PORT='4000'
   TIDB_USER='{user}'  # 例: root
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   USE_SSL='false'
   ```

   接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えるようにしてください。

6. `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`.env.example` をコピーして `.env` に名前を変更します。

   ```shell
   cp .env.example .env
   ```

2. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{host}'
   TIDB_PORT='4000'
   TIDB_USER='root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   USE_SSL='false'
   ```

   接続パラメータをプレースホルダ `{}` で置き換え、`USE_SSL` を `false` に設定するようにしてください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` で、パスワードは空です。

3. `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ3：コードを実行して結果を確認する {#step-3-run-the-code-and-check-the-result}

1. 次のコマンドを実行して、サンプルコードを実行します。

   ```shell
   make
   ```

2. 出力が一致するかどうかを確認するために、[Expected-Output.txt](https://github.com/tidb-samples/tidb-golang-sql-driver-quickstart/blob/main/Expected-Output.txt) を確認します。

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-golang-sql-driver-quickstart](https://github.com/tidb-samples/tidb-golang-sql-driver-quickstart) リポジトリを確認してください。

### TiDBに接続 {#connect-to-tidb}

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

この機能を使用する場合、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`をTiDBクラスタの実際の値で置き換える必要があります。 TiDB Serverlessでは安全な接続が必要です。したがって、`${use_ssl}`の値を`true`に設定する必要があります。

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

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

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

詳細については、[Query data](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

```golang
openDB("mysql", func(db *sql.DB) {
    updateSQL = "UPDATE player set goods = goods + ?, coins = coins + ? WHERE id = ?"
    _, err := db.Exec(updateSQL, 1, -1, "id")

    if err != nil {
        panic(err)
    }
})
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

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

Golangドライバーはデータベースへの低レベルなアクセスを提供しますが、開発者が次のことを行う必要があります。

- 手動でデータベース接続を確立および解放する。
- 手動でデータベーストランザクションを管理する。
- 手動でデータ行をデータオブジェクトにマッピングする。

複雑なSQLステートメントを書く必要がない場合は、[ORM](https://en.wikipedia.org/w/index.php?title=Object-relational_mapping)フレームワークを使用することをお勧めします。たとえば、[GORM](/develop/dev-guide-sample-application-golang-gorm.md)などの開発に役立ちます。

- 接続とトランザクションの管理のための[ボイラープレートコード](https://en.wikipedia.org/wiki/Boilerplate_code)を削減します。
- 複数のSQLステートメントの代わりにデータオブジェクトでデータを操作します。

## 次のステップ {#next-steps}

- [Go-MySQL-Driverのドキュメント](https://github.com/go-sql-driver/mysql/blob/master/README.md)からGo-MySQL-Driverのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章で、TiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
