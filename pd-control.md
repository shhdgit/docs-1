---
title: PD Control User Guide
summary: Use PD Control to obtain the state information of a cluster and tune a cluster.
---

# PD Controlユーザーガイド {#pd-control-user-guide}

PDのコマンドラインツールとして、PD Controlはクラスターの状態情報を取得し、クラスターを調整します。

## PD Controlのインストール {#install-pd-control}

> **Note:**
>
> 使用するControlツールのバージョンは、クラスターのバージョンと一致していることが推奨されます。

### TiUPコマンドの使用 {#use-tiup-command}

PD Controlを使用するには、`tiup ctl:v<CLUSTER_VERSION> pd -u http://<pd_ip>:<pd_port> [-i]`コマンドを実行します。

### インストールパッケージのダウンロード {#download-the-installation-package}

最新バージョンの`pd-ctl`を取得するには、TiDBサーバーのインストールパッケージをダウンロードします。`pd-ctl`は`ctl-{version}-linux-{arch}.tar.gz`パッケージに含まれています。

| インストールパッケージ                                                                                | OS    | アーキテクチャ | SHA256チェックサム                                                                             |
| :----------------------------------------------------------------------------------------- | :---- | :------ | :--------------------------------------------------------------------------------------- |
| `https://download.pingcap.org/tidb-community-server-{version}-linux-amd64.tar.gz` (pd-ctl) | Linux | amd64   | `https://download.pingcap.org/tidb-community-server-{version}-linux-amd64.tar.gz.sha256` |
| `https://download.pingcap.org/tidb-community-server-{version}-linux-arm64.tar.gz` (pd-ctl) | Linux | arm64   | `https://download.pingcap.org/tidb-community-server-{version}-linux-arm64.tar.gz.sha256` |

> **Note:**
>
> リンク内の`{version}`は、TiDBのバージョン番号を示しています。例えば、`amd64`アーキテクチャの`v7.1.3`のダウンロードリンクは、`https://download.pingcap.org/tidb-community-server-v7.1.3-linux-amd64.tar.gz`です。

### ソースコードからのコンパイル {#compile-from-source-code}

