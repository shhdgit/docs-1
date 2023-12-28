---
title: TiDB Binlog Tutorial
summary: Learn to deploy TiDB Binlog with a simple TiDB cluster.
---

# TiDB Binlog チュートリアル {#tidb-binlog-tutorial}

このチュートリアルでは、各コンポーネント（配置ドライバー、TiKVサーバー、TiDBサーバー、ポンプ、およびドレーナー）の単一ノードを使用して、MariaDBサーバーインスタンスにデータをプッシュするように設定された、単純なTiDB Binlogデプロイメントから始めます。

このチュートリアルは、[TiDBアーキテクチャ](/tidb-architecture.md)についてある程度の知識を持っているユーザー、すでにTiDBクラスターを設定したことがあるかもしれないユーザー（必須ではありません）、およびTiDB Binlogのハンズオン体験をしたいユーザーを対象としています。このチュートリアルは、TiDB Binlogの「試し蹴り」をするのに最適であり、そのアーキテクチャの概念に慣れるのに役立ちます。

> **警告：**
>
> このチュートリアルでTiDBをデプロイするための手順は、本番環境や開発環境にTiDBをデプロイするために使用しては**いけません**。

このチュートリアルでは、x86-64のモダンなLinuxディストリビューションを使用していることを前提としています。このチュートリアルでは、例としてVMwareで実行されている最小限のCentOS 7インストールを使用しています。既存の環境のクセに影響を受けないように、クリーンインストールから始めることをお勧めします。ローカル仮想化を使用したくない場合は、クラウドサービスを使用して簡単にCentOS 7 VMを起動できます。

## TiDB Binlogの概要 {#tidb-binlog-overview}

TiDB Binlogは、TiDBからバイナリログデータを収集し、リアルタイムのデータバックアップとレプリケーションを提供するソリューションです。これは、TiDBサーバークラスターから増分データ更新を下流プラットフォームにプッシュします。

TiDB Binlogを使用すると、増分バックアップを行ったり、1つのTiDBクラスターから別のTiDBクラスターにデータをレプリケートしたり、TiDBの更新をKafkaを介して下流プラットフォームに送信したりすることができます。

TiDB Binlogは、MySQLやMariaDBからTiDBにデータを移行する場合に特に有用です。この場合、TiDB DM（データ移行）プラットフォームを使用して、MySQL/MariaDBクラスターからデータをTiDBに取得し、その後、TiDB Binlogを使用して別の下流MySQL/MariaDBインスタンス/クラスターをTiDBクラスターと同期させることができます。TiDB Binlogにより、TiDBへのアプリケーショントラフィックを下流のMySQLやMariaDBインスタンス/クラスターにプッシュすることができるため、TiDBへの移行のリスクが低くなります。なぜなら、アプリケーションをダウンタイムやデータ損失なしにMySQLやMariaDBに簡単に戻すことができるからです。

詳細については、[TiDB Binlogクラスターユーザーガイド](/tidb-binlog/tidb-binlog-overview.md)を参照してください。

## アーキテクチャ {#architecture}

TiDB Binlogには、**ポンプ**と**ドレーナー**の2つのコンポーネントがあります。複数のポンプノードでポンプクラスターを構成します。各ポンプノードはTiDBサーバーインスタンスに接続し、クラスター内の各TiDBサーバーインスタンスで行われた更新を受信します。ドレーナーはポンプクラスターに接続し、受信した更新を特定の下流先（例えば、Kafka、別のTiDBクラスター、またはMySQL/MariaDBサーバー）の正しい形式に変換します。

![TiDB-Binlogアーキテクチャ](/media/tidb-binlog-cluster-architecture.png)

ポンプのクラスター化されたアーキテクチャにより、新しいTiDBサーバーインスタンスがTiDBクラスターに参加したり離脱したりするときに更新が失われることはありません。また、ポンプノードがポンプクラスターに参加したり離脱したりするときにも同様です。

## インストール {#installation}

この場合、MySQL Serverの代わりにMariaDB Serverを使用しています。なぜなら、RHEL/CentOS 7にはMariaDB Serverがデフォルトのパッケージリポジトリに含まれているためです。後で使用するために、クライアントとサーバーの両方が必要です。それでは、今すぐインストールしましょう：

