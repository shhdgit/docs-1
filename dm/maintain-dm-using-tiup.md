---
title: Maintain a DM Cluster Using TiUP
summary: Learn how to maintain a DM cluster using TiUP.
---

# TiUPを使用してDMクラスタを管理する {#maintain-a-dm-cluster-using-tiup}

このドキュメントでは、TiUP DMコンポーネントを使用してDMクラスタを管理する方法について紹介します。

まだDMクラスタをデプロイしていない場合は、[TiUPを使用してDMクラスタをデプロイする](/dm/deploy-a-dm-cluster-using-tiup.md)を参照してください。

> **注意:**
>
> - 次のコンポーネント間のポートが相互に接続されていることを確認してください
>   - DM-masterノード間の`peer_port`（デフォルトで`8291`）が相互に接続されています。
>   - 各DM-masterノードはすべてのDM-workerノードの`port`（デフォルトで`8262`）に接続できます。
>   - 各DM-workerノードはすべてのDM-masterノードの`port`（デフォルトで`8261`）に接続できます。
>   - TiUPノードはすべてのDM-masterノードの`port`（デフォルトで`8261`）に接続できます。
>   - TiUPノードはすべてのDM-workerノードの`port`（デフォルトで`8262`）に接続できます。

TiUP DMコンポーネントのヘルプ情報については、次のコマンドを実行してください：

```bash
tiup dm --help
```

```
Deploy a DM cluster for production

Usage:
  tiup dm [flags]
  tiup dm [command]

Available Commands:
  deploy      Deploy a DM cluster for production
  start       Start a DM cluster
  stop        Stop a DM cluster
  restart     Restart a DM cluster
  list        List all clusters
  destroy     Destroy a specified DM cluster
  audit       Show audit log of cluster operation
  exec        Run shell command on host in the dm cluster
  edit-config Edit DM cluster config
  display     Display information of a DM cluster
  reload      Reload a DM cluster's config and restart if needed
  upgrade     Upgrade a specified DM cluster
  patch       Replace the remote package with a specified package and restart the service
  scale-out   Scale out a DM cluster
  scale-in    Scale in a DM cluster
  import      Import an exist DM 1.0 cluster from dm-ansible and re-deploy 2.0 version
  help        Help about any command

Flags:
  -h, --help               help for tiup-dm
      --native-ssh         Use the native SSH client installed on local system instead of the build-in one.
      --ssh-timeout int    Timeout in seconds to connect host via SSH, ignored for operations that don't need an SSH connection. (default 5)
  -v, --version            version for tiup-dm
      --wait-timeout int   Timeout in seconds to wait for an operation to complete, ignored for operations that don't fit. (default 60)
  -y, --yes                Skip all confirmations and assumes 'yes'
```

## クラスタリストを表示 {#view-the-cluster-list}

クラスタが正常にデプロイされた後、次のコマンドを実行してクラスタリストを表示します：

```bash
tiup dm list
```

```
Name  User  Version  Path                                  PrivateKey
----  ----  -------  ----                                  ----------
prod-cluster  tidb  ${version}  /root/.tiup/storage/dm/clusters/test  /root/.tiup/storage/dm/clusters/test/ssh/id_rsa
```

## クラスタの開始 {#start-the-cluster}

クラスタが正常にデプロイされた後、次のコマンドを実行してクラスタを開始します：

```shell
tiup dm start prod-cluster
```

If you forget the name of your クラスタ, view the クラスタ list by running `tiup dm list`.

## Check the クラスタ status {#check-the-cluster-status}

TiUP provides the `tiup dm display` command to view the status of each コンポーネント in the クラスタ. With this command, you do not have to log in to each machine to see the コンポーネント status. The usage of the command is as follows:

```bash
tiup dm display prod-cluster
```

