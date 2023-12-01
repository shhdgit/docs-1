---
title: Backup & Restore FAQs
summary: Learn about Frequently Asked Questions (FAQs) and the solutions of backup and restore.
---

# バックアップとリストアのFAQ {#backup-restore-faqs}

このドキュメントは、TiDB Backup & Restore（BR）のよくある質問（FAQ）とその解決策をリストしています。

## 誤ってデータを削除したり更新した後、データを迅速に回復するためにはどうすればよいですか？ {#what-should-i-do-to-quickly-recover-data-after-mistakenly-deleting-or-updating-data}

TiDB v6.4.0では、フラッシュバック機能が導入されています。この機能を使用して、GC時間内でデータを指定した時点に迅速に回復することができます。したがって、誤操作が発生した場合、この機能を使用してデータを回復することができます。詳細については、[フラッシュバッククラスタ](/sql-statements/sql-statement-flashback-to-timestamp.md)および[フラッシュバックデータベース](/sql-statements/sql-statement-flashback-database.md)を参照してください。

## TiDB v5.4.0以降のバージョンでは、クラスタでバックアップタスクを実行する際に、なぜバックアップタスクの速度が遅くなるのですか？ {#in-tidb-v5-4-0-and-later-versions-when-backup-tasks-are-performed-on-the-cluster-under-a-heavy-workload-why-does-the-speed-of-backup-tasks-become-slow}

TiDB v5.4.0から、BRはバックアップタスクのための自動調整機能を導入しています。v5.4.0以降のクラスタでは、この機能がデフォルトで有効になっています。クラスタのワークロードが重い場合、この機能はバックアップタスクによるオンラインクラスタへの影響を軽減するために使用されます。詳細については、[バックアップ自動調整](/br/br-auto-tune.md)を参照してください。

