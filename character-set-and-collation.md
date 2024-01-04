---
title: Character Set and Collation
summary: Learn about the supported character sets and collations in TiDB.
---

# 文字セットと照合順序 {#character-set-and-collation}

このドキュメントでは、TiDBでサポートされている文字セットと照合順序について紹介します。

## コンセプト {#concepts}

文字セットは、シンボルとエンコーディングのセットです。TiDBのデフォルトの文字セットはutf8mb4で、これはMySQL 8.0以降のデフォルトと一致しています。

照合順序は、文字セット内の文字を比較するためのルールのセットであり、文字の並び替え順序です。例えば、バイナリ照合順序では、`A`と`a`は等しくないと比較されます。**Warning:**

```sql
SET NAMES utf8mb4 COLLATE utf8mb4_bin;
SELECT 'A' = 'a';
SET NAMES utf8mb4 COLLATE utf8mb4_general_ci;
SELECT 'A' = 'a';
```

```sql
SELECT 'A' = 'a';
```

```sql
+-----------+
| 'A' = 'a' |
+-----------+
|         0 |
+-----------+
1 row in set (0.00 sec)
```

```sql
SET NAMES utf8mb4 COLLATE utf8mb4_general_ci;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

```sql
SELECT 'A' = 'a';
```

```sql
+-----------+
| 'A' = 'a' |
+-----------+
|         1 |
+-----------+
1 row in set (0.00 sec)
```

TiDBはバイナリ照合をデフォルトで使用します。これは、デフォルトで大文字と小文字を区別しない照合を使用するMySQLとは異なります。

## TiDBがサポートする文字セットと照合順序 {#character-sets-and-collations-supported-by-tidb}

現在、TiDBは以下の文字セットをサポートしています：

```sql
SHOW CHARACTER SET;
```

```sql
+---------+-------------------------------------+-------------------+--------+
| Charset | Description                         | Default collation | Maxlen |
+---------+-------------------------------------+-------------------+--------+
| ascii   | US ASCII                            | ascii_bin         |      1 |
| binary  | binary                              | binary            |      1 |
| gbk     | Chinese Internal Code Specification | gbk_bin           |      2 |
| latin1  | Latin1                              | latin1_bin        |      1 |
| utf8    | UTF-8 Unicode                       | utf8_bin          |      3 |
| utf8mb4 | UTF-8 Unicode                       | utf8mb4_bin       |      4 |
+---------+-------------------------------------+-------------------+--------+
6 rows in set (0.00 sec)
```

TiDBは以下の照合順序をサポートしています：

```sql
SHOW COLLATION;
```

```sql
+--------------------+---------+------+---------+----------+---------+
| Collation          | Charset | Id   | Default | Compiled | Sortlen |
+--------------------+---------+------+---------+----------+---------+
| ascii_bin          | ascii   |   65 | Yes     | Yes      |       1 |
| binary             | binary  |   63 | Yes     | Yes      |       1 |
| gbk_bin            | gbk     |   87 |         | Yes      |       1 |
| gbk_chinese_ci     | gbk     |   28 | Yes     | Yes      |       1 |
| latin1_bin         | latin1  |   47 | Yes     | Yes      |       1 |
| utf8_bin           | utf8    |   83 | Yes     | Yes      |       1 |
| utf8_general_ci    | utf8    |   33 |         | Yes      |       1 |
| utf8_unicode_ci    | utf8    |  192 |         | Yes      |       1 |
| utf8mb4_bin        | utf8mb4 |   46 | Yes     | Yes      |       1 |
| utf8mb4_general_ci | utf8mb4 |   45 |         | Yes      |       1 |
| utf8mb4_unicode_ci | utf8mb4 |  224 |         | Yes      |       1 |
+--------------------+---------+------+---------+----------+---------+
11 rows in set (0.00 sec)
```

> **Warning:**

TiDBは、latin1をutf8のサブセットと誤って扱います。これにより、latin1とutf8のエンコーディングで異なる文字を保存すると予期しない動作が発生する可能性があります。utf8mb4文字セットを強く推奨します。詳細については、[TiDB #18955](https://github.com/pingcap/tidb/issues/18955)を参照してください。

述語に`LIKE`が含まれる場合（例：`LIKE 'prefix%'`）、かつ対象のカラムがバイナリでない照合順序に設定されている場合（接尾辞が`_bin`で終わらない場合）、オプティマイザは現在、この述語を範囲スキャンに変換することができません。代わりに、フルスキャンを実行します。その結果、このようなSQLクエリは予期しないリソース消費につながる可能性があります。

> **Note:**

TiDBのデフォルトの照合順序（接尾辞が`_bin`のバイナリ照合順序）は、[MySQLのデフォルトの照合順序](https://dev.mysql.com/doc/refman/8.0/en/charset-charsets.html)（通常、接尾辞が`_general_ci`の一般的な照合順序）とは異なります。これにより、明示的な文字セットを指定しても、暗黙のデフォルト照合順序が選択されることに依存すると、互換性のない動作が発生する可能性があります。

以下のステートメントを使用して、文字セットに対応する（#new-framework-for-collationsの下の[new framework for collations](#new-framework-for-collations)）照合順序を表示できます。

```sql
SHOW COLLATION WHERE Charset = 'utf8mb4';
```

```sql
+--------------------+---------+------+---------+----------+---------+
| Collation          | Charset | Id   | Default | Compiled | Sortlen |
+--------------------+---------+------+---------+----------+---------+
| utf8mb4_bin        | utf8mb4 |   46 | Yes     | Yes      |       1 |
| utf8mb4_general_ci | utf8mb4 |   45 |         | Yes      |       1 |
| utf8mb4_unicode_ci | utf8mb4 |  224 |         | Yes      |       1 |
+--------------------+---------+------+---------+----------+---------+
3 rows in set (0.00 sec)
```

TiDBのGBK文字セットのサポートの詳細については、[GBK](/character-set-gbk.md)を参照してください。

## TiDBにおける `utf8` と `utf8mb4` {#utf8-and-utf8mb4-in-tidb}

MySQLでは、文字セット `utf8` は最大3バイトに制限されています。これは、基本多言語面（BMP）の文字を保存するのに十分ですが、絵文字などの文字を保存するには十分ではありません。そのため、代わりに文字セット `utf8mb4` を使用することが推奨されています。

デフォルトでは、TiDBも文字セット `utf8` を最大3バイトに制限しており、TiDBで作成されたデータを安全にMySQLで復元できるようにしています。システム変数[`tidb_check_mb4_value_in_utf8`](/system-variables.md#tidb_check_mb4_value_in_utf8)の値を`OFF`に変更することで無効にすることができます。

以下は、4バイトの絵文字文字をテーブルに挿入する際のデフォルトの動作を示しています。`INSERT`ステートメントは、`utf8`文字セットでは失敗しますが、`utf8mb4`では成功します。

```sql
CREATE TABLE utf8_test (
     c char(1) NOT NULL
    ) CHARACTER SET utf8;
