---
title: TiDB Data Type
summary: Learn about the JSON data type in TiDB.
---

# JSON Type {#json-type}

TiDBは、半構造化データを格納するのに便利な`JSON`（JavaScript Object Notation）データ型をサポートしています。`JSON`データ型は、`JSON`形式の文字列を文字列カラムに格納するよりも次の利点を提供します。

- シリアル化にバイナリ形式を使用します。内部形式により、`JSON`ドキュメント要素に素早くアクセスできます。
- `JSON`カラムに格納されたJSONドキュメントの自動検証を行います。有効なドキュメントのみが格納できます。

`JSON`カラムは、他のバイナリ型のカラムと同様に直接インデックス化されませんが、生成されたカラムの形式で`JSON`ドキュメントのフィールドにインデックスを付けることができます。

```sql
CREATE TABLE city (
    id INT PRIMARY KEY,
    detail JSON,
    population INT AS (JSON_EXTRACT(detail, '$.population')),
    index index_name (population)
    );
INSERT INTO city (id,detail) VALUES (1, '{"name": "Beijing", "population": 100}');
SELECT id FROM city WHERE population >= 100;
```

詳細については、[JSON Functions](/functions-and-operators/json-functions.md)および[Generated Columns](/generated-columns.md)を参照してください。

## 制限事項 {#restrictions}

- 現在、TiDBはTiFlashに限られた`JSON`関数のプッシュダウンのみをサポートしています。詳細については、[プッシュダウン式](/tiflash/tiflash-supported-pushdown-calculations.md#push-down-expressions)を参照してください。
- TiDB Backup & Restore（BR）は、v6.3.0でJSONカラムデータのエンコード方法を変更しました。そのため、v6.3.0より前のTiDBクラスタにJSONカラムを含むデータを復元する際にBRを使用することは推奨されません。
- `DATE`、`DATETIME`、`TIME`などの非標準の`JSON`データ型を含むデータをレプリケーションツールでレプリケーションしないでください。

## MySQL互換性 {#mysql-compatibility}

- `BINARY`型のデータでJSONカラムを作成すると、MySQLは現在、データを`STRING`型と誤ってラベル付けしますが、TiDBは正しく`BINARY`型として処理します。

  ```sql
  CREATE TABLE test(a json);
  INSERT INTO test SELECT json_objectagg('a', b'01010101');

  -- TiDBでは、次のSQLステートメントを実行すると`0, 0`が返されます。MySQLでは、次のSQLステートメントを実行すると`0, 1`が返されます。
  mysql> SELECT JSON_EXTRACT(JSON_OBJECT('a', b'01010101'), '$.a') = "base64:type15:VQ==" AS r1, JSON_EXTRACT(a, '$.a') = "base64:type15:VQ==" AS r2 FROM test;
  +------+------+
  | r1   | r2   |
  +------+------+
  |    0 |    0 |
  +------+------+
  1 row in set (0.01 sec)
  ```

  詳細については、issue [#37443](https://github.com/pingcap/tidb/issues/37443)を参照してください。

- `ENUM`または`SET`から`JSON`にデータ型を変換する場合、TiDBはデータ形式の正当性をチェックします。たとえば、TiDBで次のSQLステートメントを実行するとエラーが返されます。

  ```sql
  CREATE TABLE t(e ENUM('a'));
  INSERT INTO t VALUES ('a');
  mysql> SELECT CAST(e AS JSON) FROM t;
  ERROR 3140 (22032): Invalid JSON text: The document root must not be followed by other values.
  ```

  詳細については、issue [#9999](https://github.com/pingcap/tidb/issues/9999)を参照してください。

- TiDBでは、`ORDER BY`を使用してJSON配列やJSONオブジェクトをソートできます。

  MySQLでは、`ORDER BY`を使用してJSON配列やJSONオブジェクトをソートすると、MySQLは警告を返し、ソート結果が比較操作の結果と一致しない場合があります。

  ```sql
  CREATE TABLE t(j JSON);
  INSERT INTO t VALUES ('[1,2,3,4]');
  INSERT INTO t VALUES ('[5]');

  mysql> SELECT j FROM t WHERE j < JSON_ARRAY(5);
  +--------------+
  | j            |
  +--------------+
  | [1, 2, 3, 4] |
  +--------------+
  1 row in set (0.00 sec)

  -- TiDBでは、次のSQLステートメントを実行すると正しいソート結果が返されます。MySQLでは、次のSQLステートメントを実行すると"This version of MySQL doesn't yet support 'sorting of non-scalar JSON values'."という警告が返され、ソート結果が`<`の比較結果と一致しない場合があります。
  mysql> SELECT j FROM t ORDER BY j;
  +--------------+
  | j            |
  +--------------+
  | [1, 2, 3, 4] |
  | [5]          |
  +--------------+
  2 rows in set (0.00 sec)
  ```

  詳細については、issue [#37506](https://github.com/pingcap/tidb/issues/37506)を参照してください。

- JSONカラムにデータを挿入すると、TiDBはデータの値を暗黙的に`JSON`型に変換します。

  ```sql
  CREATE TABLE t(col JSON);

  -- TiDBでは、次のINSERTステートメントは正常に実行されます。MySQLでは、次のINSERTステートメントを実行すると"Invalid JSON text"エラーが返されます。
  INSERT INTO t VALUES (3);
  ```

`JSON`データ型についての詳細については、[JSON functions](/functions-and-operators/json-functions.md)および[Generated Columns](/generated-columns.md)を参照してください。
