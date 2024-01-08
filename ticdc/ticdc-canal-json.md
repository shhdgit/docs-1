---
title: TiCDC Canal-JSON Protocol
summary: Learn the concept of TiCDC Canal-JSON Protocol and how to use it.
---

# TiCDC Canal-JSON Protocol {#ticdc-canal-json-protocol}

TiCDC Canal-JSONは、[Alibaba Canal](https://github.com/alibaba/canal)によって定義されたデータ交換形式のプロトコルです。このドキュメントでは、TiCDCでのCanal-JSONデータ形式の実装方法、TiDB拡張フィールド、Canal-JSONデータ形式の定義、公式Canalとの比較などについて学ぶことができます。

## Canal-JSONの使用 {#use-canal-json}

下流のSinkとしてメッセージキュー（MQ）を使用する場合、`sink-uri`でCanal-JSONを指定できます。TiCDCは、Eventを基本単位としてCanal-JSONメッセージをラップおよび構築し、TiDBデータ変更のEventを下流に送信します。

以下の3種類のEventがあります：

- DDL Event：DDL変更レコードを表します。上流のDDLステートメントが正常に実行された後に送信されます。DDL Eventは、インデックスが0であるMQ Partitionに送信されます。
- DML Event：行データ変更レコードを表します。このタイプのEventは、行の変更が発生したときに送信されます。変更後の行に関する情報が含まれています。
- WATERMARK Event：特別な時点を表します。この時点より前に受信したEventsが完了したことを示します。これはTiDB拡張フィールドにのみ適用され、`sink-uri`で`enable-tidb-extension`を`true`に設定した場合に効果があります。

以下は`Canal-JSON`の使用例です：

```shell
cdc cli changefeed create --server=http://127.0.0.1:8300 --changefeed-id="kafka-canal-json" --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&protocol=canal-json"
```

## TiDB拡張フィールド {#tidb-extension-field}

Canal-JSONプロトコルは元々MySQL向けに設計されています。これには、CommitTSトランザクションのTiDB固有の一意の識別子など、重要なフィールドが含まれていません。この問題を解決するために、TiCDCはCanal-JSONプロトコル形式にTiDB拡張フィールドを追加します。`sink-uri`で`enable-tidb-extension`を`true`（デフォルトでは`false`）に設定した後、TiCDCはCanal-JSONメッセージを生成する際に以下のように動作します。

- TiCDCは、`_tidb`というフィールドを含むDMLイベントおよびDDLイベントメッセージを送信します。
- TiCDCはWATERMARKイベントメッセージを送信します。

以下は例です：

```shell
cdc cli changefeed create --server=http://127.0.0.1:8300 --changefeed-id="kafka-canal-json-enable-tidb-extension" --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&protocol=canal-json&enable-tidb-extension=true"
```

## メッセージフォーマットの定義 {#definitions-of-message-formats}

このセクションでは、DDLイベント、DMLイベント、およびWATERMARKイベントのフォーマット、およびデータがコンシューマーサイドで解決される方法について説明します。

### DDLイベント {#ddl-event}

TiCDCは、DDLイベントを次のCanal-JSON形式にエンコードします。

```json
{
    "id": 0,
    "database": "test",
    "table": "",
    "pkNames": null,
    "isDdl": true,
    "type": "QUERY",
    "es": 1639633094670,
    "ts": 1639633095489,
    "sql": "drop database if exists test",
    "sqlType": null,
    "mysqlType": null,
    "data": null,
    "old": null,
    "_tidb": {     // TiDB extension field
        "commitTs": 163963309467037594
    }
}
```

次のようにフィールドが説明されています。

| フィールド     | タイプ    | 説明                                                                                               |
| :-------- | :----- | :----------------------------------------------------------------------------------------------- |
| id        | 数値     | TiCDCではデフォルト値は0です。                                                                               |
| database  | 文字列    | 行があるデータベースの名前                                                                                    |
| table     | 文字列    | 行があるテーブルの名前                                                                                      |
| pkNames   | 配列     | 主キーを構成するすべての列の名前                                                                                 |
| isDdl     | ブール    | メッセージがDDLイベントかどうか                                                                                |
| type      | 文字列    | Canal-JSONで定義されたイベントの種類                                                                          |
| es        | 数値     | メッセージが生成されたイベントの13ビット（ミリ秒）タイムスタンプ                                                                |
| ts        | 数値     | TiCDCがメッセージを生成した13ビット（ミリ秒）タイムスタンプ                                                                |
| sql       | 文字列    | isDdlが`true`の場合、対応するDDLステートメントを記録します                                                             |
| sqlType   | オブジェクト | isDdlが`false`の場合、各列のデータ型がJavaでどのように表現されるかを記録します                                                  |
| mysqlType | オブジェクト | isDdlが`false`の場合、各列のデータ型がMySQLでどのように表現されるかを記録します                                                 |
| data      | オブジェクト | isDdlが`false`の場合、各列の名前とデータ値を記録します                                                                |
| old       | オブジェクト | メッセージが更新イベントによって生成された場合のみ、更新前の各列の名前とデータ値を記録します                                                   |
| \_tidb    | オブジェクト | TiDB拡張フィールド。`enable-tidb-extension`を`true`に設定した場合のみ存在します。`commitTs`の値は行の変更を引き起こしたトランザクションのTSOです。 |

### DMLイベント {#dml-event}

TiCDCは、DMLデータ変更イベントの行を次のようにエンコードします：

```json
{
    "id": 0,
    "database": "test",
    "table": "tp_int",
    "pkNames": [
        "id"
    ],
    "isDdl": false,
    "type": "INSERT",
    "es": 1639633141221,
    "ts": 1639633142960,
    "sql": "",
    "sqlType": {
        "c_bigint": -5,
        "c_int": 4,
        "c_mediumint": 4,
        "c_smallint": 5,
        "c_tinyint": -6,
        "id": 4
    },
    "mysqlType": {
        "c_bigint": "bigint",
        "c_int": "int",
        "c_mediumint": "mediumint",
        "c_smallint": "smallint",
        "c_tinyint": "tinyint",
        "id": "int"
    },
    "data": [
        {
            "c_bigint": "9223372036854775807",
            "c_int": "2147483647",
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "127",
            "id": "2"
        }
    ],
    "old": null,
    "_tidb": {     // TiDB extension field
        "commitTs": 163963314122145239
    }
}
```

### TiCDC Canal Event {#watermark-event}

TiCDC sends a TiCDC Canal Event only when you set `enable-tidb-extension` to `true`. The value of the `type` field is `TIDB_WATERMARK`. The Event contains the `_tidb` field, and the field contains only one parameter `watermarkTs`. The value of `watermarkTs` is the TSO recorded when the Event is sent.

When you receive an Event of this type, all Events with `commitTs` less than `watermarkTs` have been sent. Because TiCDC provides the "At Least Once" semantics, data might be sent repeatedly. If a subsequent Event with `commitTs` less than `watermarkTs` is received, you can safely ignore this Event.

The following is an example of the TiCDC Canal Event.

```json
{
    "id": 0,
    "database": "",
    "table": "",
    "pkNames": null,
    "isDdl": false,
    "type": "TIDB_WATERMARK",
    "es": 1640007049196,
    "ts": 1640007050284,
    "sql": "",
    "sqlType": null,
    "mysqlType": null,
    "data": null,
    "old": null,
    "_tidb": {     // TiDB extension field
        "watermarkTs": 429918007904436226
    }
}
```

### コンシューマーサイドでのデータ解決 {#data-resolution-on-the-consumer-side}

上記の例からわかるように、Canal-JSONには統一されたデータ形式があり、異なるイベントタイプに対して異なるフィールドの埋め込みルールがあります。このJSON形式のデータを解決するための統一された方法を使用し、その後、フィールドの値をチェックしてイベントタイプを決定できます。

- `isDdl` が `true` の場合、メッセージにはDDLイベントが含まれます。
- `isDdl` が `false` の場合、`type` フィールドをさらにチェックする必要があります。`type` が `TIDB_WATERMARK` の場合、WATERMARKイベントです。それ以外の場合、DMLイベントです。

## フィールドの説明 {#field-descriptions}

Canal-JSON形式では、`mysqlType` フィールドと `sqlType` フィールドに対応するデータ型が記録されています。

### MySQLタイプフィールド {#mysql-type-field}

`mysqlType` フィールドでは、Canal-JSON形式が各列のMySQLタイプの文字列を記録しています。詳細については、[TiDBデータ型](/data-type-overview.md)を参照してください。

### SQLタイプフィールド {#sql-type-field}

`sqlType` フィールドでは、Canal-JSON形式が各列のJava SQLタイプを記録しており、これはJDBCのデータに対応するデータ型です。その値はMySQLタイプと特定のデータ値によって計算できます。マッピングは次のとおりです。

| MySQLタイプ   | Java SQLタイプコード |
| :--------- | :------------- |
| Boolean    | -6             |
| Float      | 7              |
| Double     | 8              |
| Decimal    | 3              |
| Char       | 1              |
| Varchar    | 12             |
| Binary     | 2004           |
| Varbinary  | 2004           |
| Tinytext   | 2005           |
| Text       | 2005           |
| Mediumtext | 2005           |
| Longtext   | 2005           |
| Tinyblob   | 2004           |
| Blob       | 2004           |
| Mediumblob | 2004           |
| Longblob   | 2004           |
| Date       | 91             |
| Datetime   | 93             |
| Timestamp  | 93             |
| Time       | 92             |
| Year       | 12             |
| Enum       | 4              |
| Set        | -7             |
| Bit        | -7             |
| JSON       | 12             |

## 整数型 {#integer-types}

[整数型](/data-type-numeric.md#integer-types)には、`Unsigned` 制約と値のサイズを考慮する必要があります。これには、それぞれ異なるJava SQLタイプコードが対応しており、次の表に示すようになっています。

| MySQLタイプ文字列        | 値の範囲                                         | Java SQLタイプコード |
| :----------------- | :------------------------------------------- | :------------- |
| tinyint            | \[-128, 127]                                 | -6             |
| tinyint unsigned   | \[0, 127]                                    | -6             |
| tinyint unsigned   | \[128, 255]                                  | 5              |
| smallint           | \[-32768, 32767]                             | 5              |
| smallint unsigned  | \[0, 32767]                                  | 5              |
| smallint unsigned  | \[32768, 65535]                              | 4              |
| mediumint          | \[-8388608, 8388607]                         | 4              |
| mediumint unsigned | \[0, 8388607]                                | 4              |
| mediumint unsigned | \[8388608, 16777215]                         | 4              |
| int                | \[-2147483648, 2147483647]                   | 4              |
| int unsigned       | \[0, 2147483647]                             | 4              |
| int unsigned       | \[2147483648, 4294967295]                    | -5             |
| bigint             | \[-9223372036854775808, 9223372036854775807] | -5             |
| bigint unsigned    | \[0, 9223372036854775807]                    | -5             |
| bigint unsigned    | \[9223372036854775808, 18446744073709551615] | 3              |

次の表は、TiCDCのJava SQLタイプとそのコードのマッピング関係を示しています。

| Java SQLタイプ | Java SQLタイプコード |
| :---------- | :------------- |
| CHAR        | 1              |
| DECIMAL     | 3              |
| INTEGER     | 4              |
| SMALLINT    | 5              |
| REAL        | 7              |
| DOUBLE      | 8              |
| VARCHAR     | 12             |
| DATE        | 91             |
| TIME        | 92             |
| TIMESTAMP   | 93             |
| BLOB        | 2004           |
| CLOB        | 2005           |
| BIGINT      | -5             |
| TINYINT     | -6             |
| Bit         | -7             |

Java SQLタイプに関する詳細については、[Java SQLクラスタイプ](https://docs.oracle.com/javase/8/docs/api/java/sql/Types.html)を参照してください。

## TiCDC Canal-JSONと公式Canalの比較 {#comparison-of-ticdc-canal-json-and-the-official-canal}

TiCDCがCanal-JSONデータ形式を実装する方法は、`Update`イベントと`mysqlType`フィールドを含め、公式Canalと異なります。次の表は、主な違いを示しています。

| 項目               | TiCDC Canal-JSON                                                                                            | Canal                               |
| :--------------- | :---------------------------------------------------------------------------------------------------------- | :---------------------------------- |
| `Update`タイプのイベント | デフォルトでは、`old`フィールドにすべての列データが含まれます。`only_output_updated_columns` が `true` の場合、`old`フィールドには変更された列データのみが含まれます。 | `old`フィールドには変更された列データのみが含まれます       |
| `mysqlType`フィールド | パラメータを持つタイプの場合、タイプパラメータの情報が含まれません                                                                           | パラメータを持つタイプの場合、タイプパラメータの完全な情報が含まれます |

### 公式Canalとの互換性 {#compatibility-with-the-official-canal}

v6.5.6およびv7.1.3から、TiCDC Canal-JSONは公式Canalのデータ形式との互換性をサポートしています。changefeedを作成する際に、`sink-uri` の中で `content-compatible=true` を設定すると、この機能を有効にできます。このモードでは、TiCDCは公式Canalと互換性のあるCanal-JSON形式のデータを出力します。具体的な変更点は次のとおりです。

- 各タイプの`mysqlType`フィールドには、タイプパラメータの完全な情報が含まれます。
- `Update`タイプのイベントは、変更された列のデータのみを出力します。

### `Update`タイプのイベント {#event-of-update-type}

`Update`タイプのイベントについて：

- TiCDCでは、`old`フィールドにすべての列データが含まれます
- 公式Canalでは、`old`フィールドには変更された列データのみが含まれます

以下のSQLステートメントが上流のTiDBで順次実行されたと仮定します：

```sql
create table tp_int
(
    id          int auto_increment,
    c_tinyint   tinyint   null,
    c_smallint  smallint  null,
    c_mediumint mediumint null,
    c_int       int       null,
    c_bigint    bigint    null,
    constraint pk
        primary key (id)
);

insert into tp_int(c_tinyint, c_smallint, c_mediumint, c_int, c_bigint)
values (127, 32767, 8388607, 2147483647, 9223372036854775807);

update tp_int set c_int = 0, c_tinyint = 0 where c_smallint = 32767;
```

`update` ステートメントの場合、TiCDCは以下に示すように `type` を `UPDATE` として含むイベントメッセージを出力します。 `update` ステートメントは `c_int` と `c_tinyint` 列のみを変更します。 出力イベントメッセージの `old` フィールドにはすべての列データが含まれています。

```json
{
    "id": 0,
    ...
    "type": "UPDATE",
    ...
    "sqlType": {
        ...
    },
    "mysqlType": {
        ...
    },
    "data": [
        {
            "c_bigint": "9223372036854775807",
            "c_int": "0",
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "0",
            "id": "2"
        }
    ],
    "old": [                              // In TiCDC, this field contains all the column data.
        {
            "c_bigint": "9223372036854775807",
            "c_int": "2147483647",        // Modified column
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "127",           // Modified column
            "id": "2"
        }
    ]
}
```

公式のTiCDC Canalでは、出力イベントメッセージの`old`フィールドには、以下に示すように変更された列データのみが含まれています。

```json
{
    "id": 0,
    ...
    "type": "UPDATE",
    ...
    "sqlType": {
        ...
    },
    "mysqlType": {
        ...
    },
    "data": [
        {
            "c_bigint": "9223372036854775807",
            "c_int": "0",
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "0",
            "id": "2"
        }
    ],
    "old": [                              // In Canal, this field contains only the modified column data.
        {
            "c_int": "2147483647",        // Modified column
            "c_tinyint": "127",           // Modified column
        }
    ]
}
```

### `mysqlType` フィールド {#mysqltype-field}

`mysqlType` フィールドについて、型にパラメータが含まれている場合、公式の Canal には型パラメータの完全な情報が含まれています。TiCDC にはそのような情報が含まれていません。

次の例では、テーブル定義の SQL ステートメントには、`decimal`、`char`、`varchar`、`enum` などの各列にパラメータが含まれています。TiCDC と公式の Canal によって生成された Canal-JSON フォーマットを比較することで、TiCDC には`mysqlType` フィールドに基本的な MySQL 情報のみが含まれていることがわかります。型パラメータの完全な情報が必要な場合は、他の手段で実装する必要があります。

以下の SQL ステートメントが上流の TiDB で順次実行されると仮定します：

```sql
create table t (
    id     int auto_increment,
    c_decimal    decimal(10, 4) null,
    c_char       char(16)      null,
    c_varchar    varchar(16)   null,
    c_binary     binary(16)    null,
    c_varbinary  varbinary(16) null,
    c_enum enum('a','b','c') null,
    c_set  set('a','b','c')  null,
    c_bit  bit(64)            null,
    constraint pk
        primary key (id)
);

insert into t (c_decimal, c_char, c_varchar, c_binary, c_varbinary, c_enum, c_set, c_bit)
values (123.456, "abc", "abc", "abc", "abc", 'a', 'a,b', b'1000001');
```

TiCDCの出力は次のようになります：

```json
{
    "id": 0,
    ...
    "isDdl": false,
    "sqlType": {
        ...
    },
    "mysqlType": {
        "c_binary": "binary",
        "c_bit": "bit",
        "c_char": "char",
        "c_decimal": "decimal",
        "c_enum": "enum",
        "c_set": "set",
        "c_varbinary": "varbinary",
        "c_varchar": "varchar",
        "id": "int"
    },
    "data": [
        {
            ...
        }
    ],
    "old": null,
}
```

公式のTiCDC Canalの出力は次のようになります：

```json
{
    "id": 0,
    ...
    "isDdl": false,
    "sqlType": {
        ...
    },
    "mysqlType": {
        "c_binary": "binary(16)",
        "c_bit": "bit(64)",
        "c_char": "char(16)",
        "c_decimal": "decimal(10, 4)",
        "c_enum": "enum('a','b','c')",
        "c_set": "set('a','b','c')",
        "c_varbinary": "varbinary(16)",
        "c_varchar": "varchar(16)",
        "id": "int"
    },
    "data": [
        {
            ...
        }
    ],
    "old": null,
}
```

## TiCDC Canal-JSONの変更 {#changes-in-ticdc-canal-json}

### `Delete`イベントの`Old`フィールドの変更 {#changes-in-the-old-field-of-the-delete-events}

v5.4.0から、`Delete`イベントの`old`フィールドが変更されました。

以下は`Delete`イベントメッセージです。v5.4.0以前では、`old`フィールドには"data"フィールドと同じ内容が含まれています。v5.4.0以降のバージョンでは、`old`フィールドはnullに設定されます。削除されたデータは"data"フィールドを使用して取得できます。

```
{
    "id": 0,
    "database": "test",
    ...
    "type": "DELETE",
    ...
    "sqlType": {
        ...
    },
    "mysqlType": {
        ...
    },
    "data": [
        {
            "c_bigint": "9223372036854775807",
            "c_int": "0",
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "0",
            "id": "2"
        }
    ],
    "old": null,
    // The following is an example before v5.4.0. The `old` field contains the same content as the "data" field.
    "old": [
        {
            "c_bigint": "9223372036854775807",
            "c_int": "0",
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "0",
            "id": "2"
        }
    ]
}
```
