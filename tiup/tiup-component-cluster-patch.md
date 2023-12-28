---
title: tiup cluster patch
---

# tiup cluster patch {#tiup-cluster-patch}

クラスタが実行中の状態でサービスのバイナリを動的に置き換える必要がある場合（つまり、置き換えプロセス中もクラスタを利用可能な状態に保つ必要がある場合）、`tiup cluster patch`コマンドを使用することができます。コマンドが実行されると、TiUPは以下のことを行います：

- 置き換え対象のバイナリパッケージをターゲットマシンにアップロードします。
- ターゲットサービスがTiKV、TiFlash、またはTiDB Binlogなどのストレージサービスである場合、TiUPはまずAPIを介して関連するノードをオフラインにします。
- ターゲットサービスを停止します。
- バイナリパッケージを展開し、サービスを置き換えます。
- ターゲットサービスを起動します。

## 構文 {#syntax}

```shell
tiup cluster patch <cluster-name> <package-path> [flags]
```

- `<cluster-name>`: 操作するクラスタの名前。
- `<package-path>`: 置換に使用するバイナリパッケージのパス。

### 準備 {#preparation}

`tiup cluster patch` コマンドを実行する前に、必要なバイナリパッケージをパックする必要があります。以下の手順を実行してください。

1. 次の変数を決定します：

   - `${component}`: 置換するコンポーネントの名前（例：`tidb`、`tikv`、`pd`）。
   - `${version}`: コンポーネントのバージョン（例：`v7.1.3`、`v6.5.5`）。
   - `${os}`: オペレーティングシステム（`linux`）。
   - `${arch}`: コンポーネントが実行されるプラットフォーム（`amd64`、`arm64`）。

2. 次のコマンドを使用して、現在のコンポーネントパッケージをダウンロードします：

   ```shell
   wget https://tiup-mirrors.pingcap.com/${component}-${version}-${os}-${arch}.tar.gz -O /tmp/${component}-${version}-${os}-${arch}.tar.gz
   ```

3. ファイルをパックするための一時ディレクトリを作成し、そのディレクトリに移動します：

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

6. バイナリファイルや設定ファイルを一時ディレクトリ内の対応する場所にコピーします。

7. 一時ディレクトリ内のすべてのファイルをパックします：

   ```shell
   tar czf /tmp/${component}-hotfix-${os}-${arch}.tar.gz *
   ```

上記の手順を完了したら、`tiup cluster patch` コマンドで `/tmp/${component}-hotfix-${os}-${arch}.tar.gz` を `<package-path>` として使用できます。

## オプション {#options}

### --overwrite {#overwrite}

- 特定のコンポーネント（TiDB や TiKV など）をパッチした後、tiup cluster がそのコンポーネントをスケールアウトするとき、TiUP はデフォルトで元のコンポーネントバージョンを使用します。将来クラスタをスケールアウトするときにパッチしたバージョンを使用するには、コマンドでオプション `--overwrite` を指定する必要があります。
- データ型： `BOOLEAN`
- このオプションは `false` 値でデフォルトで無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true` 値を渡すか、値を渡さないでください。

### --transfer-timeout {#transfer-timeout}

- PD や TiKV サービスを再起動するとき、TiKV/PD はまずノードを再起動するためのリーダーを別のノードに転送します。転送プロセスには時間がかかるため、オプション `--transfer-timeout` を使用して最大待機時間（秒単位）を設定できます。タイムアウト後、TiUP はサービスを直接再起動します。
- データ型： `UINT`
- このオプションが指定されていない場合、TiUP は `600` 秒待機した後にサービスを直接再起動します。

> **Note:**
>
> タイムアウト後に TiUP がサービスを直接再起動すると、サービスのパフォーマンスが不安定になる可能性があります。

### -N, --node {#n-node}

- 置換するノードを指定します。このオプションの値は、ノード ID のカンマ区切りリストです。ノード ID は、`tiup cluster display` コマンドで返される [クラスタステータステーブル](/tiup/tiup-component-cluster-display.md) の最初の列から取得できます。
- データ型： `STRINGS`
- このオプションが指定されていない場合、TiUP はデフォルトで置換するノードを選択しません。

> **Note:**
>
> オプション `-R, --role` が同時に指定されている場合、TiUP は `-N, --node` と `-R, --role` の両方の要件に一致するサービスノードを置換します。

### -R, --role {#r-role}

- 置換するロールを指定します。このオプションの値は、ノードのロールのカンマ区切りリストです。ノードにデプロイされたロールは、`tiup cluster display` コマンドで返される [クラスタステータステーブル](/tiup/tiup-component-cluster-display.md) の 2 列目から取得できます。
- データ型： `STRINGS`
- このオプションが指定されていない場合、TiUP はデフォルトで置換するロールを選択しません。

> **Note:**
>
> オプション `-N, --node` が同時に指定されている場合、TiUP は `-N, --node` と `-R, --role` の両方の要件に一致するサービスノードを置換します。

### --offline {#offline}

- 現在のクラスターが実行されていないことを宣言します。このオプションが指定された場合、TiUPはサービスリーダーを別のノードに追い出したり、サービスを再起動したりするのではなく、クラスターコンポーネントのバイナリファイルのみを置き換えます。
- データタイプ： `BOOLEAN`
- このオプションは、デフォルトで `false` の値で無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true` の値を渡すか、値を渡さないようにします。

### -h、--help {#h-help}

- ヘルプ情報を出力します。
- データタイプ： `BOOLEAN`
- このオプションは、デフォルトで `false` の値で無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true` の値を渡すか、値を渡さないようにします。

## 出力 {#outputs}

tiup-clusterの実行ログ。

[<< 前のページに戻る - TiUP Clusterコマンドリスト](/tiup/tiup-component-cluster.md#command-list)