```bash
sudo yum install -y mariadb-server
```

```bash
curl -L https://download.pingcap.org/tidb-community-server-v7.1.3-linux-amd64.tar.gz | tar xzf -
cd tidb-latest-linux-amd64
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    [kolbe@localhost ~]$ curl -LO https://download.pingcap.org/tidb-latest-linux-amd64.tar.gz | tar xzf -
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100  368M  100  368M    0     0  8394k      0  0:00:44  0:00:44 --:--:-- 11.1M
    [kolbe@localhost ~]$ cd tidb-latest-linux-amd64
    [kolbe@localhost tidb-latest-linux-amd64]$

## コンフィグレーション {#configuration}

今から、単一のインスタンスを持つ `pd-server`、`tikv-server`、`tidb-server` のそれぞれについて、シンプルな TiDB クラスタを開始します。

次のコマンドを使用して、コンフィグファイルを作成します。

```bash
printf > pd.toml %s\\n 'log-file="pd.log"' 'data-dir="pd.data"'
printf > tikv.toml %s\\n 'log-file="tikv.log"' '[storage]' 'data-dir="tikv.data"' '[pd]' 'endpoints=["127.0.0.1:2379"]' '[rocksdb]' max-open-files=1024 '[raftdb]' max-open-files=1024
printf > pump.toml %s\\n 'log-file="pump.log"' 'data-dir="pump.data"' 'addr="127.0.0.1:8250"' 'advertise-addr="127.0.0.1:8250"' 'pd-urls="http://127.0.0.1:2379"'
printf > tidb.toml %s\\n 'store="tikv"' 'path="127.0.0.1:2379"' '[log.file]' 'filename="tidb.log"' '[binlog]' 'enable=true'
printf > drainer.toml %s\\n 'log-file="drainer.log"' '[syncer]' 'db-type="mysql"' '[syncer.to]' 'host="127.0.0.1"' 'user="root"' 'password=""' 'port=3306'
```

以下のコマンドを使用して、構成の詳細を確認してください:

```bash
for f in *.toml; do echo "$f:"; cat "$f"; echo; done
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    drainer.toml:
    log-file="drainer.log"
    [syncer]
    db-type="mysql"
    [syncer.to]
    host="127.0.0.1"
    user="root"
    password=""
    port=3306

    pd.toml:
    log-file="pd.log"
    data-dir="pd.data"

    pump.toml:
    log-file="pump.log"
    data-dir="pump.data"
    addr="127.0.0.1:8250"
    advertise-addr="127.0.0.1:8250"
    pd-urls="http://127.0.0.1:2379"

    tidb.toml:
    store="tikv"
    path="127.0.0.1:2379"
    [log.file]
    filename="tidb.log"
    [binlog]
    enable=true

    tikv.toml:
    log-file="tikv.log"
    [storage]
    data-dir="tikv.data"
    [pd]
    endpoints=["127.0.0.1:2379"]
    [rocksdb]
    max-open-files=1024
    [raftdb]
    max-open-files=1024

## ブートストラップ {#bootstrapping}

各コンポーネントを起動することができます。特定の順序で行うのが最適です - まずPlacement Driver（PD）、次にTiKV Server、次にPump（TiDBはバイナリログを送信するためにPumpサービスに接続する必要があるため）、最後にTiDB Serverを起動します。

次のコマンドを使用してすべてのサービスを起動します：