```
dm Cluster: prod-cluster
dm Version: ${version}
ID                 Role          Host          Ports      OS/Arch       Status     Data Dir                           Deploy Dir
--                 ----          ----          -----      -------       ------     --------                           ----------
172.19.0.101:9093  alertmanager  172.19.0.101  9093/9094  linux/x86_64  Up         /home/tidb/data/alertmanager-9093  /home/tidb/deploy/alertmanager-9093
172.19.0.101:8261  dm-master     172.19.0.101  8261/8291  linux/x86_64  Healthy|L  /home/tidb/data/dm-master-8261     /home/tidb/deploy/dm-master-8261
172.19.0.102:8261  dm-master     172.19.0.102  8261/8291  linux/x86_64  Healthy    /home/tidb/data/dm-master-8261     /home/tidb/deploy/dm-master-8261
172.19.0.103:8261  dm-master     172.19.0.103  8261/8291  linux/x86_64  Healthy    /home/tidb/data/dm-master-8261     /home/tidb/deploy/dm-master-8261
172.19.0.101:8262  dm-worker     172.19.0.101  8262       linux/x86_64  Free       /home/tidb/data/dm-worker-8262     /home/tidb/deploy/dm-worker-8262
172.19.0.102:8262  dm-worker     172.19.0.102  8262       linux/x86_64  Free       /home/tidb/data/dm-worker-8262     /home/tidb/deploy/dm-worker-8262
172.19.0.103:8262  dm-worker     172.19.0.103  8262       linux/x86_64  Free       /home/tidb/data/dm-worker-8262     /home/tidb/deploy/dm-worker-8262
172.19.0.101:3000  grafana       172.19.0.101  3000       linux/x86_64  Up         -                                  /home/tidb/deploy/grafana-3000
172.19.0.101:9090  prometheus    172.19.0.101  9090       linux/x86_64  Up         /home/tidb/data/prometheus-9090    /home/tidb/deploy/prometheus-9090
```

`Status`列は、サービスが正常に実行されているかどうかを示すために`Up`または`Down`を使用します。

DM-masterコンポーネントの場合、ステータスに`|L`が追加されることがあり、これはDM-masterノードがリーダーであることを示します。 DM-workerコンポーネントの場合、`Free`は現在のDM-workerノードが上流にバインドされていないことを示します。

## クラスタのスケールイン {#scale-in-a-cluster}

クラスタのスケールインとは、ノードをオフラインにすることを意味します。 この操作により、指定されたノードがクラスタから削除され、残りのデータファイルが削除されます。

クラスタをスケールインすると、DM-masterおよびDM-workerコンポーネントで次の順序でDM操作が実行されます。

1. コンポーネントプロセスを停止します。
2. DM-masterのAPIを呼び出して`member`を削除します。
3. ノードに関連するデータファイルをクリーンアップします。

スケールインコマンドの基本的な使用法：

```bash
tiup dm scale-in <cluster-name> -N <node-id>
```

このコマンドを使用するには、少なくとも2つの引数、つまりクラスタ名とノードIDを指定する必要があります。ノードIDは、前のセクションで`tiup dm display`コマンドを使用して取得できます。

例えば、`172.16.5.140`のDMワーカーノードをスケールインする場合（DMマスターのスケールインと同様）、次のコマンドを実行します：

```bash
tiup dm scale-in prod-cluster -N 172.16.5.140:8262
```

## クラスタのスケールアウト {#scale-out-a-cluster}

スケールアウト操作は、デプロイメントと似た内部ロジックを持っています。まず、TiUP DMコンポーネントはノードのSSH接続を確認し、ターゲットノードに必要なディレクトリを作成し、次にデプロイメント操作を実行し、ノードサービスを開始します。

例えば、`prod-cluster`クラスタでDM-workerノードをスケールアウトする場合、次の手順を実行します（DM-masterのスケールアウトも同様の手順です）：

