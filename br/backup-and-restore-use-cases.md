---
title: TiDB Backup and Restore Use Cases
summary: Learn the use cases of backing up and restoring data using br command-line tool.
---

# TiDB バックアップとリストアのユースケース {#tidb-backup-and-restore-use-cases}

[TiDB スナップショットバックアップとリストアガイド](/br/br-snapshot-guide.md) と [TiDB ログバックアップと PITR ガイド](/br/br-pitr-guide.md) では、TiDB が提供するバックアップとリストアのソリューションであるスナップショット（フル）バックアップとリストア、ログバックアップとポイントインタイムリカバリ（PITR）について紹介します。このドキュメントでは、特定のユースケースで TiDB のバックアップとリストアのソリューションを素早く始めることができます。

AWS 上に TiDB 本番クラスターをデプロイし、ビジネスチームから以下の要件が要求されたと仮定します：

- データの変更をタイムリーにバックアップする。データベースが障害に遭遇した場合、データ損失を最小限に抑えてアプリケーションを迅速にリカバリできるようにする（数分のデータ損失は許容される）。
- 毎月ビジネス監査を実施する。特定の時間には実施しない。監査リクエストが受信された場合、過去の1ヶ月間の特定の時点でデータをクエリするためのデータベースを提供する必要があります。

PITR を使用すると、前述の要件を満たすことができます。

## TiDB クラスターと BR をデプロイする {#deploy-the-tidb-cluster-and-br}

PITR を使用するには、TiDB クラスター >= v6.2.0 をデプロイし、BR を TiDB クラスターと同じバージョンに更新する必要があります。このドキュメントでは、v7.1.3 を例として使用します。

次の表は、TiDB クラスターで PITR を使用するための推奨ハードウェアリソースを示しています。

| コンポーネント | CPU    | メモリ     | ディスク | AWS インスタンス | インスタンス数 |
| ------- | ------ | ------- | ---- | ---------- | ------- |
| TiDB    | 8 コア以上 | 16 GB以上 | SAS  | c5.2xlarge | 2       |
| PD      | 8 コア以上 | 16 GB以上 | SSD  | c5.2xlarge | 3       |
| TiKV    | 8 コア以上 | 32 GB以上 | SSD  | m5.2xlarge | 3       |
| BR      | 8 コア以上 | 16 GB以上 | SAS  | c5.2xlarge | 1       |
| Monitor | 8 コア以上 | 16 GB以上 | SAS  | c5.2xlarge | 1       |

> **Note:**
>
> - BR がバックアップとリストアのタスクを実行する際には、PD と TiKV にアクセスする必要があります。BR がすべての PD と TiKV ノードに接続できるようにしてください。
> - BR と PD サーバーは同じタイムゾーンを使用する必要があります。

TiUP を使用して TiDB クラスターをデプロイまたはアップグレードする：

- 新しい TiDB クラスターをデプロイする場合は、[TiDB クラスターをデプロイする](/production-deployment-using-tiup.md) を参照してください。
- TiDB クラスターが v6.2.0 より前の場合は、[TiDB クラスターをアップグレードする](/upgrade-tidb-using-tiup.md) を参照してください。

TiUP を使用して BR をインストールまたはアップグレードする：

- インストール：

  ```shell
  tiup install br:v7.1.3
  ```

- アップグレード：

  ```shell
  tiup update br:v7.1.3
  ```

## バックアップストレージ（Amazon S3）を設定する {#configure-backup-storage-amazon-s3}

バックアップタスクを開始する前に、バックアップストレージを準備する必要があります。これには、次のようなことが含まれます：

1. バックアップデータを保存する S3 バケットとディレクトリを準備する。
2. S3 バケットにアクセスするための権限を設定する。
3. 各バックアップデータを保存するサブディレクトリを計画する。

詳細な手順は次のとおりです：

1. S3にバックアップデータを保存するためのディレクトリを作成します。この例では、ディレクトリは`s3://tidb-pitr-bucket/backup-data`です。

   1. バケットを作成します。既存のS3を選択してバックアップデータを保存することができます。存在しない場合は、[AWSドキュメント：バケットの作成](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)を参照して、S3バケットを作成します。この例では、バケット名は`tidb-pitr-bucket`です。
   2. バックアップデータ用のディレクトリを作成します。バケット（`tidb-pitr-bucket`）内に、`backup-data`という名前のディレクトリを作成します。詳細な手順については、[AWSドキュメント：Amazon S3コンソールでフォルダを使用してオブジェクトを整理する](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-folders.html)を参照してください。

