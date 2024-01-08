---
title: TiKV Control User Guide
summary: Use TiKV Control to manage a TiKV cluster.
---

# TiKV Controlユーザーガイド {#tikv-control-user-guide}

TiKV Control（`tikv-ctl`）は、クラスターを管理するために使用されるTiKVのコマンドラインツールです。そのインストールディレクトリは次のとおりです。

- クラスターがTiUPを使用して展開されている場合、`tikv-ctl`ディレクトリは`~/.tiup/components/ctl/{VERSION}/`ディレクトリにあります。

## TiUPでTiKV Controlを使用する {#use-tikv-control-in-tiup}

> **注意:**
>
> 使用するControlツールのバージョンは、クラスターのバージョンと一致していることが推奨されています。

`tikv-ctl`は、`tiup`コマンドにも統合されています。次のコマンドを実行して`tiup`ツールを呼び出します。

```shell
tiup ctl:v<CLUSTER_VERSION> tikv
```

```
Starting component `ctl`: /home/tidb/.tiup/components/ctl/v4.0.8/ctl tikv
TiKV Control (tikv-ctl)
Release Version:   4.0.8
Edition:           Community
Git Commit Hash:   83091173e960e5a0f5f417e921a0801d2f6635ae
Git Commit Branch: heads/refs/tags/v4.0.8
UTC Build Time:    2020-10-30 08:40:33
Rust Version:      rustc 1.42.0-nightly (0de96d37f 2019-12-19)
Enable Features:   jemalloc mem-profiling portable sse protobuf-codec
Profile:           dist_release

A tool for interacting with TiKV deployments.
USAGE:
    TiKV Control (tikv-ctl) [FLAGS] [OPTIONS] [SUBCOMMAND]
FLAGS:
    -h, --help                    Prints help information
        --skip-paranoid-checks    Skip paranoid checks when open rocksdb
    -V, --version                 Prints version information
OPTIONS:
        --ca-path <ca-path>              Set the CA certificate path
        --cert-path <cert-path>          Set the certificate path
        --config <config>                TiKV config path, by default it's <deploy-dir>/conf/tikv.toml
        --data-dir <data-dir>            TiKV data directory path, check <deploy-dir>/scripts/run.sh to get it
        --decode <decode>                Decode a key in escaped format
        --encode <encode>                Encode a key in escaped format
        --to-hex <escaped-to-hex>        Convert an escaped key to hex key
        --to-escaped <hex-to-escaped>    Convert a hex key to escaped key
        --host <host>                    Set the remote host
        --key-path <key-path>            Set the private key path
        --log-level <log-level>          Set the log level [default: warn]
        --pd <pd>                        Set the address of pd
SUBCOMMANDS:
    bad-regions           Get all regions with corrupt raft
    cluster               Print the cluster id
    compact               Compact a column family in a specified range
    compact-cluster       Compact the whole cluster in a specified range in one or more column families
    consistency-check     Force a consistency-check for a specified region
    decrypt-file          Decrypt an encrypted file
    diff                  Calculate difference of region keys from different dbs
    dump-snap-meta        Dump snapshot meta file
    encryption-meta       Dump encryption metadata
    fail                  Inject failures to TiKV and recovery
    help                  Prints this message or the help of the given subcommand(s)
    metrics               Print the metrics
    modify-tikv-config    Modify tikv config, eg. tikv-ctl --host ip:port modify-tikv-config -n
                          rocksdb.defaultcf.disable-auto-compactions -v true
    mvcc                  Print the mvcc value
    print                 Print the raw value
    raft                  Print a raft log entry
    raw-scan              Print all raw keys in the range
    recover-mvcc          Recover mvcc data on one node by deleting corrupted keys
    recreate-region       Recreate a region with given metadata, but alloc new id for it
    region-properties     Show region properties
    scan                  Print the range db range
    size                  Print region size
    split-region          Split the region
    store                 Print the store id
    tombstone             Set some regions on the node to tombstone by manual
    unsafe-recover        Unsafely recover the cluster when the majority replicas are failed
```

`tiup ctl:v<CLUSTER_VERSION> tikv`の後に対応するパラメータとサブコマンドを追加できます。

## 一般オプション {#general-options}

`tikv-ctl`には2つの操作モードがあります。