```bash
./bin/pd-server --config=pd.toml &>pd.out &
./bin/tikv-server --config=tikv.toml &>tikv.out &
./pump --config=pump.toml &>pump.out &
sleep 3
./bin/tidb-server --config=tidb.toml &>tidb.out &
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    [kolbe@localhost tidb-latest-linux-amd64]$ ./bin/pd-server --config=pd.toml &>pd.out &
    [1] 20935
    [kolbe@localhost tidb-latest-linux-amd64]$ ./bin/tikv-server --config=tikv.toml &>tikv.out &
    [2] 20944
    [kolbe@localhost tidb-latest-linux-amd64]$ ./pump --config=pump.toml &>pump.out &
    [3] 21050
    [kolbe@localhost tidb-latest-linux-amd64]$ sleep 3
    [kolbe@localhost tidb-latest-linux-amd64]$ ./bin/tidb-server --config=tidb.toml &>tidb.out &
    [4] 21058

`jobs`を実行すると、実行中のデーモンのリストが表示されます。

    [kolbe@localhost tidb-latest-linux-amd64]$ jobs
    [1]   Running                 ./bin/pd-server --config=pd.toml &>pd.out &
    [2]   Running                 ./bin/tikv-server --config=tikv.toml &>tikv.out &
    [3]-  Running                 ./pump --config=pump.toml &>pump.out &
    [4]+  Running                 ./bin/tidb-server --config=tidb.toml &>tidb.out &

もしサービスの1つが起動に失敗した場合（例えば、"`Exit 1`"ではなく"`Running`"が表示される場合）、個々のサービスを再起動してみてください。

## 接続 {#connecting}

TiDBクラスターの4つのコンポーネントがすべて起動しているはずで、MariaDB/MySQLのコマンドラインクライアントを使用してポート4000でTiDBサーバーに接続できます。

```bash
mysql -h 127.0.0.1 -P 4000 -u root -e 'select tidb_version()\G'
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    [kolbe@localhost tidb-latest-linux-amd64]$ mysql -h 127.0.0.1 -P 4000 -u root -e 'select tidb_version()\G'
    *************************** 1. row ***************************
    tidb_version(): Release Version: v3.0.0-beta.1-154-gd5afff70c
    Git Commit Hash: d5afff70cdd825d5fab125c8e52e686cc5fb9a6e
    Git Branch: master
    UTC Build Time: 2019-04-24 03:10:00
    GoVersion: go version go1.12 linux/amd64
    Race Enabled: false
    TiKV Min Version: 2.1.0-alpha.1-ff3dd160846b7d1aed9079c389fc188f7f5ea13e
    Check Table Before Drop: false

この時点では、TiDBクラスターが実行されており、`pump`がクラスターからバイナリログを読み取り、そのデータディレクトリにリレーログとして保存しています。次のステップは、`drainer`が書き込めるようにMariaDBサーバーを起動することです。

次のコマンドを使用して`drainer`を起動します：

```bash
sudo systemctl start mariadb
./drainer --config=drainer.toml &>drainer.out &
```

もしMySQLサーバーをインストールしやすくするためのオペレーティングシステムを使用している場合は、それでも大丈夫です。ただし、ポート3306でリスニングしていることを確認し、ユーザー「root」で空のパスワードで接続できるか、または必要に応じてdrainer.tomlを調整してください。

```bash
mysql -h 127.0.0.1 -P 3306 -u root
```

```sql
show databases;
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    [kolbe@localhost ~]$ mysql -h 127.0.0.1 -P 3306 -u root
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 20
    Server version: 5.5.60-MariaDB MariaDB Server

    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    MariaDB [(none)]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | test               |
    | tidb_binlog        |
    +--------------------+
    5 rows in set (0.01 sec)

ここでは、`tidb_binlog`データベースが既に見えます。このデータベースには、`drainer`が使用する`checkpoint`テーブルが含まれており、TiDBクラスタからどのバイナリログが適用されたかを記録しています。

```sql
MariaDB [tidb_binlog]> use tidb_binlog;
Database changed
MariaDB [tidb_binlog]> select * from checkpoint;
+---------------------+---------------------------------------------+
| clusterID           | checkPoint                                  |
+---------------------+---------------------------------------------+
| 6678715361817107733 | {"commitTS":407637466476445697,"ts-map":{}} |
+---------------------+---------------------------------------------+
1 row in set (0.00 sec)
```

さて、TiDBサーバーにもう一つのクライアント接続を開き、テーブルを作成し、いくつかの行を挿入しましょう。 (同時に複数のクライアントを開いておくために、GNU screenの下でこれを行うことをお勧めします。)

```bash
mysql -h 127.0.0.1 -P 4000 --prompt='TiDB [\d]> ' -u root
```

```sql
create database tidbtest;
use tidbtest;
create table t1 (id int unsigned not null AUTO_INCREMENT primary key);
insert into t1 () values (),(),(),(),();
select * from t1;
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    TiDB [(none)]> create database tidbtest;
    Query OK, 0 rows affected (0.12 sec)

    TiDB [(none)]> use tidbtest;
    Database changed
    TiDB [tidbtest]> create table t1 (id int unsigned not null AUTO_INCREMENT primary key);
    Query OK, 0 rows affected (0.11 sec)

    TiDB [tidbtest]> insert into t1 () values (),(),(),(),();
    Query OK, 5 rows affected (0.01 sec)
    Records: 5  Duplicates: 0  Warnings: 0

    TiDB [tidbtest]> select * from t1;
    +----+
    | id |
    +----+
    |  1 |
    |  2 |
    |  3 |
    |  4 |
    |  5 |
    +----+
    5 rows in set (0.00 sec)

