---
title: Quickly Deploy a Local TiDB Cluster
summary: Learn how to quickly deploy a local TiDB cluster using the playground component of TiUP.
---

# ローカルTiDBクラスターを迅速にデプロイする {#quickly-deploy-a-local-tidb-cluster}

TiDBクラスターは、複数のコンポーネントで構成された分散システムです。典型的なTiDBクラスターは、少なくとも3つのPDノード、3つのTiKVノード、および2つのTiDBノードで構成されています。TiDBを素早く体験したい場合、これらのコンポーネントを手動でデプロイすることは時間がかかり、複雑になる可能性があります。このドキュメントでは、TiUPのplaygroundコンポーネントとその使用方法について紹介します。

## TiUP playgroundの概要 {#tiup-playground-overview}

playgroundコンポーネントの基本的な使用法は次のとおりです：

```bash
tiup playground ${version} [flags]
```

`tiup playground`コマンドを直接実行すると、TiUPはローカルにインストールされたTiDB、TiKV、およびPDコンポーネントを使用するか、これらのコンポーネントの安定版をインストールして、1つのTiKVインスタンス、1つのTiDBインスタンス、1つのPDインスタンス、および1つのTiFlashインスタンスで構成されるTiDBクラスターを開始します。

このコマンドは実際に次の操作を実行します：

- このコマンドはplaygroundコンポーネントのバージョンを指定しないため、TiUPは最初にインストールされたplaygroundコンポーネントの最新バージョンを確認します。最新バージョンがv1.11.3であると仮定すると、このコマンドは`tiup playground:v1.11.3`と同じように機能します。
- TiUP playgroundを使用してTiDB、TiKV、およびPDコンポーネントをインストールしていない場合、playgroundコンポーネントはこれらのコンポーネントの最新安定版をインストールし、それらのインスタンスを開始します。
- このコマンドはTiDB、PD、およびTiKVコンポーネントのバージョンを指定しないため、TiUP playgroundはデフォルトで各コンポーネントの最新バージョンを使用します。最新バージョンがv7.1.3であると仮定すると、このコマンドは`tiup playground:v1.11.3 v7.1.3`と同じように機能します。
- このコマンドは各コンポーネントの数を指定しないため、TiUP playgroundはデフォルトで1つのTiDBインスタンス、1つのTiKVインスタンス、1つのPDインスタンス、および1つのTiFlashインスタンスで構成される最小のクラスターを開始します。
- 各TiDBコンポーネントを開始した後、TiUP playgroundはクラスターが正常に開始されたことを通知し、MySQLクライアントを介してTiDBクラスターに接続する方法や[TiDBダッシュボード](/dashboard/dashboard-intro.md)にアクセスする方法など、いくつかの有用な情報を提供します。

playgroundコンポーネントのコマンドラインフラグは次のとおりです：

```bash
Flags:
      --db int                   TiDBインスタンスの数を指定します（デフォルト：1）
      --db.host host             TiDBのリッスンアドレスを指定します
      --db.port int              TiDBのポートを指定します
      --db.binpath string        TiDBインスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --db.config string         TiDBインスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --db.timeout int           開始のためのTiDBの最大待ち時間を秒単位で指定します。0は制限なしを意味します
      --drainer int              クラスターのDrainerデータを指定します
      --drainer.binpath string   Drainerバイナリファイルの場所を指定します（オプション、デバッグ用）
      --drainer.config string    Drainer構成ファイルを指定します
  -h, --help                     tiupのヘルプ
      --host string              各コンポーネントのリッスンアドレスを指定します（デフォルト：`127.0.0.1`）。他のマシンへのアクセスのために提供される場合は、`0.0.0.0`に設定します
      --kv int                   TiKVインスタンスの数を指定します（デフォルト：1）
      --kv.binpath string        TiKVインスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --kv.config string         TiKVインスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --mode string              playgroundモードを指定します：'tidb'（デフォルト）および'tikv-slim'
      --pd int                   PDインスタンスの数を指定します（デフォルト：1）
      --pd.host host             PDのリッスンアドレスを指定します
      --pd.binpath string        PDインスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --pd.config string         PDインスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --pump int                 Pumpインスタンスの数を指定します。値が`0`でない場合、TiDB Binlogが有効になります。
      --pump.binpath string      Pumpバイナリファイルの場所を指定します（オプション、デバッグ用）
      --pump.config string       Pump構成ファイルを指定します（オプション、デバッグ用）
      -T, --tag string           playgroundのタグを指定します
      --ticdc int                TiCDCインスタンスの数を指定します（デフォルト：0）
      --ticdc.binpath string     TiCDCインスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --ticdc.config string      TiCDCインスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --tiflash int              TiFlashインスタンスの数を指定します（デフォルト：1）
      --tiflash.binpath string   TiFlashインスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --tiflash.config string    TiFlashインスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --tiflash.timeout int      開始のためのTiFlashの最大待ち時間を秒単位で指定します。0は制限なしを意味します
      -v, --version              playgroundのバージョンを指定します
      --without-monitor          PrometheusとGrafanaの監視機能を無効にします。このフラグを追加しない場合、監視機能はデフォルトで有効になります。
```

