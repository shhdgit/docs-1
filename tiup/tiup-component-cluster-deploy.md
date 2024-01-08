---
title: tiup cluster deploy
---

# tiup cluster deploy {#tiup-cluster-deploy}

`tiup cluster deploy`コマンドは、新しいクラスタを展開するために使用されます。

## 構文 {#syntax}

```shell
tiup cluster deploy <cluster-name> <version> <topology.yaml> [flags]
```

- `<cluster-name>`: 新しいクラスタの名前で、既存のクラスタ名と同じにすることはできません。
- `<version>`: 展開するTiDBクラスタのバージョン番号、例：`v7.1.3`。
- `<topology.yaml>`: 準備された[topology file](/tiup/tiup-cluster-topology-reference.md)。

## オプション {#options}

### -u, --user {#u-user}

- ターゲットマシンに接続するために使用されるユーザー名を指定します。このユーザーは、ターゲットマシンで秘密のないsudoルート権限を持っている必要があります。
- データ型：`STRING`
- デフォルト：コマンドを実行する現在のユーザー。

### -i, --identity\_file {#i-identity-file}

- ターゲットマシンに接続するために使用されるキーファイルを指定します。
- データ型：`STRING`
- このオプションがコマンドで指定されていない場合、デフォルトで`~/.ssh/id_rsa`ファイルが使用されます。

### -p, --password {#p-password}

- ターゲットマシンに接続するために使用されるパスワードを指定します。このオプションを`-i/--identity_file`と同時に使用しないでください。
- データ型：`BOOLEAN`
- このオプションはデフォルトで無効になっており、デフォルト値は`false`です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

### --ignore-config-check {#ignore-config-check}

- このオプションは、構成チェックをスキップするために使用されます。コンポーネントのバイナリファイルが展開された後、TiDB、TiKV、およびPDコンポーネントの構成は、`<binary> --config-check <config-file>`を使用してチェックされます。`<binary>`は展開されたバイナリファイルのパスです。`<config-file>`はユーザー構成に基づいて生成された構成ファイルです。
- このオプションはデフォルトで無効になっており、デフォルト値は`false`です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。
- デフォルト：false

### --no-labels {#no-labels}

- このオプションは、ラベルチェックをスキップするために使用されます。
- 同じ物理マシンに2つ以上のTiKVノードが展開されている場合、PDはクラスタトポロジを学習できないリスクがあります。そのため、PDは同じリージョンの複数のレプリカを1つの物理マシンの異なるTiKVノードにスケジュールする可能性があります。これにより、この物理マシンが単一のポイントになります。このリスクを回避するために、ラベルを使用して、PDに同じリージョンを同じマシンにスケジュールしないように指示できます。ラベルの構成については、[Schedule Replicas by Topology Labels](/schedule-replicas-by-topology-labels.md)を参照してください。
- テスト環境では、このリスクが重要になる場合があり、`--no-labels`を使用してチェックをスキップできます。
- データ型：`BOOLEAN`
- このオプションはデフォルトで無効になっており、デフォルト値は`false`です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

### --skip-create-user {#skip-create-user}

- クラスタの展開中、tiup-clusterは、トポロジファイルで指定されたユーザー名が存在するかどうかをチェックします。存在しない場合、作成します。このチェックをスキップするには、`--skip-create-user`オプションを使用できます。
- データ型：`BOOLEAN`
- このオプションはデフォルトで無効になっており、デフォルト値は`false`です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

### -h, --help {#h-help}

- ヘルプ情報を表示します。
- データ型：`BOOLEAN`
- このオプションはデフォルトで無効になっており、デフォルト値は`false`です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

## 出力 {#output}

展開ログ。

[<< 前のページに戻る - TiUP Clusterコマンドリスト](/tiup/tiup-component-cluster.md#command-list)
