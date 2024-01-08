---
title: Changefeed Overview
summary: Learn basic concepts, state definitions, and state transfer of changefeeds.
---

# Changefeed Overview {#changefeed-overview}

Changefeedの概要

A changefeed is a replication task in TiCDC, which replicates the data change logs of specified tables in a TiDB cluster to the designated downstream. You can run and manage multiple changefeeds in a TiCDC cluster.
changefeedは、TiCDCのレプリケーションタスクであり、指定されたテーブルのデータ変更ログをTiDBクラスターから指定されたダウンストリームにレプリケートします。TiCDCクラスターで複数のchangefeedを実行および管理できます。

## Changefeed state transfer {#changefeed-state-transfer}

Changefeedの状態転送

The state of a replication task represents the running status of the replication task. During the running of TiCDC, replication tasks might fail with errors, be manually paused, resumed, or reach the specified `TargetTs`. These behaviors can lead to the change of the replication task state. This section describes the states of TiCDC replication tasks and the transfer relationships between states.
レプリケーションタスクの状態は、レプリケーションタスクの実行状態を表します。TiCDCの実行中に、レプリケーションタスクはエラーで失敗したり、手動で一時停止したり、再開したり、指定された`TargetTs`に到達したりすることがあります。これらの動作により、レプリケーションタスクの状態が変わることがあります。このセクションでは、TiCDCレプリケーションタスクの状態と状態間の転送関係について説明します。

![TiCDC state transfer](/media/ticdc/ticdc-changefeed-state-transfer.png)

The states in the preceding state transfer diagram are described as follows:

前述の状態転送図の状態は次のように説明されます。

- `Normal`: The replication task runs normally and the checkpoint-ts proceeds normally.
- `Normal`: レプリケーションタスクは正常に実行され、checkpoint-tsが正常に進行します。
- `Stopped`: The replication task is stopped, because the user manually pauses the changefeed. The changefeed in this state blocks GC operations.
- `Stopped`: ユーザーがchangefeedを手動で一時停止したため、レプリケーションタスクが停止します。この状態のchangefeedはGC操作をブロックします。
- `Warning`: The replication task returns an error. The replication cannot continue due to some recoverable errors. The changefeed in this state keeps trying to resume until the state transfers to `Normal`. The maximum retry time is 30 minutes. If it exceeds this time, the changefeed enters a failed state. The changefeed in this state blocks GC operations.
- `Warning`: レプリケーションタスクがエラーを返します。いくつかの回復可能なエラーのため、レプリケーションを続行できません。この状態のchangefeedは、状態が`Normal`に転送されるまで再開し続けます。最大再試行時間は30分です。これを超えると、changefeedは失敗した状態になります。この状態のchangefeedはGC操作をブロックします。
- `Finished`: The replication task is finished and has reached the preset `TargetTs`. The changefeed in this state does not block GC operations.
- `Finished`: レプリケーションタスクが完了し、事前に設定された`TargetTs`に到達しました。この状態のchangefeedはGC操作をブロックしません。
- `Failed`: The replication task fails. The changefeed in this state does not keep trying to resume. To give you enough time to handle the failure, the changefeed in this state blocks GC operations. The duration of the blockage is specified by the `gc-ttl` parameter, with a default value of 24 hours. For v7.1.1 and later v7.1 patch versions, if the underlying issue is resolved within this duration, you can manually resume the changefeed. Otherwise, if the changefeed remains in this state beyond the `gc-ttl` duration, the replication task cannot resume and cannot be recovered.
- `Failed`: レプリケーションタスクが失敗しました。この状態のchangefeedは再開し続けません。失敗を処理するための十分な時間を確保するため、この状態のchangefeedはGC操作をブロックします。ブロックの期間は、デフォルト値が24時間の`gc-ttl`パラメータで指定されます。v7.1.1およびそれ以降のv7.1パッチバージョンの場合、この期間内に基本的な問題が解決されると、changefeedを手動で再開できます。それ以外の場合、`gc-ttl`の期間を超えてこの状態のchangefeedが残ると、レプリケーションタスクは再開できず、回復できません。

> **Note:**
>
> **Note:**
>
> If the changefeed encounters errors with error codes `ErrGCTTLExceeded`, `ErrSnapshotLostByGC`, or `ErrStartTsBeforeGC`, it does not block GC operations.
> changefeedが`ErrGCTTLExceeded`、`ErrSnapshotLostByGC`、または`ErrStartTsBeforeGC`のエラーコードを持つエラーに遭遇した場合、GC操作をブロックしません。