2. BRとTiKVがS3ディレクトリにアクセスするための権限を設定します。S3バケットにアクセスする最も安全な方法であるIAMメソッドを使用して権限を付与することをお勧めします。詳細な手順については、[AWSドキュメント：ユーザーポリシーを使用してバケットへのアクセスを制御する](https://docs.aws.amazon.com/AmazonS3/latest/userguide/walkthrough1.html)を参照してください。必要な権限は次のとおりです。

   - バックアップクラスターのTiKVとBRは、`s3://tidb-pitr-bucket/backup-data`ディレクトリの`s3:ListBucket`、`s3:PutObject`、および`s3:AbortMultipartUpload`権限が必要です。
   - リストアクラスターのTiKVとBRは、`s3://tidb-pitr-bucket/backup-data`ディレクトリの`s3:ListBucket`、`s3:GetObject`、および`s3:PutObject`権限が必要です。

3. スナップショット（フル）バックアップとログバックアップを保存するディレクトリ構造を計画します。

   - すべてのスナップショットバックアップデータは、`s3://tidb-pitr-bucket/backup-data/snapshot-${date}`ディレクトリに保存されます。`${date}`はスナップショットバックアップの開始時刻です。例えば、2022/05/12 00:01:30に開始されたスナップショットバックアップは、`s3://tidb-pitr-bucket/backup-data/snapshot-20220512000130`に保存されます。
   - ログバックアップデータは、`s3://tidb-pitr-bucket/backup-data/log-backup/`ディレクトリに保存されます。

## バックアップポリシーを決定する {#determine-the-backup-policy}

最小のデータ損失、迅速なリカバリ、および1か月以内のビジネス監査の要件を満たすために、次のようにバックアップポリシーを設定できます。

- データベースのデータ変更を連続的にバックアップするために、ログバックアップを実行します。
- 2日ごとに00:00 AMにスナップショットバックアップを実行します。
- スナップショットバックアップデータとログバックアップデータを30日以内に保持し、30日より古いバックアップデータをクリーンアップします。

## ログバックアップを実行する {#run-log-backup}

ログバックアップタスクが開始されると、ログバックアッププロセスがTiKVクラスターで実行され、データベースのデータ変更がS3ストレージに連続的に送信されます。ログバックアップタスクを開始するには、次のコマンドを実行します：

```shell
tiup br log start --task-name=pitr --pd="${PD_IP}:2379" \
--storage='s3://tidb-pitr-bucket/backup-data/log-backup'
```

ログバックアップタスクが実行されている場合、バックアップの状態をクエリできます。

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

crontabなどの自動ツールを使用して、定期的にスナップショットバックアップタスクを実行することができます。例えば、2日ごとに00:00にスナップショットバックアップを実行します。

以下は2つのスナップショットバックアップの例です：

- 2022/05/14 00:00:00にスナップショットバックアップを実行する

  ```shell
  tiup br backup full --pd="${PD_IP}:2379" \
  --storage='s3://tidb-pitr-bucket/backup-data/snapshot-20220514000000' \
  --backupts='2022/05/14 00:00:00'
  ```

- 2022/05/16 00:00:00にスナップショットバックアップを実行する

  ```shell
  tiup br backup full --pd="${PD_IP}:2379" \
  --storage='s3://tidb-pitr-bucket/backup-data/snapshot-20220516000000' \
  --backupts='2022/05/16 00:00:00'
  ```

## PITRの実行 {#run-pitr}

2022/05/15 18:00:00でデータをクエリする必要があるとします。その時点のスナップショットバックアップと2022/05/14から2022/05/15 18:00:00までのログバックアップデータを復元することで、その時点までのクラスターを復元することができます。

コマンドは以下の通りです：

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

## 古いデータのクリーンアップ {#clean-up-outdated-data}

crontabなどの自動ツールを使用して、2日ごとに古いデータをクリーンアップすることができます。

例えば、以下のコマンドを実行して古いデータをクリーンアップすることができます：

- 2022/05/14 00:00:00より前のスナップショットデータを削除する

  ```shell
  rm s3://tidb-pitr-bucket/backup-data/snapshot-20220514000000
  ```

- 2022/05/14 00:00:00より前のログバックアップデータを削除する

  ```shell
  tiup br log truncate --until='2022-05-14 00:00:00 +0800' --storage='s3://tidb-pitr-bucket/backup-data/log-backup'
  ```

## 関連情報 {#see-also}

- [バックアップストレージ](/br/backup-and-restore-storages.md)
- [スナップショットバックアップとリストアコマンドマニュアル](/br/br-snapshot-manual.md)
- [ログバックアップとPITRコマンドマニュアル](/br/br-pitr-manual.md)