MariaDBクライアントに切り替えると、新しいデータベース、新しいテーブル、および新しく挿入された行が見つかるはずです。

```sql
use tidbtest;
show tables;
select * from t1;
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    MariaDB [(none)]> use tidbtest;
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A

    Database changed
    MariaDB [tidbtest]> show tables;
    +--------------------+
    | Tables_in_tidbtest |
    +--------------------+
    | t1                 |
    +--------------------+
    1 row in set (0.00 sec)

    MariaDB [tidbtest]> select * from t1;
    +----+
    | id |
    +----+
    |  1 |
    |  2 |
    |  3 |
    |  4 |
    |  5 |
    +----+
    5 rows in set (0.00 sec)

MariaDBサーバーをクエリすると、TiDBに挿入した同じ行が表示されるはずです。おめでとうございます！TiDB Binlogを設定しました！

## binlogctl {#binlogctl}

クラスタに参加したPumpとDrainerの情報は、PDに保存されます。binlogctlツールを使用して、それらの状態に関する情報をクエリや操作することができます。詳細は[binlogctlガイド](/tidb-binlog/binlog-control.md)を参照してください。

`binlogctl`を使用して、クラスタ内のPumpとDrainerの現在の状態を確認します：

```bash
./binlogctl -cmd drainers
./binlogctl -cmd pumps
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    [kolbe@localhost tidb-latest-linux-amd64]$ ./binlogctl -cmd drainers
    [2019/04/11 17:44:10.861 -04:00] [INFO] [nodes.go:47] ["query node"] [type=drainer] [node="{NodeID: localhost.localdomain:8249, Addr: 192.168.236.128:8249, State: online, MaxCommitTS: 407638907719778305, UpdateTime: 2019-04-11 17:44:10 -0400 EDT}"]

    [kolbe@localhost tidb-latest-linux-amd64]$ ./binlogctl -cmd pumps
    [2019/04/11 17:44:13.904 -04:00] [INFO] [nodes.go:47] ["query node"] [type=pump] [node="{NodeID: localhost.localdomain:8250, Addr: 192.168.236.128:8250, State: online, MaxCommitTS: 407638914024079361, UpdateTime: 2019-04-11 17:44:13 -0400 EDT}"]

Drainerを殺すと、クラスターはそれを「一時停止」状態にします。これは、クラスターがそれが再参加することを期待していることを意味します。