```

```sql
Query OK, 0 rows affected (0.09 sec)
```

```sql
CREATE TABLE utf8m4_test (
     c char(1) NOT NULL
    ) CHARACTER SET utf8mb4;
```

```sql
Query OK, 0 rows affected (0.09 sec)
```

```sql
INSERT INTO utf8_test VALUES ('😉');
```

```sql
ERROR 1366 (HY000): incorrect utf8 value f09f9889(😉) for column c
```

```sql
INSERT INTO utf8m4_test VALUES ('😉');
```

```sql
Query OK, 1 row affected (0.02 sec)
```

```sql
SELECT char_length(c), length(c), c FROM utf8_test;
```

```sql
Empty set (0.01 sec)
```

```sql
SELECT char_length(c), length(c), c FROM utf8m4_test;
```

```sql
+----------------+-----------+------+
| char_length(c) | length(c) | c    |
+----------------+-----------+------+
|              1 |         4 | 😉     |
+----------------+-----------+------+
1 row in set (0.00 sec)
```

## 異なるレイヤーでの文字セットと照合順序 {#character-set-and-collation-in-different-layers}

文字セットと照合順序は異なるレイヤーで設定できます。

### データベースの文字セットと照合順序 {#database-character-set-and-collation}

各データベースには文字セットと照合順序があります。次のステートメントを使用して、データベースの文字セットと照合順序を指定できます。

```sql
CREATE DATABASE db_name
    [[DEFAULT] CHARACTER SET charset_name]
    [[DEFAULT] COLLATE collation_name]

ALTER DATABASE db_name
    [[DEFAULT] CHARACTER SET charset_name]
    [[DEFAULT] COLLATE collation_name]
```

```json
"`DATABASE` can be replaced with `SCHEMA` here.

異なるデータベースは異なる文字セットと照合順序を使用することができます。現在のデータベースの文字セットと照合順序を確認するには、`character_set_database` と `collation_database` を使用してください:"
```

```sql
CREATE SCHEMA test1 CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

