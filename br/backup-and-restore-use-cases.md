---
title: TiDB Backup and Restore Use Cases
summary: Learn the use cases of backing up and restoring data using br command-line tool.
---

# TiDB バックアップとリストアのユースケース {#tidb-backup-and-restore-use-cases}

[TiDB スナップショットバックアップとリストアガイド](/br/br-snapshot-guide.md) および [TiDB ログバックアップとPITRガイド](/br/br-pitr-guide.md) は、TiDBによって提供されるバックアップとリストアソリューション、つまり、スナップショット（フル）バックアップとリストア、ログバックアップとポイントインタイムリカバリ（PITR）を紹介しています。このドキュメントは、特定のユースケースでのTiDBのバックアップとリストアソリューションを素早く始めるのに役立ちます。

AWSにTiDB本番クラスタをデプロイしており、ビジネスチームから以下の要件が求められていると仮定します。

- データの変更をタイムリーにバックアップします。データベースが災害に遭遇した場合、データの損失を最小限に抑えてアプリケーションを迅速にリカバリできるようにします（数分のデータ損失は許容されます）。
- 特定の時間にはないが、毎月ビジネス監査を実施します。監査のリクエストが届いた場合、過去1か月の特定の時点でデータをクエリするためのデータベースを提供する必要があります。

PITRを使用すると、前述の要件を満たすことができます。

## TiDBクラスタとBRをデプロイ {#deploy-the-tidb-cluster-and-br}

PITRを使用するには、TiDBクラスタをv6.2.0以上にデプロイし、BRをTiDBクラスタと同じバージョンに更新する必要があります。このドキュメントでは、v7.1.3を例にしています。

以下の表は、TiDBクラスタでPITRを使用するための推奨ハードウェアリソースを示しています。

| コンポーネント | CPU   | メモリ     | ディスク | AWSインスタンス  | インスタンス数 |
| ------- | ----- | ------- | ---- | ---------- | ------- |
| TiDB    | 8コア以上 | 16 GB以上 | SAS  | c5.2xlarge | 2       |
| PD      | 8コア以上 | 16 GB以上 | SSD  | c5.2xlarge | 3       |
| TiKV    | 8コア以上 | 32 GB以上 | SSD  | m5.2xlarge | 3       |
| BR      | 8コア以上 | 16 GB以上 | SAS  | c5.2xlarge | 1       |
| Monitor | 8コア以上 | 16 GB以上 | SAS  | c5.2xlarge | 1       |

> **Note:**
>
> - BRがバックアップおよびリストアタスクを実行する際、PDおよびTiKVにアクセスする必要があります。BRがすべてのPDおよびTiKVノードに接続できるようにしてください。
> - BRおよびPDサーバーは同じタイムゾーンを使用する必要があります。

TiUPを使用してTiDBクラスタをデプロイまたはアップグレードします。

- 新しいTiDBクラスタをデプロイするには、[TiDBクラスタをデプロイ](/production-deployment-using-tiup.md)を参照してください。
- TiDBクラスタがv6.2.0より前の場合は、[TiDBクラスタをアップグレード](/upgrade-tidb-using-tiup.md)を参照してアップグレードしてください。

TiUPを使用してBRをインストールまたはアップグレードします。

- インストール:

  ```shell
  tiup install br:v7.1.3
  ```

- アップグレード:

  ```shell
  tiup update br:v7.1.3
  ```

## バックアップストレージ（Amazon S3）を構成する {#configure-backup-storage-amazon-s3}

バックアップタスクを開始する前に、バックアップストレージを準備します。これには、次の側面が含まれます。

1. バックアップデータを保存するS3バケットおよびディレクトリを準備します。
2. S3バケットへのアクセス権限を構成します。
3. 各バックアップデータを保存するサブディレクトリを計画します。

詳細な手順は次のとおりです。

1. S3にディレクトリを作成してバックアップデータを保存します。この例では、ディレクトリは`s3://tidb-pitr-bucket/backup-data`です。

   1. バケットを作成します。バックアップデータを保存する既存のS3を選択できます。ない場合は、[AWSドキュメント: バケットの作成](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)を参照してS3バケットを作成します。この例では、バケット名は`tidb-pitr-bucket`です。
   2. バックアップデータ用のディレクトリを作成します。バケット（`tidb-pitr-bucket`）で、`backup-data`という名前のディレクトリを作成します。詳細な手順については、[AWSドキュメント: フォルダを使用したAmazon S3コンソールのオブジェクトの整理](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-folders.html)を参照してください。

