---
title: Connect to TiDB with node-mysql2
summary: Learn how to connect to TiDB using node-mysql2. This tutorial gives Node.js sample code snippets that work with TiDB using node-mysql2.
---

# node-mysql2を使用してTiDBに接続する {#connect-to-tidb-with-node-mysql2}

TiDBはMySQL互換のデータベースであり、[node-mysql2](https://github.com/sidorares/node-mysql2)はNode.js向けの高速な[mysqljs/mysql](https://github.com/mysqljs/mysql)互換のMySQLドライバです。

このチュートリアルでは、TiDBとnode-mysql2を使用して次のタスクを実行する方法を学ぶことができます:

-   環境をセットアップする。
-   node-mysql2を使用してTiDBクラスタに接続する。
-   アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です:

-   マシンにインストールされている[Node.js](https://nodejs.org/en) >= 16.x。
-   マシンにインストールされている[Git](https://git-scm.com/downloads)。
-   実行中のTiDBクラスタ。

**TiDBクラスタを持っていない場合は、次のように作成できます:**

<CustomContent platform="tidb">

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-nodejs-mysql2-quickstart.git
cd tidb-nodejs-mysql2-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

以下のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysql2`および`dotenv`を含む）をインストールしてください：

```shell
npm install
```

<details>
<summary><b>既存のプロジェクトに依存関係をインストールする</b></summary>

既存のプロジェクトに対して、以下のコマンドを実行してパッケージをインストールしてください：

```shell
npm install mysql2 dotenv --save
```

</details>

### ステップ3：接続情報を構成する {#step-3-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDB展開オプションに応じて行います。

<SimpleTab>
<div label="TiDBサーバーレス">

1.  [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成が操作環境に一致することを確認します。

    -   **エンドポイントタイプ** が `Public` に設定されていること。
    -   **ブランチ** が `main` に設定されていること。
    -   **Connect With** が `General` に設定されていること。
    -   **オペレーティングシステム** がアプリケーションを実行するオペレーティングシステムと一致していること。

4.  パスワードをまだ設定していない場合は、ランダムなパスワードを生成するために **Generate Password** をクリックします。

5.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

6.  `.env` ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータで対応するプレースホルダ `{}` を置き換えます：

    ```dotenv
    TIDB_HOST={ホスト}
    TIDB_PORT=4000
    TIDB_USER={ユーザー}
    TIDB_PASSWORD={パスワード}
    TIDB_DATABASE=test
    TIDB_ENABLE_SSL=true
    ```

    > **注意**
    >
    > TiDBサーバーレスの場合、パブリックエンドポイントを使用する場合は、TLS接続を `TIDB_ENABLE_SSL` を介して必ず有効にする必要があります。

7.  `.env` ファイルを保存します。

</div>
<div label="TiDB専用">

1.  [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB専用標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

5.  `.env` ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータで対応するプレースホルダ `{}` を置き換えます：

    ```dotenv
    TIDB_HOST={ホスト}
    TIDB_PORT=4000
    TIDB_USER={ユーザー}
    TIDB_PASSWORD={パスワード}
    TIDB_DATABASE=test
    TIDB_ENABLE_SSL=true
    TIDB_CA_PATH={ダウンロードしたssl_ca_path}
    ```

    > **注意**
    >
    > TiDB専用にパブリックエンドポイントを使用して接続する場合は、TLS接続を有効にすることをお勧めします。
    >
    > TLS接続を有効にするには、`TIDB_ENABLE_SSL` を `true` に変更し、接続ダイアログからダウンロードしたCA証明書のファイルパスを指定するために `TIDB_CA_PATH` を使用します。

6.  `.env` ファイルを保存します。

</div>
<div label="TiDBセルフホスト">

1.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

2.  `.env` ファイルを編集し、環境変数を次のように設定し、接続ダイアログの接続パラメータで対応するプレースホルダ `{}` を置き換えます：

    ```dotenv
    TIDB_HOST={ホスト}
    TIDB_PORT=4000
    TIDB_USER=root
    TIDB_PASSWORD={パスワード}
    TIDB_DATABASE=test
    ```

    TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3.  `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

次のコマンドを実行して、サンプルコードを実行します：

```shell
npm start
```

If the connection is successful, the console will output the version of the TiDB cluster as follows:
接続が成功した場合、コンソールにはTiDBクラスタのバージョンが以下のように出力されます。

    🔌 Connected to TiDB cluster! (TiDB version: 5.7.25-TiDB-v7.1.0)
    ⏳ Loading sample game data...
    ✅ Loaded sample game data.

    🆕 Created a new player with ID 12.
    ℹ️ Got Player 12: Player { id: 12, coins: 100, goods: 100 }
    🔢 Added 50 coins and 50 goods to player 12, updated 1 row.
    🚮 Deleted 1 player data.

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-mysql2-quickstart](https://github.com/tidb-samples/tidb-nodejs-mysql2-quickstart) リポジトリをチェックしてください。

### 接続オプションを使用して接続する {#connect-with-connection-options}

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続を確立します。

```javascript
// Step 1. Import the 'mysql' and 'dotenv' packages.
import { createConnection } from "mysql2/promise";
import dotenv from "dotenv";
import * as fs from "fs";

// Step 2. Load environment variables from .env file to process.env.
dotenv.config();

async function main() {
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
   const conn = await createConnection(options);

   // Step 4. Perform some SQL operations...

   // Step 5. Close the connection.
   await conn.end();
}

void main();
```

**注意**

TiDB Serverlessを使用する場合、パブリックエンドポイントを使用する際には、`TIDB_ENABLE_SSL`を介してTLS接続を有効にする**必要があります**。ただし、Node.jsはデフォルトで組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、TiDB Serverlessによって信頼されているため、`TIDB_CA_PATH`を介してSSL CA証明書を指定する必要は**ありません**。

### データの挿入 {#insert-data}

次のクエリは単一の`Player`レコードを作成し、`ResultSetHeader`オブジェクトを返します。

```javascript
const [rsh] = await conn.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100]);
console.log(rsh.insertId);
```

[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

次のクエリは、IDが`1`の`Player`レコードを1つ返します。

```javascript
const [rows] = await conn.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1]);
console.log(rows[0]);
```

詳細については、[クエリデータ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、IDが`1`の`Player`に`50`コインと`50`商品を追加します。

```javascript
const [rsh] = await conn.query(
    'UPDATE players SET coins = coins + ?, goods = goods + ? WHERE id = ?;',
    [50, 50, 1]
);
console.log(rsh.affectedRows);
```

[データの更新](/develop/dev-guide-update-data.md)に関する詳細は、こちらを参照してください。

### データの削除 {#delete-data}

以下のクエリは、IDが`1`の`Player`レコードを削除します。

```javascript
const [rsh] = await conn.query('DELETE FROM players WHERE id = ?;', [1]);
console.log(rsh.affectedRows);
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md) を参照してください。

## 便利なノート {#useful-notes}

-   データベース接続の管理に[接続プール](https://github.com/sidorares/node-mysql2#using-connection-pools)を使用すると、頻繁に接続を確立および破棄することによるパフォーマンスのオーバーヘッドを減らすことができます。
-   SQLインジェクションを防ぐためには、[プリペアドステートメント](https://github.com/sidorares/node-mysql2#using-prepared-statements)の使用が推奨されています。
-   複雑なSQLステートメントがあまり関与しないシナリオでは、[Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、または[Prisma](https://www.prisma.io/)などのORMフレームワークの使用が開発効率を大幅に向上させることができます。
-   データベースの大きな数値（BIGINTおよびDECIMAL列）を扱う際には、`supportBigNumbers: true`オプションを有効にすることが推奨されています。
-   ネットワークの問題によるソケットエラー`read ECONNRESET`を避けるためには、`enableKeepAlive: true`オプションを有効にすることが推奨されています。（関連する問題: [sidorares/node-mysql2#683](https://github.com/sidorares/node-mysql2/issues/683)）

## 次のステップ {#next-steps}

-   [node-mysql2のドキュメント](https://github.com/sidorares/node-mysql2#readme)からnode-mysql2ドライバのさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などです。
-   プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定資格](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
