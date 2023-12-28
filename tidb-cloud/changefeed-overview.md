---
title: Changefeed
summary: TiDB Cloud changefeed helps you stream data from TiDB Cloud to other data services.
---

# Changefeed {#changefeed}

TiDB Cloudのchangefeed機能を使用すると、TiDB Cloudから他のデータサービスにデータをストリーム配信できます。現在、TiDB CloudではApache Kafka、MySQL、TiDB Cloud、およびクラウドストレージへのデータストリーミングをサポートしています。

> **Note:**
>
> - 現在、TiDB Cloudではクラスターごとに最大100のchangefeedを許可しています。
> - 現在、TiDB Cloudではchangefeedごとに最大100のテーブルフィルタールールを許可しています。
> - [TiDB Serverlessクラスター](/tidb-cloud/select-cluster-tier.md#tidb-serverless)の場合、changefeed機能は使用できません。

changefeed機能にアクセスするには、TiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。changefeedページが表示されます。

changefeedページでは、changefeedを作成したり、既存のchangefeedのリストを表示したり、既存のchangefeedを操作したり（スケーリング、一時停止、再開、編集、削除）、既存のchangefeedを操作したりすることができます。

## changefeedを作成する {#create-a-changefeed}

changefeedを作成するには、次のチュートリアルを参照してください。

- [Apache Kafkaへのシンク](/tidb-cloud/changefeed-sink-to-apache-kafka.md)
- [MySQLへのシンク](/tidb-cloud/changefeed-sink-to-mysql.md)
- [TiDB Cloudへのシンク](/tidb-cloud/changefeed-sink-to-tidb-cloud.md)
- [クラウドストレージへのシンク](/tidb-cloud/changefeed-sink-to-cloud-storage.md)

## changefeed RCUsをクエリする {#query-changefeed-rcus}

1.ターゲットTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2.クエリしたい対応するchangefeedを見つけ、**...** > **View**をクリックします。**Action**列。
3.ページの**Specification**エリアで、現在のTiCDCレプリケーション容量ユニット（RCU）が表示されます。

## changefeedをスケーリングする {#scale-a-changefeed}

changefeedのTiCDCレプリケーション容量ユニット（RCU）をスケーリングアップまたはダウンして、changfeedを変更できます。

> **Note:**
>
> - クラスターのchangefeedをスケーリングするには、このクラスターのすべてのchangefeedが2023年3月28日以降に作成されていることを確認してください。
> - クラスターに2023年3月28日以前に作成されたchangefeedがある場合、このクラスターの既存のchangefeedおよび新しく作成されたchangefeedのいずれも、スケーリングアップまたはダウンをサポートしません。

1.ターゲットTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2.スケーリングしたい対応するchangefeedを見つけ、**...** > **Scale Up/Down**をクリックします。**Action**列。
3.新しい仕様を選択します。
4.**Submit**をクリックします。

スケーリングプロセスは約10分かかります（この間、changfeedは正常に動作します）が、新しい仕様に切り替わるのに数秒かかります（この間、changefeedは自動的に一時停止および再開されます）。

## changefeedを一時停止または再開する {#pause-or-resume-a-changefeed}

1.ターゲットTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2.一時停止または再開したい対応するchangefeedを見つけ、**...** > **Pause/Resume**をクリックします。**Action**列。

## changefeedを編集する {#edit-a-changefeed}

> **Note:**
>
> TiDB Cloudでは、現在一時停止されているchangefeedの編集のみを許可しています。

1.ターゲットTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。

2.一時停止したいchangefeedを見つけ、**...** > **Pause**をクリックします。**Action**列。

3. changefeedのステータスが`Paused`に変更されたら、対応するchangefeedを編集するために\*\*...\*\* > **Edit**をクリックします。

   TiDB Cloudはデフォルトでchangefeed構成を入力します。次の構成を変更できます。

   - Apache Kafkaシンク：すべての構成。
   - MySQLシンク：**MySQL Connection**、**Table Filter**、および**Event Filter**。
   - TiDB Cloudシンク：**TiDB Cloud Connection**、**Table Filter**、および**Event Filter**。
   - クラウドストレージシンク：**Storage Endpoint**、**Table Filter**、および**Event Filter**。

4.構成を編集した後、対応するchangefeedを再開するために\*\*...\*\* > **Resume**をクリックします。

## changefeedを削除する {#delete-a-changefeed}

1. ターゲットTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. 削除したい対応するchangefeedを見つけ、**...** > **Delete**をクリックします。

## Changefeedの課金 {#changefeed-billing}

TiDB Cloudでのchangefeedの課金については、[Changefeedの課金](/tidb-cloud/tidb-cloud-billing-ticdc-rcu.md)を参照してください。

## Changefeedの状態 {#changefeed-states}

レプリケーションタスクの状態は、レプリケーションタスクの実行状態を表します。実行中のプロセスでは、レプリケーションタスクはエラーで失敗したり、手動で一時停止したり、再開したり、指定された`TargetTs`に到達したりすることがあります。これらの動作により、レプリケーションタスクの状態が変化することがあります。

状態は次のように説明されます。

- `CREATING`：レプリケーションタスクが作成中です。
- `RUNNING`：レプリケーションタスクが正常に実行され、チェックポイント-tsが正常に進行します。
- `EDITING`：レプリケーションタスクが編集中です。
- `PAUSING`：レプリケーションタスクが一時停止中です。
- `PAUSED`：レプリケーションタスクが一時停止されています。
- `RESUMING`：レプリケーションタスクが再開中です。
- `DELETING`：レプリケーションタスクが削除中です。
- `DELETED`：レプリケーションタスクが削除されました。
- `WARNING`：レプリケーションタスクが警告を返します。リカバリ可能なエラーのため、レプリケーションを続行できません。この状態のchangefeedは、状態が`RUNNING`に変わるまで再開し続けます。この状態のchangefeedは、[GC操作](https://docs.pingcap.com/tidb/stable/garbage-collection-overview)をブロックします。
- `FAILED`：レプリケーションタスクが失敗しました。エラーのため、レプリケーションタスクを再開できず、自動的に回復することができません。問題が解決されるまで、増分データのガベージコレクション（GC）が実行される前に、失敗したchangefeedを手動で再開することができます。増分データのデフォルトのTime-To-Live（TTL）期間は24時間であり、これは、changefeedが中断されてから24時間以内にデータが削除されないことを意味します。