```bash
pkill drainer
./binlogctl -cmd drainers
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    [kolbe@localhost tidb-latest-linux-amd64]$ pkill drainer
    [kolbe@localhost tidb-latest-linux-amd64]$ ./binlogctl -cmd drainers
    [2019/04/11 17:44:22.640 -04:00] [INFO] [nodes.go:47] ["query node"] [type=drainer] [node="{NodeID: localhost.localdomain:8249, Addr: 192.168.236.128:8249, State: paused, MaxCommitTS: 407638915597467649, UpdateTime: 2019-04-11 17:44:18 -0400 EDT}"]

`binlogctl`を使用して、個々のノードを制御することができます。この場合、DrainerのNodeIDは「localhost.localdomain:8249」であり、PumpのNodeIDは「localhost.localdomain:8250」です。

このチュートリアルでの`binlogctl`の主な使用方法は、クラスターの再起動時に行われる可能性が高いです。TiDBクラスターのすべてのプロセスを終了し、再起動しようとすると（下流のMySQL/MariaDBサーバーまたはDrainerを除く）、PumpはDrainerに連絡できないため起動を拒否し、Drainerがまだ「オンライン」であると考えます。

この問題には3つの解決策があります：

- プロセスを殺す代わりに、`binlogctl`を使用してDrainerを停止する：

      ./binlogctl --pd-urls=http://127.0.0.1:2379 --cmd=drainers
      ./binlogctl --pd-urls=http://127.0.0.1:2379 --cmd=offline-drainer --node-id=localhost.localdomain:8249

- Pumpを起動する*前に*Drainerを起動する。

- PDを起動した後（DrainerとPumpを起動する前）に、一時停止されたDrainerの状態を更新するために`binlogctl`を使用する：

      ./binlogctl --pd-urls=http://127.0.0.1:2379 --cmd=update-drainer --node-id=localhost.localdomain:8249 --state=offline

## クリーンアップ {#cleanup}

TiDBクラスターとTiDB Binlogプロセスを停止するには、クラスターを構成するすべてのプロセス（pd-server、tikv-server、pump、tidb-server、drainer）を起動したシェルで`pkill -P $$`を実行することができます。各コンポーネントがきれいにシャットダウンするために十分な時間を与えるために、特定の順序で停止することが役立ちます：

```bash
for p in tidb-server drainer pump tikv-server pd-server; do pkill "$p"; sleep 1; done
```

ここには、翻訳してはいけないK-Vマップがあります。コンテンツ内のキーを翻訳せず、値で置き換える必要があります：
{"Binlog":"Binlog","component":"コンポーネント","Driver":"Driver","Pump":"Pump","Drainer":"Drainer","Architecture":"アーキテクチャ","architecture":"アーキテクチャ","production":"本番","I":"I","Y":"Y","Cluster":"クラスタ","binlog":"binlog","server":"サーバー","Configuration":"コンフィグレーション","storage":"storage","drainer":"drainer"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

    kolbe@localhost tidb-latest-linux-amd64]$ for p in tidb-server drainer pump tikv-server pd-server; do pkill "$p"; sleep 1; done
    [4]-  Done                    ./bin/tidb-server --config=tidb.toml &>tidb.out
    [5]+  Done                    ./drainer --config=drainer.toml &>drainer.out
    [3]+  Done                    ./pump --config=pump.toml &>pump.out
    [2]+  Done                    ./bin/tikv-server --config=tikv.toml &>tikv.out
    [1]+  Done                    ./bin/pd-server --config=pd.toml &>pd.out

すべてのサービスが終了した後にクラスターを再起動する場合は、最初に実行したコマンドと同じものを使用してサービスを起動してください。[`binlogctl`](#binlogctl)セクションで説明したように、`drainer`を`pump`の前に、`pump`を`tidb-server`の前に起動する必要があります。

```bash
./bin/pd-server --config=pd.toml &>pd.out &
./bin/tikv-server --config=tikv.toml &>tikv.out &
./drainer --config=drainer.toml &>drainer.out &
sleep 3
./pump --config=pump.toml &>pump.out &
sleep 3
./bin/tidb-server --config=tidb.toml &>tidb.out &
```

コンポーネントのいずれかが起動しない場合は、失敗した個々のコンポーネントを再起動してみてください。

## 結論 {#conclusion}

このチュートリアルでは、単一のPumpと単一のDrainerを持つクラスタを使用して、TiDBクラスタから下流のMariaDBサーバーにレプリケーションするために、TiDB Binlogをセットアップしました。TiDB Binlogは、TiDBクラスタの変更をキャプチャして処理するための包括的なプラットフォームであることがわかりました。

より堅牢な開発、テスト、または本番展開では、高可用性とスケーリングの目的で複数のTiDBサーバーを使用し、アプリケーションのトラフィックがPumpクラスターの問題に影響されないように、複数のPumpインスタンスを使用します。また、さまざまな下流プラットフォームに更新をプッシュするために、または増分バックアップを実装するために、追加のDrainerインスタンスを使用することもできます。
