---
title: Create TiFlash Replicas
summary: Learn how to create TiFlash replicas.
---

# TiFlash レプリカの作成 {#create-tiflash-replicas}

このドキュメントでは、テーブルやデータベースのために TiFlash レプリカを作成し、レプリカのスケジューリングに使用する利用可能ゾーンを設定する方法について説明します。

## テーブルのために TiFlash レプリカを作成する {#create-tiflash-replicas-for-tables}

TiFlash が TiKV クラスターに接続された後、データのレプリケーションはデフォルトでは開始されません。MySQL クライアントを介して TiDB に DDL ステートメントを送信することで、特定のテーブルのために TiFlash レプリカを作成することができます。

```sql
ALTER TABLE table_name SET TIFLASH REPLICA count;
```

上記コマンドのパラメーターは次のように説明されます：

- `count` はレプリカの数を示します。値が `0` の場合、レプリカは削除されます。

同じテーブルに対して複数の DDL ステートメントを実行すると、最後のステートメントのみが効果を持つことが保証されます。次の例では、テーブル `tpch50` に対して 2 つの DDL ステートメントが実行されますが、2 番目のステートメント（レプリカの削除）のみが効果を持ちます。

テーブルに 2 つのレプリカを作成します：

```sql
ALTER TABLE `tpch50`.`lineitem` SET TIFLASH REPLICA 2;
```

レプリカを削除する：

```sql
ALTER TABLE `tpch50`.`lineitem` SET TIFLASH REPLICA 0;
```

**ノート:**

- もし `t` テーブルが上記の DDL ステートメントを使用して TiFlash にレプリケートされた場合、次のステートメントを使用して作成されたテーブルも自動的に TiFlash にレプリケートされます:

  ```sql
  CREATE TABLE table_name like t;
  ```

- v4.0.6 より前のバージョンでは、TiDB Lightning を使用してデータをインポートする前に TiFlash レプリカを作成すると、データのインポートに失敗します。テーブルの TiFlash レプリカを作成する前にデータをインポートする必要があります。

- TiDB と TiDB Lightning がともに v4.0.6 以上の場合、テーブルに TiFlash レプリカがあるかどうかに関わらず、TiDB Lightning を使用してそのテーブルにデータをインポートすることができます。ただし、これにより TiDB Lightning の手順が遅くなる可能性があります。これは、ライトニング ホストの NIC 帯域幅、TiFlash ノードの CPU およびディスク負荷、および TiFlash レプリカの数に依存します。

- PD のスケジューリング パフォーマンスを低下させるため、1,000 以上のテーブルをレプリケートしないように推奨します。この制限は、後のバージョンで削除されます。

- v5.1 以降のバージョンでは、システム テーブルのレプリカを設定することはサポートされなくなりました。クラスターをアップグレードする前に、関連するシステム テーブルのレプリカをクリアする必要があります。そうしないと、クラスターを後のバージョンにアップグレードした後、システム テーブルのレプリカ設定を変更することはできません。

### レプリケーションの進捗状況を確認する {#check-replication-progress}

次のステートメントを使用して、特定のテーブルの TiFlash レプリカの状態を確認できます。テーブルは `WHERE` 句で指定します。`WHERE` 句を削除すると、すべてのテーブルのレプリカ状態を確認します。

```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = '<db_name>' and TABLE_NAME = '<table_name>';
```

上記のステートメントの結果：

- `AVAILABLE` は、このテーブルの TiFlash レプリカが利用可能かどうかを示します。 `1` は利用可能を意味し、 `0` は利用不可を意味します。レプリカが利用可能になると、このステータスは変更されません。DDL ステートメントを使用してレプリカの数を変更すると、レプリケーションのステータスが再計算されます。
- `PROGRESS` は、レプリケーションの進行状況を示します。値は `0.0` から `1.0` の間です。 `1` は少なくとも 1 つのレプリカがレプリケートされていることを意味します。

## データベースの TiFlash レプリカを作成する {#create-tiflash-replicas-for-databases}

テーブルの TiFlash レプリカを作成するのと同様に、MySQL クライアントを介して TiDB に DDL ステートメントを送信して、特定のデータベースのすべてのテーブルの TiFlash レプリカを作成することができます：

```sql
ALTER DATABASE db_name SET TIFLASH REPLICA count;
```

このステートメントでは、 `count` はレプリカの数を示します。 `0` に設定すると、レプリカが削除されます。

例：

- データベース `tpch50` のすべてのテーブルに 2 つのレプリカを作成する：

  ```sql
  ALTER DATABASE `tpch50` SET TIFLASH REPLICA 2;
  ```

