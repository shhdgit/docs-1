---
title: Create a Data Migration Task
summary: Learn how to create a migration task after the DM cluster is deployed.
---

# データ移行タスクの作成 {#create-a-data-migration-task}

このドキュメントでは、DMクラスターが正常にデプロイされた後にシンプルなデータ移行タスクを作成する方法について説明します。

## サンプルシナリオ {#sample-scenario}

このサンプルシナリオに基づいてデータ移行タスクを作成するとします。

- binlogが有効になっている2つのMySQLインスタンスと1つのTiDBインスタンスをローカルにデプロイします
- DMクラスターのDM-masterを使用してクラスターとデータ移行タスクを管理します。

各ノードの情報は次のとおりです。

| インスタンス    | サーバーアドレス  | ポート  |
| :-------- | :-------- | :--- |
| MySQL1    | 127.0.0.1 | 3306 |
| MySQL2    | 127.0.0.1 | 3307 |
| TiDB      | 127.0.0.1 | 4000 |
| DM-master | 127.0.0.1 | 8261 |

このシナリオに基づいて、次のセクションではデータ移行タスクの作成方法について説明します。

### 上流のMySQLを開始する {#start-upstream-mysql}

実行可能なMySQLインスタンスを2つ準備します。Dockerを使用してもMySQLを素早く開始することができます。コマンドは次のとおりです：

```bash
docker run --rm --name mysql-3306 -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7.22 --log-bin=mysql-bin --port=3306 --bind-address=0.0.0.0 --binlog-format=ROW --server-id=1 --gtid_mode=ON --enforce-gtid-consistency=true > mysql.3306.log 2>&1 &
docker run --rm --name mysql-3307 -p 3307:3307 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7.22 --log-bin=mysql-bin --port=3307 --bind-address=0.0.0.0 --binlog-format=ROW --server-id=1 --gtid_mode=ON --enforce-gtid-consistency=true > mysql.3307.log 2>&1 &
```

### データの準備 {#prepare-data}

- mysql-3306に例のデータを書き込む：

  ```sql
  drop database if exists `sharding1`;
  create database `sharding1`;
  use `sharding1`;
  create table t1 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4;
  create table t2 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4;
  insert into t1 (id, uid, name) values (1, 10001, 'Gabriel García Márquez'), (2 ,10002, 'Cien años de soledad');
  insert into t2 (id, uid, name) values (3,20001, 'José Arcadio Buendía'), (4,20002, 'Úrsula Iguarán'), (5,20003, 'José Arcadio');
  ```

- mysql-3307に例のデータを書き込む：

  ```sql
  drop database if exists `sharding2`;
  create database `sharding2`;
  use `sharding2`;
  create table t2 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4;
  create table t3 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4;
  insert into t2 (id, uid, name, info) values (6, 40000, 'Remedios Moscote', '{}');
  insert into t3 (id, uid, name, info) values (7, 30001, 'Aureliano José', '{}'), (8, 30002, 'Santa Sofía de la Piedad', '{}'), (9, 30003, '17 Aurelianos', NULL);
  ```

### 下流のTiDBを起動 {#start-downstream-tidb}

TiDBサーバーを実行するには、次のコマンドを使用します：

```bash
wget https://download.pingcap.org/tidb-community-server-v7.1.3-linux-amd64.tar.gz
tar -xzvf tidb-latest-linux-amd64.tar.gz
mv tidb-latest-linux-amd64/bin/tidb-server ./
./tidb-server
```

> **Warning:**
>
> 本書のTiDBのデプロイ方法は、本番環境や開発環境には**適用されません**。

## MySQLデータソースを構成する {#configure-the-mysql-data-source}

データ移行タスクを開始する前に、MySQLデータソースを構成する必要があります。

### パスワードを暗号化する {#encrypt-the-password}

> **Note:**
>
> - データベースにパスワードがない場合、この手順はスキップできます。
> - DM v1.0.6以降のバージョンでは、平文パスワードを使用してソース情報を構成できます。

安全上の理由から、暗号化されたパスワードを構成して使用することをお勧めします。MySQL/TiDBのパスワードを暗号化するには、dmctlを使用できます。パスワードが「123456」であると仮定します。

```bash
./dmctl encrypt "123456"
```

```
fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg=
```

次の手順でMySQLデータソースを作成するために、この暗号化された値を保存して使用してください。

