---
title: TiDB Log Backup and PITR Guide
summary: Learns about how to perform log backup and PITR in TiDB.
---

# TiDB ログバックアップとPITRガイド {#tidb-log-backup-and-pitr-guide}

完全バックアップ（スナップショットバックアップ）には、特定の時点での完全なクラスタデータが含まれていますが、TiDBログバックアップは、アプリケーションによって書き込まれたデータをタイムリーに指定されたストレージにバックアップできます。必要に応じて復元ポイントを選択し、つまり、ポイントインタイムリカバリ（PITR）を実行する場合は、[ログバックアップを開始](#start-log-backup)し、[定期的に完全バックアップを実行](#run-full-backup-regularly)する必要があります。

`br`（以下、`br`と呼びます）コマンドラインツールを使用してデータをバックアップまたは復元する前に、まず[ `br`をインストール](/br/br-use-overview.md#deploy-and-use-br)する必要があります。

## TiDBクラスタのバックアップ {#back-up-tidb-cluster}

### ログバックアップを開始 {#start-log-backup}

> **Note:**
>
> - 以下の例では、Amazon S3アクセスキーとシークレットキーがアクセス権を認可するために使用されていると想定しています。IAMロールを使用してアクセス権を認可する場合は、`--send-credentials-to-tikv`を`false`に設定する必要があります。
> - 他のストレージシステムや認可方法を使用してアクセス権を認可する場合は、[バックアップストレージ](/br/backup-and-restore-storages.md)に従ってパラメータ設定を調整してください。

ログバックアップを開始するには、`br log start`を実行します。クラスタは一度に1つのログバックアップタスクのみを実行できます。

```shell
tiup br log start --task-name=pitr --pd "${PD_IP}:2379" \
--storage 's3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}'
```

TiDBクラスターのバックアップタスクが開始されると、手動で停止するまでバックグラウンドで実行されます。このプロセス中、TiDBの変更ログは定期的に小さなバッチで指定されたストレージにバックアップされます。ログバックアップタスクの状態をクエリするには、次のコマンドを実行します:

```shell
tiup br log status --task-name=pitr --pd "${PD_IP}:2379"
```

予想される出力：

```
● Total 1 Tasks.
> #1 <
    name: pitr
    status: ● NORMAL
    start: 2022-05-13 11:09:40.7 +0800
      end: 2035-01-01 00:00:00 +0800
    storage: s3://backup-101/log-backup
    speed(est.): 0.00 ops/s
checkpoint[global]: 2022-05-13 11:31:47.2 +0800; gap=4m53s
```

### 定期的に完全バックアップを実行する {#run-full-backup-regularly}

スナップショットバックアップは完全バックアップの方法として使用できます。`br backup full`を実行して、クラスタのスナップショットをバックアップストレージに固定スケジュール（例えば、2日ごと）でバックアップできます。

```shell
tiup br backup full --pd "${PD_IP}:2379" \
--storage 's3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}'
```

## PITRの実行 {#run-pitr}

バックアップ保持期間内の任意の時点にクラスタを復元するには、`br restore point`を使用できます。このコマンドを実行すると、**復元したい時点**、**時点の直前の最新のスナップショットバックアップデータ**、および**ログバックアップデータ**を指定する必要があります。BRは自動的に復元に必要なデータを決定し、それらのデータを指定されたクラスタに順番に復元します。

```shell
br restore point --pd "${PD_IP}:2379" \
--storage='s3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}' \
--full-backup-storage='s3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}' \
--restored-ts '2022-05-15 18:00:00+0800'
```

データの復元中、ターミナルの進行バーで進行状況を表示できます。復元は、完全な復元とログの復元（メタファイルの復元とKVファイルの復元）の2つのフェーズに分かれています。各フェーズが完了すると、`br`は復元時間やデータサイズなどの情報を出力します。

```shell
Full Restore <--------------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
*** ["Full Restore success summary"] ****** [total-take=xxx.xxxs] [restore-data-size(after-compressed)=xxx.xxx] [Size=xxxx] [BackupTS={TS}] [total-kv=xxx] [total-kv-size=xxx] [average-speed=xxx]
Restore Meta Files <--------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
Restore KV Files <----------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
*** ["restore log success summary"] [total-take=xxx.xx] [restore-from={TS}] [restore-to={TS}] [total-kv-count=xxx] [total-size=xxx]
```

## 期限切れのデータをクリーンアップする {#clean-up-outdated-data}

[TiDB Backup and Restoreの使用概要](/br/br-use-overview.md)に記載されているように：

PITRを実行するには、リストアポイントの前に完全バックアップをリストアし、完全バックアップポイントとリストアポイントの間のログバックアップが必要です。したがって、バックアップ保持期間を超えるログバックアップについては、`br log truncate`を使用して指定された時点より前のバックアップを削除できます。**完全スナップショットの前のログバックアップのみを削除することをお勧めします**。

以下の手順は、バックアップ保持期間を超えるバックアップデータをクリーンアップする方法を説明しています：

1. バックアップ保持期間外の**最後の完全バックアップ**を取得します。

2. `validate`コマンドを使用して、バックアップに対応する時刻を取得します。2022/09/01より前のバックアップデータをクリーンアップする必要があると仮定すると、この時点より前の最後の完全バックアップを探し、それがクリーンアップされないようにします。

   ```shell
   FULL_BACKUP_TS=`tiup br validate decode --field="end-version" --storage "s3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}"| tail -n1`
   ```

3. スナップショットバックアップ`FULL_BACKUP_TS`より前のログバックアップデータを削除します：

   ```shell
   tiup br log truncate --until=${FULL_BACKUP_TS} --storage='s3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}'
   ```

4. スナップショットバックアップ`FULL_BACKUP_TS`より前のスナップショットデータを削除します：

   ```shell
   rm -rf s3://backup-101/snapshot-${date}
   ```

## PITRのパフォーマンスと影響 {#performance-and-impact-of-pitr}

### 機能 {#capabilities}

- 各TiKVノードでは、PITRはスナップショットデータを1時間あたり280 GB、ログデータを1時間あたり30 GBの速度でリストアできます。
- BRは、期限切れのログバックアップデータを1時間あたり600 GBの速度で削除します。

> **Note:**
>
> 上記の仕様は、以下の2つのテストシナリオのテスト結果に基づいています。実際のデータは異なる場合があります。
>
> - スナップショットデータのリストア速度 = スナップショットデータサイズ / (期間 \* TiKVノードの数)
> - ログデータのリストア速度 = リストアされたログデータサイズ / (期間 \* TiKVノードの数)
>
> スナップショットデータサイズとは、単一のレプリカ内のすべてのKVsの論理サイズを指します。BRは、クラスターに構成されたレプリカの数に応じてすべてのレプリカをリストアします。レプリカが多いほど、実際にリストアできるデータが多くなります。
> テストのすべてのクラスターのデフォルトレプリカ数は3です。
> 全体のリストアパフォーマンスを向上させるには、TiKV構成ファイルの[`import.num-threads`](/tikv-configuration-file.md#import)項目とBRコマンドの[`concurrency`](/br/use-br-command-line-tool.md#common-options)オプションを変更できます。

テストシナリオ1（[TiDB Cloud](https://tidbcloud.com)で実行）：

- TiKVノード数（8コア、16 GBメモリ）：21
- TiKV構成項目`import.num-threads`：8
- BRコマンドオプション`concurrency`：128
- リージョン数：183,000
- クラスターで作成された新しいログデータ：10 GB/h
- 書き込み（INSERT/UPDATE/DELETE）QPS：10,000

テストシナリオ2（TiDB Self-Hostedで実行）：

- TiKVノード数（8コア、64 GBメモリ）：6
- TiKV構成項目`import.num-threads`：8
- BRコマンドオプション`concurrency`：128
- リージョン数：50,000
- クラスターで作成された新しいログデータ：10 GB/h
- 書き込み（INSERT/UPDATE/DELETE）QPS：10,000

## 関連項目 {#see-also}

- [TiDB Backup and Restoreの使用事例](/br/backup-and-restore-use-cases.md)
- [brコマンドラインマニュアル](/br/use-br-command-line-tool.md)
- [ログバックアップとPITRアーキテクチャ](/br/br-log-architecture.md)
