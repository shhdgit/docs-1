---
title: TiDB Migration Tools Overview
summary: Learn an overview of the TiDB migration tools.
---

# TiDB マイグレーションツールの概要 {#tidb-migration-tools-overview}

TiDB は、完全なデータ移行、増分データ移行、バックアップとリストア、データレプリケーションなど、さまざまなシナリオに対応した複数のデータ移行ツールを提供しています。

このドキュメントでは、これらのツールのユーザーシナリオ、サポートされているアップストリームとダウンストリーム、利点、制限について紹介します。必要に応じて適切なツールを選択できます。

<!--以下の図は各マイグレーションツールのユーザーシナリオを示しています。

!TiDB マイグレーションツール media/migration-tools.png-->

## [TiDB データマイグレーション（DM）](/dm/dm-overview.md) {#tidb-data-migration-dm-dm-dm-overview-md}

-   **ユーザーシナリオ**: MySQL 互換データベースから TiDB へのデータ移行
-   **アップストリーム**: MySQL、MariaDB、Aurora
-   **ダウンストリーム**: TiDB
-   **利点**:
    -   完全なデータ移行と増分レプリケーションをサポートする便利で統一されたデータ移行タスク管理ツール
    -   テーブルと操作のフィルタリングをサポート
    -   シャードのマージと移行をサポート
-   **制限**: データのインポート速度は TiDB Lightning の [論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode.md) とほぼ同じであり、TiDB Lightning の [物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode.md) よりもはるかに低いため、1 TiB 未満のサイズの完全なデータを移行する場合は DM を使用することをお勧めします。

## [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) {#tidb-lightning-tidb-lightning-tidb-lightning-overview-md}

-   **ユーザーシナリオ**: TiDB への完全なデータインポート
-   **アップストリーム（インポート元ファイル）**:
    -   Dumpling からエクスポートされたファイル
    -   Amazon Aurora や Apache Hive によってエクスポートされた Parquet ファイル
    -   CSV ファイル
    -   ローカルディスクや Amazon S3 からのデータ
-   **ダウンストリーム**: TiDB
-   **利点**:
    -   大量のデータを迅速にインポートし、TiDB クラスター内の特定のテーブルを迅速に初期化することをサポート
    -   インポートの進捗状況を保存するチェックポイントをサポートし、`tidb-lightning` は再起動後に残りのインポートを続行する
    -   データのフィルタリングをサポート
-   **制限**:
    -   データのインポートに [物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md) を使用する場合、インポートプロセス中は TiDB クラスターはサービスを提供できません。
    -   TiDB サービスに影響を与えたくない場合は、TiDB Lightning の [論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode-usage.md) に従ってデータのインポートを行ってください。

## [Dumpling](/dumpling-overview.md) {#dumpling-dumpling-overview-md}

-   **ユーザーシナリオ**: MySQL または TiDB からの完全なデータエクスポート
-   **アップストリーム**: MySQL、TiDB
-   **ダウンストリーム（出力ファイル）**: SQL、CSV
-   **利点**:
    -   データを簡単にフィルタリングできるテーブルフィルタリング機能をサポート
    -   Amazon S3 へのデータのエクスポートをサポート
-   **制限**:
    -   TiDB 以外のデータベースにエクスポートされたデータをリストアする場合は、Dumpling を使用することをお勧めします。
    -   エクスポートされたデータを別の TiDB クラスターにリストアする場合は、Backup & Restore (BR) を使用することをお勧めします。

## [TiCDC](/ticdc/ticdc-overview.md) {#ticdc-ticdc-ticdc-overview-md}

-   **ユーザーシナリオ**: このツールは TiKV の変更ログを取得して実装されています。任意のアップストリーム TSO でクラスターデータを一貫した状態に復元し、他のシステムがデータ変更を購読できるようにサポートします。
-   **アップストリーム**: TiDB
-   **ダウンストリーム**: TiDB、MySQL、Kafka、MQ、Confluent、Amazon S3、GCS、Azure Blob Storage、NFS などのストレージサービス
-   **利点**: TiCDC オープンプロトコルを提供
-   **制限**: TiCDC は少なくとも1つの有効なインデックスを持つテーブルのみをレプリケートします。次のシナリオはサポートされていません:
    -   RawKV のみを使用する TiKV クラスター。
    -   TiDB での DDL 操作 `CREATE SEQUENCE` および `SEQUENCE` 関数。

## [Backup & Restore (BR)](/br/backup-and-restore-overview.md) {#backup-restore-br-br-backup-and-restore-overview-md}

-   **ユーザシナリオ**: バックアップとデータの復元によって大量のTiDBクラスタデータを移行する
-   **上流**: TiDB
-   **下流（出力ファイル）**: SST、backup.metaファイル、backup.lockファイル
-   **利点**:
    -   別のTiDBクラスタにデータを移行するのに適しています
    -   災害復旧のために外部ストレージにデータをバックアップするサポートがあります
-   **制限**:
    -   BRがデータをTiCDCまたはDrainerの上流クラスタに復元すると、TiCDCまたはDrainerによって下流にレプリケートされない
    -   BRは、`mysql.tidb`テーブルの`new_collation_enabled`値が同じであるクラスタ間の操作のみをサポートしています。

## [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) {#sync-diff-inspector-sync-diff-inspector-sync-diff-inspector-overview-md}

-   **ユーザシナリオ**: MySQLプロトコルでデータベースに格納されたデータを比較する
-   **上流**: TiDB、MySQL
-   **下流**: TiDB、MySQL
-   **利点**: 少量のデータが不整合な場合にデータを修復するために使用できます
-   **制限**:
    -   MySQLとTiDB間のデータ移行に対するオンラインチェックはサポートされていません。
    -   JSON、BIT、BINARY、BLOBなどのデータタイプはサポートされていません。

## TiUPを使用してツールをインストールする {#install-tools-using-tiup}

TiDB v4.0以降、TiUPはTiDBエコシステム内の異なるクラスタコンポーネントを管理するのに役立つパッケージマネージャとして機能します。今や、単一のコマンドを使用して任意のクラスタコンポーネントを管理できます。

### ステップ1. TiUPをインストールする {#step-1-install-tiup}

```shell
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```

グローバル環境変数を再宣言します。

```shell
source ~/.bash_profile
```

### ステップ2. コンポーネントのインストール {#step-2-install-components}

以下のコマンドを使用して、利用可能なすべてのコンポーネントを表示できます：

```shell
tiup list
```

コマンドの出力には利用可能なすべてのコンポーネントがリストされています。

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

インストールするコンポーネントを選択してください。

```shell
tiup install dumpling tidb-lightning
```

**注意:**

特定のバージョンのコンポーネントをインストールするには、`tiup install <component>[:version]` コマンドを使用してください。

### ステップ3. TiUPとそのコンポーネントを更新する（オプション） {#step-3-update-tiup-and-its-components-optional}

新しいバージョンのリリースログと互換性の注意事項を確認することをお勧めします。

```shell
tiup update --self && tiup update dm
```

## 関連項目 {#see-also}

-   [TiUPをオフラインで展開する](/production-deployment-using-tiup.md#deploy-tiup-offline)
-   [バイナリでツールをダウンロードしてインストールする](/download-ecosystem-tools.md)
