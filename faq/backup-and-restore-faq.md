---
title: Backup & Restore FAQs
summary: Learn about Frequently Asked Questions (FAQs) and the solutions of backup and restore.
---

# バックアップとリストアのFAQ {#backup-restore-faqs}

このドキュメントでは、TiDB Backup & Restore（BR）のよくある質問（FAQ）と解決策をリストします。

## 誤ってデータを削除または更新した後、データを迅速に復元するにはどうすればよいですか？ {#what-should-i-do-to-quickly-recover-data-after-mistakenly-deleting-or-updating-data}

TiDB v6.4.0では、フラッシュバック機能が導入されました。この機能を使用すると、GC時間内に指定した時刻までデータを迅速に復元できます。したがって、誤操作が発生した場合は、この機能を使用してデータを復元できます。詳細については、[クラスターのフラッシュバック](/sql-statements/sql-statement-flashback-cluster.md)と[データベースのフラッシュバック](/sql-statements/sql-statement-flashback-database.md)を参照してください。

## TiDB v5.4.0以降のバージョンでは、バックアップタスクを重いワークロードのクラスターで実行すると、バックアップタスクの速度が遅くなるのはなぜですか？ {#in-tidb-v5-4-0-and-later-versions-when-backup-tasks-are-performed-on-the-cluster-under-a-heavy-workload-why-does-the-speed-of-backup-tasks-become-slow}

TiDB v5.4.0から、BRはバックアップタスクのための自動チューニング機能を導入しました。v5.4.0以降のクラスターでは、この機能がデフォルトで有効になっています。クラスターのワークロードが重い場合、この機能はバックアップタスクに使用されるリソースを制限して、オンラインクラスターへの影響を減らします。詳細については、[バックアップの自動チューニング](/br/br-auto-tune.md)を参照してください。

