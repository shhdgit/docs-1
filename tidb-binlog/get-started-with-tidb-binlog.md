---
title: TiDB Binlog Tutorial
summary: Learn to deploy TiDB Binlog with a simple TiDB cluster.
---

# TiDB Binlog チュートリアル {#tidb-binlog-tutorial}

このチュートリアルは、各コンポーネント（配置ドライバー、TiKVサーバー、TiDBサーバー、Pump、およびDrainer）の単一ノードを使用して、MariaDBサーバーインスタンスにデータをプッシュするように設定されたシンプルなTiDB Binlogデプロイメントから始まります。

このチュートリアルは、[TiDBアーキテクチャ](/tidb-architecture.md)についてある程度の理解があり、TiDBクラスターをすでにセットアップしている（必須ではありません）ユーザーを対象としており、TiDB Binlogの実践的な経験を積みたいユーザーを対象としています。このチュートリアルは、TiDB Binlogの概念に慣れるための良い方法であり、TiDB Binlogの機能を試すためのものです。

> **警告:**
>
> このチュートリアルでTiDBをデプロイする手順は、本番環境や開発環境には使用しないでください。

このチュートリアルでは、x86-64のモダンなLinuxディストリビューションを使用していることを前提としています。このチュートリアルの例では、VMwareで実行されている最小限のCentOS 7インストールが使用されています。既存の環境の問題に影響を受けないようにするために、クリーンインストールから始めることをお勧めします。ローカル仮想化を使用したくない場合は、クラウドサービスを使用して簡単にCentOS 7 VMを起動できます。

## TiDB Binlog概要 {#tidb-binlog-overview}

TiDB Binlogは、TiDBからバイナリログデータを収集し、リアルタイムのデータバックアップとレプリケーションを提供するソリューションです。これにより、TiDBサーバークラスターからの増分データ更新が下流プラットフォームにプッシュされます。

TiDB Binlogを使用すると、増分バックアップに加えて、TiDBクラスターから別のTiDBクラスターにデータをレプリケートしたり、TiDBの更新をKafkaを介して選択した下流プラットフォームに送信したりすることができます。

TiDB Binlogは、MySQLやMariaDBからTiDBにデータを移行する際に特に有用です。この場合、TiDB DM（データ移行）プラットフォームを使用してMySQL/MariaDBクラスターからデータをTiDBに取得し、TiDB Binlogを使用してアプリケーショントラフィックをTiDBから下流のMySQL/MariaDBインスタンス/クラスターにプッシュすることができます。これにより、TiDBへの移行のリスクが低減されます。なぜなら、MySQLやMariaDBへのアプリケーションの簡単な切り替えが可能であり、ダウンタイムやデータ損失なしにMySQLやMariaDBにアプリケーショントラフィックを簡単に戻すことができるからです。

詳細については、[TiDB Binlogクラスターユーザーガイド](/tidb-binlog/tidb-binlog-overview.md)を参照してください。

## アーキテクチャ {#architecture}

TiDB Binlogには、**Pump**と**Drainer**の2つのコンポーネントが含まれています。複数のPumpノードがPumpクラスターを構成します。各PumpノードはTiDBサーバーインスタンスに接続し、クラスター内の各TiDBサーバーインスタンスで行われた更新を受信します。DrainerはPumpクラスターに接続し、受信した更新を特定の下流先（たとえば、Kafka、別のTiDBクラスター、またはMySQL/MariaDBサーバー）の正しい形式に変換します。

![TiDB-Binlog architecture](/media/tidb-binlog-cluster-architecture.png)

Pumpのクラスター化されたアーキテクチャにより、新しいTiDBサーバーインスタンスがTiDBクラスターに参加したり離れたりする場合でも、またはPumpノードがPumpクラスターに参加したり離れたりする場合でも、更新が失われることはありません。

## インストール {#installation}

この場合、MySQL Serverの代わりにMariaDB Serverを使用しています。なぜなら、RHEL/CentOS 7にはMariaDB Serverがデフォルトのパッケージリポジトリに含まれているからです。後で使用するために、クライアントとサーバーの両方が必要です。それでは、それらをインストールしましょう：

