---
title: Upgrade TiDB Using TiUP
summary: Learn how to upgrade TiDB using TiUP.
---

# TiUPを使用してTiDBをアップグレードする {#upgrade-tidb-using-tiup}

このドキュメントは、次のアップグレードパスを対象としています：

- TiDB 4.0バージョンからTiDB 7.1へのアップグレード
- TiDB 5.0-5.4バージョンからTiDB 7.1へのアップグレード
- TiDB 6.0-6.6からTiDB 7.1へのアップグレード
- TiDB 7.0からTiDB 7.1へのアップグレード

> **警告：**
>
> 1. TiFlashを5.3以前のバージョンから5.3以降にオンラインでアップグレードすることはできません。その代わり、まず旧バージョンのすべてのTiFlashインスタンスを停止し、クラスターをオフラインでアップグレードする必要があります。他のコンポーネント（TiDBやTiKVなど）がオンラインアップグレードをサポートしていない場合は、[オンラインアップグレード](#オンラインアップグレード)の警告に従ってください。
> 2. アップグレードプロセス中にDDLステートメントを実行しないでください。そうしないと、未定義の動作の問題が発生する可能性があります。
> 3. DDLステートメントがクラスターで実行されているときにTiDBクラスターをアップグレードしないでください（通常、`ADD INDEX`や列タイプの変更などの時間のかかるDDLステートメントの場合）。アップグレードする前に、[`ADMIN SHOW DDL`](/sql-statements/sql-statement-admin-show-ddl.md)コマンドを使用して、TiDBクラスターに実行中のDDLジョブがあるかどうかを確認することをお勧めします。クラスターにDDLジョブがある場合は、クラスターをアップグレードするために、DDLの実行が終了するまで待機するか、[`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)コマンドを使用してDDLジョブをキャンセルしてからクラスターをアップグレードしてください。
>
> アップグレード前のTiDBバージョンがv7.1.0以降の場合は、前述の警告2と3を無視できます。詳細については、[TiDBスムーズアップグレード](/smooth-upgrade-tidb.md)を参照してください。

> **Note:**
>
> - アップグレードするクラスターがv3.1またはそれ以前のバージョン（v3.0またはv2.1）の場合、v7.1.0またはそれ以降のv7.1.xバージョンへの直接アップグレードはサポートされていません。まずクラスターをv4.0にアップグレードし、その後ターゲットのTiDBバージョンにアップグレードする必要があります。
> - アップグレードするクラスターがv6.2より前の場合、アップグレードが一部のシナリオで停止する可能性があります。[問題を修正する方法](#v620以降のバージョンにアップグレードするときにアップグレードが停止する問題を修正する方法)を参照してください。
> - TiDBノードは、現在のTiDBバージョンを確認するために[`server-version`](/tidb-configuration-file.md#server-version)構成項目の値を使用します。したがって、予期しない動作を避けるために、TiDBクラスターをアップグレードする前に、`server-version`の値を空に設定するか、現在のTiDBクラスターの実際のバージョンに設定する必要があります。

## アップグレードの注意事項 {#upgrade-caveat}

- TiDBは現在、バージョンのダウングレードやアップグレード後の旧バージョンへのロールバックをサポートしていません。
- v4.0クラスターは、[TiUPを使用してTiDBをアップグレードする（v4.0）](https://docs.pingcap.com/tidb/v4.0/upgrade-tidb-using-tiup#import-tidb-ansible-and-the-inventoryini-configuration-to-tiup)に従って新しい管理のためにTiUP（`tiup cluster`）にクラスターをインポートする必要があります。その後、このドキュメントに従ってクラスターをv7.1.3にアップグレードすることができます。
- v3.0より前のバージョンをv7.1.3にアップデートするには：
  1. [TiDB Ansible](https://docs.pingcap.com/tidb/v3.0/upgrade-tidb-using-ansible)を使用してこのバージョンを3.0にアップデートします。
  2. TiUP（`tiup cluster`）を使用してTiDB Ansibleの構成をインポートします。
  3. [TiUPを使用してTiDBをアップグレードする（v4.0）](https://docs.pingcap.com/tidb/v4.0/upgrade-tidb-using-tiup#import-tidb-ansible-and-the-inventoryini-configuration-to-tiup)に従って、3.0バージョンを4.0にアップデートします。
  4. このドキュメントに従ってクラスターをv7.1.3にアップグレードします。
- TiDB Binlog、TiCDC、TiFlashなどのコンポーネントのバージョンをアップグレードすることができます。
- TiFlashをv6.3.0より前のバージョンからv6.3.0およびそれ以降のバージョンにアップグレードする場合は、Linux AMD64アーキテクチャではCPUがAVX2命令セットをサポートし、Linux ARM64アーキテクチャではARMv8命令セットアーキテクチャをサポートする必要があります。詳細については、[v6.3.0リリースノート](/releases/release-6.3.0.md#others)の説明を参照してください。
- 異なるバージョンの詳細な互換性の変更については、各バージョンの[リリースノート](/releases/release-notes.md)を参照してください。対応するリリースノートの「互換性の変更」セクションに従ってクラスターの構成を変更してください。
- v5.3より前のバージョンからv5.3またはそれ以降のバージョンにアップグレードするクラスターの場合、デフォルトでデプロイされるPrometheusはv2.8.1からv2.27.1にアップグレードされます。Prometheus v2.27.1にはより多くの機能が提供され、セキュリティの問題が修正されています。v2.27.1では、v2.8.1と比較して、アラートの時間表現が変更されます。詳細については、[Prometheusのコミット](https://github.com/prometheus/prometheus/commit/7646cbca328278585be15fa615e22f2a50b47d06)を参照してください。

## 準備 {#preparations}

このセクションでは、TiDBクラスターをアップグレードする前に必要な準備作業を紹介します。これには、TiUPとTiUP Clusterコンポーネントのアップグレードが含まれます。

### ステップ1：互換性の変更を確認する {#step-1-review-compatibility-changes}

TiDBリリースノートの互換性の変更を確認してください。変更がアップグレードに影響する場合は、適切なアクションを実行してください。

以下は、現在のバージョン（v7.1.3）にアップグレードする際に知っておく必要のある互換性の変更を提供します。v6.6.0またはそれ以前のバージョンから現在のバージョンにアップグレードする場合は、対応する[リリースノート](/releases/release-notes.md)で導入された中間バージョンの互換性の変更も確認する必要があります。

- TiDB v7.1.0の[互換性の変更](/releases/release-7.1.0.md#compatibility-changes)
- TiDB v7.1.1の[互換性の変更](/releases/release-7.1.1.md#compatibility-changes)
- TiDB v7.1.2の[互換性の変更](/releases/release-7.1.2.md#compatibility-changes)
- TiDB v7.1.3の[互換性の変更](/releases/release-7.1.3.md#compatibility-changes)

### ステップ2：TiUPまたはTiUPオフラインミラーをアップグレードする {#step-2-upgrade-tiup-or-tiup-offline-mirror}

TiDBクラスターをアップグレードする前に、まずTiUPまたはTiUPミラーをアップグレードする必要があります。

#### TiUPとTiUP Clusterをアップグレードする {#upgrade-tiup-and-tiup-cluster}

> **Note:**
>
> アップグレードするクラスターの制御マシンが`https://tiup-mirrors.pingcap.com`にアクセスできない場合は、このセクションをスキップし、[TiUPオフラインミラーをアップグレードする](#upgrade-tiup-offline-mirror)を参照してください。

1. TiUPのバージョンをアップグレードします。TiUPのバージョンは`1.11.3`以上を推奨します。

   ```shell
   tiup update --self
   tiup --version
   ```

2. TiUP Clusterのバージョンをアップグレードします。TiUP Clusterのバージョンは`1.11.3`以上を推奨します。

   ```shell
   tiup update cluster
   tiup cluster --version
   ```

#### TiUPオフラインミラーのアップグレード {#upgrade-tiup-offline-mirror}

> **Note:**
>
> アップグレードするクラスターがオフラインメソッドを使用してデプロイされていない場合は、このステップをスキップしてください。

[Deploy a TiDB Cluster Using TiUP - Deploy TiUP offline](/production-deployment-using-tiup.md#deploy-tiup-offline)を参照して、新しいバージョンのTiUPミラーをダウンロードし、コントロールマシンにアップロードします。`local_install.sh`を実行すると、TiUPが上書きアップグレードを完了します。

```shell
tar xzvf tidb-community-server-${version}-linux-amd64.tar.gz
sh tidb-community-server-${version}-linux-amd64/local_install.sh
source /home/tidb/.bash_profile
```

上書きアップグレード後、次のコマンドを実行して、サーバーディレクトリにサーバーとツールキットのオフラインミラーをマージします。

```bash
tar xf tidb-community-toolkit-${version}-linux-amd64.tar.gz
ls -ld tidb-community-server-${version}-linux-amd64 tidb-community-toolkit-${version}-linux-amd64
cd tidb-community-server-${version}-linux-amd64/
cp -rp keys ~/.tiup/
tiup mirror merge ../tidb-community-toolkit-${version}-linux-amd64
```

TiUP Clusterコンポーネントをアップグレードするには、ミラーをマージした後、次のコマンドを実行してください。

```shell
tiup update cluster
```

今、オフラインミラーは正常にアップグレードされました。上書き後にTiUP操作中にエラーが発生した場合、`manifest`が更新されていない可能性があります。TiUPを再実行する前に、`rm -rf ~/.tiup/manifests/*`を試してみてください。

### ステップ3：TiUPトポロジー構成ファイルの編集 {#step-3-edit-tiup-topology-configuration-file}

> **Note:**
>
> 次のいずれかの状況が当てはまる場合は、このステップをスキップしてください：
>
> - オリジナルクラスターの構成パラメーターを変更していない場合。または、`tiup cluster`を使用して構成パラメーターを変更しましたが、さらなる変更は必要ありません。
> - アップグレード後、変更されていない構成項目については、v7.1.2のデフォルトのパラメーター値を使用したい場合。

1. `vi`編集モードに入り、トポロジーファイルを編集します：

   ```shell
   tiup cluster edit-config <cluster-name>
   ```

2. [トポロジー](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)構成テンプレートのフォーマットを参照し、トポロジーファイルの`server_configs`セクションに変更したいパラメーターを記入します。

3. 変更後、<kbd>:</kbd> + <kbd>w</kbd> + <kbd>q</kbd>を入力して変更を保存し、編集モードを終了します。変更を確認するには<kbd>Y</kbd>を入力します。

> **Note:**
>
> クラスターをv7.1.3にアップグレードする前に、v4.0で変更したパラメーターがv7.1.3で互換性があることを確認してください。詳細については、[TiKV構成ファイル](/tikv-configuration-file.md)を参照してください。

### ステップ4：現在のクラスターの健全性状態をチェックする {#step-4-check-the-health-status-of-the-current-cluster}

アップグレード中に未定義の動作やその他の問題を回避するために、アップグレード前に現在のクラスターのリージョンの健全性状態をチェックすることをお勧めします。これには、`check`サブコマンドを使用できます。

```shell
tiup cluster check <cluster-name> --cluster
```

コマンドが実行されると、「リージョンステータス」のチェック結果が出力されます。

- 結果が「すべてのリージョンが健全です」の場合、現在のクラスターのすべてのリージョンが健全であり、アップグレードを続行できます。
- 結果が「リージョンが完全に健全ではありません：m miss-peer、n pending-peer」で、「他の操作を行う前に健全でないリージョンを修正してください。」のプロンプトが表示される場合、現在のクラスターの一部のリージョンが異常です。チェック結果が「すべてのリージョンが健全です」になるまで、異常をトラブルシューティングする必要があります。その後、アップグレードを続行できます。

### ステップ5：クラスターのDDLとバックアップのステータスをチェックする {#step-5-check-the-ddl-and-backup-status-of-the-cluster}

アップグレード中に未定義の動作やその他の予期しない問題が発生しないようにするために、アップグレード前に次の項目をチェックすることをお勧めします。

- クラスターのDDL：[`ADMIN SHOW DDL`](/sql-statements/sql-statement-admin-show-ddl.md)ステートメントを実行して、実行中のDDLジョブがあるかどうかを確認することをお勧めします。はいの場合は、実行を待つか、[`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)ステートメントを実行してキャンセルしてからアップグレードを実行してください。
- クラスターバックアップ：[`SHOW [BACKUPS|RESTORES]`](/sql-statements/sql-statement-show-backups.md)ステートメントを実行して、クラスターで実行中のバックアップまたはリストアタスクがあるかどうかを確認することをお勧めします。はいの場合は、アップグレードを実行する前に完了するまで待ってください。

## TiDBクラスターをアップグレードする {#upgrade-the-tidb-cluster}

このセクションでは、TiDBクラスターをアップグレードし、アップグレード後のバージョンを検証する方法について説明します。

### 指定されたバージョンにTiDBクラスターをアップグレードする {#upgrade-the-tidb-cluster-to-a-specified-version}

TiUP Clusterを使用して、2つの方法のいずれかでクラスターをアップグレードできます：オンラインアップグレードとオフラインアップグレード。

デフォルトでは、TiUP Clusterはオンラインメソッドを使用してTiDBクラスターをアップグレードします。つまり、アップグレードプロセス中もTiDBクラスターはサービスを提供できます。オンラインメソッドでは、アップグレードと再起動の前に各ノードでリーダーが1つずつ移行されます。そのため、大規模なクラスターでは、アップグレード全体の操作を完了するには長時間かかります。

アプリケーションにデータベースのメンテナンスのために停止するためのメンテナンスウィンドウがある場合は、オフラインアップグレードメソッドを使用して、アップグレード操作を迅速に実行できます。

#### オンラインアップグレード {#online-upgrade}

```shell
tiup cluster upgrade <cluster-name> <version>
```

例えば、クラスターをv7.1.3にアップグレードしたい場合は：

```shell
tiup cluster upgrade <cluster-name> v7.1.3
```

> **Note:**
>
> - オンラインアップグレードはすべてのコンポーネントを1つずつアップグレードします。TiKVのアップグレード中、TiKVインスタンスのすべてのリーダーが削除されます。デフォルトのタイムアウト時間は5分（300秒）です。このタイムアウト時間が経過すると、インスタンスは直接停止されます。
>
> - リーダーを削除せずにクラスターを即時にアップグレードするには、`--force`パラメーターを使用できます。ただし、アップグレード中に発生したエラーは無視され、アップグレードの失敗については通知されません。そのため、`--force`パラメーターは注意して使用してください。
>
> - 安定したパフォーマンスを維持するために、TiKVインスタンスのすべてのリーダーが削除されることを確認してからインスタンスを停止してください。`--transfer-timeout`を大きな値に設定することもできます。例えば、`--transfer-timeout 3600`（単位：秒）です。
>
> - TiFlashを5.3より前のバージョンから5.3以降にアップグレードするには、TiFlashを停止してからアップグレードする必要があります。次の手順で、他のコンポーネントを中断することなくTiFlashをアップグレードできます：
>   1. TiFlashインスタンスを停止する：`tiup cluster stop <cluster-name> -R tiflash`
>   2. TiDBクラスターを再起動せずにアップグレードする（ファイルのみを更新する）：`tiup cluster upgrade <cluster-name> <version> --offline`、例えば`tiup cluster upgrade <cluster-name> v6.3.0 --offline`
>   3. TiDBクラスターをリロードする：`tiup cluster reload <cluster-name>`。リロード後、TiFlashインスタンスが起動され、手動で起動する必要はありません。
>
> - TiDB Binlogを使用してクラスターをローリングアップデートする際に、新しいクラスター化インデックステーブルを作成しないようにしてください。

#### オフラインアップグレード {#offline-upgrade}

1. オフラインアップグレードを行う前に、まずクラスター全体を停止する必要があります。

   ```shell
   tiup cluster stop <cluster-name>
   ```

2. `upgrade`コマンドを`--offline`オプションとともに使用してオフラインアップグレードを実行します。`<cluster-name>`にクラスターの名前を、`<version>`にアップグレードするバージョンを入力してください。例えば、`v7.1.3`です。

   ```shell
   tiup cluster upgrade <cluster-name> <version> --offline
   ```

3. アップグレード後、クラスターは自動的に再起動されません。`start`コマンドを使用して再起動する必要があります。

   ```shell
   tiup cluster start <cluster-name>
   ```

### クラスターバージョンを確認する {#verify-the-cluster-version}

`display`コマンドを実行して、最新のクラスターバージョン`TiDB Version`を表示します：

```shell
tiup cluster display <cluster-name>
```

    Cluster type:       tidb
    Cluster name:       <cluster-name>
    Cluster version:    v7.1.3

## FAQ {#faq}

このセクションでは、TiUPを使用してTiDBクラスターを更新する際に遭遇する一般的な問題について説明します。

### エラーが発生し、アップグレードが中断された場合、このエラーを修正した後にアップグレードを再開する方法は？ {#if-an-error-occurs-and-the-upgrade-is-interrupted-how-to-resume-the-upgrade-after-fixing-this-error}

`tiup cluster upgrade`コマンドを再実行してアップグレードを再開します。アップグレード操作は以前にアップグレードされたノードを再起動します。アップグレードされたノードを再起動したくない場合は、`replay`サブコマンドを使用して操作を再試行します：

1. `tiup cluster audit`を実行して操作レコードを確認します：

   ```shell
   tiup cluster audit
   ```

   失敗したアップグレード操作レコードを見つけ、この操作レコードのIDを保持します。IDは次のステップでの`<audit-id>`の値です。

2. `tiup cluster replay <audit-id>`を実行して対応する操作を再試行します：

   ```shell
   tiup cluster replay <audit-id>
   ```

### v6.2.0以降のバージョンにアップグレードする際にアップグレードが停止する問題を修正する方法は？ {#how-to-fix-the-issue-that-the-upgrade-gets-stuck-when-upgrading-to-v6-2-0-or-later-versions}

v6.2.0以降、TiDBは[並行DDLフレームワーク](/ddl-introduction.md#how-the-online-ddl-asynchronous-change-works-in-tidb)をデフォルトで有効にして、並行DDLを実行します。このフレームワークはDDLジョブのストレージをKVキューからテーブルキューに変更します。この変更により、アップグレードがいくつかのシナリオで停止する可能性があります。次のシナリオと対応する解決策を示します：

- プラグインのロードによりアップグレードが停止する

  アップグレード中に、DDLステートメントの実行を必要とする特定のプラグインをロードすると、アップグレードが停止する可能性があります。

  **解決策**：アップグレード中にプラグインをロードしないようにします。アップグレードが完了した後にプラグインをロードします。

- オフラインアップグレードのために`kill -9`コマンドを使用することによりアップグレードが停止する

  - 注意事項：オフラインアップグレードのために`kill -9`コマンドを使用しないでください。必要な場合は、新しいバージョンのTiDBノードを2分後に再起動します。
  - アップグレードが既に停止している場合は、影響を受けるTiDBノードを再起動します。問題が発生したばかりの場合は、2分後にノードを再起動することをお勧めします。

- DDLオーナーの変更によりアップグレードが停止する

  マルチインスタンスのシナリオでは、ネットワークやハードウェアの障害によりDDLオーナーが変更される可能性があります。アップグレードフェーズで未完了のDDLステートメントがある場合、アップグレードが停止する可能性があります。

  **解決策**：

  1. アップグレードが停止したTiDBノードを終了します（`kill -9`を使用しないでください）。
  2. 新しいバージョンのTiDBノードを再起動します。

### アップグレード中にリーダーの追放が長すぎるために停止する場合、クイックアップグレードのためにこのステップをスキップする方法は？ {#the-evict-leader-has-waited-too-long-during-the-upgrade-how-to-skip-this-step-for-a-quick-upgrade}

`--force`を指定することができます。その後、PDリーダーの転送プロセスとTiKVリーダーの追放プロセスはアップグレード中にスキップされます。クラスターはバージョンを更新するために直接再起動されますが、オンラインで実行されているクラスターに大きな影響を与えます。次のコマンドでは、`<version>`はアップグレードするバージョン（例：`v7.1.3`）です。

```shell
tiup cluster upgrade <cluster-name> <version> --force
```

### TiDBクラスターをアップグレードした後、pd-ctlなどのツールのバージョンを更新する方法は？ {#how-to-update-the-version-of-tools-such-as-pd-ctl-after-upgrading-the-tidb-cluster}

TiUPを使用して、対応するバージョンの`ctl`コンポーネントをインストールすることで、ツールのバージョンをアップグレードすることができます。

```shell
tiup install ctl:v7.1.3
```
