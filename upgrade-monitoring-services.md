---
title: Upgrade Cluster Monitoring Services
summary: Learn how to upgrade the Prometheus, Grafana, and Alertmanager monitoring services for your TiDB cluster.
---

# TiDBクラスタの監視サービスをアップグレードする {#upgrade-tidb-cluster-monitoring-services}

TiDBクラスタを展開すると、TiUPはクラスタのために（Prometheus、Grafana、Alertmanagerなどの）監視サービスを自動的に展開します。クラスタをスケールアウトすると、TiUPはスケーリング中に新しく追加されたノードのための監視構成を自動的に追加します。TiUPによって自動的に展開される監視サービスは通常、これらのサードパーティの監視サービスの最新バージョンではありません。最新バージョンを使用するには、このドキュメントに従って監視サービスをアップグレードできます。

クラスタを管理する際、TiUPは独自の構成を使用して監視サービスの構成を上書きします。監視サービスの構成ファイルを直接アップグレードすると、その後のTiUP操作（`deploy`、`scale-out`、`scale-in`、`reload`）がクラスタ上でアップグレードを上書きし、エラーが発生する可能性があります。Prometheus、Grafana、Alertmanagerをアップグレードするには、構成ファイルを直接置き換えるのではなく、このドキュメントの手順に従ってください。

> **Note:**
>
> - 監視サービスが[TiUPを使用せずに手動で展開されている](/deploy-monitoring-services.md)場合、このドキュメントを参照せずに直接アップグレードできます。
> - TiDBと監視サービスの新しいバージョンとの互換性はテストされていないため、アップグレード後に一部の機能が期待どおりに動作しない可能性があります。問題がある場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を作成してください。
> - このドキュメントのアップグレード手順はTiUPバージョン1.9.0以降に適用されます。したがって、アップグレード前にTiUPバージョンを確認してください。
> - TiUPを使用してTiDBクラスタをアップグレードすると、TiUPは監視サービスをデフォルトバージョンに再展開します。TiDBのアップグレード後に監視サービスのアップグレードをやり直す必要があります。

## Prometheusのアップグレード {#upgrade-prometheus}