```sql
Query OK, 0 rows affected (0.09 sec)
```

```sql
USE test1;
```

```sql
Database changed
```

```sql
SELECT @@character_set_database, @@collation_database;
```

```sql
+--------------------------|----------------------+
| @@character_set_database | @@collation_database |
+--------------------------|----------------------+
| utf8mb4                  | utf8mb4_general_ci   |
+--------------------------|----------------------+
1 row in set (0.00 sec)
```

```sql
CREATE SCHEMA test2 CHARACTER SET latin1 COLLATE latin1_bin;
```

```sql
Query OK, 0 rows affected (0.09 sec)
```

```sql
USE test2;
```

```sql
Database changed
```

```sql
SELECT @@character_set_database, @@collation_database;
```

```sql
+--------------------------|----------------------+
| @@character_set_database | @@collation_database |
+--------------------------|----------------------+
| latin1                   | latin1_bin           |
+--------------------------|----------------------+
1 row in set (0.00 sec)
```

あなたは`INFORMATION_SCHEMA`で2つの値も見ることができます。

```sql
SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME
FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = 'db_name';
```

### テーブルの文字セットと照合順序 {#table-character-set-and-collation}

次のステートメントを使用して、テーブルの文字セットと照合順序を指定できます：

```sql
CREATE TABLE tbl_name (column_list)
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]]

ALTER TABLE tbl_name
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]
```

For example:

```sql
CREATE TABLE t1(a int) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

```sql
Query OK, 0 rows affected (0.08 sec)
```

If the table character set and 照合順序 are not specified, the database character set and 照合順序 are used as their default values.

### カラム character set and 照合順序 {#column-character-set-and-collation}

You can use the following statement to specify the character set and 照合順序 for columns:

```sql
col_name {CHAR | VARCHAR | TEXT} (col_length)
    [CHARACTER SET charset_name]
    [COLLATE collation_name]

col_name {ENUM | SET} (val_list)
    [CHARACTER SET charset_name]
    [COLLATE collation_name]
