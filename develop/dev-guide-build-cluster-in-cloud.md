---
title: Build a TiDB Serverless Cluster
summary: Learn how to build a TiDB Serverless cluster in TiDB Cloud and connect to it.
---

<!-- markdownlint-disable MD029 -->

# TiDB Serverlessクラスタの構築 {#build-a-tidb-serverless-cluster}

<CustomContent platform="tidb">

このドキュメントでは、TiDBの最も簡単な始め方について説明します。[TiDB Cloud](https://en.pingcap.com/tidb-cloud)を使用して、TiDB Serverlessクラスタを作成し、それに接続し、サンプルアプリケーションを実行します。

ローカルマシンでTiDBを実行する必要がある場合は、[ローカルでTiDBを起動する](/quick-start-with-tidb.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

このドキュメントでは、TiDB Cloudの最も簡単な始め方について説明します。TiDBクラスタを作成し、それに接続し、サンプルアプリケーションを実行します。

</CustomContent>

## ステップ1. TiDB Serverlessクラスタを作成する {#step-1-create-a-tidb-serverless-cluster}

1. TiDB Cloudアカウントを持っていない場合は、[こちら](https://tidbcloud.com/free-trial)をクリックしてアカウントを作成してください。

2. [こちら](https://tidbcloud.com/)をクリックしてTiDB Cloudアカウントにログインします。

3. [**クラスタ**](https://tidbcloud.com/console/clusters)ページで、**クラスタを作成**をクリックします。

4. **クラスタを作成**ページで、**サーバーレス**がデフォルトで選択されています。必要に応じてデフォルトのクラスタ名を更新し、その後、クラスタを作成したいリージョンを選択します。

5. TiDB Serverlessクラスタを作成するには、**作成**をクリックします。

   TiDB Cloudクラスタは約30秒で作成されます。

6. TiDB Cloudクラスタが作成されたら、クラスタの概要ページに移動し、右上隅にある**接続**をクリックします。接続ダイアログボックスが表示されます。

7. ダイアログで、希望の接続方法とオペレーティングシステムを選択して、対応する接続文字列を取得します。このドキュメントではMySQLクライアントを例にしています。

8. ランダムなパスワードを生成するには、**パスワードを生成**をクリックします。生成されたパスワードは再表示されないため、パスワードを安全な場所に保存してください。rootパスワードを設定しない場合、クラスタに接続できません。

<CustomContent platform="tidb">

> **Note:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタに接続する場合、ユーザー名にクラスタのプレフィックスを含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名のプレフィックス](https://docs.pingcap.com/tidbcloud/select-cluster-tier#user-name-prefix)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタに接続する場合、ユーザー名にクラスタのプレフィックスを含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名のプレフィックス](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

</CustomContent>

## ステップ2. クラスタに接続する {#step-2-connect-to-a-cluster}

1. MySQLクライアントがインストールされていない場合は、オペレーティングシステムを選択し、以下の手順に従ってインストールしてください。

<SimpleTab>

<div label="macOS">

macOSの場合、[Homebrew](https://brew.sh/index)をインストールしていない場合は、次のコマンドを実行してMySQLクライアントをインストールします:

```shell
brew install mysql-client
```

出力は次のとおりです:

```
mysql-client is keg-only, which means it was not symlinked into /opt/homebrew,
because it conflicts with mysql (which contains client libraries).

If you need to have mysql-client first in your PATH, run:
  echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> ~/.zshrc

For compilers to find mysql-client you may need to set:
  export LDFLAGS="-L/opt/homebrew/opt/mysql-client/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/mysql-client/include"
```

MySQLクライアントをPATHに追加するには、上記の出力で次のコマンドを見つけて実行してください（出力がドキュメントの上記の出力と一致しない場合は、代わりに出力の対応するコマンドを使用してください）:

```shell
echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> ~/.zshrc
```

その後、`source`コマンドでグローバル環境変数を宣言し、MySQLクライアントが正常にインストールされたことを確認してください:

```shell
source ~/.zshrc
mysql --version
```

期待される出力の例:

```
mysql  Ver 8.0.28 for macos12.0 on arm64 (Homebrew)
```

</div>

<div label="Linux">

Linuxの場合、以下はCentOS 7を例にしています:

```shell
yum install mysql
```

その後、MySQLクライアントが正常にインストールされたことを確認してください:

```shell
mysql --version
```

期待される出力の例:

```
mysql  Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1
```

</div>

</SimpleTab>

2. [ステップ1](#step-1-create-a-tidb-serverless-cluster)で取得した接続文字列を実行してください。

   ```shell
   mysql --connect-timeout 15 -u '<prefix>.root' -h <host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p
   ```

<CustomContent platform="tidb">

> **Note:**
>
> - TiDB Serverlessクラスタに接続する際は、[TLS接続を使用する必要があります](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)。
> - TiDB Serverlessクラスタに接続する際に問題が発生した場合は、[TiDB Serverlessクラスタへの安全な接続](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> - TiDB Serverlessクラスタに接続する際は、[TLS接続を使用する必要があります](/tidb-cloud/secure-connections-to-serverless-clusters.md)。
> - TiDB Serverlessクラスタに接続する際に問題が発生した場合は、[TiDB Serverlessクラスタへの安全な接続](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。

</CustomContent>

3. パスワードを入力してサインインしてください。

## ステップ3. SQLステートメントを実行する {#step-3-execute-a-sql-statement}

TiDB Cloudで最初のSQLステートメントを実行してみましょう。

```sql
SELECT 'Hello TiDB Cloud!';
```

期待される出力:

```sql
+-------------------+
| Hello TiDB Cloud! |
+-------------------+
| Hello TiDB Cloud! |
+-------------------+
```

実際の出力が期待される出力に似ている場合、おめでとうございます。TiDB CloudでのSQLステートメントの実行に成功しました。"