- リモートモード：`--host`オプションを使用して、TiKVのサービスアドレスを引数として受け入れます

  このモードでは、TiKVでSSLが有効になっている場合、`tikv-ctl`は関連する証明書ファイルを指定する必要があります。例：

  ```shell
  tikv-ctl --ca-path ca.pem --cert-path client.pem --key-path client-key.pem --host 127.0.0.1:20160 <subcommands>
  ```

  ただし、`tikv-ctl`がTiKVではなくPDと通信する場合、`--host`の代わりに`--pd`オプションを使用する必要があります。以下は例です：

  ```shell
  tikv-ctl --pd 127.0.0.1:2379 compact-cluster
  ```

  ```
  store:"127.0.0.1:20160" compact db:KV cf:default range:([], []) success!
  ```

- ローカルモード：

  - `--data-dir`オプションを使用して、ローカルのTiKVデータディレクトリパスを指定します。
  - `--config`オプションを使用して、ローカルのTiKV構成ファイルパスを指定します。

  このモードでは、実行中のTiKVインスタンスを停止する必要があります。

特に指定されていない限り、すべてのコマンドはリモートモードとローカルモードの両方をサポートしています。

さらに、`tikv-ctl`には2つのシンプルなコマンド`--to-hex`と`--to-escaped`があり、これらはキーの形式を簡単に変更するために使用されます。

一般的には、キーの`escaped`形式を使用します。例：

```shell
tikv-ctl --to-escaped 0xaaff
\252\377
tikv-ctl --to-hex "\252\377"
AAFF
```

> **Note:**
>
> コマンドラインでキーの`escaped`形式を指定する場合、それを二重引用符で囲む必要があります。そうしないと、bashはバックスラッシュを食べてしまい、誤った結果が返されます。

## サブコマンド、いくつかのオプションとフラグ {#subcommands-some-options-and-flags}

このセクションでは、`tikv-ctl`が詳細にサポートするサブコマンドについて説明します。いくつかのサブコマンドは多くのオプションをサポートしています。すべての詳細については、`tikv-ctl --help <subcommand>`を実行してください。

### Raft状態マシンの情報を表示 {#view-information-of-the-raft-state-machine}

`raft`サブコマンドを使用して、特定の瞬間のRaft状態マシンの状態を表示します。状態情報には、3つの構造体（**RegionLocalState**、**RaftLocalState**、**RegionApplyState**）と、特定のログの対応するエントリが含まれます。

上記の情報を取得するには、`region`および`log`サブコマンドをそれぞれ使用します。これらのサブコマンドは、リモートモードとローカルモードの両方を同時にサポートしています。

`region`サブコマンドの場合：

- 表示するリージョンを指定するには、`-r`オプションを使用します。複数のリージョンは`,`で区切られます。また、すべてのリージョンを表示するには`--all-regions`オプションを使用できます。なお、`-r`と`--all-regions`は同時に使用できません。
- 表示するリージョンの数を制限するには、`--limit`オプションを使用します（デフォルト：`16`）。
- 特定のキー範囲に含まれるリージョンをクエリするには、`--start`および`--end`オプションを使用します（デフォルト：範囲制限なし、16進数形式）。

たとえば、IDが`1239`のリージョンを出力するには、次のコマンドを使用します：

```shell
tikv-ctl --host 127.0.0.1:20160 raft region -r 1239
```

出力は次のとおりです：

```
"region id": 1239
"region state": {
    id: 1239,
    start_key: 7480000000000000FF4E5F728000000000FF1443770000000000FA,
    end_key: 7480000000000000FF4E5F728000000000FF21C4420000000000FA,
    region_epoch: {conf_ver: 1 version: 43},
    peers: [ {id: 1240 store_id: 1 role: Voter} ]
}
"raft state": {
    hard_state {term: 8 vote: 5 commit: 7}
    last_index: 8)
}
"apply state": {
    applied_index: 8 commit_index: 8 commit_term: 8
    truncated_state {index: 5 term: 5}
}
```

特定のキー範囲に含まれるリージョンをクエリするには、次のコマンドを使用します。

- キー範囲がリージョン範囲内にある場合、リージョン情報が出力されます。
- キー範囲がリージョン範囲と同じ場合、たとえば、指定されたキー範囲がリージョン `1239` と同じ場合、リージョン範囲が左閉区間および右開区間であるため、リージョン `1009` はリージョン `1239` の `end_key` を `start_key` として取り、リージョン `1009` の情報も出力されます。

```shell
tikv-ctl --host 127.0.0.1:20160 raft region --start 7480000000000000FF4E5F728000000000FF1443770000000000FA --end 7480000000000000FF4E5F728000000000FF21C4420000000000FA
```

出力は次のとおりです：