TiKVは[動的に設定](/tikv-control.md#modify-the-tikv-configuration-dynamically)することができるため、自動チューニング機能を有効または無効にすることができます。次の方法で機能を有効または無効にします。クラスターを再起動する必要はありません。

- 自動チューニングを無効にする：TiKVの設定項目[`backup.enable-auto-tune`](/tikv-configuration-file.md#enable-auto-tune-new-in-v540)を`false`に設定します。
- 自動チューニングを有効にする：`backup.enable-auto-tune`を`true`に設定します。v5.3.xからv5.4.0以降のバージョンにアップグレードしたクラスターでは、自動チューニング機能はデフォルトで無効になっています。手動で有効にする必要があります。

`tiup-cluster`を使用して自動チューニングを有効または無効にするには、[自動チューニングを使用する](/br/br-auto-tune.md#use-auto-tune)を参照してください。

また、自動チューニングはバックアップタスクに使用されるスレッドのデフォルト数を減らします。詳細については、`backup.num-threads`]\(/tikv-configuration-file.md#num-threads-1)を参照してください。そのため、Grafanaダッシュボードでは、バックアップタスクに使用される速度、CPU使用率、およびI/Oリソースの使用率は、v5.4.0より前のバージョンよりも低くなります。v5.4.0より前のバージョンでは、`backup.num-threads`のデフォルト値は`CPU * 0.75`であり、バックアップタスクに使用されるスレッドの数は論理CPUコアの75%を占めます。最大値は`32`でした。v5.4.0から、この設定項目のデフォルト値は`CPU * 0.5`になり、最大値は`8`になりました。

オフラインクラスターでバックアップタスクを実行する場合、バックアップを高速化するために、`tikv-ctl`を使用して`backup.num-threads`の値を大きな数に変更できます。

## PITRの問題 {#pitr-issues}

### [PITR](/br/br-pitr-guide.md)と[クラスターのフラッシュバック](/sql-statements/sql-statement-flashback-cluster.md)の違いは何ですか？ {#what-is-the-difference-between-pitr-br-br-pitr-guide-md-and-cluster-flashback-sql-statements-sql-statement-flashback-cluster-md}

ユースケースの観点から見ると、PITRは通常、クラスターが完全にサービス外になったり、データが破損して他のソリューションでは回復できない場合に、クラスターのデータを指定した時刻に復元するために使用されます。PITRを使用するには、データ復旧用の新しいクラスターが必要です。クラスターのフラッシュバック機能は、ユーザーの誤操作やその他の要因によって引き起こされるデータエラーのシナリオに特化しており、クラスターのデータを最新のタイムスタンプに戻すことができます。

ほとんどの場合、フラッシュバックは、人為的なミスによって引き起こされるデータエラーのための回復ソリューションとして、PITRよりも優れています。なぜなら、RPO（ほぼゼロ）とRTOがはるかに短いからです。ただし、クラスターが完全に使用できない場合、フラッシュバックはこの時点で実行できないため、PITRはこの場合、クラスターを回復する唯一の解決策です。そのため、PITRは常にデータベースのディザスタリカバリ戦略を開発する際に必要なソリューションです。フラッシュバックよりもRPO（最大5分）とRTOが長いです。

### 上流データベースが物理インポートモードでTiDB Lightningを使用してデータをインポートすると、ログバックアップ機能が使用できなくなります。なぜですか？ {#when-the-upstream-database-imports-data-using-tidb-lightning-in-the-physical-import-mode-the-log-backup-feature-becomes-unavailable-why}

現在、ログバックアップ機能はTiDB Lightningに完全に適応されていません。そのため、TiDB Lightningの物理モードでインポートされたデータはログデータにバックアップできません。

アップストリームクラスターでは、ログバックアップタスクを作成する際に、TiDB Lightningの物理モードを使用しないようにしてください。代わりに、TiDB Lightningの論理モードを使用することができます。物理モードを使用する必要がある場合は、インポートが完了した後にスナップショットバックアップを実行し、スナップショットバックアップの後の時点にPITRを復元できるようにしてください。

### インデックス追加機能の高速化は、PITRと互換性がありませんか？ {#why-is-the-acceleration-of-adding-indexes-feature-incompatible-with-pitr}

問題：[#38045](https://github.com/pingcap/tidb/issues/38045)

現在、[インデックス高速化](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)機能を使用して作成されたインデックスデータは、PITRによってバックアップすることができません。

そのため、PITRの復元が完了した後、BRはインデックス高速化によって作成されたインデックスデータを削除し、再作成します。インデックス高速化によって多数のインデックスが作成された場合や、ログバックアップ中にインデックスデータが大きい場合は、インデックスを作成した後にフルバックアップを実行することをお勧めします。

### クラスターはネットワーク分割障害から回復しましたが、ログバックアップタスクの進捗のチェックポイントはまだ再開されません。なぜですか？ {#the-cluster-has-recovered-from-the-network-partition-failure-but-the-checkpoint-of-the-log-backup-task-progress-still-does-not-resume-why}

問題：[#13126](https://github.com/tikv/tikv/issues/13126)

クラスターでネットワーク分割障害が発生した後、バックアップタスクはログをバックアップし続けることができません。一定のリトライ時間後、タスクは `ERROR` 状態に設定されます。この時点で、バックアップタスクは停止します。

この問題を解決するには、`br log resume` コマンドを手動で実行してログバックアップタスクを再開する必要があります。

### PITRを実行すると、`execute over region id` エラーが返された場合はどうすればよいですか？ {#what-should-i-do-if-the-error-execute-over-region-id-is-returned-when-i-perform-pitr}

問題：[#37207](https://github.com/pingcap/tidb/issues/37207)

この問題は、フルデータインポート中にログバックアップを有効にし、その後データインポート中の時点でデータを復元するためにPITRを実行した場合に発生することがあります。

具体的には、長時間（24時間など）ホットスポットライトが発生し、各TiKVノードのOPSが50k/sを超える場合（Grafanaでメトリクスを表示できます：**TiKV-Details** -> **Backup Log** -> **Handle Event Rate**）、この問題が発生する可能性があります。

データインポート後にスナップショットバックアップを実行し、このスナップショットバックアップに基づいてPITRを実行することをお勧めします。

## `br restore point` コマンドを使用してダウンストリームクラスターを復元した後、TiFlashからデータにアクセスできません。どうすればよいですか？ {#after-restoring-a-downstream-cluster-using-the-br-restore-point-command-data-cannot-be-accessed-from-tiflash-what-should-i-do}

現在、PITRは復元フェーズ中にデータを直接TiFlashに書き込むことをサポートしていません。代わりに、brコマンドラインツールは`ALTER TABLE table_name SET TIFLASH REPLICA ***` DDLを実行してデータをレプリケートします。そのため、PITRがデータの復元を完了した後、TiFlashレプリカはすぐには利用できません。代わりに、データがTiKVノードからレプリケートされるまで一定の時間を待つ必要があります。レプリケーションの進捗状況を確認するには、`INFORMATION_SCHEMA.tiflash_replica` テーブルの `progress` 情報を確認してください。

### ログバックアップタスクの `status` が `ERROR` になった場合はどうすればよいですか？ {#what-should-i-do-if-the-status-of-a-log-backup-task-becomes-error}

ログバックアップタスク中に、タスクが失敗し、リトライしても回復できない場合、タスクのステータスは `ERROR` になります。以下は例です：

```shell
br log status --pd x.x.x.x:2379

● Total 1 Tasks.
> #1 <
                    name: task1
                  status: ○ ERROR
                   start: 2022-07-25 13:49:02.868 +0000
                     end: 2090-11-18 14:07:45.624 +0000
                 storage: s3://tmp/br-log-backup0ef49055-5198-4be3-beab-d382a2189efb/Log
             speed(est.): 0.00 ops/s
      checkpoint[global]: 2022-07-25 14:46:50.118 +0000; gap=11h31m29s
          error[store=1]: KV:LogBackup:RaftReq
error-happen-at[store=1]: 2022-07-25 14:54:44.467 +0000; gap=11h23m35s
  error-message[store=1]: retry time exceeds: and error failed to get initial snapshot: failed to get the snapshot (region_id = 94812): Error during requesting raftstore: message: "read index not ready, reason can not read index due to merge, region 94812" read_index_not_ready { reason: "can not read index due to merge" region_id: 94812 }: failed to get initial snapshot: failed to get the snapshot (region_id = 94812): Error during requesting raftstore: message: "read index not ready, reason can not read index due to merge, region 94812" read_index_not_ready { reason: "can not read index due to merge" region_id: 94812 }: failed to get initial snapshot: failed to get the snapshot (region_id = 94812): Error during requesting raftstore: message: "read index not ready, reason can not read index due to merge, region 94812" read_index_not_ready { reason: "can not read index due to merge" region_id: 94812 }
```

この問題を解決するためには、原因を確認し、指示に従って実行してください。問題が解決した後、次のコマンドを実行してタスクを再開してください。

```shell
br log resume --task-name=task1 --pd x.x.x.x:2379
```

バックアップタスクが再開されたら、`br log status`を使用してステータスを確認できます。タスクのステータスが`NORMAL`になると、バックアップタスクは継続されます。

```shell
● Total 1 Tasks.
> #1 <
              name: task1
            status: ● NORMAL
             start: 2022-07-25 13:49:02.868 +0000
               end: 2090-11-18 14:07:45.624 +0000
           storage: s3://tmp/br-log-backup0ef49055-5198-4be3-beab-d382a2189efb/Log
       speed(est.): 15509.75 ops/s
checkpoint[global]: 2022-07-25 14:46:50.118 +0000; gap=6m28s
```

> **注意:**
>
> この機能は複数のバージョンのデータをバックアップします。長時間のバックアップタスクが失敗し、ステータスが `ERROR` になると、このタスクのチェックポイントデータが `安全ポイント` として設定され、`安全ポイント` のデータは24時間以内にガベージコレクションされません。そのため、エラーが発生した後、バックアップタスクは最後のチェックポイントから続行されます。タスクが24時間以上失敗し、最後のチェックポイントデータがガベージコレクションされている場合、タスクを再開するとエラーが報告されます。この場合、最初に `br log stop` コマンドを実行してタスクを停止し、次に新しいバックアップタスクを開始する必要があります。

### `br log resume` コマンドを使用して中断されたタスクを再開する際に `ErrBackupGCSafepointExceeded` エラーメッセージが返された場合の対処方法は何ですか？ {#what-should-i-do-if-the-error-message-errbackupgcsafepointexceeded-is-returned-when-using-the-br-log-resume-command-to-resume-a-suspended-task}

```shell
Error: failed to check gc safePoint, checkpoint ts 433177834291200000: GC safepoint 433193092308795392 exceed TS 433177834291200000: [BR:Backup:ErrBackupGCSafepointExceeded]backup GC safepoint exceeded
```

ログバックアップタスクを一時停止した後、MVCCデータがガベージコレクションされるのを防ぐために、一時停止タスクプログラムは現在のチェックポイントを自動的にサービスセーフポイントとして設定します。これにより、24時間以内に生成されたMVCCデータが保持されます。バックアップチェックポイントのMVCCデータが24時間以上生成されている場合、チェックポイントのデータはガベージコレクションされ、バックアップタスクを再開することができません。

この問題を解決するには、`br log stop`を使用して現在のタスクを削除し、`br log start`を使用してログバックアップタスクを作成します。同時に、後続のPITRのためにフルバックアップを実行することもできます。

## 機能の互換性の問題 {#feature-compatibility-issues}

### なぜ、brコマンドラインツールを使用して復元したデータをTiCDCまたはDrainerの上流クラスタにレプリケートできないのですか？ {#why-does-data-restored-using-br-command-line-tool-cannot-be-replicated-to-the-upstream-cluster-of-ticdc-or-drainer}

- **BRで復元したデータは、下流にレプリケートできません**。これは、BRがSSTファイルを直接インポートするため、下流クラスタが現在これらのファイルを上流から取得できないためです。

- v4.0.3より前のバージョンでは、復元中に生成されたDDLジョブがTiCDC/Drainerで予期しないDDL実行を引き起こす可能性があります。そのため、TiCDC/Drainerの上流クラスタで復元を実行する必要がある場合は、brコマンドラインツールを使用して復元されたすべてのテーブルをTiCDC/Drainerのブロックリストに追加します。

TiCDCのブロックリストを設定するには、[`filter.rules`](https://github.com/pingcap/tiflow/blob/7c3c2336f98153326912f3cf6ea2fbb7bcc4a20c/cmd/changefeed.toml#L16)を使用し、Drainerのブロックリストを設定するには[`syncer.ignore-table`](/tidb-binlog/tidb-binlog-configuration-file.md#ignore-table)を使用します。

### 復元中に`new_collation_enabled`の不一致が報告されるのはなぜですか？ {#why-is-new-collation-enabled-mismatch-reported-during-restore}

TiDB v6.0.0以降、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)のデフォルト値が`false`から`true`に変更されました。BRは、上流クラスタの`mysql.tidb`テーブルに`new_collation_enabled`構成をバックアップし、この構成の値が上流と下流のクラスタで一致するかどうかをチェックします。値が一致する場合、BRは上流クラスタでバックアップされたデータを下流クラスタに安全に復元します。値が一致しない場合、BRはデータの復元を実行せず、エラーを報告します。

以前のv6.0.0以前のバージョンのTiDBクラスタでデータをバックアップし、このデータをv6.0.0以降のTiDBクラスタに復元する必要がある場合は、上流と下流のクラスタで`new_collations_enabled_on_first_bootstrap`の値が一致するかどうかを手動で確認する必要があります。

- 値が一致する場合、復元コマンドに`--check-requirements=false`を追加して、この構成チェックをスキップできます。
- 値が一致しない場合、強制的に復元を実行し、BRはデータ検証エラーを報告します。

### プレースメントルールをクラスタに復元する際にエラーが発生するのはなぜですか？ {#why-does-an-error-occur-when-i-restore-placement-rules-to-a-cluster}

v6.0.0以前では、BRは[プレースメントルール](/placement-rules-in-sql.md)をサポートしていませんでした。v6.0.0以降、BRはプレースメントルールをサポートし、プレースメントルールのバックアップおよび復元モードを制御するためのコマンドラインオプション`--with-tidb-placement-mode=strict/ignore`を導入しました。デフォルト値の`strict`では、BRはプレースメントルールをインポートおよび検証しますが、値が`ignore`の場合はすべてのプレースメントルールを無視します。

## データの復元の問題 {#data-restore-issues}

### `Io(Os...)`エラーを処理するにはどうすればよいですか？ {#what-should-i-do-to-handle-the-io-os-error}

これらの問題のほとんどは、TiKVがデータをディスクに書き込む際に発生するシステムコールエラーです。たとえば、`Io(Os {code: 13, kind: PermissionDenied...})`や`Io(Os {code: 2, kind: NotFound...})`などです。

このような問題を解決するには、まずバックアップディレクトリのマウント方法とファイルシステムを確認し、別のフォルダや別のハードディスクにデータをバックアップしてみてください。

たとえば、`samba`で構築したネットワークディスクにデータをバックアップすると、`Code: 22(invalid argument)`エラーが発生する可能性があります。

### 復元中に発生した`rpc error: code = Unavailable desc =...`エラーを処理するにはどうすればよいですか？ {#what-should-i-do-to-handle-the-rpc-error-code-unavailable-desc-error-occurred-in-restore}

このエラーは、復元するクラスタの容量が不足している場合に発生する可能性があります。このクラスタまたはTiKVログの監視メトリックを確認することで、原因をさらに確認できます。

この問題を処理するには、クラスタリソースをスケールアウトし、復元中の並行性を減らし、`RATE_LIMIT`オプションを有効にすることができます。

### 復元中にエラーメッセージ`the entry too large, the max entry size is 6291456, the size of data is 7690800`が表示された場合はどうすればよいですか？ {#what-should-i-do-if-the-restore-fails-with-the-error-message-the-entry-too-large-the-max-entry-size-is-6291456-the-size-of-data-is-7690800}

バッチで作成されるテーブルの数を減らすために、`--ddl-batch-size`を`128`またはそれより小さい値に設定することができます。

バックアップデータを[`--ddl-batch-size`](/br/br-batch-create-table.md#use-batch-create-table)が`1`より大きい値で使用している場合、TiDBはDDLジョブをTiKVが維持するDDLジョブキューに書き込みます。この時、TiDBが一度に送信するすべてのテーブルスキーマの合計サイズは6MBを超えてはいけません。なぜなら、ジョブメッセージの最大値はデフォルトで`6MB`であるためです（この値を変更することは**推奨されません**。詳細は[`txn-entry-size-limit`](/tidb-configuration-file.md#txn-entry-size-limit-new-in-v50)と[`raft-entry-max-size`](/tikv-configuration-file.md#raft-entry-max-size)を参照してください）。そのため、`--ddl-batch-size`を極端に大きな値に設定すると、一度にTiDBがバッチで送信するテーブルのスキーマサイズが指定された値を超え、BRが`entry too large, the max entry size is 6291456, the size of data is 7690800`エラーを報告することがあります。

### `local`ストレージを使用する場合、バックアップされたファイルはどこに保存されますか？ {#where-are-the-backed-up-files-stored-when-i-use-local-storage}

> **Note:**
>
> BRまたはTiKVノードにネットワークファイルシステム（NFS）がマウントされていない場合、またはAmazon S3、GCS、またはAzure Blob Storageプロトコルをサポートする外部ストレージを使用する場合、BRによってバックアップされたデータは各TiKVノードで生成されます。**BRを展開する推奨される方法ではありません**。なぜなら、バックアップデータが各ノードのローカルファイルシステムに分散しているため、バックアップデータを収集するとデータの冗長性や運用・保守上の問題が発生する可能性があるからです。また、バックアップデータを収集する前に直接データを復元すると、`SST file not found`エラーが発生します。

ローカルストレージを使用する場合、`backupmeta`はBRが実行されているノードで生成され、バックアップファイルは各リージョンのリーダーノードで生成されます。

### データを復元する際に`could not read local://...:download sst failed`エラーメッセージが返された場合、どうすればよいですか？ {#what-should-i-do-if-the-error-message-could-not-read-local-download-sst-failed-is-returned-during-data-restore}

データを復元する際、各ノードは**すべての**バックアップファイル（SSTファイル）にアクセスする必要があります。デフォルトでは、`local`ストレージを使用する場合、バックアップファイルが異なるノードに分散しているため、データを復元することはできません。そのため、各TiKVノードのバックアップファイルを他のTiKVノードにコピーする必要があります。**バックアップデータをAmazon S3、Google Cloud Storage（GCS）、Azure Blob Storage、またはNFSに保存することをお勧めします**。

### `root`で実行しても`Permission denied`または`No such file or directory`エラーが発生する場合はどうすればよいですか？ {#what-should-i-do-to-handle-the-permission-denied-or-no-such-file-or-directory-error-even-if-i-have-tried-to-run-br-using-root-in-vain}

データをバックアップするには、TiKVが書き込み権限を持っているかどうかを確認する必要があります。データを復元するには、読み取り権限があるかどうかを確認する必要があります。

バックアップ操作中、ストレージ媒体がローカルディスクまたはネットワークファイルシステム（NFS）の場合、`br`を起動するユーザーとTiKVを起動するユーザーが一致していることを確認してください（`br`とTiKVが異なるマシンにある場合、ユーザーのUIDは一致している必要があります）。そうでないと、`Permission denied`の問題が発生する可能性があります。

`br`を`root`ユーザーとして実行すると、バックアップファイル（SSTファイル）がTiKVによって保存されるため、ディスクの権限の問題で失敗する可能性があります。

> **Note:**
>
> データを復元する際に同じ問題に遭遇する可能性があります。SSTファイルが最初に読み取られるとき、読み取り権限が確認されます。DDLの実行時間から、権限を確認してから`br`を実行するまでの間隔が長い可能性があります。長時間待っても`Permission denied`エラーメッセージが表示される可能性があります。

そのため、次の手順に従ってデータを復元する前に権限を確認することをお勧めします：

1. Linuxコマンドを実行してプロセスをクエリする：

   ```bash
   ps aux | grep tikv-server
   ```

   出力は次のようになります：

   ```shell
   tidb_ouo  9235 10.9  3.8 2019248 622776 ?      Ssl  08:28   1:12 bin/tikv-server --addr 0.0.0.0:20162 --advertise-addr 172.16.6.118:20162 --status-addr 0.0.0.0:20188 --advertise-status-addr 172.16.6.118:20188 --pd 172.16.6.118:2379 --data-dir /home/user1/tidb-data/tikv-20162 --config conf/tikv.toml --log-file /home/user1/tidb-deploy/tikv-20162/log/tikv.log
   tidb_ouo  9236  9.8  3.8 2048940 631136 ?      Ssl  08:28   1:05 bin/tikv-server --addr 0.0.0.0:20161 --advertise-addr 172.16.6.118:20161 --status-addr 0.0.0.0:20189 --advertise-status-addr 172.16.6.118:20189 --pd 172.16.6.118:2379 --data-dir /home/user1/tidb-data/tikv-20161 --config conf/tikv.toml --log-file /home/user1/tidb-deploy/tikv-20161/log/tikv.log
   ```

   または、次のコマンドを実行できます：

   ```bash
   ps aux | grep tikv-server | awk '{print $1}'
   ```

   出力は次のようになります：

   ```shell
   tidb_ouo
   tidb_ouo
   ```

2. `tiup`コマンドを使用してクラスターの起動情報をクエリする：

   ```bash
   tiup cluster list
   ```

   出力は次のようになります：

   ```shell
   [root@Copy-of-VM-EE-CentOS76-v1 br]# tiup cluster list
   Starting component `cluster`: /root/.tiup/components/cluster/v1.5.2/tiup-cluster list
   Name          User      Version  Path                                               PrivateKey
   ----          ----      -------  ----                                               ----------
   tidb_cluster  tidb_ouo  v5.0.2   /root/.tiup/storage/cluster/clusters/tidb_cluster  /root/.tiup/storage/cluster/clusters/tidb_cluster/ssh/id_rsa
   ```

3. バックアップディレクトリの権限を確認します。たとえば、`backup`はバックアップデータの保存に使用されます：

   ```bash
   ls -al backup
   ```

   出力は次のようになります：

   ```shell
   [root@Copy-of-VM-EE-CentOS76-v1 user1]# ls -al backup
   total 0
   drwxr-xr-x  2 root root   6 Jun 28 17:48 .
   drwxr-xr-x 11 root root 310 Jul  4 10:35 ..
   ```

   ステップ2の出力から、`tikv-server`インスタンスがユーザー`tidb_ouo`によって起動されていることがわかります。しかし、ユーザー`tidb_ouo`には`backup`への書き込み権限がありません。そのため、バックアップに失敗します。

### `mysql`スキーマのテーブルはなぜ復元されないのですか？ {#why-are-tables-in-the-mysql-schema-not-restored}

BR v5.1.0以降、フルバックアップを実行すると、BRは\*\*`mysql`スキーマのテーブル**をバックアップします。BR v6.2.0以前では、デフォルトの設定では、BRはユーザーデータのみを復元しますが、**`mysql`スキーマのテーブル\*\*は復元しません。

`mysql`スキーマ内でユーザーが作成したテーブル（システムテーブルではない）を復元するには、[テーブルフィルター](/table-filter.md#syntax)を使用して明示的にテーブルを含めることができます。次の例は、BRが通常のリストアを実行するときに`mysql.usertable`テーブルを復元する方法を示しています。

```shell
br restore full -f '*.*' -f '!mysql.*' -f 'mysql.usertable' -s $external_storage_url --with-sys-table
```

前のコマンドでは、

- `-f '*.*'`はデフォルトのルールを上書きするために使用されます
- `-f '!mysql.*'`は、明示的に指定しない限り、BRが`mysql`内のテーブルをリストアしないように指示します。
- `-f 'mysql.usertable'`は、`mysql.usertable`をリストアする必要があることを示します。

`mysql.usertable`のみをリストアする必要がある場合は、次のコマンドを実行してください：

```shell
br restore full -f 'mysql.usertable' -s $external_storage_url --with-sys-table
```

注意してほしいのは、[テーブルフィルタ](/table-filter.md#syntax)を設定しても、**BRは以下のシステムテーブルをリストアしません**:

- 統計テーブル (`mysql.stat_*`)
- システム変数テーブル (`mysql.tidb`, `mysql.global_variables`)
- [その他のシステムテーブル](https://github.com/pingcap/tidb/blob/master/br/pkg/restore/systable_restore.go#L31)

## バックアップとリカバリについて知っておくべきその他のこと {#other-things-you-may-want-to-know-about-backup-and-restore}

### バックアップデータのサイズはどのくらいですか？バックアップのレプリカはありますか？ {#what-is-the-size-of-the-backup-data-are-there-replicas-of-the-backup}

データバックアップ中、バックアップファイルは各リージョンのリーダーノードに生成されます。バックアップのサイズはデータサイズと同じで、冗長なレプリカはありません。そのため、合計データサイズは、TiKVデータの合計数をレプリカ数で割ったものになります。

ただし、ローカルストレージからデータをリカバリする場合、レプリカ数はTiKVノードの数と同じになります。なぜなら、各TiKVはすべてのバックアップファイルにアクセスする必要があるためです。

### BRを使用してバックアップまたはリカバリを実行した後、モニタリングノードに表示されるディスク使用量が一致しないのはなぜですか？ {#why-is-the-disk-usage-shown-on-the-monitoring-node-inconsistent-after-backup-or-restore-using-br}

この不一致は、バックアップで使用されるデータ圧縮率がリカバリで使用されるデフォルトの圧縮率と異なるためです。チェックサムが成功した場合は、この問題を無視できます。

### BRがバックアップデータをリカバリした後、テーブルの統計情報を更新するために`ANALYZE`ステートメントを実行する必要がありますか？ {#after-br-restores-the-backup-data-do-i-need-to-execute-the-analyze-statement-on-the-table-to-update-the-statistics-of-tidb-on-the-tables-and-indexes}

BRは統計情報をバックアップしません（v4.0.9を除く）。そのため、バックアップデータをリカバリした後、`ANALYZE TABLE`を手動で実行するか、TiDBが自動的に`ANALYZE`を実行するのを待つ必要があります。

v4.0.9では、BRはデフォルトで統計情報をバックアップしますが、これはメモリを多く消費します。バックアッププロセスがうまくいくようにするため、v4.0.10以降では統計情報のバックアップはデフォルトで無効になっています。

テーブルで`ANALYZE`を実行しない場合、不正確な統計情報のためにTiDBが最適な実行計画を選択できなくなります。クエリのパフォーマンスが重要でない場合は、`ANALYZE`を無視できます。

### 単一クラスタのデータをリストアするために複数のリストアタスクを同時に開始できますか？ {#can-i-start-multiple-restore-tasks-at-the-same-time-to-restore-the-data-of-a-single-cluster}

単一クラスタのデータをリストアするために複数のリストアタスクを同時に開始することは**強く推奨されません**。その理由は次のとおりです。

- BRがデータをリストアする際、PDの一部のグローバル設定を変更します。そのため、データリストアのために同時に複数のリストアタスクを開始すると、これらの設定が誤って上書きされ、クラスタの状態が異常になる可能性があります。
- BRはデータをリストアするために多くのクラスタリソースを消費するため、実際には並列でリストアタスクを実行しても、リストア速度は限定的にしか向上しません。
- データリストアのために複数のリストアタスクを並列で実行するテストは行われていないため、成功を保証するものではありません。

### BRはテーブルの`SHARD_ROW_ID_BITS`および`PRE_SPLIT_REGIONS`情報をバックアップしますか？リストアされたテーブルには複数のリージョンがありますか？ {#does-br-back-up-the-shard-row-id-bits-and-pre-split-regions-information-of-a-table-does-the-restored-table-have-multiple-regions}

はい。BRはテーブルの[`SHARD_ROW_ID_BITS`および`PRE_SPLIT_REGIONS`](/sql-statements/sql-statement-split-region.md#pre_split_regions)情報をバックアップします。リストアされたテーブルのデータも複数のリージョンに分割されます。
