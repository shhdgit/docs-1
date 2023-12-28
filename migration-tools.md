---
title: TiDB Migration Tools Overview
summary: Learn an overview of the TiDB migration tools.
---

# TiDB移行ツールの概要 {#tidb-migration-tools-overview}

TiDBには、完全データ移行、増分データ移行、バックアップとリストア、データレプリケーションなど、さまざまなシナリオに対応する複数のデータ移行ツールが用意されています。

このドキュメントでは、これらのツールのユーザーシナリオ、サポートされるアップストリームとダウンストリーム、利点、制限について紹介します。必要に応じて、適切なツールを選択することができます。

<!--次の図は、各移行ツールのユーザーシナリオを示しています。

!TiDB移行ツール media/migration-tools.png-->

## [TiDBデータ移行（DM）](/dm/dm-overview.md) {#tidb-data-migration-dm-dm-dm-overview-md}

- **ユーザーシナリオ**: MySQL互換データベースからTiDBへのデータ移行
- **アップストリーム**: MySQL、MariaDB、Aurora
- **ダウンストリーム**: TiDB
- **利点**:
  - 完全データ移行と増分レプリケーションをサポートする便利で統一されたデータ移行タスク管理ツール
  - テーブルと操作のフィルタリングをサポート
  - シャードのマージと移行をサポート
- **制限**: データのインポート速度は、TiDB Lightningの[論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode.md)とほぼ同じであり、TiDB Lightningの[物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode.md)よりも低いため、サイズが1 TiB未満の完全データを移行する場合は、DMを使用することをお勧めします。

## [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) {#tidb-lightning-tidb-lightning-tidb-lightning-overview-md}

- **ユーザーシナリオ**: TiDBへの完全データインポート
- **アップストリーム（インポート元ファイル）**:
  - Dumplingからエクスポートされたファイル
  - Amazon AuroraまたはApache HiveによってエクスポートされたParquetファイル
  - CSVファイル
  - ローカルディスクまたはAmazon S3からのデータ
- **ダウンストリーム**: TiDB
- **利点**:
  - 大量のデータを高速にインポートし、TiDBクラスター内の特定のテーブルを高速に初期化することをサポート
  - チェックポイントをサポートして、インポートの進捗状況を保存し、再起動後に`tidb-lightning`が残りの部分からインポートを続けることをサポート
  - データのフィルタリングをサポート
- **制限**:
  - データのインポートに[物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md)を使用する場合、インポートプロセス中にTiDBクラスターはサービスを提供できません。
  - TiDBサービスに影響を与えたくない場合は、TiDB Lightningの[論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode-usage.md)に従ってデータをインポートしてください。

## [Dumpling](/dumpling-overview.md) {#dumpling-dumpling-overview-md}

- **ユーザーシナリオ**: MySQLまたはTiDBからの完全データエクスポート
- **アップストリーム**: MySQL、TiDB
- **ダウンストリーム（出力ファイル）**: SQL、CSV
- **利点**:
  - データをより簡単にフィルタリングできるテーブルフィルタ機能をサポート
  - Amazon S3にデータをエクスポートすることをサポート
- **制限**:
  - TiDB以外のデータベースにエクスポートしたデータをリストアする場合は、Dumplingを使用することをお勧めします。
  - エクスポートしたデータを別のTiDBクラスターにリストアする場合は、バックアップ＆リストア（BR）を使用することをお勧めします。

## [TiCDC](/ticdc/ticdc-overview.md) {#ticdc-ticdc-ticdc-overview-md}

- **ユーザーシナリオ**: このツールは、TiKVの変更ログを取得して実装されています。任意のアップストリームTSOでクラスターデータを一貫した状態に復元し、他のシステムがデータ変更を購読することをサポートします。
- **アップストリーム**: TiDB
- **ダウンストリーム**: TiDB、MySQL、Kafka、MQ、Confluent、Amazon S3、GCS、Azure Blob Storage、NFSなどのストレージサービス。
- **利点**: TiCDCオープンプロトコルを提供
- **制限**: TiCDCは、少なくとも1つの有効なインデックスを持つテーブルのみをレプリケートします。次のシナリオはサポートされていません。
  - RawKVのみを使用するTiKVクラスター。
  - TiDBでのDDL操作`CREATE SEQUENCE`および`SEQUENCE`関数。

## [バックアップ＆リストア（BR）](/br/backup-and-restore-overview.md) {#backup-restore-br-br-backup-and-restore-overview-md}

- **ユーザーシナリオ**: バックアップとリストアによる大量のTiDBクラスターデータの移行
- **アップストリーム**: TiDB
- **ダウンストリーム（出力ファイル）**: SST、バックアップ.metaファイル、バックアップ.lockファイル
- **利点**:
  - 別のTiDBクラスターにデータを移行するのに適しています
  - バックアップを外部ストレージに行い、ディザスタリカバリを行うことをサポート
- **制限**:
  - BRがTiCDCまたはDrainerのダウンストリームにレストアする場合、レストアされたデータはTiCDCまたはDrainerによってダウンストリームにレプリケートされません。
  - BRは、`mysql.tidb`テーブルの`new_collation_enabled`値が同じであるクラスター間でのみ操作をサポートします。

## [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) {#sync-diff-inspector-sync-diff-inspector-sync-diff-inspector-overview-md}

- **ユーザーシナリオ**: MySQLプロトコルでデータベースに格納されたデータを比較する
- **上流**: TiDB、MySQL
- **下流**: TiDB、MySQL
- **利点**: 少量のデータが一貫性がないシナリオでデータを修復するために使用できる
- **制限**:
  - MySQLとTiDB間のデータ移行ではオンラインチェックはサポートされていません。
  - JSON、BIT、BINARY、BLOBなどのデータタイプはサポートされていません。

## TiUPを使用してツールをインストールする {#install-tools-using-tiup}

TiDB v4.0以降、TiUPはTiDBエコシステム内のさまざまなクラスターコンポーネントを管理するのに役立つパッケージマネージャーとして機能します。今では、単一のコマンドで任意のクラスターコンポーネントを管理できます。

### ステップ1. TiUPをインストールする {#step-1-install-tiup}

```shell
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```

グローバル環境変数を再宣言する：

```shell
source ~/.bash_profile
```

### ステップ2. コンポーネントのインストール {#step-2-install-components}

以下のコマンドを使用して、利用可能なすべてのコンポーネントを確認できます。

```shell
tiup list
```

コマンドの出力には、利用可能なすべてのコンポーネントがリストされます:

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

インストールするコンポーネントを選択してください:

```shell
tiup install dumpling tidb-lightning
```

> **注意:**
>
> 特定のバージョンのコンポーネントをインストールするには、`tiup install <component>[:version]`コマンドを使用してください。

### ステップ3. TiUPとそのコンポーネントの更新（オプション） {#step-3-update-tiup-and-its-components-optional}

新しいバージョンのリリースログと互換性の注意事項を確認することをお勧めします。

```shell
tiup update --self && tiup update dm
```

## 関連情報 {#see-also}

- [TiUPをオフラインでデプロイする](/production-deployment-using-tiup.md#deploy-tiup-offline)
- [バイナリでツールをダウンロードしてインストールする](/download-ecosystem-tools.md)
