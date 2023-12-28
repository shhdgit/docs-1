---
title: Migrate Data from TiDB to MySQL-compatible Databases
summary: Learn how to migrate data from TiDB to MySQL-compatible databases.
---

# TiDBからMySQL互換のデータベースにデータを移行する {#migrate-data-from-tidb-to-mysql-compatible-databases}

このドキュメントでは、TiDBクラスタからAurora、MySQL、MariaDBなどのMySQL互換のデータベースにデータを移行する方法について説明します。全体のプロセスは以下の4つのステップで構成されています。

1. 環境をセットアップする。
2. フルデータを移行する。
3. 増分データを移行する。
4. サービスをMySQL互換のクラスタに移行する。

## ステップ1. 環境をセットアップする {#step-1-set-up-the-environment}

1. 上流のTiDBクラスタをデプロイする。

   TiUP Playgroundを使用してTiDBクラスタをデプロイします。詳細については、[TiUPを使用してオンラインTiDBクラスタをデプロイおよび管理する](/tiup/tiup-cluster.md)を参照してください。

   ```shell
   # TiDBクラスタを作成する
   tiup playground --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
   # クラスタのステータスを表示する
   tiup status
   ```

2. 下流のMySQLインスタンスをデプロイする。

   - ラボ環境では、次のコマンドを実行してDockerを使用してMySQLインスタンスを迅速にデプロイできます。

     ```shell
     docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 -d mysql
     ```

   - 本番環境では、[MySQLのインストール](https://dev.mysql.com/doc/refman/8.0/en/installing.html)の手順に従ってMySQLインスタンスをデプロイできます。

3. サービスのワークロードをシミュレートする。

   ラボ環境では、`go-tpc`を使用してデータをTiDBクラスタに書き込みます。これにより、TiDBクラスタでイベントの変更が生成されます。次のコマンドを実行して、TiDBクラスタに`tpcc`という名前のデータベースを作成し、TiUP benchを使用してこのデータベースにデータを書き込みます。

   ```shell
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 prepare
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 run --time 300s
   ```

   `go-tpc`の詳細については、[TiDBでTPC-Cテストを実行する方法](/benchmark/benchmark-tidb-using-tpcc.md)を参照してください。

## ステップ2. フルデータを移行する {#step-2-migrate-full-data}

環境をセットアップしたら、[Dumpling](/dumpling-overview.md)を使用して上流のTiDBクラスタからフルデータをエクスポートできます。

> **Note:**
>
> 本番クラスタでGCを無効にしてバックアップを実行すると、クラスタのパフォーマンスに影響を与える可能性があります。このステップは、オフピーク時に実行することをお勧めします。

1. ガベージコレクション（GC）を無効にする。

   インクリメンタルマイグレーション中に新しく書き込まれたデータが削除されないようにするために、エクスポートする前にアップストリームクラスターのGCを無効にする必要があります。これにより、履歴データが削除されることはありません。

   GCを無効にするには、次のコマンドを実行します。

   ```sql
   MySQL [test]> SET GLOBAL tidb_gc_enable=FALSE;
   ```

       クエリが実行されました。0行が変更されました (0.01 sec)

   変更が有効になったことを確認するには、`tidb_gc_enable`の値をクエリします。

   ```sql
   MySQL [test]> SELECT @@global.tidb_gc_enable;
   ```

       +-------------------------+：
       | @@global.tidb_gc_enable |
       +-------------------------+
       |                       0 |
       +-------------------------+
       1 行が返されました (0.00 sec)

2. データをバックアップする。

   1. Dumplingを使用してSQL形式でデータをエクスポートする：

      ```shell
      tiup dumpling -u root -P 4000 -h 127.0.0.1 --filetype sql -t 8 -o ./dumpling_output -r 200000 -F256MiB
      ```

   2. データのエクスポートが完了したら、次のコマンドを実行してメタデータを確認します。メタデータの`Pos`はエクスポートスナップショットのTSOであり、BackupTSとして記録することができます。

      ```shell
      cat dumpling_output/metadata
      ```

          ダンプを開始しました: 2022-06-28 17:49:54
          SHOW MASTER STATUS:
                  Log: tidb-binlog
                  Pos: 434217889191428107
                  GTID:
          ダンプを完了しました: 2022-06-28 17:49:57

3. データをリストアする。

   MyLoader（オープンソースツール）を使用して、データをダウンストリームのMySQLインスタンスにインポートします。MyLoaderのインストール方法や使用方法の詳細については、[MyDumpler/MyLoader](https://github.com/mydumper/mydumper)を参照してください。なお、MyLoader v0.10以前のバージョンを使用する必要があります。より新しいバージョンでは、Dumplingによってエクスポートされたメタデータファイルを処理することができません。

   DumplingによってエクスポートされたフルデータをMySQLにインポートするには、次のコマンドを実行します。

   ```shell
   myloader -h 127.0.0.1 -P 3306 -d ./dumpling_output/
   ```

4. （オプション）データを検証する。

   [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md)を使用して、アップストリームとダウンストリームの間のデータの整合性を特定の時点でチェックすることができます。

   ```shell
   sync_diff_inspector -C ./config.yaml
   ```

   sync-diff-inspectorの設定方法の詳細については、[Configuration file description](/sync-diff-inspector/sync-diff-inspector-overview.md#configuration-file-description)を参照してください。このドキュメントでは、設定は次のようになります。

   ```toml
   # Diff Configuration.
   ######################### Datasource config #########################
   [data-sources]
   [data-sources.upstream]
           host = "127.0.0.1" # アップストリームクラスターのIPアドレスに置き換えてください
           port = 4000
           user = "root"
           password = ""
           snapshot = "434217889191428107" # バックアップ時刻（[Step 2. Migrate full data](#step-2-migrate-full-data)の「バックアップ時刻」）を設定します
   [data-sources.downstream]
           host = "127.0.0.1" # ダウンストリームクラスターのIPアドレスに置き換えてください
           port = 3306
           user = "root"
           password = ""
   ######################### Task config #########################
   [task]
           output-dir = "./output"
           source-instances = ["upstream"]
           target-instance = "downstream"
           target-check-tables = ["*.*"]
   ```

## Step 3. インクリメンタルデータをマイグレーションする {#step-3-migrate-incremental-data}

1. TiCDCをデプロイします。

   フルデータ移行が完了した後、インクリメンタルデータをレプリケートするためにTiCDCクラスタをデプロイして設定します。本番環境では、[TiCDCをデプロイする](/ticdc/deploy-ticdc.md)の指示に従ってTiCDCをデプロイします。このドキュメントでは、テストクラスタの作成時にTiCDCノードが起動されているため、TiCDCをデプロイする手順をスキップし、次のステップでchangefeedを作成することができます。

2. changefeedを作成します。

   上流クラスタで、以下のコマンドを実行して、上流から下流クラスタへのchangefeedを作成します：

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed create --server=http://127.0.0.1:8300 --sink-uri="mysql://root:@127.0.0.1:3306" --changefeed-id="upstream-to-downstream" --start-ts="434217889191428107"
   ```

   このコマンドでは、以下のパラメータが使用されます：

   - `--server`：TiCDCクラスタ内の任意のノードのIPアドレス
   - `--sink-uri`：下流クラスタのURI
   - `--changefeed-id`：changefeedのIDで、正規表現の形式である必要があります `^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$`
   - `--start-ts`：changefeedの開始タイムスタンプで、バックアップタイム（または「データをバックアップする」セクションの「バックアップTS」）である必要があります

   changefeedの設定についての詳細は、[タスク設定ファイル](/ticdc/ticdc-changefeed-config.md)を参照してください。

3. GCを有効にします。

   TiCDCを使用したインクリメンタル移行では、GCはレプリケートされた履歴データのみを削除します。そのため、changefeedを作成した後、GCを有効にするために以下のコマンドを実行する必要があります。詳細については、[TiCDCガベージコレクション（GC）セーフポイントの完全な動作は何ですか](/ticdc/ticdc-faq.md#what-is-the-complete-behavior-of-ticdc-garbage-collection-gc-safepoint)を参照してください。

   GCを有効にするには、以下のコマンドを実行します：

   ```sql
   MySQL [test]> SET GLOBAL tidb_gc_enable=TRUE;
   ```

       クエリが実行されました。0行が変更されました（0.01秒）。

   変更が有効になったことを確認するには、`tidb_gc_enable`の値をクエリします：

   ```sql
   MySQL [test]> SELECT @@global.tidb_gc_enable;
   ```

       +-------------------------+
       | @@global.tidb_gc_enable |
       +-------------------------+
       |                       1 |
       +-------------------------+
       1行が返されました（0.00秒）。

## ステップ4。 サービスを移行する {#step-4-migrate-services}

changefeedを作成した後、上流クラスタに書き込まれたデータは低レイテンシで下流クラスタにレプリケートされます。読み取りトラフィックを下流クラスタに段階的に移行することができます。一定期間観察します。下流クラスタが安定している場合は、以下の手順で書き込みトラフィックを下流クラスタに移行することができます：

1. 上流クラスタで書き込みサービスを停止します。changefeedを停止する前に、すべての上流データが下流にレプリケートされていることを確認してください。

   ```shell
   # 上流クラスタから下流クラスタへのchangefeedを停止します
   tiup cdc cli changefeed pause -c "upstream-to-downstream" --pd=http://172.16.6.122:2379
   # changefeedのステータスを表示します
   tiup cdc cli changefeed list
   ```

       [
         {
           "id": "upstream-to-downstream",
           "summary": {
           "state": "stopped",  # ステータスがstoppedであることを確認します
           "tso": 434218657561968641,
           "checkpoint": "2022-06-28 18:38:45.685", # この時刻は書き込みを停止した時刻よりも後である必要があります
           "error": null
           }
         }
       ]

2. 書き込みサービスを下流クラスタに移行した後、一定期間観察します。下流クラスタが安定している場合は、上流クラスタを破棄することができます。
