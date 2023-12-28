---
title: Get Started with Chat2Query API
summary: Learn how to use TiDB Cloud Chat2Query API to generate and execute SQL statements using AI by providing instructions.
---

# Chat2Query APIを使用する {#get-started-with-chat2query-api}

TiDB Cloudは、指示を提供することでAIを使用してSQLステートメントを生成し、実行することができるRESTfulインターフェースであるChat2Query APIを提供しています。その後、APIはクエリ結果を返します。

Chat2Query APIはHTTPSを介してのみアクセスでき、ネットワークを介して送信されるすべてのデータはTLSを使用して暗号化されることが保証されます。

> **Note:**
>
> Chat2Query APIは、[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターで利用可能です。[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターでChat2Query APIを使用するには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。

## ステップ1. Chat2Queryデータアプリを作成する {#step-1-create-a-chat2query-data-app}

プロジェクトのデータサービスページの[**データサービス**](https://tidbcloud.com/console/data-service)ページで、左ペインの<MDSvgIcon name="icon-create-data-app" /> **Create DataApp**をクリックします。データアプリ作成ダイアログが表示されます。

> **Tip:**
>
> クラスターの**Chat2Query**ページにいる場合は、右上隅の\*\*...\*\*をクリックし、**Access Chat2Query via API**を選択し、**New Chat2Query Data App**をクリックすることで、データアプリ作成ダイアログを開くこともできます。

2. ダイアログで、データアプリの名前を定義し、データソースとして使用するクラスターを選択し、**Chat2Query Data App**を**Data App**タイプとして選択します。オプションで、アプリの説明を書くこともできます。

3. **Create**をクリックします。

   新しく作成されたChat2Queryデータアプリが左ペインに表示されます。このデータアプリの下に、Chat2Queryエンドポイントのリストがあります。

## ステップ2. APIキーを作成する {#step-2-create-an-api-key}

エンドポイントを呼び出す前に、Chat2Queryデータアプリ用のAPIキーを作成する必要があります。このAPIキーは、エンドポイントがTiDB Cloudクラスター内のデータにアクセスするために使用されます。

APIキーを作成するには、次の手順を実行します。

1. [**データサービス**](https://tidbcloud.com/console/data-service)の左ペインで、Chat2Queryデータアプリをクリックして、右側の詳細を表示します。

2. **Authentication**エリアで、**Create API Key**をクリックします。

3. **Create API Key**ダイアログで、説明を入力し、APIキーの次のロールのいずれかを選択します。

   - `Chat2Query Admin`: APIキーがデータサマリーを管理し、指示に基づいてSQLステートメントを生成し、任意のSQLステートメントを実行できるようにします。

   - `Chat2Query Data Summary Management Role`: APIキーがデータサマリーを生成し、更新することができるようにします。

     > **Tip:**
     >
     > Chat2Query APIでは、データサマリーとは、AIによるデータベースの分析結果であり、データベースの説明、テーブルの説明、およびカラムの説明を含みます。データベースのデータサマリーを生成することで、指示を提供することでより正確な応答を得ることができます。

   - `Chat2Query SQL ReadOnly`: APIキーが指示に基づいてSQLステートメントを生成し、`SELECT` SQLステートメントを実行することができるようにします。

   - `Chat2Query SQL ReadWrite`: APIキーが指示に基づいてSQLステートメントを生成し、任意のSQLステートメントを実行することができるようにします。

4. **Next**をクリックします。公開鍵と秘密鍵が表示されます。

   秘密鍵をコピーして安全な場所に保存してください。このページを離れると、完全な秘密鍵を再度取得することはできません。

5. **Done**をクリックします。

## ステップ3. Chat2Queryエンドポイントを呼び出す {#step-3-call-chat2query-endpoints}

> **Note:**
>
> 各Chat2Queryデータアプリには、1日あたり100リクエストのレート制限があります。レート制限を超えると、APIは`429`エラーを返します。より多くのクォータをご希望の場合は、[リクエストを送信](https://support.pingcap.com/hc/en-us/requests/new?ticket_form_id=7800003722519)してください。

各Chat2Queryデータアプリには、次のエンドポイントがあります。

- Chat2Query v1エンドポイント: `/v1/chat2data`
- Chat2Query v2エンドポイント: `/v2`で始まるエンドポイント、例えば`/v2/dataSummaries`や`/v2/chat2data`など

> **Tip:**
>
> `/v1/chat2data`と比較して、`/v2/chat2data`は、`/v2/dataSummaries`を呼び出してデータベースを分析する必要があるため、`/v2/chat2data`が返す結果は一般的により正確です。

### エンドポイントのコード例を取得する {#get-the-code-example-of-an-endpoint}

TiDB Cloudは、Chat2Queryエンドポイントを素早く呼び出すためのコード例を提供しています。Chat2Queryエンドポイントのコード例を取得するには、次の手順を実行してください。

1. [**データサービス**](https://tidbcloud.com/console/data-service)ページの左側のペインで、Chat2Queryエンドポイントの名前をクリックします。

   エンドポイントのURL、コード例、およびリクエストメソッドなど、このエンドポイントを呼び出すための情報が右側に表示されます。

2. **Show Code Example**をクリックします。

3. 表示されたダイアログボックスで、エンドポイントを呼び出すために使用するクラスター、データベース、および認証方法を選択し、コード例をコピーします。

   > **Note:**
   >
   > `/v2/chat2data`および`/v2/jobs/{job_id}`の場合、認証方法を選択する必要はありません。

4. エンドポイントを呼び出すには、アプリケーションに例を貼り付け、例のパラメーターを自分のものに置き換え（たとえば、`${PUBLIC_KEY}`と`${PRIVATE_KEY}`のプレースホルダーをAPIキーに置き換える）、実行します。

### Chat2Query v2エンドポイントを呼び出す {#call-chat2query-v2-endpoints}

TiDB Cloudデータサービスは、次のChat2Query v2エンドポイントを提供しています。

| メソッド | エンドポイント             | 説明                                                                    |
| ---- | ------------------- | --------------------------------------------------------------------- |
| POST | `/v2/dataSummaries` | このエンドポイントは、人工知能を使用してデータベーススキーマ、テーブルスキーマ、およびカラムスキーマのデータサマリーを生成します。     |
| POST | `/v2/chat2data`     | このエンドポイントを使用すると、データサマリーIDと指示を提供することで、人工知能を使用してSQLステートメントを生成および実行できます。 |
| GET  | `/v2/jobs/{job_id}` | このエンドポイントを使用すると、データサマリー生成ジョブのステータスをクエリできます。                           |

次のセクションでは、これらのエンドポイントを呼び出す方法について説明します。

#### 1. `/v2/dataSummaries`を呼び出してデータサマリーを生成する {#1-generate-a-data-summary-by-calling-v2-datasummaries}

`/v2/chat2data`を呼び出す前に、AIによってデータベースを分析し、データサマリーを生成するために、まず`/v2/dataSummaries`を呼び出してください。これにより、`/v2/chat2data`はSQL生成時により良いパフォーマンスを得ることができます。

次に、`sp500insight`データベースを分析し、データベースのデータサマリーを生成するために`/v2/chat2data`を呼び出すコード例を示します。

```bash
curl --digest --user ${PUBLIC_KEY}:${PRIVATE_KEY} --request POST 'https://<region>.data.dev.tidbcloud.com/api/v1beta/app/chat2query-<ID>/endpoint/v2/dataSummaries'\
 --header 'content-type: application/json'\
 --data-raw '{
    "cluster_id": "10939961583884005252",
    "database": "sp500insight"
}'
```

前述の例では、リクエストボディは次のプロパティを持つJSONオブジェクトです：

- `cluster_id`：*string*。TiDBクラスターのユニークな識別子。
- `database`：*string*。データベースの名前。

例として、次のようなレスポンスが返されます：

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

#### 2. `/v2/jobs/{job_id}`を呼び出して解析状況を確認する {#2-check-the-analysis-status-by-calling-v2-jobs-job-id}

`/v2/dataSummaries` APIは非同期です。大きなデータセットを持つデータベースの場合、データベースの解析が完了し、完全なデータサマリーが返されるまで数分かかる場合があります。

データベースの解析状況を確認するには、次のように`/v2/jobs/{job_id}`エンドポイントを呼び出すことができます。

```bash
curl --digest --user ${PUBLIC_KEY}:${PRIVATE_KEY} --request GET 'https://<region>.data.dev.tidbcloud.com/api/v1beta/app/chat2query-<ID>`/endpoint/v2/jobs/{job_id}'\
 --header 'content-type: application/json'
```

例えば、次のようなレスポンスがあります：

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

もし `"status"` が `"done"` であれば、完全なデータサマリーが利用可能になり、`/v2/chat2data` を呼び出すことでこのデータベースに対してSQLステートメントを生成して実行することができます。そうでない場合は、完了するまで解析ステータスを待ち、後で確認する必要があります。

レスポンスでは、`DataSummaryObject` は与えられたデータベースのAI探索情報を表します。`DataSummaryObject` の構造は以下の通りです：

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

#### 3. `/v2/chat2data`を呼び出してSQLステートメントを生成し、実行する {#3-generate-and-execute-sql-statements-by-calling-v2-chat2data}

データベースのデータサマリーが準備できたら、クラスターID、データベース名、および質問を指定して`/v2/chat2data`を呼び出すことで、SQLステートメントを生成し実行することができます。

例えば：

```bash
curl --digest --user ${PUBLIC_KEY}:${PRIVATE_KEY} --request POST 'https://<region>.data.dev.tidbcloud.com/api/v1beta/app/chat2query-<ID>/endpoint/v2/chat2data'\
 --header 'content-type: application/json'\
 --data-raw '{
  "cluster_id": "10939961583884005252",
  "database": "sp500insight",
  "raw_question": "<Your question to generate data>"
}'
```

前述のコードでは、リクエストボディは次のプロパティを持つJSONオブジェクトです：

- `cluster_id`：*string*。TiDBクラスターの一意の識別子。
- `database`：*string*。データベースの名前。
- `raw_question`：*string*。実行したいクエリを記述する自然言語。

例として、次のようなレスポンスが返されます：

```json
{
  "code": 200,
  "msg": "",
  "result": {
    "job_id": "3966d5bd95324a6283445e3a02ccd97c"
  }
}
```

もし以下のようにステータスコード `400` を含むレスポンスを受け取った場合、データのサマリーが準備されるまで少し待つ必要があります。

```json
{
    "code": 400,
    "msg": "Data summary is not ready, please wait for a while and retry",
    "result": {}
}
```

`/v2/chat2data` APIは非同期です。`/v2/jobs/{job_id}`エンドポイントを呼び出すことでジョブの状態を確認できます。

```bash
curl --digest --user ${PUBLIC_KEY}:${PRIVATE_KEY} --request GET 'https://<region>.data.dev.tidbcloud.com/api/v1beta/app/chat2query-<ID>/endpoint/v2/jobs/{job_id}'\
 --header 'content-type: application/json'
```

例えば、次のようなレスポンスがあります：

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

### Chat2Data v1エンドポイントを呼び出す {#call-the-chat2data-v1-endpoint}

TiDB Cloudデータサービスは、次のChat2Query v1エンドポイントを提供しています：

| メソッド | エンドポイント         | 説明                                                                    |
| ---- | --------------- | --------------------------------------------------------------------- |
| POST | `/v1/chat2data` | このエンドポイントを使用すると、ターゲットデータベース名と指示を提供して、人工知能を使用してSQLステートメントを生成および実行できます。 |

`/v1/chat2data`エンドポイントを直接呼び出すことで、SQLステートメントを生成および実行できます。`/v2/chat2data`と比較して、`/v1/chat2data`はより高速なレスポンスを提供しますが、パフォーマンスは低下します。

TiDB Cloudは、エンドポイントを呼び出すためのコード例を生成します。コード例を取得して実行するには、[エンドポイントのコード例を取得する](#エンドポイントのコード例を取得する)を参照してください。

`/v1/chat2data`を呼び出す際には、次のパラメータを置き換える必要があります：

- `${PUBLIC_KEY}`と`${PRIVATE_KEY}`のプレースホルダをAPIキーに置き換えます。
- `<your table name, optional>`のプレースホルダをクエリしたいテーブル名に置き換えます。テーブル名を指定しない場合、AIはデータベース内のすべてのテーブルをクエリします。
- `<your instruction>`のプレースホルダを、AIにSQLステートメントを生成および実行するよう指示したい内容に置き換えます。

> **Note:**
>
> 各Chat2Query Data Appには、1日あたり100リクエストのレート制限があります。レート制限を超えると、APIは`429`エラーを返します。より多くのクォータをご希望の場合は、[サポートチームにリクエストを送信](https://support.pingcap.com/hc/en-us/requests/new?ticket_form_id=7800003722519)してください。
> `Chat2Query Data Summary Management Role`の役割を持つAPIキーは、Chat2Data v1エンドポイントを呼び出すことはできません。

次のコード例は、`sp500insight.users`テーブルにいくつのユーザーがいるかをカウントするために使用されます：

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

前述の例では、リクエストボディは次のプロパティを持つJSONオブジェクトです：

- `cluster_id`：*string*。TiDBクラスターの一意の識別子。
- `database`：*string*。データベースの名前。
- `tables`：*array*。 (オプション) クエリを実行するテーブル名のリスト。
- `instruction`：*string*。クエリを説明する自然言語の命令。

レスポンスは次のようになります：

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

APIコールが成功しない場合、`200`以外のステータスコードが返されます。以下は`500`ステータスコードの例です。

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

## 詳細を学ぶ {#learn-more}

- [APIキーの管理](/tidb-cloud/data-service-api-key.md)
- [データサービスのレスポンスとステータスコード](/tidb-cloud/data-service-response-and-status-code.md)
