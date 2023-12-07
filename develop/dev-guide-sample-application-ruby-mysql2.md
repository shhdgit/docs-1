---
title: Connect to TiDB with mysql2
summary: Learn how to connect to TiDB using Ruby mysql2. This tutorial gives Ruby sample code snippets that work with TiDB using mysql2 gem.
---

# TiDBとmysql2を使用して接続する {#connect-to-tidb-with-mysql2}

TiDBはMySQL互換のデータベースであり、[mysql2](https://github.com/brianmario/mysql2)はRuby向けの最も人気のあるMySQLドライバの1つです。

このチュートリアルでは、TiDBとmysql2を使用して次のタスクを実行する方法を学ぶことができます:

-   環境をセットアップする。
-   mysql2を使用してTiDBクラスタに接続する。
-   アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です:

-   あなたのマシンにインストールされている[Ruby](https://www.ruby-lang.org/en/) >= 3.0
-   あなたのマシンにインストールされている[Bundler](https://bundler.io/)
-   あなたのマシンにインストールされている[Git](https://git-scm.com/downloads)
-   実行中のTiDBクラスタ

**TiDBクラスタを持っていない場合は、次のように作成できます:**

<CustomContent platform="tidb">

-   (推奨) [TiDB Serverlessクラスタを作成する](/develop/dev-guide-build-cluster-in-cloud.md) を参照して、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタをデプロイする](/quick-start-with-tidb.md#deploy-a-local-test-cluster) または [本番用TiDBクラスタをデプロイする](/production-deployment-using-tiup.md) を参照して、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

-   (推奨) [TiDB Serverlessクラスタを作成する](/develop/dev-guide-build-cluster-in-cloud.md) を参照して、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタをデプロイする](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster) または [本番用TiDBクラスタをデプロイする](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) を参照して、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart.git
cd tidb-ruby-mysql2-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

以下のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysql2`および`dotenv`を含む）をインストールしてください：

```shell
bundle install
```

<details>
<summary><b>既存のプロジェクトの依存関係をインストールする</b></summary>

既存のプロジェクトに対して、以下のコマンドを実行してパッケージをインストールしてください：

```shell
bundle add mysql2 dotenv
```

</details>

### ステップ3：接続情報を構成する {#step-3-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDB展開オプションに応じて行います。

<SimpleTab>
<div label="TiDB Serverless">

1.  [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成が操作環境に一致することを確認します。

    -   **エンドポイントタイプ** が `Public` に設定されていること。
    -   **ブランチ** が `main` に設定されていること。
    -   **Connect With** が `General` に設定されていること。
    -   **オペレーティングシステム** がアプリケーションを実行するオペレーティングシステムと一致していること。

4.  パスワードをまだ設定していない場合は、**Generate Password** をクリックしてランダムなパスワードを生成します。

5.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

6.  `.env` ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータで対応するプレースホルダ `<>` を置き換えます：

    ```dotenv
    DATABASE_HOST=<ホスト>
    DATABASE_PORT=4000
    DATABASE_USER=<ユーザー>
    DATABASE_PASSWORD=<パスワード>
    DATABASE_NAME=test
    DATABASE_ENABLE_SSL=true
    ```

    > **注意**
    >
    > TiDB Serverlessの場合、パブリックエンドポイントを使用する場合は、`DATABASE_ENABLE_SSL` を介してTLS接続を有効にする必要があります。

7.  `.env` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1.  [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

5.  `.env` ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータで対応するプレースホルダ `<>` を置き換えます：

    ```dotenv
    DATABASE_HOST=<ホスト>
    DATABASE_PORT=4000
    DATABASE_USER=<ユーザー>
    DATABASE_PASSWORD=<パスワード>
    DATABASE_NAME=test
    DATABASE_ENABLE_SSL=true
    DATABASE_SSL_CA=<ダウンロードしたSSL_CA_パス>
    ```

    > **注意**
    >
    > TiDB Dedicatedクラスタにパブリックエンドポイントを使用して接続する場合は、TLS接続を有効にすることを推奨します。
    >
    > TLS接続を有効にするには、`DATABASE_ENABLE_SSL` を `true` に変更し、接続ダイアログからダウンロードしたCA証明書のファイルパスを指定するために `DATABASE_SSL_CA` を使用します。

6.  `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

2.  `.env` ファイルを編集し、環境変数を次のように設定し、対応するプレースホルダ `<>` を自分自身のTiDB接続情報で置き換えます：

    ```dotenv
    DATABASE_HOST=<ホスト>
    DATABASE_PORT=4000
    DATABASE_USER=<ユーザー>
    DATABASE_PASSWORD=<パスワード>
    DATABASE_NAME=test
    ```

    もしTiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3.  `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

次のコマンドを実行して、サンプルコードを実行します：

```shell
ruby app.rb
```

If the connection is successful, the console will output the version of the TiDB cluster as follows:
接続が成功した場合、コンソールにはTiDBクラスターのバージョンが以下のように出力されます。

    🔌 Connected to TiDB cluster! (TiDB version: 5.7.25-TiDB-v7.1.0)
    ⏳ Loading sample game data...
    ✅ Loaded sample game data.

    🆕 Created a new player with ID 12.
    ℹ️ Got Player 12: Player { id: 12, coins: 100, goods: 100 }
    🔢 Added 50 coins and 50 goods to player 12, updated 1 row.
    🚮 Deleted 1 player data.

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-ruby-mysql2-quickstart](https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart) リポジトリをチェックしてください。

### 接続オプションを使用してTiDBに接続する {#connect-to-tidb-with-connection-options}

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続を確立します。

```ruby
require 'dotenv/load'
require 'mysql2'
Dotenv.load # Load the environment variables from the .env file

options = {
  host: ENV['DATABASE_HOST'] || '127.0.0.1',
  port: ENV['DATABASE_PORT'] || 4000,
  username: ENV['DATABASE_USER'] || 'root',
  password: ENV['DATABASE_PASSWORD'] || '',
  database: ENV['DATABASE_NAME'] || 'test'
}
options.merge(ssl_mode: :verify_identity) unless ENV['DATABASE_ENABLE_SSL'] == 'false'
options.merge(sslca: ENV['DATABASE_SSL_CA']) if ENV['DATABASE_SSL_CA']
client = Mysql2::Client.new(options)
```

**注意**

TiDB Serverlessでは、パブリックエンドポイントを使用する場合、TLS接続を`DATABASE_ENABLE_SSL`経由で有効にする必要がありますが、`DATABASE_SSL_CA`を介してSSL CA証明書を指定する必要はありません。なぜなら、mysql2 gemが特定の順序で既存のCA証明書を検索し、ファイルが見つかるまで検索を続けるからです。

### データの挿入 {#insert-data}

次のクエリは、2つのフィールドを持つ単一のプレイヤーを作成し、`last_insert_id`を返します。

```ruby
def create_player(client, coins, goods)
  result = client.query(
    "INSERT INTO players (coins, goods) VALUES (#{coins}, #{goods});"
  )
  client.last_id
end
```

[データの挿入](/develop/dev-guide-insert-data.md)に関する詳細は、詳細情報を参照してください。

### データのクエリ {#query-data}

次のクエリは、特定のプレイヤーのレコードをIDで返します:

```ruby
def get_player_by_id(client, id)
  result = client.query(
    "SELECT id, coins, goods FROM players WHERE id = #{id};"
  )
  result.first
end
```

詳細については、[クエリデータ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、特定のプレイヤーのレコードをIDで更新しました。

```ruby
def update_player(client, player_id, inc_coins, inc_goods)
  result = client.query(
    "UPDATE players SET coins = coins + #{inc_coins}, goods = goods + #{inc_goods} WHERE id = #{player_id};"
  )
  client.affected_rows
end
```

[データの更新](/develop/dev-guide-update-data.md)に関する詳細は、こちらを参照してください。

### データの削除 {#delete-data}

以下のクエリは特定のプレイヤーのレコードを削除します：

```ruby
def delete_player_by_id(client, id)
  result = client.query(
    "DELETE FROM players WHERE id = #{id};"
  )
  client.affected_rows
end
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md) を参照してください。

## ベストプラクティス {#best-practices}

デフォルトでは、mysql2 gem は特定の順序で既存のCA証明書を検索し、ファイルが見つかるまで検索を行います。

1.  Debian、Ubuntu、Gentoo、Arch、またはSlackware の場合は `/etc/ssl/certs/ca-certificates.crt`
2.  RedHat、Fedora、CentOS、Mageia、Vercel、またはNetlify の場合は `/etc/pki/tls/certs/ca-bundle.crt`
3.  OpenSUSE の場合は `/etc/ssl/ca-bundle.pem`
4.  macOS またはAlpine（dockerコンテナ）の場合は `/etc/ssl/cert.pem`

CA証明書のパスを手動で指定することも可能ですが、異なるマシンや環境が異なる場所にCA証明書を保存しているため、複数の環境での展開において大きな不便を引き起こす可能性があります。したがって、柔軟性と異なる環境での展開の容易さのために、`sslca` を `nil` に設定することを推奨します。

## 次のステップ {#next-steps}

-   [mysql2 のドキュメント](https://github.com/brianmario/mysql2#readme) から mysql2 ドライバのより詳細な使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md) の章を通じて TiDB アプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md) です。
-   プロフェッショナルな [TiDB 開発者コース](https://www.pingcap.com/education/) を通じて学び、試験に合格した後に [TiDB 認定資格](https://www.pingcap.com/education/certification/) を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX) チャンネルで質問してください。