1. [Go](https://golang.org/) 1.20以降が必要です。Goモジュールが使用されているためです。
2. [PDプロジェクト](https://github.com/pingcap/pd)のルートディレクトリで、`make`または`make pd-ctl`コマンドを使用して、`bin/pd-ctl`をコンパイルおよび生成します。

## 使用法 {#usage}

シングルコマンドモード：

```bash
tiup ctl:v<CLUSTER_VERSION> pd store -u http://127.0.0.1:2379
```

{ "input\_content": "インタラクティブモード:" }

```bash
tiup ctl:v<CLUSTER_VERSION> pd -i -u http://127.0.0.1:2379
```

{ "input\_content": "環境変数を使用します:" }

```bash
export PD_ADDR=http://127.0.0.1:2379
tiup ctl:v<CLUSTER_VERSION> pd
```

{ "input\_content": "TLSを使用して暗号化する:" }

```bash
tiup ctl:v<CLUSTER_VERSION> pd -u https://127.0.0.1:2379 --cacert="path/to/ca" --cert="path/to/cert" --key="path/to/key"
```

## コマンドラインフラグ {#command-line-flags}

### `--cacert` {#cacert}

- PEM形式の信頼できるCAの証明書ファイルへのパスを指定します
- デフォルト: ""

### `--cert` {#cert}

- PEM形式のSSL証明書へのパスを指定します
- デフォルト: ""

### `--detach` / `-d` {#detach-d}

- シングルコマンドラインモードを使用します（readlineに入力しない）
- デフォルト: true

### `--help` / `-h` {#help-h}

- ヘルプ情報を出力します
- デフォルト: false

### `--interact` / `-i` {#interact-i}

- インタラクティブモードを使用します（readlineに入力します）
- デフォルト: false

### `--key` {#key}

- `--cert`で指定された証明書のプライベートキーである、SSLの証明書キーファイルへのパスを指定します
- デフォルト: ""

### `--pd` / `-u` {#pd-u}

- PDアドレスを指定します
- デフォルトアドレス: `http://127.0.0.1:2379`
- 環境変数: `PD_ADDR`

### `--version` / `-V` {#version-v}

- バージョン情報を出力して終了します
- デフォルト: false

## コマンド {#command}

### `cluster` {#cluster}

このコマンドを使用して、クラスターの基本情報を表示します。

使用法:

```bash
>> cluster                                     // To show the cluster information
{
  "id": 6493707687106161130,
  "max_peer_count": 3
}
```

### `config [show | set <option> <value> | placement-rules]` {#config-show-set-option-value-placement-rules}

このコマンドを使用して、構成情報を表示または変更します。

使用法:

```bash
>> config show                                // Display the config information of the scheduling
{
  "replication": {
    "enable-placement-rules": "true",
    "isolation-level": "",
    "location-labels": "",
    "max-replicas": 3,
    "strictly-match-label": "false"
  },
  "schedule": {
    "enable-cross-table-merge": "true",
    "high-space-ratio": 0.7,
    "hot-region-cache-hits-threshold": 3,
    "hot-region-schedule-limit": 4,
    "leader-schedule-limit": 4,
    "leader-schedule-policy": "count",
    "low-space-ratio": 0.8,
    "max-merge-region-keys": 200000,
    "max-merge-region-size": 20,
    "max-pending-peer-count": 64,
    "max-snapshot-count": 64,
    "max-store-down-time": "30m0s",
    "merge-schedule-limit": 8,
    "patrol-region-interval": "10ms",
    "region-schedule-limit": 2048,
    "region-score-formula-version": "v2",
    "replica-schedule-limit": 64,
    "scheduler-max-waiting-operator": 5,
    "split-merge-interval": "1h0m0s",
    "tolerant-size-ratio": 0
  }
}
>> config show all                            // Display all config information
>> config show replication                    // Display the config information of replication
{
  "max-replicas": 3,
  "location-labels": "",
  "isolation-level": "",
  "strictly-match-label": "false",
  "enable-placement-rules": "true"
}

>> config show cluster-version                // Display the current version of the cluster, which is the current minimum version of TiKV nodes in the cluster and does not correspond to the binary version.
"5.2.2"
```

- `max-snapshot-count` controls the maximum number of snapshots that a single store receives or sends out at the same time. The scheduler is restricted by this configuration to avoid taking up normal application resources. When you need to improve the speed of adding replicas or balancing, increase this value.

  ```bash
  config set max-snapshot-count 64  // Set the maximum number of snapshots to 64
  ```

- `max-pending-peer-count` controls the maximum number of pending peers in a single store. The scheduler is restricted by this configuration to avoid producing a large number of Regions without the latest log in some nodes. When you need to improve the speed of adding replicas or balancing, increase this value. Setting it to 0 indicates no limit.

  ```bash
  config set max-pending-peer-count 64  // Set the maximum number of pending peers to 64
  ```

- `max-merge-region-size` controls the upper limit on the size of Region Merge (the unit is MiB). When `regionSize` exceeds the specified value, PD does not merge it with the adjacent Region. Setting it to 0 indicates disabling Region Merge.

  ```bash
  config set max-merge-region-size 16 // Set the upper limit on the size of Region Merge to 16 MiB
  ```

- `max-merge-region-keys` controls the upper limit on the key count of Region Merge. When `regionKeyCount` exceeds the specified value, PD does not merge it with the adjacent Region.

  ```bash
  config set max-merge-region-keys 50000 // Set the upper limit on keyCount to 50000
  ```

- `split-merge-interval` controls the interval between the `split` and `merge` operations on a same Region. This means the newly split Region won't be merged within a period of time.

  ```bash
  config set split-merge-interval 24h  // Set the interval between `split` and `merge` to one day
  ```

- `enable-one-way-merge` controls whether PD only allows a Region to merge with the next Region. When you set it to `false`, PD allows a Region to merge with the adjacent two Regions.

  ```bash
  config set enable-one-way-merge true  // Enables one-way merging.
  ```

- `enable-cross-table-merge` is used to enable the merging of cross-table Regions. When you set it to `false`, PD does not merge the Regions from different tables. This option only works when key type is "table".

  ```bash
  config set enable-cross-table-merge true  // Enable cross table merge.
  ```

- `key-type` specifies the key encoding type used for the cluster. The supported options are \["table", "raw", "txn"], and the default value is "table".

  - If no TiDB instance exists in the cluster, `key-type` will be "raw" or "txn", and PD is allowed to merge Regions across tables regardless of the `enable-cross-table-merge` setting.
  - If any TiDB instance exists in the cluster, `key-type` should be "table". Whether PD can merge Regions across tables is determined by `enable-cross-table-merge`. If `key-type` is "raw", placement rules do not work.

  ```bash
  config set key-type raw  // Enable cross table merge.
  ```

- `region-score-formula-version` controls the version of the Region score formula. The value options are `v1` and `v2`. The version 2 of the formula helps to reduce redundant balance Region scheduling in some scenarios, such as taking TiKV nodes online or offline.

  ```bash
  config set region-score-formula-version v2
  ```

- `patrol-region-interval` controls the execution frequency that `replicaChecker` checks the health status of Regions. A shorter interval indicates a higher execution frequency. Generally, you do not need to adjust it.

  ```bash
  config set patrol-region-interval 10ms // Set the execution frequency of replicaChecker to 10ms
  ```

- `max-store-down-time` controls the time that PD decides the disconnected store cannot be restored if exceeded. If PD does not receive heartbeats from a store within the specified period of time, PD adds replicas in other nodes.

  ```bash
  config set max-store-down-time 30m  // Set the time within which PD receives no heartbeats and after which PD starts to add replicas to 30 minutes
  ```

- `max-store-preparing-time` controls the maximum waiting time for the store to go online. During the online stage of a store, PD can query the online progress of the store. When the specified time is exceeded, PD assumes that the store has been online and cannot query the online progress of the store again. But this does not prevent Regions from transferring to the new online store. In most scenarios, you do not need to adjust this parameter.

  The following command specifies that the maximum waiting time for the store to go online is 4 hours.

  ```bash
  config set max-store-preparing-time 4h
  ```

- `leader-schedule-limit` controls the number of tasks scheduling the leader at the same time. This value affects the speed of leader balance. A larger value means a higher speed and setting the value to 0 closes the scheduling. Usually the leader scheduling has a small load, and you can increase the value in need.

  ```bash
  config set leader-schedule-limit 4         // 4 tasks of leader scheduling at the same time at most
  ```

- `region-schedule-limit` controls the number of tasks of scheduling Regions at the same time. This value avoids too many Region balance operators being created. The default value is `2048` which is enough for all sizes of clusters, and setting the value to `0` closes the scheduling. Usually, the Region scheduling speed is limited by `store-limit`, but it is recommended that you do not customize this value unless you know exactly what you are doing.

  ```bash
  config set region-schedule-limit 2         // 2 tasks of Region scheduling at the same time at most
  ```

- `replica-schedule-limit` controls the number of tasks scheduling the replica at the same time. This value affects the scheduling speed when the node is down or removed. A larger value means a higher speed and setting the value to 0 closes the scheduling. Usually the replica scheduling has a large load, so do not set a too large value. Note that this configuration item is usually kept at the default value. If you want to change the value, you need to try a few values to see which one works best according to the real situation.

  ```bash
  config set replica-schedule-limit 4        // 4 tasks of replica scheduling at the same time at most
  ```

- `merge-schedule-limit` controls the number of Region Merge scheduling tasks. Setting the value to 0 closes Region Merge. Usually the Merge scheduling has a large load, so do not set a too large value. Note that this configuration item is usually kept at the default value. If you want to change the value, you need to try a few values to see which one works best according to the real situation.

  ```bash
  config set merge-schedule-limit 16       // 16 tasks of Merge scheduling at the same time at most
  ```

- `hot-region-schedule-limit` controls the hot Region scheduling tasks that are running at the same time. Setting its value to `0` means disabling the scheduling. It is not recommended to set a too large value. Otherwise, it might affect the system performance. Note that this configuration item is usually kept at the default value. If you want to change the value, you need to try a few values to see which one works best according to the real situation.

  ```bash
  config set hot-region-schedule-limit 4       // 4 tasks of hot Region scheduling at the same time at most
  ```

- `hot-region-cache-hits-threshold` is used to set the number of minutes required to identify a hot Region. PD can participate in the hotspot scheduling only after the Region is in the hotspot state for more than this number of minutes.

- `tolerant-size-ratio` controls the size of the balance buffer area. When the score difference between the leader or Region of the two stores is less than specified multiple times of the Region size, it is considered in balance by PD.

  ```bash
  config set tolerant-size-ratio 20        // Set the size of the buffer area to about 20 times of the average Region Size
  ```

- `low-space-ratio` controls the threshold value that is considered as insufficient store space. When the ratio of the space occupied by the node exceeds the specified value, PD tries to avoid migrating data to the corresponding node as much as possible. At the same time, PD mainly schedules the remaining space to avoid using up the disk space of the corresponding node.

  ```bash
  config set low-space-ratio 0.9              // Set the threshold value of insufficient space to 0.9
  ```

- `high-space-ratio` controls the threshold value that is considered as sufficient store space. This configuration takes effect only when `region-score-formula-version` is set to `v1`. When the ratio of the space occupied by the node is less than the specified value, PD ignores the remaining space and mainly schedules the actual data volume.

  ```bash
  config set high-space-ratio 0.5             // Set the threshold value of sufficient space to 0.5
  ```

- `cluster-version` is the version of the cluster, which is used to enable or disable some features and to deal with the compatibility issues. By default, it is the minimum version of all normally running TiKV nodes in the cluster. You can set it manually only when you need to roll it back to an earlier version.

  ```bash
  config set cluster-version 1.0.8              // Set the version of the cluster to 1.0.8
  ```

- `replication-mode` controls the replication mode of Regions in the dual data center scenario. See [Enable the DR Auto-Sync mode](/two-data-centers-in-one-city-deployment.md#enable-the-dr-auto-sync-mode) for details.

- `leader-schedule-policy` is used to select the scheduling strategy for the leader. You can schedule the leader according to `size` or `count`.

- `scheduler-max-waiting-operator` is used to control the number of waiting operators in each scheduler.

- `enable-remove-down-replica` is used to enable the feature of automatically deleting DownReplica. When you set it to `false`, PD does not automatically clean up the downtime replicas.

- `enable-replace-offline-replica` is used to enable the feature of migrating OfflineReplica. When you set it to `false`, PD does not migrate the offline replicas.

- `enable-make-up-replica` is used to enable the feature of making up replicas. When you set it to `false`, PD does not add replicas for Regions without sufficient replicas.

- `enable-remove-extra-replica` is used to enable the feature of removing extra replicas. When you set it to `false`, PD does not remove extra replicas for Regions with redundant replicas.

- `enable-location-replacement` is used to enable the isolation level checking. When you set it to `false`, PD does not increase the isolation level of a Region replica through scheduling.

- `enable-debug-metrics` is used to enable the metrics for debugging. When you set it to `true`, PD enables some metrics such as `balance-tolerant-size`.

- `enable-placement-rules` is used to enable placement rules, which is enabled by default in v5.0 and later versions.

- `store-limit-mode` is used to control the mode of limiting the store speed. The optional modes are `auto` and `manual`. In `auto` mode, the stores are automatically balanced according to the load (deprecated).

- `store-limit-version` controls the version of the store limit formula. In v1 mode, you can manually modify the `store limit` to limit the scheduling speed of a single TiKV. The v2 mode is an experimental feature. In v2 mode, you do not need to manually set the `store limit` value, as PD dynamically adjusts it based on the capability of TiKV snapshots. For more details, refer to [Principles of store limit v2](/configure-store-limit.md#principles-of-store-limit-v2).

  ```bash
  config set store-limit-version v2       // using store limit v2
  ```

- PD rounds the lowest digits of the flow number, which reduces the update of statistics caused by the changes of the Region flow information. This configuration item is used to specify the number of lowest digits to round for the Region flow information. For example, the flow `100512` will be rounded to `101000` because the default value is `3`. This configuration replaces `trace-region-flow`.

- For example, set the value of `flow-round-by-digit` to `4`:

  ```bash
  config set flow-round-by-digit 4
  ```

#### `config placement-rules [disable | enable | load | save | show | rule-group]` {#config-placement-rules-disable-enable-load-save-show-rule-group}

`config placement-rules [disable | enable | load | save | show | rule-group]`の使用法については、[配置ルールの構成](/configure-placement-rules.md#configure-rules)を参照してください。

### `health` {#health}

このコマンドを使用して、クラスターのヘルス情報を表示します。

使用法：

```bash
>> health                                // Display the health information
[
  {
    "name": "pd",
    "member_id": 13195394291058371180,
    "client_urls": [
      "http://127.0.0.1:2379"
      ......
    ],
    "health": true
  }
  ......
]
```

### `hot [read | write | store|  history <start_time> <end_time> [<key> <value>]]` {#hot-read-write-store-history-start-time-end-time-key-value}

クラスターのホットスポット情報を表示するためにこのコマンドを使用します。

使用法：

```bash
>> hot read                                // Display hot spot for the read operation
>> hot write                               // Display hot spot for the write operation
>> hot store                               // Display hot spot for all the read and write operations
>> hot history 1629294000000 1631980800000 // Display history hot spot for the specified period (milliseconds). 1629294000000 is the start time and 1631980800000 is the end time.
{
  "history_hot_region": [
    {
      "update_time": 1630864801948,
      "region_id": 103,
      "peer_id": 1369002,
      "store_id": 3,
      "is_leader": true,
      "is_learner": false,
      "hot_region_type": "read",
      "hot_degree": 152,
      "flow_bytes": 0,
      "key_rate": 0,
      "query_rate": 305,
      "start_key": "7480000000000000FF5300000000000000F8",
      "end_key": "7480000000000000FF5600000000000000F8"
    },
    ...
  ]
}
>> hot history 1629294000000 1631980800000 hot_region_type read region_id 1,2,3 store_id 1,2,3 peer_id 1,2,3 is_leader true is_learner true // Display history hotspot for the specified period with more conditions
{
  "history_hot_region": [
    {
      "update_time": 1630864801948,
      "region_id": 103,
      "peer_id": 1369002,
      "store_id": 3,
      "is_leader": true,
      "is_learner": false,
      "hot_region_type": "read",
      "hot_degree": 152,
      "flow_bytes": 0,
      "key_rate": 0,
      "query_rate": 305,
      "start_key": "7480000000000000FF5300000000000000F8",
      "end_key": "7480000000000000FF5600000000000000F8"
    },
    ...
  ]
}
```

### `label [store <name> <value>]` {#label-store-name-value}

このコマンドを使用して、クラスターのラベル情報を表示します。

使用法：

```bash
>> label                                // Display all labels
>> label store zone cn                  // Display all stores including the "zone":"cn" label
```

### `member [delete | leader_priority | leader [show | resign | transfer <member_name>]]` {#member-delete-leader-priority-leader-show-resign-transfer-member-name}

このコマンドを使用して、PDメンバーを表示したり、指定されたメンバーを削除したり、リーダーの優先度を設定したりします。

使用法：

```bash
>> member                               // Display the information of all members
{
  "header": {......},
  "members": [......],
  "leader": {......},
  "etcd_leader": {......},
}
>> member delete name pd2               // Delete "pd2"
Success!
>> member delete id 1319539429105371180 // Delete a node using id
Success!
>> member leader show                   // Display the leader information
{
  "name": "pd",
  "member_id": 13155432540099656863,
  "peer_urls": [......],
  "client_urls": [......]
}
>> member leader resign // Move leader away from the current member
......
>> member leader transfer pd3 // Migrate leader to a specified member
......
```

### `operator [check | show | add | remove]` {#operator-check-show-add-remove}

このコマンドを使用して、スケジューリング操作を表示および制御します。

使用法：

```bash
>> operator show                                        // Display all operators
>> operator show admin                                  // Display all admin operators
>> operator show leader                                 // Display all leader operators
>> operator show region                                 // Display all Region operators
>> operator add add-peer 1 2                            // Add a replica of Region 1 on store 2
>> operator add add-learner 1 2                         // Add a learner replica of Region 1 on store 2
>> operator add remove-peer 1 2                         // Remove a replica of Region 1 on store 2
>> operator add transfer-leader 1 2                     // Schedule the leader of Region 1 to store 2
>> operator add transfer-region 1 2 3 4                 // Schedule Region 1 to stores 2,3,4
>> operator add transfer-peer 1 2 3                     // Schedule the replica of Region 1 on store 2 to store 3
>> operator add merge-region 1 2                        // Merge Region 1 with Region 2
>> operator add split-region 1 --policy=approximate     // Split Region 1 into two Regions in halves, based on approximately estimated value
>> operator add split-region 1 --policy=scan            // Split Region 1 into two Regions in halves, based on accurate scan value
>> operator remove 1                                    // Remove the scheduling operation of Region 1
>> operator check 1                                     // Check the status of the operators related to Region 1
```

The splitting of リージョン starts from the position as close as possible to the middle. You can locate this position using two strategies, namely "scan" and "approximate". The difference between them is that the former determines the middle key by scanning the リージョン, and the latter obtains the approximate position by checking the statistics recorded in the SST file. Generally, the former is more accurate, while the latter consumes less I/O and can be completed faster.

### `ping` {#ping}

Use this command to view the time that `ping` PD takes.

Usage:

```bash
>> ping
time: 43.12698ms
```

### `リージョン <region_id> [--jq="<query string>"]` {#region-region-id-jq-query-string}

このコマンドを使用して、リージョン情報を表示します。jq形式の出力については、[jq-formatted-json-output-usage](#jq-formatted-json-output-usage)を参照してください。

使用法：

```bash
>> region                               //　Display the information of all Regions
{
  "count": 1,
  "regions": [......]
}

>> region 2                             // Display the information of the Region with the ID of 2
{
  "id": 2,
  "start_key": "7480000000000000FF1D00000000000000F8",
  "end_key": "7480000000000000FF1F00000000000000F8",
  "epoch": {
    "conf_ver": 1,
    "version": 15
  },
  "peers": [
    {
      "id": 40,
      "store_id": 3
    }
  ],
  "leader": {
    "id": 40,
    "store_id": 3
  },
  "written_bytes": 0,
  "read_bytes": 0,
  "written_keys": 0,
  "read_keys": 0,
  "approximate_size": 1,
  "approximate_keys": 0
}
```

### `リージョンキー [--format=raw|encode|hex] <key>` {#region-key-format-raw-encode-hex-key}

このコマンドを使用して、特定のキーが存在するリージョンをクエリします。raw、encoding、およびhex形式をサポートしています。エンコード形式の場合は、キーの周りにシングルクォートを使用する必要があります。

16進数形式の使用法（デフォルト）：

```bash
>> region key 7480000000000000FF1300000000000000F8
{
  "region": {
    "id": 2,
    ......
  }
}
```

{ "input\_content": "Raw format usage:" }

{ "input\_content": "生の形式の使用方法:" }

```bash
>> region key --format=raw abc
{
  "region": {
    "id": 2,
    ......
  }
}
```

{"input\_content": "エンコーディング形式の使用方法:"}

```bash
>> region key --format=encode 't\200\000\000\000\000\000\000\377\035_r\200\000\000\000\000\377\017U\320\000\000\000\000\000\372'
{
  "region": {
    "id": 2,
    ......
  }
}
```

### `region scan` {#region-scan}

このコマンドを使用して、すべてのリージョンを取得します。

使用法：

```bash
>> region scan
{
  "count": 20,
  "regions": [......],
}
```

### `region sibling <region_id>` {#region-sibling-region-id}

このコマンドは、特定のリージョンの隣接するリージョンをチェックするために使用します。

使用法:

```bash
>> region sibling 2
{
  "count": 2,
  "regions": [......],
}
```

### `region keys [--format=raw|encode|hex] <start_key> <end_key> <limit>` {#region-keys-format-raw-encode-hex-start-key-end-key-limit}

このコマンドを使用して、指定された範囲内のすべてのリージョンをクエリします `[startkey、endkey)`。 `endKey` なしの範囲もサポートされています。

`limit` パラメータはキーの数を制限します。 `limit` のデフォルト値は `16` で、`-1` の値は無制限のキーを意味します。

使用法：

```bash
>> region keys --format=raw a         // Display all Regions that start from the key a with a default limit count of 16
{
  "count": 16,
  "regions": [......],
}

>> region keys --format=raw a z      // Display all Regions in the range [a, z) with a default limit count of 16
{
  "count": 16,
  "regions": [......],
}

>> region keys --format=raw a z -1   // Display all Regions in the range [a, z) without a limit count
{
  "count": ...,
  "regions": [......],
}

>> region keys --format=raw a "" 20   // Display all Regions that start from the key a with a limit count of 20
{
  "count": 20,
  "regions": [......],
}
```

### `リージョンストア <store_id>` {#region-store-store-id}

このコマンドを使用して、特定のストアのすべてのリージョンをリストします。

使用法：

```bash
>> region store 2
{
  "count": 10,
  "regions": [......],
}
```

### `リージョン topread [limit]` {#region-topread-limit}

このコマンドを使用して、トップリードフローを持つリージョンの一覧を表示します。limitのデフォルト値は16です。

使用法：

```bash
>> region topread
{
  "count": 16,
  "regions": [......],
}
```

### `region topwrite [limit]` {#region-topwrite-limit}

このコマンドは、トップライトフローを持つリージョンをリストするために使用します。 リミットのデフォルト値は16です。

使用法：

```bash
>> region topwrite
{
  "count": 16,
  "regions": [......],
}
```

### `region topconfver [limit]` {#region-topconfver-limit}

このコマンドを使用して、トップ構成バージョンを持つリージョンのリストを表示します。 リミットのデフォルト値は16です。

使用法：

```bash
>> region topconfver
{
  "count": 16,
  "regions": [......],
}
```

### `region topversion [limit]` {#region-topversion-limit}

このコマンドを使用して、トップバージョンのリージョンをリストします。limitのデフォルト値は16です。

使用法：

```bash
>> region topversion
{
  "count": 16,
  "regions": [......],
}
```

### `region topsize [limit]` {#region-topsize-limit}

このコマンドを使用して、トップの近似サイズを持つリージョンをリストします。制限のデフォルト値は16です。

使用法：

```bash
>> region topsize
{
  "count": 16,
  "regions": [......],
}

```

### `region check [miss-peer | extra-peer | down-peer | pending-peer | offline-peer | empty-region | hist-size | hist-keys] [--jq="<query string>"]` {#region-check-miss-peer-extra-peer-down-peer-pending-peer-offline-peer-empty-region-hist-size-hist-keys-jq-query-string}

このコマンドを使用して、異常な状態のリージョンをチェックします。jq形式の出力については、[jq形式のJSON出力の使用方法](#jq-formatted-json-output-usage)を参照してください。

さまざまなタイプの説明：

- miss-peer: 十分なレプリカがないリージョン
- extra-peer: 追加のレプリカがあるリージョン
- down-peer: いくつかのレプリカがダウンしているリージョン
- pending-peer: いくつかのレプリカが保留中のリージョン

使用法：

```bash
>> region check miss-peer
{
  "count": 2,
  "regions": [......],
}
```

### `scheduler [show | add | remove | pause | resume | config | describe]` {#scheduler-show-add-remove-pause-resume-config-describe}

このコマンドを使用して、スケジューリングポリシーを表示および制御します。

使用法：

```bash
>> scheduler show                                 // Display all created schedulers
>> scheduler add grant-leader-scheduler 1         // Schedule all the leaders of the Regions on store 1 to store 1
>> scheduler add evict-leader-scheduler 1         // Move all the Region leaders on store 1 out
>> scheduler config evict-leader-scheduler        // Display the stores in which the scheduler is located since v4.0.0
>> scheduler add shuffle-leader-scheduler         // Randomly exchange the leader on different stores
>> scheduler add shuffle-region-scheduler         // Randomly scheduling the Regions on different stores
>> scheduler add evict-slow-store-scheduler       // When there is one and only one slow store, evict all Region leaders of that store
>> scheduler remove grant-leader-scheduler-1      // Remove the corresponding scheduler, and `-1` corresponds to the store ID
>> scheduler pause balance-region-scheduler 10    // Pause the balance-region scheduler for 10 seconds
>> scheduler pause all 10                         // Pause all schedulers for 10 seconds
>> scheduler resume balance-region-scheduler      // Continue to run the balance-region scheduler
>> scheduler resume all                           // Continue to run all schedulers
>> scheduler config balance-hot-region-scheduler  // Display the configuration of the balance-hot-region scheduler
>> scheduler describe balance-region-scheduler    // Display the running state and related diagnostic information of the balance-region scheduler
```

### `scheduler describe balance-region-scheduler` {#scheduler-describe-balance-region-scheduler}

このコマンドを使用して、`balance-region-scheduler`の実行状態と関連する診断情報を表示します。

TiDB v6.3.0以降、PDは`balance-region-scheduler`と`balance-leader-scheduler`の実行状態と簡単な診断情報を提供します。他のスケジューラやチェッカーはまだサポートされていません。この機能を有効にするには、`pd-ctl`を使用して[`enable-diagnostic`](/pd-configuration-file.md#enable-diagnostic-new-in-v630)構成項目を変更できます。

スケジューラの状態は次のいずれかです：

- `disabled`：スケジューラは利用できないか削除されています。
- `paused`：スケジューラは一時停止されています。
- `scheduling`：スケジューラはスケジューリング演算子を生成しています。
- `pending`：スケジューラはスケジューリング演算子を生成できません。`pending`状態のスケジューラについては、簡単な診断情報が返されます。簡単な情報はストアの状態を説明し、なぜこれらのストアをスケジューリングの対象に選択できないかを説明します。
- `normal`：スケジューリング演算子を生成する必要がありません。

### `scheduler config balance-leader-scheduler` {#scheduler-config-balance-leader-scheduler}

このコマンドを使用して、`balance-leader-scheduler`ポリシーを表示および制御します。

TiDB v6.0.0以降、PDは`balance-leader-scheduler`の速度を制御するための`Batch`パラメータを導入しました。このパラメータを使用するには、`pd-ctl`を使用して`balance-leader batch`構成項目を変更できます。

v6.0.0以前では、PDにはこの構成項目がないため、`balance-leader batch=1`となります。v6.0.0以降のバージョンでは、`balance-leader batch`のデフォルト値は`4`です。この構成項目を`4`より大きな値に設定するには、同時に[`scheduler-max-waiting-operator`](#config-show--set-option-value--placement-rules)（デフォルト値は`5`）により大きな値を設定する必要があります。両方の構成項目を変更した後に、期待される加速効果を得ることができます。"

```bash
scheduler config balance-leader-scheduler set batch 3 // Set the size of the operator that the balance-leader scheduler can execute in a batch to 3
```

#### `scheduler config balance-hot-region-scheduler` {#scheduler-config-balance-hot-region-scheduler}

このコマンドを使用して、`balance-hot-region-scheduler` ポリシーを表示および制御します。

使用法:

```bash
>> scheduler config balance-hot-region-scheduler  // Display all configuration of the balance-hot-region scheduler
{
  "min-hot-byte-rate": 100,
  "min-hot-key-rate": 10,
  "min-hot-query-rate": 10,
  "max-zombie-rounds": 3,
  "max-peer-number": 1000,
  "byte-rate-rank-step-ratio": 0.05,
  "key-rate-rank-step-ratio": 0.05,
  "query-rate-rank-step-ratio": 0.05,
  "count-rank-step-ratio": 0.01,
  "great-dec-ratio": 0.95,
  "minor-dec-ratio": 0.99,
  "src-tolerance-ratio": 1.05,
  "dst-tolerance-ratio": 1.05,
  "read-priorities": [
    "query",
    "byte"
  ],
  "write-leader-priorities": [
    "key",
    "byte"
  ],
  "write-peer-priorities": [
    "byte",
    "key"
  ],
  "strict-picking-store": "true",
  "enable-for-tiflash": "true",
  "rank-formula-version": "v2"
}
```

- `min-hot-byte-rate` は通常 100 である、カウントされるバイトの最小数を意味します。

  ```bash
  scheduler config balance-hot-region-scheduler set min-hot-byte-rate 100
  ```

- `min-hot-key-rate` は通常 10 である、カウントされるキーの最小数を意味します。

  ```bash
  scheduler config balance-hot-region-scheduler set min-hot-key-rate 10
  ```

- `min-hot-query-rate` は通常 10 である、カウントされるクエリの最小数を意味します。

  ```bash
  scheduler config balance-hot-region-scheduler set min-hot-query-rate 10
  ```

- `max-zombie-rounds` は、オペレータが保留中の影響と見なされるハートビートの最大数を意味します。この値を大きく設定すると、保留中の影響により多くのオペレータが含まれる可能性があります。通常、この値を調整する必要はありません。保留中の影響とは、スケジューリング中に生成されるがまだ効果を持つオペレータの影響を指します。

  ```bash
  scheduler config balance-hot-region-scheduler set max-zombie-rounds 3
  ```

- `max-peer-number` は解決される最大のピア数を意味し、スケジューラが遅すぎるのを防ぎます。

  ```bash
  scheduler config balance-hot-region-scheduler set max-peer-number 1000
  ```

- `byte-rate-rank-step-ratio`、`key-rate-rank-step-ratio`、`query-rate-rank-step-ratio`、および `count-rank-step-ratio` は、それぞれバイト、キー、クエリ、およびカウントのステップランクを意味します。ランクステップ比率は、ランクが計算される際のステップを決定します。`great-dec-ratio` と `minor-dec-ratio` は `dec` ランクを決定するために使用されます。通常、これらの項目を変更する必要はありません。

  ```bash
  scheduler config balance-hot-region-scheduler set byte-rate-rank-step-ratio 0.05
  ```

- `src-tolerance-ratio` と `dst-tolerance-ratio` は、期待スケジューラの構成項目です。`tolerance-ratio` が小さいほど、スケジューリングが容易になります。冗長なスケジューリングが発生した場合、この値を適切に増やすことができます。

  ```bash
  scheduler config balance-hot-region-scheduler set src-tolerance-ratio 1.1
  ```

- `read-priorities`、`write-leader-priorities`、および `write-peer-priorities` は、スケジューラがホットリージョンのスケジューリングに優先する次元を制御します。構成には 2 つの次元がサポートされています。

  - `read-priorities` と `write-leader-priorities` は、スケジューラが読み取りおよび書き込みリーダータイプのホットリージョンのスケジューリングに優先する次元を制御します。次元のオプションは `query`、`byte`、および `key` です。

  - `write-peer-priorities` は、スケジューラが書き込みピアタイプのホットリージョンのスケジューリングに優先する次元を制御します。次元のオプションは `byte` と `key` です。

  > **Note:**
  >
  > クラスタコンポーネントが v5.2 より前の場合、`query` 次元の構成は効果がありません。一部のコンポーネントが v5.2 以降にアップグレードされると、`byte` および `key` 次元は引き続きデフォルトでホットリージョンのスケジューリングに優先されます。クラスタのすべてのコンポーネントが v5.2 以降にアップグレードされると、このような構成は引き続き互換性のために有効です。リアルタイムの構成は `pd-ctl` コマンドを使用して表示できます。通常、これらの構成を変更する必要はありません。

  ```bash
  scheduler config balance-hot-region-scheduler set read-priorities query,byte
  ```

- `strict-picking-store` は、ホットリージョンのスケジューリングの検索スペースを制御します。通常、有効になっています。この構成項目は、`rank-formula-version` が `v1` の場合にのみ動作します。有効になっている場合、ホットリージョンのスケジューリングは、2 つの構成された次元でのホットリージョンのバランスを確保します。無効になっている場合、ホットリージョンのスケジューリングは、最初の優先度の次元でのバランスのみを確保し、他の次元でのバランスが低下する可能性があります。通常、この構成を変更する必要はありません。

  ```bash
  scheduler config balance-hot-region-scheduler set strict-picking-store true
  ```

- `rank-formula-version` は、ホットリージョンのスケジューリングで使用されるスケジューラアルゴリズムバージョンを制御します。値のオプションは `v1` および `v2` です。デフォルト値は `v2` です。

  - `v1` アルゴリズムは、TiDB v6.3.0 およびそれ以前のバージョンで使用されるスケジューラ戦略です。このアルゴリズムは、主にストア間の負荷差を減らし、他の次元での副作用を導入しないようにします。
  - `v2` アルゴリズムは、TiDB v6.3.0 で導入された実験的なスケジューラ戦略であり、TiDB v6.4.0 で一般提供 (GA) されています。このアルゴリズムは、主にストア間の均等性の割合を向上させ、わずかな副作用を考慮に入れます。`strict-picking-store` が `true` の `v1` アルゴリズムと比較すると、`v2` アルゴリズムは最初の次元の優先度の均等化により多くの注意を払います。`strict-picking-store` が `false` の `v1` アルゴリズムと比較すると、`v2` アルゴリズムは第二次元のバランスを考慮します。
  - `strict-picking-store` が `true` の `v1` アルゴリズムは保守的で、両方の次元で負荷が高いストアがある場合にのみスケジューリングが生成されます。特定のシナリオでは、次元の競合のためにバランスを続けることが不可能になる場合があります。最初の次元でのバランスをより良くするためには、`strict-picking-store` を `false` に設定する必要があります。`v2` アルゴリズムは、両方の次元でのバランスをより良くし、無効なスケジューリングを減らすことができます。

  ```bash
  scheduler config balance-hot-region-scheduler set rank-formula-version v2
  ```

- `enable-for-tiflash` は、TiFlash インスタンスに対してホットリージョンのスケジューリングが有効かどうかを制御します。通常、有効になっています。無効になっている場合、TiFlash インスタンス間のホットリージョンのスケジューリングは行われません。

  ```bash
  scheduler config balance-hot-region-scheduler set enable-for-tiflash true
  ```

### `service-gc-safepoint` {#service-gc-safepoint}

このコマンドを使用して、現在の GC セーフポイントとサービス GC セーフポイントをクエリします。出力は次のようになります:

```bash
{
  "service_gc_safe_points": [
    {
      "service_id": "gc_worker",
      "expired_at": 9223372036854775807,
      "safe_point": 439923410637160448
    }
  ],
  "gc_safe_point": 0
}
```

### `store [delete | cancel-delete | label | weight | remove-tombstone | limit ] <store_id> [--jq="<query string>"]` {#store-delete-cancel-delete-label-weight-remove-tombstone-limit-store-id-jq-query-string}

jqフォーマットの出力については、[jq-formatted-json-output-usage](#jq-formatted-json-output-usage)を参照してください。

#### ストアを取得する {#get-a-store}

すべてのストアの情報を表示するには、次のコマンドを実行します：

```bash
store
```

```
{
  "count": 3,
  "stores": [...]
}
```

idが1のストアを取得するには、次のコマンドを実行します：

```bash
store 1
```

```
......
```

#### ストアを削除する {#delete-a-store}

IDが1のストアを削除するには、次のコマンドを実行します：

```bash
store delete 1
```

`store delete`コマンドを使用して削除された`オフライン`状態のストアをキャンセルするには、`store cancel-delete`コマンドを実行します。キャンセルすると、ストアは`オフライン`から`アップ`に変更されます。`store cancel-delete`コマンドは、`墓石`状態のストアを`アップ`状態に変更することはできないことに注意してください。

IDが1のストアの削除をキャンセルするには、次のコマンドを実行します：

```bash
store cancel-delete 1
```

次のコマンドを実行して、`Tombstone` 状態のすべてのストアを削除します。

```bash
store remove-tombstone
```

> **Note:**
>
> もしPDリーダーがストアの削除中に変更された場合、[`store limit`](#configure-store-scheduling-speed)コマンドを使用してストアの制限を手動で変更する必要があります。

#### ストアラベルの管理 {#manage-store-labels}

ストアのラベルを管理するには、`store label`コマンドを実行します。

- キーが`"zone"`で値が`"cn"`であるラベルを、IDが1のストアに設定するには、次のコマンドを実行します：

  ```bash
  store label 1 zone=cn
  ```

- 例えば、IDが1のストアのラベルを更新するには、キー`"zone"`の値を`"cn"`から`"us"`に変更するには、次のコマンドを実行します：

  ```bash
  store label 1 zone=us
  ```

- IDが1のストアのすべてのラベルを書き換えるには、`--rewrite`オプションを使用します。このオプションはすべての既存のラベルを上書きすることに注意してください：

  ```bash
  store label 1 region=us-est-1 disk=ssd --rewrite
  ```

- IDが1のストアの`"disk"`ラベルを削除するには、`--delete`オプションを使用します：

  ```bash
  store label 1 disk --delete
  ```

> **Note:**
>
> - ストアのラベルは、TiKV内のラベルとPD内のラベルをマージして更新されます。具体的には、TiKV構成ファイルでストアのラベルを変更してクラスターを再起動した後、PDは独自のストアラベルをTiKVストアラベルとマージし、ラベルを更新してマージされた結果を永続化します。
> - TiUPを使用してストアのラベルを管理するには、クラスターを再起動する前にPDに格納されているラベルを空にするために、`store label <id> --force`コマンドを実行できます。

#### ストアの重みを設定する {#configure-store-weight}

IDが1のストアのリーダーウェイトを5に、リージョンウェイトを10に設定するには、次のコマンドを実行します：

```bash
store weight 1 5 10
```

#### ストアスケジューリング速度の設定 {#configure-store-scheduling-speed}

`store limit`を使用してストアのスケジューリング速度を設定できます。 `store limit`の原則と使用法の詳細については、[`store limit`](/configure-store-limit.md)を参照してください。

```bash
>> store limit                         // Show the speed limit of adding-peer operations and the limit of removing-peer operations per minute in all stores
>> store limit add-peer                // Show the speed limit of adding-peer operations per minute in all stores
>> store limit remove-peer             // Show the limit of removing-peer operations per minute in all stores
>> store limit all 5                   // Set the limit of adding-peer operations to 5 and the limit of removing-peer operations to 5 per minute for all stores
>> store limit 1 5                     // Set the limit of adding-peer operations to 5 and the limit of removing-peer operations to 5 per minute for store 1
>> store limit all 5 add-peer          // Set the limit of adding-peer operations to 5 per minute for all stores
>> store limit 1 5 add-peer            // Set the limit of adding-peer operations to 5 per minute for store 1
>> store limit 1 5 remove-peer         // Set the limit of removing-peer operations to 5 per minute for store 1
>> store limit all 5 remove-peer       // Set the limit of removing-peer operations to 5 per minute for all stores
```

> **Note:**
>
> `pd-ctl`を使用して、TiKVストアの状態（`Up`、`Disconnect`、`Offline`、`Down`、または`Tombstone`）を確認できます。各状態の関係については、[TiKVストアの各状態の関係](/tidb-scheduling.md#information-collection)を参照してください。

### `log [fatal | error | warn | info | debug]` {#log-fatal-error-warn-info-debug}

このコマンドを使用して、PDリーダーのログレベルを設定します。

使用法:

```bash
log warn
```

### `tso` {#tso}

このコマンドを使用して、TSOの物理的および論理的な時間を解析します。

使用法：

```bash
>> tso 395181938313123110        // Parse TSO
system:  2017-10-09 05:50:59 +0800 CST
logic:  120102
```

### `unsafe remove-failed-stores [store-ids | show]` {#unsafe-remove-failed-stores-store-ids-show}

> **警告:**
>
> - この機能は損失が発生するリカバリなので、TiKVはこの機能を使用した後にデータの整合性とデータインデックスの整合性を保証することができません。
> - この機能に関連する操作は、TiDBチームのサポートを受けて実行することをお勧めします。誤操作が行われた場合、クラスターを回復することが難しいかもしれません。

永続的に損傷したレプリカがデータを利用できなくする場合に、このコマンドを使用して損失が発生するリカバリ操作を実行します。以下に例を示します。詳細は[オンライン不安全リカバリ](/online-unsafe-recovery.md)に記載されています。

オンライン不安全リカバリを実行して永続的に損傷したストアを削除します：

```bash
unsafe remove-failed-stores 101,102,103
```

```bash
Success!
```

次のコマンドを使用して、オンラインの安全でないリカバリの現在または過去の状態を表示します：

```bash
unsafe remove-failed-stores show
```

```bash
[
  "Collecting cluster info from all alive stores, 10/12.",
  "Stores that have reports to PD: 1, 2, 3, ...",
  "Stores that have not reported to PD: 11, 12",
]
```

## JqフォーマットされたJSON出力の使用 {#jq-formatted-json-output-usage}

### `store`の出力を簡略化 {#simplify-the-output-of-store}

```bash
>> store --jq=".stores[].store | { id, address, state_name}"
{"id":1,"address":"127.0.0.1:20161","state_name":"Up"}
{"id":30,"address":"127.0.0.1:20162","state_name":"Up"}
...
```

### ノードの残りのスペースをクエリする {#query-the-remaining-space-of-the-node}

```bash
>> store --jq=".stores[] | {id: .store.id, available: .status.available}"
{"id":1,"available":"10 GiB"}
{"id":30,"available":"10 GiB"}
...
```

### `Up` でないすべてのノードのクエリ {#query-all-nodes-whose-status-is-not-up}

```bash
store --jq='.stores[].store | select(.state_name!="Up") | { id, address, state_name}'
```

```
{"id":1,"address":"127.0.0.1:20161""state_name":"Offline"}
{"id":5,"address":"127.0.0.1:20162""state_name":"Offline"}
...
```

### すべてのTiFlashノードをクエリします {#query-all-tiflash-nodes}

```bash
store --jq='.stores[].store | select(.labels | length>0 and contains([{"key":"engine","value":"tiflash"}])) | { id, address, state_name}'
```

```
{"id":1,"address":"127.0.0.1:20161""state_name":"Up"}
{"id":5,"address":"127.0.0.1:20162""state_name":"Up"}
...
```

### リージョンレプリカの分布状況をクエリします。 {#query-the-distribution-status-of-the-region-replicas}

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id]}"
{"id":2,"peer_stores":[1,30,31]}
{"id":4,"peer_stores":[1,31,34]}
...
```

### レプリカの数に応じてリージョンをフィルタリングする {#filter-regions-according-to-the-number-of-replicas}

例えば、レプリカの数が3でないすべてのリージョンをフィルタリングするには、次のようにします：

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(length != 3)}"
{"id":12,"peer_stores":[30,32]}
{"id":2,"peer_stores":[1,30,31,32]}
```

### ストア30のレプリカのストアIDに基づいてリージョンをフィルタリングする {#filter-regions-according-to-the-store-id-of-replicas}

例えば、ストア30にレプリカがあるすべてのリージョンをフィルタリングするには、

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(any(.==30))}"
{"id":6,"peer_stores":[1,30,31]}
{"id":22,"peer_stores":[1,30,32]}
...
```

同じ方法で、store30またはstore31にレプリカがあるすべてのリージョンも見つけることができます。

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(any(.==(30,31)))}"
{"id":16,"peer_stores":[1,30,34]}
{"id":28,"peer_stores":[1,30,32]}
{"id":12,"peer_stores":[30,32]}
...
```

### 関連するリージョンを探す {#look-for-relevant-regions-when-restoring-data}

例えば、\[store1、store30、store31] がダウンしているとき、通常のレプリカよりもダウンしているレプリカが多いすべてのリージョンを見つけることができます。

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(length as $total | map(if .==(1,30,31) then . else empty end) | length>=$total-length) }"
{"id":2,"peer_stores":[1,30,31,32]}
{"id":12,"peer_stores":[30,32]}
{"id":14,"peer_stores":[1,30,32]}
...
```

または、\[store1、store30、store31]が起動に失敗した場合、store1で安全に手動でデータを削除できるリージョンを見つけることができます。 この方法で、store1にレプリカがあるが他のDownPeersがないすべてのリージョンをフィルタリングできます。

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(length>1 and any(.==1) and all(.!=(30,31)))}"
{"id":24,"peer_stores":[1,32,33]}
```

\[store30, store31]がダウンしている場合、`remove-peer`オペレーターを作成して安全に処理できるすべてのリージョンを見つけます。つまり、1つだけのDownPeerを持つリージョンです。

```bash
>> region --jq=".regions[] | {id: .id, remove_peer: [.peers[].store_id] | select(length>1) | map(if .==(30,31) then . else empty end) | select(length==1)}"
{"id":12,"remove_peer":[30]}
{"id":4,"remove_peer":[31]}
{"id":22,"remove_peer":[30]}
...
```
