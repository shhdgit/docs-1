---
title: Query Task Status in TiDB Data Migration
summary: Learn how to query the status of a data replication task.
---

# TiDBデータ移行でのクエリタスクステータスのクエリ {#query-task-status-in-tidb-data-migration}

このドキュメントでは、`query-status`コマンドを使用してタスクのステータスとDMのサブタスクのステータスをクエリする方法について紹介します。

## クエリ結果 {#query-result}

以下の手順で`query-status`を使用することをお勧めします。

1. `query-status`を使用して、各進行中のタスクが正常な状態にあるかどうかを確認します。
2. タスクでエラーが発生した場合は、`query-status <taskName>`コマンドを使用して詳細なエラー情報を確認します。このコマンドの`<taskName>`はエラーに遭遇したタスクの名前を示します。

成功したクエリ結果は次のようになります：

```bash
» query-status
```

```json
{
    "result": true,
    "msg": "",
    "tasks": [
        {
            "taskName": "test",
            "taskStatus": "Running",
            "sources": [
                "mysql-replica-01",
                "mysql-replica-02"
            ]
        },
        {
            "taskName": "test2",
            "taskStatus": "Paused",
            "sources": [
                "mysql-replica-01",
                "mysql-replica-02"
            ]
        }
    ]
}
```

クエリ結果の一部は次のように説明されています。

