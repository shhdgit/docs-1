---
title: Migrate Data from TiDB to MySQL-compatible Databases
summary: Learn how to migrate data from TiDB to MySQL-compatible databases.
---

# TiDBからMySQL互換のデータベースにデータを移行する {#migrate-data-from-tidb-to-mysql-compatible-databases}

このドキュメントでは、TiDBクラスタからAurora、MySQL、MariaDBなどのMySQL互換のデータベースにデータを移行する方法について説明します。全体のプロセスには、以下の4つのステップが含まれています。

1. 環境をセットアップする。
2. フルデータを移行する。
3. 増分データを移行する。
4. サービスをMySQL互換のクラスタに移行する。

## ステップ1. 環境をセットアップする {#step-1-set-up-the-environment}

1. 上流にTiDBクラスタをデプロイする。

   TiUP Playgroundを使用してTiDBクラスタをデプロイします。詳細については、[TiUPを使用してオンラインTiDBクラスタをデプロイおよび管理する](/tiup/tiup-cluster.md)を参照してください。

   ```shell
   # TiDBクラスタを作成
   tiup playground --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
   # クラスタのステータスを表示
   tiup status
   ```

2. 下流にMySQLインスタンスをデプロイする。

   - ラボ環境では、次のコマンドを実行してDockerを使用して素早くMySQLインスタンスをデプロイできます。

     ```shell
     docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 -d mysql
     ```

   - 本番環境では、[MySQLのインストール](https://dev.mysql.com/doc/refman/8.0/en/installing.html)の手順に従ってMySQLインスタンスをデプロイできます。

3. サービスのワークロードをシミュレートする。

   ラボ環境では、`go-tpc`を使用してTiDBクラスタにデータを書き込むことができます。これにより、TiDBクラスタでイベントの変更が生成されます。次のコマンドを実行して、TiDBクラスタに`tpcc`という名前のデータベースを作成し、その後、このデータベースにデータを書き込むためにTiUP benchを使用します。

   ```shell
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 prepare
   tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 run --time 300s
   ```

   `go-tpc`の詳細については、[TiDBでTPC-Cテストを実行する方法](/benchmark/benchmark-tidb-using-tpcc.md)を参照してください。

## ステップ2. フルデータの移行 {#step-2-migrate-full-data}

環境を設定した後、[Dumpling](/dumpling-overview.md)を使用して、上流のTiDBクラスタからフルデータをエクスポートできます。

> **注意:**
>
> 本番クラスタでは、GCを無効にしてバックアップを実行すると、クラスタのパフォーマンスに影響を与える場合があります。このステップは、オフピーク時に完了することをお勧めします。

1. ガベージコレクション（GC）を無効にします。

   インクリメンタルな移行中に新しく書き込まれたデータが削除されないようにするために、フルデータをエクスポートする前に、上流クラスタのGCを無効にする必要があります。これにより、履歴データが削除されなくなります。

   次のコマンドを実行して、GCを無効にします。

   ```sql
   MySQL [test]> SET GLOBAL tidb_gc_enable=FALSE;
   ```

   ```
   クエリ OK、0 行が変更されました (0.01 秒)
   ```

   変更が有効になっていることを確認するには、`tidb_gc_enable`の値をクエリしてください。

   ```sql
   MySQL [test]> SELECT @@global.tidb_gc_enable;
   ```

   ```
   +-------------------------+：
   | @@global.tidb_gc_enable |
   +-------------------------+
   |                       0 |
   +-------------------------+
   1 行が返されました (0.00 秒)
   ```

2. データをバックアップします。

   1. Dumplingを使用してSQL形式でデータをエクスポートします。

      ```shell
      tiup dumpling -u root -P 4000 -h 127.0.0.1 --filetype sql -t 8 -o ./dumpling_output -r 200000 -F256MiB
      ```

   2. データのエクスポートが完了したら、次のコマンドを実行してメタデータを確認します。メタデータの`Pos`はエクスポートスナップショットのTSOであり、BackupTSとして記録できます。

      ```shell
      cat dumpling_output/metadata
      ```

      ```
      Started dump at: 2022-06-28 17:49:54
      SHOW MASTER STATUS:
              Log: tidb-binlog
              Pos: 434217889191428107
              GTID:
      Finished dump at: 2022-06-28 17:49:57
      ```

3. データを復元します。

   MyLoader（オープンソースツール）を使用して、データを下流のMySQLインスタンスにインポートします。MyLoaderのインストール方法や使用方法の詳細については、[MyDumpler/MyLoader](https://github.com/mydumper/mydumper)を参照してください。なお、MyLoader v0.10またはそれ以前のバージョンを使用する必要があります。それ以降のバージョンでは、Dumplingによってエクスポートされたメタデータファイルを処理できません。

   DumplingによってエクスポートされたフルデータをMySQLにインポートするには、次のコマンドを実行してください。

   ```shell
   myloader -h 127.0.0.1 -P 3306 -d ./dumpling_output/
   ```

4. （オプション）データを検証します。

   [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md)を使用して、特定の時点で上流と下流のデータの整合性を確認できます。

   ```shell
   sync_diff_inspector -C ./config.yaml
   ```

   sync-diff-inspectorの構成方法の詳細については、[Configuration file description](/sync-diff-inspector/sync-diff-inspector-overview.md#configuration-file-description)を参照してください。このドキュメントでは、次のように構成されています：

   ```toml
   # Diff Configuration.
   ######################### Datasource config #########################
   [data-sources]
   [data-sources.upstream]
           host = "127.0.0.1" # 上流クラスタのIPアドレスに値を置き換えてください
           port = 4000
           user = "root"
           password = ""
           snapshot = "434217889191428107" # スナップショットを実際のバックアップ時刻に設定します（[ステップ2. フルデータの移行](#step-2-migrate-full-data)の「バックアップデータ」セクションのBackupTS）
   [data-sources.downstream]
           host = "127.0.0.1" # 下流クラスタのIPアドレスに値を置き換えてください
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

## ステップ3. 増分データの移行 {#step-3-migrate-incremental-data}

1. TiCDCをデプロイします。

   完全なデータ移行が完了した後、TiCDCクラスタをデプロイして構成し、増分データをレプリケートします。 本番環境では、[TiCDCのデプロイ](/ticdc/deploy-ticdc.md)で指示されたようにTiCDCをデプロイします。 このドキュメントでは、テストクラスタの作成時にTiCDCノードが起動されています。したがって、TiCDCをデプロイする手順をスキップし、次のステップに進んで変更フィードを作成できます。

2. 変更フィードを作成します。

   上流クラスタで、次のコマンドを実行して上流から下流クラスタに変更フィードを作成します。

   ```shell
   tiup cdc:v<CLUSTER_VERSION> cli changefeed create --server=http://127.0.0.1:8300 --sink-uri="mysql://root:@127.0.0.1:3306" --changefeed-id="upstream-to-downstream" --start-ts="434217889191428107"
   ```

   このコマンドでは、次のパラメータがあります。

   - `--server`: TiCDCクラスタ内の任意のノードのIPアドレス
   - `--sink-uri`: 下流クラスタのURI
   - `--changefeed-id`: 変更フィードID、正規表現の形式である必要があります、`^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$`
   - `--start-ts`: 変更フィードの開始タイムスタンプ、バックアップタイム（または[ステップ2. 完全なデータの移行](#step-2-migrate-full-data)の「データのバックアップ」セクションのBackupTS）である必要があります

   変更フィードの構成の詳細については、[タスク構成ファイル](/ticdc/ticdc-changefeed-config.md)を参照してください。

3. GCを有効にします。

   TiCDCを使用した増分移行では、GCはレプリケートされた履歴データのみを削除します。したがって、変更フィードを作成した後、次のコマンドを実行してGCを有効にする必要があります。詳細については、[TiCDCガベージコレクション（GC）セーフポイントの完全な動作は何ですか](/ticdc/ticdc-faq.md#what-is-the-complete-behavior-of-ticdc-garbage-collection-gc-safepoint)を参照してください。

   GCを有効にするには、次のコマンドを実行します。

   ```sql
   MySQL [test]> SET GLOBAL tidb_gc_enable=TRUE;
   ```

   ```
   クエリが正常に実行されました。 (0.01 sec)
   ```

   変更が有効になったことを確認するには、`tidb_gc_enable`の値をクエリしてください。

   ```sql
   MySQL [test]> SELECT @@global.tidb_gc_enable;
   ```

   ```
   +-------------------------+
   | @@global.tidb_gc_enable |
   +-------------------------+
   |                       1 |
   +-------------------------+
   1行が返されました (0.00 sec)
   ```

## ステップ4. サービスの移行 {#step-4-migrate-services}

変更フィードを作成した後、上流クラスタに書き込まれたデータは低レイテンシで下流クラスタにレプリケートされます。読み取りトラフィックを下流クラスタに徐々に移行できます。一定期間読み取りトラフィックを観察します。下流クラスタが安定している場合、次の手順で書き込みトラフィックを下流クラスタにも移行できます。

1. 上流クラスタで書き込みサービスを停止します。変更フィードを停止する前に、すべての上流データが下流にレプリケートされていることを確認してください。

   ```shell
   # 上流クラスタから下流クラスタへの変更フィードを停止します
   tiup cdc cli changefeed pause -c "upstream-to-downstream" --pd=http://172.16.6.122:2379
   # 変更フィードのステータスを表示します
   tiup cdc cli changefeed list
   ```

   ```
   [
     {
       "id": "upstream-to-downstream",
       "summary": {
       "state": "stopped",  # ステータスが停止していることを確認してください
       "tso": 434218657561968641,
       "checkpoint": "2022-06-28 18:38:45.685", # この時間は書き込みを停止した時間よりも後である必要があります
       "error": null
       }
     }
   ]
   ```

2. 書き込みサービスを下流クラスタに移行した後、一定期間観察します。下流クラスタが安定している場合、上流クラスタを破棄できます。"
