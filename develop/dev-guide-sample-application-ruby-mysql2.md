---
title: Connect to TiDB with mysql2
summary: Learn how to connect to TiDB using Ruby mysql2. This tutorial gives Ruby sample code snippets that work with TiDB using mysql2 gem.
---

# mysql2を使用してTiDBに接続する {#connect-to-tidb-with-mysql2}

TiDBはMySQL互換のデータベースであり、[mysql2](https://github.com/brianmario/mysql2)はRubyで最も人気のあるMySQLドライバーの1つです。

このチュートリアルでは、TiDBとmysql2を使用して次のタスクを実行する方法を学ぶことができます。

- 環境をセットアップする。
- mysql2を使用してTiDBクラスターに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedで動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Ruby](https://www.ruby-lang.org/en/) >= 3.0がマシンにインストールされていること
- [Bundler](https://bundler.io/)がマシンにインストールされていること
- [Git](https://git-scm.com/downloads)がマシンにインストールされていること
- 実行中のTiDBクラスター

**TiDBクラスターを持っていない場合は、次のように作成することができます。**

<CustomContent platform="tidb">

- (推奨) [TiDB Serverlessクラスターを作成する](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成する。
- [ローカルテストTiDBクラスターをデプロイする](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターをデプロイする](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成する。

</CustomContent>
<CustomContent platform="tidb-cloud">

- (推奨) [TiDB Serverlessクラスターを作成する](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成する。
- [ローカルテストTiDBクラスターをデプロイする](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターをデプロイする](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成する。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1：サンプルアプリリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart.git
cd tidb-ruby-mysql2-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

サンプルアプリケーションに必要なパッケージ（`mysql2`と`dotenv`を含む）をインストールするには、次のコマンドを実行してください：

```shell
bundle install
```

<details>
<summary><b>既存プロジェクトの依存関係をインストールする</b></summary>

既存のプロジェクトに対して、以下のコマンドを実行してパッケージをインストールしてください：

```shell
bundle add mysql2 dotenv
```

### ステップ3：接続情報を設定する {#step-3-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDBデプロイメントオプションに応じて接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が操作環境に一致することを確認します。

   - **エンドポイントタイプ**は`Public`に設定されています。
   - **Branch**は`main`に設定されています。
   - **Connect With**は`General`に設定されています。
   - **Operating System**は、アプリケーションを実行するオペレーティングシステムに一致しています。

4. パスワードをまだ設定していない場合は、ランダムなパスワードを生成するには、**パスワードを生成**をクリックします。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

6. `.env`ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータで対応するプレースホルダー`<>`を置き換えます。

   ```dotenv
   DATABASE_HOST=<host>
   DATABASE_PORT=4000
   DATABASE_USER=<user>
   DATABASE_PASSWORD=<password>
   DATABASE_NAME=test
   DATABASE_ENABLE_SSL=true
   ```

   > **Note**
   >
   > TiDB Serverlessの場合、パブリックエンドポイントを使用する場合は、`DATABASE_ENABLE_SSL`を使用してTLS接続を有効にする必要があります。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. **どこからでもアクセスを許可**をクリックし、**TiDBクラスタCAをダウンロード**をクリックして、CA証明書をダウンロードします。

   接続文字列を取得する詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

5. `.env`ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータで対応するプレースホルダー`<>`を置き換えます。

   ```dotenv
   DATABASE_HOST=<host>
   DATABASE_PORT=4000
   DATABASE_USER=<user>
   DATABASE_PASSWORD=<password>
   DATABASE_NAME=test
   DATABASE_ENABLE_SSL=true
   DATABASE_SSL_CA=<downloaded_ssl_ca_path>
   ```

   > **Note**
   >
   > パブリックエンドポイントを使用してTiDB Dedicatedクラスタに接続する場合は、TLS接続を有効にすることをお勧めします。
   >
   > TLS接続を有効にするには、`DATABASE_ENABLE_SSL`を`true`に変更し、接続ダイアログからダウンロードしたCA証明書のファイルパスを`DATABASE_SSL_CA`に指定します。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

2. `.env`ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータで対応するプレースホルダー`<>`を置き換えます。

   ```dotenv
   DATABASE_HOST=<host>
   DATABASE_PORT=4000
   DATABASE_USER=<user>
   DATABASE_PASSWORD=<password>
   DATABASE_NAME=test
   ```

   TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行し、結果を確認する {#step-4-run-the-code-and-check-the-result}

次のコマンドを実行して、サンプルコードを実行します。

```shell
ruby app.rb
```

接続に成功した場合、コンソールには次のようにTiDBクラスタのバージョンが出力されます:

    🔌 Connected to TiDB cluster! (TiDB version: 5.7.25-TiDB-v7.1.3)
    ⏳ Loading sample game data...
    ✅ Loaded sample game data.

    🆕 Created a new player with ID 12.
    ℹ️ Got Player 12: Player { id: 12, coins: 100, goods: 100 }
    🔢 Added 50 coins and 50 goods to player 12, updated 1 row.
    🚮 Deleted 1 player data.

## サンプルコードの断片 {#sample-code-snippets}

あなたは、以下のサンプルコードの断片を参考にして、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-ruby-mysql2-quickstart](https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart) リポジトリをご覧ください。

### 接続オプションを使用して TiDB に接続する {#connect-to-tidb-with-connection-options}

以下のコードは、環境変数で定義されたオプションを使用して TiDB に接続を確立します。

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

> **Note**
>
> TiDB Serverlessでは、パブリックエンドポイントを使用する場合、TLS接続を有効にする必要がありますが、`DATABASE_ENABLE_SSL`を使用してSSL CA証明書を指定する必要はありません。mysql2 gemは、ファイルが見つかるまで特定の順序で既存のCA証明書を検索します。

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

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

次のクエリは、IDによって特定のプレイヤーのレコードを返します。

```ruby
def get_player_by_id(client, id)
  result = client.query(
    "SELECT id, coins, goods FROM players WHERE id = #{id};"
  )
  result.first
end
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、IDによって特定のプレイヤーのレコードを更新します。

```ruby
def update_player(client, player_id, inc_coins, inc_goods)
  result = client.query(
    "UPDATE players SET coins = coins + #{inc_coins}, goods = goods + #{inc_goods} WHERE id = #{player_id};"
  )
  client.affected_rows
end
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除 {#delete-data}

以下のクエリは、特定のプレイヤーのレコードを削除します：

```ruby
def delete_player_by_id(client, id)
  result = client.query(
    "DELETE FROM players WHERE id = #{id};"
  )
  client.affected_rows
end
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## ベストプラクティス {#best-practices}

デフォルトでは、mysql2 gemはファイルが発見されるまで特定の順序で既存のCA証明書を検索することができます。

1. Debian、Ubuntu、Gentoo、Arch、またはSlackwareの場合は`/etc/ssl/certs/ca-certificates.crt`
2. RedHat、Fedora、CentOS、Mageia、Vercel、またはNetlifyの場合は`/etc/pki/tls/certs/ca-bundle.crt`
3. OpenSUSEの場合は`/etc/ssl/ca-bundle.pem`
4. macOSまたはAlpine（dockerコンテナ）の場合は`/etc/ssl/cert.pem`

CA証明書のパスを手動で指定することも可能ですが、異なるマシンや環境ではCA証明書が異なる場所に保存される可能性があるため、複数の環境でのデプロイメントシナリオでは重大な不便を引き起こす可能性があります。そのため、`sslca`を`nil`に設定することをお勧めします。これにより、さまざまな環境での柔軟性とデプロイメントの容易さが実現されます。

## 次のステップ {#next-steps}

- [mysql2のドキュメント](https://github.com/brianmario/mysql2#readme)からmysql2ドライバのより詳細な使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などです。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)チャンネルで質問してください。
