---
title: Deploy and Maintain an Online TiDB Cluster Using TiUP
summary: Learns how to deploy and maintain an online TiDB cluster using TiUP.
---

# TiUPを使用してオンラインTiDBクラスターをデプロイおよび管理する {#deploy-and-maintain-an-online-tidb-cluster-using-tiup}

このドキュメントでは、TiUPクラスターコンポーネントの使用方法に焦点を当てます。オンラインデプロイの完全な手順については、[TiUPを使用したTiDBクラスターのデプロイ](/production-deployment-using-tiup.md)を参照してください。

ローカルテストデプロイに使用される[TiUPプレイグラウンドコンポーネント](/tiup/tiup-playground.md)と同様に、TiUPクラスターコンポーネントは本番環境のTiDBを迅速にデプロイします。プレイグラウンドと比較して、クラスターコンポーネントはアップグレード、スケーリング、さらには操作や監査など、より強力な本番クラスター管理機能を提供します。

クラスターコンポーネントのヘルプ情報については、次のコマンドを実行してください：

```bash
tiup cluster
```

    Starting component `cluster`: /home/tidb/.tiup/components/cluster/v1.11.3/cluster
    Deploy a TiDB cluster for production

    Usage:
      tiup cluster [command]

    Available Commands:
      check       Precheck a cluster
      deploy      Deploy a cluster for production
      start       Start a TiDB cluster
      stop        Stop a TiDB cluster
      restart     Restart a TiDB cluster
      scale-in    Scale in a TiDB cluster
      scale-out   Scale out a TiDB cluster
      destroy     Destroy a specified cluster
      clean       (Experimental) Clean up a specified cluster
      upgrade     Upgrade a specified TiDB cluster
      display     Display information of a TiDB cluster
      list        List all clusters
      audit       Show audit log of cluster operation
      import      Import an existing TiDB cluster from TiDB-Ansible
      edit-config Edit TiDB cluster config
      reload      Reload a TiDB cluster's config and restart if needed
      patch       Replace the remote package with a specified package and restart the service
      help        Help about any command

    Flags:
      -c, --concurrency int     Maximum number of concurrent tasks allowed (defaults to `5`)
          --format string       (EXPERIMENTAL) The format of output, available values are [default, json] (default "default")
      -h, --help                help for tiup
          --ssh string          (Experimental) The executor type. Optional values are 'builtin', 'system', and 'none'.
          --ssh-timeout uint    Timeout in seconds to connect a host via SSH. Operations that don't need an SSH connection are ignored. (default 5)
      -v, --version            TiUP version
          --wait-timeout uint   Timeout in seconds to wait for an operation to complete. Inapplicable operations are ignored. (defaults to `120`)
      -y, --yes                 Skip all confirmations and assumes 'yes'

## クラスタのデプロイ {#deploy-the-cluster}

クラスタをデプロイするには、`tiup cluster deploy`コマンドを実行します。コマンドの使用方法は以下の通りです：

```bash
tiup cluster deploy <cluster-name> <version> <topology.yaml> [flags]
```

このコマンドでは、クラスタ名、TiDBクラスタのバージョン（例：`v7.1.3`）、およびクラスタのトポロジーファイルを指定する必要があります。

トポロジーファイルを作成するには、[例](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)を参照してください。以下のファイルは、最も単純なトポロジーの例です。

