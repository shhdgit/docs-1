---
title: Scale a TiDB Cluster Using TiUP
summary: Learn how to scale the TiDB cluster using TiUP.
---

# TiUPを使用してTiDBクラスターをスケールする {#scale-a-tidb-cluster-using-tiup}

TiDBクラスターの容量を増減することで、オンラインサービスを中断することなく行うことができます。

このドキュメントでは、TiUPを使用してTiDB、TiKV、PD、TiCDC、またはTiFlashクラスターをスケールする方法について説明します。TiUPをインストールしていない場合は、[Step 2. デプロイマシンにTiUPをデプロイする](/production-deployment-using-tiup.md#step-2-deploy-tiup-on-the-control-machine)の手順に従ってください。

現在のクラスター名のリストを表示するには、`tiup cluster list`を実行します。

例えば、クラスターの元のトポロジーが以下のような場合：

| ホストIP    | サービス           |
| :------- | :------------- |
| 10.0.1.3 | TiDB + TiFlash |
| 10.0.1.4 | TiDB + PD      |
| 10.0.1.5 | TiKV + Monitor |
| 10.0.1.1 | TiKV           |
| 10.0.1.2 | TiKV           |

## TiDB/PD/TiKVクラスターをスケールアウトする {#scale-out-a-tidb-pd-tikv-cluster}

このセクションでは、`10.0.1.5`ホストにTiDBノードを追加する方法を示します。

> **Note:**
>
> PDノードを追加する場合も同様の手順を実行できます。TiKVノードを追加する前に、クラスターの負荷に応じて事前にPDスケジューリングパラメータを調整することをお勧めします。

1. スケールアウトのトポロジを設定します：

   > **Note:**
   >
   > - ポートとディレクトリ情報はデフォルトでは必要ありません。
   > - 1つのマシンに複数のインスタンスがデプロイされている場合、それぞれに異なるポートとディレクトリを割り当てる必要があります。ポートやディレクトリに競合がある場合、デプロイやスケーリング中に通知が表示されます。
   > - TiUP v1.0.0以降、スケールアウトの設定は元のクラスタのグローバル設定を継承します。

   `scale-out.yml`ファイルにスケールアウトのトポロジ設定を追加します：

   ```shell
   vi scale-out.yml
   ```

   ```ini
   tidb_servers:
   - host: 10.0.1.5
     ssh_port: 22
     port: 4000
     status_port: 10080
     deploy_dir: /tidb-deploy/tidb-4000
     log_dir: /tidb-deploy/tidb-4000/log
   ```

   以下はTiKVの設定ファイルのテンプレートです：

   ```ini
   tikv_servers:
   - host: 10.0.1.5
     ssh_port: 22
     port: 20160
     status_port: 20180
     deploy_dir: /tidb-deploy/tikv-20160
     data_dir: /tidb-data/tikv-20160
     log_dir: /tidb-deploy/tikv-20160/log
   ```

   以下はPDの設定ファイルのテンプレートです：

   ```ini
   pd_servers:
   - host: 10.0.1.5
     ssh_port: 22
     name: pd-1
     client_port: 2379
     peer_port: 2380
     deploy_dir: /tidb-deploy/pd-2379
     data_dir: /tidb-data/pd-2379
     log_dir: /tidb-deploy/pd-2379/log
   ```

   現在のクラスタの設定を表示するには、`tiup cluster edit-config <cluster-name>`を実行します。`global`と`server_configs`のパラメータ設定は`scale-out.yml`に継承されるため、`scale-out.yml`でも有効になります。

2. スケールアウトコマンドを実行します：

   `scale-out`コマンドを実行する前に、`check`コマンドと`check --apply`コマンドを使用してクラスタ内の潜在的なリスクを検出し、自動的に修復します：

   1. 潜在的なリスクをチェックします：

      ```shell
      tiup cluster check <cluster-name> scale-out.yml --cluster --user root [-p] [-i /home/root/.ssh/gcp_rsa]
      ```

   2. 自動修復を有効にします：

      ```shell
      tiup cluster check <cluster-name> scale-out.yml --cluster --apply --user root [-p] [-i /home/root/.ssh/gcp_rsa]
      ```

   3. `scale-out`コマンドを実行します：

      ```shell
      tiup cluster scale-out <cluster-name> scale-out.yml [-p] [-i /home/root/.ssh/gcp_rsa]
      ```

   上記のコマンドでは：

   - `scale-out.yml`はスケールアウトの設定ファイルです。
   - `--user root`は、クラスタのスケールアウトを完了するために、ターゲットマシンに`root`ユーザーとしてログインすることを示します。`root`ユーザーは、ターゲットマシンに対して`ssh`と`sudo`の権限を持っていることが期待されます。代わりに、`ssh`と`sudo`の権限を持つ他のユーザーを使用してデプロイを完了することもできます。
   - `[-i]`と`[-p]`はオプションです。パスワードなしでターゲットマシンにログインするように設定した場合、これらのパラメータは必要ありません。そうでない場合は、2つのパラメータのうちいずれかを選択します。`[-i]`は、ターゲットマシンにアクセスできる`root`ユーザー（または`--user`で指定された他のユーザー）の秘密鍵です。`[-p]`は、ユーザーのパスワードを対話的に入力するために使用されます。

   `Scaled cluster <cluster-name> out successfully`と表示された場合、スケールアウト操作は成功です。

3. クラスタの状態を確認します：

   ```shell
   tiup cluster display <cluster-name>
   ```

   ブラウザを使用して<http://10.0.1.5:3000>にアクセスし、クラスタと新しいノードの状態を監視するためのモニタリングプラットフォームにアクセスします。

スケールアウト後、クラスタのトポロジは次のようになります：

| ホストIP    | サービス                      |
| :------- | :------------------------ |
| 10.0.1.3 | TiDB + TiFlash            |
| 10.0.1.4 | TiDB + PD                 |
| 10.0.1.5 | **TiDB** + TiKV + Monitor |
| 10.0.1.1 | TiKV                      |
| 10.0.1.2 | TiKV                      |

## TiFlashクラスターのスケールアウト {#scale-out-a-tiflash-cluster}

このセクションでは、`10.0.1.4`ホストにTiFlashノードを追加する方法を示します。

> **Note:**
>
> 既存のTiDBクラスターにTiFlashノードを追加する場合は、次の点に注意してください。
>
> - 現在のTiDBバージョンがTiFlashを使用できることを確認してください。そうでない場合は、TiDBクラスターをv5.0またはそれ以降のバージョンにアップグレードしてください。
> - `tiup ctl:v<CLUSTER_VERSION> pd -u http://<pd_ip>:<pd_port> config set enable-placement-rules true`コマンドを実行して、Placement Rules機能を有効にします。または、[pd-ctl](/pd-control.md)で対応するコマンドを実行します。

1. `scale-out.yml`ファイルにノード情報を追加します。

   TiFlashノード情報を追加するには、`scale-out.yml`ファイルを作成します。

   ```ini
   tiflash_servers:
   - host: 10.0.1.4
   ```

   現在、IPアドレスのみを追加でき、ドメイン名は追加できません。

2. スケールアウトコマンドを実行します。

   ```shell
   tiup cluster scale-out <cluster-name> scale-out.yml
   ```

   > **Note:**
   >
   > 前提として、コマンドを実行するユーザーと新しいマシンの間で相互信頼が設定されていることを前提としています。相互信頼を設定できない場合は、`-p`オプションを使用して新しいマシンのパスワードを入力するか、`-i`オプションを使用して秘密鍵ファイルを指定します。

3. クラスターのステータスを表示します。

   ```shell
   tiup cluster display <cluster-name>
   ```

   ブラウザで<http://10.0.1.5:3000>にアクセスし、クラスターと新しいノードのステータスを表示します。

スケールアウト後、クラスタートポロジは次のようになります。

| ホストIP    | サービス                    |
| :------- | :---------------------- |
| 10.0.1.3 | TiDB + TiFlash          |
| 10.0.1.4 | TiDB + PD + **TiFlash** |
| 10.0.1.5 | TiDB+ TiKV + Monitor    |
| 10.0.1.1 | TiKV                    |
| 10.0.1.2 | TiKV                    |

## TiCDCクラスターのスケールアウト {#scale-out-a-ticdc-cluster}

このセクションでは、`10.0.1.3`および`10.0.1.4`ホストに2つのTiCDCノードを追加する方法を示します。

1. `scale-out.yml`ファイルにノード情報を追加します。

   TiCDCノード情報を追加するには、`scale-out.yml`ファイルを作成します。

   ```ini
   cdc_servers:
     - host: 10.0.1.3
       gc-ttl: 86400
       data_dir: /tidb-data/cdc-8300
     - host: 10.0.1.4
       gc-ttl: 86400
       data_dir: /tidb-data/cdc-8300
   ```

2. スケールアウトコマンドを実行します。

   ```shell
   tiup cluster scale-out <cluster-name> scale-out.yml
   ```

   > **Note:**
   >
   > 前提として、コマンドを実行するユーザーと新しいマシンの間で相互信頼が設定されていることを前提としています。相互信頼を設定できない場合は、`-p`オプションを使用して新しいマシンのパスワードを入力するか、`-i`オプションを使用して秘密鍵ファイルを指定します。

3. クラスターのステータスを表示します。

   ```shell
   tiup cluster display <cluster-name>
   ```

   ブラウザで<http://10.0.1.5:3000>にアクセスし、クラスターと新しいノードのステータスを表示します。

スケールアウト後、クラスタートポロジは次のようになります。

| ホストIP    | サービス                            |
| :------- | :------------------------------ |
| 10.0.1.3 | TiDB + TiFlash + **TiCDC**      |
| 10.0.1.4 | TiDB + PD + TiFlash + **TiCDC** |
| 10.0.1.5 | TiDB+ TiKV + Monitor            |
| 10.0.1.1 | TiKV                            |
| 10.0.1.2 | TiKV                            |

## TiDB/PD/TiKVクラスターのスケールイン {#scale-in-a-tidb-pd-tikv-cluster}

このセクションでは、`10.0.1.5`ホストからTiKVノードを削除する方法を示します。

> **Note:**
>
> - TiDBやPDノードを削除するには、同様の手順を実行できます。
> - TiKV、TiFlash、およびTiDB Binlogコンポーネントは非同期でオフラインになり、停止プロセスには長い時間がかかるため、TiUPは異なる方法でオフラインにします。詳細については、[コンポーネントのオフラインプロセスの特別な処理](/tiup/tiup-component-cluster-scale-in.md#particular-handling-of-components-offline-process)を参照してください。
> - TiKVのPDクライアントは、PDノードのリストをキャッシュします。現在のバージョンのTiKVには、PDノードを自動的に定期的に更新するメカニズムがあり、TiKVによってキャッシュされたPDノードのリストの有効期限が切れる問題を緩和するのに役立ちます。ただし、PDをスケールアウトした後は、スケール前に存在したすべてのPDノードを一度に直接削除することを避けるようにしてください。必要に応じて、以前に存在したすべてのPDノードをオフラインにする前に、PDリーダーを新しく追加されたPDノードに切り替えるようにしてください。

1. ノードID情報を表示する：

   ```shell
   tiup cluster display <cluster-name>
   ```

       Starting /root/.tiup/components/cluster/v1.11.3/cluster display <cluster-name>
       TiDB Cluster: <cluster-name>
       TiDB Version: v7.1.3
       ID              Role         Host        Ports                            Status  Data Dir                Deploy Dir
       --              ----         ----        -----                            ------  --------                ----------
       10.0.1.3:8300   cdc          10.0.1.3    8300                             Up      data/cdc-8300           deploy/cdc-8300
       10.0.1.4:8300   cdc          10.0.1.4    8300                             Up      data/cdc-8300           deploy/cdc-8300
       10.0.1.4:2379   pd           10.0.1.4    2379/2380                        Healthy data/pd-2379            deploy/pd-2379
       10.0.1.1:20160  tikv         10.0.1.1    20160/20180                      Up      data/tikv-20160         deploy/tikv-20160
       10.0.1.2:20160  tikv         10.0.1.2    20160/20180                      Up      data/tikv-20160         deploy/tikv-20160
       10.0.1.5:20160  tikv         10.0.1.5    20160/20180                      Up      data/tikv-20160         deploy/tikv-20160
       10.0.1.3:4000   tidb         10.0.1.3    4000/10080                       Up      -                       deploy/tidb-4000
       10.0.1.4:4000   tidb         10.0.1.4    4000/10080                       Up      -                       deploy/tidb-4000
       10.0.1.5:4000   tidb         10.0.1.5    4000/10080                       Up      -                       deploy/tidb-4000
       10.0.1.3:9000   tiflash      10.0.1.3    9000/8123/3930/20170/20292/8234  Up      data/tiflash-9000       deploy/tiflash-9000
       10.0.1.4:9000   tiflash      10.0.1.4    9000/8123/3930/20170/20292/8234  Up      data/tiflash-9000       deploy/tiflash-9000
       10.0.1.5:9090   prometheus   10.0.1.5    9090                             Up      data/prometheus-9090    deploy/prometheus-9090
       10.0.1.5:3000   grafana      10.0.1.5    3000                             Up      -                       deploy/grafana-3000
       10.0.1.5:9093   alertmanager 10.0.1.5    9093/9294                        Up      data/alertmanager-9093  deploy/alertmanager-9093

2. スケールインコマンドを実行する：

   ```shell
   tiup cluster scale-in <cluster-name> --node 10.0.1.5:20160
   ```

   `--node`パラメータはオフラインにするノードのIDです。

   `Scaled cluster <cluster-name> in successfully`が表示されたら、スケールイン操作は成功です。

3. クラスターの状態を確認する：

   スケールインプロセスには時間がかかります。次のコマンドを実行してスケールインの状態を確認できます：

   ```shell
   tiup cluster display <cluster-name>
   ```

   スケールインするノードが`Tombstone`になったら、スケールイン操作は成功です。

   ブラウザで<http://10.0.1.5:3000>にアクセスし、クラスターの状態を確認できます。

現在のトポロジは以下の通りです：

| ホストIP    | サービス                              |
| :------- | :-------------------------------- |
| 10.0.1.3 | TiDB + TiFlash + TiCDC            |
| 10.0.1.4 | TiDB + PD + TiFlash + TiCDC       |
| 10.0.1.5 | TiDB + Monitor **(TiKVが削除されました)** |
| 10.0.1.1 | TiKV                              |
| 10.0.1.2 | TiKV                              |

## TiFlashクラスターのスケールイン {#scale-in-a-tiflash-cluster}

このセクションでは、`10.0.1.4`ホストからTiFlashノードを削除する方法を示します。

### 1. 残りのTiFlashノードの数に応じて、テーブルのレプリカの数を調整する {#1-adjust-the-number-of-replicas-of-the-tables-according-to-the-number-of-remaining-tiflash-nodes}

1. スケールイン後のTiFlashノードの数よりも多くのTiFlashレプリカを持つテーブルがあるかどうかをクエリします。`tobe_left_nodes`はスケールイン後のTiFlashノードの数を表します。クエリ結果が空の場合、TiFlashのスケールインを開始できます。クエリ結果が空でない場合は、関連するテーブルのTiFlashレプリカの数を変更する必要があります。

   ```sql
   SELECT * FROM information_schema.tiflash_replica WHERE REPLICA_COUNT >  'tobe_left_nodes';
   ```

2. スケールイン後のTiFlashノードの数よりも多くのTiFlashレプリカを持つすべてのテーブルに対して、次のステートメントを実行します。`new_replica_num`は`tobe_left_nodes`以下である必要があります。

   ```sql
   ALTER TABLE <db-name>.<table-name> SET tiflash replica 'new_replica_num';
   ```

3. ステップ1を再度実行し、スケールイン後のTiFlashノードの数よりも多くのTiFlashレプリカを持つテーブルがないことを確認します。

### 2. スケールイン操作を実行する {#2-perform-the-scale-in-operation}

次のいずれかのソリューションを使用してスケールイン操作を実行します。

#### ソリューション1. TiUPを使用してTiFlashノードを削除する {#solution-1-use-tiup-to-remove-a-tiflash-node}

1. ダウンさせるノードの名前を確認します。

   ```shell
   tiup cluster display <cluster-name>
   ```

2. TiFlashノードを削除します（ノード名がStep 1で`10.0.1.4:9000`であると仮定します）。

   ```shell
   tiup cluster scale-in <cluster-name> --node 10.0.1.4:9000
   ```

#### ソリューション2. 手動でTiFlashノードを削除する {#solution-2-manually-remove-a-tiflash-node}

特殊な場合（ノードを強制的にダウンさせる必要がある場合など）や、TiUPのスケールイン操作が失敗した場合は、次の手順で手動でTiFlashノードを削除できます。

1. pd-ctlのstoreコマンドを使用して、このTiFlashノードに対応するストアIDを表示します。

   - [pd-ctl](/pd-control.md)でstoreコマンドを入力します（バイナリファイルはtidb-ansibleディレクトリの`resources/bin`にあります）。

   - TiUPデプロイメントを使用する場合は、`pd-ctl`を`tiup ctl:v<CLUSTER_VERSION> pd`に置き換えます。

   ```shell
   tiup ctl:v<CLUSTER_VERSION> pd -u http://<pd_ip>:<pd_port> store
   ```

   > **Note:**
   >
   > クラスターに複数のPDインスタンスが存在する場合、上記のコマンドでアクティブなPDインスタンスのIPアドレス：ポートを指定するだけです。

2. pd-ctlでTiFlashノードを削除します。

   - pd-ctlで`store delete <store_id>`を入力します（`<store_id>`は前のステップで見つかったTiFlashノードのストアIDです）。

   - TiUPデプロイメントを使用する場合は、`pd-ctl`を`tiup ctl:v<CLUSTER_VERSION> pd`に置き換えます。

     ```shell
     tiup ctl:v<CLUSTER_VERSION> pd -u http://<pd_ip>:<pd_port> store delete <store_id>
     ```

   > **Note:**
   >
   > クラスターに複数のPDインスタンスが存在する場合、上記のコマンドでアクティブなPDインスタンスのIPアドレス：ポートを指定するだけです。

3. TiFlashノードのストアが消えるか、`state_name`が`Tombstone`になるまで待ちます。その後、TiFlashプロセスを停止します。

4. TiFlashデータファイルを手動で削除します（場所はクラスタートポロジーファイルのTiFlash構成の`data_dir`ディレクトリにあります）。

5. 次のコマンドを使用して、ダウンしたTiFlashノードに関するクラスタートポロジーの情報を削除します。

   ```shell
   tiup cluster scale-in <cluster-name> --node <pd_ip>:<pd_port> --force
   ```

> **Note:**
>
> すべてのクラスタのTiFlashノードが停止する前に、TiFlashにレプリケートされたすべてのテーブルがキャンセルされていない場合、PDでレプリケーションルールを手動でクリーンアップする必要があります。そうしないと、TiFlashノードを正常にダウンさせることができません。

PDでレプリケーションルールを手動でクリーンアップする手順は以下の通りです：

1. 現在のPDインスタンスで、TiFlashに関連するすべてのデータレプリケーションルールを表示します：

   ```shell
   curl http://<pd_ip>:<pd_port>/pd/api/v1/config/rules/group/tiflash
   ```

       [
         {
           "group_id": "tiflash",
           "id": "table-45-r",
           "override": true,
           "start_key": "7480000000000000FF2D5F720000000000FA",
           "end_key": "7480000000000000FF2E00000000000000F8",
           "role": "learner",
           "count": 1,
           "label_constraints": [
             {
               "key": "engine",
               "op": "in",
               "values": [
                 "tiflash"
               ]
             }
           ]
         }
       ]

2. TiFlashに関連するすべてのデータレプリケーションルールを削除します。`id`が`table-45-r`のルールを例として、以下のコマンドで削除します：

   ```shell
   curl -v -X DELETE http://<pd_ip>:<pd_port>/pd/api/v1/config/rule/tiflash/table-45-r
   ```

3. クラスタのステータスを表示します：

   ```shell
   tiup cluster display <cluster-name>
   ```

   ブラウザで<http://10.0.1.5:3000>にアクセスし、クラスタと新しいノードのステータスを表示します。

スケールアウト後、クラスタのトポロジは以下のようになります：

| ホストIP    | サービス                                    |
| :------- | :-------------------------------------- |
| 10.0.1.3 | TiDB + TiFlash + TiCDC                  |
| 10.0.1.4 | TiDB + PD + TiCDC **(TiFlashは削除されました)** |
| 10.0.1.5 | TiDB+ Monitor                           |
| 10.0.1.1 | TiKV                                    |
| 10.0.1.2 | TiKV                                    |

## TiCDCクラスタのスケールイン {#scale-in-a-ticdc-cluster}

このセクションでは、`10.0.1.4`ホストからTiCDCノードを削除する方法を示します。

1. ノードをオフラインにします：

   ```shell
   tiup cluster scale-in <cluster-name> --node 10.0.1.4:8300
   ```

2. クラスタのステータスを表示します：

   ```shell
   tiup cluster display <cluster-name>
   ```

   ブラウザで<http://10.0.1.5:3000>にアクセスし、クラスタのステータスを表示します。

現在のトポロジは以下のようになります：

| ホストIP    | サービス                            |
| :------- | :------------------------------ |
| 10.0.1.3 | TiDB + TiFlash + TiCDC          |
| 10.0.1.4 | TiDB + PD + **(TiCDCは削除されました)** |
| 10.0.1.5 | TiDB + Monitor                  |
| 10.0.1.1 | TiKV                            |
| 10.0.1.2 | TiKV                            |