### ソース構成ファイルの編集 {#edit-the-source-configuration-file}

次の構成を `conf/source1.yaml` に書いてください。

```yaml
# MySQL1 Configuration.

source-id: "mysql-replica-01"

# Indicates whether GTID is enabled
enable-gtid: true

from:
  host: "127.0.0.1"
  user: "root"
  password: "fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg="
  port: 3306
```

MySQL2データソースでは、上記の設定を `conf/source2.yaml` にコピーします。 `name` を `mysql-replica-02` に変更し、 `password` と `port` を適切な値に変更する必要があります。

### ソースの作成 {#create-a-source}

DMクラスターにMySQL1のデータソース設定をdmctlを使用してロードするには、ターミナルで次のコマンドを実行します：

```bash
./dmctl --master-addr=127.0.0.1:8261 operate-source create conf/source1.yaml
```

MySQL2の場合、上記のコマンドの構成ファイルをMySQL2のものと置き換えます。

## データ移行タスクの作成 {#create-a-data-migration-task}

[準備されたデータ](#prepare-data)をインポートした後、MySQL1およびMySQL2のインスタンスにいくつかのシャードテーブルがあります。これらのテーブルは同じ構造であり、テーブル名にはすべて「t」の接頭辞があります。これらのテーブルが存在するデータベースにはすべて「sharding」の接頭辞があり、プライマリキーまたはユニークキーの間には競合がありません（各シャードテーブルには、他のテーブルとは異なるプライマリキーまたはユニークキーがあります）。

さて、これらのシャードテーブルをTiDBの`db_target.t_target`テーブルに移行する必要があるとします。手順は次のとおりです。

1. タスクの構成ファイルを作成します：

   ```yaml
   ---
   name: test
   task-mode: all
   shard-mode: "悲観的"
   target-database:
     host: "127.0.0.1"
     port: 4000
     user: "root"
     password: "" # パスワードが空でない場合は、dmctlで暗号化されたパスワードを使用することをお勧めします。

   mysql-instances:
     - source-id: "mysql-replica-01"
       block-allow-list:  "instance"  # この構成は、v2.0.0-beta.2より新しいDMバージョンに適用されます。それ以外の場合はblack-white-listを使用してください。
       route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
       mydumper-thread: 4
       loader-thread: 16
       syncer-thread: 16
     - source-id: "mysql-replica-02"
       block-allow-list:  "instance"  # この構成は、v2.0.0-beta.2より新しいDMバージョンに適用されます。それ以外の場合はblack-white-listを使用してください。
       route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
       mydumper-thread: 4
       loader-thread: 16
       syncer-thread: 16
   block-allow-list:  # この構成は、v2.0.0-beta.2より新しいDMバージョンに適用されます。それ以外の場合はblack-white-listを使用してください。
     instance:
       do-dbs: ["~^sharding[\\d]+"]
       do-tables:
       - db-name: "~^sharding[\\d]+"
         tbl-name: "~^t[\\d]+"
   routes:
     sharding-route-rules-table:
       schema-pattern: sharding*
       table-pattern: t*
       target-schema: db_target
       target-table: t_target
     sharding-route-rules-schema:
       schema-pattern: sharding*
       target-schema: db_target
   ```

2. dmctlを使用してタスクを作成するには、上記の構成を`conf/task.yaml`ファイルに書き込みます：

   ```bash
   ./dmctl --master-addr 127.0.0.1:8261 start-task conf/task.yaml
   ```

   ```
   {
       "result": true,
       "msg": "",
       "sources": [
           {
               "result": true,
               "msg": "",
               "source": "mysql-replica-01",
               "worker": "worker1"
           },
           {
               "result": true,
               "msg": "",
               "source": "mysql-replica-02",
               "worker": "worker2"
           }
       ]
   }
   ```

これで、MySQL1およびMySQL2のインスタンスからTiDBにシャードテーブルを移行するタスクが正常に作成されました。

## データの検証 {#verify-data}

上流のMySQLシャードテーブルでデータを変更できます。その後、[sync-diff-inspector](/sync-diff-inspector/shard-diff.md)を使用して、上流と下流のデータが一致しているかどうかを確認できます。一致するデータは、移行タスクが正常に機能していることを意味し、クラスタが正常に機能していることを示します。
