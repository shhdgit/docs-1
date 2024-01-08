---
title: Connect to TiDB with mysql2
summary: Learn how to connect to TiDB using Ruby mysql2. This tutorial gives Ruby sample code snippets that work with TiDB using mysql2 gem.
---

# Connect to TiDB with mysql2 {#connect-to-tidb-with-mysql2}

TiDBはMySQL互換のデータベースであり、[mysql2](https://github.com/brianmario/mysql2)はRuby向けの最も人気のあるMySQLドライバの1つです。

このチュートリアルでは、TiDBとmysql2を使用して次のタスクを実行する方法を学ぶことができます。

- 環境をセットアップします。
- mysql2を使用してTiDBクラスタに接続します。
- アプリケーションをビルドして実行します。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Ruby](https://www.ruby-lang.org/en/) >= 3.0がマシンにインストールされていること
- [Bundler](https://bundler.io/)がマシンにインストールされていること
- [Git](https://git-scm.com/downloads)がマシンにインストールされていること
- 実行中のTiDBクラスタ

**TiDBクラスタを持っていない場合は、次のように作成できます:**

<CustomContent platform="tidb">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart.git
cd tidb-ruby-mysql2-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

以下のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysql2`および`dotenv`を含む）をインストールします：

```shell
bundle install
```

<details>
<summary><b>既存のプロジェクトの依存関係をインストールする</b></summary>

既存のプロジェクトについては、次のコマンドを実行してパッケージをインストールしてください：

```shell
bundle add mysql2 dotenv
```

</details>

### ステップ3：接続情報の構成 {#step-3-configure-connection-information}

選択したTiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致することを確認します。

   - **Endpoint Type** が `Public` に設定されていること。
   - **Branch** が `main` に設定されていること。
   - **Connect With** が `General` に設定されていること。
   - **Operating System** がアプリケーションを実行しているオペレーティングシステムと一致していること。

4. パスワードをまだ設定していない場合は、ランダムなパスワードを生成するために **Generate Password** をクリックします。

5. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します：

   ```shell
   cp .env.example .env
   ```

6. `.env` ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータの対応するプレースホルダ `<>` を置き換えます：

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
   > TiDB Serverlessの場合、パブリックエンドポイントを使用する場合は、`DATABASE_ENABLE_SSL` を介してTLS接続を有効にする必要があります。

7. `.env` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します：

   ```shell
   cp .env.example .env
   ```

5. `.env` ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータの対応するプレースホルダ `<>` を置き換えます：

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
   > TiDB Dedicatedクラスタにパブリックエンドポイントを使用して接続する場合は、TLS接続を有効にすることをお勧めします。
   >
   > TLS接続を有効にするには、`DATABASE_ENABLE_SSL` を `true` に変更し、接続ダイアログからダウンロードしたCA証明書のファイルパスを指定するために `DATABASE_SSL_CA` を使用します。

6. `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します：

   ```shell
   cp .env.example .env
   ```

2. `.env` ファイルを編集し、環境変数を次のように設定し、対応するプレースホルダ `<>` を自分自身のTiDB接続情報で置き換えます：

   ```dotenv
   DATABASE_HOST=<host>
   DATABASE_PORT=4000
   DATABASE_USER=<user>
   DATABASE_PASSWORD=<password>
   DATABASE_NAME=test
   ```

   TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3. `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行して結果を確認 {#step-4-run-the-code-and-check-the-result}

次のコマンドを実行してサンプルコードを実行します：

```shell
ruby app.rb
```

接続に成功した場合、コンソールにはTiDBクラスタのバージョンが以下のように出力されます:

```
🔌 Connected to TiDB cluster! (TiDB version: 5.7.25-TiDB-v7.1.3)
⏳ Loading sample game data...
✅ Loaded sample game data.

🆕 Created a new player with ID 12.
ℹ️ Got Player 12: Player { id: 12, coins: 100, goods: 100 }
🔢 Added 50 coins and 50 goods to player 12, updated 1 row.
🚮 Deleted 1 player data.
```

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了できます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-ruby-mysql2-quickstart](https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart) リポジトリをチェックしてください。

### 接続オプションを使用して TiDB に接続する {#connect-to-tidb-with-connection-options}

次のコードは、環境変数で定義されたオプションを使用して TiDB に接続を確立します：

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
> TiDB Serverlessの場合、パブリックエンドポイントを使用する場合は、TLS接続を`DATABASE_ENABLE_SSL`を介して有効にする必要がありますが、`DATABASE_SSL_CA`を介してSSL CA証明書を指定する必要はありません。なぜなら、mysql2 gemがファイルが見つかるまで特定の順序で既存のCA証明書を検索するからです。

### データの挿入 {#insert-data}

次のクエリは、2つのフィールドを持つ単一のプレイヤーを作成し、`last_insert_id`を返します：

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

次のクエリは、特定のプレイヤーのレコードを返します：

```ruby
def get_player_by_id(client, id)
  result = client.query(
    "SELECT id, coins, goods FROM players WHERE id = #{id};"
  )
  result.first
end
```

詳細については、[Query data](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

以下のクエリは、特定のプレイヤーのレコードをIDで更新しました。

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

次のクエリは特定のプレイヤーのレコードを削除します：

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

デフォルトでは、mysql2 gemは特定の順序で既存のCA証明書を検索し、ファイルが見つかるまで検索します。

1. Debian、Ubuntu、Gentoo、Arch、またはSlackwareの場合は `/etc/ssl/certs/ca-certificates.crt`
2. RedHat、Fedora、CentOS、Mageia、Vercel、またはNetlifyの場合は `/etc/pki/tls/certs/ca-bundle.crt`
3. OpenSUSEの場合は `/etc/ssl/ca-bundle.pem`
4. macOSまたはAlpine（dockerコンテナ）の場合は `/etc/ssl/cert.pem`

CA証明書のパスを手動で指定することは可能ですが、異なるマシンや環境が異なる場所にCA証明書を保存しているため、複数の環境での展開シナリオで大きな不便を引き起こす可能性があります。したがって、異なる環境での柔軟性と展開の容易さのために、`sslca`を`nil`に設定することが推奨されています。

## 次のステップ {#next-steps}

- [mysql2のドキュメント](https://github.com/brianmario/mysql2#readme)からmysql2ドライバの使用法をさらに学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章でTiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)チャンネルで質問してください。
