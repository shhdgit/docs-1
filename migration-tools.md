---
title: TiDB Migration Tools Overview
summary: Learn an overview of the TiDB migration tools.
---

# TiDB移行ツールの概要 {#tidb-migration-tools-overview}

TiDBには、完全なデータ移行、増分データ移行、バックアップとリストア、データレプリケーションなど、さまざまなシナリオに対応した複数のデータ移行ツールが用意されています。

このドキュメントでは、ユーザーシナリオ、サポートされている上流および下流、利点、およびこれらのツールの制限について紹介します。必要に応じて適切なツールを選択できます。

<!--以下の図は、各移行ツールのユーザーシナリオを示しています。

!TiDB移行ツール media/migration-tools.png-->

## [TiDBデータ移行（DM）](/dm/dm-overview.md) {#tidb-data-migration-dm-dm-dm-overview-md}

- **ユーザーシナリオ**: MySQL互換データベースからTiDBへのデータ移行
- **上流**: MySQL、MariaDB、Aurora
- **下流**: TiDB
- **利点**:
  - 完全なデータ移行および増分レプリケーションをサポートする便利で統一されたデータ移行タスク管理ツール
  - テーブルと操作のフィルタリングをサポート
  - シャードのマージと移行をサポート
- **制限**: データのインポート速度は、TiDB Lightningの[論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode.md)とほぼ同じであり、TiDB Lightningの[物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode.md)よりもはるかに低いため、1 TiB未満のサイズの完全なデータを移行する場合はDMの使用を推奨します。

## [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) {#tidb-lightning-tidb-lightning-tidb-lightning-overview-md}

- **ユーザーシナリオ**: TiDBへの完全なデータインポート
- **上流（インポート元ファイル）**:
  - Dumplingからエクスポートされたファイル
  - Amazon AuroraまたはApache HiveによってエクスポートされたParquetファイル
  - CSVファイル
  - ローカルディスクまたはAmazon S3からのデータ
- **下流**: TiDB
- **利点**:
  - 大量のデータを迅速にインポートし、TiDBクラスター内の特定のテーブルを迅速に初期化することをサポート
  - インポートの進行状況を保存するためのチェックポイントをサポートし、`tidb-lightning`は再起動後に残りのインポートを続行します
  - データのフィルタリングをサポート
- **制限**:
  - データのインポートに[物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md)を使用する場合、インポートプロセス中はTiDBクラスターがサービスを提供できません。
  - TiDBサービスに影響を与えたくない場合は、TiDB Lightningの[論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode-usage.md)に従ってデータのインポートを実行してください。

## [Dumpling](/dumpling-overview.md) {#dumpling-dumpling-overview-md}

- **ユーザーシナリオ**: MySQLまたはTiDBからの完全なデータエクスポート
- **上流**: MySQL、TiDB
- **下流（出力ファイル）**: SQL、CSV
- **利点**:
  - データのフィルタリングを容易にするテーブルフィルタ機能をサポート
  - Amazon S3にデータをエクスポートすることをサポート
- **制限**:
  - TiDB以外のデータベースにエクスポートされたデータをリストアする場合は、Dumplingの使用を推奨します。
  - エクスポートされたデータを別のTiDBクラスターにリストアする場合は、バックアップとリストア（BR）を使用することをお勧めします。

## [TiCDC](/ticdc/ticdc-overview.md) {#ticdc-ticdc-ticdc-overview-md}

- **ユーザーシナリオ**: このツールは、TiKVの変更ログを取得して実装されています。任意の上流TSOでクラスターデータを一貫した状態に復元し、他のシステムがデータ変更を購読できるようにサポートします。
- **上流**: TiDB
- **下流**: TiDB、MySQL、Kafka、MQ、Confluent、Amazon S3、GCS、Azure Blob Storage、NFSなどのストレージサービス。
- **利点**: TiCDCオープンプロトコルを提供
- **制限**: TiCDCは、少なくとも1つの有効なインデックスを持つテーブルのみをレプリケートします。次のシナリオはサポートされていません:
  - RawKVのみを使用するTiKVクラスター。
  - TiDBのDDL操作`CREATE SEQUENCE`および`SEQUENCE`関数。