- データベース `tpch50` に作成された TiFlash レプリカを削除する：

  ```sql
  ALTER DATABASE `tpch50` SET TIFLASH REPLICA 0;
  ```

> **Note:**
>
> - このステートメントは実際には一連の DDL 操作を実行します。これらの操作はリソースを消費します。ステートメントの実行中に中断された場合、実行された操作はロールバックされず、実行されなかった操作は継続されません。
>
> - ステートメントを実行した後、このデータベースで TiFlash レプリカの数を設定したり、DDL 操作を実行したりしないでください。**このデータベースのすべてのテーブルがレプリケートされるまで**。そうしないと、予期しない結果が発生する可能性があります。これには次のものがあります。
>   - データベースのテーブルの数を 2 に設定し、その後すべてのテーブルがレプリケートされる前に数を 1 に変更した場合、すべてのテーブルの最終的な TiFlash レプリカの数は必ずしも 1 または 2 ではありません。
>   - ステートメントを実行した後、ステートメントの実行が完了する前にこのデータベースでテーブルを作成すると、新しいテーブルの TiFlash レプリカが作成されるかどうかは**必ずしも**保証されません。
>   - ステートメントを実行した後、ステートメントの実行が完了する前にこのデータベースのテーブルにインデックスを追加すると、ステートメントがハングし、インデックスが追加された後にのみ再開される可能性があります。
>
> - ステートメントの実行が完了した後にこのデータベースでテーブルを作成すると、これらの新しいテーブルの TiFlash レプリカは自動的に作成されません。
>
> - このステートメントは、システムテーブル、ビュー、一時テーブル、および TiFlash でサポートされていない文字セットのテーブルをスキップします。

### レプリケーションの進捗状況を確認する {#check-replication-progress}

テーブルに TiFlash レプリカを作成するのと同様に、DDL ステートメントの実行が成功したとしても、レプリケーションが完了したとは限りません。次の SQL ステートメントを実行して、ターゲットテーブルのレプリケーションの進捗状況を確認できます：

```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = '<db_name>';
```

データベース内の TiFlash レプリカを持たないテーブルをチェックするには、次の SQL ステートメントを実行できます。

```sql
SELECT TABLE_NAME FROM information_schema.tables where TABLE_SCHEMA = "<db_name>" and TABLE_NAME not in (SELECT TABLE_NAME FROM information_schema.tiflash_replica where TABLE_SCHEMA = "<db_name>");
```

## TiFlash レプリケーションの高速化 {#speed-up-tiflash-replication}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> このセクションは TiDB Cloud には適用されません。

</CustomContent>

TiFlash レプリカが追加される前に、各 TiKV インスタンスはフルテーブルスキャンを実行し、スキャンされたデータを "スナップショット" として TiFlash に送信してレプリカを作成します。デフォルトでは、オンラインサービスへの影響を最小限に抑えるため、TiFlash レプリカはリソース使用量を抑えてゆっくりと追加されます。TiKV と TiFlash ノードに余分な CPU とディスク IO リソースがある場合は、次の手順を実行することで TiFlash レプリケーションを加速することができます。

