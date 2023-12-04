---
title: TiCDC Server Configurations
summary: Learn the CLI and configuration parameters used in TiCDC.
---

# TiCDCサーバーの設定 {#ticdc-server-configurations}

このドキュメントでは、TiCDCで使用されるCLIおよび設定ファイルパラメータについて説明します。

## `cdc server` CLIパラメータ {#cdc-server-cli-parameters}

以下は、`cdc server`コマンドで使用可能なオプションの説明です：

-   `addr`: TiCDCのリスニングアドレス、HTTP APIアドレス、およびTiCDCサービスのPrometheusアドレスです。デフォルト値は`127.0.0.1:8300`です。
-   `advertise-addr`: クライアントがTiCDCにアクセスするための公開アドレスです。未指定の場合、値は`addr`と同じです。
-   `pd`: PDエンドポイントのカンマ区切りリストです。
-   `config`: TiCDCが使用する構成ファイルのアドレス（オプション）。このオプションはTiCDC v5.0.0以降でサポートされています。このオプションはTiUP v1.4.0以降でTiCDCデプロイメントで使用できます。詳細な構成の説明については、[TiCDC Changefeed Configurations](/ticdc/ticdc-changefeed-config.md)を参照してください。
-   `data-dir`: TiCDCがファイルを保存するためにディスクを使用する必要がある場合に使用するディレクトリを指定します。TiCDCが使用するソートエンジンとリドゥログは、一時ファイルを保存するためにこのディレクトリを使用します。このディレクトリの空きディスク容量が500 GiB以上であることを推奨します。TiUPを使用している場合は、[`cdc_servers`](/tiup/tiup-cluster-topology-reference.md#cdc_servers)セクションで`data_dir`を構成するか、デフォルトの`data_dir`パスを`global`で直接使用できます。
-   `gc-ttl`: TiCDCによって設定されたサービスレベルの`GC safepoint`のTTL（Time To Live）およびレプリケーションタスクが一時停止できる期間（秒）です。デフォルト値は`86400`で、つまり24時間です。注意：TiCDCレプリケーションタスクの一時停止はTiCDC GC safepointの進行に影響を与え、それは上流のTiDB GCの進行に影響を与えます。詳細については、[Complete Behavior of TiCDC GC safepoint](/ticdc/ticdc-faq.md#what-is-the-complete-behavior-of-ticdc-garbage-collection-gc-safepoint)を参照してください。
-   `log-file`: TiCDCプロセスが実行されているときにログが出力されるパスです。このパラメータが指定されていない場合、ログは標準出力（stdout）に書き込まれます。
-   `log-level`: TiCDCプロセスが実行されているときのログレベルです。デフォルト値は`"info"`です。
-   `ca`: TLS接続用のPEM形式のCA証明書ファイルのパスを指定します（オプション）。
-   `cert`: TLS接続用のPEM形式の証明書ファイルのパスを指定します（オプション）。
-   `cert-allowed-cn`: TLS接続用のPEM形式の共通名のパスを指定します（オプション）。
-   `key`: TLS接続用のPEM形式の秘密鍵ファイルのパスを指定します（オプション）。
-   `tz`: TiCDCサービスで使用されるタイムゾーンです。TiCDCは、内部で`TIMESTAMP`などの時刻データ型を変換するときや、下流にデータをレプリケートするときにこのタイムゾーンを使用します。デフォルトはプロセスが実行されているローカルタイムゾーンです。`time-zone`（`sink-uri`で）と`tz`を同時に指定した場合、内部のTiCDCプロセスは`tz`で指定されたタイムゾーンを使用し、シンクは下流にデータをレプリケートするために`time-zone`で指定されたタイムゾーンを使用します。`tz`で指定されたタイムゾーンが`time-zone`（`sink-uri`で）で指定されたタイムゾーンと同じであることを確認してください。
-   `cluster-id`: （オプション）TiCDCクラスターのIDです。デフォルト値は`default`です。`cluster-id`はTiCDCクラスターの一意の識別子です。同じ`cluster-id`を持つTiCDCノードは同じクラスターに属します。`cluster-id`の長さは最大で128文字です。`cluster-id`は`^[a-zA-Z0-9]+(-[a-zA-Z0-9]+)*$`のパターンに従う必要があり、次のいずれかではない必要があります：`owner`、`capture`、`task`、`changefeed`、`job`、`meta`。

## `cdc server`設定ファイルパラメータ {#cdc-server-configuration-file-parameters}

以下は、`cdc server`コマンドで`config`オプションで指定された構成ファイルの説明です：

```toml
# 以下のパラメータの設定方法はCLIパラメータと同じですが、CLIパラメータの方が優先度が高いです。
addr = "127.0.0.1:8300"
advertise-addr = ""
log-file = ""
log-level = "info"
data-dir = ""
gc-ttl = 86400 # 24 h
tz = "System"
cluster-id = "default"

[security]
  ca-path = ""
  cert-path = ""
  key-path = ""

# TiCDCとetcdサービス間のセッション期間で、秒単位で測定されます。このパラメータはオプションで、デフォルト値は10です。
capture-session-ttl = 10 # 10秒

# TiCDCクラスターのOwnerモジュールがレプリケーション進捗をプッシュしようとする間隔。このパラメータはオプションで、デフォルト値は `50000000` ナノ秒（つまり、50ミリ秒）です。このパラメータは2つの方法で設定できます：数値のみを指定する（例：`40000000` として設定すると、40000000ナノ秒、つまり40ミリ秒を表します）、または数値と単位の両方を指定する（例：`40ms` と直接設定する）。
owner-flush-interval = 50000000 # 50ミリ秒

# TiCDCクラスターのProcessorモジュールがレプリケーション進捗をプッシュしようとする間隔。このパラメータはオプションで、デフォルト値は `50000000` ナノ秒（つまり、50ミリ秒）です。このパラメータの設定方法は `owner-flush-interval` と同じです。
processor-flush-interval = 50000000 # 50ミリ秒

# [log]
# # zapログモジュールの内部エラーログの出力先。このパラメータはオプションで、デフォルト値は "stderr" です。
#   error-output = "stderr"
#   [log.file]
#     # 1つのログファイルの最大サイズ。このパラメータはオプションで、デフォルト値は300です。
#     max-size = 300 # 300 MiB
#     # ログファイルを保持する最大日数。このパラメータはオプションで、デフォルト値は `0` で、削除しないことを示します。
#     max-days = 0
#     # 保持するログファイルの数。このパラメータはオプションで、デフォルト値は `0` で、すべてのログファイルを保持することを示します。
#     max-backups = 0

#[sorter]
# デフォルトで開始される8つのpebble DBのSorterモジュールの共有pebbleブロックキャッシュのサイズ。デフォルト値は128です。
# cache-size-in-mb = 128
# sorterファイルがデータディレクトリ（`data-dir`）に対して保存されるディレクトリ。このパラメータはオプションで、デフォルト値は "/tmp/sorter" です。
# sorter-dir = "/tmp/sorter"

# [kv-client]
#   単一のRegionワーカーで使用できるスレッドの数。このパラメータはオプションで、デフォルト値は8です。
#   worker-concurrent = 8
#   TiCDCの共有スレッドプールでのスレッド数。主にKVイベントの処理に使用されます。このパラメータはオプションで、デフォルト値は0で、デフォルトのプールサイズはCPUコア数の2倍です。
#   worker-pool-size = 0
#   Region接続のリトライ期間。このパラメータはオプションで、デフォルト値は `60000000000` ナノ秒（つまり、1分）です。このパラメータは2つの方法で設定できます：数値のみを指定する（例：`50000000` として設定すると、50000000ナノ秒、つまり50ミリ秒を表します）、または数値と単位の両方を指定する（例：`50ms` と直接設定する）。
#   region-retry-duration = 60000000000
```
