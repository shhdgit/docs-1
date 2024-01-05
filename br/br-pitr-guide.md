---
title: TiDB Log Backup and PITR Guide
summary: Learns about how to perform log backup and PITR in TiDB.
---

# TiDBログバックアップおよびPITRガイド {#tidb-log-backup-and-pitr-guide}

フルバックアップ（スナップショットバックアップ）には、特定の時点でのフルクラスタデータが含まれますが、TiDBログバックアップは、アプリケーションによって書き込まれたデータをタイムリーに指定されたストレージにバックアップできます。必要な復元ポイントを選択する場合、つまり、ポイントインタイムリカバリ（PITR）を実行する場合は、[ログバックアップを開始](#start-log-backup)し、[定期的にフルバックアップを実行](#run-full-backup-regularly)することができます。

`br`（以下`br`と呼びます）コマンドラインツールを使用してデータをバックアップまたは復元する前に、まず[brをインストール](/br/br-use-overview.md#deploy-and-use-br)する必要があります。

## TiDBクラスタのバックアップ {#back-up-tidb-cluster}

### ログバックアップを開始 {#start-log-backup}

> **Note:**
>
> - 以下の例では、Amazon S3アクセスキーとシークレットキーを使用して権限を承認することを前提としています。IAMロールを使用して権限を承認する場合は、`--send-credentials-to-tikv`を`false`に設定する必要があります。
> - 他のストレージシステムや承認方法を使用して権限を承認する場合は、[バックアップストレージ](/br/backup-and-restore-storages.md)に従ってパラメータ設定を調整してください。

ログバックアップを開始するには、`br log start`を実行します。クラスタは一度に1つのログバックアップタスクのみを実行できます。

```shell
tiup br log start --task-name=pitr --pd "${PD_IP}:2379" \
--storage 's3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}'
```

ログバックアップタスクが開始されると、TiDBクラスタのバックグラウンドで実行され、手動で停止するまで続行されます。このプロセス中、TiDBの変更ログは定期的に指定されたストレージに小さなバッチでバックアップされます。ログバックアップタスクのステータスをクエリするには、次のコマンドを実行します。

```shell
tiup br log status --task-name=pitr --pd "${PD_IP}:2379"
```

期待される出力：

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

### 定期的にフルバックアップを実行 {#run-full-backup-regularly}

スナップショットバックアップはフルバックアップの一種として使用できます。クラスタスナップショットをバックアップストレージに定期的にバックアップするには、`br backup full`を実行できます（たとえば、2日ごとなどの固定スケジュールで）。

```shell
tiup br backup full --pd "${PD_IP}:2379" \
--storage 's3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}'
```

## PITRの実行 {#run-pitr}

バックアップ保持期間内の任意の時点にクラスタを復元するには、`br restore point`を使用できます。このコマンドを実行すると、**復元したい時点**、**時点より前の最新のスナップショットバックアップデータ**、および**ログバックアップデータ**を指定する必要があります。BRは、復元に必要なデータを自動的に決定して読み取り、それらのデータを指定されたクラスタに順番に復元します。

```shell
br restore point --pd "${PD_IP}:2379" \
--storage='s3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}' \
--full-backup-storage='s3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}' \
--restored-ts '2022-05-15 18:00:00+0800'
```

データの復元中には、ターミナルの進行状況バーを使用して進行状況を表示できます。復元はフル復元とログ復元（メタファイルの復元とKVファイルの復元）の2つのフェーズに分かれています。各フェーズが完了すると、`br`は復元時間やデータサイズなどの情報を出力します。

```shell
Full Restore <--------------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
*** ["Full Restore success summary"] ****** [total-take=xxx.xxxs] [restore-data-size(after-compressed)=xxx.xxx] [Size=xxxx] [BackupTS={TS}] [total-kv=xxx] [total-kv-size=xxx] [average-speed=xxx]
Restore Meta Files <--------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
Restore KV Files <----------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
*** ["restore log success summary"] [total-take=xxx.xx] [restore-from={TS}] [restore-to={TS}] [total-kv-count=xxx] [total-size=xxx]
```

## 期限切れのデータのクリーンアップ {#clean-up-outdated-data}

[TiDBバックアップとリストアの使用概要](/br/br-use-overview.md)に記載されているように：

PITRを実行するには、復元ポイントより前のフルバックアップと、フルバックアップポイントと復元ポイントの間のログバックアップを復元する必要があります。したがって、バックアップ保持期間を超えるログバックアップについては、`br log truncate`を使用して指定した時点より前のバックアップを削除できます。**フルスナップショットの前のログバックアップのみを削除することをお勧めします**。

バックアップ保持期間を超えるバックアップデータをクリーンアップする手順は次のとおりです：

1. バックアップ保持期間外の**最後のフルバックアップ**を取得します。

2. `validate`コマンドを使用して、バックアップに対応する時点を取得します。2022/09/01より前のバックアップデータをクリーンアップする必要があると仮定すると、この時点より前の最後のフルバックアップを探し、それがクリーンアップされないように確認する必要があります。

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

- 各TiKVノードで、PITRはスナップショットデータを280 GB/h、ログデータを30 GB/hの速度で復元できます。
- BRは、期限切れのログバックアップデータを600 GB/hの速度で削除します。

> **Note:**
>
> 上記の仕様は、以下の2つのテストシナリオのテスト結果に基づいています。実際のデータは異なる場合があります。
>
> - スナップショットデータの復元速度 = スナップショットデータサイズ / （期間 \* TiKVノード数）
> - ログデータの復元速度 = 復元されたログデータサイズ / （期間 \* TiKVノード数）
>
> スナップショットデータサイズは、単一のレプリカのすべてのKVの論理サイズを指し、実際に復元されるデータの実際の量ではありません。BRは、クラスタの構成されたレプリカ数に従ってすべてのレプリカを復元します。レプリカが多いほど、実際に復元できるデータが多くなります。
> テストのすべてのクラスタのデフォルトレプリカ数は3です。
> 全体的な復元パフォーマンスを向上させるには、TiKV構成ファイルの[`import.num-threads`](/tikv-configuration-file.md#import)項目と、BRコマンドの[`concurrency`](/br/use-br-command-line-tool.md#common-options)オプションを変更できます。

テストシナリオ1（[TiDB Cloud](https://tidbcloud.com)で実行）：

- TiKVノード数（8コア、16 GBメモリ）：21
- TiKV構成項目`import.num-threads`：8
- BRコマンドオプション`concurrency`：128
- リージョン数：183,000
- クラスタで作成された新しいログデータ：10 GB/h
- 書き込み（INSERT/UPDATE/DELETE）QPS：10,000

テストシナリオ2（TiDB Self-Hostedで実行）：

- TiKVノード数（8コア、64 GBメモリ）：6
- TiKV構成項目`import.num-threads`：8
- BRコマンドオプション`concurrency`：128
- リージョン数：50,000
- クラスタで作成された新しいログデータ：10 GB/h
- 書き込み（INSERT/UPDATE/DELETE）QPS：10,000

## 関連項目 {#see-also}

- [TiDBバックアップとリストアの使用事例](/br/backup-and-restore-use-cases.md)
- [brコマンドラインマニュアル](/br/use-br-command-line-tool.md)
- [ログバックアップとPITRアーキテクチャ](/br/br-log-architecture.md)
