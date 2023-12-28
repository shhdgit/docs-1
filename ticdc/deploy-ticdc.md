---
title: Deploy and Maintain TiCDC
summary: Learn the hardware and software recommendations for deploying and running TiCDC, and how to deploy and maintain it.
---

# TiCDCのデプロイと管理 {#deploy-and-maintain-ticdc}

このドキュメントでは、ハードウェアとソフトウェアの推奨事項を含め、TiCDCクラスタのデプロイと管理方法について説明します。TiCDCは、新しいTiDBクラスタと一緒にデプロイするか、既存のTiDBクラスタにコンポーネントとして追加することができます。

## ソフトウェアとハードウェアの推奨事項 {#software-and-hardware-recommendations}

本番環境では、TiCDCのソフトウェアとハードウェアの推奨事項は以下の通りです：

| Linux OS                 |  バージョン |
| :----------------------- | :----: |
| Red Hat Enterprise Linux | 7.3 以降 |
| CentOS                   | 7.3 以降 |

| CPU    | メモリ     | ディスク         | ネットワーク                 | TiCDCクラスタインスタンスの数（本番環境の最小要件） |
| :----- | :------ | :----------- | :--------------------- | :--------------------------- |
| 16コア以上 | 64 GB以上 | 500 GB以上 SSD | 10ギガビットネットワークカード（2つ推奨） | 2                            |

詳細は、[ソフトウェアとハードウェアの推奨事項](/hardware-and-software-requirements.md)を参照してください。

## TiUPを使用してTiCDCを含む新しいTiDBクラスタをデプロイする {#deploy-a-new-tidb-cluster-that-includes-ticdc-using-tiup}

TiUPを使用して新しいTiDBクラスタをデプロイする際は、同時にTiCDCもデプロイすることができます。TiUPがTiDBクラスタを起動するために使用する設定ファイルに、`cdc_servers`セクションを追加するだけで済みます。以下は例です：

```shell
cdc_servers:
  - host: 10.0.1.20
    gc-ttl: 86400
    data_dir: "/cdc-data"
  - host: 10.0.1.21
    gc-ttl: 86400
    data_dir: "/cdc-data"
```

さらに参考:

