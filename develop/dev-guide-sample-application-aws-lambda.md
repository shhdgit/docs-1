---
title: Connect to TiDB with mysql2 in AWS Lambda Function
summary: This article describes how to build a CRUD application using TiDB and mysql2 in AWS Lambda Function and provides a simple example code snippet.
---

# AWS Lambda Functionでmysql2を使用してTiDBに接続する {#connect-to-tidb-with-mysql2-in-aws-lambda-function}

TiDBはMySQL互換のデータベースであり、[AWS Lambda Function](https://aws.amazon.com/lambda/)はコンピューティングサービスであり、[mysql2](https://github.com/sidorares/node-mysql2)はNode.js向けの人気のあるオープンソースドライバーです。

このチュートリアルでは、TiDBとmysql2をAWS Lambda Functionで使用して、次のタスクを実行する方法を学ぶことができます。

- 環境をセットアップする。
- mysql2を使用してTiDBクラスターに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。
- AWS Lambda Functionをデプロイする。

> **注意**
>
> このチュートリアルは、TiDB ServerlessとTiDB Self-Hostedの両方で動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Node.js **18**](https://nodejs.org/en/download/) 以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスター。
- 管理者権限を持つ[AWSユーザー](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)。
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)

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

AWSアカウントやユーザーを持っていない場合は、[Lambdaのはじめ方](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html)ガイドの手順に従って作成できます。

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

