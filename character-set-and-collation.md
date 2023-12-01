---
title: Character Set and Collation
summary: Learn about the supported character sets and collations in TiDB.
---

# 文字セットと照合 {#character-set-and-collation}

この文書では、TiDBでサポートされている文字セットと照合について紹介します。

## 概念 {#concepts}

文字セットは、記号とエンコーディングのセットです。TiDBのデフォルトの文字セットはutf8mb4であり、これはMySQL 8.0以降のデフォルトと一致しています。

照合は、文字セット内の文字を比較するためのルールセットであり、文字の並び替え順序です。例えば、バイナリ照合では、`A`と`a`は等しくないと比較されます。

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

TiDBはバイナリ照合をデフォルトで使用します。これは、デフォルトで大文字小文字を区別しない照合を使用するMySQLとは異なります。

## TiDBでサポートされている文字セットと照合 {#character-sets-and-collations-supported-by-tidb}

現在、TiDBは以下の文字セットをサポートしています。

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

TiDBは以下の照合順序をサポートしています。

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

**警告:**

TiDBはlatin1をutf8のサブセットとして誤って扱います。これにより、latin1とutf8のエンコーディングで異なる文字を保存すると予期しない動作が発生する可能性があります。utf8mb4文字セットを強く推奨します。詳細については、[TiDB #18955](https://github.com/pingcap/tidb/issues/18955)を参照してください。

もし述語に文字列のプレフィックスを含む場合、例えば `LIKE 'prefix%'` のような場合、かつ対象の列がバイナリでない照合順序に設定されている場合（接尾辞が `_bin` で終わらない場合）、オプティマイザは現在、この述語を範囲スキャンに変換することができません。代わりに、完全なスキャンを実行します。その結果、このようなSQLクエリは予期しないリソース消費を引き起こす可能性があります。

**注意:**

TiDBのデフォルトの照合順序（接尾辞 `_bin` を持つバイナリ照合順序）は、[MySQLのデフォルトの照合順序](https://dev.mysql.com/doc/refman/8.0/en/charset-charsets.html)（通常、接尾辞 `_general_ci` を持つ一般的な照合順序）とは異なります。これにより、明示的な文字セットを指定しているが暗黙のデフォルト照合順序を選択することに依存している場合、互換性のない動作が発生する可能性があります。

以下のステートメントを使用して、文字セットに対応する照合順序（[照合順序の新しいフレームワーク](#new-framework-for-collations)）を表示できます。

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

MySQLでは、文字セット `utf8` は最大3バイトまでと制限されています。これは、基本多言語面（BMP）の文字を保存するには十分ですが、絵文字などの文字を保存するには不十分です。そのため、文字セット `utf8mb4` を使用することが推奨されています。

デフォルトでは、TiDBも文字セット `utf8` を最大3バイトまでと制限しており、これはTiDBで作成されたデータが安全にMySQLで復元できるようにするためです。システム変数 [`tidb_check_mb4_value_in_utf8`](/system-variables.md#tidb_check_mb4_value_in_utf8) の値を `OFF` に変更することで、この制限を無効にすることができます。

以下は、4バイトの絵文字文字をテーブルに挿入する際のデフォルトの動作を示しています。`utf8`文字セットでは`INSERT`文が失敗しますが、`utf8mb4`では成功します。

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

## 異なるレイヤーでの文字セットと照合 {#character-set-and-collation-in-different-layers}

文字セットと照合は異なるレイヤーで設定することができます。

### データベースの文字セットと照合 {#database-character-set-and-collation}

各データベースには文字セットと照合があります。次の文を使用して、データベースの文字セットと照合を指定できます：

```sql
CREATE DATABASE db_name
    [[DEFAULT] CHARACTER SET charset_name]
    [[DEFAULT] COLLATE collation_name]

ALTER DATABASE db_name
    [[DEFAULT] CHARACTER SET charset_name]
    [[DEFAULT] COLLATE collation_name]
```

`DATABASE`はここで`SCHEMA`に置き換えることができます。

異なるデータベースは異なる文字セットと照合順序を使用することがあります。現在のデータベースの文字セットと照合順序を確認するには、`character_set_database`と`collation_database`を使用してください。

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

`INFORMATION_SCHEMA` でも2つの値を見ることができます。

```sql
SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME
FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = 'db_name';
```

### テーブルの文字セットと照合 {#table-character-set-and-collation}

次の文を使用して、テーブルの文字セットと照合を指定できます：

```sql
CREATE TABLE tbl_name (column_list)
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]]

ALTER TABLE tbl_name
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]
```

例えば：
// translated content start
例えば：
// translated content end

```sql
CREATE TABLE t1(a int) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

```sql
Query OK, 0 rows affected (0.08 sec)
```

If the table character set and collation are not specified, the database character set and collation are used as their default values.

### カラムの文字セットと照合 {#column-character-set-and-collation}

次の文を使用して、カラムの文字セットと照合を指定できます：

```sql
col_name {CHAR | VARCHAR | TEXT} (col_length)
    [CHARACTER SET charset_name]
    [COLLATE collation_name]

col_name {ENUM | SET} (val_list)
    [CHARACTER SET charset_name]
    [COLLATE collation_name]
```

カラムの文字セットと照合が指定されていない場合、テーブルの文字セットと照合がデフォルト値として使用されます。

### 文字列の文字セットと照合 {#string-character-sets-and-collation}

各文字列は文字セットと照合に対応しています。文字列を使用する際には、このオプションが利用可能です。

```sql
[_charset_name]'string' [COLLATE collation_name]
```

Example:
例:

```sql
SELECT 'string';
SELECT _utf8mb4'string';
SELECT _utf8mb4'string' COLLATE utf8mb4_general_ci;
```

規則：

-   ルール1：`CHARACTER SET charset_name` と `COLLATE collation_name` を指定すると、`charset_name` 文字セットと `collation_name` 照合が直接使用されます。
-   ルール2：`CHARACTER SET charset_name` を指定しても `COLLATE collation_name` を指定しない場合、`charset_name` 文字セットと `charset_name` のデフォルト照合が使用されます。
-   ルール3：`CHARACTER SET charset_name` や `COLLATE collation_name` を指定しない場合、システム変数 `character_set_connection` と `collation_connection` で指定された文字セットと照合が使用されます。

### クライアント接続の文字セットと照合 {#client-connection-character-set-and-collation}

-   サーバーの文字セットと照合は、`character_set_server` と `collation_server` システム変数の値です。

-   デフォルトデータベースの文字セットと照合は、`character_set_database` と `collation_database` システム変数の値です。

`character_set_connection` と `collation_connection` を使用して、各接続の文字セットと照合を指定できます。`character_set_client` 変数はクライアントの文字セットを設定するためのものです。

結果を返す前に、`character_set_results` システム変数は、サーバーがクライアントにクエリ結果を返す際の文字セットを示します。これには結果のメタデータも含まれます。

クライアントに関連する文字セットと照合を設定するために、次の文を使用できます：

-   `SET NAMES 'charset_name' [COLLATE 'collation_name']`

    `SET NAMES` は、クライアントがサーバーに対してSQLステートメントを送信する際に使用する文字セットを示します。`SET NAMES utf8mb4` は、クライアントからのすべてのリクエストとサーバーからの結果が utf8mb4 を使用することを示します。

    `SET NAMES 'charset_name'` 文は、次の文の組み合わせと同等です：

    ```sql
    SET character_set_client = charset_name;
    SET character_set_results = charset_name;
    SET character_set_connection = charset_name;
    ```

    `COLLATE` はオプションです。省略されている場合、`charset_name` のデフォルト照合が `collation_connection` に設定されます。

-   `SET CHARACTER SET 'charset_name'`

    `SET NAMES` と同様に、`SET NAMES 'charset_name'` 文は次の文の組み合わせと同等です：

    ```sql
    SET character_set_client = charset_name;
    SET character_set_results = charset_name;
    SET charset_connection = @@charset_database;
    SET collation_connection = @@collation_database;
    ```

## 文字セットと照合の選択優先順位 {#selection-priorities-of-character-sets-and-collations}

文字列 > 列 > テーブル > データベース > サーバー

## 文字セットと照合の選択に関する一般的なルール {#general-rules-on-selecting-character-sets-and-collation}

-   ルール1：`CHARACTER SET charset_name` と `COLLATE collation_name` を指定すると、`charset_name` 文字セットと `collation_name` 照合が直接使用されます。
-   ルール2：`CHARACTER SET charset_name` を指定して `COLLATE collation_name` を指定しない場合、`charset_name` 文字セットと `charset_name` のデフォルト照合が使用されます。
-   ルール3：`CHARACTER SET charset_name` や `COLLATE collation_name` を指定しない場合、より高い最適化レベルの文字セットと照合が使用されます。

## 文字の有効性チェック {#validity-check-of-characters}

指定された文字セットが `utf8` または `utf8mb4` の場合、TiDB は有効な `utf8` 文字のみをサポートします。無効な文字の場合、TiDB は `incorrect utf8 value` エラーを報告します。TiDB での文字の有効性チェックは MySQL 8.0 と互換性がありますが、MySQL 5.7 やそれ以前のバージョンとは互換性がありません。

このエラー報告を無効にするには、`set @@tidb_skip_utf8_check=1;` を使用して文字のチェックをスキップします。

> **注意:**
>
> 文字のチェックをスキップすると、TiDB はアプリケーションによって書かれた不正な UTF-8 文字を検出できず、`ANALYZE` が実行されるとデコードエラーや他の未知のエンコーディングの問題が発生する可能性があります。書かれた文字列の有効性を保証できない場合は、文字のチェックをスキップすることは推奨されません。

## 照合サポートフレームワーク {#collation-support-framework}

<CustomContent platform="tidb">

照合の構文サポートと意味サポートは、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap) 構成項目の影響を受けます。構文サポートと意味サポートは異なります。前者は TiDB が照合を解析して設定できることを示します。後者は TiDB が文字列を比較する際に正しく照合を使用できることを示します。

</CustomContent>

v4.0より前、TiDBは[古い照合フレームワーク](#old-framework-for-collations)のみを提供していました。このフレームワークでは、TiDBは構文解析上はほとんどのMySQL照合をサポートしていますが、意味論的にはすべての照合をバイナリ照合として扱っています。

v4.0以降、TiDBは[新しい照合フレームワーク](#new-framework-for-collations)をサポートしています。このフレームワークでは、TiDBは異なる照合を意味論的に解析し、文字列を比較する際に厳密に照合を遵守します。

### 古い照合フレームワーク {#old-framework-for-collations}

v4.0より前、TiDBではほとんどのMySQL照合を指定することができ、これらの照合はデフォルトの照合に従って処理されます。つまり、バイト順序が文字順序を決定します。MySQLとは異なり、TiDBは文字の末尾の空白を処理しません。これにより以下のような動作の違いが生じます。

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

TiDBでは、前述の文が正常に実行されます。MySQLでは、比較がスペースが埋められた後に行われるため、`Duplicate entry 'a '`エラーが返されます。

### 新しい照合フレームワーク {#new-framework-for-collations}

TiDB v4.0以降、完全な照合フレームワークが導入されました。

<CustomContent platform="tidb">

この新しいフレームワークは、照合を意味的に解析し、クラスタが最初に初期化されるときに新しいフレームワークを有効にするかどうかを決定する`new_collations_enabled_on_first_bootstrap`構成項目を導入します。新しいフレームワークを有効にするには、`new_collations_enabled_on_first_bootstrap`を`true`に設定します。詳細については、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)を参照してください。

既に初期化されたTiDBクラスタの場合、`mysql.tidb`テーブルの`new_collation_enabled`変数を使用して新しい照合が有効かどうかを確認できます：

> **注意:**
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

<CustomContent platform="tidb-cloud">

この新しいフレームワークは、意味論的に解析する照合順序をサポートしています。TiDBは、クラスタが初めて初期化されるときに、新しいフレームワークをデフォルトで有効にします。

新しいフレームワークでは、TiDBは`utf8_general_ci`、`utf8mb4_general_ci`、`utf8_unicode_ci`、`utf8mb4_unicode_ci`、`gbk_chinese_ci`、`gbk_bin`の照合順序をサポートしており、これはMySQLと互換性があります。

`utf8_general_ci`、`utf8mb4_general_ci`、`utf8_unicode_ci`、`utf8mb4_unicode_ci`、`gbk_chinese_ci`のいずれかが使用されると、文字列の比較は大文字とアクセントを無視します。同時に、TiDBは照合順序の`PADDING`の動作も修正します。

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

**注意:**

TiDBにおけるパディングの実装はMySQLと異なります。MySQLでは、パディングはスペースを埋めることで実装されますが、TiDBでは末尾のスペースを切り取ることで実装されます。ほとんどの場合、両方のアプローチは同じです。唯一の例外は、文字列の末尾にスペース（0x20）よりも少ない文字が含まれている場合です。例えば、TiDBでは `'a' < 'a\t'` の結果は `1` ですが、MySQLでは `'a' < 'a\t'` は `'a ' < 'a\t'` と同等であり、結果は `0` です。

## 式中の照合順序の強制度値 {#coercibility-values-of-collations-in-expressions}

式に複数の異なる照合順序の節が含まれる場合、計算で使用される照合順序を推測する必要があります。ルールは次のとおりです:

-   明示的な `COLLATE` 節の強制度値は `0` です。
-   2つの文字列の照合順序が互換性がない場合、異なる照合順序を持つ2つの文字列の連結の強制度値は `1` です。
-   列、`CAST()`、`CONVERT()`、または `BINARY()` の照合順序の強制度値は `2` です。
-   システム定数（`USER()` または `VERSION()` が返す文字列）の強制度値は `3` です。
-   定数の強制度値は `4` です。
-   数値または中間変数の強制度値は `5` です。
-   `NULL` または `NULL` から派生した式の強制度値は `6` です。

TiDBは照合順序を推測する際、強制度値が低い式の照合順序を使用することを好みます。2つの節の強制度値が同じ場合、照合順序は次の優先順位に従って決定されます:

binary > utf8mb4\_bin > (utf8mb4\_general\_ci = utf8mb4\_unicode\_ci) > utf8\_bin > (utf8\_general\_ci = utf8\_unicode\_ci) > latin1\_bin > ascii\_bin

TiDBは次の状況で照合順序を推測できず、エラーを報告します:

-   2つの節の照合順序が異なり、両方の節の強制度値が `0` の場合。
-   2つの節の照合順序が互換性がなく、式の返される型が `String` の場合。

## `COLLATE` 節 {#collate-clause}

TiDBは`COLLATE` 節を使用して式の照合順序を指定することをサポートしています。この式の強制度値は `0` であり、最も優先されます。以下は例です:

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

[接続文字セットと照合](https://dev.mysql.com/doc/refman/5.7/en/charset-connection.html)の詳細については、詳細を参照してください。