```bash
sudo yum install -y mariadb-server
```

```bash
curl -L https://download.pingcap.org/tidb-community-server-v7.1.3-linux-amd64.tar.gz | tar xzf -
cd tidb-latest-linux-amd64
```

期待される出力：

```
[kolbe@localhost ~]$ curl -LO https://download.pingcap.org/tidb-latest-linux-amd64.tar.gz | tar xzf -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  368M  100  368M    0     0  8394k      0  0:00:44  0:00:44 --:--:-- 11.1M
[kolbe@localhost ~]$ cd tidb-latest-linux-amd64
[kolbe@localhost tidb-latest-linux-amd64]$
```

## コンフィグレーション {#configuration}

Now we'll start a simple TiDB クラスタ, with a single instance for each of `pd-サーバー`, `tikv-サーバー`, and `tidb-サーバー`.

Populate the config files using:

```bash
printf > pd.toml %s\\n 'log-file="pd.log"' 'data-dir="pd.data"'
printf > tikv.toml %s\\n 'log-file="tikv.log"' '[storage]' 'data-dir="tikv.data"' '[pd]' 'endpoints=["127.0.0.1:2379"]' '[rocksdb]' max-open-files=1024 '[raftdb]' max-open-files=1024
printf > pump.toml %s\\n 'log-file="pump.log"' 'data-dir="pump.data"' 'addr="127.0.0.1:8250"' 'advertise-addr="127.0.0.1:8250"' 'pd-urls="http://127.0.0.1:2379"'
printf > tidb.toml %s\\n 'store="tikv"' 'path="127.0.0.1:2379"' '[log.file]' 'filename="tidb.log"' '[binlog]' 'enable=true'
printf > drainer.toml %s\\n 'log-file="drainer.log"' '[syncer]' 'db-type="mysql"' '[syncer.to]' 'host="127.0.0.1"' 'user="root"' 'password=""' 'port=3306'
```

以下のコマンドを使用して、構成の詳細を表示します:

```bash
for f in *.toml; do echo "$f:"; cat "$f"; echo; done
```

期待される出力：

```
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
```

## ブートストラップ {#bootstrapping}

これで各コンポーネントを開始できます。これは特定の順序で行うのが最良です - まずPlacement Driver（PD）、次にTiKV Server、次にPump（なぜならTiDBはバイナリログを送信するためにPumpサービスに接続する必要があるからです）、最後にTiDB Serverです。

次のコマンドを使用してすべてのサービスを開始します：

```bash
./bin/pd-server --config=pd.toml &>pd.out &
./bin/tikv-server --config=tikv.toml &>tikv.out &
./pump --config=pump.toml &>pump.out &
sleep 3
./bin/tidb-server --config=tidb.toml &>tidb.out &
```

期待される出力：

```
[kolbe@localhost tidb-latest-linux-amd64]$ ./bin/pd-server --config=pd.toml &>pd.out &
[1] 20935
[kolbe@localhost tidb-latest-linux-amd64]$ ./bin/tikv-server --config=tikv.toml &>tikv.out &
[2] 20944
[kolbe@localhost tidb-latest-linux-amd64]$ ./pump --config=pump.toml &>pump.out &
[3] 21050
[kolbe@localhost tidb-latest-linux-amd64]$ sleep 3
[kolbe@localhost tidb-latest-linux-amd64]$ ./bin/tidb-server --config=tidb.toml &>tidb.out &
[4] 21058
```

もし `jobs` を実行すると、実行中のデーモンのリストが表示されるはずです。

```
[kolbe@localhost tidb-latest-linux-amd64]$ jobs
[1]   Running                 ./bin/pd-server --config=pd.toml &>pd.out &
[2]   Running                 ./bin/tikv-server --config=tikv.toml &>tikv.out &
[3]-  Running                 ./pump --config=pump.toml &>pump.out &
[4]+  Running                 ./bin/tidb-server --config=tidb.toml &>tidb.out &
```

もしサービスの1つが起動に失敗した場合（たとえば、「実行中」の代わりに「Exit 1」と表示される場合）、個々のサービスを再起動してみてください。

## Connecting {#connecting}

