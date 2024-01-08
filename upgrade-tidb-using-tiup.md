---
title: Upgrade TiDB Using TiUP
summary: Learn how to upgrade TiDB using TiUP.
---

# TiUPを使用したTiDBのアップグレード {#upgrade-tidb-using-tiup}

このドキュメントは、次のアップグレードパスを対象としています。

- TiDB 4.0バージョンからTiDB 7.1へのアップグレード。
- TiDB 5.0-5.4バージョンからTiDB 7.1へのアップグレード。
- TiDB 6.0-6.6からTiDB 7.1へのアップグレード。
- TiDB 7.0からTiDB 7.1へのアップグレード。

> **Warning:**
>
> 1. TiFlashを5.3以前のバージョンから5.3以降にオンラインでアップグレードすることはできません。代わりに、最初に早期バージョンのすべてのTiFlashインスタンスを停止し、その後クラスタをオフラインでアップグレードする必要があります。他のコンポーネント（TiDBやTiKVなど）がオンラインアップグレードをサポートしていない場合は、[オンラインアップグレード](#online-upgrade)の警告に従って手順を実行してください。
> 2. アップグレードプロセス中にDDLステートメントを実行しないでください。そうしないと、未定義の動作の問題が発生する可能性があります。
> 3. DDLステートメントがクラスタで実行中の場合は、TiDBクラスタをアップグレードしないでください（通常は`ADD INDEX`や列の型変更など、時間のかかるDDLステートメントの場合）。アップグレードする前に、[`ADMIN SHOW DDL`](/sql-statements/sql-statement-admin-show-ddl.md)コマンドを使用してTiDBクラスタに実行中のDDLジョブがあるかどうかを確認することをお勧めします。クラスタにDDLジョブがある場合は、DDLの実行が終了するまでクラスタのアップグレードを待つか、[`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)コマンドを使用してDDLジョブをキャンセルしてからクラスタをアップグレードしてください。

> **Note:**
>
> - アップグレード対象のクラスタがv3.1またはそれ以前のバージョン（v3.0またはv2.1）の場合、v7.1.0またはそれ以降のv7.1.xバージョンへの直接アップグレードはサポートされていません。まずクラスタをv4.0にアップグレードし、その後ターゲットのTiDBバージョンにアップグレードする必要があります。
> - アップグレード対象のクラスタがv6.2より前の場合、アップグレード中にクラスタがv6.2またはそれ以降のバージョンにアップグレードすると、いくつかのシナリオでアップグレードが停滞する可能性があります。[問題の修正方法](#how-to-fix-the-issue-that-the-upgrade-gets-stuck-when-upgrading-to-v620-or-later-versions)を参照してください。
> - TiDBノードは、[`server-version`](/tidb-configuration-file.md#server-version)構成項目の値を使用して現在のTiDBバージョンを確認します。したがって、予期しない動作を避けるために、TiDBクラスタをアップグレードする前に、`server-version`の値を空に設定するか、現在のTiDBクラスタの実際のバージョンに設定する必要があります。

## アップグレードの注意事項 {#upgrade-caveat}

- TiDBは現在、バージョンのダウングレードやアップグレード後の以前のバージョンへのロールバックをサポートしていません。
- TiDB Ansibleを使用して管理されているv4.0クラスタの場合、[TiUPを使用したTiDBのアップグレード（v4.0）](https://docs.pingcap.com/tidb/v4.0/upgrade-tidb-using-tiup#import-tidb-ansible-and-the-inventoryini-configuration-to-tiup)に従って、新しい管理のためにクラスタをTiUP（`tiup cluster`）にインポートする必要があります。その後、このドキュメントに従ってクラスタをv7.1.3にアップグレードできます。
- v3.0より前のバージョンをv7.1.3にアップグレードする場合：
  1. [TiDB Ansible](https://docs.pingcap.com/tidb/v3.0/upgrade-tidb-using-ansible)を使用してこのバージョンを3.0にアップグレードします。
  2. TiUP（`tiup cluster`）を使用してTiDB Ansible構成をインポートします。
  3. [TiUPを使用したTiDBのアップグレード（v4.0）](https://docs.pingcap.com/tidb/v4.0/upgrade-tidb-using-tiup#import-tidb-ansible-and-the-inventoryini-configuration-to-tiup)に従って、3.0バージョンを4.0にアップグレードします。
  4. このドキュメントに従ってクラスタをv7.1.3にアップグレードします。
- TiDB Binlog、TiCDC、TiFlashなどのコンポーネントのバージョンをアップグレードすることがサポートされています。
- v6.3.0より前のバージョンからv6.3.0およびそれ以降のバージョンにTiFlashをアップグレードする場合、Linux AMD64アーキテクチャの場合はCPUがAVX2命令セットをサポートし、Linux ARM64アーキテクチャの場合はARMv8命令セットアーキテクチャをサポートしている必要があります。詳細については、[v6.3.0リリースノート](/releases/release-6.3.0.md#others)の説明を参照してください。
- 異なるバージョンの詳細な互換性の変更については、各バージョンの[リリースノート](/releases/release-notes.md)を参照してください。対応するリリースノートの「互換性の変更」セクションに従ってクラスタ構成を変更してください。
- v5.3より前のバージョンからv5.3またはそれ以降のバージョンにアップグレードするクラスタの場合、デフォルトでデプロイされるPrometheusはv2.8.1からv2.27.1にアップグレードされます。Prometheus v2.27.1はより多くの機能を提供し、セキュリティの問題を修正しています。v2.27.1では、v2.8.1と比較してアラートの時間表現が変更されています。詳細については、詳細については[Prometheus commit](https://github.com/prometheus/prometheus/commit/7646cbca328278585be15fa615e22f2a50b47d06)を参照してください。

## 準備 {#preparations}

このセクションでは、TiDBクラスタをアップグレードする前に必要な準備作業について説明します。これには、TiUPおよびTiUP Clusterコンポーネントのアップグレードが含まれます。

### ステップ1：互換性の変更を確認する {#step-1-review-compatibility-changes}

TiDBリリースノートの互換性の変更を確認します。アップグレードに影響する変更がある場合は、それに応じて対応してください。

以下は、v7.0.0から現在のバージョン（v7.1.3）にアップグレードする際に知っておく必要のある互換性の変更を提供します。v6.6.0またはそれ以前のバージョンから現在のバージョンにアップグレードする場合は、対応する[リリースノート](/releases/release-notes.md)で導入された中間バージョンの互換性の変更も確認する必要があります。

- TiDB v7.1.0 [互換性の変更](/releases/release-7.1.0.md#compatibility-changes)
- TiDB v7.1.1 [互換性の変更](/releases/release-7.1.1.md#compatibility-changes)
- TiDB v7.1.2 [互換性の変更](/releases/release-7.1.2.md#compatibility-changes)
- TiDB v7.1.3 [互換性の変更](/releases/release-7.1.3.md#compatibility-changes)

### ステップ2：TiUPまたはTiUPオフラインミラーのアップグレード {#step-2-upgrade-tiup-or-tiup-offline-mirror}

TiDBクラスタをアップグレードする前に、まずTiUPまたはTiUPミラーをアップグレードする必要があります。

#### TiUPおよびTiUP Clusterのアップグレード {#upgrade-tiup-and-tiup-cluster}

> **Note:**
>
> アップグレード対象のクラスタの制御マシンが`https://tiup-mirrors.pingcap.com`にアクセスできない場合は、このセクションをスキップして[オフラインミラーのアップグレード](#upgrade-tiup-offline-mirror)を参照してください。

1. TiUPのバージョンをアップグレードします。TiUPのバージョンが`1.11.3`またはそれ以降であることが推奨されています。

   ```shell
   tiup update --self
   tiup --version
   ```

2. TiUP Clusterのバージョンをアップグレードします。TiUP Clusterのバージョンが`1.11.3`またはそれ以降であることが推奨されています。

   ```shell
   tiup update cluster
   tiup cluster --version
   ```

#### TiUPオフラインミラーのアップグレード {#upgrade-tiup-offline-mirror}

> **Note:**
>
> アップグレード対象のクラスタがオフライン方法を使用して展開されていない場合は、このステップをスキップしてください。

新しいバージョンのTiUPミラーをダウンロードし、制御マシンにアップロードするために、[TiUPミラーのデプロイ](/production-deployment-using-tiup.md#deploy-tiup-offline)を参照してください。`local_install.sh`を実行した後、TiUPは上書きアップグレードを完了します。"

```shell
tar xzvf tidb-community-server-${version}-linux-amd64.tar.gz
sh tidb-community-server-${version}-linux-amd64/local_install.sh
source /home/tidb/.bash_profile
```

上書きアップグレード後、次のコマンドを実行して、サーバーとツールキットのオフラインミラーをサーバーディレクトリにマージします。

```bash
tar xf tidb-community-toolkit-${version}-linux-amd64.tar.gz
ls -ld tidb-community-server-${version}-linux-amd64 tidb-community-toolkit-${version}-linux-amd64
cd tidb-community-server-${version}-linux-amd64/
cp -rp keys ~/.tiup/
tiup mirror merge ../tidb-community-toolkit-${version}-linux-amd64
```

ミラーをマージした後、次のコマンドを実行して、TiUPクラスターコンポーネントをアップグレードします。

```shell
tiup update cluster
```

今、オフラインミラーは正常にアップグレードされました。上書き後のTiUP操作中にエラーが発生した場合、`manifest`が更新されていない可能性があります。TiUPを再実行する前に、`rm -rf ~/.tiup/manifests/*`を試してみることができます。

### ステップ3：TiUPトポロジー構成ファイルの編集 {#step-3-edit-tiup-topology-configuration-file}

> **Note:**
>
> 次のいずれかの状況が当てはまる場合は、このステップをスキップしてください。
>
> - オリジナルクラスターの構成パラメータを変更していない。または、`tiup cluster`を使用して構成パラメータを変更しましたが、それ以上の変更が必要ない。
> - アップグレード後、変更されていない構成項目にv7.1.2のデフォルトパラメータ値を使用したい場合。

1. `vi`編集モードに入ってトポロジーファイルを編集します：

   ```shell
   tiup cluster edit-config <cluster-name>
   ```

2. [トポロジー](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)構成テンプレートの形式を参照し、トポロジーファイルの`server_configs`セクションに変更したいパラメータを記入します。

3. 変更後、<kbd>:</kbd> + <kbd>w</kbd> + <kbd>q</kbd>を入力して変更を保存し、編集モードを終了します。変更を確認するには<kbd>Y</kbd>を入力します。

> **Note:**
>
> クラスターをv7.1.3にアップグレードする前に、v4.0で変更したパラメータがv7.1.3で互換性があることを確認してください。詳細については、[TiKV Configuration File](/tikv-configuration-file.md)を参照してください。

### ステップ4：クラスターのDDLおよびバックアップの状態を確認する {#step-4-check-the-ddl-and-backup-status-of-the-cluster}

アップグレード中の未定義の動作やその他の予期しない問題を回避するために、アップグレード前に次の項目を確認することをお勧めします。

- クラスターDDL：[`ADMIN SHOW DDL`](/sql-statements/sql-statement-admin-show-ddl.md)ステートメントを実行して、実行中のDDLジョブがあるかどうかを確認することをお勧めします。もし実行中のDDLジョブがある場合は、その実行を待つか、[`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)ステートメントを実行してキャンセルしてからアップグレードを行ってください。
- クラスターバックアップ：[`SHOW [BACKUPS|RESTORES]`](/sql-statements/sql-statement-show-backups.md)ステートメントを実行して、クラスターで実行中のバックアップまたはリストアタスクがあるかどうかを確認することをお勧めします。もし実行中のバックアップまたはリストアタスクがある場合は、その完了を待ってからアップグレードを行ってください。

### ステップ5：現在のクラスターのヘルスステータスを確認する {#step-5-check-the-health-status-of-the-current-cluster}

アップグレード中の未定義の動作やその他の問題を回避するために、アップグレード前に現在のクラスターのリージョンのヘルスステータスを確認することをお勧めします。これには、`check`サブコマンドを使用することができます。

```shell
tiup cluster check <cluster-name> --cluster
```

コマンドが実行された後、"リージョンのステータス"チェック結果が出力されます。

- 結果が "すべてのリージョンが健全です" の場合、現在のクラスタ内のすべてのリージョンが健全であり、アップグレードを継続できます。
- 結果が "リージョンが完全に健全でない: m miss-peer, n pending-peer" と "他の操作を行う前に健全でないリージョンを修正してください。" のプロンプトとともに表示される場合、現在のクラスタ内の一部のリージョンが異常です。チェック結果が "すべてのリージョンが健全です" になるまで異常をトラブルシューティングする必要があります。その後、アップグレードを継続できます。

## TiDBクラスタのアップグレード {#upgrade-the-tidb-cluster}

このセクションでは、TiDBクラスタのアップグレード方法とアップグレード後のバージョンの検証について説明します。

### TiDBクラスタを指定バージョンにアップグレード {#upgrade-the-tidb-cluster-to-a-specified-version}

クラスタをオンラインアップグレードとオフラインアップグレードのいずれかの方法でアップグレードできます。

デフォルトでは、TiUP Clusterはオンラインメソッドを使用してTiDBクラスタをアップグレードします。つまり、アップグレードプロセス中もTiDBクラスタはサービスを提供できます。オンラインメソッドでは、アップグレードと再起動の前に各ノードでリーダーが1つずつ移行されます。したがって、大規模なクラスタの場合、アップグレード全体の操作を完了するには長い時間がかかります。

データベースのメンテナンスのためにデータベースの停止が必要な場合、オフラインアップグレードメソッドを使用して迅速にアップグレード操作を実行できます。

#### オンラインアップグレード" {#online-upgrade}

```shell
tiup cluster upgrade <cluster-name> <version>
```

例えば、クラスタをv7.1.3にアップグレードしたい場合：

```shell
tiup cluster upgrade <cluster-name> v7.1.3
```

> **Note:**
>
> - オンラインアップグレードは、すべてのコンポーネントを1つずつアップグレードします。 TiKVのアップグレード中、TiKVインスタンスのすべてのリーダーはインスタンスを停止する前に排除されます。デフォルトのタイムアウト時間は5分（300秒）です。このタイムアウト時間が経過すると、インスタンスは直接停止されます。

> - クラスタを即座にアップグレードするには、`--force`パラメータを使用できますが、アップグレード中に発生したエラーは無視され、つまりアップグレードの失敗について通知されません。したがって、`--force`パラメータを慎重に使用してください。

> - 安定したパフォーマンスを維持するためには、TiKVインスタンスのすべてのリーダーが停止する前にインスタンスを停止してください。たとえば、`--transfer-timeout 3600`（単位：秒）のように、`--transfer-timeout`をより大きな値に設定できます。

> - 5.3より前のバージョンから5.3以降にTiFlashをアップグレードするには、TiFlashを停止してからアップグレードする必要があります。次の手順に従って、他のコンポーネントを中断することなくTiFlashをアップグレードできます：
>   1. TiFlashインスタンスを停止します：`tiup cluster stop <cluster-name> -R tiflash`
>   2. TiDBクラスターを再起動せずにアップグレードします（ファイルのみを更新）：`tiup cluster upgrade <cluster-name> <version> --offline`、例：`tiup cluster upgrade <cluster-name> v6.3.0 --offline`
>   3. TiDBクラスターを再読み込みします：`tiup cluster reload <cluster-name>`。再読み込み後、TiFlashインスタンスが起動し、手動で起動する必要はありません。

> - TiDB Binlogを使用してクラスターにローリングアップデートを適用する際に、新しいクラスターインデックステーブルを作成しないようにしてください。

#### オフラインアップグレード {#offline-upgrade}

1. オフラインアップグレードの前に、まずクラスター全体を停止する必要があります。

   ```shell
   tiup cluster stop <cluster-name>
   ```

2. `--offline`オプションを使用して`upgrade`コマンドを使用してオフラインアップグレードを実行します。`<cluster-name>`にクラスターの名前、`<version>`にアップグレードするバージョンを入力します。例：`v7.1.3`のように。

   ```shell
   tiup cluster upgrade <cluster-name> <version> --offline
   ```

3. アップグレード後、クラスターは自動的に再起動されません。再起動するには`start`コマンドを使用する必要があります。

   ```shell
   tiup cluster start <cluster-name>
   ```

### クラスターバージョンの確認 {#verify-the-cluster-version}

`display`コマンドを実行して、最新のクラスターバージョン`TiDB Version`を表示します。"

```shell
tiup cluster display <cluster-name>
```

```
Cluster type:       tidb
Cluster name:       <cluster-name>
Cluster version:    v7.1.3
```

## FAQ {#faq}

このセクションでは、TiUPを使用してTiDBクラスタを更新する際に遭遇する一般的な問題について説明します。

### エラーが発生し、アップグレードが中断された場合、このエラーを修正した後にアップグレードを再開する方法は？ {#if-an-error-occurs-and-the-upgrade-is-interrupted-how-to-resume-the-upgrade-after-fixing-this-error}

`tiup cluster upgrade`コマンドを再実行してアップグレードを再開します。アップグレード操作は以前にアップグレードされたノードを再起動します。アップグレードされたノードを再起動したくない場合は、`replay`サブコマンドを使用して操作を再試行します。

1. 操作の記録を表示するには、`tiup cluster audit`を実行します。

   ```shell
   tiup cluster audit
   ```

   失敗したアップグレード操作の記録を見つけ、この操作のIDを保持します。次のステップで使用する`<audit-id>`の値です。

2. 対応する操作を再試行するには、`tiup cluster replay <audit-id>`を実行します。

   ```shell
   tiup cluster replay <audit-id>
   ```

### v6.2.0以降のバージョンにアップグレードする際にアップグレードが停滞する問題を修正する方法は？ {#how-to-fix-the-issue-that-the-upgrade-gets-stuck-when-upgrading-to-v6-2-0-or-later-versions}

v6.2.0から、TiDBは[concurrent DDL framework](/ddl-introduction.md#how-the-online-ddl-asynchronous-change-works-in-tidb)をデフォルトで有効にして、並行してDDLを実行します。このフレームワークはDDLジョブのストレージをKVキューからテーブルキューに変更します。この変更により、アップグレードが特定のシナリオで停滞することがあります。この問題を引き起こす可能性のあるシナリオと対応する解決策は次のとおりです。

- プラグインの読み込みによりアップグレードが停滞する

  アップグレード中に、DDLステートメントの実行を必要とする特定のプラグインの読み込みが停滞の原因となることがあります。

  **解決策**：アップグレード中にプラグインの読み込みを避けます。アップグレードが完了した後にプラグインを読み込みます。

- オフラインアップグレードのために`kill -9`コマンドを使用することによりアップグレードが停滞する

  - 注意事項：オフラインアップグレードのために`kill -9`コマンドを使用しないでください。必要な場合は、新しいバージョンのTiDBノードを2分後に再起動します。
  - アップグレードが既に停滞している場合は、影響を受けるTiDBノードを再起動します。問題が発生した場合は、ノードを2分後に再起動することをお勧めします。

- DDLオーナーの変更によりアップグレードが停滞する

  複数のインスタンスのシナリオでは、ネットワークまたはハードウェアの障害によりDDLオーナーが変更されることがあります。アップグレードフェーズで未完了のDDLステートメントがある場合、アップグレードが停滞することがあります。

  **解決策**：

  1. 停滞しているTiDBノードを終了します（`kill -9`を使用しないでください）。
  2. 新しいバージョンのTiDBノードを再起動します。

### アップグレード中にリーダーの追放が長すぎるという警告が表示された場合、このステップをスキップして迅速にアップグレードする方法は？ {#the-evict-leader-has-waited-too-long-during-the-upgrade-how-to-skip-this-step-for-a-quick-upgrade}

`--force`を指定することができます。その後、PDリーダーの転送およびTiKVリーダーの追放のプロセスがアップグレード中にスキップされます。クラスタは直接再起動されてバージョンが更新されますが、これはオンラインで実行されているクラスタに大きな影響を与えます。次のコマンドでは、`<version>`はアップグレードするバージョン（たとえば`v7.1.3`）です。

```shell
tiup cluster upgrade <cluster-name> <version> --force
```

### TiDBクラスタをアップグレードした後、pd-ctlなどのツールのバージョンを更新する方法 {#how-to-update-the-version-of-tools-such-as-pd-ctl-after-upgrading-the-tidb-cluster}

対応するバージョンの`ctl`コンポーネントをインストールすることで、TiUPを使用してツールのバージョンをアップグレードできます。

```shell
tiup install ctl:v7.1.3
```