- `result`: クエリが成功したかどうか。
- `msg`: クエリが失敗した場合に返されるエラーメッセージ。
- `tasks`: 移行タスクのリスト。各タスクには次のフィールドが含まれています。
  - `taskName`: タスクの名前。
  - `taskStatus`: タスクのステータス。`taskStatus`の詳細な説明については、[タスクステータス](#task-status)を参照してください。
  - `sources`: 上流のMySQLデータベースのリスト。

## タスクステータス {#task-status}

DM移行タスクのステータスは、DM-workerに割り当てられた各サブタスクのステータスに依存します。サブタスクステータスの詳細については、[サブタスクステータス](#subtask-status)を参照してください。以下の表は、サブタスクステータスがタスクステータスにどのように関連しているかを示しています。

| タスク内のサブタスクステータス                                                                            | タスクステータス                                |
| :----------------------------------------------------------------------------------------- | :-------------------------------------- |
| 1つのサブタスクが `paused` の状態でエラー情報が返されます。                                                        | `エラー - サブタスクでエラーが発生しました`                |
| Syncフェーズの1つのサブタスクが `Running` の状態ですが、Relay処理ユニットが実行されていない（`Error`/`Paused`/`Stopped`の状態）です。 | `エラー - RelayステータスがError/Paused/Stopped` |
| 1つのサブタスクが `Paused` の状態でエラー情報が返されません。                                                       | `一時停止`                                  |
| すべてのサブタスクが `New` の状態です。                                                                    | `新規`                                    |
| すべてのサブタスクが `Finished` の状態です。                                                               | `完了`                                    |
| すべてのサブタスクが `Stopped` の状態です。                                                                | `停止`                                    |
| その他の状況                                                                                     | `実行中`                                   |

## 詳細なクエリ結果 {#detailed-query-result}

```bash
» query-status test
```

```json
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "sourceStatus": {
                "source": "mysql-replica-01",
                "worker": "worker1",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [
                {
                    "name": "test",
                    "stage": "Running",
                    "unit": "Sync",
                    "result": null,
                    "unresolvedDDLLockID": "test-`test`.`t_target`",
                    "sync": {
                        "masterBinlog": "(bin.000001, 3234)",
                        "masterBinlogGtid": "c0149e17-dff1-11e8-b6a8-0242ac110004:1-14",
                        "syncerBinlog": "(bin.000001, 2525)",
                        "syncerBinlogGtid": "",
                        "blockingDDLs": [
                            "USE `test`; ALTER TABLE `test`.`t_target` DROP COLUMN `age`;"
                        ],
                        "unresolvedGroups": [
                            {
                                "target": "`test`.`t_target`",
                                "DDLs": [
                                    "USE `test`; ALTER TABLE `test`.`t_target` DROP COLUMN `age`;"
                                ],
                                "firstPos": "(bin|000001.000001, 3130)",
                                "synced": [
                                    "`test`.`t2`"
                                    "`test`.`t3`"
                                    "`test`.`t1`"
                                ],
                                "unsynced": [
                                ]
                            }
                        ],
                        "synced": false,
                        "totalRows": "12",
                        "totalRps": "1",
                        "recentRps": "1"
                    }
                }
            ]
        },
        {
            "result": true,
            "msg": "",
            "sourceStatus": {
                "source": "mysql-replica-02",
                "worker": "worker2",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [
                {
                    "name": "test",
                    "stage": "Running",
                    "unit": "Load",
                    "result": null,
                    "unresolvedDDLLockID": "",
                    "load": {
                        "finishedBytes": "115",
                        "totalBytes": "452",
                        "progress": "25.44 %",
                        "bps": "2734"
                    }
                }
            ]
        },
        {
            "result": true,
            "sourceStatus": {
                "source": "mysql-replica-03",
                "worker": "worker3",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [
                {
                    "name": "test",
                    "stage": "Paused",
                    "unit": "Load",
                    "result": {
                        "isCanceled": false,
                        "errors": [
                            {
                                "Type": "ExecSQL",
                                "msg": "Error 1062: Duplicate entry '1155173304420532225' for key 'PRIMARY'\n/home/jenkins/workspace/build_dm/go/src/github.com/pingcap/tidb-enterprise-tools/loader/db.go:160: \n/home/jenkins/workspace/build_dm/go/src/github.com/pingcap/tidb-enterprise-tools/loader/db.go:105: \n/home/jenkins/workspace/build_dm/go/src/github.com/pingcap/tidb-enterprise-tools/loader/loader.go:138: file test.t1.sql"
                            }
                        ],
                        "detail": null
                    },
                    "unresolvedDDLLockID": "",
                    "load": {
                        "finishedBytes": "0",
                        "totalBytes": "156",
                        "progress": "0.00 %",
                        "bps": "0"
                    }
                }
            ]
        },
        {
            "result": true,
            "msg": "",
            "sourceStatus": {
                "source": "mysql-replica-04",
                "worker": "worker4",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [
                {
                    "name": "test",
                    "stage": "Running",
                    "unit": "Dump",
                    "result": null,
                    "unresolvedDDLLockID": "",
                    "dump": {
                        "totalTables": "10",
                        "completedTables": "3",
                        "finishedBytes": "2542",
                        "finishedRows": "32",
                        "estimateTotalRows": "563",
                        "progress": "30.52 %",
                        "bps": "445"
                    }
                }
            ]
        },
    ]
}
```

返された結果の一部は次のように説明されています。

- `result`: クエリが成功したかどうか。
- `msg`: クエリが失敗した場合に返されるエラーメッセージ。
- `sources`: 上流のMySQLインスタンスのリスト。各ソースには次のフィールドが含まれています。
  - `result`
  - `msg`
  - `sourceStatus`: 上流のMySQLデータベースの情報。
  - `subTaskStatus`: 上流のMySQLデータベースのすべてのサブタスクの情報。各サブタスクには次のフィールドが含まれる場合があります。
    - `name`: サブタスクの名前。
    - `stage`: サブタスクのステータス。"stage"の"subTaskStatus"の"sources"のステータスの説明とステータスの切り替え関係については、[サブタスクステータス](#subtask-status)を参照してください。
    - `unit`: DMの処理ユニット。"Check"、"Dump"、"Load"、"Sync"を含みます。
    - `result`: サブタスクが失敗した場合のエラー情報を表示します。
    - `unresolvedDDLLockID`: 異常な状態でのシャーディングDDLロックID。"sources"の"subTaskStatus"の"unresolvedDDLLockID"の操作の詳細については、[シャーディングDDLロックの手動ハンドリング](/dm/manually-handling-sharding-ddl-locks.md)を参照してください。
    - `sync`: `Sync`処理ユニットのレプリケーション情報。この情報は現在の処理ユニットと同じコンポーネントに関するものです。
      - `masterBinlog`: 上流データベースのbinlog位置。
      - `masterBinlogGtid`: 上流データベースのGTID情報。
      - `syncerBinlog`: `Sync`処理ユニットでレプリケートされたbinlogの位置。
      - `syncerBinlogGtid`: GTIDを使用してレプリケートされたbinlogの位置。
      - `blockingDDLs`: 現在ブロックされているDDLリスト。このDM-workerのすべての上流テーブルが「synced」ステータスにある場合のみ空ではありません。この場合、実行されるべきまたはスキップされるシャーディングDDLステートメントを示します。
      - `unresolvedGroups`: 解決されていないシャーディンググループ。各グループには次のフィールドが含まれます。
        - `target`: レプリケートする下流のデータベーステーブル。
        - `DDLs`: DDLステートメントのリスト。
        - `firstPos`: シャーディングDDLステートメントの開始位置。
        - `synced`: `Sync`ユニットで実行された上流のシャーディングテーブル。
        - `unsynced`: このシャーディングDDLステートメントが実行されていない上流テーブル。上流のテーブルがレプリケーションを完了していない場合、`blockingDDLs`は空です。
      - `synced`: 増分レプリケーションが上流に追いついており、上流と同じbinlog位置を持っているかどうか。`Sync`バックグラウンドではリアルタイムでセーブポイントが更新されないため、`synced`の`false`は常にレプリケーション遅延が存在することを意味しません。
      - `totalRows`: このサブタスクでレプリケートされた合計行数。
      - `totalRps`: このサブタスクで秒間にレプリケートされた行数。
      - `recentRps`: 直近1秒間にこのサブタスクでレプリケートされた行数。
    - `load`: `Load`処理ユニットのレプリケーション情報。
      - `finishedBytes`: ロードされたバイト数。
      - `totalBytes`: ロードする必要のある合計バイト数。
      - `progress`: ロードプロセスの進行状況。
      - `bps`: フルロードの速度。
    - `dump`: `Dump`処理ユニットのレプリケーション情報。
      - `totalTables`: ダンプするテーブルの数。
      - `completedTables`: ダンプされたテーブルの数。
      - `finishedBytes`: ダンプされたバイト数。
      - `finishedRows`: ダンプされた行数。
      - `estimateTotalRows`: ダンプする見積もりの行数。
      - `progress`: ダンププロセスの進行状況。
      - `bps`: バイト/秒でのダンプ速度。

## サブタスクステータス {#subtask-status}

### ステータスの説明 {#status-description}

- `New`:

  - 初期ステータス。
  - サブタスクがエラーに遭遇しない場合、`Running`に切り替わります。それ以外の場合は`Paused`に切り替わります。

- `Running`: 通常の実行ステータス。

- `Paused`:

  - 一時停止ステータス。
  - サブタスクがエラーに遭遇した場合、`Paused`に切り替わります。
  - サブタスクが`Running`ステータスのときに`pause-task`を実行すると、タスクは`Paused`に切り替わります。
  - このステータスのとき、`resume-task`コマンドを実行してタスクを再開できます。

- `Stopped`:

  - 停止ステータス。
  - サブタスクが`Running`または`Paused`ステータスのときに`stop-task`を実行すると、タスクは`Stopped`に切り替わります。
  - このステータスのとき、`resume-task`を使用してタスクを再開することはできません。

- `Finished`:

  - 完了したサブタスクのステータス。
  - フルレプリケーションサブタスクが正常に終了した場合のみ、タスクはこのステータスに切り替わります。

### ステータス切り替え図 {#status-switch-diagram}

```
                                         error occurs
                            New --------------------------------|
                             |                                  |
                             |           resume-task            |
                             |  |----------------------------|  |
                             |  |                            |  |
                             |  |                            |  |
                             v  v        error occurs        |  v
  Finished <-------------- Running -----------------------> Paused
                             ^  |        or pause-task       |
                             |  |                            |
                  start task |  | stop task                  |
                             |  |                            |
                             |  v        stop task           |
                           Stopped <-------------------------|
```
