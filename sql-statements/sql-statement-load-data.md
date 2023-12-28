---
title: LOAD DATA | TiDB SQL Statement Reference
summary: An overview of the usage of LOAD DATA for the TiDB database.
---

# データのロード {#load-data}

`LOAD DATA`ステートメントは、TiDBテーブルにデータをバッチでロードします。

TiDB v7.0.0から、`LOAD DATA` SQLステートメントは以下の機能をサポートしています：

- S3やGCSからのデータのインポートをサポート
- 新しいパラメーター`FIELDS DEFINED NULL BY`を追加

> **警告：**
>
> 新しいパラメーター`FIELDS DEFINED NULL BY`とS3やGCSからのデータのインポートのサポートは実験的なものです。本番環境では使用しないことをお勧めします。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を報告できます。

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

## パラメーター {#parameters}

### `LOCAL` {#local}

`LOCAL`を使用すると、クライアント上のデータファイルを指定してインポートできます。ファイルパラメーターは、クライアント上のファイルシステムパスである必要があります。

TiDB Cloudを使用している場合、ローカルのデータファイルをロードするには、TiDB Cloudに接続する際に接続文字列に`--local-infile`オプションを追加する必要があります。

- TiDB Serverlessの接続文字列の例は次のとおりです。

      mysql --connect-timeout 15 -u '<user_name>' -h <host_name> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p<your_password> --local-infile

- TiDB Dedicatedの接続文字列の例は次のとおりです。

      mysql --connect-timeout 15 --ssl-mode=VERIFY_IDENTITY --ssl-ca=<CA_path> --tls-version="TLSv1.2" -u root -h <host_name> -P 4000 -D test -p<your_password> --local-infile

### S3およびGCSストレージ {#s3-and-gcs-storage}

<CustomContent platform="tidb">

