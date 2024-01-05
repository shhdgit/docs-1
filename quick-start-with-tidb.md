---
title: Quick Start Guide for the TiDB Database Platform
summary: Learn how to quickly get started with the TiDB platform and see if TiDB is the right choice for you.
---

# TiDBデータベースプラットフォームのクイックスタートガイド {#quick-start-guide-for-the-tidb-database-platform}

このガイドでは、TiDBの素早い始め方を提供します。本番環境以外では、次のいずれかの方法を使用してTiDBデータベースを展開できます。

- [ローカルテストクラスターを展開](#deploy-a-local-test-cluster)（macOSおよびLinux用）
- [単一のマシンで本番展開をシミュレート](#simulate-production-deployment-on-a-single-machine)（Linux用）

さらに、[TiDB Playground](https://play.tidbcloud.com/?utm_source=docs\&utm_medium=tidb_quick_start)でTiDBの機能を試すことができます。

> **Note:**
>
> このガイドで提供されている展開方法は、**クイックスタート専用**であり、**本番用ではありません**。
>
> - セルフホスト型の本番クラスターを展開するには、[本番展開ガイド](/production-deployment-using-tiup.md)を参照してください。
> - KubernetesでTiDBを展開するには、[KubernetesでTiDBを始める](https://docs.pingcap.com/tidb-in-kubernetes/stable/get-started)を参照してください。
> - クラウドでTiDBを管理するには、[TiDB Cloudクイックスタート](https://docs.pingcap.com/tidbcloud/tidb-cloud-quickstart)を参照してください。

## ローカルテストクラスターのデプロイ {#deploy-a-local-test-cluster}

- シナリオ：単一のmacOSまたはLinuxサーバーを使用してテスト用のローカルTiDBクラスターを迅速にデプロイします。このようなクラスターをデプロイすることで、TiDBの基本的なアーキテクチャやTiDB、TiKV、PD、およびモニタリングコンポーネントなどのコンポーネントの操作を学ぶことができます。

<SimpleTab>
<div label="macOS">

分散システムとして、基本的なTiDBテストクラスターは通常、2つのTiDBインスタンス、3つのTiKVインスタンス、3つのPDインスタンス、およびオプションのTiFlashインスタンスで構成されています。TiUP Playgroundを使用すると、次の手順に従ってテストクラスターを迅速に構築できます。

1. TiUPをダウンロードしてインストールします：

   ```shell
   curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
   ```

   以下のメッセージが表示された場合、TiUPが正常にインストールされています：

   ```log
   Successfully set mirror to https://tiup-mirrors.pingcap.com
   Detected shell: zsh
   Shell profile:  /Users/user/.zshrc
   /Users/user/.zshrc has been modified to add tiup to PATH
   open a new terminal or source /Users/user/.zshrc to use it
   Installed path: /Users/user/.tiup/bin/tiup
   ===============================================
   Have a try:     tiup playground
   ===============================================
   ```

   上記の出力でShellプロファイルのパスに注意してください。次のステップでこのパスを使用する必要があります。

2. グローバル環境変数を宣言します：

   > **Note:**
   >
   > インストール後、TiUPは対応するShellプロファイルファイルの絶対パスを表示します。このパスに従って、次の`source`コマンドで`${your_shell_profile}`を変更する必要があります。この場合、`${your_shell_profile}`はステップ1の出力から`/Users/user/.zshrc`です。

   ```shell
   source ${your_shell_profile}
   ```

3. 現在のセッションでクラスターを起動します：

   - 最新バージョンのTiDBクラスターを1つのTiDBインスタンス、1つのTiKVインスタンス、1つのPDインスタンス、および1つのTiFlashインスタンスで起動するには、次のコマンドを実行します：

     ```shell
     tiup playground
     ```

   - TiDBのバージョンと各コンポーネントのインスタンス数を指定するには、次のようなコマンドを実行します：

     ```shell
     tiup playground v7.1.3 --db 2 --pd 3 --kv 3
     ```

     このコマンドは、v7.1.3などのバージョンのクラスターをローカルマシンにダウンロードして起動します。最新バージョンを表示するには、`tiup list tidb`を実行します。

     このコマンドはクラスターのアクセス方法を返します：

     ```log
     CLUSTER START SUCCESSFULLY, Enjoy it ^-^
     To connect TiDB: mysql --comments --host 127.0.0.1 --port 4001 -u root -p (no password)
     To connect TiDB: mysql --comments --host 127.0.0.1 --port 4000 -u root -p (no password)
     To view the dashboard: http://127.0.0.1:2379/dashboard
     PD client endpoints: [127.0.0.1:2379 127.0.0.1:2382 127.0.0.1:2384]
     To view Prometheus: http://127.0.0.1:9090
     To view Grafana: http://127.0.0.1:3000
     ```

     > **Note:**
     >
     > - v5.2.0以降、TiDBはApple M1チップを使用しているマシンで`tiup playground`を実行することをサポートしています。
     > - この方法で操作されるプレイグラウンドでは、テストデプロイメントが完了した後、TiUPは元のクラスターデータをクリーンアップします。コマンドを再実行すると新しいクラスターが取得できます。
     > - データをストレージに永続化する場合は、`tiup --tag <your-tag> playground ...`を実行します。詳細については、[TiUPリファレンス](/tiup/tiup-reference.md#-t---tag)ガイドを参照してください。

4. 新しいセッションを開始してTiDBにアクセスします：

   - TiUPクライアントを使用してTiDBに接続します。

     ```shell
     tiup client
     ```

   - または、MySQLクライアントを使用してTiDBに接続できます。

     ```shell
     mysql --host 127.0.0.1 --port 4000 -u root
     ```

5. TiDBのPrometheusダッシュボードには、<http://127.0.0.1:9090>でアクセスできます。

6. <http://127.0.0.1:2379/dashboard>で[TiDBダッシュボード](/dashboard/dashboard-intro.md)にアクセスします。デフォルトのユーザー名は`root`で、パスワードは空です。

7. TiDBのGrafanaダッシュボードには、<http://127.0.0.1:3000>でアクセスできます。デフォルトのユーザー名とパスワードはいずれも`admin`です。

8. (オプション) [TiFlashにデータをロード](/tiflash/tiflash-overview.md#use-tiflash)して解析します。

9. テストデプロイメント後にクラスターをクリーンアップします：

   1. 上記のTiDBサービスを停止するには、<kbd>Control+C</kbd>を押します。

   2. サービスが停止した後、次のコマンドを実行します：

      ```shell
      tiup clean --all
      ```

> **Note:**
>
> TiUP Playgroundはデフォルトで`127.0.0.1`でリッスンし、サービスはローカルからのみアクセス可能です。サービスを外部からアクセス可能にする場合は、`--host`パラメータを使用してリッスンアドレスを指定します。

1. TiUPをダウンロードしてインストールします：

   ```shell
   curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
   ```

   以下のメッセージが表示された場合、TiUPのインストールに成功しています：

   ```log
   Successfully set mirror to https://tiup-mirrors.pingcap.com
   Detected shell: zsh
   Shell profile:  /Users/user/.zshrc
   /Users/user/.zshrc has been modified to add tiup to PATH
   open a new terminal or source /Users/user/.zshrc to use it
   Installed path: /Users/user/.tiup/bin/tiup
   ===============================================
   Have a try:     tiup playground
   ===============================================
   ```

   上記の出力でShellプロファイルのパスに注意してください。次のステップでそのパスを使用する必要があります。

2. グローバル環境変数を宣言します：

   > **Note:**
   >
   > インストール後、TiUPは対応するShellプロファイルファイルの絶対パスを表示します。そのパスに従って、次の`source`コマンドで`${your_shell_profile}`を変更する必要があります。

   ```shell
   source ${your_shell_profile}
   ```

3. 現在のセッションでクラスターを起動します：

   - 最新バージョンのTiDBクラスターを1つのTiDBインスタンス、1つのTiKVインスタンス、1つのPDインスタンス、および1つのTiFlashインスタンスで起動するには、次のコマンドを実行します：

     ```shell
     tiup playground
     ```

   - TiDBのバージョンと各コンポーネントのインスタンス数を指定するには、次のようなコマンドを実行します：

     ```shell
     tiup playground v7.1.3 --db 2 --pd 3 --kv 3
     ```

     このコマンドは、v7.1.3などのバージョンのクラスターをローカルマシンにダウンロードして起動します。最新バージョンを表示するには、`tiup list tidb`を実行します。

     このコマンドは、クラスターのアクセス方法を返します：

     ```log
     CLUSTER START SUCCESSFULLY, Enjoy it ^-^
     To connect TiDB: mysql --host 127.0.0.1 --port 4000 -u root -p (no password) --comments
     To view the dashboard: http://127.0.0.1:2379/dashboard
     PD client endpoints: [127.0.0.1:2379]
     To view the Prometheus: http://127.0.0.1:9090
     To view the Grafana: http://127.0.0.1:3000
     ```

     > **Note:**
     >
     > この方法で操作されるplaygroundでは、テストデプロイメントが完了すると、TiUPが元のクラスターデータをクリーンアップします。コマンドを再実行すると新しいクラスターが取得できます。
     > データをストレージに永続化する場合は、`tiup --tag <your-tag> playground ...`を実行します。詳細については、[TiUPリファレンス](/tiup/tiup-reference.md#-t---tag)ガイドを参照してください。

4. 新しいセッションを開始してTiDBにアクセスします：

   - TiUPクライアントを使用してTiDBに接続します。

     ```shell
     tiup client
     ```

   - または、MySQLクライアントを使用してTiDBに接続できます。

     ```shell
     mysql --host 127.0.0.1 --port 4000 -u root
     ```

5. TiDBのPrometheusダッシュボードには、<http://127.0.0.1:9090>でアクセスできます。

6. TiDBの[ダッシュボード](/dashboard/dashboard-intro.md)には、<http://127.0.0.1:2379/dashboard>でアクセスできます。デフォルトのユーザー名は`root`で、パスワードは空です。

7. TiDBのGrafanaダッシュボードには、<http://127.0.0.1:3000>を介してアクセスできます。デフォルトのユーザー名とパスワードは`admin`です。

8. (オプション) [TiFlashにデータをロード](/tiflash/tiflash-overview.md#use-tiflash)して解析します。

9. テストデプロイメント後にクラスターをクリーンアップします：

   1. <kbd>Control+C</kbd>を押してプロセスを停止します。

   2. サービスが停止した後、次のコマンドを実行します：

      ```shell
      tiup clean --all
      ```

> **Note:**
>
> TiUP Playgroundはデフォルトで`127.0.0.1`でリッスンし、サービスはローカルからのみアクセス可能です。サービスを外部からアクセス可能にする場合は、`--host`パラメータを使用してリッスンアドレスを指定して、ネットワークインターフェースカード（NIC）を外部からアクセス可能なIPアドレスにバインドしてください。

## 単一のマシン上で本番展開をシミュレート {#simulate-production-deployment-on-a-single-machine}

- シナリオ：最小のTiDBクラスターを完全なトポロジーで体験し、単一のLinuxサーバー上で本番展開手順をシミュレートします。

このセクションでは、TiUPの最小トポロジーのYAMLファイルを使用してTiDBクラスターを展開する方法について説明します。

### 準備 {#prepare}

TiDBクラスターを展開する前に、ターゲットマシンが次の要件を満たしていることを確認してください。

- CentOS 7.3以降のバージョンがインストールされていること。
- Linux OSがTiDBおよび関連ソフトウェアのインストールパッケージをダウンロードするためにインターネットにアクセスできること。

最小のTiDBクラスタートポロジーは、次のインスタンスで構成されています。

> **Note:**
>
> インスタンスのIPアドレスは例として示されています。実際の展開では、IPアドレスを実際のIPアドレスに置き換えてください。

| インスタンス  | カウント | IP                                     | コンフィグレーション                              |
| :------ | :--- | :------------------------------------- | :-------------------------------------- |
| TiKV    | 3    | 10.0.1.1 <br/> 10.0.1.1 <br/> 10.0.1.1 | ポートとディレクトリの競合を避ける                       |
| TiDB    | 1    | 10.0.1.1                               | デフォルトのポート <br/> グローバルディレクトリのコンフィギュレーション |
| PD      | 1    | 10.0.1.1                               | デフォルトのポート <br/> グローバルディレクトリのコンフィギュレーション |
| TiFlash | 1    | 10.0.1.1                               | デフォルトのポート <br/> グローバルディレクトリのコンフィギュレーション |
| Monitor | 1    | 10.0.1.1                               | デフォルトのポート <br/> グローバルディレクトリのコンフィギュレーション |

ターゲットマシンのその他の要件は次のとおりです。

- `root`ユーザーとそのパスワードが必要です。
- [ターゲットマシンのファイアウォールサービスを停止](/check-before-deployment.md#check-and-stop-the-firewall-service-of-target-machines)するか、TiDBクラスターノードで必要なポートを開いてください。
- 現在、TiUPクラスターはx86\_64（AMD64）およびARMアーキテクチャでTiDBを展開することをサポートしています。

  - AMD64では、CentOS 7.3以降のバージョンを使用することをお勧めします。
  - ARMでは、CentOS 7.6 1810を使用することをお勧めします。

### 展開 {#deploy}

> **Note:**
>
> ターゲットマシンに通常のユーザーまたは`root`ユーザーとしてログインできます。以下の手順では、`root`ユーザーを例にしています。

1. TiUPをダウンロードしてインストールします。

   ```shell
   curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
   ```

2. グローバル環境変数を宣言します。

   > **Note:**
   >
   > インストール後、TiUPは対応するシェルプロファイルの絶対パスを表示します。次の`source`コマンドで`${your_shell_profile}`をパスに応じて変更する必要があります。

   ```shell
   source ${your_shell_profile}
   ```

3. TiUPのクラスターコンポーネントをインストールします。

   ```shell
   tiup cluster
   ```

4. もしTiUPクラスターが既にマシンにインストールされている場合、ソフトウェアのバージョンを更新します。

   ```shell
   tiup update --self && tiup update cluster
   ```

5. `root`ユーザー権限を使用して`sshd`サービスの接続制限を増やします。これは、TiUPが複数のマシンで展開をシミュレートする必要があるためです。

   1. `/etc/ssh/sshd_config`を変更し、`MaxSessions`を`20`に設定します。
   2. `sshd`サービスを再起動します。

      ```shell
      service sshd restart
      ```

6. クラスターを作成して開始します。

   構成ファイルを次のテンプレートに従って編集し、`topo.yaml`という名前で保存します。

   ```yaml
   # # グローバル変数はすべての展開に適用され、デフォルト値として使用されます
   # # 展開の特定の値が欠落している場合。
   global:
    user: "tidb"
    ssh_port: 22
    deploy_dir: "/tidb-deploy"
    data_dir: "/tidb-data"

   # # 監視変数はすべてのマシンに適用されます。
   monitored:
    node_exporter_port: 9100
    blackbox_exporter_port: 9115

   server_configs:
    tidb:
      instance.tidb_slow_log_threshold: 300
    tikv:
      readpool.storage.use-unified-pool: false
      readpool.coprocessor.use-unified-pool: true
    pd:
      replication.enable-placement-rules: true
      replication.location-labels: ["host"]
    tiflash:
      logger.level: "info"

   pd_servers:
    - host: 10.0.1.1

   tidb_servers:
    - host: 10.0.1.1

   tikv_servers:
    - host: 10.0.1.1
      port: 20160
      status_port: 20180
      config:
        server.labels: { host: "logic-host-1" }

    - host: 10.0.1.1
      port: 20161
      status_port: 20181
      config:
        server.labels: { host: "logic-host-2" }

    - host: 10.0.1.1
      port: 20162
      status_port: 20182
      config:
        server.labels: { host: "logic-host-3" }

   tiflash_servers:
    - host: 10.0.1.1

   monitoring_servers:
    - host: 10.0.1.1

   grafana_servers:
    - host: 10.0.1.1
   ```

   - `user: "tidb"`: `tidb`システムユーザー（展開中に自動的に作成される）を使用してクラスターの内部管理を実行します。デフォルトでは、SSHを介してターゲットマシンにログインするためにポート22を使用します。
   - `replication.enable-placement-rules`: これはTiFlashが正常に実行されることを確認するために設定されるPDパラメータです。
   - `host`: ターゲットマシンのIPアドレス。

7. クラスター展開コマンドを実行します。

   ```shell
   tiup cluster deploy <cluster-name> <version> ./topo.yaml --user root -p
   ```

   - `<cluster-name>`: クラスター名を設定します
   - `<version>`: TiDBクラスターバージョンを設定します（例：`v7.1.3`）。`tiup list tidb`コマンドを実行してサポートされているすべてのTiDBバージョンを確認できます。
   - `-p`: ターゲットマシンに接続するためのパスワードを指定します。

     > **Note:**
     >
     > シークレットキーを使用する場合は、`-i`を使用してキーのパスを指定できます。`-i`と`-p`を同時に使用しないでください。

   "y"を入力し、`root`ユーザーのパスワードを入力して展開を完了します。

   ```log
   Do you want to continue? [y/N]:  y
   Input SSH password:
   ```

8. クラスターを開始します。

   ```shell
   tiup cluster start <cluster-name>
   ```

9. クラスターにアクセスします。

   - MySQLクライアントをインストールします。すでにインストールされている場合は、この手順をスキップしてください。

     ```shell
     yum -y install mysql
     ```

   - TiDBにアクセスします。パスワードは空です。

     ```shell
     mysql -h 10.0.1.1 -P 4000 -u root
     ```

   - Grafanaモニタリングダッシュボードにアクセスします：<http://{grafana-ip}:3000>。デフォルトのユーザー名とパスワードはどちらも`admin`です。

   - [TiDBダッシュボード](/dashboard/dashboard-intro.md)にアクセスします：<http://{pd-ip}:2379/dashboard>。デフォルトのユーザー名は`root`で、パスワードは空です。

   - 現在展開されているクラスターリストを表示するには：

     ```shell
     tiup cluster list
     ```

   - クラスタートポロジーとステータスを表示するには：

     ```shell
     tiup cluster display <cluster-name>
     ```

## 次のステップ {#what-s-next}

ローカルテスト環境にTiDBクラスターを展開したばかりの場合、次のステップがあります：

- [TiDBでの基本的なSQL操作](/basic-sql-operations.md)を参照して基本的なSQL操作について学びます。
- [TiDBへのデータ移行](/migration-overview.md)を参照して、TiDBへのデータ移行を行うこともできます。

本番環境でTiDBクラスターを展開する準備が整った場合、次のステップがあります：

- [TiUPを使用したTiDBの展開](/production-deployment-using-tiup.md)
- または、[TiDB Operatorを使用したクラウド上でのTiDBの展開](https://docs.pingcap.com/tidb-in-kubernetes/stable)を参照して、TiDBをクラウド上で展開することもできます。

TiFlashを使用した分析ソリューションをお探しの場合、次のステップがあります：

- [TiFlashの使用](/tiflash/tiflash-overview.md#use-tiflash)
- [TiFlashの概要](/tiflash/tiflash-overview.md)
