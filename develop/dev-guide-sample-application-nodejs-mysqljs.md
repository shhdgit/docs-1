---
title: Connect to TiDB with mysql.js
summary: Learn how to connect to TiDB using mysql.js. This tutorial gives Node.js sample code snippets that work with TiDB using mysql.js.
---

# mysql.jsを使用してTiDBに接続する {#connect-to-tidb-with-mysql-js}

TiDBはMySQL互換のデータベースであり、[mysql.js](https://github.com/mysqljs/mysql)ドライバーはMySQLプロトコルを実装した純粋なNode.js JavaScriptクライアントです。

このチュートリアルでは、TiDBとmysql.jsドライバーを使用して次のタスクを実行する方法を学ぶことができます。

- 環境を設定します。
- mysql.jsドライバーを使用してTiDBクラスターに接続します。
- アプリケーションをビルドして実行します。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと連携します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Node.js](https://nodejs.org/en) >= 16.xがマシンにインストールされていること。
- [Git](https://git-scm.com/downloads)がマシンにインストールされていること。
- 実行中のTiDBクラスター。

**TiDBクラスターがない場合は、次のように作成できます:**

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

### ステップ1：サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-nodejs-mysqljs-quickstart.git
cd tidb-nodejs-mysqljs-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

以下のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysql`および`dotenv`を含む）をインストールします：

```shell
npm install
```

<details>
<summary><b>既存のプロジェクトに依存関係をインストールする</b></summary>

既存のプロジェクトに対して、以下のコマンドを実行してパッケージをインストールしてください：

```shell
npm install mysql dotenv --save
```

</details>

### ステップ3：接続情報の構成 {#step-3-configure-connection-information}

選択したTiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致していることを確認します。

   - **Endpoint Type** が `Public` に設定されていること。
   - **Branch** が `main` に設定されていること。
   - **Connect With** が `General` に設定されていること。
   - **Operating System** がアプリケーションを実行しているオペレーティングシステムと一致していること。

4. パスワードをまだ設定していない場合は、ランダムなパスワードを生成するには **Generate Password** をクリックします。

5. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します：

   ```shell
   cp .env.example .env
   ```

6. `.env` ファイルを編集し、接続ダイアログの接続パラメータで対応するプレースホルダ `{}` を置き換えて、環境変数を次のように設定します：

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
   > TiDB Serverlessの場合、パブリックエンドポイントを使用する場合は、TLS接続を有効にする必要があります。

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

5. `.env` ファイルを編集し、接続ダイアログの接続パラメータで対応するプレースホルダ `{}` を置き換えて、環境変数を次のように設定します：

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
   > TiDB Dedicatedにパブリックエンドポイントを使用して接続する場合は、TLS接続を有効にすることをお勧めします。
   >
   > TLS接続を有効にするには、`TIDB_ENABLE_SSL` を `true` に変更し、接続ダイアログからダウンロードしたCA証明書のファイルパスを指定するには `TIDB_CA_PATH` を使用します。

6. `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します：

   ```shell
   cp .env.example .env
   ```

2. `.env` ファイルを編集し、クラスタの接続パラメータで対応するプレースホルダ `{}` を置き換えて、環境変数を次のように設定します：

   ```dotenv
   TIDB_HOST={host}
   TIDB_PORT=4000
   TIDB_USER=root
   TIDB_PASSWORD={password}
   TIDB_DATABASE=test
   ```

   TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` で、パスワードは空です。

3. `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行して結果を確認 {#step-4-run-the-code-and-check-the-result}

次のコマンドを実行してサンプルコードを実行します：

```shell
npm start
```

接続に成功した場合、コンソールにはTiDBクラスタのバージョンが次のように出力されます:

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

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-mysqljs-quickstart](https://github.com/tidb-samples/tidb-nodejs-mysqljs-quickstart) リポジトリをチェックしてください。

### 接続オプションで接続 {#connect-with-connection-options}

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続を確立します：

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

> **Note**
>
> TiDB Serverlessの場合、パブリックエンドポイントを使用する場合は、`TIDB_ENABLE_SSL`を介してTLS接続を**必ず**有効にする必要があります。ただし、Node.jsはデフォルトでTiDB Serverlessによって信頼されている組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、`TIDB_CA_PATH`を介してSSL CA証明書を指定する必要はありません。

### データの挿入 {#insert-data}

次のクエリは単一の`Player`レコードを作成し、新しく作成されたレコードのIDを返します：

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

次のクエリは、ID `1`の単一の `Player` レコードを返します：

```javascript
conn.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1], (err, rows) => {
   if (err) {
      console.error(err);
   } else {
      console.log(rows[0]);
   }
});
```

詳細については、[Query data](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、IDが`1`の`Player`に`50`コインと`50`商品を追加します。

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

次のクエリは、ID `1` の `Player` レコードを削除します：

```javascript
conn.query('DELETE FROM players WHERE id = ?;', [1], (err, ok) => {
    if (err) {
        reject(err);
    } else {
        resolve(ok.affectedRows);
    }
});
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート {#useful-notes}

- データベース接続を管理するために[接続プール](https://github.com/mysqljs/mysql#pooling-connections)を使用すると、頻繁に接続を確立および破棄することによるパフォーマンスのオーバーヘッドを減らすことができます。

- SQLインジェクション攻撃を回避するために、SQLを実行する前に[クエリ値のエスケープ](https://github.com/mysqljs/mysql#escaping-query-values)を推奨します。

  > **Note**
  >
  > `mysqljs/mysql`パッケージはまだプリペアドステートメントをサポートしていません。クライアント側での値のエスケープのみを行います（関連する問題: [mysqljs/mysql#274](https://github.com/mysqljs/mysql/issues/274)）。
  >
  > SQLインジェクションを回避したり、バッチの挿入/更新の効率を向上させるためにこの機能を使用したい場合は、代わりに[mysql2](https://github.com/sidorares/node-mysql2)パッケージを使用することをお勧めします。

- 複雑なSQLステートメントがないシナリオで開発効率を向上させるためにORMフレームワークを使用することが推奨されます。例: [Sequelize](https://sequelize.org/), [TypeORM](https://typeorm.io/), および [Prisma](/develop/dev-guide-sample-application-nodejs-prisma.md)。

- データベースで大きな数値（`BIGINT`および`DECIMAL`列）を扱う場合は、`supportBigNumbers: true`オプションを有効にすることをお勧めします。

## 次のステップ {#next-steps}

- [mysql.jsのドキュメント](https://github.com/mysqljs/mysql#readme)からmysql.jsドライバのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例: [データの挿入](/develop/dev-guide-insert-data.md), [データの更新](/develop/dev-guide-update-data.md), [データの削除](/develop/dev-guide-delete-data.md), [データのクエリ](/develop/dev-guide-get-data-from-single-table.md), [トランザクション](/develop/dev-guide-transaction-overview.md), [SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