TiDBクラスターの4つのコンポーネントがすべて実行されているはずです。MariaDB/MySQLコマンドラインクライアントを使用して、ポート4000でTiDBサーバーに接続できるようになりました。

```bash
mysql -h 127.0.0.1 -P 4000 -u root -e 'select tidb_version()\G'
```

期待される出力：

```
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
```

この時点で、TiDBクラスターが実行されており、`pump`がクラスターからバイナリログを読み取り、そのデータディレクトリにリレーログとして保存しています。次のステップは、`drainer`が書き込むことができるMariaDBサーバーを起動することです。

次のコマンドを使用して`drainer`を起動します：

```bash
sudo systemctl start mariadb
./drainer --config=drainer.toml &>drainer.out &
```

もしMySQLサーバーを簡単にインストールできるオペレーティングシステムを使用している場合は、それも問題ありません。ただし、ポート3306でリスニングしていることを確認し、ユーザー"root"と空のパスワードで接続できるか、または必要に応じてdrainer.tomlを調整できることを確認してください。

```bash
mysql -h 127.0.0.1 -P 3306 -u root
```

```sql
show databases;
```

期待される出力：

```
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
```

ここでは、`tidb_binlog`データベースがすでに見えます。これには、`drainer`がTiDBクラスターからのバイナリログが適用されたポイントまで記録するために使用する`checkpoint`テーブルが含まれています。

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

さて、別のクライアント接続をTiDBサーバーに開きましょう。これにより、テーブルを作成し、いくつかの行を挿入することができます。（同時に複数のクライアントを開いておくために、GNUスクリーンの下で行うことをお勧めします。）

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

期待される出力：

```
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
```

MariaDBクライアントに切り替えると、新しいデータベース、新しいテーブル、および新しく挿入された行が見つかるはずです。

```sql
use tidbtest;
show tables;
select * from t1;
```

期待される出力：

```
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
```

TiDBサーバーをクエリすると、TiDBに挿入した同じ行が表示されるはずです。おめでとうございます！あなたはTiDB Binlogを設定しました！

## binlogctl {#binlogctl}

クラスタに参加したPumpsとDrainersに関する情報はPDに保存されています。binlogctlツールを使用して、それらの状態に関する情報をクエリおよび操作できます。詳細については、[binlogctlガイド](/tidb-binlog/binlog-control.md)を参照してください。

`binlogctl`を使用して、クラスタ内のPumpsとDrainersの現在の状態を表示します。

```bash
./binlogctl -cmd drainers
./binlogctl -cmd pumps
```

期待される出力：

```
[kolbe@localhost tidb-latest-linux-amd64]$ ./binlogctl -cmd drainers
[2019/04/11 17:44:10.861 -04:00] [INFO] [nodes.go:47] ["query node"] [type=drainer] [node="{NodeID: localhost.localdomain:8249, Addr: 192.168.236.128:8249, State: online, MaxCommitTS: 407638907719778305, UpdateTime: 2019-04-11 17:44:10 -0400 EDT}"]

[kolbe@localhost tidb-latest-linux-amd64]$ ./binlogctl -cmd pumps
[2019/04/11 17:44:13.904 -04:00] [INFO] [nodes.go:47] ["query node"] [type=pump] [node="{NodeID: localhost.localdomain:8250, Addr: 192.168.236.128:8250, State: online, MaxCommitTS: 407638914024079361, UpdateTime: 2019-04-11 17:44:13 -0400 EDT}"]
```

Drainerを停止すると、クラスタはそれを「一時停止」状態にし、クラスタはそれが再参加することを期待しています。

```bash
pkill drainer
./binlogctl -cmd drainers
```

期待される出力：

```
[kolbe@localhost tidb-latest-linux-amd64]$ pkill drainer
[kolbe@localhost tidb-latest-linux-amd64]$ ./binlogctl -cmd drainers
[2019/04/11 17:44:22.640 -04:00] [INFO] [nodes.go:47] ["query node"] [type=drainer] [node="{NodeID: localhost.localdomain:8249, Addr: 192.168.236.128:8249, State: paused, MaxCommitTS: 407638915597467649, UpdateTime: 2019-04-11 17:44:18 -0400 EDT}"]
```