> **注意**
>
> 完全なコードスニペットと実行手順については、[tidb-samples/tidb-aws-lambda-quickstart](https://github.com/tidb-samples/tidb-aws-lambda-quickstart)のGitHubリポジトリを参照してください。

### ステップ1：サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```bash
git clone git@github.com:tidb-samples/tidb-aws-lambda-quickstart.git
cd tidb-aws-lambda-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

サンプルアプリケーションに必要なパッケージ（`mysql2`を含む）をインストールするには、次のコマンドを実行してください。

```bash
npm install
```

### ステップ3：接続情報を設定する {#step-3-configure-connection-information}

TiDBのデプロイオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>

<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が、オペレーティング環境と一致していることを確認します。

   - **Endpoint Type**が`Public`に設定されていること

   - **Branch**が`main`に設定されていること

   - **Connect With**が`General`に設定されていること

   - **Operating System**が環境に一致していること

   > **注意**
   >
   > Node.jsアプリケーションでは、SSL CA証明書を提供する必要はありません。なぜなら、Node.jsはTLS（SSL）接続を確立する際に、デフォルトで組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するからです。

4. ランダムなパスワードを作成するには、**Generate Password**をクリックします。

   > **ヒント**
   >
   > 以前にパスワードを生成したことがある場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成することができます。

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

「{prefix}」というプレフィックスを取得した値に置き換えてください。

</div>

</SimpleTab>

### ステップ 4: コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

1. (前提条件) [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)をインストールします。

2. バンドルをビルドします:

   ```bash
   npm run build
   ```

3. サンプル Lambda 関数を呼び出します:

   ```bash
   sam local invoke --env-vars env.json -e events/event.json "tidbHelloWorldFunction"
   ```

4. ターミナルで出力を確認します。出力が以下のようになる場合、接続に成功しています:

   ```bash
   {"statusCode":200,"body":"{\"results\":[{\"Hello World\":\"Hello World\"}]}"}
   ```

接続が成功したことを確認したら、[次のセクション](#aws-lambda-関数をデプロイする)に従って AWS Lambda 関数をデプロイできます。

## AWS Lambda 関数をデプロイする {#deploy-the-aws-lambda-function}

AWS Lambda 関数をデプロイするには、[SAM CLI](#sam-cli-デプロイ-推奨)または[Web コンソール](#web-コンソール-デプロイ)のいずれかを使用できます。

### SAM CLI デプロイ (推奨) {#sam-cli-deployment-recommended}

1. ([前提条件](#前提条件)) [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)をインストールします。

2. バンドルをビルドします:

   ```bash
   npm run build
   ```

3. [`template.yml`](https://github.com/tidb-samples/tidb-aws-lambda-quickstart/blob/main/template.yml)の環境変数を更新します:

   ```yaml
   Environment:
     Variables:
       TIDB_HOST: {tidb_server_host}
       TIDB_PORT: 4000
       TIDB_USER: {prefix}.root
       TIDB_PASSWORD: {password}
   ```

4. AWS 環境変数を設定します ([Short-term credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-short-term.html)を参照してください):

   ```bash
   export AWS_ACCESS_KEY_ID={your_access_key_id}
   export AWS_SECRET_ACCESS_KEY={your_secret_access_key}
   export AWS_SESSION_TOKEN={your_session_token}
   ```

5. AWS Lambda 関数をデプロイします:

   ```bash
   sam deploy --guided

   # 例:

   # Configuring SAM deploy
   # ======================

   #        Looking for config file [samconfig.toml] :  Not found

   #        Setting default arguments for 'sam deploy'
   #        =========================================
   #        Stack Name [sam-app]: tidb-aws-lambda-quickstart
   #        AWS Region [us-east-1]: 
   #        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
   #        Confirm changes before deploy [y/N]: 
   #        #SAM needs permission to be able to create roles to connect to the resources in your template
   #        Allow SAM CLI IAM role creation [Y/n]: 
   #        #Preserves the state of previously provisioned resources when an operation fails
   #        Disable rollback [y/N]: 
   #        tidbHelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
   #        tidbHelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
   #        tidbHelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
   #        tidbHelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
   #        Save arguments to configuration file [Y/n]: 
   #        SAM configuration file [samconfig.toml]: 
   #        SAM configuration environment [default]: 

   #        Looking for resources needed for deployment:
   #        Creating the required resources...
   #        Successfully created!
   ```

### Web コンソール デプロイ {#web-console-deployment}

1. AWS Lambda コンソールに移動します。

2. 「関数の作成」をクリックします。

3. 「一から作成」をクリックします。

4. 「関数名」を入力します。

5. 「ランタイム」を選択します。

6. 「関数の作成」をクリックします。

7. 「関数コード」セクションで、コードをアップロードする方法を選択します。

   - 「.zip ファイルをアップロード」を選択し、[`index.js`](https://github.com/tidb-samples/tidb-aws-lambda-quickstart/blob/main/index.js)と[`package.json`](https://github.com/tidb-samples/tidb-aws-lambda-quickstart/blob/main/package.json)を含む ZIP ファイルをアップロードします。

   - 「コードを直接入力」を選択し、[`index.js`](https://github.com/tidb-samples/tidb-aws-lambda-quickstart/blob/main/index.js)と[`package.json`](https://github.com/tidb-samples/tidb-aws-lambda-quickstart/blob/main/package.json)のコードをコピーして貼り付けます。

8. 「環境変数」セクションで、環境変数を設定します。

   - `TIDB_HOST`、`TIDB_PORT`、`TIDB_USER`、`TIDB_PASSWORD`を設定します。

9. 「基本設定」セクションで、メモリとタイムアウトを設定します。

10. 「関数の作成」をクリックします。

11. 「テスト」をクリックします。

12. 「テストイベントを作成」をクリックします。

13. 「イベント名」を入力します。

14. 「作成」をクリックします。

15. 「テスト」をクリックします。

16. 「実行結果」を確認します。

17. 「関数の作成」をクリックします。

18. 「トリガーを追加」をクリックします。

19. 「トリガーを選択」で「API Gateway」を選択します。

20. 「API」を選択します。

21. 「API の作成」をクリックします。

22. 「API 名」を入力します。

23. 「アクション」をクリックします。

24. 「API の作成」をクリックします。

25. 「API の作成」をクリックします。

26. 「API の作成」をクリックします。

27. 「API の作成」をクリックします。

28. 「API の作成」をクリックします。

29. 「API の作成」をクリックします。

30. 「API の作成」をクリックします。

31. 「API の作成」をクリックします。

32. 「API の作成」をクリックします。

33. 「API の作成」をクリックします。

34. 「API の作成」をクリックします。

35. 「API の作成」をクリックします。

36. 「API の作成」をクリックします。

37. 「API の作成」をクリックします。

38. 「API の作成」をクリックします。

39. 「API の作成」をクリックします。

40. 「API の作成」をクリックします。

41. 「API の作成」をクリックします。

42. 「API の作成」をクリックします。

43. 「API の作成」をクリックします。

44. 「API の作成」をクリックします。

45. 「API の作成」をクリックします。

46. 「API の作成」をクリックします。

47. 「API の作成」をクリックします。

48. 「API の作成」をクリックします。

49. 「API の作成」をクリックします。

50. 「API の作成」をクリックします。

51. 「API の作成」をクリックします。

52. 「API の作成」をクリック

53. バンドルをビルドする：

    ```bash
    npm run build

    # AWS Lambda用にバンドルする
    # =====================
    # dist/index.zip
    ```

54. [AWS Lambdaコンソール](https://console.aws.amazon.com/lambda/home#/functions)にアクセスする。

55. [Lambda関数の作成](https://docs.aws.amazon.com/lambda/latest/dg/lambda-nodejs.html)に従って、Node.js Lambda関数を作成する。

56. [Lambdaデプロイメントパッケージ](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-package.html#gettingstarted-package-zip)に従って、`dist/index.zip`ファイルをアップロードする。

57. Lambda関数で[対応する接続文字列をコピーして設定する](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)。

    1. Lambdaコンソールの[Functions](https://console.aws.amazon.com/lambda/home#/functions)ページで、**Configuration**タブを選択し、**Environment variables**を選択する。
    2. **Edit**を選択する。
    3. データベースへのアクセス資格情報を追加するには、次の操作を行う：
       - **Add environment variable**を選択し、**Key**に`TIDB_HOST`を入力し、**Value**にホスト名を入力する。
       - **Add environment variable**を選択し、**Key**に`TIDB_PORT`を入力し、**Value**にポート番号を入力する（デフォルトは4000）。
       - **Add environment variable**を選択し、**Key**に`TIDB_USER`を入力し、**Value**にユーザー名を入力する。
       - **Add environment variable**を選択し、**Key**に`TIDB_PASSWORD`を入力し、**Value**にデータベースを作成した際に選択したパスワードを入力する。
       - **Save**を選択する。

## サンプルコードスニペット {#sample-code-snippets}

以下のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードと実行方法については、[tidb-samples/tidb-aws-lambda-quickstart](https://github.com/tidb-samples/tidb-aws-lambda-quickstart)リポジトリを参照してください。

### TiDBに接続する {#connect-to-tidb}

以下のコードは、環境変数で定義されたオプションを使用してTiDBに接続します：

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

以下のクエリは単一の `Player` レコードを作成し、`ResultSetHeader` オブジェクトを返します。

```typescript
const [rsh] = await pool.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100]);
console.log(rsh.insertId);
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

次のクエリは、ID `1`の単一の `Player` レコードを返します。

```typescript
const [rows] = await pool.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1]);
console.log(rows[0]);
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリでは、IDが`1`の`Player`に`50`のコインと`50`の商品が追加されます：

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

- データベース接続を管理するために[接続プール](https://github.com/sidorares/node-mysql2#using-connection-pools)を使用することで、頻繁に接続を確立し破棄することによるパフォーマンスのオーバーヘッドを減らすことができます。
- SQLインジェクションを防ぐために、[プリペアドステートメント](https://github.com/sidorares/node-mysql2#using-prepared-statements)を使用することをお勧めします。
- 複雑なSQL文が多く含まれないシナリオでは、[Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、または[Prisma](https://www.prisma.io/)などのORMフレームワークを使用することで、開発効率を大幅に向上させることができます。
- アプリケーションのRESTful APIを構築する場合は、[AWS LambdaとAPI Gatewayを使用することをお勧めします](https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html)。
- TiDB ServerlessとAWS Lambdaを使用して高性能なアプリケーションを設計する場合は、[このブログ](https://aws.amazon.com/blogs/apn/designing-high-performance-applications-using-serverless-tidb-cloud-and-aws-lambda/)を参照してください。

## 次のステップ {#next-steps}

- AWS Lambda FunctionでTiDBを使用する方法の詳細については、[TiDB-Lambda-integration/aws-lambda-bookstore Demo](https://github.com/pingcap/TiDB-Lambda-integration/blob/main/aws-lambda-bookstore/README.md)を参照してください。また、AWS API Gatewayを使用してアプリケーションのRESTful APIを構築することもできます。
- `mysql2`の使用方法については、[mysql2のドキュメント](https://github.com/sidorares/node-mysql2/tree/master/documentation/en)を参照してください。
- AWS Lambdaの使用方法については、[LambdaのAWS開発者ガイド](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)を参照してください。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学び、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などを参照してください。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後、[TiDB認定](https://www.pingcap.com/education/certification/)を取得してください。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