1. `scale.yaml`ファイルを作成し、新しいワーカーノードの情報を追加します：

   > **Note:**
   >
   > 新しいノードの説明のみを含むトポロジファイルを作成する必要があります。既存のノードは含めません。
   > デプロイメントディレクトリなどの追加の構成項目については、[TiUP構成パラメータの例](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml)を参照してください。

   ```yaml
   ---

   worker_servers:
     - host: 172.16.5.140

   ```

2. スケールアウト操作を実行します。TiUP DMは`scale.yaml`に記述されたポート、ディレクトリ、およびその他の情報に基づいて、対応するノードをクラスタに追加します。

   ```shell
   tiup dm scale-out prod-cluster scale.yaml
   ```

   コマンドが実行された後、`tiup dm display prod-cluster`を実行して、スケールアウトされたクラスタの状態を確認できます。

## ローリングアップグレード {#rolling-upgrade}

> **Note:**
>
> v2.0.5以降、dmctlは[クラスタのデータソースとタスク構成のエクスポートとインポート](/dm/dm-export-import-config.md)をサポートしています。
>
> アップグレード前に`config export`を使用してクラスタの構成ファイルをエクスポートできます。アップグレード後、以前のバージョンにダウングレードする必要がある場合は、最初に以前のクラスタを再デプロイし、次に`config import`を使用して以前の構成ファイルをインポートできます。
>
> v2.0.5より前のクラスタでは、dmctl v2.0.5以降を使用してデータソースとタスク構成ファイルをエクスポートおよびインポートできます。
>
> v2.0.2以降のクラスタでは、リレーワーカーに関連する構成を自動的にインポートすることは現在サポートされていません。`start-relay`コマンドを使用して、手動で[リレーログを開始](/dm/relay-log.md#enable-and-disable-relay-log)できます。

ローリングアップグレードプロセスは、アプリケーションにできるだけ透明に行われ、ビジネスに影響を与えません。操作は異なるノードによって異なります。

### アップグレードコマンド {#upgrade-command}

`tiup dm upgrade`コマンドを実行して、DMクラスタをアップグレードできます。例えば、次のコマンドはクラスタを`${version}`にアップグレードします。このコマンドを実行する前に`${version}`を必要なバージョンに変更してください：

```bash
tiup dm upgrade prod-cluster ${version}
```

## 構成の更新 {#update-configuration}

コンポーネントの構成を動的に更新したい場合、TiUP DMコンポーネントは各クラスターの現在の構成を保存します。この構成を編集するには、`tiup dm edit-config <cluster-name>`コマンドを実行します。例えば：

```bash
tiup dm edit-config prod-cluster
```

TiUP DMはviエディタで構成ファイルを開きます。他のエディタを使用する場合は、`EDITOR`環境変数を使用してエディタをカスタマイズしてください。たとえば、`export EDITOR=nano`とします。ファイルを編集した後、変更内容を保存します。新しい構成をクラスタに適用するには、次のコマンドを実行します：

```bash
tiup dm reload prod-cluster
```

コマンドは構成をターゲットマシンに送信し、構成を有効にするためにクラスタを再起動します。

## コンポーネントの更新 {#update-component}

通常のアップグレードの場合、`upgrade`コマンドを使用できます。ただし、デバッグなどのシナリオでは、現在実行中のコンポーネントを一時的なパッケージで置き換える必要がある場合があります。これを実現するには、`patch`コマンドを使用します。

```bash
tiup dm patch --help
```

```
Replace the remote package with a specified package and restart the service

Usage:
  tiup dm patch <cluster-name> <package-path> [flags]

Flags:
  -h, --help                   help for patch
  -N, --node strings           Specify the nodes
      --overwrite              Use this package in the future scale-out operations
  -R, --role strings           Specify the role
      --transfer-timeout int   Timeout in seconds when transferring dm-master leaders (default 600)

Global Flags:
      --native-ssh         Use the native SSH client installed on local system instead of the build-in one.
      --ssh-timeout int    Timeout in seconds to connect host via SSH, ignored for operations that don't need an SSH connection. (default 5)
      --wait-timeout int   Timeout in seconds to wait for an operation to complete, ignored for operations that don't fit. (default 60)
  -y, --yes                Skip all confirmations and assumes 'yes'
```

If a DM-master hotfix package is in `/tmp/dm-master-hotfix.tar.gz` and you want to replace all the DM-master packages in the cluster, run the following command:
DM-masterのホットフィックスパッケージが`/tmp/dm-master-hotfix.tar.gz`にある場合、クラスタ内のすべてのDM-masterパッケージを置き換えたい場合は、次のコマンドを実行します。

```bash
tiup dm patch prod-cluster /tmp/dm-master-hotfix.tar.gz -R dm-master
```

クラスタ内のDM-masterパッケージを1つだけ置き換えることもできます。

```bash
tiup dm patch prod-cluster /tmp/dm--hotfix.tar.gz -N 172.16.4.5:8261
```

## DM-Ansible で展開された DM 1.0 クラスタをインポートおよびアップグレードする {#import-and-upgrade-a-dm-1-0-cluster-deployed-using-dm-ansible}

> **Note:**
>
> - TiUP は DM 1.0 クラスタの DM Portal コンポーネントのインポートをサポートしていません。
> - インポートする前に元のクラスタを停止する必要があります。
> - 2.0 にアップグレードする必要のあるタスクに対して `stop-task` を実行しないでください。
> - TiUP は DM クラスタを v2.0.0-rc.2 またはそれ以降のバージョンにのみインポートすることができます。
> - `import` コマンドは、DM 1.0 クラスタから新しい DM 2.0 クラスタにデータをインポートするために使用されます。既存の DM 2.0 クラスタに DM マイグレーションタスクをインポートする必要がある場合は、[v1.0.x から v2.0+ への TiDB データマイグレーションを手動でアップグレードする](/dm/manually-upgrade-dm-1.0-to-2.0.md) を参照してください。
> - 一部のコンポーネントのデプロイディレクトリは元のクラスタと異なります。詳細を表示するには `display` コマンドを実行できます。
> - インポートする前に `tiup update --self && tiup update dm` を実行して、TiUP DM コンポーネントが最新バージョンであることを確認してください。
> - インポート後、クラスタには DM-master ノードが 1 つだけ存在します。DM-master をスケールアウトするには [クラスタをスケールアウトする](#scale-out-a-cluster) を参照してください。

TiUP がリリースされる前は、DM-Ansible が DM クラスタを展開するためによく使用されていました。TiUP が DM-Ansible によって展開された DM 1.0 クラスタを引き継ぐためには、`import` コマンドを使用します。

たとえば、DM Ansible を使用して展開されたクラスタをインポートするには：

```bash
tiup dm import --dir=/path/to/dm-ansible --cluster-version ${version}
```

実行するには、 `tiup list dm-master` を実行して、TiUP がサポートする最新のクラスタバージョンを表示します。

`import` コマンドの使用手順は次のとおりです。

1. TiUP は、以前に DM-Ansible を使用して展開した DM クラスタに基づいて、トポロジファイル [`topology.yml`](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml) を生成します。
2. トポロジファイルが生成されたことを確認した後、それを使用して v2.0 以降のバージョンの DM クラスタを展開できます。

展開が完了したら、`tiup dm start` コマンドを実行してクラスタを起動し、DM カーネルのアップグレードプロセスを開始できます。

## 操作ログの表示 {#view-the-operation-log}

操作ログを表示するには、`audit` コマンドを使用します。`audit` コマンドの使用方法は次のとおりです：

```bash
Usage:
  tiup dm audit [audit-id] [flags]

Flags:
  -h, --help   help for audit
```

`[audit-id]`引数が指定されていない場合、コマンドは実行されたコマンドのリストを表示します。例えば：

```bash
tiup dm audit
```

```
ID      Time                  Command
--      ----                  -------
4D5kQY  2020-08-13T05:38:19Z  tiup dm display test
4D5kNv  2020-08-13T05:36:13Z  tiup dm list
4D5kNr  2020-08-13T05:36:10Z  tiup dm deploy -p prod-cluster ${version} ./examples/dm/minimal.yaml
```

最初の列は `audit-id` です。特定のコマンドの実行ログを表示するには、`audit-id` 引数を以下のように渡します。

```bash
tiup dm audit 4D5kQY
```

## DMクラスタのホストでコマンドを実行する {#run-commands-on-a-host-in-the-dm-cluster}

DMクラスタのホストでコマンドを実行するには、`exec`コマンドを使用します。`exec`コマンドの使用法は次のとおりです：

```bash
Usage:
  tiup dm exec <cluster-name> [flags]

Flags:
      --command string   the command run on cluster host (default "ls")
  -h, --help             help for exec
  -N, --node strings     Only exec on host with specified nodes
  -R, --role strings     Only exec on host with specified roles
      --sudo             use root permissions (default false)
```

例えば、すべてのDMノードで `ls /tmp` を実行するには、次のコマンドを実行します：

```bash
tiup dm exec prod-cluster --command='ls /tmp'
```

## dmctl {#dmctl}

TiUPはDMクラスタコントローラー`dmctl`を統合しています。

dmctlを使用するには、次のコマンドを実行します：

```bash
tiup dmctl [args]
```

"dmctlのバージョンを指定します。このコマンドを実行する前に`${version}`を必要なバージョンに変更してください。"

```
tiup dmctl:${version} [args]
```

以前のdmctlコマンドでソースを追加するには、`dmctl --master-addr master1:8261 operate-source create /tmp/source1.yml`となります。dmctlがTiUPに統合された後、コマンドは次のようになります：

```bash
tiup dmctl --master-addr master1:8261 operate-source create /tmp/source1.yml
```

## システムのネイティブSSHクライアントを使用してクラスタに接続する {#use-the-system-s-native-ssh-client-to-connect-to-cluster}

上記のすべての操作は、TiUPに組み込まれたSSHクライアントを使用してクラスタマシンで実行され、クラスタに接続してコマンドを実行します。ただし、一部のシナリオでは、制御マシンシステムのネイティブSSHクライアントを使用して、そのようなクラスタ操作を実行する必要がある場合もあります。例えば：

- 認証のためにSSHプラグインを使用する
- カスタマイズされたSSHクライアントを使用する

その後、`--native-ssh`コマンドラインフラグを使用してシステムネイティブのコマンドラインツールを有効にすることができます：

- クラスタをデプロイする：`tiup dm deploy <cluster-name> <version> <topo> --native-ssh`。`<cluster-name>`にクラスタの名前、`<version>`にデプロイするDMバージョン（例：`v7.1.3`）、`<topo>`にトポロジファイル名を入力してください。
- クラスタを起動する：`tiup dm start <cluster-name> --native-ssh`。
- クラスタをアップグレードする：`tiup dm upgrade ... --native-ssh`

上記のすべてのクラスタ操作コマンドに`--native-ssh`を追加して、システムのネイティブSSHクライアントを使用することができます。

すべてのコマンドにこのようなフラグを追加するのを避けるために、`TIUP_NATIVE_SSH`システム変数を使用して、ローカルSSHクライアントを使用するかどうかを指定することができます。

```sh
export TIUP_NATIVE_SSH=true
# or
export TIUP_NATIVE_SSH=1
# or
export TIUP_NATIVE_SSH=enable
```

If you specify this environment variable and `--native-ssh` at the same time, `--native-ssh` has higher priority.

> **Note:**
>
> クラスタのデプロイメントプロセス中に、接続のためにパスワードを使用する必要があるか、キーファイルに`passphrase`が構成されている場合は、制御マシンに`sshpass`がインストールされていることを確認する必要があります。そうでない場合、タイムアウトエラーが報告されます。
