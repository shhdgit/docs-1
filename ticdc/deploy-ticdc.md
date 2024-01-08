---
title: Deploy and Maintain TiCDC
summary: Learn the hardware and software recommendations for deploying and running TiCDC, and how to deploy and maintain it.
---

# デプロイと管理TiCDC {#deploy-and-maintain-ticdc}

このドキュメントでは、ハードウェアおよびソフトウェアの推奨事項を含むTiCDCクラスタをデプロイおよび管理する方法について説明します。新しいTiDBクラスタと一緒にTiCDCをデプロイするか、TiDBクラスタにTiCDCコンポーネントを追加することができます。

## ソフトウェアおよびハードウェアの推奨事項 {#software-and-hardware-recommendations}

本番環境では、TiCDCのソフトウェアおよびハードウェアの推奨事項は次のとおりです。

| Linux OS                 |    Version   |
| :----------------------- | :----------: |
| Red Hat Enterprise Linux | 7.3 以降のバージョン |
| CentOS                   | 7.3 以降のバージョン |

| CPU    | Memory  | Disk         | Network                   | 本番環境の最小要件に対するTiCDCクラスタインスタンスの数 |
| :----- | :------ | :----------- | :------------------------ | :----------------------------- |
| 16コア以上 | 64 GB以上 | 500 GB以上のSSD | 10ギガビットネットワークカード（2つが好ましい） | 2                              |

詳細については、[ソフトウェアおよびハードウェアの推奨事項](/hardware-and-software-requirements.md)を参照してください。

## TiUPを使用して新しいTiDBクラスタをデプロイし、TiCDCを含める {#deploy-a-new-tidb-cluster-that-includes-ticdc-using-tiup}

TiUPを使用して新しいTiDBクラスタをデプロイする際に、同時にTiCDCをデプロイすることもできます。TiUPがTiDBクラスタを起動するために使用する構成ファイルに`cdc_servers`セクションを追加するだけです。以下は例です。

```shell
cdc_servers:
  - host: 10.0.1.20
    gc-ttl: 86400
    data_dir: "/cdc-data"
  - host: 10.0.1.21
    gc-ttl: 86400
    data_dir: "/cdc-data"
```

その他の参照情報：

