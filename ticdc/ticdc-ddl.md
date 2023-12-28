---
title: Changefeed DDL Replication
summary: Learn about the DDL statements supported by TiCDC and some special cases.
---

# Changefeed DDL レプリケーション {#changefeed-ddl-replication}

このドキュメントでは、TiCDCにおけるDDLレプリケーションのルールと特殊ケースについて説明します。

## DDL許可リスト {#ddl-allow-list}

現在、TiCDCは許可リストを使用して、DDLステートメントをレプリケートするかどうかを決定します。許可リストに含まれているDDLステートメントのみがダウンストリームにレプリケートされます。許可リストに含まれていないDDLステートメントはレプリケートされません。

TiCDCがサポートするDDLステートメントの許可リストは以下の通りです：

- データベースの作成
- データベースの削除
- テーブルの作成
- テーブルの削除
- カラムの追加
- カラムの削除
- インデックスの作成 / インデックスの追加
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
- テーブルのリカバリー
- プライマリキーの追加
- プライマリキーの削除
- 自動IDの再ベース
- テーブルのインデックスの可視性の変更
- パーティションの交換
- パーティションの再構成
- テーブルのTTLの変更
- テーブルのTTLの削除

## DDLレプリケーションの考慮事項 {#ddl-replication-considerations}

レプリケーションプロセス中に一部のコンテキストが欠落するため、TiCDCは`RENAME TABLE` DDLのレプリケーションに制約があります。

### 単一のテーブルをリネームするDDLステートメント {#rename-a-single-table-in-a-ddl-statement}

DDLステートメントが単一のテーブルをリネームする場合、古いテーブル名がフィルタールールと一致する場合にのみ、TiCDCはDDLステートメントをレプリケートします。以下は例です。

あなたのchangefeedの設定ファイルが以下のような場合を想定します：

```toml
[filter]
rules = ['test.t*']
```

TiCDCは、このタイプのDDLを以下のように処理します：

| DDL                                   | レプリケートするかどうか          | 処理の理由                                                                                          |
| ------------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------- |
| `RENAME TABLE test.t1 TO test.t2`     | レプリケートする              | `test.t1`はフィルタールールに一致します                                                                       |
| `RENAME TABLE test.t1 TO ignore.t1`   | レプリケートする              | `test.t1`はフィルタールールに一致します                                                                       |
| `RENAME TABLE ignore.t1 TO ignore.t2` | 無視する                  | `ignore.t1`はフィルタールールに一致しません                                                                    |
| `RENAME TABLE test.n1 TO test.t1`     | エラーを報告してレプリケーションを終了する | `test.n1`はフィルタールールに一致しませんが、`test.t1`はフィルタールールに一致します。この操作は不正です。この場合、処理方法についてはエラーメッセージを参照してください。 |
| `RENAME TABLE ignore.t1 TO test.t1`   | エラーを報告してレプリケーションを終了する | 上記と同じ理由。                                                                                       |

### DDLステートメントで複数のテーブルの名前を変更する {#rename-multiple-tables-in-a-ddl-statement}

DDLステートメントで複数のテーブルの名前を変更する場合、TiCDCは古いデータベース名、古いテーブル名、および新しいデータベース名がすべてフィルタールールに一致する場合にのみ、DDLステートメントをレプリケートします。

また、TiCDCはテーブル名を入れ替える`RENAME TABLE`DDLをサポートしていません。以下は例です。

あなたのchangefeedの設定ファイルが以下のような場合を想定してください：

```toml
[filter]
rules = ['test.t*']
```

TiCDCは、このタイプのDDLを以下のように処理します：

| DDL                                                                        | レプリケートするかどうか | 処理の理由                                                                                                               |
| -------------------------------------------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------- |
| `RENAME TABLE test.t1 TO test.t2, test.t3 TO test.t4`                      | レプリケートする     | すべてのデータベース名とテーブル名がフィルタルールに一致するため。                                                                                   |
| `RENAME TABLE test.t1 TO test.ignore1, test.t3 TO test.ignore2`            | レプリケートする     | 古いデータベース名、古いテーブル名、および新しいデータベース名がフィルタルールに一致するため。                                                                     |
| `RENAME TABLE test.t1 TO ignore.t1, test.t2 TO test.t22;`                  | エラーを報告する     | 新しいデータベース名 `ignore` がフィルタルールに一致しないため。                                                                               |
| `RENAME TABLE test.t1 TO test.t4, test.t3 TO test.t1, test.t4 TO test.t3;` | エラーを報告する     | `RENAME TABLE` DDLは、TiCDCが正しく処理できない1つのDDLステートメントで `test.t1` と `test.t3` の名前を入れ替えるため、この場合は処理方法についてエラーメッセージを参照してください。 |