- 詳細な操作については、[初期化構成ファイルの編集](/production-deployment-using-tiup.md#step-3-initialize-cluster-topology-file)を参照してください。
- 詳細な設定可能なフィールドについては、[TiUPを使用して`cdc_servers`を設定する](/tiup/tiup-cluster-topology-reference.md#cdc_servers)を参照してください。
- TiDBクラスタをデプロイする詳細な手順については、[TiUPを使用してTiDBクラスタをデプロイする](/production-deployment-using-tiup.md)を参照してください。

> **Note:**
>
> TiCDCをインストールする前に、TiUP制御マシンとTiCDCホスト間で[SSH相互信頼とパスワードなしでのsudoの手動設定](/check-before-deployment.md#manually-configure-the-ssh-mutual-trust-and-sudo-without-password)を行っていることを確認してください。

## TiUPを使用して既存のTiDBクラスタにTiCDCを追加またはスケールアウトする {#add-or-scale-out-ticdc-to-an-existing-tidb-cluster-using-tiup}

TiCDCクラスタをスケールアウトする方法は、デプロイする方法と似ています。スケールアウトにはTiUPを使用することをお勧めします。

1. `scale-out.yml`ファイルを作成し、TiCDCノードの情報を追加します。以下は例です:

   ```shell
   cdc_servers:
     - host: 10.1.1.1
       gc-ttl: 86400
       data_dir: /tidb-data/cdc-8300
     - host: 10.1.1.2
       gc-ttl: 86400
       data_dir: /tidb-data/cdc-8300
     - host: 10.0.1.4
       gc-ttl: 86400
       data_dir: /tidb-data/cdc-8300
   ```

2. TiUP制御マシンでスケールアウトコマンドを実行します:

   ```shell
   tiup cluster scale-out <cluster-name> scale-out.yml
   ```

より詳細な使用例については、[TiCDCクラスタのスケールアウト](/scale-tidb-using-tiup.md#scale-out-a-ticdc-cluster)を参照してください。

## 既存のTiDBクラスタからTiCDCを削除またはスケールインする {#delete-or-scale-in-ticdc-from-an-existing-tidb-cluster-using-tiup}

TiCDCノードをスケールインするには、TiUPを使用することをお勧めします。以下はスケールインコマンドです:

```shell
tiup cluster scale-in <cluster-name> --node 10.0.1.4:8300
```

より多くのユースケースについては、[TiUPを使用してTiDBをスケールインする](/scale-tidb-using-tiup.md#scale-in-a-ticdc-cluster)を参照してください。

## TiUPを使用してTiCDCをアップグレードする {#upgrade-ticdc-using-tiup}

TiUPを使用してTiDBクラスタをアップグレードすることができます。この際、TiCDCもアップグレードされます。アップグレードコマンドを実行すると、TiUPが自動的にTiCDCコンポーネントをアップグレードします。以下は例です：

```shell
tiup update --self && \
tiup update --all && \
tiup cluster upgrade <cluster-name> <version> --transfer-timeout 600
```

> **注意:**
>
> 前述のコマンドでは、実際のクラスタ名とクラスタバージョンを `<cluster-name>` と `<version>` に置き換える必要があります。例えば、バージョンは v7.1.3 になります。

### アップグレードの注意事項 {#upgrade-cautions}

TiCDC クラスタをアップグレードする際には、以下の点に注意する必要があります。

- TiCDC v4.0.2 では `changefeed` が再構成されます。詳細は、[コンフィグレーションファイルの互換性に関する注意事項](/ticdc/ticdc-compatibility.md#cli-and-configuration-file-compatibility)を参照してください。

- アップグレード中に問題が発生した場合は、解決策については [アップグレードの FAQ](/upgrade-tidb-using-tiup.md#faq) を参照してください。

- v6.3.0 以降、TiCDC はローリングアップグレードをサポートしています。アップグレード中、レプリケーションのレイテンシーは安定し、大きく変動しません。ローリングアップグレードは、以下の条件を満たす場合に自動的に有効になります。

- TiCDC が v6.3.0 以降であること。
  - TiUP が v1.11.3 以降であること。
  - クラスタ内で少なくとも 2 つの TiCDC インスタンスが実行されていること。

## TiUP を使用して TiCDC クラスタのコンフィグレーションを変更する {#modify-ticdc-cluster-configurations-using-tiup}

このセクションでは、[`tiup cluster edit-config`](/tiup/tiup-component-cluster-edit-config.md) コマンドを使用して TiCDC のコンフィグレーションを変更する方法について説明します。以下の例では、`gc-ttl` のデフォルト値を `86400` から `172800` (48 時間) に変更する必要があると仮定します。

1. `tiup cluster edit-config` コマンドを実行します。`<cluster-name>` を実際のクラスタ名に置き換えてください。

   ```shell
    tiup cluster edit-config <cluster-name>
   ```

2. vi エディタで、`cdc` の [`server-configs`](/tiup/tiup-cluster-topology-reference.md#server_configs) を変更します。

   ```shell
   server_configs:
     tidb: {}
     tikv: {}
     pd: {}
     tiflash: {}
     tiflash-learner: {}
     pump: {}
     drainer: {}
     cdc:
       gc-ttl: 172800
   ```

   上記のコマンドでは、`gc-ttl` が 48 時間に設定されています。

3. `tiup cluster reload -R cdc` コマンドを実行して、コンフィグレーションをリロードします。

## TiUP を使用して TiCDC を停止および開始する {#stop-and-start-ticdc-using-tiup}

TiUP を使用して、簡単に TiCDC ノードを停止および開始することができます。コマンドは以下のとおりです。

- TiCDC を停止する: `tiup cluster stop -R cdc`
- TiCDC を開始する: `tiup cluster start -R cdc`
- TiCDC を再起動する: `tiup cluster restart -R cdc`

## TiCDC に TLS を有効にする {#enable-tls-for-ticdc}

[TiDB コンポーネント間で TLS を有効にする](/enable-tls-between-components.md)を参照してください。

## コマンドラインツールを使用して TiCDC のステータスを表示する {#view-ticdc-status-using-the-command-line-tool}

以下のコマンドを実行して、TiCDC クラスタのステータスを表示します。`v<CLUSTER_VERSION>` を TiCDC クラスタのバージョン (例: `v7.1.3`) に置き換える必要があります。

```shell
tiup cdc:v<CLUSTER_VERSION> cli capture list --server=http://10.0.10.25:8300
```

```shell
[
  {
    "id": "806e3a1b-0e31-477f-9dd6-f3f2c570abdd",
    "is-owner": true,
    "address": "127.0.0.1:8300",
    "cluster-id": "default"
  },
  {
    "id": "ea2a4203-56fe-43a6-b442-7b295f458ebc",
    "is-owner": false,
    "address": "127.0.0.1:8301",
    "cluster-id": "default"
  }
]
```

- `id`：サービスプロセスのIDを示します。
- `is-owner`：サービスプロセスがオーナーノードであるかどうかを示します。
- `address`：サービスプロセスが外部にインターフェースを提供するアドレスを示します。
- `cluster-id`：TiCDCクラスタのIDを示します。デフォルト値は `default` です。
