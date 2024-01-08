---
title: Connect to TiDB with Rails framework and ActiveRecord ORM
summary: Learn how to connect to TiDB using the Rails framework. This tutorial gives Ruby sample code snippets that work with TiDB using the Rails framework and ActiveRecord ORM.
---

# RailsフレームワークとActiveRecord ORMを使用してTiDBに接続 {#connect-to-tidb-with-rails-framework-and-activerecord-orm}

TiDBはMySQL互換のデータベースで、[Rails](https://github.com/rails/rails)はRubyで書かれた人気のあるWebアプリケーションフレームワークであり、[ActiveRecord ORM](https://github.com/rails/rails/tree/main/activerecord)はRailsのオブジェクト関係マッピングです。

このチュートリアルでは、TiDBとRailsを使用して次のタスクを実行する方法を学ぶことができます。

- 環境をセットアップします。
- Railsを使用してTiDBクラスタに接続します。
- アプリケーションをビルドして実行します。オプションで、ActiveRecord ORMを使用して基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと動作します。

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

## サンプルアプリを実行してTiDBに接続 {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-ruby-rails-quickstart.git
cd tidb-ruby-rails-quickstart
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

### ステップ3：接続情報を構成する {#step-3-configure-connection-information}

選択したTiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログで、**Connect With**ドロップダウンリストから`Rails`を選択し、**Endpoint Type**のデフォルト設定を`Public`に保ちます。

4. パスワードをまだ設定していない場合は、**Generate Password**をクリックしてランダムなパスワードを生成します。

5. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します：

   ```shell
   cp .env.example .env
   ```

6. `.env`ファイルを編集し、`DATABASE_URL`環境変数を次のように設定し、接続文字列を接続ダイアログから変数値としてコピーします。

   ```dotenv
   DATABASE_URL=mysql2://<user>:<password>@<host>:<port>/<database_name>?ssl_mode=verify_identity
   ```

   > **Note**
   >
   > TiDB Serverlessの場合、パブリックエンドポイントを使用する場合は、`ssl_mode=verify_identity`クエリパラメータでTLS接続を有効にする必要があります。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、次に**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します：

   ```shell
   cp .env.example .env
   ```

5. `.env`ファイルを編集し、`DATABASE_URL`環境変数を次のように設定し、接続文字列を接続ダイアログから変数値としてコピーし、`sslca`クエリパラメータを接続ダイアログからダウンロードしたCA証明書のファイルパスに設定します：

   ```dotenv
   DATABASE_URL=mysql2://<user>:<password>@<host>:<port>/<database>?ssl_mode=verify_identity&sslca=/path/to/ca.pem
   ```

   > **Note**
   >
   > TiDB Dedicatedに接続する際は、パブリックエンドポイントを使用してTLS接続を有効にすることをお勧めします。
   >
   > TLS接続を有効にするには、`ssl_mode`クエリパラメータの値を`verify_identity`に変更し、`sslca`の値を接続ダイアログからダウンロードしたCA証明書のファイルパスに変更します。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します：

   ```shell
   cp .env.example .env
   ```

2. `.env`ファイルを編集し、`DATABASE_URL`環境変数を次のように設定し、TiDB接続情報の`<user>`、`<password>`、`<host>`、`<port>`、`<database>`を自分自身のものに置き換えます：

   ```dotenv
   DATABASE_URL=mysql2://<user>:<password>@<host>:<port>/<database>
   ```

   TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

1. データベースとテーブルを作成します：

   ```shell
   bundle exec rails db:create
   bundle exec rails db:migrate
   ```

2. サンプルデータをシードします：

   ```shell
   bundle exec rails db:seed
   ```

3. 次のコマンドを実行してサンプルコードを実行します：

   ```shell
   bundle exec rails runner ./quickstart.rb
   ```

接続が成功した場合、コンソールにTiDBクラスタのバージョンが次のように出力されます：

```
🔌 Connected to TiDB cluster! (TiDB version: 5.7.25-TiDB-v7.1.3)
⏳ Loading sample game data...
✅ Loaded sample game data.

🆕 Created a new player with ID 12.
ℹ️ Got Player 12: Player { id: 12, coins: 100, goods: 100 }
🔢 Added 50 coins and 50 goods to player 12, updated 1 row.
🚮 Deleted 1 player data.
```

## Sample code snippets {#sample-code-snippets}

あなた自身のアプリケーション開発を完了するために、以下のサンプルコードスニペットを参照できます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-ruby-rails-quickstart](https://github.com/tidb-samples/tidb-ruby-rails-quickstart) リポジトリをチェックしてください。

### 接続オプションを使用して TiDB に接続する {#connect-to-tidb-with-connection-options}

`config/database.yml` の以下のコードは、環境変数で定義されたオプションを使用して TiDB に接続を確立します。

```yml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  url: <%= ENV["DATABASE_URL"] %>

development:
  <<: *default

test:
  <<: *default
  database: quickstart_test

production:
  <<: *default
```

> **Note**
>
> TiDB Serverlessの場合、パブリックエンドポイントを使用する場合、`DATABASE_URL`で`ssl_mode`クエリパラメータを`verify_identity`に設定してTLS接続を**必ず**有効にする必要がありますが、`DATABASE_URL`でSSL CA証明書を指定する必要はありません。なぜなら、mysql2 gemがファイルが見つかるまで特定の順序で既存のCA証明書を検索するからです。

### データの挿入 {#insert-data}

次のクエリは、2つのフィールドを持つ単一のPlayerを作成し、作成された`Player`オブジェクトを返します：

```ruby
new_player = Player.create!(coins: 100, goods: 100)
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

次のクエリは、特定のプレイヤーのレコードを返します：

```ruby
player = Player.find_by(id: new_player.id)
```

詳細については、[Query data](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、`Player`オブジェクトを更新します：

```ruby
player.update(coins: 50, goods: 50)
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除 {#delete-data}

次のクエリは `Player` オブジェクトを削除します：

```ruby
player.destroy
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## ベストプラクティス {#best-practices}

デフォルトでは、mysql2 gem（ActiveRecord ORMがTiDBに接続するために使用）は、ファイルが見つかるまで特定の順序で既存のCA証明書を検索します。

1. /etc/ssl/certs/ca-certificates.crt # Debian / Ubuntu / Gentoo / Arch / Slackware
2. /etc/pki/tls/certs/ca-bundle.crt # RedHat / Fedora / CentOS / Mageia / Vercel / Netlify
3. /etc/ssl/ca-bundle.pem # OpenSUSE
4. /etc/ssl/cert.pem # MacOS / Alpine（dockerコンテナ）

CA証明書のパスを手動で指定することは可能ですが、異なるマシンや環境が異なる場所にCA証明書を保存しているため、このアプローチは複数の環境での展開においてかなりの不便を引き起こす可能性があります。したがって、異なる環境での柔軟性と展開の容易さのために、`sslca`を`nil`に設定することを推奨します。

## 次のステップ {#next-steps}

- [ActiveRecordのドキュメント](https://guides.rubyonrails.org/active_record_basics.html)からActiveRecord ORMのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じてTiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)チャンネルで質問してください。
