---
title: tiup cluster patch
---

# tiup cluster patch {#tiup-cluster-patch}

クラスタが実行中の間にサービスのバイナリを動的に置き換える必要がある場合（つまり、置換プロセス中にクラスタを利用可能に保つ必要がある場合）、`tiup cluster patch`コマンドを使用できます。コマンドが実行された後、TiUPは次のことを行います。

- 置換のためのバイナリパッケージをターゲットマシンにアップロードします。
- ターゲットサービスがTiKV、TiFlash、またはTiDB Binlogなどのストレージサービスである場合、TiUPはまずAPI経由で関連ノードをオフラインにします。
- ターゲットサービスを停止します。
- バイナリパッケージを展開してサービスを置き換えます。
- ターゲットサービスを開始します。

## 構文 {#syntax}

```shell
tiup cluster patch <cluster-name> <package-path> [flags]
```

- `<cluster-name>`: 操作するクラスタの名前。
- `<package-path>`: 置換に使用されるバイナリパッケージへのパス。

### 準備 {#preparation}

`tiup cluster patch`コマンドを実行する前に、必要なバイナリパッケージをパッケージ化する必要があります。次の手順を実行してください。

1. 次の変数を決定します：

   - `${component}`: 置き換えるコンポーネントの名前（`tidb`、`tikv`、または`pd`など）。
   - `${version}`: コンポーネントのバージョン（`v7.1.3`または`v6.5.5`など）。
   - `${os}`: オペレーティングシステム（`linux`）。
   - `${arch}`: コンポーネントが実行されるプラットフォーム（`amd64`、`arm64`）。

2. 次のコマンドを使用して、現在のコンポーネントパッケージをダウンロードします：

   ```shell
   wget https://tiup-mirrors.pingcap.com/${component}-${version}-${os}-${arch}.tar.gz -O /tmp/${component}-${version}-${os}-${arch}.tar.gz
   ```

3. 一時ディレクトリを作成してファイルをパッケージ化し、そのディレクトリに移動します：

   ```shell
   mkdir -p /tmp/package && cd /tmp/package
   ```

4. 元のバイナリパッケージを展開します：

   ```shell
   tar xf /tmp/${component}-${version}-${os}-${arch}.tar.gz
   ```

5. 一時ディレクトリ内のファイル構造を確認します：

   ```shell
   find .
   ```

6. 一時ディレクトリ内の対応する場所にバイナリファイルまたは構成ファイルをコピーします。

7. 一時ディレクトリ内のすべてのファイルをパッケージ化します：

   ```shell
   tar czf /tmp/${component}-hotfix-${os}-${arch}.tar.gz *
   ```

上記の手順を完了した後、`tiup cluster patch`コマンドで`/tmp/${component}-hotfix-${os}-${arch}.tar.gz`を`<package-path>`として使用できます。

## オプション {#options}

### --overwrite {#overwrite}

- 特定のコンポーネント（TiDBやTiKVなど）をパッチした後、tiup clusterがコンポーネントをスケールアウトすると、TiUPはデフォルトで元のコンポーネントバージョンを使用します。将来クラスタがスケールアウトする際にパッチしたバージョンを使用するには、コマンドでオプション`--overwrite`を指定する必要があります。
- データ型：`BOOLEAN`
- このオプションはデフォルトで`false`値で無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

### --transfer-timeout {#transfer-timeout}

- PDまたはTiKVサービスを再起動する際、TiKV/PDはまずノードのリーダーを別のノードに転送します。転送プロセスには時間がかかるため、オプション`--transfer-timeout`を使用して最大待機時間（秒単位）を設定できます。タイムアウト後、TiUPはサービスを直接再起動します。
- データ型：`UINT`
- このオプションが指定されていない場合、TiUPは`600`秒待機した後にサービスを直接再起動します。

> **Note:**
>
> タイムアウト後にTiUPがサービスを直接再起動すると、サービスのパフォーマンスが不安定になる可能性があります。

### -N, --node {#n-node}

- 置き換えるノードを指定します。このオプションの値は、ノードIDのコンマ区切りリストです。`tiup cluster display`コマンドによって返される[クラスタステータステーブル](/tiup/tiup-component-cluster-display.md)の最初の列からノードIDを取得できます。
- データ型：`STRINGS`
- このオプションが指定されていない場合、TiUPはデフォルトで置き換えるノードを選択しません。

> **Note:**
>
> 同時に`-R, --role`オプションが指定されている場合、TiUPは`-N, --node`と`-R, --role`の両方の要件に一致するサービスノードを置き換えます。

### -R, --role {#r-role}

- 置き換える役割を指定します。このオプションの値は、ノードの役割のコンマ区切りリストです。`tiup cluster display`コマンドによって返される[クラスタステータステーブル](/tiup/tiup-component-cluster-display.md)の2番目の列からノードに展開された役割を取得できます。
- データ型：`STRINGS`
- このオプションが指定されていない場合、TiUPはデフォルトで置き換える役割を選択しません。

> **Note:**
>
> 同時に`-N, --node`オプションが指定されている場合、TiUPは`-N, --node`と`-R, --role`の両方の要件に一致するサービスノードを置き換えます。

### --offline {#offline}

- 現在のクラスタが実行されていないことを宣言します。このオプションが指定されている場合、TiUPはサービスリーダーを別のノードに追い出したり、サービスを再起動するのではなく、クラスタコンポーネントのバイナリファイルのみを置き換えます。
- データ型：`BOOLEAN`
- このオプションはデフォルトで`false`値で無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

### -h, --help {#h-help}

- ヘルプ情報を表示します。
- データ型：`BOOLEAN`
- このオプションはデフォルトで`false`値で無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

## 出力 {#outputs}

tiup-clusterの実行ログ。

[<< 前のページに戻る - TiUP Clusterコマンドリスト](/tiup/tiup-component-cluster.md#command-list)
