---
title: Get Started with Chat2Query API
summary: Learn how to use TiDB Cloud Chat2Query API to generate and execute SQL statements using AI by providing instructions.
---

# Chat2Query APIのはじめ方 {#get-started-with-chat2query-api}

TiDB CloudはChat2Query APIを提供しており、RESTfulインターフェースを使用してAIを利用してSQLステートメントを生成および実行することができます。その後、APIはクエリの結果を返します。

Chat2Query APIはHTTPSを介してのみアクセスでき、ネットワークを介して送信されるすべてのデータがTLSを使用して暗号化されることを保証します。

> **Note:**
>
> Chat2Query APIは[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターで利用可能です。[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターでChat2Query APIを使用するには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

## ステップ1. Chat2Queryデータアプリの作成 {#step-1-create-a-chat2query-data-app}

プロジェクトのデータアプリを作成するには、次の手順を実行します。

1. プロジェクトの[**データサービス**](https://tidbcloud.com/console/data-service)ページで、左ペインの<MDSvgIcon name="icon-create-data-app" /> **Create DataApp**をクリックします。データアプリ作成ダイアログが表示されます。

   > **Tip:**
   >
   > クラスターの**Chat2Query**ページにいる場合は、右上隅の\*\*...\*\*をクリックし、**Access Chat2Query via API**を選択し、**New Chat2Query Data App**をクリックすることで、データアプリ作成ダイアログを開くこともできます。

2. ダイアログで、データアプリの名前を定義し、希望するクラスターをデータソースとして選択し、**Chat2Query Data App**を**Data App**タイプとして選択します。オプションで、アプリの説明を記述することもできます。

3. **Create**をクリックします。

   新しく作成されたChat2Queryデータアプリが左ペインに表示されます。このデータアプリの下に、Chat2Queryエンドポイントのリストが表示されます。

## ステップ2. APIキーの作成 {#step-2-create-an-api-key}

エンドポイントを呼び出す前に、Chat2Queryデータアプリ用のAPIキーを作成する必要があります。このAPIキーは、TiDB Cloudクラスター内のデータにアクセスするためにエンドポイントによって使用されます。

APIキーを作成するには、次の手順を実行します。

1. [**データサービス**](https://tidbcloud.com/console/data-service)の左ペインで、Chat2Queryデータアプリをクリックして、右側で詳細を表示します。

2. **Authentication**エリアで、**Create API Key**をクリックします。

3. **Create API Key**ダイアログで、説明を入力し、次の役割のいずれかをAPIキーに選択します。

   - `Chat2Query Admin`: APIキーがデータサマリーの管理、指示に基づいたSQLステートメントの生成、および任意のSQLステートメントの実行を許可します。

   - `Chat2Query Data Summary Management Role`: APIキーがデータサマリーの生成と更新のみを許可します。

     > **Tip:**
     >
     > Chat2Query APIでは、データサマリーはAIによるデータベースの分析結果であり、データベースの説明、テーブルの説明、およびカラムの説明を含みます。データベースのデータサマリーを生成することで、指示に基づいてSQLステートメントを生成する際により正確な応答を得ることができます。

   - `Chat2Query SQL ReadOnly`: APIキーが指示に基づいたSQLステートメントの生成と`SELECT` SQLステートメントの実行のみを許可します。

   - `Chat2Query SQL ReadWrite`: APIキーが指示に基づいたSQLステートメントの生成と任意のSQLステートメントの実行を許可します。

4. **Next**をクリックします。公開鍵と秘密鍵が表示されます。

   このページを離れる前に、秘密鍵を安全な場所にコピーして保存してください。このページを離れると、再び完全な秘密鍵を取得することはできません。

5. **Done**をクリックします。

## ステップ3. Chat2Queryエンドポイントの呼び出し {#step-3-call-chat2query-endpoints}

> **Note:**
>
> 各Chat2Queryデータアプリには、1日あたり100リクエストのレート制限があります。レート制限を超えると、APIは`429`エラーを返します。より多くのクォータをご希望の場合は、[リクエストを送信](https://support.pingcap.com/hc/en-us/requests/new?ticket_form_id=7800003722519)してください。

各Chat2Queryデータアプリには、次のエンドポイントがあります。

- Chat2Query v1エンドポイント: `/v1/chat2data`
- Chat2Query v2エンドポイント: `/v2`で始まるエンドポイント、例えば`/v2/dataSummaries`および`/v2/chat2data`

> **Tip:**
>
> `/v1/chat2data`と比較して、`/v2/chat2data`は最初に`/v2/dataSummaries`を呼び出してデータベースを分析する必要があるため、`/v2/chat2data`が一般的により正確な結果を返します。

### エンドポイントのコード例を取得 {#get-the-code-example-of-an-endpoint}

TiDB Cloudでは、Chat2Queryエンドポイントを迅速に呼び出すためのコード例を提供しています。Chat2Queryエンドポイントのコード例を取得するには、次の手順を実行します。

1. [**データサービス**](https://tidbcloud.com/console/data-service)ページの左ペインで、Chat2Queryエンドポイントの名前をクリックします。

   エンドポイントの呼び出しに関する情報が右側に表示されます。エンドポイントURL、コード例、およびリクエストメソッドなどが表示されます。

2. **Show Code Example**をクリックします。

3. 表示されたダイアログボックスで、エンドポイントを呼び出す際に使用するクラスター、データベース、および認証方法を選択し、コード例をコピーします。

   > **Note:**
   >
   > `/v2/chat2data`および`/v2/jobs/{job_id}`の場合、認証方法のみを選択する必要があります。

4. エンドポイントを呼び出すには、アプリケーションに例を貼り付け、例のパラメータを自分のものに置き換え（たとえば`${PUBLIC_KEY}`および`${PRIVATE_KEY}`のプレースホルダをAPIキーに置き換え）して実行します。

### Chat2Query v2エンドポイントの呼び出し {#call-chat2query-v2-endpoints}

TiDB Cloudデータサービスでは、次のChat2Query v2エンドポイントが提供されています。

| メソッド | エンドポイント             | 説明                                                                |
| ---- | ------------------- | ----------------------------------------------------------------- |
| POST | `/v2/dataSummaries` | このエンドポイントは、人工知能を使用してデータベーススキーマ、テーブルスキーマ、およびカラムスキーマのデータサマリーを生成します。 |
| POST | `/v2/chat2data`     | このエンドポイントは、データサマリーIDと指示を提供して人工知能を使用してSQLステートメントを生成および実行することができます。 |
| GET  | `/v2/jobs/{job_id}` | このエンドポイントは、データサマリー生成ジョブのステータスをクエリすることができます。                       |

次のセクションでは、これらのエンドポイントの呼び出し方について学びます。

#### 1. `/v2/dataSummaries`を呼び出してデータサマリーを生成 {#1-generate-a-data-summary-by-calling-v2-datasummaries}

`/v2/chat2data`を呼び出す前に、AIにデータベースを分析させ、`/v2/dataSummaries`を呼び出して最初にデータサマリーを生成し、後で`/v2/chat2data`でSQL生成のパフォーマンスを向上させることができます。

以下は、`/v2/chat2data`を呼び出して`sp500insight`データベースを分析し、データベースのデータサマリーを生成するコード例です：

```bash
curl --digest --user ${PUBLIC_KEY}:${PRIVATE_KEY} --request POST 'https://<region>.data.dev.tidbcloud.com/api/v1beta/app/chat2query-<ID>/endpoint/v2/dataSummaries'\
 --header 'content-type: application/json'\
 --data-raw '{
    "cluster_id": "10939961583884005252",
    "database": "sp500insight"
}'
```

先行する例では、リクエスト本文は次のプロパティを持つJSONオブジェクトです。

- `cluster_id`: *string*. TiDBクラスターのユニークな識別子。
- `database`: *string*. データベースの名前。

例として、次のような応答があります:

```json
{
  "code": 200,
  "msg": "",
  "result": {
    "data_summary_id": 481235,
    "job_id": "79c2b3d36c074943ab06a29e45dd5887"
  }
}
```

#### 2. `/v2/jobs/{job_id}` を呼び出して解析ステータスを確認します {#2-check-the-analysis-status-by-calling-v2-jobs-job-id}

`/v2/dataSummaries` API は非同期です。大規模なデータセットを持つデータベースの場合、データベースの解析が完了し、完全なデータサマリーが返されるまで数分かかる場合があります。

データベースの解析ステータスを確認するには、次のように `/v2/jobs/{job_id}` エンドポイントを呼び出すことができます。

```bash
curl --digest --user ${PUBLIC_KEY}:${PRIVATE_KEY} --request GET 'https://<region>.data.dev.tidbcloud.com/api/v1beta/app/chat2query-<ID>`/endpoint/v2/jobs/{job_id}'\
 --header 'content-type: application/json'
```

以下は例としてのレスポンスです：

```json
{
  "code": 200,
  "msg": "",
  "result": {
    "ended_at": 1699518950, // A UNIX timestamp indicating when the job is finished
    "job_id": "79c2b3d36c074943ab06a29e45dd5887",  // ID of current job
    "result": DataSummaryObject, // AI exploration information of the given database
    "status": "done" // Status of the current job
  }
}
```

もし`"status"`が`"done"`であれば、完全なデータサマリーが準備され、`/v2/chat2data`を呼び出すことでこのデータベースのためのSQLステートメントを生成して実行できます。そうでない場合は、完了するまで後で解析ステータスを待って確認する必要があります。

応答では、`DataSummaryObject`は与えられたデータベースのAI探査情報を表します。`DataSummaryObject`の構造は以下の通りです:

```json
{
    "cluster_id": 10939961583884005252, // Your cluster id
    "db_name": "sp500insight", // Database name
    "db_schema": { // Database schema information
        "users": { // A table named "users"
            "columns": { // Columns in table "users"
                "user_id": {
                    "default": null,
                    "description": "The unique identifier for each user.",
                    "name": "user_id",
                    "nullable": true,
                    "type": "int(11)"
                }
            },
            "description": "This table represents the user data and includes the date and time when each user was created.",
            "key_attributes": [ // Key attributes of table "user"
                "user_id",
            ],
            "primary_key": "id",
            "table_name": "users", // Table name in the database
        }
    },
    "entity": { // Entities abstracted by AI
        "users": {
            "attributes": ["user_id"],
            "involved_tables": ["users"],
            "name": "users",
            "summary": "This table represents the user data and includes the date and time when each user was created."
        }
    },
    "org_id": 30061,
    "project_id": 3198952,
    "short_summary": "Comprehensive finance data for analysis and decision-making.",
    "status": "done",
    "summary": "This data source contains information about companies, indexes, and historical stock price data. It is used for financial analysis, investment decision-making, and market research in the finance domain.",
    "summary_keywords": [
        "users"
    ],
    "table_relationship": {}
}
```

#### 3. Generate and execute SQL statements by calling `/v2/chat2data` {#3-generate-and-execute-sql-statements-by-calling-v2-chat2data}

データベースのデータサマリーが準備できたら、クラスターID、データベース名、および質問を提供して `/v2/chat2data` を呼び出すことで、SQLステートメントを生成および実行できます。

たとえば：

```bash
curl --digest --user ${PUBLIC_KEY}:${PRIVATE_KEY} --request POST 'https://<region>.data.dev.tidbcloud.com/api/v1beta/app/chat2query-<ID>/endpoint/v2/chat2data'\
 --header 'content-type: application/json'\
 --data-raw '{
  "cluster_id": "10939961583884005252",
  "database": "sp500insight",
  "raw_question": "<Your question to generate data>"
}'
```

先行するコードでは、リクエストボディは次のプロパティを持つJSONオブジェクトです。

- `cluster_id`: *string*. TiDBクラスターのユニークな識別子。
- `database`: *string*. データベースの名前。
- `raw_question`: *string*. 欲しいクエリを記述する自然言語。

例として、次のような応答があります：

```json
{
  "code": 200,
  "msg": "",
  "result": {
    "job_id": "3966d5bd95324a6283445e3a02ccd97c"
  }
}
```

もし以下のようにステータスコード `400` で応答を受け取った場合、それはデータの要約が準備されるのを待つ必要があることを意味します。

```json
{
    "code": 400,
    "msg": "Data summary is not ready, please wait for a while and retry",
    "result": {}
}
```

`/v2/chat2data` APIは非同期です。`/v2/jobs/{job_id}`エンドポイントを呼び出すことで、ジョブの状態を確認できます。

```bash
curl --digest --user ${PUBLIC_KEY}:${PRIVATE_KEY} --request GET 'https://<region>.data.dev.tidbcloud.com/api/v1beta/app/chat2query-<ID>/endpoint/v2/jobs/{job_id}'\
 --header 'content-type: application/json'
```

以下は例としてのレスポンスです：

```json
{
  "code": 200,
  "msg": "",
  "result": {
    "ended_at": 1699581661,
    "job_id": "3966d5bd95324a6283445e3a02ccd97c",
    "result": {
      "question_id": "8c4c15cf-a808-45b8-bff7-2ca819a1b6d5",
      "raw_question": "count the users", // The original question you provide
      "task_tree": {
        "0": {
          "clarified_task": "count the users", // Task that AI understands
          "description": "",
          "columns": [ // Columns that are queried in the generated SQL statement
            {
              "col": "user_count"
            }
          ],
          "rows": [ // Query result of generated SQL statement
            [
              "1"
            ]
          ],
          "sequence_no": 0,
          "sql": "SELECT COUNT(`user_id`) AS `user_count` FROM `users`;",
          "task": "count the users",
          "task_id": "0"
        }
      },
      "time_elapsed": 3.854671001434326
    },
    "status": "done"
  }
}
```

### チャット2データ v1 エンドポイントを呼び出す {#call-the-chat2data-v1-endpoint}

TiDB Cloudデータサービスは、次のChat2Query v1エンドポイントを提供します。

| メソッド | エンドポイント         | 説明                                                                    |
| ---- | --------------- | --------------------------------------------------------------------- |
| POST | `/v1/chat2data` | このエンドポイントを使用すると、ターゲットデータベース名と指示を提供して、人工知能を使用してSQLステートメントを生成および実行できます。 |

`/v1/chat2data` エンドポイントを直接呼び出して、SQLステートメントを生成および実行できます。 `/v2/chat2data` と比較して、`/v1/chat2data` はより迅速な応答を提供しますが、パフォーマンスは低くなります。

TiDB Cloudは、エンドポイントを呼び出すためのコード例を生成します。例とコードを取得するには、[エンドポイントのコード例を取得](#get-the-code-example-of-an-endpoint)を参照してください。

`/v1/chat2data` を呼び出す際に、次のパラメータを置き換える必要があります。

- `${PUBLIC_KEY}` および `${PRIVATE_KEY}` のプレースホルダをAPIキーで置き換えます。
- `<your table name, optional>` のプレースホルダをクエリしたいテーブル名で置き換えます。テーブル名を指定しない場合、AIはデータベース内のすべてのテーブルをクエリします。
- `<your instruction>` のプレースホルダを、AIに生成および実行するSQLステートメントの指示で置き換えます。

> **Note:**
>
> 各Chat2Queryデータアプリには、1日あたり100件のリクエストの制限があります。制限を超えると、APIは `429` エラーを返します。より多くのクォータをご希望の場合は、[リクエストを送信](https://support.pingcap.com/hc/en-us/requests/new?ticket_form_id=7800003722519)してください。
> `Chat2Queryデータサマリ管理ロール` を持つAPIキーは、Chat2Data v1エンドポイントを呼び出すことができません。

次のコード例は、`sp500insight.users` テーブルにいくつのユーザーがいるかをカウントするために使用されます：

```bash
curl --digest --user ${PUBLIC_KEY}:${PRIVATE_KEY} --request POST 'https://<region>.data.dev.tidbcloud.com/api/v1beta/app/chat2query-<ID>/endpoint/chat2data'\
 --header 'content-type: application/json'\
 --data-raw '{
    "cluster_id": "10939961583884005252",
    "database": "sp500insight",
    "tables": ["users"],
    "instruction": "count the users"
}'
```

先行する例では、リクエスト本文は次のプロパティを持つJSONオブジェクトです。

- `cluster_id`: *string*. TiDBクラスターのユニークな識別子。
- `database`: *string*. データベースの名前。
- `tables`: *array*. (オプション) クエリするテーブル名のリスト。
- `instruction`: *string*. 欲しいクエリを記述する自然言語の指示。

レスポンスは次のようになります:

```json
{
  "type": "chat2data_endpoint",
  "data": {
    "columns": [
      {
        "col": "COUNT(`user_id`)",
        "data_type": "BIGINT",
        "nullable": false
      }
    ],
    "rows": [
      {
        "COUNT(`user_id`)": "1"
      }
    ],
    "result": {
      "code": 200,
      "message": "Query OK!",
      "start_ms": 1699529488292,
      "end_ms": 1699529491901,
      "latency": "3.609656403s",
      "row_count": 1,
      "row_affect": 0,
      "limit": 1000,
      "sql": "SELECT COUNT(`user_id`) FROM `users`;",
      "ai_latency": "3.054822491s"
    }
  }
}
```

もしAPIの呼び出しが成功しない場合、`200`以外のステータスコードが返されます。以下は`500`ステータスコードの例です:

```json
{
  "type": "chat2data_endpoint",
  "data": {
    "columns": [],
    "rows": [],
    "result": {
      "code": 500,
      "message": "internal error! defaultPermissionHelper: rpc error: code = DeadlineExceeded desc = context deadline exceeded",
      "start_ms": "",
      "end_ms": "",
      "latency": "",
      "row_count": 0,
      "row_affect": 0,
      "limit": 0
    }
  }
}
```

## Learn more {#learn-more}

- [APIキーの管理](/tidb-cloud/data-service-api-key.md)
- [データサービスの応答とステータスコード](/tidb-cloud/data-service-response-and-status-code.md)