TiDBとの互換性を向上させるために、TiDBインストールパッケージで提供されるPrometheusインストールパッケージを使用することをお勧めします。TiDBインストールパッケージのPrometheusのバージョンは固定されています。より新しいPrometheusバージョンを使用したい場合は、[Prometheusリリースノート](https://github.com/prometheus/prometheus/releases)を参照して各バージョンの新機能を確認し、本番環境に適したバージョンを選択できます。また、PingCAPの技術スタッフに推奨バージョンを問い合わせることもできます。

次のアップグレード手順では、Prometheusウェブサイトから希望するバージョンのPrometheusインストールパッケージをダウンロードし、それをTiUPが使用できるPrometheusパッケージを作成する必要があります。

### ステップ1. Prometheusウェブサイトから新しいPrometheusインストールパッケージをダウンロードする {#step-1-download-a-new-prometheus-installation-package-from-the-prometheus-website}

[Prometheusダウンロードページ](https://prometheus.io/download/)から新しいインストールパッケージをダウンロードし、それを展開します。

### ステップ2. TiDBが提供するPrometheusインストールパッケージをダウンロードする {#step-2-download-the-prometheus-installation-package-provided-by-tidb}

1. [TiDBダウンロードページ](https://www.pingcap.com/download/)からTiDB **Server Package**をダウンロードし、それを展開します。
2. 展開されたファイルから、`prometheus-v{version}-linux-amd64.tar.gz`を見つけてそれを展開します。

   ```bash
   tar -xzf prometheus-v{version}-linux-amd64.tar.gz
   ```

### ステップ3. TiUPが使用できる新しいPrometheusパッケージを作成する {#step-3-create-a-new-prometheus-package-that-tiup-can-use}

1. [ステップ1](#ステップ1-prometheusウェブサイトから新しいprometheusインストールパッケージをダウンロードする)で展開されたファイルをコピーし、それを[ステップ2](#ステップ2-tidbが提供するprometheusインストールパッケージをダウンロードする)で展開された`./prometheus-v{version}-linux-amd64/prometheus`ディレクトリ内のファイルに置き換えます。
2. `./prometheus-v{version}-linux-amd64`ディレクトリを再圧縮し、新しい圧縮パッケージを`prometheus-v{new-version}.tar.gz`として名前を付けます。`{new-version}`は必要に応じて指定できます。

   ```bash
   cd prometheus-v{version}-linux-amd64
   tar -zcvf ../prometheus-v{new-version}.tar.gz ./
   ```

### ステップ4. 新しく作成したPrometheusパッケージを使用してPrometheusをアップグレードする {#step-4-upgrade-prometheus-using-the-newly-created-prometheus-package}

次のコマンドを実行してPrometheusをアップグレードします：

```bash
tiup cluster patch <cluster-name> prometheus-v{new-version}.tar.gz -R prometheus
```

アップグレード後、Prometheusサーバーのホームページ（通常は`http://<Prometheus-server-host-name>:9090`）に移動し、トップナビゲーションメニューで**Status**をクリックし、次に**Runtime & Build Information**ページを開いてPrometheusのバージョンを確認し、アップグレードが成功したかどうかを確認できます。

## Grafanaのアップグレード {#upgrade-grafana}

TiDBとの互換性を向上させるために、TiDBインストールパッケージで提供されるGrafanaインストールパッケージを使用することをお勧めします。TiDBインストールパッケージのGrafanaのバージョンは固定されています。より新しいGrafanaバージョンを使用したい場合は、[Grafanaリリースノート](https://grafana.com/docs/grafana/latest/whatsnew/)を参照して各バージョンの新機能を確認し、本番環境に適したバージョンを選択できます。また、PingCAPの技術スタッフに推奨バージョンを問い合わせることもできます。

次のアップグレード手順では、Grafanaウェブサイトから希望するバージョンのGrafanaインストールパッケージをダウンロードし、それをTiUPが使用できるGrafanaパッケージを作成する必要があります。

### ステップ1. Grafanaウェブサイトから新しいGrafanaインストールパッケージをダウンロードする {#step-1-download-a-new-grafana-installation-package-from-the-grafana-website}

1. [Grafanaダウンロードページ](https://grafana.com/grafana/download?pg=get\&plcmt=selfmanaged-box1-cta1)から新しいインストールパッケージをダウンロードします。必要に応じて`OSS`または`Enterprise`エディションを選択できます。
2. ダウンロードしたパッケージを展開します。

### ステップ2. TiDBが提供するGrafanaインストールパッケージをダウンロードする {#step-2-download-the-grafana-installation-package-provided-by-tidb}

1. [TiDBダウンロードページ](https://www.pingcap.com/download)からTiDB **Server Package**をダウンロードし、それを展開します。
2. 展開されたファイルから、`grafana-v{version}-linux-amd64.tar.gz`を見つけてそれを展開します。

   ```bash
   tar -xzf grafana-v{version}-linux-amd64.tar.gz
   ```

### ステップ3. TiUPが使用できる新しいGrafanaパッケージを作成する {#step-3-create-a-new-grafana-package-that-tiup-can-use}

1. [ステップ1](#ステップ1-grafanaウェブサイトから新しいgrafanaインストールパッケージをダウンロードする)で展開されたファイルをコピーし、それを[ステップ2](#ステップ2-tidbが提供するgrafanaインストールパッケージをダウンロードする)で展開された`./grafana-v{version}-linux-amd64/`ディレクトリ内のファイルに置き換えます。
2. `./grafana-v{version}-linux-amd64`ディレクトリを再圧縮し、新しい圧縮パッケージを`grafana-v{new-version}.tar.gz`として名前を付けます。`{new-version}`は必要に応じて指定できます。

   ```bash
   cd grafana-v{version}-linux-amd64
   tar -zcvf ../grafana-v{new-version}.tar.gz ./
   ```

### ステップ4. 新しく作成したGrafanaパッケージを使用してGrafanaをアップグレードする {#step-4-upgrade-grafana-using-the-newly-created-grafana-package}

次のコマンドを実行してGrafanaをアップグレードします：

```bash
tiup cluster patch <cluster-name> grafana-v{new-version}.tar.gz -R grafana
```

アップグレード後、Grafanaサーバーのホームページ（通常は`http://<Grafana-server-host-name>:3000`）に移動し、ページ上でGrafanaのバージョンを確認してアップグレードが成功したかどうかを確認できます。

## Alertmanagerのアップグレード {#upgrade-alertmanager}

TiDBインストールパッケージに含まれるAlertmanagerパッケージは、Prometheusウェブサイトから直接取得されています。そのため、Alertmanagerをアップグレードする際には、Prometheusウェブサイトから新しいAlertmanagerのバージョンをダウンロードしてインストールするだけで十分です。

### ステップ1. Prometheusウェブサイトから新しいAlertmanagerインストールパッケージをダウンロードする {#step-1-download-a-new-alertmanager-installation-package-from-the-prometheus-website}

[Prometheusダウンロードページ](https://prometheus.io/download/#alertmanager)から`alertmanager`インストールパッケージをダウンロードします。

### ステップ2. ダウンロードしたインストールパッケージを使用してAlertmanagerをアップグレードする {#step-2-upgrade-alertmanager-using-the-downloaded-installation-package}

次のコマンドを実行してAlertmanagerをアップグレードします：

```bash
tiup cluster patch <cluster-name> alertmanager-v{new-version}-linux-amd64.tar.gz -R alertmanager
```

アップグレード後、Alertmanagerサーバーのホームページ（通常は`http://<Alertmanager-server-host-name>:9093`）に移動し、トップナビゲーションメニューで**Status**をクリックし、Alertmanagerのバージョンを確認してアップグレードが成功したかどうかを確認できます。