## 例 {#examples}

### 利用可能なTiDBバージョンを確認する {#check-available-tidb-versions}

```shell
tiup list tidb
```

### 特定のバージョンのTiDBクラスターを開始する {#start-a-tidb-cluster-of-a-specific-version}

```shell
tiup playground ${version}
```

`${version}`を対象のバージョン番号に置き換えます。

### NightlyバージョンのTiDBクラスターを開始する {#start-a-tidb-cluster-of-the-nightly-version}

```shell
tiup playground nightly
```

上記のコマンドでは、`nightly`はTiDBの最新開発バージョンを示します。

### PDのデフォルト構成を上書きする {#override-pd-s-default-configuration}

まず、[PD構成テンプレート](https://github.com/pingcap/pd/blob/master/conf/config.toml)をコピーする必要があります。コピーしたファイルを`~/config/pd.toml`に配置し、必要に応じて変更を加えた後、次のコマンドを実行してPDのデフォルト構成を上書きできます：

```shell
tiup playground --pd.config ~/config/pd.toml
```

### デフォルトのバイナリファイルを置き換える {#replace-the-default-binary-files}

デフォルトでは、playgroundを開始すると、各コンポーネントは公式ミラーからバイナリファイルを使用して開始されます。テスト用に一時的にコンパイルされたローカルバイナリファイルをクラスターに配置したい場合は、置換用の`--{comp}.binpath`フラグを使用できます。たとえば、次のコマンドを実行してTiDBのバイナリファイルを置き換えます：

```shell
tiup playground --db.binpath /xx/tidb-server
```

### 複数のコンポーネントインスタンスを開始する {#start-multiple-component-instances}

デフォルトでは、各TiDB、TiKV、およびPDコンポーネントについて1つのインスタンスが開始されます。各コンポーネントの複数のインスタンスを開始するには、次のフラグを追加します：

```shell
tiup playground --db 3 --pd 3 --kv 3
```

### TiDBクラスターを開始する際にタグを指定する {#specify-a-tag-when-starting-the-tidb-cluster}

TiUP playgroundを使用して開始したTiDBクラスターを停止した後、クラスターデータは自動的にクリーンアップされます。TiUP playgroundを使用してTiDBクラスターを開始し、クラスターデータが自動的にクリーンアップされないようにするには、クラスターを開始する際にタグを指定できます。タグを指定した後、`~/.tiup/data`ディレクトリにクラスターデータが保存されます。次のコマンドを実行してタグを指定します：

```shell
tiup playground --tag <tagname>
```

この方法で開始されたクラスターでは、クラスターが停止された後もデータファイルが保持されます。次回クラスターを開始する際にこのタグを使用して、クラスターが停止されてから保持されているデータを使用できます。

## playgroundで開始したTiDBクラスターに迅速に接続する {#quickly-connect-to-the-tidb-cluster-started-by-playground}

TiUPには`client`コンポーネントがあり、これを使用してplaygroundで開始したローカルTiDBクラスターを自動的に検出して接続できます。使用法は次のとおりです：

```shell
tiup client
```

このコマンドは、コンソール上で現在のマシンでplaygroundによって開始されたTiDBクラスターのリストを提供します。接続するTiDBクラスターを選択します。`Enter`をクリックすると、組み込みのMySQLクライアントが開いてTiDBに接続されます。

## 開始したクラスターの情報を表示する {#view-information-of-the-started-cluster}

```shell
tiup playground display
```

上記のコマンドは、次の結果を返します：

```
Pid    Role     Uptime
---    ----     ------
84518  pd       35m22.929404512s
84519  tikv     35m22.927757153s
84520  pump     35m22.92618275s
86189  tidb     exited
86526  tidb     34m28.293148663s
86190  drainer  35m19.91349249s
```

## クラスターをスケールアウトする {#scale-out-a-cluster}

クラスターをスケールアウトするためのコマンドラインパラメータは、クラスターを開始するためのものと類似しています。次のコマンドを実行して2つのTiDBインスタンスをスケールアウトできます：

```shell
tiup playground scale-out --db 2
```

## クラスターをスケールインする {#scale-in-a-cluster}

`tiup playground scale-in`コマンドで対応するインスタンスをスケールインするために`pid`を指定できます。`pid`を表示するには、`tiup playground display`を実行します。

```shell
tiup playground scale-in --pid 86526
```
