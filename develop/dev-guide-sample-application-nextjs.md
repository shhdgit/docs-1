---
title: Connect to TiDB with mysql2 in Next.js
summary: This article describes how to build a CRUD application using TiDB and mysql2 in Next.js and provides a simple example code snippet.
---

# mysql2を使用してNext.jsでTiDBに接続する {#connect-to-tidb-with-mysql2-in-next-js}

TiDBはMySQL互換のデータベースであり、[mysql2](https://github.com/sidorares/node-mysql2)はNode.js向けの人気のあるオープンソースドライバーです。

このチュートリアルでは、TiDBとmysql2を使用してNext.jsで次のタスクを実行する方法を学ぶことができます。

- 環境を設定する。
- mysql2を使用してTiDBクラスタに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意**
>
> このチュートリアルは、TiDB ServerlessとTiDB Self-Hostedで動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Node.js **18**](https://nodejs.org/en/download/) またはそれ以降。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

> **注意**
>
> 完全なコードスニペットと実行手順については、[tidb-nextjs-vercel-quickstart](https://github.com/tidb-samples/tidb-nextjs-vercel-quickstart) GitHubリポジトリを参照してください。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします:

```bash
git clone git@github.com:tidb-samples/tidb-nextjs-vercel-quickstart.git
cd tidb-nextjs-vercel-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysql2`を含む）をインストールします：

```bash
npm install
```

### ステップ3：接続情報の構成 {#step-3-configure-connection-information}

選択したTiDBデプロイメントオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>

<div label="TiDB Serverless">

1. [**クラスタ**ページ](https://tidbcloud.com/console/clusters)に移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境と一致することを確認します。

   - **エンドポイントタイプ**が`Public`に設定されていること

   - **ブランチ**が`main`に設定されていること

   - **接続先**が`General`に設定されていること

   - **オペレーティングシステム**が環境に一致していること

   > **注意**
   >
   > Node.jsアプリケーションでは、TLS（SSL）接続を確立する際に、Node.jsがデフォルトで組み込まれている[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、SSL CA証明書を提供する必要はありません。

4. ランダムなパスワードを作成するには、**パスワードを生成**をクリックします。

   > **ヒント**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**パスワードをリセット**をクリックして新しいパスワードを生成できます。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します：

   ```bash
   # Linux
   cp .env.example .env
   ```

   ```powershell
   # Windows
   Copy-Item ".env.example" -Destination ".env"
   ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```bash
   TIDB_HOST='{gateway-region}.aws.tidbcloud.com'
   TIDB_PORT='4000'
   TIDB_USER='{prefix}.root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   ```

   `{}`内のプレースホルダを接続ダイアログで取得した値で置き換えます。

7. `.env`ファイルを保存します。

</div>

<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します：

   ```bash
   # Linux
   cp .env.example .env
   ```

   ```powershell
   # Windows
   Copy-Item ".env.example" -Destination ".env"
   ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```bash
   TIDB_HOST='{tidb_server_host}'
   TIDB_PORT='4000'
   TIDB_USER='root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   ```

   `{}`内のプレースホルダを**接続**ウィンドウで取得した値で置き換えます。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`で、パスワードは空です。

3. `.env`ファイルを保存します。

</div>

</SimpleTab>

### ステップ4：コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

1. アプリケーションを起動します：

   ```bash
   npm run dev
   ```

2. ブラウザを開き、`http://localhost:3000`にアクセスします（実際のポート番号はターミナルで確認し、デフォルトは`3000`です）。

3. **RUN SQL**をクリックしてサンプルコードを実行します。

4. ターミナルで出力を確認します。出力が以下のようなものであれば、接続に成功しています：

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

独自のアプリケーション開発を完了するためのサンプルコードスニペットを参照できます。

完全なサンプルコードとその実行方法については、[tidb-nextjs-vercel-quickstart](https://github.com/tidb-samples/tidb-nextjs-vercel-quickstart)リポジトリをチェックしてください。

### TiDBへの接続 {#connect-to-tidb}

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続を確立します：

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

次のクエリは単一の `Player` レコードを作成し、`ResultSetHeader` オブジェクトを返します：

```javascript
const [rsh] = await pool.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100]);
console.log(rsh.insertId);
```

For more information, refer to [データの挿入](/develop/dev-guide-insert-data.md).

### データのクエリ {#query-data}

次のクエリは、ID `1` によって単一の `Player` レコードを返します：

```javascript
const [rows] = await pool.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1]);
console.log(rows[0]);
```

For more information, refer to [Query data](/develop/dev-guide-get-data-from-single-table.md).

### データの更新 {#update-data}

次のクエリは、IDが`1`の`Player`に`50`コインと`50`商品を追加します：

```javascript
const [rsh] = await pool.query(
    'UPDATE players SET coins = coins + ?, goods = goods + ? WHERE id = ?;',
    [50, 50, 1]
);
console.log(rsh.affectedRows);
```

For more information, refer to [データの更新](/develop/dev-guide-update-data.md).

### データの削除 {#delete-data}

次のクエリは、IDが`1`の`Player`レコードを削除します：

```javascript
const [rsh] = await pool.query('DELETE FROM players WHERE id = ?;', [1]);
console.log(rsh.affectedRows);
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート {#useful-notes}

- [接続プール](https://github.com/sidorares/node-mysql2#using-connection-pools)を使用してデータベース接続を管理すると、頻繁に接続を確立および破棄することによるパフォーマンスオーバーヘッドを減らすことができます。
- SQLインジェクションを回避するためには、[プリペアドステートメント](https://github.com/sidorares/node-mysql2#using-prepared-statements)を使用することをお勧めします。
- 複雑なSQLステートメントが多く含まれないシナリオでは、[Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、または[Prisma](https://www.prisma.io/)などのORMフレームワークを使用すると、開発効率が大幅に向上します。

## 次のステップ {#next-steps}

- ORMとNext.jsを使用して複雑なアプリケーションを構築する方法の詳細については、[Bookshop Demo](https://github.com/pingcap/tidb-prisma-vercel-demo)を参照してください。
- [node-mysql2のドキュメント](https://github.com/sidorares/node-mysql2/tree/master/documentation/en)からnode-mysql2ドライバーの使用方法について詳しく学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスについて学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