2. BRおよびTiKVがS3ディレクトリにアクセスするための権限を構成します。S3バケットへのアクセス権限をIAMメソッドを使用して付与することをお勧めします。詳細な手順については、[AWSドキュメント: ユーザーポリシーでバケットへのアクセスを制御する](https://docs.aws.amazon.com/AmazonS3/latest/userguide/walkthrough1.html)を参照してください。必要な権限は次のとおりです。

   - バックアップクラスタのTiKVおよびBRは、`s3://tidb-pitr-bucket/backup-data`ディレクトリの`s3:ListBucket`、`s3:PutObject`、`s3:AbortMultipartUpload`権限が必要です。
   - リストアクラスタのTiKVおよびBRは、`s3://tidb-pitr-bucket/backup-data`ディレクトリの`s3:ListBucket`、`s3:GetObject`、`s3:PutObject`権限が必要です。

3. スナップショット（フル）バックアップおよびログバックアップを保存するディレクトリ構造を計画します。

   - すべてのスナップショットバックアップデータは、`s3://tidb-pitr-bucket/backup-data/snapshot-${date}`ディレクトリに保存されます。`${date}`はスナップショットバックアップの開始時刻です。例えば、2022/05/12 00:01:30に開始したスナップショットバックアップは`s3://tidb-pitr-bucket/backup-data/snapshot-20220512000130`に保存されます。
   - ログバックアップデータは、`s3://tidb-pitr-bucket/backup-data/log-backup/`ディレクトリに保存されます。

## バックアップポリシーを決定する {#determine-the-backup-policy}

データ損失の最小化、迅速なリカバリ、および1か月以内のビジネス監査の要件を満たすために、次のようにバックアップポリシーを設定できます。

- データベースのデータ変更を連続的にバックアップするためにログバックアップを実行します。
- 2日ごとに00:00 AMにスナップショットバックアップを実行します。
- 30日以内にスナップショットバックアップデータとログバックアップデータを保持し、30日以上前のバックアップデータをクリーンアップします。

## ログバックアップを実行する {#run-log-backup}

ログバックアップタスクを開始した後、ログバックアッププロセスはデータベースのデータ変更を連続的にTiKVクラスタで実行し、S3ストレージに送信します。ログバックアップタスクを開始するには、次のコマンドを実行します：

```shell
tiup br log start --task-name=pitr --pd="${PD_IP}:2379" \
--storage='s3://tidb-pitr-bucket/backup-data/log-backup'
```

ログバックアップタスクが実行されているとき、バックアップの状態をクエリできます。

```shell
tiup br log status --task-name=pitr --pd="${PD_IP}:2379"

● Total 1 Tasks.
> #1 <
    name: pitr
    status: ● NORMAL
    start: 2022-05-13 11:09:40.7 +0800
      end: 2035-01-01 00:00:00 +0800
    storage: s3://tidb-pitr-bucket/backup-data/log-backup
    speed(est.): 0.00 ops/s
checkpoint[global]: 2022-05-13 11:31:47.2 +0800; gap=4m53s
```

## スナップショットバックアップの実行 {#run-snapshot-backup}

定期的にcrontabなどの自動ツールを使用してスナップショットバックアップタスクを実行できます。たとえば、2日ごとに00:00にスナップショットバックアップを実行します。

以下は2つのスナップショットバックアップの例です。

- 2022/05/14 00:00:00にスナップショットバックアップを実行

  ```shell
  tiup br backup full --pd="${PD_IP}:2379" \
  --storage='s3://tidb-pitr-bucket/backup-data/snapshot-20220514000000' \
  --backupts='2022/05/14 00:00:00'
  ```

- 2022/05/16 00:00:00にスナップショットバックアップを実行

  ```shell
  tiup br backup full --pd="${PD_IP}:2379" \
  --storage='s3://tidb-pitr-bucket/backup-data/snapshot-20220516000000' \
  --backupts='2022/05/16 00:00:00'
  ```

## PITRの実行 {#run-pitr}

2022/05/15 18:00:00のデータをクエリする必要があるとします。PITRを使用して、2022/05/14に取得したスナップショットバックアップとその後のスナップショットと2022/05/15 18:00:00の間のログバックアップデータを復元して、クラスターをその時点に復元できます。

コマンドは次のとおりです：

```shell
tiup br restore point --pd="${PD_IP}:2379" \
--storage='s3://tidb-pitr-bucket/backup-data/log-backup' \
--full-backup-storage='s3://tidb-pitr-bucket/backup-data/snapshot-20220514000000' \
--restored-ts '2022-05-15 18:00:00+0800'

Full Restore <--------------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
[2022/05/29 18:15:39.132 +08:00] [INFO] [collector.go:69] ["Full Restore success summary"] [total-ranges=12] [ranges-succeed=xxx] [ranges-failed=0] [split-region=xxx.xxxµs] [restore-ranges=xxx] [total-take=xxx.xxxs] [restore-data-size(after-compressed)=xxx.xxx] [Size=xxxx] [BackupTS={TS}] [total-kv=xxx] [total-kv-size=xxx] [average-speed=xxx]
Restore Meta Files <--------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
Restore KV Files <----------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
[2022/05/29 18:15:39.325 +08:00] [INFO] [collector.go:69] ["restore log success summary"] [total-take=xxx.xx] [restore-from={TS}] [restore-to={TS}] [total-kv-count=xxx] [total-size=xxx]
```

## Clean up outdated data {#clean-up-outdated-data}

自動ツール（crontabなど）を使用して、2日ごとに古いデータをクリーンアップできます。

たとえば、次のコマンドを実行して古いデータをクリーンアップできます。

- 2022/05/14 00:00:00より前のスナップショットデータを削除します

  ```shell
  rm s3://tidb-pitr-bucket/backup-data/snapshot-20220514000000
  ```

- 2022/05/14 00:00:00より前のログバックアップデータを削除します

  ```shell
  tiup br log truncate --until='2022-05-14 00:00:00 +0800' --storage='s3://tidb-pitr-bucket/backup-data/log-backup'
  ```

## 関連情報 {#see-also}

- [バックアップストレージ](/br/backup-and-restore-storages.md)
- [スナップショットバックアップおよびリストアコマンドマニュアル](/br/br-snapshot-manual.md)
- [ログバックアップおよびPITRコマンドマニュアル](/br/br-pitr-manual.md)