```
"region state": {
    id: 1009
    start_key: 7480000000000000FF4E5F728000000000FF21C4420000000000FA,
    end_key: 7480000000000000FF5000000000000000F8,
    ...
}
"region state": {
    id: 1239
    start_key: 7480000000000000FF4E5F728000000000FF06C6D60000000000FA,
    end_key: 7480000000000000FF4E5F728000000000FF1443770000000000FA,
    ...
}
```

### リージョンサイズの表示 {#view-the-region-size}

`size`コマンドを使用して、リージョンのサイズを表示します：

```shell
tikv-ctl --data-dir /path/to/tikv size -r 2
```

出力は次のとおりです：

```
region id: 2
cf default region size: 799.703 MB
cf write region size: 41.250 MB
cf lock region size: 27616
```

### 特定の範囲のMVCCを表示するためのスキャン {#scan-to-view-mvcc-of-a-specific-range}

`scan`コマンドの`--from`および`--to`オプションは、2つのエスケープされた生のキーの形式を受け入れ、`--show-cf`フラグを使用して表示する必要があるカラムファミリーを指定します。

```shell
tikv-ctl --data-dir /path/to/tikv scan --from 'zm' --limit 2 --show-cf lock,default,write
```

```
key: zmBootstr\377a\377pKey\000\000\377\000\000\373\000\000\000\000\000\377\000\000s\000\000\000\000\000\372
         write cf value: start_ts: 399650102814441473 commit_ts: 399650102814441475 short_value: "20"
key: zmDB:29\000\000\377\000\374\000\000\000\000\000\000\377\000H\000\000\000\000\000\000\371
         write cf value: start_ts: 399650105239273474 commit_ts: 399650105239273475 short_value: "\000\000\000\000\000\000\000\002"
         write cf value: start_ts: 399650105199951882 commit_ts: 399650105213059076 short_value: "\000\000\000\000\000\000\000\001"
```

### 指定されたキーのMVCCを表示 {#view-mvcc-of-a-given-key}

`scan`コマンドと同様に、`mvcc`コマンドを使用して、指定されたキーのMVCCを表示できます。

```shell
tikv-ctl --data-dir /path/to/tikv mvcc -k "zmDB:29\000\000\377\000\374\000\000\000\000\000\000\377\000H\000\000\000\000\000\000\371" --show-cf=lock,write,default
```

```
key: zmDB:29\000\000\377\000\374\000\000\000\000\000\000\377\000H\000\000\000\000\000\000\371
         write cf value: start_ts: 399650105239273474 commit_ts: 399650105239273475 short_value: "\000\000\000\000\000\000\000\002"
         write cf value: start_ts: 399650105199951882 commit_ts: 399650105213059076 short_value: "\000\000\000\000\000\000\000\001"
```

このコマンドでは、キーは生のキーのエスケープ形式でもあります。

### 生のキーをスキャン {#scan-raw-keys}

`raw-scan`コマンドは、RocksDBから直接スキャンします。データキーをスキャンするには、キーに `'z'` 接頭辞を追加する必要があることに注意してください。

スキャンする範囲を指定するには、`--from` と `--to` オプションを使用します（デフォルトでは無制限）。出力するキーの最大数を制限するには `--limit` を使用します（デフォルトでは30）。スキャンするcfを指定するには `--cf` を使用します（`default`、`write`、または `lock` にすることができます）。

```shell
tikv-ctl --data-dir /var/lib/tikv raw-scan --from 'zt' --limit 2 --cf default
```

```
key: "zt\200\000\000\000\000\000\000\377\005_r\200\000\000\000\000\377\000\000\001\000\000\000\000\000\372\372b2,^\033\377\364", value: "\010\002\002\002%\010\004\002\010root\010\006\002\000\010\010\t\002\010\n\t\002\010\014\t\002\010\016\t\002\010\020\t\002\010\022\t\002\010\024\t\002\010\026\t\002\010\030\t\002\010\032\t\002\010\034\t\002\010\036\t\002\010 \t\002\010\"\t\002\010s\t\002\010&\t\002\010(\t\002\010*\t\002\010,\t\002\010.\t\002\0100\t\002\0102\t\002\0104\t\002"
key: "zt\200\000\000\000\000\000\000\377\025_r\200\000\000\000\000\377\000\000\023\000\000\000\000\000\372\372b2,^\033\377\364", value: "\010\002\002&slow_query_log_file\010\004\002P/usr/local/mysql/data/localhost-slow.log"

Total scanned keys: 2
```

### 特定のキーの値を出力する {#print-a-specific-key-value}

キーの値を出力するには、`print`コマンドを使用します。

### リージョンに関するいくつかのプロパティを出力する {#print-some-properties-about-region}