## [バックアップとリストア（BR）](/br/backup-and-restore-overview.md) {#backup-restore-br-br-backup-and-restore-overview-md}

- **ユーザーシナリオ**: バックアップとデータのリストアによって大量のTiDBクラスターデータを移行
- **上流**: TiDB
- **下流（出力ファイル）**: SST、backup.metaファイル、backup.lockファイル
- **利点**:
  - 別のTiDBクラスターにデータを移行するのに適しています
  - データを災害復旧のための外部ストレージにバックアップすることをサポート
- **制限**:
  - BRがデータをTiCDCまたはDrainerの上流クラスターにリストアする場合、リストアされたデータはTiCDCまたはDrainerによって下流にレプリケートされません。
  - BRは、`mysql.tidb`テーブルの`new_collation_enabled`値が同じクラスター間の操作のみをサポートします。

## [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) {#sync-diff-inspector-sync-diff-inspector-sync-diff-inspector-overview-md}

- **ユーザシナリオ**: データベースに保存されているデータをMySQLプロトコルで比較する
- **上流**: TiDB、MySQL
- **下流**: TiDB、MySQL
- **利点**: 少量のデータが一貫性がないシナリオでデータを修復するために使用できる
- **制限**:
  - MySQLとTiDB間のデータ移行に対するオンラインチェックはサポートされていません。
  - JSON、BIT、BINARY、BLOBなどのデータタイプはサポートされていません。

## TiUPを使用してツールをインストールする {#install-tools-using-tiup}

TiDB v4.0以降、TiUPはTiDBエコシステム内の異なるクラスタコンポーネントを管理するのに役立つパッケージマネージャとして機能します。これで、単一のコマンドを使用して任意のクラスタコンポーネントを管理できます。

### ステップ1. TiUPをインストール {#step-1-install-tiup}

```shell
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```

グローバル環境変数を再宣言します：

```shell
source ~/.bash_profile
```

### ステップ2. コンポーネントのインストール {#step-2-install-components}

次のコマンドを使用して、利用可能なすべてのコンポーネントを表示できます:

```shell
tiup list
```

コマンドの出力には利用可能なすべてのコンポーネントがリストされます。

```bash
Available components:
Name            Owner    Description
----            -----    -----------
bench           pingcap  Benchmark database with different workloads
br              pingcap  TiDB/TiKV cluster backup restore tool
cdc             pingcap  CDC is a change data capture tool for TiDB
client          pingcap  Client to connect playground
cluster         pingcap  Deploy a TiDB cluster for production
ctl             pingcap  TiDB controller suite
dm              pingcap  Data Migration Platform manager
dmctl           pingcap  dmctl component of Data Migration Platform
errdoc          pingcap  Document about TiDB errors
pd-recover      pingcap  PD Recover is a disaster recovery tool of PD, used to recover the PD cluster which cannot start or provide services normally
playground      pingcap  Bootstrap a local TiDB cluster for fun
tidb            pingcap  TiDB is an open source distributed HTAP database compatible with the MySQL protocol
tidb-lightning  pingcap  TiDB Lightning is a tool used for fast full import of large amounts of data into a TiDB cluster
tiup            pingcap  TiUP is a command-line component management tool that can help to download and install TiDB platform components to the local system
```

コンポーネントを選択してインストールしてください：

```shell
tiup install dumpling tidb-lightning
```

**Note:**

特定のバージョンのコンポーネントをインストールするには、`tiup install <component>[:version]` コマンドを使用してください。

### ステップ3. TiUPおよびそのコンポーネントの更新（オプション） {#step-3-update-tiup-and-its-components-optional}

新しいバージョンのリリースログと互換性の注意事項を確認することをお勧めします。

```shell
tiup update --self && tiup update dm
```

## 関連項目 {#see-also}

- [TiUPをオフラインでデプロイする](/production-deployment-using-tiup.md#deploy-tiup-offline)
- [バイナリでツールをダウンロードしてインストールする](/download-ecosystem-tools.md)