- 詳細な操作については、[初期化構成ファイルの編集](/production-deployment-using-tiup.md#step-3-initialize-cluster-topology-file)を参照してください。
- 詳細な設定可能なフィールドについては、[TiUPを使用した`cdc_servers`の構成](/tiup/tiup-cluster-topology-reference.md#cdc_servers)を参照してください。
- TiDBクラスタをデプロイする詳細な手順については、[TiUPを使用したTiDBクラスタのデプロイ](/production-deployment-using-tiup.md)を参照してください。

> **Note:**
>
> TiCDCをインストールする前に、TiUP制御マシンとTiCDCホストの間で[SSH相互信頼とパスワードなしでのsudoの手動構成](/check-before-deployment.md#manually-configure-the-ssh-mutual-trust-and-sudo-without-password)を確認してください。

## TiUPを使用して既存のTiDBクラスタにTiCDCを追加またはスケールアウト {#add-or-scale-out-ticdc-to-an-existing-tidb-cluster-using-tiup}

TiCDCクラスタのスケールアウト方法は、デプロイ方法と類似しています。スケールアウトにはTiUPを使用することをお勧めします。

1. `scale-out.yml`ファイルを作成して、TiCDCノード情報を追加します。以下は例です。

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

2. TiUP制御マシンでスケールアウトコマンドを実行します。

   ```shell
   tiup cluster scale-out <cluster-name> scale-out.yml
   ```

その他の使用例については、[TiCDCクラスタのスケールアウト](/scale-tidb-using-tiup.md#scale-out-a-ticdc-cluster)を参照してください。

## TiUPを使用して既存のTiDBクラスタからTiCDCを削除またはスケールイン {#delete-or-scale-in-ticdc-from-an-existing-tidb-cluster-using-tiup}

TiCDCノードをスケールインする際には、TiUPを使用することをお勧めします。以下はスケールインコマンドです。

```shell
tiup cluster scale-in <cluster-name> --node 10.0.1.4:8300
```

その他の使用例については、[TiCDCクラスタのスケールイン](/scale-tidb-using-tiup.md#scale-in-a-ticdc-cluster)を参照してください。

## TiUPを使用してTiCDCをアップグレード {#upgrade-ticdc-using-tiup}

TiUPを使用してTiDBクラスタをアップグレードし、その際にTiCDCもアップグレードすることができます。アップグレードコマンドを実行した後、TiUPは自動的にTiCDCコンポーネントをアップグレードします。以下は例です。

```shell
tiup update --self && \
tiup update --all && \
tiup cluster upgrade <cluster-name> <version> --transfer-timeout 600
```

> **Note:**
>
> 上記のコマンドでは、`<cluster-name>`と`<version>`を実際のクラスタ名とクラスタバージョンに置き換える必要があります。たとえば、バージョンはv7.1.3になります。

### アップグレードの注意事項 {#upgrade-cautions}

TiCDCクラスタをアップグレードする際には、以下に注意する必要があります。

- TiCDC v4.0.2では`changefeed`が再構成されました。詳細については、[構成ファイルの互換性に関する注意事項](/ticdc/ticdc-compatibility.md#cli-and-configuration-file-compatibility)を参照してください。

- アップグレード中に問題が発生した場合は、解決策については[アップグレードFAQ](/upgrade-tidb-using-tiup.md#faq)を参照してください。

- v6.3.0以降、TiCDCはローリングアップグレードをサポートしています。アップグレード中、レプリケーションのレイテンシが安定し、大幅に変動しません。ローリングアップグレードは、以下の条件を満たす場合に自動的に有効になります。

- TiCDCがv6.3.0以降であること。
  - TiUPがv1.11.3以降であること。
  - クラスタで少なくとも2つのTiCDCインスタンスが実行されていること。

## TiUPを使用してTiCDCクラスタの構成を変更 {#modify-ticdc-cluster-configurations-using-tiup}

このセクションでは、[`tiup cluster edit-config`](/tiup/tiup-component-cluster-edit-config.md)コマンドを使用してTiCDCの構成を変更する方法について説明します。以下の例では、`gc-ttl`のデフォルト値を`86400`から`172800`（48時間）に変更する必要があると仮定しています。

1. `tiup cluster edit-config`コマンドを実行します。`<cluster-name>`を実際のクラスタ名に置き換えてください。

   ```shell
    tiup cluster edit-config <cluster-name>
   ```

2. viエディタで、`cdc` [`server-configs`](/tiup/tiup-cluster-topology-reference.md#server_configs)を変更します。

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

   上記のコマンドでは、`gc-ttl`が48時間に設定されています。

3. `tiup cluster reload -R cdc`コマンドを実行して構成を再読み込みします。

## TiUPを使用してTiCDCを停止および開始 {#stop-and-start-ticdc-using-tiup}

TiUPを使用してTiCDCノードを簡単に停止および開始することができます。コマンドは次のとおりです。

- TiCDCを停止：`tiup cluster stop -R cdc`
- TiCDCを開始：`tiup cluster start -R cdc`
- TiCDCを再起動：`tiup cluster restart -R cdc`

## TiCDCのTLSを有効にする {#enable-tls-for-ticdc}

[TiDBコンポーネント間のTLSを有効にする](/enable-tls-between-components.md)を参照してください。

## コマンドラインツールを使用してTiCDCのステータスを表示 {#view-ticdc-status-using-the-command-line-tool}

以下のコマンドを実行してTiCDCクラスタのステータスを表示します。`v<CLUSTER_VERSION>`をTiCDCクラスタのバージョン（たとえば`v7.1.3`）に置き換える必要があります。

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
- `is-owner`：サービスプロセスが所有ノードであるかどうかを示します。
- `address`：サービスプロセスが外部にインターフェースを提供するアドレスを示します。
- `cluster-id`：TiCDCクラスタのIDを示します。デフォルト値は`default`です。"
