---
title: Character Set and Collation
summary: Learn about the supported character sets and collations in TiDB.
---

# 文字セットと照合順序 {#character-set-and-collation}

このドキュメントでは、TiDBでサポートされている文字セットと照合順序について紹介します。

## 概念 {#concepts}

文字セットとは、記号と符号化方式の集合です。TiDBのデフォルトの文字セットはutf8mb4で、MySQL 8.0以降のデフォルトと一致します。

照合順序とは、文字セット内の文字を比較するためのルールと、文字の並び順を指定するものです。例えば、バイナリ照合順序では`A`と`a`は等しくないと判断されます。

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

TiDBはバイナリ照合順序を使用することがデフォルトです。これはMySQLと異なり、デフォルトでは大文字小文字を区別しない照合順序を使用します。

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

TiDBは以下の照合順序をサポートしています:

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

> **警告：**
>
> TiDBは、latin1をutf8のサブセットとして誤って扱います。これにより、latin1とutf8のエンコーディングで異なる文字を格納すると予期しない動作が発生する可能性があります。utf8mb4文字セットを使用することを強くお勧めします。詳細については、[TiDB #18955](https://github.com/pingcap/tidb/issues/18955)を参照してください。
>
> プレディケートに文字列のプレフィックスを含む場合（`LIKE 'prefix%'`など）、ターゲットカラムがバイナリでない照合順序（接尾辞が`_bin`で終わらない）に設定されている場合、最適化プログラムは現在このプレディケートを範囲スキャンに変換できません。その代わりに、フルスキャンを実行します。その結果、このようなSQLクエリは予期しないリソース消費を引き起こす可能性があります。

> **Note:**
>
> TiDBのデフォルトの照合順序（バイナリ照合順序、接尾辞が`_bin`）は、[MySQLのデフォルトの照合順序](https://dev.mysql.com/doc/refman/8.0/en/charset-charsets.html)（一般的な照合順序、接尾辞が`_general_ci`）とは異なります。これにより、明示的な文字セットを指定しても、暗黙のデフォルトの照合順序が選択されることに依存する場合に、互換性のない動作が発生する可能性があります。

以下のステートメントを使用して、文字セットに対応する（[照合順序の新しいフレームワーク](#new-framework-for-collations)の下で）照合順序を表示できます。

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

TiDBがGBK文字セットをサポートする詳細については、[GBK](/character-set-gbk.md)を参照してください。

## TiDBにおける`utf8`と`utf8mb4` {#utf8-and-utf8mb4-in-tidb}

MySQLでは、文字セットの`utf8`は最大3バイトに制限されています。これは、基本多言語面（BMP）の文字を格納するのに十分ですが、絵文字などの文字を格納するには不十分です。そのため、代わりに文字セットの`utf8mb4`を使用することが推奨されます。

デフォルトでは、TiDBも文字セットの`utf8`を最大3バイトに制限しています。これにより、TiDBで作成されたデータをMySQLで安全に復元できるようになります。システム変数[`tidb_check_mb4_value_in_utf8`](/system-variables.md#tidb_check_mb4_value_in_utf8)の値を`OFF`に変更することで、これを無効にすることができます。

次の例は、4バイトの絵文字文字をテーブルに挿入するときのデフォルトの動作を示しています。`utf8`文字セットでは`INSERT`文が失敗しますが、`utf8mb4`では成功します。

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

文字セットと照合順序は異なるレイヤーで設定することができます。

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

`DATABASE`はここで`SCHEMA`に置き換えることができます。

異なるデータベースは異なる文字セットと照合順序を使用することができます。現在のデータベースの文字セットと照合順序を確認するには、`character_set_database`と`collation_database`を使用してください。

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

`INFORMATION_SCHEMA`で2つの値を見ることもできます：

```sql
SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME
FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = 'db_name';
```

### テーブルの文字セットと照合順序 {#table-character-set-and-collation}

テーブルの文字セットと照合順序を指定するには、次の文を使用できます。

```sql
CREATE TABLE tbl_name (column_list)
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]]

ALTER TABLE tbl_name
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]
```

例えば：

```sql
CREATE TABLE t1(a int) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

```sql
Query OK, 0 rows affected (0.08 sec)
```

もしテーブルの文字セットと照合順序が指定されていない場合、データベースの文字セットと照合順序がデフォルト値として使用されます。

### カラムの文字セットと照合順序 {#column-character-set-and-collation}

以下のステートメントを使用して、カラムの文字セットと照合順序を指定することができます。

```sql
col_name {CHAR | VARCHAR | TEXT} (col_length)
    [CHARACTER SET charset_name]
    [COLLATE collation_name]

col_name {ENUM | SET} (val_list)
    [CHARACTER SET charset_name]
    [COLLATE collation_name]
```

カラムの文字セットと照合順序が指定されていない場合、テーブルの文字セットと照合順序がデフォルト値として使用されます。

### 文字列の文字セットと照合順序 {#string-character-sets-and-collation}

各文字列は文字セットと照合順序に対応します。文字列を使用する場合、このオプションが利用可能です：

```sql
[_charset_name]'string' [COLLATE collation_name]
```

例：

// 例：

例：

ここには翻訳しないK-Vマップがあります。コンテンツのキーを翻訳せず、値で置き換える必要があります：
{"collation":"照合順序","I":"I","Y":"Y","**Note:**":"**Note:**","layer":"レイヤー","Column":"カラム","TEXT":"TEXT","server":"サーバー","MySQL 5.7":"MySQL 5.7"}

// 入力コンテンツの開始

例：

// 入力コンテンツの終了

```sql
SELECT 'string';
SELECT _utf8mb4'string';
SELECT _utf8mb4'string' COLLATE utf8mb4_general_ci;
```

ルール：

- ルール1：`CHARACTER SET charset_name`と`COLLATE collation_name`を指定した場合、`charset_name`の文字セットと`collation_name`の照合順序が直接使用されます。
- ルール2：`CHARACTER SET charset_name`を指定したが、`COLLATE collation_name`を指定しなかった場合、`charset_name`の文字セットと`charset_name`のデフォルト照合順序が使用されます。
- ルール3：`CHARACTER SET charset_name`と`COLLATE collation_name`のどちらも指定しなかった場合、システム変数`character_set_connection`と`collation_connection`で指定された文字セットと照合順序が使用されます。

### クライアント接続の文字セットと照合順序 {#client-connection-character-set-and-collation}

- サーバーの文字セットと照合順序は、`character_set_server`と`collation_server`のシステム変数の値です。

- デフォルトデータベースの文字セットと照合順序は、`character_set_database`と`collation_database`のシステム変数の値です。

各接続に対して文字セットと照合順序を指定するには、`character_set_connection`と`collation_connection`を使用できます。`character_set_client`変数はクライアントの文字セットを設定するためのものです。

結果を返す前に、`character_set_results`システム変数は、サーバーがクライアントにクエリ結果を返す際の文字セットを示します。これには、結果のメタデータも含まれます。

クライアントに関連する文字セットと照合順序を設定するには、次の文を使用できます。

- `SET NAMES 'charset_name' [COLLATE 'collation_name']`

  `SET NAMES`は、クライアントがサーバーにSQL文を送信する際に使用する文字セットを示します。`SET NAMES utf8mb4`は、クライアントからのすべてのリクエストがutf8mb4を使用し、サーバーからの結果も同様になることを示します。

  `SET NAMES 'charset_name'`文は、次の文の組み合わせと同等です。

  ```sql
  SET character_set_client = charset_name;
  SET character_set_results = charset_name;
  SET character_set_connection = charset_name;
  ```

  `COLLATE`はオプションです。省略された場合、`charset_name`のデフォルト照合順序が`collation_connection`に設定されます。

- `SET CHARACTER SET 'charset_name'`

  `SET NAMES`と同様に、`SET NAMES 'charset_name'`文は次の文の組み合わせと同等です。

  ```sql
  SET character_set_client = charset_name;
  SET character_set_results = charset_name;
  SET charset_connection = @@charset_database;
  SET collation_connection = @@collation_database;
  ```

## 文字セットと照合順序の選択優先順位 {#selection-priorities-of-character-sets-and-collations}

文字列 > カラム > テーブル > データベース > サーバー

## 文字セットと照合順序の選択に関する一般的なルール {#general-rules-on-selecting-character-sets-and-collation}

- ルール1：`CHARACTER SET charset_name`と`COLLATE collation_name`を指定した場合、`charset_name`の文字セットと`collation_name`の照合順序が直接使用されます。
- ルール2：`CHARACTER SET charset_name`を指定したが、`COLLATE collation_name`を指定しなかった場合、`charset_name`の文字セットと`charset_name`のデフォルト照合順序が使用されます。
- ルール3：`CHARACTER SET charset_name`と`COLLATE collation_name`のどちらも指定しなかった場合、最適化レベルが高い文字セットと照合順序が使用されます。

## 文字の妥当性チェック {#validity-check-of-characters}

指定された文字セットが`utf8`または`utf8mb4`の場合、TiDBは有効な`utf8`文字のみをサポートします。無効な文字の場合、TiDBは`incorrect utf8 value`エラーを報告します。TiDBの文字の妥当性チェックはMySQL 8.0と互換性がありますが、MySQL 5.7以前とは互換性がありません。

このエラー報告を無効にするには、`set @@tidb_skip_utf8_check=1;`を使用して文字のチェックをスキップします。

> **Note:**
>
> 文字のチェックをスキップすると、TiDBはアプリケーションによって書き込まれた不正なUTF-8文字を検出できなくなり、`ANALYZE`が実行された際にデコードエラーが発生し、その他の未知のエンコーディング問題が発生する可能性があります。アプリケーションが書き込まれた文字列の妥当性を保証できない場合、文字のチェックをスキップすることは推奨されません。

## 照合順序のサポートフレームワーク {#collation-support-framework}

<CustomContent platform="tidb">

照合順序の構文サポートと意味サポートは、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)構成項目の影響を受けます。構文サポートと意味サポートは異なります。前者は、TiDBが照合順序を解析して設定できることを示します。後者は、TiDBが文字列を比較する際に照合順序を正しく使用できることを示します。

</CustomContent>

v4.0以前、TiDBは[照合順序の古いフレームワーク](#old-framework-for-collations)のみを提供していました。このフレームワークでは、TiDBは構文的にはMySQLのほとんどの照合順序を解析できますが、意味的にはすべての照合順序をバイナリ照合順序として扱います。

v4.0以降、TiDBは[照合順序の新しいフレームワーク](#new-framework-for-collations)をサポートしています。このフレームワークでは、TiDBは異なる照合順序を意味的に解析し、文字列を比較する際に厳密に照合順序に従います。

### 照合順序の古いフレームワーク {#old-framework-for-collations}

v4.0以前、TiDBではMySQLのほとんどの照合順序を指定することができ、これらの照合順序はデフォルトの照合順序に従って処理されます。つまり、バイトの順序が文字の順序を決定します。MySQLとは異なり、TiDBは文字の末尾のスペースを処理しないため、次のような動作の違いが生じます：

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

TiDBでは、前のステートメントが正常に実行されます。MySQLでは、`utf8mb4_general_ci`が大文字と小文字を区別しないため、`Duplicate entry 'a'`エラーが報告されます。

```sql
INSERT INTO t1 VALUES ('a ');
```

```sql
Query OK, 1 row affected
```

TiDBでは、前述のステートメントが正常に実行されます。MySQLでは、スペースが埋められた後に比較が行われるため、`Duplicate entry 'a '`エラーが返されます。

### 照合順序の新しいフレームワーク {#new-framework-for-collations}

TiDB v4.0以降、照合順序の完全なフレームワークが導入されました。

<CustomContent platform="tidb">

この新しいフレームワークでは、照合順序を意味的に解析することができ、クラスターを初期化する際に新しいフレームワークを有効にするかどうかを決定するための`new_collations_enabled_on_first_bootstrap`構成項目が導入されます。新しいフレームワークを有効にするには、`new_collations_enabled_on_first_bootstrap`を`true`に設定します。詳細については、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)を参照してください。

既に初期化されたTiDBクラスターでは、`mysql.tidb`テーブルの`new_collation_enabled`変数を使用して、新しい照合順序が有効になっているかどうかを確認できます：

> **Note:**
>
> `mysql.tidb`テーブルのクエリ結果が`new_collations_enabled_on_first_bootstrap`の値と異なる場合、`mysql.tidb`テーブルの結果が実際の値になります。

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

この新しいフレームワークでは、照合順序を意味的に解析することができます。TiDBは、クラスターが初期化されると、この新しいフレームワークをデフォルトで有効にします。

新しいフレームワークでは、TiDBはMySQLと互換性のある`utf8_general_ci`、`utf8mb4_general_ci`、`utf8_unicode_ci`、`utf8mb4_unicode_ci`、`gbk_chinese_ci`、`gbk_bin`の照合順序をサポートしています。

`utf8_general_ci`、`utf8mb4_general_ci`、`utf8_unicode_ci`、`utf8mb4_unicode_ci`、`gbk_chinese_ci`のいずれかが使用される場合、文字列の比較は大文字とアクセントを無視します。同時に、TiDBは照合順序の`PADDING`の振る舞いも修正します。

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
> TiDBにおけるパディングの実装はMySQLと異なります。MySQLでは、スペースを埋めることでパディングが実装されますが、TiDBでは末尾のスペースを切り取ることでパディングが実装されます。ほとんどの場合、この2つのアプローチは同じです。唯一の例外は、文字列の末尾にスペースよりも小さい文字が含まれる場合です（0x20）。例えば、TiDBでは`'a' < 'a\t'`の結果は`1`になりますが、MySQLでは`'a' < 'a\t'`は`'a ' < 'a\t'`と同等であり、結果は`0`になります。

## 式における照合順序の強制値 {#coercibility-values-of-collations-in-expressions}

式に複数の異なる照合順序の句が含まれる場合、計算に使用される照合順序を推測する必要があります。ルールは以下の通りです：

- 明示的な`COLLATE`句の強制値は`0`です。
- 2つの文字列の照合順序が互換性がない場合、異なる照合順序を持つ2つの文字列の連結の強制値は`1`です。
- カラム、`CAST()`、`CONVERT()`、または`BINARY()`の照合順序の強制値は`2`です。
- システム定数（`USER()`または`VERSION()`で返される文字列）の照合順序の強制値は`3`です。
- 定数の強制値は`4`です。
- 数値または中間変数の強制値は`5`です。
- `NULL`または`NULL`から派生した式の強制値は`6`です。

照合順序を推測する際、TiDBは強制値が低い式の照合順序を優先的に使用します。2つの句の強制値が同じ場合、照合順序は以下の優先順位に従って決定されます：

binary > utf8mb4\_bin > (utf8mb4\_general\_ci = utf8mb4\_unicode\_ci) > utf8\_bin > (utf8\_general\_ci = utf8\_unicode\_ci) > latin1\_bin > ascii\_bin

TiDBは以下の状況で照合順序を推測できず、エラーを報告します：

- 2つの句の照合順序が異なり、両方の強制値が`0`の場合。
- 2つの句の照合順序が互換性がなく、式の返される型が`String`の場合。

## `COLLATE`句 {#collate-clause}

TiDBでは、式の照合順序を指定するために`COLLATE`句を使用することができます。この式の強制値は`0`であり、最も優先されます。以下の例を参照してください：

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

詳細については、[接続文字セットと照合順序](https://dev.mysql.com/doc/refman/5.7/en/charset-connection.html)を参照してください。
