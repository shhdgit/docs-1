---
title: TiCDC Server Configurations
summary: Learn the CLI and configuration parameters used in TiCDC.
---

# TiCDCサーバーの構成 {#ticdc-server-configurations}

このドキュメントでは、TiCDCで使用されるCLIおよび構成ファイルパラメータについて説明します。

## `cdc server` CLIパラメータ {#cdc-server-cli-parameters}

次に、`cdc server`コマンドで使用可能なオプションの説明です。

- `addr`：TiCDCのリッスンアドレス、TiCDCサービスのHTTP APIアドレス、およびPrometheusアドレス。デフォルト値は`127.0.0.1:8300`です。
- `advertise-addr`：クライアントがTiCDCにアクセスするための公開アドレス。指定されていない場合、`addr`と同じ値です。
- `pd`：PDエンドポイントのカンマ区切りリスト。
- `config`：TiCDCが使用する構成ファイルのアドレス（オプション）。このオプションは、TiCDC v5.0.0以降でサポートされています。このオプションは、TiUP v1.4.0以降でTiCDCデプロイメントで使用できます。詳細な構成の説明については、[TiCDC Changefeed Configurations](/ticdc/ticdc-changefeed-config.md)を参照してください。
- `data-dir`：TiCDCがファイルを保存するためにディスクを使用する必要があるディレクトリを指定します。TiCDCが使用するソートエンジンとリドゥログは、一時ファイルを保存するためにこのディレクトリを使用します。このディレクトリの空きディスク容量が500 GiB以上であることをお勧めします。TiUPを使用している場合、[`cdc_servers`](/tiup/tiup-cluster-topology-reference.md#cdc_servers)セクションで`data_dir`を構成するか、`global`でデフォルトの`data_dir`パスを直接使用できます。
- `gc-ttl`：TiCDCによって設定されたサービスレベルの`GC safepoint`のTTL（Time To Live）およびレプリケーションタスクが一時停止できる期間（秒単位）。デフォルト値は`86400`で、つまり24時間です。注意：TiCDCレプリケーションタスクの一時停止は、TiCDC GC safepointの進行状況に影響を与えます。つまり、上流のTiDB GCの進行状況に影響を与えます。詳細については、[Complete Behavior of TiCDC GC safepoint](/ticdc/ticdc-faq.md#what-is-the-complete-behavior-of-ticdc-garbage-collection-gc-safepoint)を参照してください。
- `log-file`：TiCDCプロセスが実行されているときにログが出力されるパス。このパラメータが指定されていない場合、ログは標準出力（stdout）に書き込まれます。
- `log-level`：TiCDCプロセスが実行されているときのログレベル。デフォルト値は`"info"`です。
- `ca`：TLS接続用のCA証明書ファイルのパス（オプション）。
- `cert`：TLS接続用の証明書ファイルのパス（オプション）。
- `cert-allowed-cn`：TLS接続用の共通名のパス（オプション）。
- `key`：TLS接続用の秘密鍵ファイルのパス（オプション）。
- `tz`：TiCDCサービスで使用されるタイムゾーン。TiCDCは、内部的に`TIMESTAMP`などの時間データ型を変換するときや、下流にデータをレプリケートするときにこのタイムゾーンを使用します。デフォルトはプロセスが実行されるローカルタイムゾーンです。`time-zone`（`sink-uri`内）と`tz`を同時に指定する場合、内部のTiCDCプロセスは`tz`で指定されたタイムゾーンを使用し、シンクは`time-zone`で指定されたタイムゾーンを使用して下流にデータをレプリケートします。`tz`で指定されたタイムゾーンが`time-zone`（`sink-uri`内）で指定されたタイムゾーンと同じであることを確認してください。
- `cluster-id`：（オプション）TiCDCクラスターのID。デフォルト値は`default`です。`cluster-id`はTiCDCクラスターの一意の識別子です。同じ`cluster-id`を持つTiCDCノードは同じクラスターに属します。`cluster-id`の長さは最大128文字です。`cluster-id`は`^[a-zA-Z0-9]+(-[a-zA-Z0-9]+)*$`のパターンに従う必要があり、次のいずれかであってはなりません：`owner`、`capture`、`task`、`changefeed`、`job`、`meta`。

## `cdc server`構成ファイルパラメータ {#cdc-server-configuration-file-parameters}

次に、`cdc server`コマンドの`config`オプションで指定された構成ファイルについて説明します：

```toml
# 以下のパラメータの構成方法はCLIパラメータと同じですが、CLIパラメータの方が優先度が高いです。
addr = "127.0.0.1:8300"
advertise-addr = ""
log-file = ""
log-level = "info"
data-dir = ""
gc-ttl = 86400 # 24 h
tz = "System"
cluster-id = "default"
# このパラメータは、GOGCのチューニングのための最大メモリ閾値（バイト単位）を指定します。閾値を小さく設定すると、GCの頻度が増加します。閾値を大きく設定すると、GCの頻度が減少し、TiCDCプロセスのメモリリソースをより多く消費します。メモリ使用量がこの閾値を超えると、GOGC Tunerは動作を停止します。デフォルト値は0で、GOGC Tunerが無効であることを示します。
gc-tuner-memory-threshold = 0

[security]
  ca-path = ""
  cert-path = ""
  key-path = ""

# TiCDCとetcdサービス間のセッション期間（秒単位）。このパラメータはオプションで、デフォルト値は10です。
capture-session-ttl = 10 # 10s

# TiCDCクラスターのOwnerモジュールがレプリケーション進行状況をプッシュしようとする間隔。このパラメータはオプションで、デフォルト値は`50000000`ナノ秒（つまり50ミリ秒）です。このパラメータは2つの方法で構成できます：数値のみを指定する（たとえば、`40000000`として構成すると、40000000ナノ秒、つまり40ミリ秒を表します）、または数値と単位の両方を指定する（たとえば、`40ms`として直接構成すると、40ミリ秒を表します）。
owner-flush-interval = 50000000 # 50 ms

# TiCDCクラスターのProcessorモジュールがレプリケーション進行状況をプッシュしようとする間隔。このパラメータはオプションで、デフォルト値は`50000000`ナノ秒（つまり50ミリ秒）です。このパラメータの構成方法は`owner-flush-interval`と同じです。
processor-flush-interval = 50000000 # 50 ms

# [log]
# # zapログモジュールの内部エラーログの出力先。このパラメータはオプションで、デフォルト値は"stderr"です。
#   error-output = "stderr"
#   [log.file]
#     # 単一のログファイルの最大サイズ（MiB単位）。このパラメータはオプションで、デフォルト値は300です。
#     max-size = 300 # 300 MiB
#     # ログファイルを保持する最大日数。このパラメータはオプションで、デフォルト値は`0`で、削除しないことを示します。
#     max-days = 0
#     # 保持するログファイルの数。このパラメータはオプションで、デフォルト値は`0`で、すべてのログファイルを保持することを示します。
#     max-backups = 0

#[sorter]
# デフォルトで開始される8つのpebble DBのSorterモジュールの共有pebbleブロックキャッシュのサイズ（MiB単位）。デフォルト値は128です。
# cache-size-in-mb = 128
# ソーターファイルが保存されるディレクトリ（`data-dir`に対する相対的なディレクトリ）。このパラメータはオプションで、デフォルト値は"/tmp/sorter"です。
# sorter-dir = "/tmp/sorter"

# [kv-client]
#   単一のRegionワーカーで使用できるスレッド数。このパラメータはオプションで、デフォルト値は8です。
#   worker-concurrent = 8
#   TiCDCの共有スレッドプールで使用できるスレッド数。主にKVイベントの処理に使用されます。このパラメータはオプションで、デフォルト値は0で、デフォルトのプールサイズはCPUコア数の2倍です。
#   worker-pool-size = 0
#   Region接続のリトライ期間。このパラメータはオプションで、デフォルト値は`60000000000`ナノ秒（つまり1分）です。このパラメータは2つの方法で構成できます：数値のみを指定する（たとえば、`50000000`として構成すると、50000000ナノ秒、つまり50ミリ秒を表します）、または数値と単位の両方を指定する（たとえば、`50ms`として直接構成すると、50ミリ秒を表します）。
#   region-retry-duration = 60000000000
```
