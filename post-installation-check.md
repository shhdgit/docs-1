---
title: Check Cluster Status
summary: Learn how to check the running status of the TiDB cluster.
---

# クラスターの状態を確認する {#check-cluster-status}

TiDBクラスターがデプロイされた後、クラスターが正常に動作しているかどうかを確認する必要があります。このドキュメントでは、TiUPコマンド、[TiDBダッシュボード](/dashboard/dashboard-intro.md)、およびGrafanaを使用してクラスターの状態を確認する方法、およびTiDBデータベースにログインして簡単なSQL操作を実行する方法について説明します。

## TiDBクラスターの状態を確認する {#check-the-tidb-cluster-status}

このセクションでは、TiUPコマンド、[TiDBダッシュボード](/dashboard/dashboard-intro.md)、およびGrafanaを使用してTiDBクラスターの状態を確認する方法について説明します。

### TiUPを使用する {#use-tiup}

`tiup cluster display <cluster-name>`コマンドを使用して、クラスターの状態を確認します。例えば：

```shell
tiup cluster display tidb-test
```

各ノードの`Status`情報が`Up`であれば、クラスターは正常に動作しています。

### TiDBダッシュボードを使用する {#use-tidb-dashboard}

1. `${pd-ip}:${pd-port}/dashboard`にアクセスして、TiDBダッシュボードにログインします。ユーザー名とパスワードはTiDBの`root`ユーザーと同じです。`root`のパスワードを変更した場合は、変更したパスワードを入力してください。デフォルトではパスワードは空です。

   ![TiDB-Dashboard](/media/tiup/tidb-dashboard.png)

2. ホームページにはTiDBクラスターのノード情報が表示されます。

   ![TiDB-Dashboard-status](/media/tiup/tidb-dashboard-status.png)

### Grafanaを使用する {#use-grafana}

1. `${Grafana-ip}:3000`にアクセスして、Grafanaモニタリングにログインします。デフォルトのユーザー名とパスワードはどちらも`admin`です。

2. TiDBのポート状態と負荷モニタリング情報を確認するには、**Overview**をクリックします。

   ![Grafana-overview](/media/tiup/grafana-overview.png)

## データベースにログインして簡単な操作を実行する {#log-in-to-the-database-and-perform-simple-operations}

> **Note:**
>
> データベースにログインする前に、MySQLクライアントをインストールしてください。

以下のコマンドを実行してデータベースにログインします：

```shell
mysql -u root -h ${tidb_server_host_IP_address} -P 4000
```

`${tidb_server_host_IP_address}`は、`tidb_servers`に設定されたIPアドレスの1つであり、[クラスタトポロジファイルを初期化する](/production-deployment-using-tiup.md#step-3-initialize-cluster-topology-file)際に使用されます。例えば、`10.0.1.7`のようなものです。

以下の情報は、ログインが成功したことを示しています：

```sql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 407
Server version: 5.7.25-TiDB-v7.1.3 TiDB Server (Apache License 2.0) Community Edition, MySQL 5.7 compatible

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

### データベース操作 {#database-operations}

- TiDBのバージョンを確認する：

  ```sql
  select tidb_version()\G
  ```

  期待される出力：

  ```sql
  *************************** 1. row ***************************
  tidb_version(): リリースバージョン: v5.0.0
  エディション: コミュニティ
  Gitコミットハッシュ: 689a6b6439ae7835947fcaccf329a3fc303986cb
  Gitブランチ: HEAD
  UTCビルド時間: 2020-05-28 11:09:45
  Goバージョン: go1.13.4
  レース有効化: false
  TiKV最小バージョン: v3.0.0-60965b006877ca7234adaced7890d7b029ed1306
  ドロップ前にテーブルをチェック: false
  1行がセットされました (0.00 sec)
  ```

- `pingcap`という名前のデータベースを作成する：

  ```sql
  create database pingcap;
  ```

  期待される出力：

  ```sql
  クエリーOK、0行が影響を受けました (0.10 sec)
  ```

  `pingcap`データベースに切り替える：

  ```sql
  use pingcap;
  ```

  期待される出力：

  ```sql
  データベースが変更されました
  ```

- `tab_tidb`という名前のテーブルを作成する：

  ```sql
  CREATE TABLE `tab_tidb` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL DEFAULT '',
  `age` int(11) NOT NULL DEFAULT 0,
  `version` varchar(20) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  KEY `idx_age` (`age`));
  ```

  期待される出力：

  ```sql
  クエリーOK、0行が影響を受けました (0.11 sec)
  ```

- データを挿入する：

  ```sql
  insert into `tab_tidb` values (1,'TiDB',5,'TiDB-v5.0.0');
  ```

  期待される出力：

  ```sql
  クエリーOK、1行が影響を受けました (0.03 sec)
  ```

- `tab_tidb`のエントリーを表示する：

  ```sql
  select * from tab_tidb;
  ```

  期待される出力：

  ```sql
  +----+------+-----+-------------+
  | id | name | age | version     |
  +----+------+-----+-------------+
  |  1 | TiDB |   5 | TiDB-v5.0.0 |
  +----+------+-----+-------------+
  1行がセットされました (0.00 sec)
  ```

- TiKVのストア状態、`store_id`、容量、および稼働時間を表示する：

  ```sql
  select STORE_ID,ADDRESS,STORE_STATE,STORE_STATE_NAME,CAPACITY,AVAILABLE,UPTIME from INFORMATION_SCHEMA.TIKV_STORE_STATUS;
  ```

  期待される出力：

  ```sql
  +----------+--------------------+-------------+------------------+----------+-----------+--------------------+
  | STORE_ID | ADDRESS            | STORE_STATE | STORE_STATE_NAME | CAPACITY | AVAILABLE | UPTIME             |
  +----------+--------------------+-------------+------------------+----------+-----------+--------------------+
  |        1 | 10.0.1.1:20160 |           0 | Up               | 49.98GiB | 46.3GiB   | 5h21m52.474864026s |
  |        4 | 10.0.1.2:20160 |           0 | Up               | 49.98GiB | 46.32GiB  | 5h21m52.522669177s |
  |        5 | 10.0.1.3:20160 |           0 | Up               | 49.98GiB | 45.44GiB  | 5h21m52.713660541s |
  +----------+--------------------+-------------+------------------+----------+-----------+--------------------+
  3行がセットされました (0.00 sec)
  ```

- TiDBを終了する：

  ```sql
  exit
  ```

  期待される出力：

  ```sql
  Bye
  ```