> **Note:**
>
> デプロイやスケーリングに使用されるTiUPクラスタコンポーネントで使用されるトポロジーファイルは、[yaml](https://yaml.org/spec/1.2/spec.html)の構文で記述されているため、インデントが正しいことを確認してください。

```yaml
---

pd_servers:
  - host: 172.16.5.134
    name: pd-134
  - host: 172.16.5.139
    name: pd-139
  - host: 172.16.5.140
    name: pd-140

tidb_servers:
  - host: 172.16.5.134
  - host: 172.16.5.139
  - host: 172.16.5.140

tikv_servers:
  - host: 172.16.5.134
  - host: 172.16.5.139
  - host: 172.16.5.140

tiflash_servers:
  - host: 172.16.5.141
  - host: 172.16.5.142
  - host: 172.16.5.143

grafana_servers:
  - host: 172.16.5.134

monitoring_servers:
  - host: 172.16.5.134
```

デフォルトでは、TiUPはamd64アーキテクチャで実行されるバイナリファイルとしてデプロイされます。ターゲットマシンがarm64アーキテクチャの場合は、トポロジーファイルで設定することができます。

```yaml
global:
  arch: "arm64"           # Configures all machines to use the binary files of the arm64 architecture by default

tidb_servers:
  - host: 172.16.5.134
    arch: "amd64"         # Configures this machine to use the binary files of the amd64 architecture
  - host: 172.16.5.139
    arch: "arm64"         # Configures this machine to use the binary files of the arm64 architecture
  - host: 172.16.5.140    # Machines that are not configured with the arch field use the default value in the global field, which is arm64 in this case.

...
```

ファイルを `/tmp/topology.yaml` として保存してください。もしTiDB v7.1.3を使用し、クラスター名が `prod-cluster` である場合は、以下のコマンドを実行してください。

```shell
tiup cluster deploy -p prod-cluster v7.1.3 /tmp/topology.yaml
```

実行中、TiUPは再度トポロジーを確認し、ターゲットマシンのrootパスワードを要求します（`-p`フラグはパスワードの入力を意味します）。

```bash
Please confirm your topology:
TiDB Cluster: prod-cluster
TiDB Version: v7.1.3
Type        Host          Ports                            OS/Arch       Directories
----        ----          -----                            -------       -----------
pd          172.16.5.134  2379/2380                        linux/x86_64  deploy/pd-2379,data/pd-2379
pd          172.16.5.139  2379/2380                        linux/x86_64  deploy/pd-2379,data/pd-2379
pd          172.16.5.140  2379/2380                        linux/x86_64  deploy/pd-2379,data/pd-2379
tikv        172.16.5.134  20160/20180                      linux/x86_64  deploy/tikv-20160,data/tikv-20160
tikv        172.16.5.139  20160/20180                      linux/x86_64  deploy/tikv-20160,data/tikv-20160
tikv        172.16.5.140  20160/20180                      linux/x86_64  deploy/tikv-20160,data/tikv-20160
tidb        172.16.5.134  4000/10080                       linux/x86_64  deploy/tidb-4000
tidb        172.16.5.139  4000/10080                       linux/x86_64  deploy/tidb-4000
tidb        172.16.5.140  4000/10080                       linux/x86_64  deploy/tidb-4000
tiflash     172.16.5.141  9000/8123/3930/20170/20292/8234  linux/x86_64  deploy/tiflash-9000,data/tiflash-9000
tiflash     172.16.5.142  9000/8123/3930/20170/20292/8234  linux/x86_64  deploy/tiflash-9000,data/tiflash-9000
tiflash     172.16.5.143  9000/8123/3930/20170/20292/8234  linux/x86_64  deploy/tiflash-9000,data/tiflash-9000
prometheus  172.16.5.134  9090         deploy/prometheus-9090,data/prometheus-9090
grafana     172.16.5.134  3000         deploy/grafana-3000
Attention:
    1. If the topology is not what you expected, check your yaml file.
    2. Please confirm there is no port/directory conflicts in same host.
Do you want to continue? [y/N]:
```

パスワードを入力すると、TiUP clusterは必要なコンポーネントをダウンロードし、それらを対応するマシンにデプロイします。次のメッセージが表示されたら、デプロイは成功です。

```bash
Deployed cluster `prod-cluster` successfully
```

## クラスターリストの表示 {#view-the-cluster-list}

クラスターが正常にデプロイされた後、次のコマンドを実行してクラスターリストを表示します。

```bash
tiup cluster list
```

    Starting /root/.tiup/components/cluster/v1.11.3/cluster list
    Name          User  Version    Path                                               PrivateKey
    ----          ----  -------    ----                                               ----------
    prod-cluster  tidb  v7.1.3    /root/.tiup/storage/cluster/clusters/prod-cluster  /root/.tiup/storage/cluster/clusters/prod-cluster/ssh/id_rsa

## クラスターの開始 {#start-the-cluster}

クラスターが正常にデプロイされた後、次のコマンドを実行してクラスターを開始します。

```shell
tiup cluster start prod-cluster
```

もしクラスターの名前を忘れた場合は、`tiup cluster list`を実行してクラスターのリストを表示します。

TiUPは`systemd`を使用してデーモンプロセスを起動します。プロセスが予期せず終了した場合、15秒後に再起動されます。

## クラスターの状態をチェックする {#check-the-cluster-status}

TiUPは`tiup cluster display`コマンドを提供して、クラスター内の各コンポーネントの状態を表示します。このコマンドを使用すると、各マシンにログインしてコンポーネントの状態を確認する必要はありません。コマンドの使用方法は以下の通りです：

```bash
tiup cluster display prod-cluster
```

    Starting /root/.tiup/components/cluster/v1.11.3/cluster display prod-cluster
    TiDB Cluster: prod-cluster
    TiDB Version: v7.1.3
    ID                  Role        Host          Ports                            OS/Arch       Status  Data Dir              Deploy Dir
    --                  ----        ----          -----                            -------       ------  --------              ----------
    172.16.5.134:3000   grafana     172.16.5.134  3000                             linux/x86_64  Up      -                     deploy/grafana-3000
    172.16.5.134:2379   pd          172.16.5.134  2379/2380                        linux/x86_64  Up|L    data/pd-2379          deploy/pd-2379
    172.16.5.139:2379   pd          172.16.5.139  2379/2380                        linux/x86_64  Up|UI   data/pd-2379          deploy/pd-2379
    172.16.5.140:2379   pd          172.16.5.140  2379/2380                        linux/x86_64  Up      data/pd-2379          deploy/pd-2379
    172.16.5.134:9090   prometheus  172.16.5.134  9090                             linux/x86_64  Up      data/prometheus-9090  deploy/prometheus-9090
    172.16.5.134:4000   tidb        172.16.5.134  4000/10080                       linux/x86_64  Up      -                     deploy/tidb-4000
    172.16.5.139:4000   tidb        172.16.5.139  4000/10080                       linux/x86_64  Up      -                     deploy/tidb-4000
    172.16.5.140:4000   tidb        172.16.5.140  4000/10080                       linux/x86_64  Up      -                     deploy/tidb-4000
    172.16.5.141:9000   tiflash     172.16.5.141  9000/8123/3930/20170/20292/8234  linux/x86_64  Up      data/tiflash-9000     deploy/tiflash-9000
    172.16.5.142:9000   tiflash     172.16.5.142  9000/8123/3930/20170/20292/8234  linux/x86_64  Up      data/tiflash-9000     deploy/tiflash-9000
    172.16.5.143:9000   tiflash     172.16.5.143  9000/8123/3930/20170/20292/8234  linux/x86_64  Up      data/tiflash-9000     deploy/tiflash-9000
    172.16.5.134:20160  tikv        172.16.5.134  20160/20180                      linux/x86_64  Up      data/tikv-20160       deploy/tikv-20160
    172.16.5.139:20160  tikv        172.16.5.139  20160/20180                      linux/x86_64  Up      data/tikv-20160       deploy/tikv-20160
    172.16.5.140:20160  tikv        172.16.5.140  20160/20180                      linux/x86_64  Up      data/tikv-20160       deploy/tikv-20160

`Status`列は、サービスが正常に実行されているかどうかを示すために、`Up`または`Down`を使用します。

PDコンポーネントの場合、`Up`または`Down`の後に`|L`または`|UI`が追加されることがあります。`|L`は、PDノードがリーダーであることを示し、`|UI`は[TiDBダッシュボード](/dashboard/dashboard-intro.md)がPDノードで実行されていることを示します。

## クラスターのスケールイン {#scale-in-a-cluster}

> **Note:**
>
> このセクションでは、スケールインコマンドの構文のみを説明します。オンラインスケーリングの詳細な手順については、[TiUPを使用してTiDBクラスターをスケールする](/scale-tidb-using-tiup.md)を参照してください。

クラスターのスケールインとは、特定のノードをオフラインにすることを意味します。この操作により、指定したノードがクラスターから削除され、残りのファイルが削除されます。

TiKV、TiFlash、およびTiDB Binlogコンポーネントのオフラインプロセスは非同期であるため（ノードをAPI経由で削除する必要があります）、プロセスには長い時間がかかります（ノードが正常にオフラインになったかどうかを継続的に監視する必要があります）。そのため、TiKV、TiFlash、およびTiDB Binlogコンポーネントには特別な処理が行われます。

- TiKV、TiFlash、およびBinlogの場合：

  - TiUPクラスターは、APIを介してノードをオフラインにし、プロセスが完了するのを待たずに直接終了します。
  - その後、クラスター操作に関連するコマンドが実行されると、TiUPクラスターは、オフラインになったTiKV、TiFlash、またはBinlogノードがあるかどうかを調べます。ない場合、TiUPクラスターは指定された操作を続行します。ある場合、TiUPクラスターは次の手順を実行します。

    1. オフラインになったノードのサービスを停止します。
    2. ノードに関連するデータファイルをクリーンアップします。
    3. クラスタートポロジからノードを削除します。

- その他のコンポーネントの場合：

  - PDコンポーネントをダウンさせるとき、TiUPクラスターはAPIを介して指定したノードをクラスターからすばやく削除し、指定したPDノードのサービスを停止し、関連するデータファイルを削除します。
  - その他のコンポーネントをダウンさせるとき、TiUPクラスターはノードのサービスを直接停止し、関連するデータファイルを削除します。

スケールインコマンドの基本的な使用方法：

```bash
tiup cluster scale-in <cluster-name> -N <node-id>
```

このコマンドを使用するには、少なくとも2つのフラグを指定する必要があります：クラスター名とノードID。ノードIDは、前のセクションで`tiup cluster display`コマンドを使用して取得できます。

例えば、`172.16.5.140`のTiKVノードをオフラインにするには、次のコマンドを実行します：

```bash
tiup cluster scale-in prod-cluster -N 172.16.5.140:20160
```

`tiup cluster display`を実行することで、TiKVノードが`Offline`とマークされていることがわかります。

```bash
tiup cluster display prod-cluster
```

    Starting /root/.tiup/components/cluster/v1.11.3/cluster display prod-cluster
    TiDB Cluster: prod-cluster
    TiDB Version: v7.1.3
    ID                  Role        Host          Ports                            OS/Arch       Status   Data Dir              Deploy Dir
    --                  ----        ----          -----                            -------       ------   --------              ----------
    172.16.5.134:3000   grafana     172.16.5.134  3000                             linux/x86_64  Up       -                     deploy/grafana-3000
    172.16.5.134:2379   pd          172.16.5.134  2379/2380                        linux/x86_64  Up|L     data/pd-2379          deploy/pd-2379
    172.16.5.139:2379   pd          172.16.5.139  2379/2380                        linux/x86_64  Up|UI    data/pd-2379          deploy/pd-2379
    172.16.5.140:2379   pd          172.16.5.140  2379/2380                        linux/x86_64  Up       data/pd-2379          deploy/pd-2379
    172.16.5.134:9090   prometheus  172.16.5.134  9090                             linux/x86_64  Up       data/prometheus-9090  deploy/prometheus-9090
    172.16.5.134:4000   tidb        172.16.5.134  4000/10080                       linux/x86_64  Up       -                     deploy/tidb-4000
    172.16.5.139:4000   tidb        172.16.5.139  4000/10080                       linux/x86_64  Up       -                     deploy/tidb-4000
    172.16.5.140:4000   tidb        172.16.5.140  4000/10080                       linux/x86_64  Up       -                     deploy/tidb-4000
    172.16.5.141:9000   tiflash     172.16.5.141  9000/8123/3930/20170/20292/8234  linux/x86_64  Up       data/tiflash-9000     deploy/tiflash-9000
    172.16.5.142:9000   tiflash     172.16.5.142  9000/8123/3930/20170/20292/8234  linux/x86_64  Up       data/tiflash-9000     deploy/tiflash-9000
    172.16.5.143:9000   tiflash     172.16.5.143  9000/8123/3930/20170/20292/8234  linux/x86_64  Up       data/tiflash-9000     deploy/tiflash-9000
    172.16.5.134:20160  tikv        172.16.5.134  20160/20180                      linux/x86_64  Up       data/tikv-20160       deploy/tikv-20160
    172.16.5.139:20160  tikv        172.16.5.139  20160/20180                      linux/x86_64  Up       data/tikv-20160       deploy/tikv-20160
    172.16.5.140:20160  tikv        172.16.5.140  20160/20180                      linux/x86_64  Offline  data/tikv-20160       deploy/tikv-20160

ノード上のデータをPDが他のTiKVノードにスケジュールした後、このノードは自動的に削除されます。

## クラスタのスケールアウト {#scale-out-a-cluster}

> **Note:**
>
> このセクションでは、スケールアウトコマンドの構文のみを説明します。オンラインスケーリングの詳細な手順については、[TiUPを使用してTiDBクラスタをスケールする](/scale-tidb-using-tiup.md)を参照してください。

スケールアウト操作には、デプロイと同様の内部ロジックがあります。まず、TiUPクラスタコンポーネントはノードのSSH接続を確保し、ターゲットノードに必要なディレクトリを作成し、次にデプロイ操作を実行し、ノードサービスを起動します。

PDをスケールアウトすると、ノードは`join`によってクラスタに追加され、PDに関連するサービスの設定が更新されます。他のサービスをスケールアウトすると、サービスは直接起動され、クラスタに追加されます。

すべてのサービスは、スケールアウト時に正当性検証を実施します。検証結果により、スケールアウトが成功したかどうかが表示されます。

`tidb-test`クラスタにTiKVノードとPDノードを追加するには、次の手順を実行します。

1. `scale.yaml`ファイルを作成し、新しいTiKVノードとPDノードのIPを追加します。

   > **Note:**
   >
   > 既存のノードではなく、新しいノードの説明のみを含むトポロジファイルを作成する必要があります。

   ```yaml
   ---

   pd_servers:
     - host: 172.16.5.140

   tikv_servers:
     - host: 172.16.5.140
   ```

2. スケールアウト操作を実行します。TiUPクラスタは、`scale.yaml`に記載されたポート、ディレクトリなどの情報に基づいて、対応するノードをクラスタに追加します。

   ```shell
   tiup cluster scale-out tidb-test scale.yaml
   ```

   コマンドを実行した後、`tiup cluster display tidb-test`を実行して、スケールアウトされたクラスタの状態を確認できます。

## ローリングアップグレード {#rolling-upgrade}

> **Note:**
>
> このセクションでは、アップグレードコマンドの構文のみを説明します。オンラインアップグレードの詳細な手順については、[TiUPを使用してTiDBをアップグレードする](/upgrade-tidb-using-tiup.md)を参照してください。

ローリングアップグレード機能は、TiDBの分散機能を活用しています。アップグレードプロセスは、アプリケーションにできるだけ透過的に行われ、ビジネスに影響しません。

アップグレード前に、TiUPクラスタは各コンポーネントの設定ファイルが合理的かどうかをチェックします。合理的であれば、コンポーネントはノードごとにアップグレードされます。合理的でない場合、TiUPはエラーを報告して終了します。操作はノードによって異なります。

### 異なるノードの操作 {#operations-for-different-nodes}

- PDノードのアップグレード

  - 最初に、リーダーノード以外のノードをアップグレードします。
  - すべてのリーダーノードがアップグレードされた後、リーダーノードをアップグレードします。
    - アップグレードツールは、リーダーをアップグレード済みのノードに移行するコマンドをPDに送信します。
    - リーダーの役割が別のノードに切り替わった後、以前のリーダーノードをアップグレードします。
  - アップグレード中に、不健全なノードが検出された場合、ツールはこのアップグレード操作を停止して終了します。原因を手動で分析し、問題を修正してから、再度アップグレードを実行する必要があります。

- TiKVノードのアップグレード

  - 最初に、このTiKVノードのリージョンリーダーを移行するスケジューリング操作を追加します。これにより、アップグレードプロセスがビジネスに影響しないようになります。
  - リーダーが移行した後、このTiKVノードをアップグレードします。
  - アップグレードされたTiKVが正常に起動した後、リーダーのスケジューリングを削除します。

- その他のサービスのアップグレード

  - サービスを正常に停止し、ノードを更新します。

### アップグレードコマンド {#upgrade-command}

アップグレードコマンドのフラグは次のとおりです：

```bash
Usage:
  cluster upgrade <cluster-name> <version> [flags]

Flags:
      --force                  Force upgrade won't transfer leader
  -h, --help                   help for upgrade
      --transfer-timeout int   Timeout in seconds when transferring PD and TiKV store leaders (default 600)

Global Flags:
      --ssh string          (Experimental) The executor type. Optional values are 'builtin', 'system', and 'none'.
      --wait-timeout int  Timeout of waiting the operation
      --ssh-timeout int   Timeout in seconds to connect host via SSH, ignored for operations that don't need an SSH connection. (default 5)
  -y, --yes               Skip all confirmations and assumes 'yes'
```

例えば、次のコマンドでクラスターをv7.1.3にアップグレードします：

```bash
tiup cluster upgrade tidb-test v7.1.3
```

## 設定の更新 {#update-configuration}

コンポーネントの設定を動的に更新したい場合、TiUPクラスターコンポーネントは各クラスターの現在の設定を保存します。この設定を編集するには、`tiup cluster edit-config <cluster-name>`コマンドを実行します。例えば：

```bash
tiup cluster edit-config prod-cluster
```

TiUP clusterはviエディターで設定ファイルを開きます。他のエディターを使用する場合は、`EDITOR`環境変数を使用してエディターをカスタマイズしてください。例えば、`export EDITOR=nano`のように設定します。

ファイルを編集した後、変更内容を保存してください。新しい設定をクラスターに適用するには、次のコマンドを実行してください。

```bash
tiup cluster reload prod-cluster
```

コマンドは設定をターゲットマシンに送信し、設定を有効にするためにクラスターを再起動します。

> **Note:**
>
> 監視コンポーネントの場合、対応するインスタンスにカスタム設定パスを追加するために `tiup cluster edit-config` コマンドを実行して設定をカスタマイズします。例えば：

```yaml
---

grafana_servers:
  - host: 172.16.5.134
    dashboard_dir: /path/to/local/dashboards/dir

monitoring_servers:
  - host: 172.16.5.134
    rule_dir: /path/to/local/rules/dir

alertmanager_servers:
  - host: 172.16.5.134
    config_file: /path/to/local/alertmanager.yml
```

指定されたパスのファイルの内容とフォーマットの要件は以下の通りです：

- `grafana_servers`の`dashboard_dir`フィールドで指定されたフォルダには、完全な`*.json`ファイルが含まれている必要があります。
- `monitoring_servers`の`rule_dir`フィールドで指定されたフォルダには、完全な`*.rules.yml`ファイルが含まれている必要があります。
- `alertmanager_servers`の`config_file`フィールドで指定されたファイルのフォーマットについては、[Alertmanagerの設定テンプレート](https://github.com/pingcap/tiup/blob/master/embed/templates/config/alertmanager.yml)を参照してください。

`tiup reload`を実行すると、TiUPはまずターゲットマシンの古い設定ファイルをすべて削除し、その後コントロールマシンからターゲットマシンの対応する設定ディレクトリに対応する設定をアップロードします。そのため、特定の設定ファイルを変更する場合は、すべての設定ファイル（変更されていないものも含む）が同じディレクトリにあることを確認してください。たとえば、Grafanaの`tidb.json`ファイルを変更するには、まずGrafanaの`dashboards`ディレクトリからすべての`*.json`ファイルをローカルディレクトリにコピーする必要があります。そうしないと、ターゲットマシンには他のJSONファイルが欠落してしまいます。

> **Note:**
>
> `grafana_servers`の`dashboard_dir`フィールドを設定した場合、`tiup cluster rename`コマンドを実行してクラスタをリネームした後、以下の操作を完了する必要があります：
>
> 1. ローカルの`dashboards`ディレクトリで、クラスタ名を新しいクラスタ名に変更します。
> 2. ローカルの`dashboards`ディレクトリで、`datasource`を新しいクラスタ名に変更します。なぜなら、`datasource`はクラスタ名に基づいて名付けられるからです。
> 3. `tiup cluster reload -R grafana`コマンドを実行します。

## コンポーネントの更新 {#update-component}

通常のアップグレードでは、`upgrade`コマンドを使用できます。しかし、デバッグなどのシナリオでは、現在実行中のコンポーネントを一時的なパッケージで置き換える必要がある場合があります。その場合は、`patch`コマンドを使用します：

```bash
tiup cluster patch --help
```

    Replace the remote package with a specified package and restart the service

    Usage:
      cluster patch <cluster-name> <package-path> [flags]

    Flags:
      -h, --help                    help for patch
      -N, --node strings            Specify the nodes
          --offline                 Patch a stopped cluster
          --overwrite               Use this package in the future scale-out operations
      -R, --role strings            Specify the roles
          --transfer-timeout uint   Timeout in seconds when transferring PD and TiKV store leaders, also for TiCDC drain one capture (default 600)

    Global Flags:
      -c, --concurrency int     max number of parallel tasks allowed (default 5)
          --format string       (EXPERIMENTAL) The format of output, available values are [default, json] (default "default")
          --ssh string          (EXPERIMENTAL) The executor type: 'builtin', 'system', 'none'.
          --ssh-timeout uint    Timeout in seconds to connect host via SSH, ignored for operations that don't need an SSH connection. (default 5)
          --wait-timeout uint   Timeout in seconds to wait for an operation to complete, ignored for operations that don't fit. (default 120)
      -y, --yes                 Skip all confirmations and assumes 'yes'

もしTiDBのホットフィックスパッケージが`/tmp/tidb-hotfix.tar.gz`にある場合、クラスター内のすべてのTiDBパッケージを置き換えたい場合は、次のコマンドを実行してください。

```bash
tiup cluster patch test-cluster /tmp/tidb-hotfix.tar.gz -R tidb
```

クラスタ内のTiDBパッケージを1つだけ置き換えることもできます。

```bash
tiup cluster patch test-cluster /tmp/tidb-hotfix.tar.gz -N 172.16.4.5:4000
```

## TiDB Ansibleクラスターのインポート {#import-tidb-ansible-cluster}

> **Note:**
>
> 現在、TiUPクラスターはTiSparkをサポートしていません。TiSparkが有効になっているTiDBクラスターをインポートすることはできません。

TiUPがリリースされる前は、TiDB AnsibleがTiDBクラスターをデプロイするためによく使用されていました。TiUPがTiDB Ansibleによってデプロイされたクラスターを引き継ぐためには、`import`コマンドを使用します。

`import`コマンドの使用方法は以下の通りです：

```bash
tiup cluster import --help
```

    Import an exist TiDB cluster from TiDB-Ansible

    Usage:
      cluster import [flags]

    Flags:
      -d, --dir string         The path to TiDB-Ansible directory
      -h, --help               help for import
          --inventory string   The name of inventory file (default "inventory.ini")
          --no-backup          Don't backup ansible dir, useful when there're multiple inventory files
      -r, --rename NAME        Rename the imported cluster to NAME

    Global Flags:
          --ssh string        (Experimental) The executor type. Optional values are 'builtin', 'system', and 'none'.
          --wait-timeout int  Timeout of waiting the operation
          --ssh-timeout int   Timeout in seconds to connect host via SSH, ignored for operations that don't need an SSH connection. (default 5)
      -y, --yes               Skip all confirmations and assumes 'yes'

TiDB Ansibleクラスターをインポートするには、次のいずれかのコマンドを使用できます：

```bash
cd tidb-ansible
tiup cluster import
```

```bash
tiup cluster import --dir=/path/to/tidb-ansible
```

## 操作ログの表示 {#view-the-operation-log}

操作ログを表示するには、`audit`コマンドを使用します。`audit`コマンドの使用方法は以下の通りです：

```bash
Usage:
  tiup cluster audit [audit-id] [flags]

Flags:
  -h, --help   help for audit
```

`[audit-id]`フラグが指定されていない場合、コマンドは実行されたコマンドのリストを表示します。例えば：

```bash
tiup cluster audit
```

    Starting component `cluster`: /home/tidb/.tiup/components/cluster/v1.11.3/cluster audit
    ID      Time                       Command
    --      ----                       -------
    4BLhr0  2023-12-21T23:55:09+08:00  /home/tidb/.tiup/components/cluster/v1.11.3/cluster deploy test v7.1.3 /tmp/topology.yaml
    4BKWjF  2022-12-21T23:36:57+08:00  /home/tidb/.tiup/components/cluster/v1.11.3/cluster deploy test v7.1.3 /tmp/topology.yaml
    4BKVwH  2023-12-21T23:02:08+08:00  /home/tidb/.tiup/components/cluster/v1.11.3/cluster deploy test v7.1.3 /tmp/topology.yaml
    4BKKH1  2023-12-21T16:39:04+08:00  /home/tidb/.tiup/components/cluster/v1.11.3/cluster destroy test
    4BKKDx  2023-12-21T16:36:57+08:00  /home/tidb/.tiup/components/cluster/v1.11.3/cluster deploy test v7.1.3 /tmp/topology.yaml

最初の列は `audit-id` です。特定のコマンドの実行ログを表示するには、次のようにコマンドの `audit-id` をフラグとして渡します。

```bash
tiup cluster audit 4BLhr0
```

## TiDBクラスタのホストでコマンドを実行する {#run-commands-on-a-host-in-the-tidb-cluster}

TiDBクラスタのホストでコマンドを実行するには、`exec`コマンドを使用します。`exec`コマンドの使用方法は以下の通りです：

```bash
Usage:
  cluster exec <cluster-name> [flags]

Flags:
      --command string   the command run on cluster host (default "ls")
  -h, --help             help for exec
  -N, --node strings     Only exec on host with specified nodes
  -R, --role strings     Only exec on host with specified roles
      --sudo             use root permissions (default false)

Global Flags:
      --ssh-timeout int   Timeout in seconds to connect host via SSH, ignored for operations that don't need an SSH connection. (default 5)
  -y, --yes               Skip all confirmations and assumes 'yes'
```

例えば、すべてのTiDBノードで`ls /tmp`を実行するには、次のコマンドを実行します：

```bash
tiup cluster exec test-cluster --command='ls /tmp'
```

## クラスターコントローラー {#cluster-controllers}

TiUPがリリースされる前は、`tidb-ctl`、`tikv-ctl`、`pd-ctl`などのツールを使用してクラスターを制御することができました。ツールをダウンロードして使用しやすくするために、TiUPはこれらをオールインワンのコンポーネントである`ctl`に統合しています。

```bash
Usage:
  tiup ctl:v<CLUSTER_VERSION> {tidb/pd/tikv/binlog/etcd} [flags]

Flags:
  -h, --help   help for tiup
```

このコマンドは、以前のツールと対応関係があります：

```bash
tidb-ctl [args] = tiup ctl tidb [args]
pd-ctl [args] = tiup ctl pd [args]
tikv-ctl [args] = tiup ctl tikv [args]
binlogctl [args] = tiup ctl bindlog [args]
etcdctl [args] = tiup ctl etcd [args]
```

例えば、以前は `pd-ctl -u http://127.0.0.1:2379 store` を実行してストアを表示していた場合、TiUPで次のコマンドを実行することができます。

```bash
tiup ctl:v<CLUSTER_VERSION> pd -u http://127.0.0.1:2379 store
```

## ターゲットマシンの環境チェック {#environment-checks-for-target-machines}

`check`コマンドを使用して、ターゲットマシンの環境で一連のチェックを実行し、チェック結果を出力することができます。`check`コマンドを実行することで、一般的な不合理な設定やサポートされていない状況を見つけることができます。コマンドのフラグリストは以下の通りです：

```bash
Usage:
  tiup cluster check <topology.yml | cluster-name> [flags]
Flags:
      --apply                  Try to fix failed checks
      --cluster                Check existing cluster, the input is a cluster name.
      --enable-cpu             Enable CPU thread count check
      --enable-disk            Enable disk IO (fio) check
      --enable-mem             Enable memory size check
  -h, --help                   help for check
  -i, --identity_file string   The path of the SSH identity file. If specified, public key authentication will be used.
  -p, --password               Use password of target hosts. If specified, password authentication will be used.
      --user string            The user name to login via SSH. The user must has root (or sudo) privilege.
```

デフォルトでは、このコマンドはデプロイ前に環境をチェックするために使用されます。`--cluster`フラグを指定してモードを切り替えることで、既存のクラスターのターゲットマシンをチェックすることもできます。例えば、

```bash
# check deployed servers before deployment
tiup cluster check topology.yml --user tidb -p
# check deployed servers of an existing cluster
tiup cluster check <cluster-name> --cluster
```

CPUスレッド数のチェック、メモリサイズのチェック、およびディスクパフォーマンスのチェックはデフォルトで無効になっています。本番環境では、これらのチェックを有効にし、パフォーマンスを最適化するためにパスすることが推奨されます。

- CPU：スレッド数が16以上の場合、チェックはパスします。
- メモリ：物理メモリの合計サイズが32 GB以上の場合、チェックはパスします。
- ディスク：`data_dir`のパーティションで`fio`テストを実行し、結果を記録します。

チェックを実行する際に、`--apply`フラグが指定されている場合、プログラムは自動的に失敗したアイテムを修復します。自動修復は、構成やシステムパラメータを変更して調整できる一部のアイテムに限られます。その他の修復されないアイテムは、実際の状況に応じて手動で処理する必要があります。

環境チェックはクラスタをデプロイするために必要ではありません。本番環境では、デプロイする前に環境チェックを実行し、すべてのチェックアイテムをパスすることを推奨します。すべてのチェックアイテムがパスしない場合、クラスタはデプロイされて正常に実行されるかもしれませんが、最適なパフォーマンスは得られない可能性があります。

## クラスタに接続するためにシステムのネイティブSSHクライアントを使用する {#use-the-system-s-native-ssh-client-to-connect-to-cluster}

クラスタマシンで実行される上記のすべての操作は、TiUPに組み込まれたSSHクライアントを使用してクラスタに接続し、コマンドを実行します。ただし、一部のシナリオでは、クラスタ操作を実行するために制御マシンシステムのネイティブSSHクライアントを使用する必要があります。例えば：

- 認証のためにSSHプラグインを使用する
- カスタマイズされたSSHクライアントを使用する

その場合、`--ssh=system`コマンドラインフラグを使用してシステムネイティブのコマンドラインツールを有効にすることができます：

- クラスタをデプロイする：`tiup cluster deploy <cluster-name> <version> <topo> --ssh=system`。`<cluster-name>`にクラスタの名前を入力し、`<version>`にデプロイするTiDBのバージョン（例：`v7.1.3`）を入力し、`<topo>`にトポロジーファイルを入力します。
- クラスタを起動する：`tiup cluster start <cluster-name> --ssh=system`
- クラスタをアップグレードする：`tiup cluster upgrade ... --ssh=system`

上記のすべてのクラスタ操作コマンドに`--ssh=system`を追加することで、システムのネイティブSSHクライアントを使用することができます。

毎回フラグを追加するのを避けるために、`TIUP_NATIVE_SSH`システム変数を使用してローカルSSHクライアントを使用するかどうかを指定することができます：

```shell
export TIUP_NATIVE_SSH=true
# or
export TIUP_NATIVE_SSH=1
# or
export TIUP_NATIVE_SSH=enable
```

この環境変数を指定し、同時に `--ssh` を指定すると、 `--ssh` が優先されます。

> **注意:**
>
> クラスタのデプロイプロセス中に、接続にパスワード (`-p`) を使用する必要がある場合、またはキーファイルに `passphrase` が設定されている場合、制御マシンに `sshpass` がインストールされていることを確認する必要があります。そうしないと、タイムアウトエラーが発生します。

## 制御マシンの移行と TiUP データのバックアップ {#migrate-control-machine-and-back-up-tiup-data}

TiUP データは、ユーザーのホームディレクトリにある `.tiup` ディレクトリに保存されます。制御マシンを移行するには、次の手順に従って `.tiup` ディレクトリを対象のマシンにコピーします。

1. 元のマシンのホームディレクトリで `tar czvf tiup.tar.gz .tiup` を実行します。
2. `tiup.tar.gz` を対象マシンのホームディレクトリにコピーします。
3. 対象マシンのホームディレクトリで `tar xzvf tiup.tar.gz` を実行します。
4. `.tiup` ディレクトリを `PATH` 環境変数に追加します。

   `bash` を使用し、 `tidb` ユーザーである場合、 `~/.bashrc` に `export PATH=/home/tidb/.tiup/bin:$PATH` を追加し、 `source ~/.bashrc` を実行します。その後、使用するシェルとユーザーに応じて、対応する調整を行います。

> **注意:**
>
> 異常な状況（制御マシンのディスク損傷など）によって TiUP データが失われることを避けるために、定期的に `.tiup` ディレクトリをバックアップすることをお勧めします。

## クラスタのデプロイと O\&M のためのメタファイルのバックアップとリストア {#back-up-and-restore-meta-files-for-cluster-deployment-and-o-m}

O\&M に使用されるメタファイルが失われると、TiUP を使用してクラスタを管理することができなくなります。次のコマンドを実行して、定期的にメタファイルをバックアップすることをお勧めします。

```bash
tiup cluster meta backup ${cluster_name}
```

メタファイルが失われた場合は、次のコマンドを実行して復元することができます：

```bash
tiup cluster meta restore ${cluster_name} ${backup_file}
```

> **注意:**
>
> メタファイルを復元する操作は、現在のメタファイルを上書きします。そのため、メタファイルが紛失した場合にのみ、メタファイルを復元することをお勧めします。