TiKVは[動的に構成する](/tikv-control.md#modify-the-tikv-configuration-dynamically)ことができる自動調整機能をサポートしています。TiKV-ctlを使用して、クラスタを再起動することなく、次の方法でこの機能を有効または無効にすることができます。

-   自動調整を無効にする：TiKVの構成項目[`backup.enable-auto-tune`](/tikv-configuration-file.md#enable-auto-tune-new-in-v540)を`false`に設定します。
-   自動調整を有効にする：`backup.enable-auto-tune`を`true`に設定します。v5.3.xからv5.4.0またはそれ以降のバージョンにアップグレードされたクラスタでは、自動調整機能はデフォルトで無効になっています。手動で有効にする必要があります。

自動調整により、バックアップタスクで使用されるスレッドのデフォルト数が減少します。詳細については、`backup.num-threads`]\(/tikv-configuration-file.md#num-threads-1)を参照してください。そのため、Grafanaダッシュボードでは、v5.4.0より前のバージョンよりも、バックアップタスクによって使用される速度、CPU使用率、およびI/Oリソース利用率が低くなります。v5.4.0より前では、`backup.num-threads`のデフォルト値は`CPU * 0.75`であり、つまり、バックアップタスクで使用されるスレッドの数は論理CPUコアの75%を占めます。その最大値は`32`でした。v5.4.0から、この構成項目のデフォルト値は`CPU * 0.5`になり、その最大値は`8`になります。

オフラインクラスタでバックアップタスクを実行する場合、`tikv-ctl`を使用して`backup.num-threads`の値を大きな数に変更してバックアップを高速化することができます。

## PITRの問題 {#pitr-issues}

### [PITR](/br/br-pitr-guide.md)と[クラスタフラッシュバック](/sql-statements/sql-statement-flashback-to-timestamp.md)の違いは何ですか？ {#what-is-the-difference-between-pitr-br-br-pitr-guide-md-and-cluster-flashback-sql-statements-sql-statement-flashback-to-timestamp-md}

ユースケースの観点から、PITRは通常、クラスタが完全にサービス外であるか、データが破損して他の解決策では回復できない場合に、クラスタのデータを指定した時点に復元するために使用されます。PITRを使用するには、データ復旧用の新しいクラスタが必要です。クラスタフラッシュバック機能は、ユーザーの誤操作やその他の要因によって引き起こされるデータエラーのシナリオに特に設計されており、クラスタのデータをそのデータエラーが発生する前の最新のタイムスタンプにインプレースで復元することができます。

ほとんどの場合、フラッシュバックは、人為的なミスによるデータエラーのための回復ソリューションとして、RPO（ほぼゼロに近い）およびRTOがはるかに短いため、PITRよりも優れています。ただし、クラスタが完全に利用できない場合、フラッシュバックはこの時点で実行できないため、この場合はPITRがクラスタを回復する唯一の解決策です。したがって、PITRは常にデータベースの災害復旧戦略を開発する際に必要なソリューションです。それにもかかわらず、PITRはRPO（最大5分）およびRTOがフラッシュバックよりも長いため、フラッシュバックよりも長いRPO（最大5分）およびRTOを持っています。

### 上流データベースがTiDB Lightningを物理インポートモードでデータをインポートすると、ログバックアップ機能が利用できなくなります。なぜですか？ {#when-the-upstream-database-imports-data-using-tidb-lightning-in-the-physical-import-mode-the-log-backup-feature-becomes-unavailable-why}

現在、ログバックアップ機能はTiDB Lightningに完全に適応されていません。そのため、TiDB Lightningの物理モードでインポートされたデータはログデータにバックアップすることができません。

アップストリームクラスターでログバックアップタスクを作成する場合、TiDB Lightningの物理モードを使用しないでデータをインポートすることを避けてください。代わりに、TiDB Lightningの論理モードを使用することができます。物理モードを使用する必要がある場合は、インポートが完了した後にスナップショットバックアップを実行してください。これにより、スナップショットバックアップの後の時点でPITRを復元することができます。

### インデックスの追加機能の高速化がPITRと互換性がないのはなぜですか？ {#why-is-the-acceleration-of-adding-indexes-feature-incompatible-with-pitr}

問題: [#38045](https://github.com/pingcap/tidb/issues/38045)

現在、[インデックスの高速化](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)機能を介して作成されたインデックスデータはPITRでバックアップすることができません。

そのため、PITRの復旧が完了した後、BRはインデックスの高速化によって作成されたインデックスデータを削除し、その後再作成します。インデックスの高速化によって多くのインデックスが作成された場合やログバックアップ中にインデックスデータが大きい場合は、インデックスを作成した後にフルバックアップを実行することをお勧めします。

### クラスターはネットワーク分割障害から回復しましたが、ログバックアップタスクの進行状況のチェックポイントはまだ再開されません。なぜですか？ {#the-cluster-has-recovered-from-the-network-partition-failure-but-the-checkpoint-of-the-log-backup-task-progress-still-does-not-resume-why}

問題: [#13126](https://github.com/tikv/tikv/issues/13126)

クラスターでネットワーク分割障害が発生した後、バックアップタスクはログのバックアップを続行することができません。一定のリトライ時間後、タスクは`ERROR`状態に設定されます。この時点で、バックアップタスクは停止します。

この問題を解決するには、ログバックアップタスクを再開するために`br log resume`コマンドを手動で実行する必要があります。

### PITRを実行する際に`execute over region id`エラーが返された場合、どうすればよいですか？ {#what-should-i-do-if-the-error-execute-over-region-id-is-returned-when-i-perform-pitr}

問題: [#37207](https://github.com/pingcap/tidb/issues/37207)

この問題は通常、フルデータのインポート中にログバックアップを有効にし、その後データのインポート中の時点でデータを復元するためにPITRを実行した場合に発生します。

具体的には、長時間（24時間など）にわたって大量のホットスポット書き込みがある場合（各TiKVノードのOPSが50k/sを超える場合）、この問題が発生する可能性があります（Grafanaでメトリクスを表示できます: **TiKV-Details** -> **Backup Log** -> **Handle Event Rate**）。

データのインポート後にスナップショットバックアップを実行し、このスナップショットバックアップに基づいてPITRを実行することをお勧めします。

## `br restore point`コマンドを使用してダウンストリームクラスターを復元した後、TiFlashからデータにアクセスできません。どうすればよいですか？ {#after-restoring-a-downstream-cluster-using-the-br-restore-point-command-data-cannot-be-accessed-from-tiflash-what-should-i-do}

現在、PITRはデータの復元中に直接データをTiFlashに書き込むことをサポートしていません。代わりに、brコマンドラインツールは`ALTER TABLE table_name SET TIFLASH REPLICA ***` DDLを実行してデータをレプリケートします。そのため、PITRがデータの復元を完了した後、TiFlashレプリカはすぐに利用できるわけではありません。代わりに、TiKVノードからデータがレプリケートされるまで一定の時間を待つ必要があります。レプリケーションの進行状況を確認するには、`INFORMATION_SCHEMA.tiflash_replica`テーブルの`progress`情報を確認してください。

### ログバックアップタスクの`status`が`ERROR`になった場合、どうすればよいですか？ {#what-should-i-do-if-the-status-of-a-log-backup-task-becomes-error}

ログバックアップタスク中に、タスクの状態が失敗し、リトライしても回復できない場合、タスクの状態は`ERROR`になります。以下はその例です:

```shell
br log status --pd x.x.x.x:2379

● 合計1つのタスク。
> #1 <
                    名前: task1
                  状態: ○ エラー
                   開始: 2022-07-25 13:49:02.868 +0000
                     終了: 2090-11-18 14:07:45.624 +0000
                 ストレージ: s3://tmp/br-log-backup0ef49055-5198-4be3-beab-d382a2189efb/Log
             速度(推定): 0.00 ops/s
      チェックポイント[グローバル]: 2022-07-25 14:46:50.118 +0000; ギャップ=11時間31分29秒
          エラー[ストア=1]: KV:LogBackup:RaftReq
エラー発生時[ストア=1]: 2022-07-25 14:54:44.467 +0000; ギャップ=11時間23分35秒
  エラーメッセージ[ストア=1]: リトライ時間が超過: 初期スナップショットの取得に失敗しました: 初期スナップショットの取得に失敗しました (region_id = 94812): ラフトストアのリクエスト中にエラーが発生しました: メッセージ: "リードインデックスが準備できていません、理由はマージのためにインデックスを読むことができません、リージョン94812" read_index_not_ready { reason: "can not read index due to merge" region_id: 94812 }: 初期スナップショットの取得に失敗しました: 初期スナップショットの取得に失敗しました (region_id = 94812): ラフトストアのリクエスト中にエラーが発生しました: メッセージ: "リードインデックスが準備できていません、理由はマージのためにインデックスを読むことができません、リージョン94812" read_index_not_ready { reason: "can not read index due to merge" region_id: 94812 }: 初期スナップショットの取得に失敗しました: 初期スナップショットの取得に失敗しました (region_id = 94812): ラフトストアのリクエスト中にエラーが発生しました: メッセージ: "リードインデックスが準備できていません、理由はマージのためにインデックスを読むことができません、リージョン94812" read_index_not_ready { reason: "can not read index due to merge" region_id: 94812 }
```

```shell
br log resume --task-name=task1 --pd x.x.x.x:2379
```

バックアップタスクが再開された後、`br log status` を使用して状態を確認できます。タスクの状態が `NORMAL` になると、バックアップタスクが継続されます。

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

**注意:**

この機能はデータの複数バージョンをバックアップします。長時間のバックアップタスクが失敗し、ステータスが `ERROR` になった場合、このタスクのチェックポイントデータは `セーフポイント` として設定され、`セーフポイント` のデータは24時間以内にガベージコレクションされません。したがって、エラーが発生した後にバックアップタスクを再開すると、最後のチェックポイントからバックアップタスクが継続されます。もしタスクが24時間以上失敗し、最後のチェックポイントデータがガベージコレクションされている場合、タスクを再開する際にエラーが報告されます。この場合、`br log stop` コマンドを実行してタスクを一旦停止し、新しいバックアップタスクを開始するしかありません。

### 中断されたタスクを再開する際に `br log resume` コマンドを使用した際に `ErrBackupGCSafepointExceeded` エラーメッセージが返された場合、どのように対処すればよいですか？ {#what-should-i-do-if-the-error-message-errbackupgcsafepointexceeded-is-returned-when-using-the-br-log-resume-command-to-resume-a-suspended-task}

```shell
Error: failed to check gc safePoint, checkpoint ts 433177834291200000: GC safepoint 433193092308795392 exceed TS 433177834291200000: [BR:Backup:ErrBackupGCSafepointExceeded]backup GC safepoint exceeded
```

ログバックアップタスクを一時停止した後、MVCCデータがガベージコレクションされないようにするために、一時停止タスクプログラムは現在のチェックポイントを自動的にサービスセーフポイントとして設定します。これにより、24時間以内に生成されたMVCCデータが保持されます。バックアップチェックポイントのMVCCデータが24時間を超えて生成されている場合、そのチェックポイントのデータはガベージコレクションされ、バックアップタスクは再開できません。

この問題を解決するためには、`br log stop`を使用して現在のタスクを削除し、`br log start`を使用してログバックアップタスクを作成します。同時に、後続のPITRのためにフルバックアップを実行することができます。

## 機能の互換性の問題 {#feature-compatibility-issues}

### なぜbrコマンドラインツールを使用して復元されたデータをTiCDCやDrainerの上流クラスタにレプリケートできないのですか？ {#why-does-data-restored-using-br-command-line-tool-cannot-be-replicated-to-the-upstream-cluster-of-ticdc-or-drainer}

-   **BRを使用して復元されたデータは下流にレプリケートできません**。これはBRがSSTファイルを直接インポートするためであり、下流クラスタは現在これらのファイルを上流から取得できないためです。

-   v4.0.3より前では、復元中に生成されたDDLジョブがTiCDC/Drainerで予期しないDDL実行を引き起こす可能性があります。したがって、TiCDC/Drainerの上流クラスタで復元を実行する必要がある場合は、brコマンドラインツールを使用して復元されたすべてのテーブルをTiCDC/Drainerのブロックリストに追加してください。

TiCDCのブロックリストを構成するには[`filter.rules`](https://github.com/pingcap/tiflow/blob/7c3c2336f98153326912f3cf6ea2fbb7bcc4a20c/cmd/changefeed.toml#L16)を使用し、Drainerのブロックリストを構成するには[`syncer.ignore-table`](/tidb-binlog/tidb-binlog-configuration-file.md#ignore-table)を使用します。

### 復元中に`new_collation_enabled`の不一致が報告されるのはなぜですか？ {#why-is-new-collation-enabled-mismatch-reported-during-restore}

TiDB v6.0.0以降、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)のデフォルト値が`false`から`true`に変更されました。BRは上流クラスタの`mysql.tidb`テーブルに`new_collation_enabled`構成をバックアップし、その後この構成の値が上流と下流のクラスタで一致しているかどうかをチェックします。値が一致する場合、BRは安全に上流クラスタでバックアップされたデータを下流クラスタに復元します。値が一致しない場合、BRはデータの復元を実行せずにエラーを報告します。

v6.0.0以前のTiDBクラスタでデータをバックアップし、このデータをv6.0.0以降のTiDBクラスタに復元したい場合は、手動で`new_collations_enabled_on_first_bootstrap`の値が上流と下流のクラスタで一致しているかどうかを確認する必要があります。

-   値が一致する場合、復元コマンドに`--check-requirements=false`を追加してこの構成チェックをスキップできます。
-   値が一致しない場合、強制的に復元を実行しようとすると、BRはデータ検証エラーを報告します。

### クラスタに配置ルールを復元しようとするとエラーが発生するのはなぜですか？ {#why-does-an-error-occur-when-i-restore-placement-rules-to-a-cluster}

v6.0.0以前では、BRは[配置ルール](/placement-rules-in-sql.md)をサポートしていませんでした。v6.0.0以降、BRは配置ルールをサポートし、配置ルールのバックアップと復元モードを制御するためのコマンドラインオプション`--with-tidb-placement-mode=strict/ignore`を導入しました。デフォルト値が`strict`の場合、BRは配置ルールをインポートして検証しますが、値が`ignore`の場合はすべての配置ルールを無視します。

## データの復元に関する問題 {#data-restore-issues}

### `Io(Os...)`エラーを処理するにはどうすればよいですか？ {#what-should-i-do-to-handle-the-io-os-error}

これらの問題のほとんどは、TiKVがデータをディスクに書き込む際に発生するシステムコールエラーです。たとえば、`Io(Os {code: 13, kind: PermissionDenied...})`や`Io(Os {code: 2, kind: NotFound...})`などです。

このような問題に対処するためには、まずバックアップディレクトリのマウント方法とファイルシステムをチェックし、別のフォルダや別のハードディスクにデータをバックアップしてみてください。

たとえば、`samba`で構築されたネットワークディスクにデータをバックアップする際に`Code: 22(invalid argument)`エラーが発生することがあります。

### 復元中に`rpc error: code = Unavailable desc =...`エラーが発生した場合はどうすればよいですか？ {#what-should-i-do-to-handle-the-rpc-error-code-unavailable-desc-error-occurred-in-restore}

このエラーは、復元するクラスタの容量が不足している場合に発生する可能性があります。このクラスタの監視メトリクスやTiKVログをチェックして原因をさらに確認できます。

この問页を処理するためには、クラスタリソースをスケールアウトし、復元中の並行性を減らし、`RATE_LIMIT`オプションを有効にすることができます。

### エラーメッセージ`the entry too large, the max entry size is 6291456, the size of data is 7690800`で復元に失敗した場合はどうすればよいですか？ {#what-should-i-do-if-the-restore-fails-with-the-error-message-the-entry-too-large-the-max-entry-size-is-6291456-the-size-of-data-is-7690800}

`--ddl-batch-size`を`128`またはそれ以下の値に設定することで、バッチで作成されるテーブルの数を減らすことができます。

`--ddl-batch-size`の値が`1`より大きい場合、BRを使用してバックアップデータを復元する際に、TiDBはテーブルの作成のDDLジョブをTiKVが維持するDDLジョブキューに書き込みます。この時、TiDBによって一度に送信されるすべてのテーブルスキーマの合計サイズは、デフォルトでジョブメッセージの最大値が`6 MB`であるため、`6 MB`を超えてはならない（この値を変更することは**推奨されていません**。詳細は[`txn-entry-size-limit`](/tidb-configuration-file.md#txn-entry-size-limit-new-in-v50)および[`raft-entry-max-size`](/tikv-configuration-file.md#raft-entry-max-size)を参照）。したがって、`--ddl-batch-size`を過度に大きな値に設定すると、一度にTiDBによって送信されるテーブルのスキーマサイズが指定された値を超えるため、BRは`entry too large, the max entry size is 6291456, the size of data is 7690800`エラーを報告します。

### `local`ストレージを使用する場合、バックアップファイルはどこに保存されますか？ {#where-are-the-backed-up-files-stored-when-i-use-local-storage}

> **注意:**
>
> BRまたはTiKVノードにNFSがマウントされていない場合、またはAmazon S3、GCS、またはAzure Blob Storageプロトコルをサポートする外部ストレージを使用している場合、BRによってバックアップされたデータは各TiKVノードで生成されます。**これはBRを展開する推奨される方法ではありません**、なぜならバックアップデータが各ノードのローカルファイルシステムに散在しているため、データの冗長性や運用・保守上の問題が発生する可能性があります。また、バックアップデータを収集する前にデータを直接復元すると、`SST file not found`エラーが発生します。

`local`ストレージを使用する場合、`backupmeta`はBRが実行されているノードで生成され、バックアップファイルは各リージョンのリーダーノードで生成されます。

### データの復元中に`could not read local://...:download sst failed`エラーメッセージが返された場合はどうすればよいですか？ {#what-should-i-do-if-the-error-message-could-not-read-local-download-sst-failed-is-returned-during-data-restore}

データを復元する際、各ノードは**すべて**のバックアップファイル（SSTファイル）にアクセスする必要があります。デフォルトでは、`local`ストレージを使用する場合、バックアップファイルは異なるノードに散在しているため、データを復元することができません。したがって、各TiKVノードのバックアップファイルを他のTiKVノードにコピーする必要があります。**Amazon S3、Google Cloud Storage（GCS）、Azure Blob Storage、またはNFSにバックアップデータを保存することをお勧めします**。

### `root`で`br`を実行しても`Permission denied`または`No such file or directory`エラーが発生した場合はどうすればよいですか？ {#what-should-i-do-to-handle-the-permission-denied-or-no-such-file-or-directory-error-even-if-i-have-tried-to-run-br-using-root-in-vain}

バックアップディレクトリへのTiKVのアクセス権限を確認する必要があります。データをバックアップする際には、TiKVが書き込み権限を持っているかを確認してください。データを復元する際には、読み取り権限を持っているかを確認してください。

バックアップ操作中、ストレージメディアがローカルディスクまたはネットワークファイルシステム（NFS）の場合、`br`を開始するユーザーとTiKVを開始するユーザーが一致していることを確認してください（`br`とTiKVが異なるマシンにある場合、ユーザーのUIDが一致している必要があります）。そうでない場合、`Permission denied`の問題が発生する可能性があります。

`root`ユーザーとして`br`を実行しても、ディスクの権限の問題により失敗することがあります。なぜなら、バックアップファイル（SSTファイル）はTiKVによって保存されるためです。

> **注意:**
>
> データの復元中に同じ問題に遭遇することがあります。SSTファイルが初めて読み込まれる際に、読み取り権限が検証されます。DDLの実行時間から推測すると、権限の確認と`br`の実行の間に長い間隔がある可能性があります。長い時間待ってから`Permission denied`エラーメッセージを受け取ることがあります。

そのため、データの復元前に権限を確認することをお勧めします。以下の手順に従ってデータの復元前に権限を確認してください：

1.  プロセスクエリのためのLinuxコマンドを実行します：

    ```bash
    ps aux | grep tikv-server
    ```

    出力は以下の通りです：

    ```shell
    tidb_ouo  9235 10.9  3.8 2019248 622776 ?      Ssl  08:28   1:12 bin/tikv-server --addr 0.0.0.0:20162 --advertise-addr 172.16.6.118:20162 --status-addr 0.0.0.0:20188 --advertise-status-addr 172.16.6.118:20188 --pd 172.16.6.118:2379 --data-dir /home/user1/tidb-data/tikv-20162 --config conf/tikv.toml --log-file /home/user1/tidb-deploy/tikv-20162/log/tikv.log
    tidb_ouo  9236  9.8  3.8 2048940 631136 ?      Ssl  08:28   1:05 bin/tikv-server --addr 0.0.0.0:20161 --advertise-addr 172.16.6.118:20161 --status-addr 0.0.0.0:20189 --advertise-status-addr 172.16.6.118:20189 --pd 172.16.6.118:2379 --data-dir /home/user1/tidb-data/tikv-20161 --config conf/tikv.toml --log-file /home/user1/tidb-deploy/tikv-20161/log/tikv.log
    ```

    または、以下のコマンドを実行することもできます：

    ```bash
    ps aux | grep tikv-server | awk '{print $1}'
    ```

    出力は以下の通りです：

    ```shell
    tidb_ouo
    tidb_ouo
    ```

2.  `tiup`コマンドを使用してクラスターの起動情報をクエリします：

    ```bash
    tiup cluster list
    ```

    出力は以下の通りです：

    ```shell
    [root@Copy-of-VM-EE-CentOS76-v1 br]# tiup cluster list
    Starting component `cluster`: /root/.tiup/components/cluster/v1.5.2/tiup-cluster list
    Name          User      Version  Path                                               PrivateKey
    ----          ----      -------  ----                                               ----------
    tidb_cluster  tidb_ouo  v5.0.2   /root/.tiup/storage/cluster/clusters/tidb_cluster  /root/.tiup/storage/cluster/clusters/tidb_cluster/ssh/id_rsa
    ```

3.  バックアップディレクトリの権限を確認します。例えば、`backup`はバックアップデータの保存先です：

    ```bash
    ls -al backup
    ```

    出力は以下の通りです：

    ```shell
    [root@Copy-of-VM-EE-CentOS76-v1 user1]# ls -al backup
    total 0
    drwxr-xr-x  2 root root   6 Jun 28 17:48 .
    drwxr-xr-x 11 root root 310 Jul  4 10:35 ..
    ```

    ステップ2の出力から、`tikv-server`インスタンスがユーザー`tidb_ouo`によって起動されていることがわかります。しかし、ユーザー`tidb_ouo`には`backup`への書き込み権限がありません。そのため、バックアップが失敗します。

### `mysql`スキーマのテーブルが復元されない理由 {#why-are-tables-in-the-mysql-schema-not-restored}

BR v5.1.0から、フルバックアップを実行すると、BRは\*\*`mysql`スキーマのテーブル**をバックアップします。BR v6.2.0以前では、デフォルトの構成では、BRはユーザーデータを復元しますが、**`mysql`スキーマのテーブル\*\*を復元しません。

`mysql`スキーマ内でユーザーが作成したテーブルを復元するには、[テーブルフィルタ](/table-filter.md#syntax)を使用してテーブルを明示的に含めることができます。次の例は、BRが通常の復元を実行する際に、`mysql.usertable`テーブルを復元する方法を示しています。

```shell
br restore full -f '*.*' -f '!mysql.*' -f 'mysql.usertable' -s $external_storage_url --with-sys-table
```

In the preceding command,

-   `-f '*.*'` はデフォルトのルールを上書きするために使用されます
-   `-f '!mysql.*'` は、`mysql` 内のテーブルをそれ以外が指定されない限り、BRが復元しないように指示します。
-   `-f 'mysql.usertable'` は、`mysql.usertable` を復元することを示します。

`mysql.usertable` のみを復元する必要がある場合は、次のコマンドを実行してください。

```shell
br restore full -f 'mysql.usertable' -s $external_storage_url --with-sys-table
```

注意してください。[テーブルフィルタ](/table-filter.md#syntax)を設定しても、**BRは以下のシステムテーブルを復元しません**:

-   統計テーブル (`mysql.stat_*`)
-   システム変数テーブル (`mysql.tidb`, `mysql.global_variables`)
-   [その他のシステムテーブル](https://github.com/pingcap/tidb/blob/master/br/pkg/restore/systable_restore.go#L31)

## バックアップとリストアに関するその他の情報 {#other-things-you-may-want-to-know-about-backup-and-restore}

### バックアップデータのサイズはどのくらいですか？バックアップのレプリカはありますか？ {#what-is-the-size-of-the-backup-data-are-there-replicas-of-the-backup}

データバックアップ中、各リージョンのリーダーノードにバックアップファイルが生成されます。バックアップのサイズはデータサイズと同じであり、冗長なレプリカはありません。したがって、総データサイズはおおよそ TiKV データの総数をレプリカの数で割ったものです。

ただし、ローカルストレージからデータをリストアする場合、レプリカの数は TiKV ノードの数と同じです。なぜなら、各 TiKV はすべてのバックアップファイルにアクセスする必要があるからです。

### BRを使用してバックアップまたはリストア後に監視ノードに表示されるディスク使用量が一貫していないのはなぜですか？ {#why-is-the-disk-usage-shown-on-the-monitoring-node-inconsistent-after-backup-or-restore-using-br}

この不一致は、バックアップで使用されるデータ圧縮率がリストアで使用されるデフォルトの率と異なるためです。チェックサムが成功した場合は、この問題を無視できます。

### BRがバックアップデータをリストアした後、テーブルの統計情報とインデックスの TiDB の統計情報を更新するために `ANALYZE` ステートメントを実行する必要がありますか？ {#after-br-restores-the-backup-data-do-i-need-to-execute-the-analyze-statement-on-the-table-to-update-the-statistics-of-tidb-on-the-tables-and-indexes}

BRは統計情報をバックアップしません（v4.0.9を除く）。したがって、バックアップデータをリストアした後、`ANALYZE TABLE` を手動で実行するか、TiDB が自動的に `ANALYZE` を実行するのを待つ必要があります。

v4.0.9では、BRはデフォルトで統計情報をバックアップしますが、これはあまりにも多くのメモリを消費します。バックアッププロセスがうまくいくようにするために、v4.0.10からは統計情報のバックアップがデフォルトで無効になっています。

テーブルで `ANALYZE` を実行しない場合、不正確な統計情報のために TiDB は最適な実行プランを選択できません。クエリのパフォーマンスが重要でない場合は、`ANALYZE` を無視できます。

### 1つのクラスタのデータをリストアするために複数のリストアタスクを同時に開始できますか？ {#can-i-start-multiple-restore-tasks-at-the-same-time-to-restore-the-data-of-a-single-cluster}

**強くお勧めしません**。以下の理由から、1つのクラスタのデータをリストアするために複数のリストアタスクを同時に開始することは強くお勧めしません。

-   BRがデータをリストアする際、PDの一部のグローバル設定を変更します。したがって、同時に複数のリストアタスクを開始すると、これらの設定が誤って上書きされ、クラスタの異常な状態を引き起こす可能性があります。
-   BRはデータをリストアするために多くのクラスタリソースを消費するため、実際には複数のリストアタスクを並行して実行することは、リストア速度をわずかに向上させるだけです。
-   データをリストアするために複数のリストアタスクを並行して実行するテストは行われていないため、成功が保証されていません。

### BRはテーブルの `SHARD_ROW_ID_BITS` および `PRE_SPLIT_REGIONS` 情報をバックアップしますか？リストアされたテーブルには複数のリージョンがありますか？ {#does-br-back-up-the-shard-row-id-bits-and-pre-split-regions-information-of-a-table-does-the-restored-table-have-multiple-regions}

はい。BRはテーブルの [`SHARD_ROW_ID_BITS` および `PRE_SPLIT_REGIONS`](/sql-statements/sql-statement-split-region.md#pre_split_regions) 情報をバックアップします。リストアされたテーブルのデータも複数のリージョンに分割されます。
