---
title: LOAD DATA | TiDB SQL Statement Reference
summary: An overview of the usage of LOAD DATA for the TiDB database.
---

# データのロード {#load-data}

`LOAD DATA` ステートメントは、TiDB テーブルにデータをバッチでロードします。

TiDB v7.0.0 から、`LOAD DATA` SQL ステートメントは次の機能をサポートしています。

- S3 および GCS からのデータのインポートのサポート
- 新しいパラメータ `FIELDS DEFINED NULL BY` の追加

> **Warning:**
>
> 新しいパラメータ `FIELDS DEFINED NULL BY` および S3 および GCS からのデータのインポートのサポートは実験的です。本番環境で使用することは推奨されません。この機能は事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHub で [issue](https://github.com/pingcap/tidb/issues) を報告できます。

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

`LOCAL`を使用して、クライアント上のデータファイルを指定してインポートできます。ファイルパラメータはクライアント上のファイルシステムパスである必要があります。

TiDB Cloudを使用している場合、`LOAD DATA`ステートメントを使用してローカルデータファイルをロードするには、TiDB Cloudに接続する際に接続文字列に`--local-infile`オプションを追加する必要があります。

- TiDB Serverlessの接続文字列の例は次のとおりです：

  ```
  mysql --connect-timeout 15 -u '<user_name>' -h <host_name> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p<your_password> --local-infile
  ```

- TiDB Dedicatedの接続文字列の例は次のとおりです：

  ```
  mysql --connect-timeout 15 --ssl-mode=VERIFY_IDENTITY --ssl-ca=<CA_path> --tls-version="TLSv1.2" -u root -h <host_name> -P 4000 -D test -p<your_password> --local-infile
  ```

### S3およびGCSストレージ {#s3-and-gcs-storage}

<CustomContent platform="tidb">

`LOCAL`を指定しない場合、ファイルパラメータは[外部ストレージ](/br/backup-and-restore-storages.md)で詳細に説明されているように、有効なS3またはGCSパスである必要があります。

</CustomContent>

<CustomContent platform="tidb-cloud">

`LOCAL`を指定しない場合、ファイルパラメータは[外部ストレージ](https://docs.pingcap.com/tidb/stable/backup-and-restore-storages)で詳細に説明されているように、有効なS3またはGCSパスである必要があります。

</CustomContent>

データファイルがS3またはGCSに保存されている場合、個々のファイルをインポートするか、ワイルドカード文字`*`を使用して複数のファイルを一致させることができます。ワイルドカードはサブディレクトリ内のファイルを再帰的に処理しないことに注意してください。次にいくつかの例を示します：

- 単一ファイルのインポート：`s3://<bucket-name>/path/to/data/foo.csv`
- 指定されたパス内のすべてのファイルのインポート：`s3://<bucket-name>/path/to/data/*`
- 指定されたパスで`.csv`で終わるすべてのファイルのインポート：`s3://<bucket-name>/path/to/data/*.csv`
- 指定されたパスで`foo`で始まるすべてのファイルのインポート：`s3://<bucket-name>/path/to/data/foo*`
- 指定されたパスで`foo`で始まり`.csv`で終わるすべてのファイルのインポート：`s3://<bucket-name>/path/to/data/foo*.csv`

### `Fields`、`Lines`、および`Ignore Lines` {#fields-lines-and-ignore-lines}

`Fields`および`Lines`パラメータを使用して、データ形式の処理方法を指定できます。

- `FIELDS TERMINATED BY`：データの区切り文字を指定します。
- `FIELDS ENCLOSED BY`：データの囲み文字を指定します。
- `LINES TERMINATED BY`：行の終端文字を指定します。特定の文字で行を終了させたい場合に使用します。

`DEFINED NULL BY`を使用して、データファイル内のNULL値の表現方法を指定できます。

- MySQLの動作に従い、`ESCAPED BY`がnullでない場合、たとえばデフォルト値`\`が使用されている場合、`\N`はNULL値と見なされます。
- `DEFINED NULL BY`を使用する場合、`DEFINED NULL BY 'my-null'`のように、`my-null`はNULL値と見なされます。
- `DEFINED NULL BY ... OPTIONALLY ENCLOSED`を使用する場合、たとえば`DEFINED NULL BY 'my-null' OPTIONALLY ENCLOSED`のように、`my-null`および`"my-null"`（`ENCLOSED BY '"'`が想定される場合）はNULL値と見なされます。
- `DEFINED NULL BY`または`DEFINED NULL BY ... OPTIONALLY ENCLOSED`を使用せず、`ENCLOSED BY`を使用する場合、たとえば`ENCLOSED BY '"'`の場合、`NULL`はNULL値と見なされます。この動作はMySQLと一致しています。
- それ以外の場合、NULL値とは見なされません。

次のデータ形式を例に取り上げます：

```
"bob","20","street 1"\r\n
"alice","33","street 1"\r\n
```

もし`bob`、`20`、および`street 1`を抽出したい場合は、フィールドの区切り文字を`','`、囲む文字を`'\"'`として指定してください。

```sql
FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n'
```

もし、前述のパラメータを指定しない場合、インポートされたデータはデフォルトで以下のように処理されます:

```sql
FIELDS TERMINATED BY '\t' ENCLOSED BY '' ESCAPED BY '\\'
LINES TERMINATED BY '\n' STARTING BY ''
```

ファイルの最初の `number` 行を `IGNORE <number> LINES` パラメータを設定して無視できます。 たとえば、`IGNORE 1 LINES` を設定すると、ファイルの最初の行が無視されます。

## Examples {#examples}

次の例では、`LOAD DATA` を使用してデータをインポートします。 コンマがフィールドの区切り文字として指定されています。 データを囲む二重引用符は無視されます。 ファイルの最初の行が無視されます。

<CustomContent platform="tidb">

`ERROR 1148 (42000): the used command is not allowed with this TiDB version` が表示される場合は、トラブルシューティングのために [ERROR 1148 (42000): the used command is not allowed with this TiDB version](/error-codes.md#mysql-native-error-messages) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

`ERROR 1148 (42000): the used command is not allowed with this TiDB version` が表示される場合は、トラブルシューティングのために [ERROR 1148 (42000): the used command is not allowed with this TiDB version](https://docs.pingcap.com/tidb/stable/error-codes#mysql-native-error-messages) を参照してください。

</CustomContent>

```sql
LOAD DATA LOCAL INFILE '/mnt/evo970/data-sets/bikeshare-data/2017Q4-capitalbikeshare-tripdata.csv' INTO TABLE trips FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n' IGNORE 1 LINES (duration, start_date, end_date, start_station_number, start_station, end_station_number, end_station, bike_number, member_type);
```

```sql
Query OK, 815264 rows affected (39.63 sec)
Records: 815264  Deleted: 0  Skipped: 0  Warnings: 0
```

`LOAD DATA`は、`FIELDS ENCLOSED BY`および`FIELDS TERMINATED BY`のパラメータとして、16進ASCII文字表現やバイナリASCII文字表現を使用することもサポートしています。次の例を参照してください:

```sql
LOAD DATA LOCAL INFILE '/mnt/evo970/data-sets/bikeshare-data/2017Q4-capitalbikeshare-tripdata.csv' INTO TABLE trips FIELDS TERMINATED BY x'2c' ENCLOSED BY b'100010' LINES TERMINATED BY '\r\n' IGNORE 1 LINES (duration, start_date, end_date, start_station_number, start_station, end_station_number, end_station, bike_number, member_type);
```

上記の例では、`x'2c'`は`,`文字の16進表現であり、`b'100010'`は`"`文字のバイナリ表現です。

## MySQL互換性 {#mysql-compatibility}

`LOAD DATA`ステートメントの構文は、文字セットオプションを除いてMySQLと互換性がありますが、これらは解析されますが無視されます。構文の互換性の違いがある場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)することができます。

<CustomContent platform="tidb">

> **Note:**
>
> - TiDB v4.0.0より前のバージョンでは、`LOAD DATA`は20000行ごとにコミットされ、設定できません。
> - TiDB v4.0.0からv6.6.0までのバージョンでは、デフォルトでTiDBはすべての行を1つのトランザクションでコミットします。ただし、`LOAD DATA`ステートメントを固定数の行ごとにコミットする必要がある場合は、[`tidb_dml_batch_size`](/system-variables.md#tidb_dml_batch_size)を所定の行数に設定できます。
> - TiDB v7.0.0からは、`tidb_dml_batch_size`は`LOAD DATA`に影響を与えなくなり、TiDBはすべての行を1つのトランザクションでコミットします。
> - TiDB v4.0.0またはそれ以前のバージョンからアップグレードした場合、`ERROR 8004 (HY000) at line 1: Transaction is too large, size: 100000058`というエラーが発生することがあります。このエラーを解決する推奨される方法は、`tidb.toml`ファイルで[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)の値を増やすことです。
> - トランザクションでコミットされる行数に関係なく、`LOAD DATA`は明示的なトランザクションの[`ROLLBACK`](/sql-statements/sql-statement-rollback.md)ステートメントによってロールバックされません。
> - `LOAD DATA`ステートメントは、TiDBのトランザクションモードの構成に関係なく、常に楽観的トランザクションモードで実行されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
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
