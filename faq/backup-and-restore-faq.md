---
title: Backup & Restore FAQs
summary: Learn about Frequently Asked Questions (FAQs) and the solutions of backup and restore.
---

# バックアップとリストアのFAQ {#backup-restore-faqs}

このドキュメントは、TiDB Backup & Restore（BR）のよくある質問（FAQ）とその解決策をリストします。

## 誤ってデータを削除または更新した後、データを迅速に回復するにはどうすればよいですか？ {#what-should-i-do-to-quickly-recover-data-after-mistakenly-deleting-or-updating-data}

TiDB v6.4.0では、フラッシュバック機能が導入されています。この機能を使用して、GC時間内に指定された時点までデータを迅速に回復できます。したがって、誤操作が発生した場合は、この機能を使用してデータを回復できます。詳細については、[フラッシュバッククラスタ](/sql-statements/sql-statement-flashback-cluster.md)および[フラッシュバックデータベース](/sql-statements/sql-statement-flashback-database.md)を参照してください。

## TiDB v5.4.0以降のバージョンでは、クラスターでバックアップタスクを実行すると、なぜバックアップタスクの速度が遅くなるのですか？ {#in-tidb-v5-4-0-and-later-versions-when-backup-tasks-are-performed-on-the-cluster-under-a-heavy-workload-why-does-the-speed-of-backup-tasks-become-slow}

TiDB v5.4.0から、BRはバックアップタスクのための自動チューニング機能を導入しています。v5.4.0以降のクラスターでは、この機能がデフォルトで有効になります。クラスターのワークロードが重い場合、この機能はバックアップタスクが使用するリソースを制限し、オンラインクラスターへの影響を軽減します。詳細については、[バックアップ自動チューニング](/br/br-auto-tune.md)を参照してください。

