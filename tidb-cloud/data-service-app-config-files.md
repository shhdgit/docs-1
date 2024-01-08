---
title: Data App Configuration Files
summary: This document describes the configuration files of Data App in TiDB Cloud.
---

# データアプリのコンフィグレーションファイル {#data-app-configuration-files}

このドキュメントでは、TiDB Cloud内の[データアプリ](/tidb-cloud/tidb-cloud-glossary.md#data-app)のコンフィグレーションファイルについて説明します。

もし[データアプリをGitHubに接続](/tidb-cloud/data-service-manage-github-connection.md)した場合、GitHubの指定されたディレクトリにデータアプリのコンフィグレーションファイルが以下のように見つかります：

```
├── <Your Data App directory>
│   ├── data_sources
│   │   └── cluster.json
│   ├── dataapp_config.json
│   ├── http_endpoints
│   │   ├── config.json
│   │   └── sql
│   │       ├── <method>-<endpoint-path1>.sql
│   │       ├── <method>-<endpoint-path2>.sql
│   │       └── <method>-<endpoint-path3>.sql
```

## データソースのコンフィグレーション {#data-source-configuration}

データアプリのデータソースは、リンクされたTiDBクラスタから取得されます。`data_sources/cluster.json`でデータソースのコンフィグレーションを見つけることができます。

```
├── <Your Data App directory>
│   ├── data_sources
│   │   └── cluster.json
```

各データアプリについて、1つまたは複数のTiDBクラスタにリンクすることができます。

以下は`cluster.json`の例のコンフィグレーションです。この例では、このデータアプリには2つのリンクされたクラスタがあります。

```json
[
  {
    "cluster_id": <Cluster ID1>
  },
  {
    "cluster_id": <Cluster ID2>
  }
]
```

フィールドの説明は次のとおりです：

| フィールド        | タイプ | 説明                                                                                                                                                        |
| ------------ | --- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cluster_id` | 整数  | TiDBクラスタのIDです。クラスタのURLから取得できます。たとえば、クラスタのURLが `https://tidbcloud.com/console/clusters/1234567891234567890/overview` の場合、クラスタIDは `1234567891234567890` です。 |

## データアプリの構成 {#data-app-configuration}

データアプリのプロパティには、App ID、名前、タイプが含まれます。プロパティは `dataapp_config.json` ファイルで見つけることができます。

```
├── <Your Data App directory>
│   ├── dataapp_config.json
```

以下は`dataapp_config.json`のコンフィグレーションの例です。

```json
{
  "app_id": "<Data App ID>",
  "app_name": "<Data App name>",
  "app_type": "dataapi",
  "app_version": "<Data App version>",
  "description": "<Data App description>"
}
```

各フィールドの説明は次のとおりです：

| フィールド         | タイプ    | 説明                                                                                                                            |
| ------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `app_id`      | String | データアプリID。`dataapp_config.json`ファイルが別のデータアプリからコピーされ、現在のデータアプリのIDに更新する場合を除き、このフィールドを変更しないでください。それ以外の場合、この変更によってトリガーされた展開は失敗します。 |
| `app_name`    | String | データアプリ名。                                                                                                                      |
| `app_type`    | String | データアプリタイプは、`"dataapi"`のみです。                                                                                                   |
| `app_version` | String | データアプリバージョンは、`"<major>.<minor>.<patch>"`形式です。たとえば、`"1.0.0"`。                                                                  |
| `description` | String | データアプリの説明。                                                                                                                    |

## HTTPエンドポイントの構成 {#http-endpoint-configuration}

データアプリディレクトリ内で、`http_endpoints/config.json`にエンドポイントの構成と、`http_endpoints/sql/<method>-<endpoint-name>.sql`にSQLファイルを見つけることができます。

```
├── <Your Data App directory>
│   ├── http_endpoints
│   │   ├── config.json
│   │   └── sql
│   │       ├── <method>-<endpoint-path1>.sql
│   │       ├── <method>-<endpoint-path2>.sql
│   │       └── <method>-<endpoint-path3>.sql
```

### エンドポイントのコンフィグレーション {#endpoint-configuration}

各データアプリには、1つ以上のエンドポイントが存在する場合があります。`http_endpoints/config.json` で、データアプリのすべてのエンドポイントの構成を見つけることができます。

以下は `config.json` の例の構成です。この例では、このデータアプリには2つのエンドポイントがあります。

```json
[
  {
    "name": "<Endpoint name1>",
    "description": "<Endpoint description1>",
    "method": "<HTTP method1>",
    "endpoint": "<Endpoint path1>",
    "data_source": {
      "cluster_id": <Cluster ID1>
    },
    "params": [],
    "settings": {
      "timeout": <Endpoint timeout>,
      "row_limit": <Maximum rows>,
      "enable_pagination": <0 | 1>,
      "cache_enabled": <0 | 1>,
      "cache_ttl": <time-to-live period>
    },
    "tag": "Default",
    "batch_operation": <0 | 1>,
    "sql_file": "<SQL file directory1>",
    "type": "sql_endpoint",
    "return_type": "json"
  },
  {
    "name": "<Endpoint name2>",
    "description": "<Endpoint description2>",
    "method": "<HTTP method2>",
    "endpoint": "<Endpoint path2>",
    "data_source": {
      "cluster_id": <Cluster ID2>
    },
    "params": [
      {
        "name": "<Parameter name>",
        "type": "<Parameter type>",
        "required": <0 | 1>,
        "default": "<Parameter default value>",
        "description": "<Parameter description>"
      }
    ],
    "settings": {
      "timeout": <Endpoint timeout>,
      "row_limit": <Maximum rows>,
      "enable_pagination": <0 | 1>,
      "cache_enabled": <0 | 1>,
      "cache_ttl": <time-to-live period>
    },
    "tag": "Default",
    "batch_operation": <0 | 1>,
    "sql_file": "<SQL file directory2>",
    "type": "sql_endpoint",
    "return_type": "json"
  }
]
```

各フィールドの説明は次のとおりです：

| フィールド                        | タイプ | 説明                                                                                                                                                                                                                                                                 |
| ---------------------------- | --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `name`                       | 文字列 | エンドポイント名。                                                                                                                                                                                                                                                          |
| `description`                | 文字列 | (オプション) エンドポイントの説明。                                                                                                                                                                                                                                                |
| `method`                     | 文字列 | エンドポイントのHTTPメソッド。 `GET` を使用してデータを取得し、 `POST` を使用してデータを作成または挿入し、 `PUT` を使用してデータを更新または変更し、 `DELETE` を使用してデータを削除できます。                                                                                                                                                 |
| `endpoint`                   | 文字列 | Data App内のエンドポイントの一意のパス。 パスには、文字、数字、アンダースコア（ `_` ）、およびスラッシュ（ `/` ）のみが使用でき、スラッシュ（ `/` ）で始まり、文字、数字、またはアンダースコア（ `_` ）で終わる必要があります。 たとえば、 `/my_endpoint/get_id` のようなものです。 パスの長さは64文字未満である必要があります。                                                                       |
| `cluster_id`                 | 文字列 | エンドポイントのTiDBクラスターのID。 TiDBクラスターのURLから取得できます。 たとえば、クラスターURLが `https://tidbcloud.com/console/clusters/1234567891234567890/overview` の場合、クラスターIDは `1234567891234567890` です。                                                                                           |
| `params`                     | 配列  | エンドポイントで使用されるパラメーター。 パラメーターを定義することで、エンドポイントを介してクエリ内のパラメーター値を動的に置換できます。 `params` では、1つまたは複数のパラメーターを定義できます。 各パラメーターについては、その `name` 、 `type` 、 `required` 、および `default` フィールドを定義する必要があります。 エンドポイントにパラメーターが必要ない場合は、 `"params": []` のようにして `params` を空のままにしておくことができます。 |
| `params.name`                | 文字列 | パラメーターの名前。 名前には文字、数字、アンダースコア（ `_` ）のみを含めることができ、文字またはアンダースコア（ `_` ）で始める必要があります。 ページネーションのリクエスト結果のために予約されているパラメーター名である `page` および `page_size` を使用しないでください。                                                                                                           |
| `params.type`                | 文字列 | パラメーターのデータ型。 サポートされている値は `string` 、 `number` 、 `integer` 、 `boolean` 、および `array` です。 `string` タイプのパラメーターを使用する場合、引用符（ `'` または `"` ）を追加する必要はありません。 たとえば、 `foo` は `string` タイプの場合に有効であり、 `"foo"` は `"\"foo\""` として処理されます。                                            |
| `params.required`            | 整数  | リクエストでパラメーターが必要かどうかを指定します。 サポートされている値は `0` （必要なし）および `1` （必要）です。 デフォルト値は `0` です。                                                                                                                                                                                   |
| `params.enum`                | 文字列 | (オプション) パラメーターの値オプションを指定します。 このフィールドは、 `params.type` が `string` 、 `number` 、または `integer` に設定されている場合にのみ有効です。 複数の値を指定するには、コンマ（`,`）で区切ることができます。                                                                                                                      |
| `params.default`             | 文字列 | パラメーターのデフォルト値。 指定したパラメーターの型と一致することを確認してください。 そうでない場合、エンドポイントはエラーを返します。 `ARRAY` タイプのパラメーターのデフォルト値は文字列であり、複数の値を区切るためにコンマ（`,`）を使用できます。                                                                                                                                |
| `params.description`         | 文字列 | パラメーターの説明。                                                                                                                                                                                                                                                         |
| `settings.timeout`           | 整数  | ミリ秒単位のエンドポイントのタイムアウト。 デフォルト値は `30000` です。 `1` から `60000` の整数に設定できます。                                                                                                                                                                                               |
| `settings.row_limit`         | 整数  | エンドポイントが操作または返すことができる最大行数。 デフォルト値は `1000` です。 `batch_operation` が `0` に設定されている場合、 `1` から `2000` の整数に設定できます。 `batch_operation` が `1` に設定されている場合、 `1` から `100` の整数に設定できます。                                                                                           |
| `settings.enable_pagination` | 整数  | リクエストによって返される結果のページネーションを有効にするかどうかを制御します。 サポートされている値は `0` （無効）および `1` （有効）です。 デフォルト値は `0` です。                                                                                                                                                                      |
| `settings.cache_enabled`     | 整数  | `GET` リクエストで返される応答を指定された有効期間（TTL）内にキャッシュするかどうかを制御します。 サポートされている値は `0` （無効）および `1` （有効）です。 デフォルト値は `0` です。                                                                                                                                                          |
| `settings.cache_ttl`         | 整数  | `settings.cache_enabled` が `1` に設定されている場合のキャッシュされた応答の有効期間（TTL）を秒単位で指定します。 `30` から `600` の整数に設定できます。 TTL期間中に同じ `GET` リクエストを行うと、Data Serviceはデータベースからデータを再取得する代わりにキャッシュされた応答を直接返し、クエリのパフォーマンスが向上します。                                                                 |
| `tag`                        | 文字列 | エンドポイントのタグ。 デフォルト値は `"Default"` です。                                                                                                                                                                                                                                |
| `batch_operation`            | 整数  | エンドポイントをバッチモードで操作するかどうかを制御します。 サポートされている値は `0` （無効）および `1` （有効）です。 `1` に設定されている場合、単一のリクエストで複数の行を操作できます。 このオプションを有効にするには、リクエストメソッドが `POST` または `PUT` であることを確認してください。                                                                                                |
| `sql_file`                   | 文字列 | エンドポイントのSQLファイルディレクトリ。 たとえば、 `"sql/GET-v1.sql"` です。                                                                                                                                                                                                                |
| `type`                       | 文字列 | エンドポイントのタイプ。 `"sql_endpoint"` のみです。                                                                                                                                                                                                                                |
| `return_type`                | 文字列 | エンドポイントの応答形式。 `"json"` のみです。                                                                                                                                                                                                                                       |

### SQLファイルの構成 {#sql-file-configuration}

エンドポイントのSQLファイルは、エンドポイントを介してデータをクエリするためのSQLステートメントを指定します。 Data AppのエンドポイントSQLファイルは `http_endpoints/sql/` ディレクトリで見つけることができます。 各エンドポイントには、対応するSQLファイルがある必要があります。

SQLファイルの名前は `<method>-<endpoint-path>.sql` の形式で、 `<method>` と `<endpoint-path>` は [`http_endpoints/config.json`](#endpoint-configuration) での `method` と `endpoint` の構成と一致する必要があります。

SQLファイルでは、テーブル結合クエリ、複雑なクエリ、および集計関数などのステートメントを記述できます。 以下はSQLファイルの例です。

```sql
/* Getting Started:
Enter "USE {database};" before entering your SQL statements.
Type "--your question" + Enter to try out AI-generated SQL queries in the TiDB Cloud console.
Declare a parameter like "Where id = ${arg}".
*/
USE sample_data;
SELECT
  rank,
  company_name,
FROM
  global_fortune_500_2018_2022
WHERE
  country = ${country};
```

SQLファイルを作成する際は、次の点に注意してください。

- SQLファイルの先頭で、SQLステートメントでデータベースを指定する必要があります。例えば、`USE database_name;`のようになります。

- エンドポイントのパラメータを定義するには、SQLステートメントに`${variable-name}`のような変数プレースホルダーとして挿入することができます。

  先の例では、`${country}`がエンドポイントのパラメータとして使用されています。このパラメータを使用すると、エンドポイントのcurlコマンドでクエリするための希望する国を指定できます。

  > **Note:**
  >
  > - パラメータ名は大文字と小文字が区別されます。
  > - パラメータはテーブル名や列名にすることはできません。
  > - SQLファイル内のパラメータ名は、[`http_endpoints/config.json`](#endpoint-configuration)で構成されたパラメータ名と一致する必要があります。