```

If the カラム character set and 照合順序 are not specified, the table character set and 照合順序 are used as their default values.

### String character sets and 照合順序 {#string-character-sets-and-collation}

Each string corresponds to a character set and a 照合順序. When you use a string, this option is available:

```sql
[_charset_name]'string' [COLLATE collation_name]
```

{"input\_content": "例:"}

```sql
SELECT 'string';
SELECT _utf8mb4'string';
SELECT _utf8mb4'string' COLLATE utf8mb4_general_ci;
```

Rules:

- Rule 1: If you specify `CHARACTER SET charset_name` and `COLLATE collation_name`, then the `charset_name` character set and the `collation_name` collation are used directly.
- Rule 2: If you specify `CHARACTER SET charset_name` but do not specify `COLLATE collation_name`, the `charset_name` character set and the default collation of `charset_name` are used.
- Rule 3: If you specify neither `CHARACTER SET charset_name` nor `COLLATE collation_name`, the character set and collation given by the system variables `character_set_connection` and `collation_connection` are used.

### Client connection character set and collation {#client-connection-character-set-and-collation}

- The server character set and collation are the values of the `character_set_server` and `collation_server` system variables.

- The character set and collation of the default database are the values of the `character_set_database` and `collation_database` system variables.

You can use `character_set_connection` and `collation_connection` to specify the character set and collation for each connection. The `character_set_client` variable is to set the client character set.

Before returning the result, the `character_set_results` system variable indicates the character set in which the server returns query results to the client, including the metadata of the result.

You can use the following statement to set the character set and collation that is related to the client:

- `SET NAMES 'charset_name' [COLLATE 'collation_name']`

  `SET NAMES` indicates what character set the client will use to send SQL statements to the server. `SET NAMES utf8mb4` indicates that all the requests from the client use utf8mb4, as well as the results from the server.

  The `SET NAMES 'charset_name'` statement is equivalent to the following statement combination:

  ```sql
  SET character_set_client = charset_name;
  SET character_set_results = charset_name;
  SET character_set_connection = charset_name;
  ```

  `COLLATE` is optional, if absent, the default collation of the `charset_name` is used to set the `collation_connection`.

- `SET CHARACTER SET 'charset_name'`

  Similar to `SET NAMES`, the `SET NAMES 'charset_name'` statement is equivalent to the following statement combination:

  ```sql
  SET character_set_client = charset_name;
  SET character_set_results = charset_name;
  SET charset_connection = @@charset_database;
  SET collation_connection = @@collation_database;
  ```

## Selection priorities of character sets and collations {#selection-priorities-of-character-sets-and-collations}

String > Column > Table > Database > Server

## General rules on selecting character sets and collation {#general-rules-on-selecting-character-sets-and-collation}

- Rule 1: If you specify `CHARACTER SET charset_name` and `COLLATE collation_name`, then the `charset_name` character set and the `collation_name` collation are used directly.
- Rule 2: If you specify `CHARACTER SET charset_name` and do not specify `COLLATE collation_name`, then the `charset_name` character set and the default collation of `charset_name` are used.
- Rule 3: If you specify neither `CHARACTER SET charset_name` nor `COLLATE collation_name`, the character set and collation with higher optimization levels are used.

## Validity check of characters {#validity-check-of-characters}

If the specified character set is `utf8` or `utf8mb4`, TiDB only supports the valid `utf8` characters. For invalid characters, TiDB reports the `incorrect utf8 value` error. This validity check of characters in TiDB is compatible with MySQL 8.0 but incompatible with MySQL 5.7 or earlier versions.

To disable this error reporting, use `set @@tidb_skip_utf8_check=1;` to skip the character check.

> **Note:**
>
> If the character check is skipped, TiDB might fail to detect illegal UTF-8 characters written by the application, cause decoding errors when `ANALYZE` is executed, and introduce other unknown encoding issues. If your application cannot guarantee the validity of the written string, it is not recommended to skip the character check.

## 照合順序サポートフレームワーク {#collation-support-framework}

<CustomContent platform="tidb">

照合の構文サポートおよび意味サポートは、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap) 構成項目に影響を受けます。構文サポートと意味サポートは異なります。前者は、TiDBが照合を解析および設定できることを示します。後者は、TiDBが文字列を比較する際に照合を正しく使用できることを示します。

</CustomContent>

v4.0より前、TiDBは[照合の古いフレームワーク](#old-framework-for-collations)のみを提供します。このフレームワークでは、TiDBは構文的にはMySQLのほとんどの照合を解析できますが、意味的にはすべての照合をバイナリ照合として扱います。

v4.0以降、TiDBは[照合の新しいフレームワーク](#new-framework-for-collations)をサポートしています。このフレームワークでは、TiDBは異なる照合を意味的に解析し、文字列を比較する際に厳密に照合に従います。

### 照合の古いフレームワーク {#old-framework-for-collations}

v4.0より前、TiDBではMySQLのほとんどの照合を指定でき、これらの照合はデフォルトの照合に従って処理されます。つまり、バイト順が文字順を決定します。MySQLとは異なり、TiDBは文字の末尾のスペースを処理しないため、次のような動作の違いが発生します：

```sql
CREATE TABLE t(a varchar(20) charset utf8mb4 collate utf8mb4_general_ci PRIMARY KEY);
```

```sql
Query OK, 0 rows affected
```

```sql
INSERT INTO t VALUES ('A');
```

```sql
Query OK, 1 row affected
```

```sql
INSERT INTO t VALUES ('a');
```

```sql
Query OK, 1 row affected
```

TiDBでは、前述の文が正常に実行されます。MySQLでは、`utf8mb4_general_ci`が大文字と小文字を区別しないため、`Duplicate entry 'a'`エラーが報告されます。

```sql
INSERT INTO t1 VALUES ('a ');
```

```sql
Query OK, 1 row affected
```

TiDBでは、前述のステートメントが正常に実行されます。MySQLでは、比較はスペースが埋められた後に行われるため、`Duplicate entry 'a '`エラーが返されます。

### 照合順序の新しいフレームワーク {#new-framework-for-collations}

TiDB v4.0以降、照合順序の完全なフレームワークが導入されました。

<CustomContent platform="tidb">

この新しいフレームワークは、照合順序を意味的に解析し、クラスターが最初に初期化されるときに新しいフレームワークを有効にするかどうかを決定する`new_collations_enabled_on_first_bootstrap`構成項目を導入します。新しいフレームワークを有効にするには、`new_collations_enabled_on_first_bootstrap`を`true`に設定します。詳細については、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)を参照してください。

既に初期化されたTiDBクラスターでは、`mysql.tidb`テーブルの`new_collation_enabled`変数を使用して新しい照合順序が有効かどうかを確認できます。

> **Note:**
>
> `mysql.tidb`テーブルのクエリ結果が`new_collations_enabled_on_first_bootstrap`の値と異なる場合、`mysql.tidb`テーブルの結果が実際の値です。

```sql
SELECT VARIABLE_VALUE FROM mysql.tidb WHERE VARIABLE_NAME='new_collation_enabled';
```

```sql
+----------------+
| VARIABLE_VALUE |
+----------------+
| True           |
+----------------+
1 row in set (0.00 sec)
```

</CustomContent>

<CustomContent platform="tidb-cloud">

この新しいフレームワークは、意味論的に照合順序を解析することをサポートしています。TiDBは、クラスターが最初に初期化されるときに、新しいフレームワークをデフォルトで有効にします。

新しいフレームワークでは、TiDBは`utf8_general_ci`、`utf8mb4_general_ci`、`utf8_unicode_ci`、`utf8mb4_unicode_ci`、`gbk_chinese_ci`、および`gbk_bin`の照合順序をサポートしており、これはMySQLと互換性があります。

`utf8_general_ci`、`utf8mb4_general_ci`、`utf8_unicode_ci`、`utf8mb4_unicode_ci`、および`gbk_chinese_ci`のいずれかが使用されると、文字列の比較は大文字とアクセントを無視します。同時に、TiDBは照合順序の`PADDING`の動作も修正します。

```sql
CREATE TABLE t(a varchar(20) charset utf8mb4 collate utf8mb4_general_ci PRIMARY KEY);
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

