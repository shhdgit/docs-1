---
title: TiCDC Server Configurations
summary: Learn the CLI and configuration parameters used in TiCDC.
---

# TiCDCサーバーの設定 {#ticdc-server-configurations}

このドキュメントでは、TiCDCで使用されるCLIおよび設定ファイルパラメーターについて説明します。

## `cdc server` CLIパラメーター {#cdc-server-cli-parameters}

以下は、`cdc server`コマンドで使用可能なオプションの説明です：

- `addr`：TiCDCのリスニングアドレス、HTTP APIアドレス、およびPrometheusアドレス。デフォルト値は`127.0.0.1:8300`です。
- `advertise-addr`：クライアントがTiCDCにアクセスするためのアドバタイズドアドレス。指定しない場合、値は`addr`と同じです。
- `pd`：カンマ区切りのPDエンドポイントのリスト。
- `config`：TiCDCが使用する設定ファイルのアドレス（オプション）。このオプションは、TiCDC v5.0.0以降でサポートされています。このオプションは、TiUP v1.4.0以降で使用できます。詳細な設定の説明については、[TiCDC Changefeed Configurations](/ticdc/ticdc-changefeed-config.md)を参照してください。
- `data-dir`：TiCDCがファイルを保存するためにディスクを使用する必要がある場合に使用するディレクトリを指定します。TiCDCで使用されるソートエンジンとリドログは、一時ファイルを保存するためにこのディレクトリを使用します。このディレクトリの空きディスク容量が500 GiB以上であることを推奨します。TiUPを使用している場合は、[`cdc_servers`](/tiup/tiup-cluster-topology-reference.md#cdc_servers)セクションで`data_dir`を設定するか、`global`でデフォルトの`data_dir`パスを直接使用できます。
- `gc-ttl`：TiCDCによって設定されたサービスレベルの`GC safepoint`のTTL（Time To Live）およびレプリケーションタスクが一時停止できる期間（秒単位）。デフォルト値は`86400`で、24時間を意味します。注意：TiCDCレプリケーションタスクの一時停止は、TiCDC GC safepointの進行状況に影響を与え、上流のTiDB GCの進行状況に影響を与えることに注意してください。詳細については、[Complete Behavior of TiCDC GC safepoint](/ticdc/ticdc-faq.md#what-is-the-complete-behavior-of-ticdc-garbage-collection-gc-safepoint)を参照してください。
- `log-file`：TiCDCプロセスが実行されているときにログが出力されるパス。このパラメーターが指定されていない場合、ログは標準出力（stdout）に書き込まれます。
- `log-level`：TiCDCプロセスが実行されているときのログレベル。デフォルト値は`"info"`です。
- `ca`：TLS接続用のCA証明書ファイルのパス（オプション）を指定します。
- `cert`：TLS接続用の証明書ファイルのパス（オプション）を指定します。
- `cert-allowed-cn`：TLS接続用の共通名のパス（オプション）を指定します。
- `key`：TLS接続用の秘密鍵ファイルのパス（オプション）を指定します。
- `tz`：TiCDCサービスで使用されるタイムゾーン。TiCDCは、内部的に`TIMESTAMP`などの時刻データ型を変換するときや、下流にデータをレプリケートするときにこのタイムゾーンを使用します。デフォルトは、プロセスが実行されるローカルタイムゾーンです。`time-zone`（`sink-uri`内）と`tz`を同時に指定した場合、内部のTiCDCプロセスは`tz`で指定されたタイムゾーンを使用し、シンクは下流にデータをレプリケートするために`time-zone`で指定されたタイムゾーンを使用します。`tz`で指定されたタイムゾーンが`time-zone`（`sink-uri`内）で指定されたタイムゾーンと同じであることを確認してください。
- `cluster-id`：（オプション）TiCDCクラスターのID。デフォルト値は`default`です。`cluster-id`は、TiCDCクラスターの一意の識別子です。同じ`cluster-id`を持つTiCDCノードは同じクラスターに属します。`cluster-id`の長さは、最大で128文字です。`cluster-id`は`^[a-zA-Z0-9]+(-[a-zA-Z0-9]+)*$`のパターンに従う必要があり、次のいずれかであってはなりません：`owner`、`capture`、`task`、`changefeed`、`job`、`meta`。

## `cdc server`設定ファイルパラメーター {#cdc-server-configuration-file-parameters}

以下は、`cdc server`コマンドで`config`オプションで指定された設定ファイルの説明です：

```toml
# The configuration method of the following parameters is the same as that of CLI parameters, but the CLI parameters have higher priorities.
addr = "127.0.0.1:8300"
advertise-addr = ""
log-file = ""
log-level = "info"
data-dir = ""
gc-ttl = 86400 # 24 h
tz = "System"
cluster-id = "default"
# This parameter specifies the maximum memory threshold (in bytes) for tuning GOGC. Setting a smaller threshold increases the GC frequency. Setting a larger threshold reduces GC frequency and consumes more memory resources for the TiCDC process. Once the memory usage exceeds this threshold, GOGC Tuner stops working. The default value is 0, indicating that GOGC Tuner is disabled.
gc-tuner-memory-threshold = 0

[security]
  ca-path = ""
  cert-path = ""
  key-path = ""

# The session duration between TiCDC and etcd services, measured in seconds. This parameter is optional and its default value is 10.
capture-session-ttl = 10 # 10s

# The interval at which the Owner module in the TiCDC cluster attempts to push the replication progress. This parameter is optional and its default value is `50000000` nanoseconds (that is, 50 milliseconds). You can configure this parameter in two ways: specifying only the number (for example, configuring it as `40000000` represents 40000000 nanoseconds, which is 40 milliseconds), or specifying both the number and unit (for example, directly configuring it as `40ms`).
owner-flush-interval = 50000000 # 50 ms

# The interval at which the Processor module in the TiCDC cluster attempts to push the replication progress. This parameter is optional and its default value is `50000000` nanoseconds (that is, 50 milliseconds). The configuration method of this parameter is the same as that of `owner-flush-interval`.
processor-flush-interval = 50000000 # 50 ms

# [log]
# # The output location for internal error logs of the zap log module. This parameter is optional and its default value is "stderr".
#   error-output = "stderr"
#   [log.file]
#     # The maximum size of a single log file, measured in MiB. This parameter is optional and its default value is 300.
#     max-size = 300 # 300 MiB
#     # The maximum number of days to retain log files. This parameter is optional and its default value is `0`, indicating never to delete.
#     max-days = 0
#     # The number of log files to retain. This parameter is optional and its default value is `0`, indicating to keep all log files.
#     max-backups = 0

#[sorter]
# The size of the shared pebble block cache in the Sorter module for the 8 pebble DBs started by default, measured in MiB. The default value is 128.
# cache-size-in-mb = 128
# The directory where sorter files are stored relative to the data directory (`data-dir`). This parameter is optional and its default value is "/tmp/sorter".
# sorter-dir = "/tmp/sorter"

# [kv-client]
#   The number of threads that can be used in a single Region worker. This parameter is optional and its default value is 8.
#   worker-concurrent = 8
#   The number of threads in the shared thread pool of TiCDC, mainly used for processing KV events. This parameter is optional and its default value is 0, indicating that the default pool size is twice the number of CPU cores.
#   worker-pool-size = 0
#   The retry duration of Region connections. This parameter is optional and its default value is `60000000000` nanoseconds (that is, 1 minute). You can configure this parameter in two ways: specifying only the number (for example, configuring it as `50000000` represents 50000000 nanoseconds, which is 50 milliseconds), or specifying both the number and unit (for example, directly configuring it as `50ms`).
#   region-retry-duration = 60000000000
```