`LOCAL`を指定しない場合、ファイルパラメーターは有効なS3またはGCSパスである必要があります。詳細は、[外部ストレージ](/br/backup-and-restore-storages.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

`LOCAL`を指定しない場合、ファイルパラメーターは有効なS3またはGCSパスである必要があります。詳細は、[外部ストレージ](https://docs.pingcap.com/tidb/stable/backup-and-restore-storages)を参照してください。

</CustomContent>

データファイルがS3またはGCSに保存されている場合、個々のファイルをインポートするか、ワイルドカード文字`*`を使用して複数のファイルを一致させてインポートすることができます。ワイルドカードはサブディレクトリ内のファイルを再帰的に処理しません。次にいくつかの例を示します。

- 単一のファイルをインポートする：`s3://<bucket-name>/path/to/data/foo.csv`
- 指定されたパスのすべてのファイルをインポートする：`s3://<bucket-name>/path/to/data/*`
- 指定されたパスで終わるすべてのファイルをインポートする：`s3://<bucket-name>/path/to/data/*.csv`
- 指定されたパスで始まるすべてのファイルをインポートする：`s3://<bucket-name>/path/to/data/foo*`
- 指定されたパスで始まり、`.csv`で終わるすべてのファイルをインポートする：`s3://<bucket-name>/path/to/data/foo*.csv`

### `Fields`、`Lines`、および`Ignore Lines` {#fields-lines-and-ignore-lines}

`Fields`および`Lines`パラメーターを使用して、データ形式の処理方法を指定できます。

- `FIELDS TERMINATED BY`：データの区切り文字を指定します。
- `FIELDS ENCLOSED BY`：データの囲み文字を指定します。
- `LINES TERMINATED BY`：行の終端文字を指定します。特定の文字で行を終了する場合は、このパラメーターを使用します。

`DEFINED NULL BY`を使用して、データファイル内でNULL値がどのように表されるかを指定できます。

- MySQLの動作に従い、`ESCAPED BY`がnullでない場合、たとえばデフォルト値`\`が使用されている場合、`\N`はNULL値と見なされます。
- `DEFINED NULL BY`を使用する場合、たとえば`DEFINED NULL BY 'my-null'`のように使用する場合、`my-null`はNULL値と見なされます。
- `DEFINED NULL BY ... OPTIONALLY ENCLOSED`を使用する場合、たとえば`DEFINED NULL BY 'my-null' OPTIONALLY ENCLOSED`のように使用する場合、`my-null`および`"my-null"`（`ENCLOSED BY '"'`を想定）はNULL値と見なされます。
- `DEFINED NULL BY`または`DEFINED NULL BY ... OPTIONALLY ENCLOSED`を使用せず、`ENCLOSED BY`を使用する場合、たとえば`ENCLOSED BY '"'`のように使用する場合、`NULL`はNULL値と見なされます。この動作はMySQLと一致します。
- その他の場合、NULL値と見なされません。

次のデータ形式を例に説明します。

    "bob","20","street 1"\r\n
    "alice","33","street 1"\r\n

`bob`, `20`, `street 1`を抽出したい場合は、フィールドデリミタを`','`、囲み文字を`'\"'`として指定してください：

```sql
FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n'
```

前述のパラメータを指定しない場合、インポートされたデータはデフォルトで以下のように処理されます:

```sql
FIELDS TERMINATED BY '\t' ENCLOSED BY '' ESCAPED BY '\\'
LINES TERMINATED BY '\n' STARTING BY ''
```

ファイルの最初の `number` 行は、`IGNORE <number> LINES` パラメータを設定することで無視できます。例えば、`IGNORE 1 LINES` を設定すると、ファイルの最初の行は無視されます。

## 例 {#examples}

次の例では、`LOAD DATA` を使用してデータをインポートします。フィールドの区切り文字としてコンマが指定されています。データを囲む二重引用符は無視されます。ファイルの最初の行は無視されます。

<CustomContent platform="tidb">

`ERROR 1148 (42000): the used command is not allowed with this TiDB version` が表示される場合は、[ERROR 1148 (42000): the used command is not allowed with this TiDB version](/error-codes.md#mysql-native-error-messages) を参照してトラブルシューティングを行ってください。

</CustomContent>

<CustomContent platform="tidb-cloud">

`ERROR 1148 (42000): the used command is not allowed with this TiDB version` が表示される場合は、[ERROR 1148 (42000): the used command is not allowed with this TiDB version](https://docs.pingcap.com/tidb/stable/error-codes#mysql-native-error-messages) を参照してトラブルシューティングを行ってください。

</CustomContent>

```sql
LOAD DATA LOCAL INFILE '/mnt/evo970/data-sets/bikeshare-data/2017Q4-capitalbikeshare-tripdata.csv' INTO TABLE trips FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n' IGNORE 1 LINES (duration, start_date, end_date, start_station_number, start_station, end_station_number, end_station, bike_number, member_type);
```

```sql
Query OK, 815264 rows affected (39.63 sec)
Records: 815264  Deleted: 0  Skipped: 0  Warnings: 0
```

`LOAD DATA`は、`FIELDS ENCLOSED BY`および`FIELDS TERMINATED BY`のパラメータとして、16進数のASCII文字表現やバイナリのASCII文字表現を使用することもサポートしています。以下の例を参照してください。

```sql
LOAD DATA LOCAL INFILE '/mnt/evo970/data-sets/bikeshare-data/2017Q4-capitalbikeshare-tripdata.csv' INTO TABLE trips FIELDS TERMINATED BY x'2c' ENCLOSED BY b'100010' LINES TERMINATED BY '\r\n' IGNORE 1 LINES (duration, start_date, end_date, start_station_number, start_station, end_station_number, end_station, bike_number, member_type);
```

上記の例では、`x'2c'`は`,`文字の16進表現であり、`b'100010'`は`"`文字の2進表現です。

## MySQL互換性 {#mysql-compatibility}

`LOAD DATA`ステートメントの構文はMySQLと互換性がありますが、文字セットオプションは解析されますが無視されます。構文の互換性の違いが見つかった場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)することができます。

<CustomContent platform="tidb">

> **Note:**
>
> - TiDB v4.0.0より前のバージョンでは、`LOAD DATA`は20000行ごとにコミットされ、設定することはできません。
> - TiDB v4.0.0からv6.6.0までのバージョンでは、TiDBはデフォルトですべての行を1つのトランザクションでコミットします。ただし、`LOAD DATA`ステートメントを固定数の行ごとにコミットする必要がある場合は、[`tidb_dml_batch_size`](/system-variables.md#tidb_dml_batch_size)を設定することができます。
> - TiDB v7.0.0からは、`tidb_dml_batch_size`は`LOAD DATA`に影響を与えなくなり、TiDBはすべての行を1つのトランザクションでコミットします。
> - TiDB v4.0.0またはそれ以前のバージョンからアップグレードした場合、`ERROR 8004 (HY000) at line 1: Transaction is too large, size: 100000058`が発生する可能性があります。このエラーを解決する推奨方法は、`tidb.toml`ファイルで[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)の値を増やすことです。
> - トランザクションでコミットされる行数に関係なく、`LOAD DATA`は明示的なトランザクション内の[`ROLLBACK`](/sql-statements/sql-statement-rollback.md)ステートメントによってロールバックされません。
> - `LOAD DATA`ステートメントは、TiDBのトランザクションモードの設定に関係なく、常に楽観的トランザクションモードで実行されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> - TiDB v4.0.0より前のバージョンでは、`LOAD DATA`は20000行ごとにコミットされ、設定することはできません。
> - TiDB v4.0.0からv6.6.0までのバージョンでは、TiDBはデフォルトですべての行を1つのトランザクションでコミットします。ただし、`LOAD DATA`ステートメントを固定数の行ごとにコミットする必要がある場合は、[`tidb_dml_batch_size`](/system-variables.md#tidb_dml_batch_size)を設定することができます。
> - TiDB v7.0.0からは、`tidb_dml_batch_size`は`LOAD DATA`に影響を与えなくなり、TiDBはすべての行を1つのトランザクションでコミットします。
> - TiDB v4.0.0またはそれ以前のバージョンからアップグレードした場合、`ERROR 8004 (HY000) at line 1: Transaction is too large, size: 100000058`が発生する可能性があります。このエラーを解決するためには、[TiDB Cloudサポート](https://docs.pingcap.com/tidbcloud/tidb-cloud-support)に連絡して、[`txn-total-size-limit`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#txn-total-size-limit)の値を増やすことができます。
> - トランザクションでコミットされる行数に関係なく、`LOAD DATA`は明示的なトランザクション内の[`ROLLBACK`](/sql-statements/sql-statement-rollback.md)ステートメントによってロールバックされません。
> - `LOAD DATA`ステートメントは、TiDBのトランザクションモードの設定に関係なく、常に楽観的トランザクションモードで実行されます。

</CustomContent>

## 関連情報 {#see-also}

<CustomContent platform="tidb">

- [INSERT](/sql-statements/sql-statement-insert.md)
- [TiDBの楽観的トランザクションモデル](/optimistic-transaction.md)
- [TiDBの悲観的トランザクションモード](/pessimistic-transaction.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

- [INSERT](/sql-statements/sql-statement-insert.md)
- [TiDBの楽観的トランザクションモデル](/optimistic-transaction.md)
- [TiDBの悲観的トランザクションモード](/pessimistic-transaction.md)

</CustomContent>