1. [動的設定 SQL ステートメント](https://docs.pingcap.com/tidb/stable/dynamic-config) を使用して、一時的に各 TiKV と TiFlash インスタンスのスナップショット書き込み速度制限を増やします。

   ```sql
   -- 両方の設定のデフォルト値は 100MiB で、スナップショットの書き込みに使用される最大ディスク帯域幅は 100MiB/s 以下です。
   SET CONFIG tikv `server.snap-io-max-bytes-per-sec` = '300MiB';
   SET CONFIG tiflash `raftstore-proxy.server.snap-max-write-bytes-per-sec` = '300MiB';
   ```

   これらの SQL ステートメントを実行すると、クラスターを再起動することなく設定が即座に有効になります。ただし、レプリケーション速度は引き続き PD の制限によって制限されるため、現時点では加速が観測できません。

2. [PD Control](https://docs.pingcap.com/tidb/stable/pd-control) を使用して、新しいレプリカ速度制限を段階的に緩和します。

   デフォルトの新しいレプリカ速度制限は 30 で、つまりおよそ 1 分に 30 個のリージョンが TiFlash レプリカを追加します。次のコマンドを実行すると、すべての TiFlash インスタンスの制限が 60 に調整され、元の速度の 2 倍になります。

   ```shell
   tiup ctl:v<CLUSTER_VERSION> pd -u http://<PD_ADDRESS>:2379 store limit all engine tiflash 60 add-peer
   ```

   > 上記のコマンドでは、`v<CLUSTER_VERSION>` を実際のクラスターバージョン（例えば `v7.1.3`）に、`<PD_ADDRESS>:2379` を任意の PD ノードのアドレスに置き換える必要があります。例えば:
   >
   > ```shell
   > tiup ctl:v7.1.3 pd -u http://192.168.1.4:2379 store limit all engine tiflash 60 add-peer
   > ```

   数分後に、TiFlash ノードの CPU とディスク IO リソース使用量が大幅に増加し、TiFlash がレプリカをより速く作成することができるようになります。同時に、TiKV ノードの CPU とディスク IO リソース使用量も増加します。

   この時点で TiKV と TiFlash ノードにまだ余分なリソースがある場合、オンラインサービスのレイテンシーが大幅に増加しない場合は、さらに制限を緩和することができます。例えば、元の速度の 3 倍にすることができます。

   ```shell
   tiup ctl:v<CLUSTER_VERSION> pd -u http://<PD_ADDRESS>:2379 store limit all engine tiflash 90 add-peer
   ```

3. TiFlash レプリケーションが完了したら、オンラインサービスへの影響を減らすためにデフォルトの設定に戻します。

   次の PD Control コマンドを実行して、デフォルトの新しいレプリカ速度制限を復元します。

   ```shell
   tiup ctl:v<CLUSTER_VERSION> pd -u http://<PD_ADDRESS>:2379 store limit all engine tiflash 30 add-peer
   ```

   次の SQL ステートメントを実行して、デフォルトのスナップショット書き込み速度制限を復元します。

   ```sql
   SET CONFIG tikv `server.snap-io-max-bytes-per-sec` = '100MiB';
   SET CONFIG tiflash `raftstore-proxy.server.snap-max-write-bytes-per-sec` = '100MiB';
   ```

## 利用可能なゾーンを設定する {#set-available-zones}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> このセクションは TiDB Cloud には適用されません。

</CustomContent>

レプリカを構成する際、ディザスタリカバリのために複数のデータセンターに TiFlash レプリカを分散させる必要がある場合は、以下の手順に従って利用可能なゾーンを設定することができます。

1. クラスター構成ファイルで TiFlash ノードのラベルを指定します。

       tiflash_servers:
         - host: 172.16.5.81
             logger.level: "info"
           learner_config:
             server.labels:
               zone: "z1"
         - host: 172.16.5.82
           config:
             logger.level: "info"
           learner_config:
             server.labels:
               zone: "z1"
         - host: 172.16.5.85
           config:
             logger.level: "info"
           learner_config:
             server.labels:
               zone: "z2"

   以前のバージョンでは、`flash.proxy.labels` 構成は利用可能なゾーン名の特殊文字を正しく処理できません。利用可能なゾーンの名前を構成するには、`learner_config` 内の `server.labels` を使用することをお勧めします。

2. クラスターを起動した後、レプリカを作成する際にラベルを指定します。

   ```sql
   ALTER TABLE table_name SET TIFLASH REPLICA count LOCATION LABELS location_labels;
   ```

   例:

   ```sql
   ALTER TABLE t SET TIFLASH REPLICA 2 LOCATION LABELS "zone";
   ```

3. PD はラベルに基づいてレプリカをスケジュールします。この例では、PD はそれぞれのテーブル `t` の 2 つのレプリカを 2 つの利用可能なゾーンにスケジュールします。スケジュールを表示するには、pd-ctl を使用できます。

   ```shell
   > tiup ctl:v<CLUSTER_VERSION> pd -u http://<PD_ADDRESS>:2379 store

       ...
       "address": "172.16.5.82:23913",
       "labels": [
         { "key": "engine", "value": "tiflash"},
         { "key": "zone", "value": "z1" }
       ],
       "region_count": 4,

       ...
       "address": "172.16.5.81:23913",
       "labels": [
         { "key": "engine", "value": "tiflash"},
         { "key": "zone", "value": "z1" }
       ],
       "region_count": 5,
       ...

       "address": "172.16.5.85:23913",
       "labels": [
         { "key": "engine", "value": "tiflash"},
         { "key": "zone", "value": "z2" }
       ],
       "region_count": 9,
       ...
   ```

<CustomContent platform="tidb">

ラベルを使用してレプリカをスケジュールする詳細については、[トポロジラベルによるレプリカのスケジュール](/schedule-replicas-by-topology-labels.md)、[1 つの地域展開における複数のデータセンター](/multi-data-centers-in-one-city-deployment.md)、および [2 つの地域に配置された 3 つのデータ センター](/three-data-centers-in-two-cities-deployment.md) を参照してください。

</CustomContent>