### SQLモード {#sql-mode}

デフォルトでは、TiCDCはDDLステートメントを解析するためにTiDBのデフォルトのSQLモードを使用します。上流のTiDBクラスターがデフォルト以外のSQLモードを使用している場合は、TiCDCの設定ファイルでSQLモードを指定する必要があります。そうしないと、TiCDCはDDLステートメントを正しく解析できない可能性があります。TiDBのSQLモードの詳細については、[SQLモード](/sql-mode.md)を参照してください。

例えば、上流のTiDBクラスターが `ANSI_QUOTES` モードを使用している場合、changefeedの設定ファイルでSQLモードを指定する必要があります：

````yaml
config:
  case-sensitive: true
  enable-old-value: false
  filter:
    rules:
      - do-tables:
          - db-name: "*"
          - tbl-name: "*"
  mounter:
    worker-num: 16
  sink:
    dispatchers:
      - blackhole:
          blackhole-config:
            check-requirements: false
  cdc:
    schema-storage:
      storage: file
      path: ./schema
    sort-dir: ./sorter
    filter:
      rules:
        - do-tables-or-filters:
            - db-name: "*"
            - tbl-name: "*"
    enable-old-value: false
    from:
      host: 127.0.0.1
      port: 4000
      user: root
      password: ""
    to:
      host: 127.0.0.1
      port: 4001
      user: root
      password: ""
      security:
        ca-path: ""
        cert-path: ""
        key-path: ""
    kafka:
      pd-addrs: ""
      kafka-version: 2.4.0
      max-msg-size: 67108864
      client-id: ""
      tls:
        enabled: false
        cert-path: ""
        key-path: ""
        ca-path: ""
      sasl:
        enable: false
        user: ""
        password: ""
    sync-point-interval: 0
    gc-ttl: 86400
    owner-flush-interval: 0
    owner-flush-size: 0
    processor-flush-interval: 0
    processor-flush-size: 0
    sort-engine-size: 0
    sort-engine-max-memory-consumption: 0
    sort-engine-max-memory-percentage: 0
    sort-engine-max-concurrent-writes: 0
    sort-dir: ""
    sort-engine-max-file-size: 0
    sort-engine-max-open-files: 0
    sort-engine-max-merge-concurrent-files: 0
    sort-engine-max-memory-percentage-for-heap: 0
    sort-engine-max-memory-percentage-for-heap-4-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-8-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-16-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-32-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-64-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-128-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-256-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-512-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-1024-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-2048-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-4096-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-8192-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-16384-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-32768-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-65536-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-131072-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-262144-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-524288-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-1048576-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-2097152-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-4194304-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-8388608-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-16777216-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-33554432-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-67108864-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-134217728-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-268435456-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-536870912-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-1073741824-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-2147483648-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-4294967296-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-8589934592-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-17179869184-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-34359738368-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-68719476736-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-137438953472-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-274877906944-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-549755813888-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-1099511627776-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-2199023255552-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-4398046511104-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-8796093022208-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-17592186044416-multi-way-merge: 0
    sort-engine-max-memory-percentage-for-heap-35184372088832-multi-way-merge: 0
    sort

```toml
# In the value, "ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION" is the default SQL mode of TiDB.
# "ANSI_QUOTES" is the SQL mode added to your upstream TiDB cluster.

sql-mode = "ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,ANSI_QUOTES"
````

SQLモードが設定されていない場合、TiCDCは一部のDDLステートメントを正しく解析できない可能性があります。例えば：

```sql
CREATE TABLE "t1" ("a" int PRIMARY KEY);
```

デフォルトのTiDB SQLモードでは、二重引用符は識別子ではなく文字列として扱われるため、TiCDCはDDLステートメントを正しく解析できません。

そのため、レプリケーションタスクを作成する際には、アップストリームのTiDBクラスタで使用されているSQLモードを構成ファイルで指定することを推奨します。
