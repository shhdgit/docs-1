---
title: Changefeed DDL Replication
summary: Learn about the DDL statements supported by TiCDC and some special cases.
---

# Changefeed DDL Replication {#changefeed-ddl-replication}

このドキュメントでは、TiCDCにおけるDDLレプリケーションのルールと特殊なケースについて説明します。

## DDLの許可リスト {#ddl-allow-list}

現在、TiCDCは許可リストを使用して、DDLステートメントをレプリケートするかどうかを決定します。許可リストに含まれているDDLステートメントのみがダウンストリームにレプリケートされます。許可リストに含まれていないDDLステートメントはレプリケートされません。

TiCDCがサポートするDDLステートメントの許可リストは次のとおりです。

- データベースの作成
- データベースの削除
- テーブルの作成
- テーブルの削除
- カラムの追加
- カラムの削除
- インデックスの作成/追加
- インデックスの削除
- テーブルの切り捨て
- カラムの変更
- テーブルの名前変更
- カラムのデフォルト値の変更
- テーブルのコメントの変更
- インデックスの名前変更
- パーティションの追加
- パーティションの削除
- パーティションの切り捨て
- ビューの作成
- ビューの削除
- テーブルの文字セットの変更
- データベースの文字セットの変更
- テーブルの回復
- プライマリキーの追加
- プライマリキーの削除
- 自動IDの再設定
- テーブルのインデックスの可視性の変更
- パーティションの交換
- パーティションの再構成
- テーブルのTTLの変更
- テーブルのTTLの削除

## DDLレプリケーションの考慮事項 {#ddl-replication-considerations}

レプリケーションプロセス中に一部のコンテキストが欠落するため、TiCDCは`RENAME TABLE` DDLのレプリケーションに制約があります。

### DDLステートメントで単一のテーブルを名前変更する場合 {#rename-a-single-table-in-a-ddl-statement}

DDLステートメントが単一のテーブルの名前を変更する場合、古いテーブル名がフィルタールールに一致する場合のみ、TiCDCはDDLステートメントをレプリケートします。以下は例です。

あなたのchangefeedの構成ファイルが次のようになっていると仮定します。

```toml
[filter]
rules = ['test.t*']
```

TiCDCはこのタイプのDDLを次のように処理します。

| DDL                                   | レプリケートするかどうか        | 処理の理由                                                                                |
| ------------------------------------- | ------------------- | ------------------------------------------------------------------------------------ |
| `RENAME TABLE test.t1 TO test.t2`     | レプリケート              | `test.t1`がフィルタールールに一致                                                                |
| `RENAME TABLE test.t1 TO ignore.t1`   | レプリケート              | `test.t1`がフィルタールールに一致                                                                |
| `RENAME TABLE ignore.t1 TO ignore.t2` | 無視                  | `ignore.t1`がフィルタールールに一致しない                                                           |
| `RENAME TABLE test.n1 TO test.t1`     | エラーを報告してレプリケーションを終了 | `test.n1`がフィルタールールに一致せず、しかし`test.t1`がフィルタールールに一致する。この操作は違法です。この場合はエラーメッセージを参照してください。 |
| `RENAME TABLE ignore.t1 TO test.t1`   | エラーを報告してレプリケーションを終了 | 上記と同じ理由                                                                              |

### DDLステートメントで複数のテーブルを名前変更する場合 {#rename-multiple-tables-in-a-ddl-statement}

DDLステートメントが複数のテーブルの名前を変更する場合、TiCDCは古いデータベース名、古いテーブル名、および新しいデータベース名がすべてフィルタールールに一致する場合のみ、DDLステートメントをレプリケートします。

さらに、TiCDCはテーブル名を交換する`RENAME TABLE` DDLをサポートしていません。以下は例です。

あなたのchangefeedの構成ファイルが次のようになっていると仮定します。

```toml
[filter]
rules = ['test.t*']
```

TiCDCはこのタイプのDDLを次のように処理します。

| DDL                                                                        | レプリケートするかどうか | 処理の理由                                                                                                      |
| -------------------------------------------------------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------- |
| `RENAME TABLE test.t1 TO test.t2, test.t3 TO test.t4`                      | レプリケート       | すべてのデータベース名とテーブル名がフィルタールールに一致                                                                              |
| `RENAME TABLE test.t1 TO test.ignore1, test.t3 TO test.ignore2`            | レプリケート       | 古いデータベース名、古いテーブル名、新しいデータベース名がフィルタールールに一致                                                                   |
| `RENAME TABLE test.t1 TO ignore.t1, test.t2 TO test.t22;`                  | エラーを報告       | 新しいデータベース名`ignore`がフィルタールールに一致しない                                                                          |
| `RENAME TABLE test.t1 TO test.t4, test.t3 TO test.t1, test.t4 TO test.t3;` | エラーを報告       | `RENAME TABLE` DDLが`test.t1`と`test.t3`の名前を1つのDDLステートメントで交換しており、これをTiCDCが正しく処理できません。この場合はエラーメッセージを参照してください。 |

### SQLモード {#sql-mode}

デフォルトでは、TiCDCはDDLステートメントを解析するためにTiDBのデフォルトSQLモードを使用します。上流のTiDBクラスタがデフォルトでないSQLモードを使用している場合、TiCDCの構成ファイルでSQLモードを指定する必要があります。そうしないと、TiCDCはDDLステートメントを正しく解析できない可能性があります。TiDBのSQLモードについての詳細は、[SQL Mode](/sql-mode.md)を参照してください。

たとえば、上流のTiDBクラスタが`ANSI_QUOTES`モードを使用している場合、changefeedの構成ファイルで次のようにSQLモードを指定する必要があります。

```toml
# 値の中で、"ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"はTiDBのデフォルトSQLモードです。
# "ANSI_QUOTES"は上流のTiDBクラスタに追加されたSQLモードです。

sql-mode = "ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,ANSI_QUOTES"
```

SQLモードが構成されていない場合、TiCDCは一部のDDLステートメントを正しく解析できない可能性があります。たとえば：

```sql
CREATE TABLE "t1" ("a" int PRIMARY KEY);
```

TiDBのデフォルトSQLモードでは、二重引用符は識別子ではなく文字列として扱われるため、TiCDCはこのDDLステートメントを正しく解析できません。

したがって、レプリケーションタスクを作成する際には、上流のTiDBクラスタで使用されているSQLモードを構成ファイルで指定することをお勧めします。
