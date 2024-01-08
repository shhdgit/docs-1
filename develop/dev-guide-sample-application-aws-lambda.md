---
title: Connect to TiDB with mysql2 in AWS Lambda Function
summary: This article describes how to build a CRUD application using TiDB and mysql2 in AWS Lambda Function and provides a simple example code snippet.
---

# Connect to TiDB with mysql2 in AWS Lambda Function {#connect-to-tidb-with-mysql2-in-aws-lambda-function}

TiDBはMySQL互換のデータベースで、[AWS Lambda Function](https://aws.amazon.com/lambda/)はコンピューティングサービスであり、[mysql2](https://github.com/sidorares/node-mysql2)はNode.js向けの人気のあるオープンソースドライバーです。

このチュートリアルでは、以下のタスクを実行するために、AWS Lambda FunctionでTiDBとmysql2を使用する方法を学ぶことができます。

- 環境をセットアップします。
- mysql2を使用してTiDBクラスターに接続します。
- アプリケーションをビルドして実行します。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。
- AWS Lambda Functionをデプロイします。

> **注意**
>
> このチュートリアルは、TiDB ServerlessとTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、以下が必要です。

- [Node.js **18**](https://nodejs.org/en/download/) またはそれ以降。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスター。
- 管理者権限を持つ[AWSユーザー](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)。
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)

<CustomContent platform="tidb">

**TiDBクラスターがない場合は、以下のように作成できます。**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターがない場合は、以下のように作成できます。**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

AWSアカウントまたはユーザーがない場合は、[Lambdaのはじめに](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html)ガイドの手順に従って作成できます。

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

> **注意**
>
> 完全なコードスニペットと実行手順については、[tidb-samples/tidb-aws-lambda-quickstart](https://github.com/tidb-samples/tidb-aws-lambda-quickstart) GitHubリポジトリを参照してください。

### ステップ1：サンプルアプリリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで以下のコマンドを実行して、サンプルコードリポジトリをクローンします：

```bash
git clone git@github.com:tidb-samples/tidb-aws-lambda-quickstart.git
cd tidb-aws-lambda-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysql2`を含む）をインストールします：

```bash
npm install
```

### ステップ3：接続情報を構成する {#step-3-configure-connection-information}

選択したTiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>

<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致していることを確認します。

   - **エンドポイントタイプ**が`Public`に設定されていること

   - **Branch**が`main`に設定されていること

   - **Connect With**が`General`に設定されていること

   - **Operating System**が環境に一致していること

   > **注意**
   >
   > Node.jsアプリケーションでは、TLS（SSL）接続を確立する際に、Node.jsがデフォルトで組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、SSL CA証明書を提供する必要はありません。

4. **Generate Password**をクリックしてランダムなパスワードを作成します。

   > **ヒント**
   >
   > 以前にパスワードを生成した場合、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成できます。

5. 対応する接続文字列を`env.json`にコピーして貼り付けます。以下は例です：

   ```json
   {
     "Parameters": {
       "TIDB_HOST": "{gateway-region}.aws.tidbcloud.com",
       "TIDB_PORT": "4000",
       "TIDB_USER": "{prefix}.root",
       "TIDB_PASSWORD": "{password}"
     }
   }
   ```

   `{}`内のプレースホルダを接続ダイアログで取得した値に置き換えます。

</div>

<div label="TiDB Self-Hosted">

対応する接続文字列を`env.json`にコピーして貼り付けます。以下は例です：

```json
{
  "Parameters": {
    "TIDB_HOST": "{tidb_server_host}",
    "TIDB_PORT": "4000",
    "TIDB_USER": "root",
    "TIDB_PASSWORD": "{password}"
  }
}
```

`{}`のプレースホルダを**Connect**ウィンドウで取得した値に置き換えます。

</div>

</SimpleTab>

### ステップ4：コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

1. (前提条件) [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)をインストールします。

2. バンドルをビルドします：

   ```bash
   npm run build
   ```

3. サンプルのLambda関数を呼び出します：

   ```bash
   sam local invoke --env-vars env.json -e events/event.json "tidbHelloWorldFunction"
   ```

4. ターミナルで出力を確認します。出力が以下のようなものであれば、接続に成功しています：

   ```bash
   {"statusCode":200,"body":"{\"results\":[{\"Hello World\":\"Hello World\"}]}"}
   ```

接続が成功したことを確認したら、[次のセクション](#deploy-the-aws-lambda-function)に従ってAWS Lambda関数をデプロイできます。

## AWS Lambda関数をデプロイする {#deploy-the-aws-lambda-function}

AWS Lambda関数は、[SAM CLI](#sam-cli-deployment-recommended)または\[Webコンソールデプロイ]を使用してデプロイできます。

### SAM CLIデプロイ（推奨） {#sam-cli-deployment-recommended}

1. ([前提条件](#prerequisites)) [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)をインストールします。

2. バンドルをビルドします：

   ```bash
   npm run build
   ```

3. [`template.yml`](https://github.com/tidb-samples/tidb-aws-lambda-quickstart/blob/main/template.yml)の環境変数を更新します：

   ```yaml
   Environment:
     Variables:
       TIDB_HOST: {tidb_server_host}
       TIDB_PORT: 4000
       TIDB_USER: {prefix}.root
       TIDB_PASSWORD: {password}
   ```

4. AWS環境変数を設定します（[短期認証情報](https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-short-term.html)を参照）：

   ```bash
   export AWS_ACCESS_KEY_ID={your_access_key_id}
   export AWS_SECRET_ACCESS_KEY={your_secret_access_key}
   export AWS_SESSION_TOKEN={your_session_token}
   ```

5. AWS Lambda関数をデプロイします：

   ```bash
   sam deploy --guided

   # 例：

   # SAMデプロイの設定
   # ======================

   #        設定ファイル[samconfig.toml]を検索中：見つかりません

   #        'sam deploy'のデフォルト引数を設定中
   #        =========================================
   #        スタック名[sam-app]: tidb-aws-lambda-quickstart
   #        AWSリージョン[us-east-1]: 
   #        #デプロイ前にリソースの変更を確認し、デプロイを開始するには「Y」を入力してください
   #        デプロイ前に変更を確認しますか？[y/N]: 
   #        #SAMは、テンプレート内のリソースに接続するためのロールを作成できるようにするための許可が必要です
   #        SAM CLI IAMロールの作成を許可しますか？[Y/n]: 
   #        #操作が失敗した場合に以前にプロビジョニングされたリソースの状態を保持します
   #        ロールバックを無効にしますか？[y/N]: 
   #        tidbHelloWorldFunctionには認可が定義されていない場合があります。これでよろしいですか？[y/N]: y
   #        tidbHelloWorldFunctionには認可が定義されていない場合があります。これでよろしいですか？[y/N]: y
   #        tidbHelloWorldFunctionには認可が定義されていない場合があります。これでよろしいですか？[y/N]: y
   #        tidbHelloWorldFunctionには認可が定義されていない場合があります。これでよろしいですか？[y/N]: y
   #        設定を構成ファイルに保存しますか？[Y/n]: 
   #        SAM構成ファイル[samconfig.toml]: 
   #        SAM構成環境[default]: 

   #        デプロイに必要なリソースを検索中：
   #        必要なリソースを作成中...
   #        作成が完了しました！
   ```

### Webコンソールデプロイ {#web-console-deployment}

1. バンドルをビルドします：

   ```bash
   npm run build

   # AWS Lambda用のバンドル
   # =====================
   # dist/index.zip
   ```

2. [AWS Lambdaコンソール](https://console.aws.amazon.com/lambda/home#/functions)を訪れます。

3. [Lambda関数の作成](https://docs.aws.amazon.com/lambda/latest/dg/lambda-nodejs.html)の手順に従って、Node.js Lambda関数を作成します。

4. [Lambdaデプロイパッケージ](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-package.html#gettingstarted-package-zip)の手順に従い、`dist/index.zip`ファイルをアップロードします。

5. Lambda関数で[対応する接続文字列をコピーして設定](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)します。

   1. Lambdaコンソールの[Functions](https://console.aws.amazon.com/lambda/home#/functions)ページで、**Configuration**タブを選択し、**Environment variables**を選択します。
   2. **編集**を選択します。
   3. データベースのアクセス資格情報を追加するには、次の手順を実行します：
      - **環境変数の追加**を選択し、**Key**に`TIDB_HOST`、**Value**にホスト名を入力します。
      - **環境変数の追加**を選択し、**Key**に`TIDB_PORT`、**Value**にポート番号（デフォルトは4000）を入力します。
      - **環境変数の追加**を選択し、**Key**に`TIDB_USER`、**Value**にユーザー名を入力します。
      - **環境変数の追加**を選択し、**Key**に`TIDB_PASSWORD`、**Value**にデータベース作成時に選択したパスワードを入力します。
      - **保存**を選択します。

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-aws-lambda-quickstart](https://github.com/tidb-samples/tidb-aws-lambda-quickstart)リポジトリをチェックしてください。

### TiDBへの接続 {#connect-to-tidb}

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続を確立します："

```typescript
// lib/tidb.ts
import mysql from 'mysql2';

let pool: mysql.Pool | null = null;

function connect() {
  return mysql.createPool({
    host: process.env.TIDB_HOST, // TiDB host, for example: {gateway-region}.aws.tidbcloud.com
    port: process.env.TIDB_PORT ? Number(process.env.TIDB_PORT) : 4000, // TiDB port, default: 4000
    user: process.env.TIDB_USER, // TiDB user, for example: {prefix}.root
    password: process.env.TIDB_PASSWORD, // TiDB password
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

export function getPool(): mysql.Pool {
  if (!pool) {
    pool = connect();
  }
  return pool;
}
```

### データの挿入 {#insert-data}

次のクエリは単一の `Player` レコードを作成し、`ResultSetHeader` オブジェクトを返します:

```typescript
const [rsh] = await pool.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100]);
console.log(rsh.insertId);
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

次のクエリは、ID `1`の単一の `Player` レコードを返します：

```typescript
const [rows] = await pool.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1]);
console.log(rows[0]);
```

詳細については、[Query data](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、IDが`1`の`Player`に`50`コインと`50`の商品を追加します：

```typescript
const [rsh] = await pool.query(
    'UPDATE players SET coins = coins + ?, goods = goods + ? WHERE id = ?;',
    [50, 50, 1]
);
console.log(rsh.affectedRows);
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除 {#delete-data}

次のクエリは、ID `1` の `Player` レコードを削除します：

```typescript
const [rsh] = await pool.query('DELETE FROM players WHERE id = ?;', [1]);
console.log(rsh.affectedRows);
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート {#useful-notes}

- データベース接続を管理するために[接続プール](https://github.com/sidorares/node-mysql2#using-connection-pools)を使用すると、頻繁に接続を確立および破棄することによって引き起こされるパフォーマンスのオーバーヘッドを減らすことができます。
- SQLインジェクションを回避するためには、[プリペアドステートメント](https://github.com/sidorares/node-mysql2#using-prepared-statements)を使用することをお勧めします。
- 複雑なSQLステートメントが多く含まれないシナリオでは、[Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、または[Prisma](https://www.prisma.io/)などのORMフレームワークを使用すると、開発効率が大幅に向上します。
- アプリケーションのためのRESTful APIを構築する場合、[AWS Lambda with API Gateway](https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html)を使用することをお勧めします。
- TiDB ServerlessとAWS Lambdaを使用して高性能なアプリケーションを設計する場合は、[このブログ](https://aws.amazon.com/blogs/apn/designing-high-performance-applications-using-serverless-tidb-cloud-and-aws-lambda/)を参照してください。

## 次のステップ {#next-steps}

- AWS Lambda関数でTiDBを使用する方法の詳細については、[TiDB-Lambda-integration/aws-lambda-bookstore Demo](https://github.com/pingcap/TiDB-Lambda-integration/blob/main/aws-lambda-bookstore/README.md)を参照してください。また、AWS API Gatewayを使用してアプリケーションのためのRESTful APIを構築することもできます。
- [mysql2](https://github.com/sidorares/node-mysql2/tree/master/documentation/en)のドキュメントから`mysql2`のさらなる使用法を学んでください。
- [Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)のAWS開発者ガイドからAWS Lambdaのさらなる使用法を学んでください。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学んでください。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)など。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学んで、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得してください。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問してください。または、[サポートチケットを作成](https://support.pingcap.com/)してください。
