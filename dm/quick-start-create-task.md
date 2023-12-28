---
title: Create a Data Migration Task
summary: Learn how to create a migration task after the DM cluster is deployed.
---

# データ移行タスクの作成 {#create-a-data-migration-task}

このドキュメントでは、DMクラスターが正常にデプロイされた後に、簡単なデータ移行タスクを作成する方法について説明します。

## サンプルシナリオ {#sample-scenario}

このサンプルシナリオを基に、データ移行タスクを作成するとします。

- binlogが有効なMySQLインスタンスを2つデプロイする
- ローカルにTiDBインスタンスを1つデプロイする
- DMクラスターのDM-masterを使用して、クラスターとデータ移行タスクを管理する

各ノードの情報は以下の通りです。

| インスタンス    | サーバーアドレス  | ポート  |
| :-------- | :-------- | :--- |
| MySQL1    | 127.0.0.1 | 3306 |
| MySQL2    | 127.0.0.1 | 3307 |
| TiDB      | 127.0.0.1 | 4000 |
| DM-master | 127.0.0.1 | 8261 |

このシナリオに基づいて、以下のセクションではデータ移行タスクの作成方法について説明します。

### 上流のMySQLを起動する {#start-upstream-mysql}

実行可能なMySQLインスタンスを2つ準備します。また、Dockerを使用してMySQLを素早く起動することもできます。コマンドは以下の通りです。

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

### 下流のTiDBを起動する {#start-downstream-tidb}

TiDBサーバーを実行するには、次のコマンドを使用します：

```bash
wget https://download.pingcap.org/tidb-community-server-v7.1.3-linux-amd64.tar.gz
tar -xzvf tidb-latest-linux-amd64.tar.gz
mv tidb-latest-linux-amd64/bin/tidb-server ./
./tidb-server
```

> **警告：**
>
> このドキュメントでのTiDBのデプロイ方法は、本番環境や開発環境には**適用できません**。

## MySQLデータソースの設定 {#configure-the-mysql-data-source}

データ移行タスクを開始する前に、MySQLデータソースを設定する必要があります。

### パスワードの暗号化 {#encrypt-the-password}

> **注意：**
>
> - データベースにパスワードがない場合は、このステップをスキップできます。
> - DM v1.0.6以降のバージョンでは、ソース情報を構成するために平文のパスワードを使用できます。

安全上の理由から、暗号化されたパスワードを設定して使用することをお勧めします。dmctlを使用してMySQL/TiDBのパスワードを暗号化することができます。パスワードが"123456"であると仮定します。

```bash
./dmctl encrypt "123456"
```

    fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg=

以下の手順で、MySQLデータソースを作成するために、この暗号化された値を保存して使用してください。

### ソースの設定ファイルを編集する {#edit-the-source-configuration-file}

`conf/source1.yaml`に以下の設定を書き込んでください。

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

MySQL2のデータソースでは、上記の設定を`conf/source2.yaml`にコピーします。`name`を`mysql-replica-02`に変更し、`password`と`port`を適切な値に変更する必要があります。

### ソースを作成する {#create-a-source}

dmctlを使用してDMクラスターにMySQL1のデータソース設定をロードするには、ターミナルで次のコマンドを実行します：

```bash
./dmctl --master-addr=127.0.0.1:8261 operate-source create conf/source1.yaml
```

MySQL2の場合、上記のコマンドの設定ファイルをMySQL2のものに置き換えます。

## データ移行タスクの作成 {#create-a-data-migration-task}

[準備されたデータ](#prepare-data)をインポートした後、MySQL1とMySQL2の両方のインスタンスにいくつかのシャードされたテーブルがあります。これらのテーブルは同じ構造を持ち、テーブル名の接頭辞にはすべて「t」が付いています。これらのテーブルがあるデータベースにはすべて「sharding」が付いており、プライマリキーやユニークキーには競合がありません（各シャードされたテーブルでは、プライマリキーやユニークキーは他のテーブルと異なります）。

今、これらのシャードされたテーブルをTiDBの`db_target.t_target`テーブルに移行する必要があるとします。手順は以下のとおりです。

1. タスクの設定ファイルを作成します：

   ```yaml
   ---
   name: test
   task-mode: all
   shard-mode: "pessimistic"
   target-database:
     host: "127.0.0.1"
     port: 4000
     user: "root"
     password: "" # パスワードが空でない場合は、dmctlで暗号化されたパスワードを使用することをお勧めします。

   mysql-instances:
     - source-id: "mysql-replica-01"
       block-allow-list:  "instance"  # この設定はDMバージョンv2.0.0-beta.2よりも新しいバージョンに適用されます。それ以外の場合は、black-white-listを使用してください。
       route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
       mydumper-thread: 4
       loader-thread: 16
       syncer-thread: 16
     - source-id: "mysql-replica-02"
       block-allow-list:  "instance"  # この設定はDMバージョンv2.0.0-beta.2よりも新しいバージョンに適用されます。それ以外の場合は、black-white-listを使用してください。
       route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
       mydumper-thread: 4
       loader-thread: 16
       syncer-thread: 16
   block-allow-list:  # この設定はDMバージョンv2.0.0-beta.2よりも新しいバージョンに適用されます。それ以外の場合は、black-white-listを使用してください。
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

2. dmctlを使用してタスクを作成するには、上記の設定を`conf/task.yaml`ファイルに書き込みます：

   ```bash
   ./dmctl --master-addr 127.0.0.1:8261 start-task conf/task.yaml
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

これで、MySQL1とMySQL2のインスタンスからTiDBにシャードされたテーブルを移行するタスクが正常に作成されました。

## データの検証 {#verify-data}

上流のMySQLのシャードされたテーブルでデータを変更することができます。その後、[sync-diff-inspector](/sync-diff-inspector/shard-diff.md)を使用して、上流と下流のデータが一致しているかどうかを確認します。一致するデータは、移行タスクが正常に動作していることを意味し、クラスターが正常に動作していることを示します。
