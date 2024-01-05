---
title: Deploy a TiDB Cluster Using TiUP
summary: Learn how to easily deploy a TiDB cluster using TiUP.
---

# TiUPを使用してTiDBクラスターをデプロイする {#deploy-a-tidb-cluster-using-tiup}

[TiUP](https://github.com/pingcap/tiup)は、TiDB 4.0で導入されたクラスターの操作とメンテナンスツールです。TiUPは、Golangで書かれたクラスター管理コンポーネントである[TiUP cluster](https://github.com/pingcap/tiup/tree/master/components/cluster)を提供します。TiUP clusterを使用すると、TiDBクラスターのデプロイ、開始、停止、破棄、スケーリング、アップグレードなどの日常的なデータベース操作を簡単に行うことができ、TiDBクラスターのパラメータを管理することができます。

TiUPは、TiDB、TiFlash、TiDB Binlog、TiCDC、および監視システムのデプロイをサポートしています。このドキュメントでは、異なるトポロジーのTiDBクラスターをデプロイする方法について紹介します。

## ステップ1. 前提条件と事前チェック {#step-1-prerequisites-and-precheck}

以下のドキュメントを読んでいることを確認してください。

- [ハードウェアおよびソフトウェア要件](/hardware-and-software-requirements.md)
- [デプロイ前の環境およびシステム構成のチェック](/check-before-deployment.md)

## ステップ2. コントロールマシンにTiUPをデプロイする {#step-2-deploy-tiup-on-the-control-machine}

TiUPをコントロールマシンにオンラインまたはオフラインでデプロイすることができます。

### オンラインでTiUPをデプロイする {#deploy-tiup-online}

通常のユーザーアカウント（`tidb`ユーザーを例とします）でコントロールマシンにログインします。その後、`tidb`ユーザーによってTiUPのインストールおよびクラスター管理を行うことができます。

1. 次のコマンドを実行してTiUPをインストールします。

   ```shell
   curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
   ```

2. TiUPの環境変数を設定します。

   1. グローバル環境変数を再宣言します。

      ```shell
      source .bash_profile
      ```

   2. TiUPがインストールされているかどうかを確認します。

      ```shell
      which tiup
      ```

3. TiUPクラスターコンポーネントをインストールします。

   ```shell
   tiup cluster
   ```

4. 既にTiUPがインストールされている場合は、TiUPクラスターコンポーネントを最新バージョンに更新します。

   ```shell
   tiup update --self && tiup update cluster
   ```

   `Update successfully!`と表示された場合、TiUPクラスターは正常に更新されています。

5. 現在のTiUPクラスターのバージョンを確認します。

   ```shell
   tiup --binary cluster
   ```

### オフラインでTiUPをデプロイする {#deploy-tiup-offline}

TiUPを使用してオフラインでTiDBクラスターをデプロイする手順は次のとおりです。

#### TiUPオフラインコンポーネントパッケージの準備 {#prepare-the-tiup-offline-component-package}

方法1：[公式ダウンロードページ](https://www.pingcap.com/download/)で、対象のTiDBバージョンのオフラインミラーパッケージ（TiUPオフラインパッケージを含む）を選択します。サーバーパッケージとツールキットパッケージを同時にダウンロードする必要があることに注意してください。

方法2：`tiup mirror clone`を使用してオフラインコンポーネントパッケージを手動でパックします。詳細な手順は次のとおりです。

1. オンラインでTiUPパッケージマネージャーをインストールします。

   1. TiUPツールをインストールします。

      ```shell
      curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
      ```

   2. グローバル環境変数を再宣言します。

      ```shell
      source .bash_profile
      ```

   3. TiUPがインストールされているかどうかを確認します。

      ```shell
      which tiup
      ```

2. TiUPを使用してミラーを取得します。

   1. インターネットにアクセスできるマシンで必要なコンポーネントを取得します。

      ```shell
      tiup mirror clone tidb-community-server-${version}-linux-amd64 ${version} --os=linux --arch=amd64
      ```

      上記のコマンドは、現在のディレクトリに`tidb-community-server-${version}-linux-amd64`というディレクトリを作成し、クラスターを開始するために必要なコンポーネントパッケージを含んでいます。

   2. `tar`コマンドを使用してコンポーネントパッケージをパックし、パッケージを隔離された環境のコントロールマシンに送信します。

      ```bash
      tar czvf tidb-community-server-${version}-linux-amd64.tar.gz tidb-community-server-${version}-linux-amd64
      ```

      `tidb-community-server-${version}-linux-amd64.tar.gz`は独立したオフライン環境パッケージです。

3. オフラインミラーをカスタマイズするか、既存のオフラインミラーの内容を調整します。

   既存のオフラインミラーを調整したい場合（コンポーネントの新しいバージョンを追加するなど）、次の手順を実行します。

   1. オフラインミラーを取得する際に、コンポーネントおよびバージョン情報などをパラメーターで指定して不完全なオフラインミラーを取得できます。たとえば、次のコマンドを実行して、TiUP v1.11.3およびTiUP Cluster v1.11.3のみを含むオフラインミラーを取得できます。

      ```bash
      tiup mirror clone tiup-custom-mirror-v1.11.3 --tiup v1.11.3 --cluster v1.11.3
      ```

      特定のプラットフォームのコンポーネントのみが必要な場合は、`--os`または`--arch`パラメーターを使用して指定できます。

   2. "TiUPを使用してミラーを取得する"のステップ2を参照し、この不完全なオフラインミラーを隔離された環境のコントロールマシンに送信します。

   3. 隔離された環境のコントロールマシンで現在のオフラインミラーのパスを確認します。TiUPツールが最新バージョンの場合、次のコマンドを実行して現在のミラーアドレスを取得できます。

      ```bash
      tiup mirror show
      ```

      上記のコマンドの出力に`show`コマンドが存在しないと表示された場合、古いバージョンのTiUPを使用している可能性があります。この場合、`$HOME/.tiup/tiup.toml`から現在のミラーアドレスを取得できます。このミラーアドレスを記録します。以下の手順では、`${base_mirror}`をこのアドレスの参照に使用します。

   4. 不完全なオフラインミラーを既存のオフラインミラーにマージします。

      まず、現在のオフラインミラーの`keys`ディレクトリを`$HOME/.tiup/`ディレクトリにコピーします。

      ```bash
      cp -r ${base_mirror}/keys $HOME/.tiup/
      ```

      次に、TiUPコマンドを使用して不完全なオフラインミラーを使用中のミラーにマージします。

      ```bash
      tiup mirror merge tiup-custom-mirror-v1.11.3
      ```

   5. 上記の手順が完了したら、`tiup list`コマンドを実行して結果を確認します。このドキュメントの例では、`tiup list tiup`および`tiup list cluster`の出力により、`v1.11.3`の対応するコンポーネントが利用可能であることが示されます。

#### オフラインTiUPコンポーネントをデプロイする {#deploy-the-offline-tiup-component}

対象クラスターのコントロールマシンにパッケージを送信した後、次のコマンドを実行してTiUPコンポーネントをインストールします：

```bash
tar xzvf tidb-community-server-${version}-linux-amd64.tar.gz && \
sh tidb-community-server-${version}-linux-amd64/local_install.sh && \
source /home/tidb/.bash_profile
```

`local_install.sh`スクリプトは、自動的に`tiup mirror set tidb-community-server-${version}-linux-amd64`コマンドを実行して、現在のミラーアドレスを`tidb-community-server-${version}-linux-amd64`に設定します。

#### オフラインパッケージのマージ {#merge-offline-packages}

[公式ダウンロードページ](https://www.pingcap.com/download/)からオフラインパッケージをダウンロードした場合、サーバーパッケージとツールキットパッケージをオフラインミラーにマージする必要があります。`tiup mirror clone`コマンドを使用してオフラインコンポーネントパッケージを手動でパッケージ化した場合、この手順はスキップできます。

次のコマンドを実行して、オフラインツールキットパッケージをサーバーパッケージディレクトリにマージします：

```bash
tar xf tidb-community-toolkit-${version}-linux-amd64.tar.gz
ls -ld tidb-community-server-${version}-linux-amd64 tidb-community-toolkit-${version}-linux-amd64
cd tidb-community-server-${version}-linux-amd64/
cp -rp keys ~/.tiup/
tiup mirror merge ../tidb-community-toolkit-${version}-linux-amd64
```

別のディレクトリにミラーを切り替えるには、「tiup mirror set <mirror-dir>」コマンドを実行します。ミラーをオンライン環境に切り替えるには、「tiup mirror set <https://tiup-mirrors.pingcap.com」コマンドを実行します。>

## ステップ3. クラスタトポロジファイルの初期化 {#step-3-initialize-cluster-topology-file}

次のコマンドを実行して、クラスタトポロジファイルを作成します：

```shell
tiup cluster template > topology.yaml
```

以下の2つの一般的なシナリオでは、コマンドを実行して推奨されるトポロジーテンプレートを生成できます。

- ハイブリッドデプロイメントの場合：複数のインスタンスが単一のマシンに展開されます。詳細については、[ハイブリッドデプロイメントトポロジー](/hybrid-deployment-topology.md)を参照してください。

  ```shell
  tiup cluster template --full > topology.yaml
  ```

- 地理的に分散したデータセンターにTiDBクラスタを展開する場合：詳細については、[地理的に分散したデプロイメントトポロジー](/geo-distributed-deployment-topology.md)を参照してください。

  ```shell
  tiup cluster template --multi-dc > topology.yaml
  ```

`vi topology.yaml`を実行して、構成ファイルの内容を確認してください。

```shell
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/tidb-deploy"
  data_dir: "/tidb-data"
server_configs: {}
pd_servers:
  - host: 10.0.1.4
  - host: 10.0.1.5
  - host: 10.0.1.6
tidb_servers:
  - host: 10.0.1.7
  - host: 10.0.1.8
  - host: 10.0.1.9
tikv_servers:
  - host: 10.0.1.1
  - host: 10.0.1.2
  - host: 10.0.1.3
monitoring_servers:
  - host: 10.0.1.4
grafana_servers:
  - host: 10.0.1.4
alertmanager_servers:
  - host: 10.0.1.4
```

以下の例は、7つの一般的なシナリオをカバーしています。トポロジの説明と対応するリンクのテンプレートに従って、構成ファイル（`topology.yaml`という名前）を変更する必要があります。その他のシナリオの場合は、構成テンプレートを編集してください。

| アプリケーション                                                               | 構成タスク                                                               | 構成ファイルテンプレート                                                                                                                                                                                                                                                                                                                                                                                           | トポロジの説明                                                                                                                                       |
| :--------------------------------------------------------------------- | :------------------------------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| OLTP                                                                   | [デプロイの最小トポロジ](/minimal-deployment-topology.md)                      | [シンプルな最小構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-mini.yaml) <br/> [フルな最小構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-mini.yaml)                                                                                                                                                                                                 | これは、tidb-server、tikv-server、およびpd-serverを含む基本的なクラスタートポロジです。                                                                                   |
| HTAP                                                                   | [TiFlashトポロジのデプロイ](/tiflash-deployment-topology.md)                 | [シンプルなTiFlash構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-tiflash.yaml) <br/> [フルなTiFlash構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-tiflash.yaml)                                                                                                                                                                                 | これは、最小クラスタートポロジにTiFlashをデプロイするものです。TiFlashは列指向のストレージエンジンであり、徐々に標準的なクラスタートポロジになります。                                                            |
| [TiCDC](/ticdc/ticdc-overview.md)を使用した増分データのレプリケーション                   | [TiCDCトポロジのデプロイ](/ticdc-deployment-topology.md)                     | [シンプルなTiCDC構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-cdc.yaml) <br/> [フルなTiCDC構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-cdc.yaml)                                                                                                                                                                                             | これは、最小クラスタートポロジにTiCDCをデプロイするものです。TiCDCは、TiDB、MySQL、Kafka、MQ、およびストレージサービスなど、複数のダウンストリームプラットフォームをサポートしています。                                     |
| [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md)を使用した増分データのレプリケーション | [TiDB Binlogトポロジのデプロイ](/tidb-binlog-deployment-topology.md)         | [シンプルなTiDB Binlog構成テンプレート（ダウンストリームとしてMySQLを使用）](https://github.com/pingcap/docs/blob/master/config-templates/simple-tidb-binlog.yaml) <br/> [シンプルなTiDB Binlog構成テンプレート（ダウンストリームとしてファイルを使用）](https://github.com/pingcap/docs/blob/master/config-templates/simple-file-binlog.yaml) <br/> [フルなTiDB Binlog構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-tidb-binlog.yaml) | これは、最小クラスタートポロジにTiDB Binlogをデプロイするものです。                                                                                                       |
| Spark上でOLAPを使用                                                         | [TiSparkトポロジのデプロイ](/tispark-deployment-topology.md)                 | [シンプルなTiSpark構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-tispark.yaml) <br/> [フルなTiSpark構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-tispark.yaml)                                                                                                                                                                                 | これは、最小クラスタートポロジにTiSparkをデプロイするものです。TiSparkは、TiDB/TiKVの上でApache Sparkを実行し、OLAPクエリに答えるために構築されたコンポーネントです。現在、TiUPクラスターのTiSparkへのサポートはまだ**実験的**です。 |
| 1台のマシン上で複数のインスタンスをデプロイ                                                 | [ハイブリッドトポロジのデプロイ](/hybrid-deployment-topology.md)                   | [ハイブリッドデプロイメントのシンプルな構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-multi-instance.yaml) <br/> [ハイブリッドデプロイメントのフルな構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-multi-instance.yaml)                                                                                                                                                     | ディレクトリ、ポート、リソース比率、およびラベルの追加構成が必要な場合、デプロイトポロジも適用されます。                                                                                          |
| データセンター間でTiDBクラスターをデプロイ                                                | [地理的に分散したデプロイメントトポロジのデプロイ](/geo-distributed-deployment-topology.md) | [地理的に分散したデプロイメントの構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/geo-redundancy-deployment.yaml)                                                                                                                                                                                                                                                                                | このトポロジは、2つの都市に3つのデータセンターの典型的なアーキテクチャを取り上げています。地理的に分散したデプロイメントアーキテクチャと、注意が必要なキー構成を紹介しています。                                                     |

> **Note:**
>
> - グローバルに有効なパラメータについては、構成ファイルの`server_configs`セクションで対応するコンポーネントのこれらのパラメータを構成してください。
> - 特定のノードで有効なパラメータについては、このノードの`config`でこれらのパラメータを構成してください。
> - `log.slow-threshold`などの構成のサブカテゴリを示すために`.`を使用してください。その他のフォーマットについては、[TiUP構成テンプレート](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)を参照してください。
> - ターゲットマシンで作成されるユーザーグループ名を指定する必要がある場合は、[この例](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml#L7)を参照してください。

構成の詳細については、次の構成例を参照してください：

- [TiDB `config.toml.example`](https://github.com/pingcap/tidb/blob/release-7.1/config/config.toml.example)
- [TiKV `config.toml.example`](https://github.com/tikv/tikv/blob/master/etc/config-template.toml)
- [PD `config.toml.example`](https://github.com/pingcap/pd/blob/master/conf/config.toml)
- [TiFlash `config.toml.example`](https://github.com/pingcap/tiflash/blob/master/etc/config-template.toml)

## ステップ4. デプロイコマンドを実行する {#step-4-run-the-deployment-command}

> **Note:**
>
> TiUPを使用してTiDBをデプロイする際にセキュリティ認証のために秘密鍵や対話型パスワードを使用できます：
>
> - 秘密鍵を使用する場合は、`-i`または`--identity_file`を介して鍵のパスを指定してください。
> - パスワードを使用する場合は、`-p`フラグを追加してパスワードの対話型ウィンドウに入力してください。
> - パスワードフリーログインがターゲットマシンに構成されている場合は、認証は不要です。
>
> 一般的に、TiUPは`topology.yaml`ファイルで指定されたユーザーとグループをターゲットマシンに作成しますが、次の例外があります：
>
> - `topology.yaml`で構成されたユーザー名がすでにターゲットマシンに存在する場合。
> - ユーザーを作成する手順を明示的にスキップするためにコマンドラインで`--skip-create-user`オプションを使用した場合。

`deploy`コマンドを実行する前に、`check`および`check --apply`コマンドを使用して、クラスター内の潜在的なリスクを検出および自動的に修復してください：

1. 潜在的なリスクをチェックする：

   ```shell
   tiup cluster check ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
   ```

2. 自動修復を有効にする：

   ```shell
   tiup cluster check ./topology.yaml --apply --user root [-p] [-i /home/root/.ssh/gcp_rsa]
   ```

3. TiDBクラスターをデプロイする：

   ```shell
   tiup cluster deploy tidb-test v7.1.3 ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
   ```

上記の`tiup cluster deploy`コマンドでは：

- `tidb-test`はデプロイされるTiDBクラスターの名前です。
- `v7.1.3`はデプロイされるTiDBクラスターのバージョンです。最新のサポートされているバージョンは、`tiup list tidb`を実行して確認できます。
- `topology.yaml`は初期化構成ファイルです。
- `--user root`は、クラスターのデプロイを完了するためにターゲットマシンに`root`ユーザーとしてログインすることを示しています。`root`ユーザーは、ターゲットマシンに対して`ssh`および`sudo`権限を持っていることが期待されます。代わりに、デプロイを完了するために`ssh`および`sudo`権限を持つ他のユーザーを使用することもできます。
- `[-i]`および`[-p]`はオプションです。パスワードなしでターゲットマシンにログインを構成している場合、これらのパラメータは必要ありません。そうでない場合は、2つのパラメータのうちの1つを選択してください。`[-i]`は、ターゲットマシンにアクセス権を持つ`root`ユーザー（または`--user`で指定された他のユーザー）の秘密鍵です。`[-p]`は、ユーザーパスワードを対話型で入力するために使用されます。

出力ログの最後には、「`tidb-test`クラスターが正常にデプロイされました」と表示されます。これは、デプロイが成功したことを示しています。

## ステップ5. TiUPで管理されているクラスターをチェックする" {#step-5-check-the-clusters-managed-by-tiup}

```shell
tiup cluster list
```

TiUPは複数のTiDBクラスタを管理することができます。前述のコマンドは、TiUPによって現在管理されているすべてのクラスタの情報を出力します。これには、クラスタ名、デプロイユーザー、バージョン、およびシークレットキー情報が含まれます。

## ステップ6. デプロイされたTiDBクラスタのステータスを確認する {#step-6-check-the-status-of-the-deployed-tidb-cluster}

たとえば、次のコマンドを実行して、`tidb-test`クラスタのステータスを確認できます：

```shell
tiup cluster display tidb-test
```

予想される出力には、インスタンスID、役割、ホスト、リスニングポート、およびステータス（クラスタがまだ開始されていないため、ステータスは`Down`/`inactive`になります）、およびディレクトリ情報が含まれます。

## ステップ7. TiDBクラスタを開始する {#step-7-start-a-tidb-cluster}

TiUPクラスタv1.9.0以降、セーフスタートが新しい開始方法として導入されました。この方法を使用してデータベースを開始すると、データベースのセキュリティが向上します。この方法を使用することをお勧めします。

セーフスタート後、TiUPは自動的にTiDBルートユーザーのパスワードを生成し、そのパスワードをコマンドラインインターフェイスで返します。

> **注意:**
>
> - TiDBクラスタのセーフスタート後、パスワードなしでルートユーザーでTiDBにログインすることはできません。したがって、将来のログインのためにコマンド出力で返されたパスワードを記録する必要があります。
>
> - パスワードは1回だけ生成されます。それを記録しないか、忘れた場合は、パスワードを変更するために[ルートパスワードを忘れる](/user-account-management.md#forget-the-root-password)を参照してください。

方法1: セーフスタート

```shell
tiup cluster start tidb-test --init
```

もし出力が以下のようであれば、開始は成功です：

```shell
Started cluster `tidb-test` successfully.
The root password of TiDB database has been changed.
The new password is: 'y_+3Hwp=*AWz8971s6'.
Copy and record it to somewhere safe, it is only displayed once, and will not be stored.
The generated password can NOT be got again in future.
```

Method 2: 標準的な開始

```shell
tiup cluster start tidb-test
```

もし出力ログに``Started cluster `tidb-test` successfully``というメッセージが含まれている場合、起動は成功です。標準的な起動後、パスワードなしでrootユーザーを使用してデータベースにログインできます。

## ステップ8. TiDBクラスターの実行状態を確認します {#step-8-verify-the-running-status-of-the-tidb-cluster}

```shell
tiup cluster display tidb-test
```

If the output log shows `Up` status, the cluster is running properly.

## See also {#see-also}

If you have deployed [TiFlash](/tiflash/tiflash-overview.md) along with the TiDB cluster, see the following documents:

- [Use TiFlash](/tiflash/tiflash-overview.md#use-tiflash)
- [Maintain a TiFlash Cluster](/tiflash/maintain-tiflash.md)
- [TiFlash Alert Rules and Solutions](/tiflash/tiflash-alert-rules.md)
- [Troubleshoot TiFlash](/tiflash/troubleshoot-tiflash.md)

If you have deployed [TiCDC](/ticdc/ticdc-overview.md) along with the TiDB cluster, see the following documents:

- [Changefeed Overview](/ticdc/ticdc-changefeed-overview.md)
- [Manage Changefeed](/ticdc/ticdc-manage-changefeed.md)
- [Troubleshoot TiCDC](/ticdc/troubleshoot-ticdc.md)
- [TiCDC FAQs](/ticdc/ticdc-faq.md)