```sql
INSERT INTO t VALUES ('A');
```

```sql
Query OK, 1 row affected (0.00 sec)
```

```sql
INSERT INTO t VALUES ('a');
```

```sql
ERROR 1062 (23000): Duplicate entry 'a' for key 't.PRIMARY' # TiDB is compatible with the case-insensitive collation of MySQL.
```

```sql
INSERT INTO t VALUES ('a ');
```

```sql
ERROR 1062 (23000): Duplicate entry 'a ' for key 't.PRIMARY' # TiDB modifies the `PADDING` behavior to be compatible with MySQL.
```

> **Note:**
>
> TiDBのパディングの実装はMySQLと異なります。MySQLでは、パディングはスペースを埋めることで実装されます。一方、TiDBでは、パディングは末尾のスペースを切り取ることで実装されます。ほとんどの場合、両方のアプローチは同じです。唯一の例外は、文字列の末尾にスペース（0x20）よりも少ない文字が含まれている場合です。例えば、TiDBでは、`'a' < 'a\t'`の結果は`1`ですが、MySQLでは、`'a' < 'a\t'`は`'a ' < 'a\t'`と同等であり、結果は`0`です。

## 式での照合の強制値 {#coercibility-values-of-collations-in-expressions}

式に複数の異なる照合の節が関与する場合、計算で使用される照合を推測する必要があります。ルールは次のとおりです。

- 明示的な`COLLATE`節の強制値は`0`です。
- 2つの文字列の照合が互換性がない場合、異なる照合を持つ2つの文字列の連結の強制値は`1`です。
- カラム、`CAST()`、`CONVERT()`、または`BINARY()`の照合の強制値は`2`です。
- システム定数（`USER()`または`VERSION()`によって返される文字列）の照合の強制値は`3`です。
- 定数の強制値は`4`です。
- 数値または中間変数の強制値は`5`です。
- `NULL`または`NULL`から派生した式の強制値は`6`です。

照合を推測する際、TiDBは強制値が低い式の照合を優先します。2つの節の強制値が同じ場合、照合は次の優先順位に従って決定されます。

binary > utf8mb4\_bin > (utf8mb4\_general\_ci = utf8mb4\_unicode\_ci) > utf8\_bin > (utf8\_general\_ci = utf8\_unicode\_ci) > latin1\_bin > ascii\_bin

TiDBは次の状況で照合を推測できず、エラーを報告します。

- 2つの節の照合が異なり、両方の節の強制値が`0`である場合。
- 2つの節の照合が互換性がなく、式の返される型が`String`である場合。

## `COLLATE`節 {#collate-clause}

TiDBは、式の照合を指定するために`COLLATE`節を使用することをサポートしています。この式の強制値は`0`で、最も優先されます。次の例を参照してください。

```sql
SELECT 'a' = _utf8mb4 'A' collate utf8mb4_general_ci;
```

```sql
+-----------------------------------------------+
| 'a' = _utf8mb4 'A' collate utf8mb4_general_ci |
+-----------------------------------------------+
|                                             1 |
+-----------------------------------------------+
1 row in set (0.00 sec)
```

For more details, see [Connection Character Sets and 照合順序](https://dev.mysql.com/doc/refman/5.7/en/charset-connection.html).