TiKVはリージョンの状態の詳細を記録するため、リージョンのSSTファイルにいくつかの統計情報を書き込みます。これらのプロパティを表示するには、`tikv-ctl`を`region-properties`サブコマンドとともに実行します。

```shell
tikv-ctl --host localhost:20160 region-properties -r 2
```

```
num_files: 0
num_entries: 0
num_deletes: 0
mvcc.min_ts: 18446744073709551615
mvcc.max_ts: 0
mvcc.num_rows: 0
mvcc.num_puts: 0
mvcc.num_versions: 0
mvcc.max_row_versions: 0
middle_key_by_approximate_size:
```

プロパティは、リージョンが健全かどうかを確認するために使用できます。そうでない場合は、リージョンを修正するためにそれらを使用できます。たとえば、`middle_key_approximate_size` でリージョンを手動で分割します。

### 各 TiKV のデータを手動でコンパクト化 {#compact-data-of-each-tikv-manually}

各 TiKV のデータを手動でコンパクト化するには、`compact` コマンドを使用します。

- `--from` および `--to` オプションを使用して、エスケープされた生のキー形式でコンパクト範囲を指定します。設定しない場合、全範囲がコンパクト化されます。

- `--region` オプションを使用して、特定のリージョンの範囲をコンパクト化します。設定されている場合、`--from` および `--to` は無視されます。

- `-c` オプションを使用して、カラムファミリー名を指定します。デフォルト値は `default` です。オプションの値は `default`、`lock`、`write` です。

- `-d` オプションを使用して、コンパクト化を実行する RocksDB を指定します。デフォルト値は `kv` です。オプションの値は `kv` および `raft` です。

- `--threads` オプションを使用して、TiKV のコンパクト化の並行性を指定できます。デフォルト値は `8` です。一般的に、より高い並行性はより高速なコンパクト化速度をもたらしますが、サービスに影響を与える可能性があります。シナリオに基づいて適切な並行性カウントを選択する必要があります。

- `--bottommost` オプションを使用して、TiKV がコンパクト化を実行する際に、最下層のファイルを含めるか除外するかを指定します。値のオプションは `default`、`skip`、`force` です。デフォルト値は `default` です。
  - `default` は、Compaction Filter 機能が有効になっている場合にのみ、最下層のファイルが含まれることを意味します。
  - `skip` は、TiKV がコンパクト化を実行する際に、最下層のファイルが除外されることを意味します。
  - `force` は、TiKV がコンパクト化を実行する際に、最下層のファイルが常に含まれることを意味します。

- ローカルモードでデータをコンパクト化するには、次のコマンドを使用します：

  ```shell
  tikv-ctl --data-dir /path/to/tikv compact -d kv
  ```

- リモートモードでデータをコンパクト化するには、次のコマンドを使用します：

  ```shell
  tikv-ctl --host ip:port compact -d kv
  ```

### TiKV クラスタ全体のデータを手動でコンパクト化 {#compact-data-of-the-whole-tikv-cluster-manually}

`compact-cluster` コマンドを使用して、TiKV クラスタ全体のデータを手動でコンパクト化します。このコマンドのフラグは、`compact` コマンドの意味と使用法と同じです。唯一の違いは次のとおりです：

- `compact-cluster` コマンドの場合、`--pd` を使用して PD のアドレスを指定し、`tikv-ctl` がクラスタ内のすべての TiKV ノードをコンパクト化の対象として特定できるようにします。
- `compact` コマンドの場合、`--data-dir` または `--host` を使用して、単一の TiKV をコンパクト化の対象として指定します。

### リージョンを墓石に設定する {#set-a-region-to-tombstone}

`墓石` コマンドは、通常、sync-log が有効になっていない状況で使用され、Raft 状態機械に書き込まれたデータの一部が電源が切れたことによって失われた場合に使用されます。

TiKV インスタンスでは、このコマンドを使用して、一部のリージョンのステータスを墓石に設定できます。その後、インスタンスを再起動すると、これらのリージョンは、そのリージョンの Raft 状態機械が損傷したことによる再起動の失敗を回避するためにスキップされます。これらのリージョンには、他の TiKV インスタンスに十分な健全なレプリカが必要です。これにより、Raft メカニズムを介して読み取りと書き込みを継続できます。

一般的な場合、このリージョンの対応する Peer を `remove-peer` コマンドを使用して削除できます。"

```shell
pd-ctl operator add remove-peer <region_id> <store_id>
```

次に、`tikv-ctl`ツールを使用して、対応するTiKVインスタンスでリージョンを墓石に設定し、このリージョンの起動時のヘルスチェックをスキップします。

