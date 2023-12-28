---
title: Connect to TiDB with mysql2 in Next.js
summary: This article describes how to build a CRUD application using TiDB and mysql2 in Next.js and provides a simple example code snippet.
---

# Next.jsでmysql2を使用してTiDBに接続する {#connect-to-tidb-with-mysql2-in-next-js}

TiDBはMySQL互換のデータベースであり、[mysql2](https://github.com/sidorares/node-mysql2)はNode.jsで人気のあるオープンソースのドライバーです。

このチュートリアルでは、次のタスクを実行するために、Next.jsでTiDBとmysql2を使用する方法を学ぶことができます。

- 環境をセットアップする。
- mysql2を使用してTiDBクラスターに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意**
>
> このチュートリアルは、TiDB ServerlessとTiDB Self-Hostedで動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Node.js **18**](https://nodejs.org/en/download/) 以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)を参照して、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を参照して、ローカルクラスターを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

> **注意**
>
> 完全なコードスニペットと実行手順については、[tidb-nextjs-vercel-quickstart](https://github.com/tidb-samples/tidb-nextjs-vercel-quickstart) GitHubリポジトリを参照してください。

### ステップ1：サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```bash
git clone git@github.com:tidb-samples/tidb-nextjs-vercel-quickstart.git
cd tidb-nextjs-vercel-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

サンプルアプリケーションに必要なパッケージ（`mysql2`を含む）をインストールするには、次のコマンドを実行してください。

```bash
npm install
```

### ステップ3：接続情報を設定する {#step-3-configure-connection-information}

TiDBクラスターに接続するには、選択したTiDBデプロイメントオプションに応じて設定します。

<SimpleTab>

<div label="TiDB Serverless">

1. [**クラスター**ページ](https://tidbcloud.com/console/clusters)に移動し、ターゲットクラスターの名前をクリックして、概要ページに移動します。

2. 右上の**接続**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が、オペレーティング環境に一致することを確認します。

   - **エンドポイントタイプ**は`Public`に設定されています。

   - **Branch**は`main`に設定されています。

   - **Connect With**は`General`に設定されています。

   - **Operating System**は、環境に一致しています。

   > **注意**
   >
   > Node.jsアプリケーションでは、SSL CA証明書を提供する必要はありません。なぜなら、Node.jsはTLS（SSL）接続を確立する際に、デフォルトで組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するからです。

4. ランダムなパスワードを作成するには、**Generate Password**をクリックします。

   > **ヒント**
   >
   > 以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成することができます。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```bash
   # Linux
   cp .env.example .env
   ```

   ```powershell
   # Windows
   Copy-Item ".env.example" -Destination ".env"
   ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のようになります。

   ```bash
   TIDB_HOST='{gateway-region}.aws.tidbcloud.com'
   TIDB_PORT='4000'
   TIDB_USER='{prefix}.root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   ```

   `{}`内のプレースホルダーを接続ダイアログで取得した値に置き換えます。

7. `.env`ファイルを保存します。

</div>

<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```bash
   # Linux
   cp .env.example .env
   ```

   ```powershell
   # Windows
   Copy-Item ".env.example" -Destination ".env"
   ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のようになります。

   ```bash
   TIDB_HOST='{tidb_server_host}'
   TIDB_PORT='4000'
   TIDB_USER='root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   ```

   `{}`内のプレースホルダーを**Connect**ウィンドウで取得した値に置き換えます。TiDBをローカルで実行している場合は、デフォルトのホストアドレスは`127.0.0.1`で、パスワードは空です。

3. `.env`ファイルを保存します。

</div>

</SimpleTab>

### ステップ4：コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

1. アプリケーションを起動します。

   ```bash
   npm run dev
   ```

2. ブラウザを開き、`http://localhost:3000`にアクセスします（実際のポート番号はターミナルで確認し、デフォルトは`3000`です）。

3. サンプルコードを実行するには、**RUN SQL**をクリックします。

4. ターミナルで出力を確認します。出力が次の例のようであれば、接続に成功しています。

   ```json
   {
     "results": [
       {
         "Hello World": "Hello World"
       }
     ]
   }
   ```

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードと実行方法については、[tidb-nextjs-vercel-quickstart](https://github.com/tidb-samples/tidb-nextjs-vercel-quickstart)リポジトリを参照してください。

### TiDBに接続する {#connect-to-tidb}

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続します。

```javascript
// src/lib/tidb.js
import mysql from 'mysql2';

let pool = null;

export function connect() {
  return mysql.createPool({
    host: process.env.TIDB_HOST, // TiDB host, for example: {gateway-region}.aws.tidbcloud.com
    port: process.env.TIDB_PORT || 4000, // TiDB port, default: 4000
    user: process.env.TIDB_USER, // TiDB user, for example: {prefix}.root
    password: process.env.TIDB_PASSWORD, // The password of TiDB user.
    database: process.env.TIDB_DATABASE || 'test', // TiDB database name, default: test
    ssl: {
      minVersion: 'TLSv1.2',
      rejectUnauthorized: true,
    },
    connectionLimit: 1, // Setting connectionLimit to "1" in a serverless function environment optimizes resource usage, reduces costs, ensures connection stability, and enables seamless scalability.
    maxIdle: 1, // max idle connections, the default value is the same as `connectionLimit`
    enableKeepAlive: true,
  });
}

export function getPool() {
  if (!pool) {
    pool = createPool();
  }
  return pool;
}
```

### データの挿入 {#insert-data}

以下のクエリは単一の `Player` レコードを作成し、`ResultSetHeader` オブジェクトを返します。

```javascript
const [rsh] = await pool.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100]);
console.log(rsh.insertId);
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

次のクエリは、ID `1`の単一の`Player`レコードを返します：

```javascript
const [rows] = await pool.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1]);
console.log(rows[0]);
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、IDが`1`の`Player`に`50`のコインと`50`の商品を追加します：

```javascript
const [rsh] = await pool.query(
    'UPDATE players SET coins = coins + ?, goods = goods + ? WHERE id = ?;',
    [50, 50, 1]
);
console.log(rsh.affectedRows);
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除 {#delete-data}

次のクエリは、ID `1` の `Player` レコードを削除します：

```javascript
const [rsh] = await pool.query('DELETE FROM players WHERE id = ?;', [1]);
console.log(rsh.affectedRows);
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート {#useful-notes}

- データベース接続を管理するために[接続プール](https://github.com/sidorares/node-mysql2#using-connection-pools)を使用することで、頻繁に接続を確立し破棄することによるパフォーマンスのオーバーヘッドを減らすことができます。
- SQLインジェクションを防ぐために、[プリペアドステートメント](https://github.com/sidorares/node-mysql2#using-prepared-statements)を使用することを推奨します。
- 複雑なSQL文が多く含まれないシナリオでは、[Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、または[Prisma](https://www.prisma.io/)などのORMフレームワークを使用することで、開発効率を大幅に向上させることができます。

## 次のステップ {#next-steps}

- ORMとNext.jsを使用して複雑なアプリケーションを構築する方法の詳細については、[Bookshop Demo](https://github.com/pingcap/tidb-prisma-vercel-demo)を参照してください。
- [node-mysql2のドキュメント](https://github.com/sidorares/node-mysql2/tree/master/documentation/en)から、node-mysql2ドライバーのより詳細な使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータの取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)のベストプラクティスを学びます。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後、[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
