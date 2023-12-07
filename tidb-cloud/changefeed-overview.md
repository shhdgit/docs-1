---
title: Changefeed
summary: TiDB Cloud changefeed helps you stream data from TiDB Cloud to other data services.
---

# Changefeed {#changefeed}

TiDB Cloudのchangefeedは、TiDB Cloudから他のデータサービスにデータをストリーム配信するのに役立ちます。現在、TiDB CloudはApache Kafka、MySQL、TiDB Cloud、およびクラウドストレージにデータをストリーム配信することをサポートしています。

> **注意:**
>
> -   現在、TiDB Cloudではクラスタごとに最大100のchangefeedのみが許可されています。
> -   現在、TiDB Cloudでは1つのchangefeedごとに最大100のテーブルフィルタルールのみが許可されています。
> -   [TiDB Serverlessクラスタ](/tidb-cloud/select-cluster-tier.md#tidb-serverless)では、changefeed機能は利用できません。

changefeed機能にアクセスするには、TiDBクラスタのクラスタ概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。changefeedページが表示されます。

changefeedページでは、changefeedを作成したり、既存のchangefeedのリストを表示したり、既存のchangefeedを操作したり（スケーリング、一時停止、再開、編集、削除など）、することができます。

## changefeedを作成する {#create-a-changefeed}

changefeedを作成するには、次のチュートリアルを参照してください:

-   [Apache Kafkaへのデータ転送](/tidb-cloud/changefeed-sink-to-apache-kafka.md)
-   [MySQLへのデータ転送](/tidb-cloud/changefeed-sink-to-mysql.md)
-   [TiDB Cloudへのデータ転送](/tidb-cloud/changefeed-sink-to-tidb-cloud.md)
-   [クラウドストレージへのデータ転送](/tidb-cloud/changefeed-sink-to-cloud-storage.md)

## Changefeed RCUsをクエリする {#query-changefeed-rcus}

1.  対象のTiDBクラスタのクラスタ概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2.  クエリしたい対応するchangefeedを見つけ、**...** > **View**をクリックします。
3.  ページの**Specification**エリアで、現在のTiCDC Replication Capacity Units（RCUs）を確認できます。

## changefeedをスケーリングする {#scale-a-changefeed}

changefeedのTiCDC Replication Capacity Units（RCUs）をスケーリングアップまたはダウンして変更することができます。

> **注意:**
>
> -   クラスタのchangefeedをスケーリングするには、このクラスタのすべてのchangefeedが2023年3月28日以降に作成されたことを確認してください。
> -   クラスタに2023年3月28日より前に作成されたchangefeedがある場合、このクラスタの既存のchangefeedおよび新しく作成されたchangefeedのいずれもスケーリングアップまたはダウンをサポートしていません。

1.  対象のTiDBクラスタのクラスタ概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2.  スケーリングしたい対応するchangefeedを見つけ、**...** > **Scale Up/Down**をクリックします。
3.  新しい仕様を選択します。
4.  **Submit**をクリックします。

スケーリングプロセスの完了には約10分かかります（この間、changefeedは通常動作します）、新しい仕様に切り替わるのに数秒かかります（この間、changefeedは自動的に一時停止および再開されます）。

## changefeedを一時停止または再開する {#pause-or-resume-a-changefeed}

1.  対象のTiDBクラスタのクラスタ概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2.  一時停止または再開したい対志するchangefeedを見つけ、**...** > **Pause/Resume**をクリックします。

## changefeedを編集する {#edit-a-changefeed}

> **注意:**
>
> TiDB Cloudでは現在、一時停止状態のchangefeedのみを編集することができます。

1.  対象のTiDBクラスタのクラスタ概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。

2.  編集したいchangefeedを見つけ、**...** > **Pause**をクリックします。

3.  changefeedの状態が`Paused`に変わったら、対応するchangefeedを編集するために\*\*...\*\* > **Edit**をクリックします。

    TiDB Cloudはデフォルトでchangefeedの構成を設定します。以下の構成を変更することができます:

    -   Apache Kafka sink: すべての構成。
    -   MySQL sink: **MySQL Connection**、**Table Filter**、**Event Filter**。
    -   TiDB Cloud sink: **TiDB Cloud Connection**、**Table Filter**、**Event Filter**。
    -   クラウドストレージ sink: **Storage Endpoint**、**Table Filter**、**Event Filter**。

4.  構成を編集した後、対応するchangefeedを再開するために\*\*...\*\* > **Resume**をクリックします。

## changefeedを削除する {#delete-a-changefeed}

1.  ターゲットTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2.  削除したい対応するchangefeedを見つけ、**...** > **Delete**を**Action**列でクリックします。

## Changefeedの請求 {#changefeed-billing}

TiDB Cloudでchangefeedの請求を学ぶには、[Changefeed billing](/tidb-cloud/tidb-cloud-billing-ticdc-rcu.md)を参照してください。

## Changefeedの状態 {#changefeed-states}

レプリケーションタスクの状態は、レプリケーションタスクの実行状態を表します。実行中のプロセス中、レプリケーションタスクはエラーで失敗することがあり、手動で一時停止、再開、または指定された`TargetTs`に到達することがあります。これらの動作により、レプリケーションタスクの状態が変化することがあります。

状態は以下のように説明されます：

-   `CREATING`: レプリケーションタスクが作成中です。
-   `RUNNING`: レプリケーションタスクは正常に実行され、チェックポイント-tsが正常に進行します。
-   `EDITING`: レプリケーションタスクが編集中です。
-   `PAUSING`: レプリケーションタスクが一時停止中です。
-   `PAUSED`: レプリケーションタスクが一時停止されています。
-   `RESUMING`: レプリケーションタスクが再開中です。
-   `DELETING`: レプリケーションタスクが削除中です。
-   `DELETED`: レプリケーションタスクが削除されました。
-   `WARNING`: レプリケーションタスクが警告を返します。回復可能なエラーのため、レプリケーションは続行できません。この状態のchangefeedは`RUNNING`に状態が変わるまで再開し続けます。この状態のchangefeedは[GC operations](https://docs.pingcap.com/tidb/stable/garbage-collection-overview)をブロックします。
-   `FAILED`: レプリケーションタスクが失敗しました。いくつかのエラーのため、レプリケーションタスクは再開できず、自動的に回復することはできません。インクリメンタルデータのガベージコレクション（GC）の前に問題が解決される場合、失敗したchangefeedを手動で再開できます。インクリメンタルデータのデフォルトのTime-To-Live（TTL）期間は24時間であり、これはchangefeedが中断されてから24時間以内にGCメカニズムがデータを削除しないことを意味します。