```shell
tikv-ctl --data-dir /path/to/tikv tombstone -p 127.0.0.1:2379 -r <region_id>
```

```
success!
```

しかし、いくつかのケースでは、このリージョンのこのピアを PD から簡単に削除することができない場合があります。そのため、`tikv-ctl` の `--force` オプションを指定して、強制的にピアを墓石に設定することができます。

```shell
tikv-ctl --data-dir /path/to/tikv tombstone -p 127.0.0.1:2379 -r <region_id>,<region_id> --force
```

```
success!
```

> **注意:**
>
> - `tombstone`コマンドはローカルモードのみをサポートしています。
> - `-p`オプションの引数は、`http`接頭辞なしでPDエンドポイントを指定します。PDエンドポイントを指定することで、PDが安全にTombstoneに切り替えられるかどうかを問い合わせます。

### TiKVに`consistency-check`リクエストを送信する {#send-a-consistency-check-request-to-tikv}

`consistency-check`コマンドを使用して、特定のリージョンの対応するRaft内のレプリカ間で一貫性チェックを実行します。チェックに失敗すると、TiKV自体がpanicになります。`--host`で指定されたTiKVインスタンスがリージョンのリーダーでない場合、エラーが報告されます。

```shell
tikv-ctl --host 127.0.0.1:20160 consistency-check -r 2
success!
tikv-ctl --host 127.0.0.1:20161 consistency-check -r 2
DebugClient::check_region_consistency: RpcFailure(RpcStatus { status: Unknown, details: Some("StringError(\"Leader is on store 1\")") })
```

> **Note:**
>
> - `consistency-check`コマンドの使用は**推奨されていません**。これはTiDBのガベージコレクションと互換性がなく、誤ってエラーを報告する可能性があります。
> - このコマンドはリモートモードのみをサポートしています。
> - このコマンドが`success!`を返しても、TiKVがpanicしていないかどうかを確認する必要があります。これは、このコマンドがリーダーに一貫性チェックを要求する提案であり、クライアントからはチェックプロセス全体が成功したかどうかがわからないためです。

### スナップショットメタのダンプ {#dump-snapshot-meta}

このサブコマンドは、指定されたパスのスナップショットメタファイルを解析して結果を出力します。

### Raft状態機械が壊れているリージョンを出力 {#print-the-regions-where-the-raft-state-machine-corrupts}

TiKVが起動している間にリージョンをチェックするのを避けるために、`tombstone`コマンドを使用して、Raft状態機械がエラーを報告するリージョンをTombstoneに設定できます。このコマンドを実行する前に、`bad-regions`コマンドを使用してエラーのあるリージョンを見つけ、複数のツールを組み合わせて自動処理を行います。

```shell
tikv-ctl --data-dir /path/to/tikv bad-regions
```

```
all regions are healthy
```

コマンドが正常に実行された場合、上記の情報が出力されます。コマンドが失敗した場合は、悪いリージョンのリストが出力されます。現在、検出できるエラーには、`last index`、`commit index`、`apply index`の不一致やRaftログの損失などが含まれます。スナップショットファイルの損傷などのその他の状況については、さらなるサポートが必要です。

### リージョンのプロパティを表示 {#view-region-properties}

- `/path/to/tikv`に展開されたTiKVインスタンスのリージョン2のプロパティをローカルで表示するには：

  ```shell
  tikv-ctl --data-dir /path/to/tikv/data region-properties -r 2
  ```

- `127.0.0.1:20160`で実行されているTiKVインスタンスのリージョン2のプロパティをオンラインで表示するには：

  ```shell
  tikv-ctl --host 127.0.0.1:20160 region-properties -r 2
  ```

### TiKV構成を動的に変更 {#modify-the-tikv-configuration-dynamically}

