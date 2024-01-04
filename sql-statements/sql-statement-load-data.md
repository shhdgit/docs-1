---
title: LOAD DATA | TiDB SQL Statement Reference
summary: An overview of the usage of LOAD DATA for the TiDB database.
---

# LOAD DATA {#load-data}

`LOAD DATA` ステートメントは、TiDB テーブルにデータをバッチでロードします。

TiDB v7.0.0 から、`LOAD DATA` SQL ステートメントは次の機能をサポートしています。

- S3 および GCS からのデータのインポートをサポート
- 新しいパラメータ `FIELDS DEFINED NULL BY` の追加

> **Warning:**
>
> 新しいパラメータ `FIELDS DEFINED NULL BY` および S3 および GCS からのデータのインポートのサポートは実験的です。本番環境で使用しないことをお勧めします。この機能は事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHub の [issue](https://github.com/pingcap/tidb/issues) で報告できます。

## 概要 {#synopsis}

```ebnf+diagram
LoadDataStmt ::=
    'LOAD' 'DATA' LocalOpt 'INFILE' stringLit DuplicateOpt 'INTO' 'TABLE' TableName CharsetOpt Fields Lines IgnoreLines ColumnNameOrUserVarListOptWithBrackets LoadDataSetSpecOpt

LocalOpt ::= ('LOCAL')?

Fields ::=
    ('TERMINATED' 'BY' stringLit
    | ('OPTIONALLY')? 'ENCLOSED' 'BY' stringLit
    | 'ESCAPED' 'BY' stringLit
    | 'DEFINED' 'NULL' 'BY' stringLit ('OPTIONALLY' 'ENCLOSED')?)?
```

## パラメータ {#parameters}

### `LOCAL` {#local}

`LOCAL` を使用して、クライアント上のデータファイルを指定してインポートできます。ファイルパラメータはクライアント上のファイルシステムパスである必要があります。

TiDB Cloud を使用している場合、`LOAD DATA` ステートメントを使用してローカルのデータファイルをロードするには、TiDB Cloud に接続する際に接続文字列に `--local-infile` オプションを追加する必要があります。

- TiDB Serverless の接続文字列の例は次のとおりです：

  ```
  mysql --connect-timeout 15 -u '<user_name>' -h <host_name> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p<your_password> --local-infile
  ```

- TiDB Dedicated の接続文字列の例は次のとおりです：

  ```
  mysql --connect-timeout 15 --ssl-mode=VERIFY_IDENTITY --ssl-ca=<CA_path> --tls-version="TLSv1.2" -u root -h <host_name> -P 4000 -D test -p<your_password> --local-infile
  ```

### S3 および GCS ストレージ {#s3-and-gcs-storage}

<CustomContent platform="tidb">

`LOCAL` を指定しない場合、ファイルパラメータは [外部ストレージ](/br/backup-and-restore-storages.md) で詳細に説明されているように、有効な S3 または GCS パスである必要があります。

</CustomContent>

<CustomContent platform="tidb-cloud">

`LOCAL` を指定しない場合、ファイルパラメータは [外部ストレージ](https://docs.pingcap.com/tidb/stable/backup-and-restore-storages) で詳細に説明されているように、有効な S3 または GCS パスである必要があります。

</CustomContent>

データファイルが S3 または GCS に保存されている場合、個々のファイルをインポートするか、ワイルドカード文字 `*` を使用して複数のファイルを一致させてインポートできます。ワイルドカードはサブディレクトリ内のファイルを再帰的に処理しないことに注意してください。以下はいくつかの例です：

- 単一のファイルをインポート：`s3://<bucket-name>/path/to/data/foo.csv`
- 指定されたパス内のすべてのファイルをインポート：`s3://<bucket-name>/path/to/data/*`
- 指定されたパス内で `.csv` で終わるすべてのファイルをインポート：`s3://<bucket-name>/path/to/data/*.csv`
- 指定されたパス内で `foo` で始まるすべてのファイルをインポート：`s3://<bucket-name>/path/to/data/foo*`
- 指定されたパス内で `foo` で始まり `.csv` で終わるすべてのファイルをインポート：`s3://<bucket-name>/path/to/data/foo*.csv`

### `Fields`、`Lines`、および `Ignore Lines` {#fields-lines-and-ignore-lines}

`Fields` および `Lines` パラメータを使用してデータ形式の処理方法を指定できます。

- `FIELDS TERMINATED BY`：データの区切り文字を指定します。
- `FIELDS ENCLOSED BY`：データの囲み文字を指定します。
- `LINES TERMINATED BY`：行の終端文字を指定します。特定の文字で行を終了させたい場合に使用します。

`DEFINED NULL BY` を使用して、データファイル内で NULL 値がどのように表されるかを指定できます。

- MySQL の動作に従い、`ESCAPED BY` が null でない場合、たとえばデフォルト値 `\` が使用されている場合、`\N` は NULL 値と見なされます。
- `DEFINED NULL BY` を使用すると、`DEFINED NULL BY 'my-null'` のように、`my-null` が NULL 値と見なされます。
- `DEFINED NULL BY ... OPTIONALLY ENCLOSED` を使用すると、たとえば `DEFINED NULL BY 'my-null' OPTIONALLY ENCLOSED` のように、`my-null` および `"my-null"` (`ENCLOSED BY '"'` を想定) が NULL 値と見なされます。
- `DEFINED NULL BY` または `DEFINED NULL BY ... OPTIONALLY ENCLOSED` を使用せず、`ENCLOSED BY` を使用する場合、たとえば `ENCLOSED BY '"'` の場合、`NULL` が NULL 値と見なされます。この動作は MySQL と一致しています。
- それ以外の場合、NULL 値と見なされません。

次のデータ形式を例に取り上げます：

```
"bob","20","street 1"\r\n
"alice","33","street 1"\r\n
```

"input\_content": "もし `bob`、`20`、および `street 1` を抽出したい場合は、フィールドの区切り文字を `','` 、囲む文字を `'\"'` として指定してください。"

```sql
FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n'
```

もし、前述のパラメータを指定しない場合、インポートされたデータはデフォルトで以下のように処理されます:

```sql
FIELDS TERMINATED BY '\t' ENCLOSED BY '' ESCAPED BY '\\'
LINES TERMINATED BY '\n' STARTING BY ''
```

You can ignore the first `number` lines of a file by configuring the `IGNORE <number> LINES` parameter. For example, if you configure `IGNORE 1 LINES`, the first line of a file is ignored.

## Examples {#examples}

The following example imports data using `LOAD DATA`. Comma is specified as the field delimiter. The double quotation marks that enclose the data are ignored. The first line of the file is ignored.

<CustomContent platform="TiDB">

If you see `ERROR 1148 (42000): the used command is not allowed with this TiDB version`, refer to [ERROR 1148 (42000): the used command is not allowed with this TiDB version](/error-codes.md#mysql-native-error-messages) for troubleshooting.

</CustomContent>

<CustomContent platform="TiDB Cloud">

If you see `ERROR 1148 (42000): the used command is not allowed with this TiDB version`, refer to [ERROR 1148 (42000): the used command is not allowed with this TiDB version](https://docs.pingcap.com/tidb/stable/error-codes#mysql-native-error-messages) for troubleshooting.

</CustomContent>

```sql
LOAD DATA LOCAL INFILE '/mnt/evo970/data-sets/bikeshare-data/2017Q4-capitalbikeshare-tripdata.csv' INTO TABLE trips FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n' IGNORE 1 LINES (duration, start_date, end_date, start_station_number, start_station, end_station_number, end_station, bike_number, member_type);
```

```sql
Query OK, 815264 rows affected (39.63 sec)
Records: 815264  Deleted: 0  Skipped: 0  Warnings: 0
```

"`LOAD DATA`は、`FIELDS ENCLOSED BY`および`FIELDS TERMINATED BY`のパラメータとして、16進ASCII文字表現やバイナリASCII文字表現を使用することもサポートしています。次の例を参照してください:"

```sql
LOAD DATA LOCAL INFILE '/mnt/evo970/data-sets/bikeshare-data/2017Q4-capitalbikeshare-tripdata.csv' INTO TABLE trips FIELDS TERMINATED BY x'2c' ENCLOSED BY b'100010' LINES TERMINATED BY '\r\n' IGNORE 1 LINES (duration, start_date, end_date, start_station_number, start_station, end_station_number, end_station, bike_number, member_type);
```

上記の例では、`x'2c'`は`,`文字の16進表現であり、`b'100010'`は`"`文字のバイナリ表現です。

## MySQL互換性 {#mysql-compatibility}

`LOAD DATA`ステートメントの構文は、文字セットオプションを除いてMySQLと互換性がありますが、これらは解析されますが無視されます。構文の互換性の違いがある場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)することができます。

<CustomContent platform="tidb">

> **注意:**
>
> - TiDB v4.0.0より前のバージョンでは、`LOAD DATA`は20000行ごとにコミットされ、設定できません。
> - TiDB v4.0.0からv6.6.0までのバージョンでは、デフォルトでTiDBはすべての行を1つのトランザクションでコミットします。ただし、`LOAD DATA`ステートメントを固定数の行ごとにコミットする必要がある場合は、[`tidb_dml_batch_size`](/system-variables.md#tidb_dml_batch_size)を所定の行数に設定できます。
> - TiDB v7.0.0からは、`tidb_dml_batch_size`は`LOAD DATA`に影響を与えなくなり、TiDBはすべての行を1つのトランザクションでコミットします。
> - TiDB v4.0.0またはそれ以前のバージョンからアップグレードした場合、`ERROR 8004 (HY000) at line 1: Transaction is too large, size: 100000058`というエラーが発生することがあります。このエラーを解決する推奨される方法は、`tidb.toml`ファイルで[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)の値を増やすことです。
> - トランザクションでコミットされる行数に関係なく、`LOAD DATA`は明示的なトランザクションの[`ROLLBACK`](/sql-statements/sql-statement-rollback.md)ステートメントによってロールバックされません。
> - `LOAD DATA`ステートメントは、TiDBのトランザクションモードの構成に関係なく、常に楽観的トランザクションモードで実行されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> - TiDB v4.0.0より前のバージョンでは、`LOAD DATA`は20000行ごとにコミットされ、設定できません。
> - TiDB v4.0.0からv6.6.0までのバージョンでは、デフォルトでTiDBはすべての行を1つのトランザクションでコミットします。ただし、`LOAD DATA`ステートメントを固定数の行ごとにコミットする必要がある場合は、[`tidb_dml_batch_size`](/system-variables.md#tidb_dml_batch_size)を所定の行数に設定できます。
> - v7.0.0からは、`tidb_dml_batch_size`は`LOAD DATA`に影響を与えなくなり、TiDBはすべての行を1つのトランザクションでコミットします。
> - TiDB v4.0.0またはそれ以前のバージョンからアップグレードした場合、`ERROR 8004 (HY000) at line 1: Transaction is too large, size: 100000058`というエラーが発生することがあります。このエラーを解決するためには、[TiDB Cloud Support](https://docs.pingcap.com/tidbcloud/tidb-cloud-support)に連絡して、[`txn-total-size-limit`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#txn-total-size-limit)の値を増やすことができます。
> - トランザクションでコミットされる行数に関係なく、`LOAD DATA`は明示的なトランザクションの[`ROLLBACK`](/sql-statements/sql-statement-rollback.md)ステートメントによってロールバックされません。
> - `LOAD DATA`ステートメントは、TiDBのトランザクションモードの構成に関係なく、常に楽観的トランザクションモードで実行されます。

</CustomContent>

## 関連項目 {#see-also}

<CustomContent platform="tidb">

- [INSERT](/sql-statements/sql-statement-insert.md)
- [TiDB Optimistic Transaction Model](/optimistic-transaction.md)
- [TiDB Pessimistic Transaction Mode](/pessimistic-transaction.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

- [INSERT](/sql-statements/sql-statement-insert.md)
- [TiDB Optimistic Transaction Model](/optimistic-transaction.md)
- [TiDB Pessimistic Transaction Mode](/pessimistic-transaction.md)

</CustomContent>