The numbers in the preceding state transfer diagram are described as follows.

前述の状態転送図の数字は次のように説明されます。

- ① Run the `changefeed pause` command.
- ① `changefeed pause`コマンドを実行します。
- ② Run the `changefeed resume` command to resume the replication task.
- ② `changefeed resume`コマンドを実行してレプリケーションタスクを再開します。
- ③ Recoverable errors occur during the `changefeed` operation, and the operation is retried automatically.
- ③ `changefeed`操作中に回復可能なエラーが発生し、操作が自動的に再試行されます。
- ④ The changefeed automatic retry succeeds, and `checkpoint-ts` continues to advance.
- ④ changefeedの自動再試行が成功し、`checkpoint-ts`が引き続き進行します。
- ⑤ The changefeed automatic retry exceeds 30 minutes and fails. The changefeed enters the failed state. At this time, the changefeed continues to block upstream GC for a duration specified by `gc-ttl`.
- ⑤ changefeedの自動再試行が30分を超えて失敗します。changefeedは失敗した状態になります。この時点で、changefeedは`gc-ttl`で指定された期間、上流GCをブロックし続けます。
- ⑥ The changefeed encounters an unrecoverable error and directly enters the failed state. At this time, the changefeed continues to block upstream GC for a duration specified by `gc-ttl`.
- ⑥ changefeedが回復不能なエラーに遭遇し、直接失敗した状態になります。この時点で、changefeedは`gc-ttl`で指定された期間、上流GCをブロックし続けます。
- ⑦ The replication progress of the changefeed reaches the value set by `target-ts`, and the replication is completed.
- ⑦ changefeedのレプリケーション進捗が`target-ts`で設定された値に到達し、レプリケーションが完了します。
- ⑧ The changefeed has been suspended for a duration longer than the value specified by `gc-ttl`, thus encountering GC advancement errors, and cannot be resumed.
- ⑧ changefeedは、`gc-ttl`で指定された値よりも長い期間一時停止されているため、GCの進行エラーに遭遇し、再開できません。
- ⑨ For v7.1.1 and later v7.1 patch versions, if the cause of the failure has been resolved, and the changefeed was suspended for a duration shorter than the value specified by `gc-ttl`, run the `changefeed resume` command to resume the replication task.
- ⑨ v7.1.1およびそれ以降のv7.1パッチバージョンの場合、失敗の原因が解決され、changefeedが`gc-ttl`で指定された値よりも短い期間一時停止されている場合、`changefeed resume`コマンドを実行してレプリケーションタスクを再開します。

## Operate changefeeds {#operate-changefeeds}

changefeedの操作

You can manage a TiCDC cluster and its replication tasks using the command-line tool `cdc cli`. For details, see [Manage TiCDC changefeeds](/ticdc/ticdc-manage-changefeed.md).
コマンドラインツール`cdc cli`を使用して、TiCDCクラスターとそのレプリケーションタスクを管理できます。詳細については、[Manage TiCDC changefeeds](/ticdc/ticdc-manage-changefeed.md)を参照してください。

You can also use the HTTP interface (the TiCDC OpenAPI feature) to manage a TiCDC cluster and its replication tasks. For details, see [TiCDC OpenAPI](/ticdc/ticdc-open-api.md).
HTTPインターフェース（TiCDC OpenAPI機能）を使用して、TiCDCクラスターとそのレプリケーションタスクを管理することもできます。詳細については、[TiCDC OpenAPI](/ticdc/ticdc-open-api.md)を参照してください。

If your TiCDC is deployed using TiUP, you can start `cdc cli` by running the `tiup ctl:v<CLUSTER_VERSION> cdc` command. Replace `v<CLUSTER_VERSION>` with the TiCDC cluster version, such as `v7.1.3`. You can also run `cdc cli` directly.
TiCDCがTiUPを使用して展開されている場合、`tiup ctl:v<CLUSTER_VERSION> cdc`コマンドを実行して`cdc cli`を起動できます。`v<CLUSTER_VERSION>`を`v7.1.3`などのTiCDCクラスターバージョンで置き換えます。また、`cdc cli`を直接実行することもできます。
