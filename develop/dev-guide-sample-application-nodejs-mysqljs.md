---
title: Connect to TiDB with mysql.js
summary: Learn how to connect to TiDB using mysql.js. This tutorial gives Node.js sample code snippets that work with TiDB using mysql.js.
---

# mysql.jsを使用してTiDBに接続する {#connect-to-tidb-with-mysql-js}

TiDBはMySQL互換のデータベースであり、[mysql.js](https://github.com/mysqljs/mysql)ドライバーはMySQLプロトコルを実装した純粋なNode.js JavaScriptクライアントです。

このチュートリアルでは、TiDBとmysql.jsドライバーを使用して、次のタスクを実行する方法を学ぶことができます。

- 環境をセットアップする。
- mysql.jsドライバーを使用してTiDBクラスターに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedで動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Node.js](https://nodejs.org/en) >= 16.xがマシンにインストールされていること。
- [Git](https://git-scm.com/downloads)がマシンにインストールされていること。
- 実行中のTiDBクラスター。

**TiDBクラスターを持っていない場合は、次のように作成することができます。**

<CustomContent platform="tidb">

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1：サンプルアプリリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-nodejs-mysqljs-quickstart.git
cd tidb-nodejs-mysqljs-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

サンプルアプリケーションに必要なパッケージ（`mysql`および`dotenv`を含む）をインストールするには、次のコマンドを実行してください。

```shell
npm install
```

<details>
<summary><b>既存のプロジェクトに依存関係をインストールする</b></summary>

既存のプロジェクトに対して、以下のコマンドを実行してパッケージをインストールしてください：

```shell
npm install mysql dotenv --save
```

### ステップ3：接続情報を設定する {#step-3-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDBデプロイメントオプションに応じて設定を行います。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が、実行環境に合致することを確認します。

   - **Endpoint Type**が`Public`に設定されていること。
   - **Branch**が`main`に設定されていること。
   - **Connect With**が`General`に設定されていること。
   - **Operating System**が、アプリケーションを実行するオペレーティングシステムと一致すること。

4. パスワードをまだ設定していない場合は、**Generate Password**をクリックしてランダムなパスワードを生成します。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします。

   ```shell
   cp .env.example .env
   ```

6. `.env`ファイルを編集し、環境変数を次のように設定します。対応するプレースホルダ`{}`を接続ダイアログの接続パラメータで置き換えます。

   ```dotenv
   TIDB_HOST={host}
   TIDB_PORT=4000
   TIDB_USER={user}
   TIDB_PASSWORD={password}
   TIDB_DATABASE=test
   TIDB_ENABLE_SSL=true
   ```

   > **Note**
   >
   > TiDB Serverlessでは、パブリックエンドポイントを使用する場合、TLS接続を有効にする必要があります。そのためには、`TIDB_ENABLE_SSL`を使用してTLS接続を有効にする必要があります。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします。

   ```shell
   cp .env.example .env
   ```

5. `.env`ファイルを編集し、環境変数を次のように設定します。対応するプレースホルダ`{}`を接続ダイアログの接続パラメータで置き換えます。

   ```dotenv
   TIDB_HOST={host}
   TIDB_PORT=4000
   TIDB_USER={user}
   TIDB_PASSWORD={password}
   TIDB_DATABASE=test
   TIDB_ENABLE_SSL=true
   TIDB_CA_PATH={downloaded_ssl_ca_path}
   ```

   > **Note**
   >
   > パブリックエンドポイントを使用してTiDB Dedicatedに接続する場合、TLS接続を有効にすることをお勧めします。
   >
   > TLS接続を有効にするには、`TIDB_ENABLE_SSL`を`true`に変更し、`TIDB_CA_PATH`を使用して接続ダイアログからダウンロードしたCA証明書のファイルパスを指定します。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします。

   ```shell
   cp .env.example .env
   ```

2. `.env`ファイルを編集し、クラスタの接続パラメータを使用して対応するプレースホルダ`{}`を置き換えます。例として、次のような設定を行います。

   ```dotenv
   TIDB_HOST={host}
   TIDB_PORT=4000
   TIDB_USER=root
   TIDB_PASSWORD={password}
   TIDB_DATABASE=test
   ```

   TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行し、結果を確認する {#step-4-run-the-code-and-check-the-result}

次のコマンドを実行して、サンプルコードを実行します。

// The input content end

```shell
npm start
```

接続に成功した場合、コンソールには次のようにTiDBクラスタのバージョンが出力されます：

    🔌 Connected to TiDB cluster! (TiDB version: 5.7.25-TiDB-v7.1.3)
    ⏳ Loading sample game data...
    ✅ Loaded sample game data.

    🆕 Created a new player with ID 12.
    ℹ️ Got Player 12: Player { id: 12, coins: 100, goods: 100 }
    🔢 Added 50 coins and 50 goods to player 12, updated 1 row.
    🚮 Deleted 1 player data.

## サンプルコードの断片 {#sample-code-snippets}

あなたは、自分のアプリケーション開発を完了するために、以下のサンプルコードの断片を参照することができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-mysqljs-quickstart](https://github.com/tidb-samples/tidb-nodejs-mysqljs-quickstart) リポジトリをチェックしてください。

### 接続オプションを使用して接続する {#connect-with-connection-options}

以下のコードは、環境変数で定義されたオプションを使用して、TiDBに接続を確立します：

```javascript
// Step 1. Import the 'mysql' and 'dotenv' packages.
import { createConnection } from "mysql";
import dotenv from "dotenv";
import * as fs from "fs";

// Step 2. Load environment variables from .env file to process.env.
dotenv.config();

// Step 3. Create a connection to the TiDB cluster.
const options = {
    host: process.env.TIDB_HOST || '127.0.0.1',
    port: process.env.TIDB_PORT || 4000,
    user: process.env.TIDB_USER || 'root',
    password: process.env.TIDB_PASSWORD || '',
    database: process.env.TIDB_DATABASE || 'test',
    ssl: process.env.TIDB_ENABLE_SSL === 'true' ? {
        minVersion: 'TLSv1.2',
        ca: process.env.TIDB_CA_PATH ? fs.readFileSync(process.env.TIDB_CA_PATH) : undefined
    } : null,
}
const conn = createConnection(options);

// Step 4. Perform some SQL operations...

// Step 5. Close the connection.
conn.end();
```

> **Note:**
>
> TiDB Serverlessを使用する場合、パブリックエンドポイントを使用する際には、`TIDB_ENABLE_SSL`を有効にする必要があります。ただし、`TIDB_CA_PATH`を使用してSSL CA証明書を指定する必要はありません。なぜなら、Node.jsはデフォルトで組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用し、TiDB Serverlessによって信頼されるからです。

### データの挿入 {#insert-data}

以下のクエリを実行すると、単一の`Player`レコードが作成され、新しく作成されたレコードのIDが返されます。

```javascript
conn.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100], (err, ok) => {
   if (err) {
       console.error(err);
   } else {
       console.log(ok.insertId);
   }
});
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

次のクエリは、ID `1`の`Player`レコードを1つ返します。

```javascript
conn.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1], (err, rows) => {
   if (err) {
      console.error(err);
   } else {
      console.log(rows[0]);
   }
});
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、IDが`1`の`Player`に`50`のコインと`50`の商品を追加します：

```javascript
conn.query(
   'UPDATE players SET coins = coins + ?, goods = goods + ? WHERE id = ?;',
   [50, 50, 1],
   (err, ok) => {
      if (err) {
         console.error(err);
      } else {
          console.log(ok.affectedRows);
      }
   }
);
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除 {#delete-data}

次のクエリは、IDが`1`の`Player`レコードを削除します。

```javascript
conn.query('DELETE FROM players WHERE id = ?;', [1], (err, ok) => {
    if (err) {
        reject(err);
    } else {
        resolve(ok.affectedRows);
    }
});
```

[Delete data](/develop/dev-guide-delete-data.md)については、詳しくは[こちら](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート {#useful-notes}

- データベース接続を管理するために[接続プール](https://github.com/mysqljs/mysql#pooling-connections)を使用することで、頻繁に接続を確立し破棄することによるパフォーマンスのオーバーヘッドを減らすことができます。

- SQLインジェクション攻撃を防ぐために、SQLを実行する前に[クエリ値をエスケープ](https://github.com/mysqljs/mysql#escaping-query-values)することを推奨します。

  > **Note**
  >
  > `mysqljs/mysql`パッケージはまだプリペアドステートメントをサポートしていません。クライアント側でのみ値をエスケープします（関連する問題：[mysqljs/mysql#274](https://github.com/mysqljs/mysql/issues/274)）。
  >
  > SQLインジェクションを防ぐためにこの機能を使用したり、バッチの挿入/更新の効率を向上させるためにこの機能を使用したい場合は、代わりに[mysql2](https://github.com/sidorares/node-mysql2)パッケージを使用することをお勧めします。

- [Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、および[Prisma](/develop/dev-guide-sample-application-nodejs-prisma.md)などのORMフレームワークを使用することで、複数の複雑なSQL文がないシナリオで開発効率を向上させることができます。

- データベースで大きな数値（`BIGINT`および`DECIMAL`列）を扱う場合は、`supportBigNumbers: true`オプションを有効にすることをお勧めします。

## 次のステップ {#next-steps}

- [mysql.jsのドキュメント](https://github.com/mysqljs/mysql#readme)からmysql.jsドライバーのより詳細な使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例：[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後、[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
