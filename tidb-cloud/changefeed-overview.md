---
title: Changefeed
summary: TiDB Cloud changefeed helps you stream data from TiDB Cloud to other data services.
---

# Changefeed {#changefeed}

TiDB Cloud changefeedは、TiDB Cloudから他のデータサービスにデータをストリーム配信するのに役立ちます。現在、TiDB CloudはApache Kafka、MySQL、TiDB Cloud、およびクラウドストレージへのデータストリーミングをサポートしています。

> **Note:**
>
> - 現在、TiDB Cloudではクラスターごとに最大100のchangefeedのみが許可されています。
> - 現在、TiDB Cloudではchangefeedごとに最大100のテーブルフィルタールールのみが許可されています。
> - [TiDB Serverlessクラスター](/tidb-cloud/select-cluster-tier.md#tidb-serverless)の場合、changefeed機能は利用できません。

changefeed機能にアクセスするには、TiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。changefeedページが表示されます。

changefeedページでは、changefeedを作成したり、既存のchangefeedのリストを表示したり、既存のchangefeedを操作したり（スケーリング、一時停止、再開、編集、削除など）、することができます。

## changefeedを作成する {#create-a-changefeed}

changefeedを作成するには、次のチュートリアルを参照してください：

- [Apache Kafkaへのシンク](/tidb-cloud/changefeed-sink-to-apache-kafka.md)
- [MySQLへのシンク](/tidb-cloud/changefeed-sink-to-mysql.md)
- [TiDB Cloudへのシンク](/tidb-cloud/changefeed-sink-to-tidb-cloud.md)
- [クラウドストレージへのシンク](/tidb-cloud/changefeed-sink-to-cloud-storage.md)

## Changefeed RCUsをクエリする {#query-changefeed-rcus}

1. 対象のTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. クエリしたい対応するchangefeedを見つけ、**...** > **View**を**Action**列でクリックします。
3. ページの**Specification**エリアで現在のTiCDC Replication Capacity Units（RCUs）を確認できます。

## changefeedをスケーリングする {#scale-a-changefeed}

changefeedのTiCDC Replication Capacity Units（RCUs）をスケーリングアップまたはダウンして変更できます。

> **Note:**
>
> - クラスターのchangefeedをスケーリングするには、このクラスターのすべてのchangefeedが2023年3月28日以降に作成されたことを確認してください。
> - クラスターに2023年3月28日以前に作成されたchangefeedがある場合、このクラスターの既存のchangefeedおよび新しく作成されたchangefeedのいずれもスケーリングアップまたはダウンをサポートしません。

1. 対象のTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. スケーリングしたい対応するchangefeedを見つけ、**...** > **Scale Up/Down**を**Action**列でクリックします。
3. 新しい仕様を選択します。
4. **Submit**をクリックします。

スケーリングプロセスの完了には約10分かかります（この間、changefeedは正常に動作します）。新しい仕様に切り替わるのには数秒かかります（この間、changefeedは自動的に一時停止および再開されます）。

## changefeedを一時停止または再開する {#pause-or-resume-a-changefeed}

1. 対象のTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. 一時停止または再開したい対志するchangefeedを見つけ、**...** > **Pause/Resume**を**Action**列でクリックします。

## changefeedを編集する {#edit-a-changefeed}

> **Note:**
>
> TiDB Cloudでは現在、一時停止状態のchangefeedのみを編集できます。

1. 対象のTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。

2. 一時停止したいchangefeedを見つけ、**...** > **Pause**を**Action**列でクリックします。

3. changefeedのステータスが`Paused`に変わったら、対応するchangefeedを編集するために\*\*...\*\* > **Edit**をクリックします。

   TiDB Cloudは、デフォルトでchangefeed構成を自動的に生成します。次の構成を変更できます：

   - Apache Kafkaシンク：すべての構成。
   - MySQLシンク：**MySQL Connection**、**Table Filter**、および**Event Filter**。
   - TiDB Cloudシンク：**TiDB Cloud Connection**、**Table Filter**、および**Event Filter**。
   - クラウドストレージシンク：**Storage Endpoint**、**Table Filter**、および**Event Filter**。

4. 構成を編集した後、対応するchangefeedを再開するために\*\*...\*\* > **Resume**をクリックします。

## changefeedを削除する {#delete-a-changefeed}

1. 対象のTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. 削除したい対志するchangefeedを見つけ、**...** > **Delete**を**Action**列でクリックします。

## changefeedの請求 {#changefeed-billing}

TiDB Cloudのchangefeedの請求については、[Changefeed billing](/tidb-cloud/tidb-cloud-billing-ticdc-rcu.md)を参照してください。

## changefeedの状態 {#changefeed-states}

レプリケーションタスクの状態は、レプリケーションタスクの実行状態を表します。実行プロセス中に、レプリケーションタスクはエラーで失敗したり、手動で一時停止したり、再開したり、指定された`TargetTs`に到達したりすることがあります。これらの動作により、レプリケーションタスクの状態が変化する可能性があります。

状態は次のように説明されます：

- `CREATING`：レプリケーションタスクが作成中です。
- `RUNNING`：レプリケーションタスクは正常に実行され、チェックポイント-tsが正常に進行しています。
- `EDITING`：レプリケーションタスクが編集中です。
- `PAUSING`：レプリケーションタスクが一時停止中です。
- `PAUSED`：レプリケーションタスクが一時停止中です。
- `RESUMING`：レプリケーションタスクが再開中です。
- `DELETING`：レプリケーションタスクが削除中です。
- `DELETED`：レプリケーションタスクが削除されました。
- `WARNING`：レプリケーションタスクが警告を返します。回復可能なエラーのため、レプリケーションは続行できません。この状態のchangefeedは`RUNNING`に状態が変わるまで再開し続けます。この状態のchangefeedは[GC operations](https://docs.pingcap.com/tidb/stable/garbage-collection-overview)をブロックします。
- `FAILED`：レプリケーションタスクが失敗しました。いくつかのエラーのため、レプリケーションタスクは再開できず、自動的に回復することはできません。問題がGCの増分データの前に解決される場合、失敗したchangefeedを手動で再開できます。増分データのデフォルトのTime-To-Live（TTL）期間は24時間であり、これはchangefeedが中断されてから24時間以内にデータを削除しないことを意味します。"