`modify-tikv-config`コマンドを使用して、構成引数を動的に変更できます。現在、動的に変更できるTiKV構成項目と詳細な変更は、SQLステートメントを使用して構成を変更するのと一致しています。詳細については、[TiKV構成を動的に変更](/dynamic-config.md#modify-tikv-configuration-dynamically)を参照してください。

- `-n`は構成項目のフルネームを指定するために使用されます。動的に変更できる構成項目のリストについては、[TiKV構成を動的に変更](/dynamic-config.md#modify-tikv-configuration-dynamically)を参照してください。
- `-v`は構成値を指定するために使用されます。

`shared block cache`のサイズを設定します：

```shell
tikv-ctl --host ip:port modify-tikv-config -n storage.block-cache.capacity -v 10GB
```

```
success
```

`shared block cache`が無効になっている場合、`write` CFの`block cache size`を設定します。

```shell
tikv-ctl --host ip:port modify-tikv-config -n rocksdb.writecf.block-cache-size -v 256MB
```

```
success
```

```shell
tikv-ctl --host ip:port modify-tikv-config -n raftdb.defaultcf.disable-auto-compactions -v true
```

```
success
```

```shell
tikv-ctl --host ip:port modify-tikv-config -n raftstore.sync-log -v false
```

```
success
```

コンパクションレート制限が蓄積されたコンパクション保留バイトを引き起こす場合、`rate-limiter-auto-tuned`モードを無効にするか、コンパクションフローの制限を高く設定します。

```shell
tikv-ctl --host ip:port modify-tikv-config -n rocksdb.rate-limiter-auto-tuned -v false
```

```
success
```

```shell
tikv-ctl --host ip:port modify-tikv-config -n rocksdb.rate-bytes-per-sec -v "1GB"
```

```
success
```

### 複数のレプリカの障害からサービスを回復させるためにリージョンに強制的に作用させる（非推奨） {#force-regions-to-recover-services-from-failure-of-multiple-replicas-deprecated}

> **警告:**
>
> この機能の使用は推奨されません。代わりに、`pd-ctl`でオンラインの安全でないリカバリを使用できます。これにより、サービスの停止などの追加の操作は不要です。詳細な紹介については、[オンラインの安全でないリカバリ](/online-unsafe-recovery.md)を参照してください。

`unsafe-recover remove-fail-stores`コマンドを使用して、リージョンのピアリストから失敗したマシンを削除できます。このコマンドを実行する前に、対象のTiKVストアのサービスを停止してファイルロックを解放する必要があります。

`-s`オプションは、カンマで区切られた複数の`store_id`を受け入れ、関連するリージョンを指定するために`-r`フラグを使用します。特定のストアのすべてのリージョンでこの操作を実行する必要がある場合は、単に`--all-regions`を指定するだけです。

> **警告:**
>
> - 何らかの誤操作が行われると、クラスタを回復するのが難しいかもしれません。潜在的なリスクに注意し、本番環境でこの機能を使用しないでください。
> - `--all-regions`オプションを使用する場合、このコマンドをクラスタに接続されている残りのストアすべてで実行する必要があります。これらの健全なストアが損傷したストアを回復する前にサービスの提供を停止することを確認する必要があります。そうしないと、リージョンのレプリカの一貫性のないピアリストが`split-region`または`remove-peer`を実行するとエラーが発生し、他のメタデータ間の不一致が発生し、最終的にリージョンが利用できなくなります。
> - `remove-fail-stores`を実行した後は、削除されたノードを再起動したり、これらのノードをクラスタに追加したりすることはできません。そうしないと、メタデータが不一致になり、最終的にリージョンが利用できなくなります。

```shell
tikv-ctl --data-dir /path/to/tikv unsafe-recover remove-fail-stores -s 3 -r 1001,1002
```

```
success!
```

```shell
tikv-ctl --data-dir /path/to/tikv unsafe-recover remove-fail-stores -s 4,5 --all-regions
```

その後、TiKVを再起動すると、リージョンは残りの健全なレプリカでサービスを提供し続けることができます。このコマンドは、複数のTiKVストアが損傷または削除された場合に一般的に使用されます。

> **Note:**
>
> - 指定されたリージョンのピアが存在するすべてのストアでこのコマンドを実行することが期待されています。
> - このコマンドはローカルモードのみをサポートしています。正常に実行されると `success!` が出力されます。

### MVCCデータの破損からの回復 {#recover-from-mvcc-data-corruption}

MVCCデータの破損によってTiKVが通常に実行できない場合には、`recover-mvcc`コマンドを使用します。これにより、3つのCF（"default"、"write"、"lock"）が相互に照合され、さまざまな種類の不整合から回復します。

- `-r`オプションを使用して、`region_id`で関連するリージョンを指定します。
- `-p`オプションを使用して、PDエンドポイントを指定します。

```shell
tikv-ctl --data-dir /path/to/tikv recover-mvcc -r 1001,1002 -p 127.0.0.1:2379
success!
```

> **Note:**
>
> - このコマンドはローカルモードのみをサポートしています。正常に実行されると `success!` が表示されます。
> - `-p` オプションの引数は、`http` 接頭辞なしで PD エンドポイントを指定します。PD エンドポイントを指定することで、指定された `region_id` が有効かどうかをクエリすることができます。
> - 指定されたリージョンのピアが存在するすべてのストアでこのコマンドを実行する必要があります。

### Ldb Command {#ldb-command}

`ldb` コマンドラインツールには、複数のデータアクセスおよびデータベース管理コマンドがあります。いくつかの例を以下に示します。詳細については、`tikv-ctl ldb` を実行すると表示されるヘルプメッセージを参照するか、RocksDB のドキュメントを確認してください。

データアクセスシーケンスの例：

既存の RocksDB を HEX でダンプするには：

```shell
tikv-ctl ldb --hex --db=/tmp/db dump
```

既存のRocksDBのマニフェストをダンプするには：

```shell
tikv-ctl ldb --hex manifest_dump --path=/tmp/db/MANIFEST-000001
```

あなたは、`--column_family=<string>`コマンドラインを使用して、クエリが対象とするカラムファミリーを指定できます。

`--try_load_options`は、データベースオプションファイルを読み込んでデータベースを開くためのオプションです。データベースが稼働している場合は、常にこのオプションをオンにしておくことをお勧めします。デフォルトオプションでデータベースを開くと、LSMツリーがめちゃくちゃになる可能性があり、これは自動的に回復できません。

### 暗号化メタデータのダンプ {#dump-encryption-metadata}

`encryption-meta`サブコマンドを使用して、暗号化メタデータをダンプできます。このサブコマンドでは、データファイルの暗号化情報と使用されるデータ暗号化キーのリストの2種類のメタデータをダンプできます。

データファイルの暗号化情報をダンプするには、`encryption-meta dump-file`サブコマンドを使用します。TiKVの展開に`data-dir`を指定するためのTiKV構成ファイルを作成する必要があります。

```
# conf.toml
[storage]
data-dir = "/path/to/tikv/data"
```

`--path`オプションを使用して、対象のデータファイルの絶対パスまたは相対パスを指定できます。データファイルが暗号化されていない場合、コマンドは空の出力を返す可能性があります。`--path`が指定されていない場合、すべてのデータファイルの暗号化情報が出力されます。

```shell
tikv-ctl --config=./conf.toml encryption-meta dump-file --path=/path/to/tikv/data/db/CURRENT
```

```
/path/to/tikv/data/db/CURRENT: key_id: 9291156302549018620 iv: E3C2FDBF63FC03BFC28F265D7E78283F method: Aes128Ctr
```

データ暗号化キーをダンプするには、`encryption-meta dump-key` サブコマンドを使用します。`data-dir` に加えて、構成ファイルで使用されている現在のマスターキーを指定する必要があります。マスターキーの構成方法については、[Encryption-At-Rest](/encryption-at-rest.md) を参照してください。また、このコマンドでは、`security.encryption.previous-master-key` 構成は無視され、マスターキーのローテーションはトリガーされません。

```
# conf.toml
[storage]
data-dir = "/path/to/tikv/data"

[security.encryption.master-key]
type = "kms"
key-id = "0987dcba-09fe-87dc-65ba-ab0987654321"
region = "us-west-2"
```

**Note:** もしマスターキーがAWS KMSキーである場合、 `tikv-ctl` はKMSキーへのアクセス権限を持っている必要があります。 AWS KMSキーへのアクセス権限は、環境変数、AWSデフォルトの構成ファイル、または適切なIAMロールを介して `tikv-ctl` に付与できます。使用方法については、AWSのドキュメントを参照してください。

`--ids` オプションを使用して、カンマ区切りのデータ暗号化キーIDのリストを指定できます。 `--ids` が指定されていない場合、すべてのデータ暗号化キーが出力され、最新のアクティブなデータ暗号化キーのIDとともに出力されます。

コマンドを使用すると、アクションが機密情報を公開する可能性があるという警告が表示されます。続行するには "I consent" と入力してください。

```shell
tikv-ctl --config=./conf.toml encryption-meta dump-key
```

```
This action will expose encryption key(s) as plaintext. Do not output the result in file on disk.
Type "I consent" to continue, anything else to exit: I consent
current key id: 9291156302549018620
9291156302549018620: key: 8B6B6B8F83D36BE2467ED55D72AE808B method: Aes128Ctr creation_time: 1592938357
```

```shell
tikv-ctl --config=./conf.toml encryption-meta dump-key --ids=9291156302549018620
```

```
This action will expose encryption key(s) as plaintext. Do not output the result in file on disk.
Type "I consent" to continue, anything else to exit: I consent
9291156302549018620: key: 8B6B6B8F83D36BE2467ED55D72AE808B method: Aes128Ctr creation_time: 1592938357
```

> **Note:**
>
> このコマンドはデータ暗号化キーを平文で公開します。本番環境では、出力をファイルにリダイレクトしないでください。後で出力ファイルを削除しても、ディスクから内容をきれいに消去することができない場合があります。

### 損傷したSSTファイルに関連する情報を出力 {#print-information-related-to-damaged-sst-files}

TiKVの損傷したSSTファイルは、TiKVプロセスがpanicを引き起こす可能性があります。TiDB v6.1.0より前では、これらのファイルはTiKVを直ちにpanicさせます。TiDB v6.1.0以降、TiKVプロセスはSSTファイルが損傷した1時間後にpanicします。

損傷したSSTファイルをクリーンアップするには、TiKV Controlで`bad-ssts`コマンドを実行して必要な情報を表示できます。以下は例としてのコマンドと出力です。

> **Note:**
>
> このコマンドを実行する前に、実行中のTiKVインスタンスを停止してください。

```shell
tikv-ctl --data-dir </path/to/tikv> bad-ssts --pd <endpoint>
```

```
--------------------------------------------------------
corruption info:
data/tikv-21107/db/000014.sst: Corruption: Bad table magic number: expected 9863518390377041911, found 759105309091689679 in data/tikv-21107/db/000014.sst

sst meta:
14:552997[1 .. 5520]['0101' seq:1, type:1 .. '7A7480000000000000FF0F5F728000000000FF0002160000000000FAFA13AB33020BFFFA' seq:2032, type:1] at level 0 for Column family "default"  (ID 0)
it isn't easy to handle local data, start key:0101

overlap region:
RegionInfo { region: id: 4 end_key: 7480000000000000FF0500000000000000F8 region_epoch { conf_ver: 1 version: 2 } peers { id: 5 store_id: 1 }, leader: Some(id: 5 store_id: 1) }

refer operations:
tikv-ctl ldb --db=/path/to/tikv/db unsafe_remove_sst_file 000014
tikv-ctl --data-dir=/path/to/tikv tombstone -r 4 --pd <endpoint>
--------------------------------------------------------
corruption analysis has completed
```

上記の出力から、損傷したSSTファイルの情報が最初に出力され、その後にメタ情報が出力されることがわかります。

- `sst meta`の部分では、`14`はSSTファイル番号を意味し、`552997`はファイルサイズを意味し、その後に最小および最大のシーケンス番号などの他のメタ情報が続きます。
- `overlap region`の部分は、関連するリージョンの情報を示します。この情報はPDサーバーを介して取得されます。
- `suggested operations`の部分では、損傷したSSTファイルをクリーンアップするための提案が提供されます。この提案を使用してファイルをクリーンアップし、TiKVインスタンスを再起動できます。

### リージョンの`RegionReadProgress`の状態を取得する {#get-the-state-of-a-region-s-regionreadprogress}

v6.5.4、v7.1.2、およびv7.3.0から、TiKVは`get-region-read-progress`サブコマンドを導入し、リゾルバと`RegionReadProgress`の最新の詳細を取得します。Grafana（`Min Resolved TS Region`および`Min Safe TS Region`）または`DataIsNotReady`ログから取得できるリージョンIDとTiKVを指定する必要があります。

- `--log`（オプション）：指定された場合、TiKVはこのTiKVのリージョンのリゾルバ内の最小の`start_ts`を`INFO`レベルでログに記録します。このオプションは、事前にresolved-tsをブロックする可能性のあるロックを特定するのに役立ちます。

- `--min-start-ts`（オプション）：指定された場合、TiKVはログからこの値よりも小さい`start_ts`を持つロックをフィルタリングします。これを使用してログに興味のあるトランザクションを指定できます。デフォルトでは`0`で、フィルタリングは行われません。

以下は例です：

```
./tikv-ctl --host 127.0.0.1:20160 get-region-read-progress -r 14 --log --min-start-ts 0
```

出力は次のとおりです：

```
Region read progress:
    exist: true,
    safe_ts: 0,
    applied_index: 92,
    pending front item (oldest) ts: 0,
    pending front item (oldest) applied index: 0,
    pending back item (latest) ts: 0,
    pending back item (latest) applied index: 0,
    paused: false,
Resolver:
    exist: true,
    resolved_ts: 0,
    tracked index: 92,
    number of locks: 0,
    number of transactions: 0,
    stopped: false,
```

サブコマンドは、Stale Readおよびsafe-tsに関連する問題の診断に役立ちます。詳細については、[TiKVのStale Readとsafe-tsの理解](/troubleshoot-stale-read.md)を参照してください。