"binlogctl"を使用して個々のノードを制御することができます。この場合、drainerのNodeIDは "localhost.localdomain:8249" であり、PumpのNodeIDは "localhost.localdomain:8250" です。

このチュートリアルでの "binlogctl" の主な使用目的は、おそらくクラスタの再起動の際でしょう。TiDBクラスタのすべてのプロセスを終了し、それらを再起動しようとすると（下流のMySQL/MariaDBサーバーまたはDrainerを除く）、PumpはDrainerに連絡できないため起動を拒否し、Drainerがまだ "オンライン" であると信じます。

この問題には3つの解決策があります：

- プロセスを終了させる代わりに、`binlogctl`を使用してDrainerを停止する：

  ```
  ./binlogctl --pd-urls=http://127.0.0.1:2379 --cmd=drainers
  ./binlogctl --pd-urls=http://127.0.0.1:2379 --cmd=offline-drainer --node-id=localhost.localdomain:8249
  ```

- Pumpを開始する*前に*Drainerを開始します。

- DrainerとPumpを開始する*前に*PDを起動した後、`binlogctl`を使用して一時停止したDrainerの状態を更新します：

  ```
  ./binlogctl --pd-urls=http://127.0.0.1:2379 --cmd=update-drainer --node-id=localhost.localdomain:8249 --state=offline
  ```

## クリーンアップ {#cleanup}

TiDBクラスタとTiDB Binlogプロセスを停止するには、クラスタを構成するすべてのプロセス（pd-server、tikv-server、pump、tidb-server、drainer）を開始したシェルで `pkill -P $$` を実行できます。各コンポーネントがきちんとシャットダウンするために、特定の順序で停止することが役立ちます。

```bash
for p in tidb-server drainer pump tikv-server pd-server; do pkill "$p"; sleep 1; done
```

期待される出力：

```
kolbe@localhost tidb-latest-linux-amd64]$ for p in tidb-server drainer pump tikv-server pd-server; do pkill "$p"; sleep 1; done
[4]-  Done                    ./bin/tidb-server --config=tidb.toml &>tidb.out
[5]+  Done                    ./drainer --config=drainer.toml &>drainer.out
[3]+  Done                    ./pump --config=pump.toml &>pump.out
[2]+  Done                    ./bin/tikv-server --config=tikv.toml &>tikv.out
[1]+  Done                    ./bin/pd-server --config=pd.toml &>pd.out
```

本番のサービスがすべて終了した後にクラスタを再起動する場合は、最初にサービスを起動したときと同じコマンドを使用してください。前述の[`binlogctl`](#binlogctl)セクションで議論されているように、`pump`の前に`drainer`を、そして`tidb-server`の前に`pump`を起動する必要があります。

```bash
./bin/pd-server --config=pd.toml &>pd.out &
./bin/tikv-server --config=tikv.toml &>tikv.out &
./drainer --config=drainer.toml &>drainer.out &
sleep 3
./pump --config=pump.toml &>pump.out &
sleep 3
./bin/tidb-server --config=tidb.toml &>tidb.out &
```

コンポーネントのいずれかが起動に失敗した場合は、失敗した個々のコンポーネントを再起動してみてください。

## Conclusion {#conclusion}

このチュートリアルでは、TiDB Binlogをセットアップして、TiDBクラスターからダウンストリームのMariaDBサーバーにレプリケートする方法を説明しました。この際、単一のPumpと単一のDrainerを備えたクラスターを使用しました。TiDB Binlogは、TiDBクラスターへの変更のキャプチャと処理のための包括的なプラットフォームであることがわかります。

より堅牢な開発、テスト、または本番展開では、高可用性とスケーリングを目的として複数のTiDBサーバーを使用し、Pumpクラスター内の問題がTiDBサーバーインスタンスへのアプリケーショントラフィックに影響を与えないようにするために、複数のPumpインスタンスを使用します。また、追加のDrainerインスタンスを使用して、異なるダウンストリームプラットフォームに更新をプッシュしたり、増分バックアップを実装したりすることもあります。