TiKVは、[動的に構成](/tikv-control.md#modify-the-tikv-configuration-dynamically)することができます。TiKVの構成項目[`backup.enable-auto-tune`](/tikv-configuration-file.md#enable-auto-tune-new-in-v540)を`false`に設定することで、自動チューニングを無効にすることができます。また、`backup.enable-auto-tune`を`true`に設定することで、自動チューニングを有効にすることができます。v5.3.xからv5.4.0以降のバージョンにアップグレードされたクラスターの場合、自動チューニング機能はデフォルトで無効になっています。手動で有効にする必要があります。

`tiup`を使用して自動チューニングを有効または無効にする方法については、[自動チューニングの使用](/br/br-auto-tune.md#use-auto-tune)を参照してください。

さらに、自動チューニングはバックアップタスクで使用されるスレッドのデフォルト数を減らします。詳細については、`backup.num-threads`]\(/tikv-configuration-file.md#num-threads-1)を参照してください。そのため、Grafanaダッシュボードでは、v5.4.0より前のバージョンよりも、バックアップタスクで使用される速度、CPU使用率、およびI/Oリソース利用率が低くなります。v5.4.0より前では、`backup.num-threads`のデフォルト値は`CPU * 0.75`であり、バックアップタスクで使用されるスレッド数は論理CPUコアの75%を占めます。最大値は`32`でした。v5.4.0から、この構成項目のデフォルト値は`CPU * 0.5`になり、最大値は`8`になります。

オフラインクラスターでバックアップタスクを実行する場合、`tikv-ctl`を使用して`backup.num-threads`の値を大きな数に変更してバックアップを高速化することができます。

## PITRの問題 {#pitr-issues}

### [PITR](/br/br-pitr-guide.md)と[クラスターフラッシュバック](/sql-statements/sql-statement-flashback-cluster.md)の違いは何ですか？ {#what-is-the-difference-between-pitr-br-br-pitr-guide-md-and-cluster-flashback-sql-statements-sql-statement-flashback-cluster-md}

ユースケースの観点から、PITRは通常、クラスターが完全にサービス外であるか、データが破損して他の解決策で回復できない場合に、クラスターのデータを指定された時点に復元するために使用されます。PITRを使用するには、データ回復用の新しいクラスターが必要です。クラスターフラッシュバック機能は、ユーザーの誤操作やその他の要因によって引き起こされたデータエラーシナリオに特に設計されており、クラスターのデータをその場で最新のタイムスタンプに復元することができます。

ほとんどの場合、フラッシュバックは、人為的なミスによるデータエラーの場合にはPITRよりも優れた回復ソリューションです。RPO（ほぼゼロ）およびRTOがはるかに短いためです。ただし、クラスターが完全に利用できない場合、フラッシュバックはこの時点で実行できないため、この場合はPITRがクラスターを回復する唯一の解決策です。したがって、PITRは常にデータベースのディザスタリカバリ戦略を開発する際に必要な解決策です。RPO（最大5分）およびRTOがフラッシュバックよりも長いにもかかわらずです。

### 上流データベースが物理インポートモードでTiDB Lightningを使用してデータをインポートすると、ログバックアップ機能が利用できなくなります。なぜですか？ {#when-the-upstream-database-imports-data-using-tidb-lightning-in-the-physical-import-mode-the-log-backup-feature-becomes-unavailable-why}

現在、ログバックアップ機能はTiDB Lightningに完全に適応されていません。そのため、TiDB Lightningの物理モードでインポートされたデータはログデータにバックアップできません。

ログバックアップタスクを作成する上流クラスターでは、TiDB Lightningの物理モードを使用しないでください。代わりに、TiDB Lightningの論理モードを使用できます。物理モードを使用する必要がある場合は、インポートが完了した後にスナップショットバックアップを実行して、その後にPITRをスナップショットバックアップの時点に復元できるようにします。

### インデックスの追加の加速機能はPITRと互換性がないのはなぜですか？ {#why-is-the-acceleration-of-adding-indexes-feature-incompatible-with-pitr}

問題：[#38045](https://github.com/pingcap/tidb/issues/38045)

現在、[インデックスの加速](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)機能で作成されたインデックスデータはPITRでバックアップできません。

そのため、PITRの回復が完了した後、BRはインデックスの加速で作成されたインデックスデータを削除し、その後再作成します。インデックスの加速で多数のインデックスが作成された場合やログバックアップ中にインデックスデータが大きい場合は、インデックスを作成した後にフルバックアップを実行することをお勧めします。

### クラスターはネットワーク分割障害から回復しましたが、ログバックアップタスクの進行状況のチェックポイントはまだ再開されていません。なぜですか？ {#the-cluster-has-recovered-from-the-network-partition-failure-but-the-checkpoint-of-the-log-backup-task-progress-still-does-not-resume-why}

問題：[#13126](https://github.com/tikv/tikv/issues/13126)

クラスターのネットワーク分割障害後、バックアップタスクはログのバックアップを続行できません。一定のリトライ時間後、タスクは`ERROR`状態に設定されます。この時点で、バックアップタスクは停止します。

この問題を解決するには、`br log resume`コマンドを手動で実行してログバックアップタスクを再開する必要があります。

### PITRを実行する際に`execute over region id`エラーが返された場合はどうすればよいですか？ {#what-should-i-do-if-the-error-execute-over-region-id-is-returned-when-i-perform-pitr}

問題：[#37207](https://github.com/pingcap/tidb/issues/37207)

この問題は、フルデータインポート中にログバックアップを有効にし、その後データインポート中の時点でデータを復元する場合に通常発生します。

具体的には、長時間（24時間など）にわたって大量のホットスポット書き込みがある場合（各TiKVノードのOPSが50k/sを超える場合）、この問題が発生する可能性があります（Grafanaでメトリクスを表示できます：**TiKV-Details** -> **Backup Log** -> **Handle Event Rate**）。

データインポート後にスナップショットバックアップを実行し、このスナップショットバックアップを基にPITRを実行することをお勧めします。

## `br restore point`コマンドを使用して下流クラスターを復元した後、TiFlashからデータにアクセスできません。どうすればよいですか？ {#after-restoring-a-downstream-cluster-using-the-br-restore-point-command-data-cannot-be-accessed-from-tiflash-what-should-i-do}

現在、PITRは復元フェーズ中にデータを直接TiFlashに書き込むことをサポートしていません。代わりに、brコマンドラインツールは`ALTER TABLE table_name SET TIFLASH REPLICA ***` DDLを実行してデータを複製します。そのため、PITRがデータの復元を完了した後、TiFlashレプリカはすぐには利用できません。代わりに、データがTiKVノードから複製されるまで、ある程度の時間を待つ必要があります。複製の進行状況を確認するには、`INFORMATION_SCHEMA.tiflash_replica`テーブルの`progress`情報を確認してください。

### ログバックアップタスクの`status`が`ERROR`になった場合はどうすればよいですか？ {#what-should-i-do-if-the-status-of-a-log-backup-task-becomes-error}

ログバックアップタスク中に、タスクのステータスが失敗し、リトライしても回復できない場合、タスクのステータスは`ERROR`になります。以下は例です：

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

この問題に対処するために、原因のエラーメッセージを確認し、指示通りに実行してください。問題が解決された後、次のコマンドを実行してタスクを再開します：

```shell
br log resume --task-name=task1 --pd x.x.x.x:2379
```

バックアップタスクが再開された後、`br log status`を使用してステータスを確認できます。タスクのステータスが`NORMAL`になると、バックアップタスクが継続されます。

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

> **Note:**
>
> この機能は、データの複数のバージョンをバックアップします。長時間のバックアップタスクが失敗し、ステータスが `ERROR` になると、このタスクのチェックポイントデータが `セーフポイント` として設定され、`セーフポイント` のデータは24時間以内にガベージコレクションされません。したがって、エラーを再開した後、バックアップタスクは最後のチェックポイントから続行されます。タスクが24時間以上失敗し、最後のチェックポイントデータがガベージコレクションされた場合、タスクを再開するとエラーが報告されます。この場合、`br log stop` コマンドを実行して最初にタスクを停止し、その後新しいバックアップタスクを開始するだけです。

### `br log resume` コマンドを使用して中断されたタスクを再開しようとすると、エラーメッセージ `ErrBackupGCSafepointExceeded` が返された場合、どうすればよいですか？ {#what-should-i-do-if-the-error-message-errbackupgcsafepointexceeded-is-returned-when-using-the-br-log-resume-command-to-resume-a-suspended-task}

```shell
Error: failed to check gc safePoint, checkpoint ts 433177834291200000: GC safepoint 433193092308795392 exceed TS 433177834291200000: [BR:Backup:ErrBackupGCSafepointExceeded]backup GC safepoint exceeded
```

ログバックアップタスクを一時停止した後、MVCCデータがガベージコレクションされないようにするために、一時停止タスクプログラムは現在のチェックポイントを自動的にサービスセーフポイントとして設定します。これにより、24時間以内に生成されたMVCCデータが保持されます。バックアップチェックポイントのMVCCデータが24時間を超える場合、チェックポイントのデータがガベージコレクションされ、バックアップタスクは再開できません。

この問題を解決するには、`br log stop`を使用して現在のタスクを削除し、`br log start`を使用してログバックアップタスクを作成します。同時に、後続のPITRのためにフルバックアップを実行できます。

## 機能の互換性の問題 {#feature-compatibility-issues}

### TiCDCまたはDrainerの上流クラスタに復元されたデータが、なぜレプリケートされないのですか？ {#why-does-data-restored-using-br-command-line-tool-cannot-be-replicated-to-the-upstream-cluster-of-ticdc-or-drainer}

- **BRを使用して復元されたデータは、下流にレプリケートできません**。これは、BRがSSTファイルを直接インポートするためであり、下流クラスタは現在、これらのファイルを上流から取得できないためです。

- v4.0.3より前では、復元中に生成されたDDLジョブは、TiCDC/Drainerで予期しないDDL実行を引き起こす可能性があります。したがって、TiCDC/Drainerの上流クラスタで復元を実行する必要がある場合は、brコマンドラインツールを使用して復元されたすべてのテーブルをTiCDC/Drainerのブロックリストに追加してください。

TiCDCのブロックリストを構成するには、[`filter.rules`](https://github.com/pingcap/tiflow/blob/7c3c2336f98153326912f3cf6ea2fbb7bcc4a20c/cmd/changefeed.toml#L16)を使用し、Drainerのブロックリストを構成するには、[`syncer.ignore-table`](/tidb-binlog/tidb-binlog-configuration-file.md#ignore-table)を使用します。

### 復元中に`new_collation_enabled`の不一致が報告された場合、どうすればよいですか？ {#why-is-new-collation-enabled-mismatch-reported-during-restore}

TiDB v6.0.0以降、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)のデフォルト値は`false`から`true`に変更されました。BRは、上流クラスタの`mysql.tidb`テーブルで`new_collation_enabled`構成をバックアップし、その後、この構成の値が上流と下流のクラスタで一貫しているかどうかをチェックします。値が一貫している場合、BRは安全に上流クラスタでバックアップされたデータを下流クラスタに復元します。値が一貫していない場合、BRはデータの復元を実行せず、エラーを報告します。

v6.0.0以前のTiDBクラスタのデータをバックアップし、このデータをv6.0.0以降のTiDBクラスタに復元する場合は、上流と下流のクラスタで`new_collations_enabled_on_first_bootstrap`の値が一貫しているかどうかを手動で確認する必要があります。

- 値が一貫している場合、復元コマンドに`--check-requirements=false`を追加して、この構成チェックをスキップできます。
- 値が一貫していない場合、強制的に復元を実行し、BRはデータ検証エラーを報告します。

### クラスタに配置ルールを復元する際にエラーが発生した場合、どうすればよいですか？ {#why-does-an-error-occur-when-i-restore-placement-rules-to-a-cluster}

v6.0.0以前では、BRは[配置ルール](/placement-rules-in-sql.md)をサポートしていませんでした。v6.0.0以降、BRは配置ルールをサポートし、配置ルールのバックアップおよび復元モードを制御するためのコマンドラインオプション`--with-tidb-placement-mode=strict/ignore`を導入しました。デフォルト値`strict`では、BRは配置ルールをインポートおよび検証しますが、値が`ignore`の場合はすべての配置ルールを無視します。

## データの復元に関する問題 {#data-restore-issues}

### `Io(Os...)`エラーを処理するにはどうすればよいですか？ {#what-should-i-do-to-handle-the-io-os-error}

これらの問題のほとんどは、TiKVがデータをディスクに書き込む際に発生するシステムコールエラーです。たとえば、`Io(Os {code: 13, kind: PermissionDenied...})`や`Io(Os {code: 2, kind: NotFound...})`などです。

このような問題に対処するには、まずバックアップディレクトリのマウント方法とファイルシステムを確認し、別のフォルダや別のハードディスクにデータをバックアップしてみてください。

たとえば、`samba`によって構築されたネットワークディスクにデータをバックアップする際に、`Code: 22(invalid argument)`エラーが発生することがあります。

### 復元中に`rpc error: code = Unavailable desc =...`エラーが発生した場合、どうすればよいですか？ {#what-should-i-do-to-handle-the-rpc-error-code-unavailable-desc-error-occurred-in-restore}

このエラーは、復元するクラスタの容量が不十分の場合に発生する可能性があります。このクラスタの監視メトリクスやTiKVログを確認することで、原因をさらに確認できます。

この問題に対処するためには、クラスタリソースをスケーリングアウトし、復元中の並行性を減らし、`RATE_LIMIT`オプションを有効にすることができます。

### 復元中に`the entry too large, the max entry size is 6291456, the size of data is 7690800`というエラーメッセージが表示された場合、どうすればよいですか？ {#what-should-i-do-if-the-restore-fails-with-the-error-message-the-entry-too-large-the-max-entry-size-is-6291456-the-size-of-data-is-7690800}

`--ddl-batch-size`を`128`またはそれ以下の値に設定することで、バッチで作成されるテーブルの数を減らすことができます。

[`--ddl-batch-size`](/br/br-batch-create-table.md#use-batch-create-table)の値が`1`より大きい場合、BRを使用してバックアップデータを復元する際に、TiDBはテーブル作成のDDLジョブをTiKVが維持するDDLジョブキューに書き込みます。この時、TiDBによって一度に送信されるすべてのテーブルスキーマの合計サイズは、デフォルトでジョブメッセージの最大値が`6 MB`であるため、`6 MB`を超えてはなりません（この値を変更することは**推奨されません**。詳細については、[`txn-entry-size-limit`](/tidb-configuration-file.md#txn-entry-size-limit-new-in-v50)および[`raft-entry-max-size`](/tikv-configuration-file.md#raft-entry-max-size)を参照してください）。したがって、`--ddl-batch-size`を過度に大きな値に設定すると、一度にTiDBによって送信されるテーブルのスキーマサイズが指定された値を超え、BRが`the entry too large, the max entry size is 6291456, the size of data is 7690800`エラーを報告します。

### `local`ストレージを使用してバックアップされたファイルはどこに保存されますか？ {#where-are-the-backed-up-files-stored-when-i-use-local-storage}

> **Note:**
>
> BRまたはTiKVノードにNFSがマウントされていない場合、またはAmazon S3、GCS、またはAzure Blob Storageプロトコルをサポートする外部ストレージを使用している場合、BRによってバックアップされたデータは各TiKVノードで生成されます。**BRを展開する推奨される方法ではありません**。なぜなら、バックアップデータが各ノードのローカルファイルシステムに散在しているため、データの冗長性や運用および保守の問題が発生する可能性があります。また、バックアップデータを収集する前にデータを直接復元すると、`SST file not found`エラーが発生します。

ローカルストレージを使用する場合、`backupmeta`はBRが実行されているノードに生成され、バックアップファイルは各リージョンのリーダーノードに生成されます。

### データを復元中に`could not read local://...:download sst failed`というエラーメッセージが返された場合、どうすればよいですか？ {#what-should-i-do-if-the-error-message-could-not-read-local-download-sst-failed-is-returned-during-data-restore}

データを復元する際、各ノードは**すべて**のバックアップファイル（SSTファイル）にアクセスできる必要があります。デフォルトでは、`local`ストレージを使用する場合、バックアップファイルが異なるノードに散在しているため、データを復元することができません。そのため、各TiKVノードのバックアップファイルを他のTiKVノードにコピーする必要があります。**Amazon S3、Google Cloud Storage（GCS）、Azure Blob Storage、またはNFS**にバックアップデータを保存することをお勧めします。

### `Permission denied` エラーまたは `No such file or directory` エラーを処理するにはどうすればよいですか？たとえ `root` で `br` を実行しようとしても、無駄になることがありますか？ {#what-should-i-do-to-handle-the-permission-denied-or-no-such-file-or-directory-error-even-if-i-have-tried-to-run-br-using-root-in-vain}

バックアップディレクトリに TiKV がアクセスできるかどうかを確認する必要があります。データをバックアップするには、TiKV が書き込み権限を持っているかどうかを確認してください。データを復元するには、読み取り権限があるかどうかを確認してください。

バックアップ操作中、ストレージメディアがローカルディスクまたはネットワークファイルシステム（NFS）の場合、`br` を開始するユーザーと TiKV を開始するユーザーが一致していることを確認してください（`br` と TiKV が異なるマシンにある場合、ユーザーの UID が一致している必要があります）。そうでないと、`Permission denied` の問題が発生する可能性があります。

`root` ユーザーとして `br` を実行すると、ディスクの権限のために失敗することがあります。なぜなら、バックアップファイル（SST ファイル）は TiKV によって保存されるからです。

> **Note:**
>
> データを復元する際にも同じ問題に遭遇する可能性があります。SST ファイルが初めて読み取られると、読み取り権限が検証されます。DDL の実行時間から、権限の確認と `br` の実行の間に長い間隔がある可能性があります。長い時間待ってから `Permission denied` エラーメッセージを受け取ることがあります。

そのため、次の手順に従ってデータを復元する前に権限を確認することをお勧めします：

1. プロセスクエリのための Linux コマンドを実行します：

   ```bash
   ps aux | grep tikv-server
   ```

   出力は次のようになります：

   ```shell
   tidb_ouo  9235 10.9  3.8 2019248 622776 ?      Ssl  08:28   1:12 bin/tikv-server --addr 0.0.0.0:20162 --advertise-addr 172.16.6.118:20162 --status-addr 0.0.0.0:20188 --advertise-status-addr 172.16.6.118:20188 --pd 172.16.6.118:2379 --data-dir /home/user1/tidb-data/tikv-20162 --config conf/tikv.toml --log-file /home/user1/tidb-deploy/tikv-20162/log/tikv.log
   tidb_ouo  9236  9.8  3.8 2048940 631136 ?      Ssl  08:28   1:05 bin/tikv-server --addr 0.0.0.0:20161 --advertise-addr 172.16.6.118:20161 --status-addr 0.0.0.0:20189 --advertise-status-addr 172.16.6.118:20189 --pd 172.16.6.118:2379 --data-dir /home/user1/tidb-data/tikv-20161 --config conf/tikv.toml --log-file /home/user1/tidb-deploy/tikv-20161/log/tikv.log
   ```

   または次のコマンドを実行できます：

   ```bash
   ps aux | grep tikv-server | awk '{print $1}'
   ```

   出力は次のようになります：

   ```shell
   tidb_ouo
   tidb_ouo
   ```

2. `tiup` コマンドを使用してクラスターの起動情報をクエリします：

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

3. バックアップディレクトリの権限を確認します。たとえば、`backup` はバックアップデータの保存に使用されます：

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

   ステップ 2 の出力から、`tikv-server` インスタンスがユーザー `tidb_ouo` によって開始されていることがわかります。しかし、ユーザー `tidb_ouo` は `backup` に対して書き込み権限を持っていません。そのため、バックアップに失敗します。

### `mysql` スキーマのテーブルが復元されないのはなぜですか？ {#why-are-tables-in-the-mysql-schema-not-restored}

BR v5.1.0 以降、フルバックアップを実行すると、BR は **`mysql` スキーマのテーブル** をバックアップします。BR v6.2.0 以前では、デフォルトの構成では、BR はユーザーデータのみを復元しますが、**`mysql` スキーマ** のテーブルは復元しません。

`mysql` スキーマ内でユーザーによって作成されたテーブル（システムテーブルではない）を復元するには、[テーブルフィルタ](/table-filter.md#syntax)を使用してテーブルを明示的に含めることができます。次の例は、BR が通常の復元を実行する際に `mysql.usertable` テーブルを復元する方法を示しています。

```shell
br restore full -f '*.*' -f '!mysql.*' -f 'mysql.usertable' -s $external_storage_url --with-sys-table
```

先行するコマンドでは、

- `-f '*.*'` はデフォルトのルールを上書きするために使用されます
- `-f '!mysql.*'` は、BRに`mysql`内のテーブルを明示的に指定しない限り、復元しないように指示します。
- `-f 'mysql.usertable'` は、`mysql.usertable`を復元する必要があることを示します。

`mysql.usertable`のみを復元する必要がある場合は、次のコマンドを実行してください：

```shell
br restore full -f 'mysql.usertable' -s $external_storage_url --with-sys-table
```

注意してください。[テーブルフィルタ](/table-filter.md#syntax)を構成しても、**BRは次のシステムテーブルを復元しません**。

- 統計テーブル（`mysql.stat_*`）
- システム変数テーブル（`mysql.tidb`、`mysql.global_variables`）
- [その他のシステムテーブル](https://github.com/pingcap/tidb/blob/master/br/pkg/restore/systable_restore.go#L31)

## バックアップとリストアについて知っておくべき他のこと {#other-things-you-may-want-to-know-about-backup-and-restore}

### バックアップデータのサイズはどのくらいですか？バックアップにレプリカはありますか？ {#what-is-the-size-of-the-backup-data-are-there-replicas-of-the-backup}

データバックアップ中、バックアップファイルは各リージョンのリーダーノードに生成されます。バックアップのサイズはデータサイズと同じであり、冗長なレプリカはありません。したがって、合計データサイズは、レプリカの数でTiKVデータの合計数を割ったものとなります。

ただし、ローカルストレージからデータをリストアする場合、レプリカの数はTiKVノードの数と同じであり、各TiKVはすべてのバックアップファイルにアクセスする必要があります。

### BRを使用してバックアップまたはリストア後に監視ノードに表示されるディスク使用量が一貫しないのはなぜですか？ {#why-is-the-disk-usage-shown-on-the-monitoring-node-inconsistent-after-backup-or-restore-using-br}

この不一致は、バックアップで使用されるデータ圧縮率がリストアで使用されるデフォルトの率と異なるためです。チェックサムが成功した場合、この問題は無視できます。

### BRがバックアップデータをリストアした後、テーブルの統計情報を更新するために`ANALYZE`ステートメントを実行する必要がありますか？ {#after-br-restores-the-backup-data-do-i-need-to-execute-the-analyze-statement-on-the-table-to-update-the-statistics-of-tidb-on-the-tables-and-indexes}

BRは統計情報をバックアップしません（v4.0.9を除く）。したがって、バックアップデータをリストアした後、`ANALYZE TABLE`を手動で実行するか、TiDBが自動的に`ANALYZE`を実行するのを待つ必要があります。

v4.0.9では、BRはデフォルトで統計情報をバックアップしますが、これにはあまりにも多くのメモリが必要です。バックアッププロセスがうまくいくようにするために、v4.0.10からは統計情報のバックアップがデフォルトで無効になっています。

テーブルで`ANALYZE`を実行しない場合、不正確な統計情報のためにTiDBは最適な実行プランを選択できません。クエリのパフォーマンスが重要でない場合は、`ANALYZE`を無視できます。

### 単一クラスタのデータをリストアするために同時に複数のリストアタスクを開始できますか？ {#can-i-start-multiple-restore-tasks-at-the-same-time-to-restore-the-data-of-a-single-cluster}

**強くお勧めしません**。単一クラスタのデータをリストアするために同時に複数のリストアタスクを開始することは、次の理由から強くお勧めしません。

- BRがデータをリストアする際、PDの一部のグローバル構成を変更します。したがって、同時に複数のリストアタスクを開始すると、これらの構成が誤って上書きされ、クラスタの状態が異常になる可能性があります。
- BRはデータをリストアするために多くのクラスタリソースを消費するため、実際には、並列でリストアタスクを実行することは、リストア速度を限定的に向上させるだけです。
- データをリストアするために複数のリストアタスクを並行して実行するテストは行われていないため、成功が保証されていません。

### BRはテーブルの`SHARD_ROW_ID_BITS`および`PRE_SPLIT_REGIONS`情報をバックアップしますか？リストアされたテーブルには複数のリージョンがありますか？ {#does-br-back-up-the-shard-row-id-bits-and-pre-split-regions-information-of-a-table-does-the-restored-table-have-multiple-regions}

はい。BRはテーブルの[`SHARD_ROW_ID_BITS`および`PRE_SPLIT_REGIONS`](/sql-statements/sql-statement-split-region.md#pre_split_regions)情報をバックアップします。リストアされたテーブルのデータも複数のリージョンに分割されます。
