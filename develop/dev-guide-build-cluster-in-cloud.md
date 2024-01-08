---
title: Build a TiDB Serverless Cluster
summary: Learn how to build a TiDB Serverless cluster in TiDB Cloud and connect to it.
---

<!-- markdownlint-disable MD029 -->

# TiDB Serverless クラスタの構築 {#build-a-tidb-serverless-cluster}

<CustomContent platform="tidb">

このドキュメントでは、TiDB の最も簡単な始め方について説明します。[TiDB Cloud](https://en.pingcap.com/tidb-cloud) を使用して、TiDB Serverless クラスタを作成し、それに接続し、サンプルアプリケーションを実行します。

ローカルマシンで TiDB を実行する必要がある場合は、[Starting TiDB Locally](/quick-start-with-tidb.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

このドキュメントでは、TiDB Cloud の最も簡単な始め方について説明します。TiDB クラスタを作成し、それに接続し、サンプルアプリケーションを実行します。

</CustomContent>

## ステップ 1. TiDB Serverless クラスタを作成する {#step-1-create-a-tidb-serverless-cluster}

1. TiDB Cloud アカウントをお持ちでない場合は、[こちら](https://tidbcloud.com/free-trial) をクリックしてアカウントを作成してください。

2. TiDB Cloud アカウントに[ログイン](https://tidbcloud.com/)します。

3. [**Clusters**](https://tidbcloud.com/console/clusters) ページで、**Create Cluster** をクリックします。

4. **Create Cluster** ページで、**Serverless** がデフォルトで選択されています。必要に応じてデフォルトのクラスタ名を更新し、その後、クラスタを作成するリージョンを選択します。

5. TiDB Serverless クラスタを作成するには、**Create** をクリックします。

   TiDB Cloud クラスタは約 30 秒で作成されます。

6. TiDB Cloud クラスタが作成されたら、クラスタの概要ページに移動し、右上隅の **Connect** をクリックします。接続ダイアログボックスが表示されます。

7. ダイアログで、希望の接続方法とオペレーティングシステムを選択して、対応する接続文字列を取得します。このドキュメントでは MySQL クライアントを例にしています。

8. ランダムなパスワードを生成するには、**Generate Password** をクリックします。生成されたパスワードは再表示されないため、パスワードを安全な場所に保存してください。root パスワードを設定しない場合、クラスタに接続できません。

<CustomContent platform="tidb">

> **Note:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスタに接続する場合、ユーザー名にクラスタのプレフィックスを含め、名前を引用符で囲む必要があります。詳細については、[User name prefix](https://docs.pingcap.com/tidbcloud/select-cluster-tier#user-name-prefix) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスタに接続する場合、ユーザー名にクラスタのプレフィックスを含め、名前を引用符で囲む必要があります。詳細については、[User name prefix](/tidb-cloud/select-cluster-tier.md#user-name-prefix) を参照してください。

</CustomContent>

## ステップ 2. クラスタに接続する {#step-2-connect-to-a-cluster}

1. MySQL クライアントがインストールされていない場合は、オペレーティングシステムを選択し、以下の手順に従ってインストールしてください。

<SimpleTab>

<div label="macOS">

macOS の場合、[Homebrew](https://brew.sh/index) をインストールしていない場合は、次のコマンドを実行して MySQL クライアントをインストールします:

```shell
brew install mysql-client
```

出力は次のようになります:

```
mysql-client is keg-only, which means it was not symlinked into /opt/homebrew,
because it conflicts with mysql (which contains client libraries).

If you need to have mysql-client first in your PATH, run:
  echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> ~/.zshrc

For compilers to find mysql-client you may need to set:
  export LDFLAGS="-L/opt/homebrew/opt/mysql-client/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/mysql-client/include"
```

MySQL クライアントを PATH に追加するには、上記の出力で次のコマンドを見つけて実行してください（出力がこのドキュメントの上記の出力と一致しない場合は、代わりに出力に対応するコマンドを使用してください）:

```shell
echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> ~/.zshrc
```

その後、`source` コマンドでグローバル環境変数を宣言し、MySQL クライアントが正常にインストールされたことを確認してください:

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

Linux の場合、以下は CentOS 7 を例にしています:

```shell
yum install mysql
```

その後、MySQL クライアントが正常にインストールされたことを確認してください:

```shell
mysql --version
```

期待される出力の例:

```
mysql  Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1
```

</div>

</SimpleTab>

2. [ステップ 1](#step-1-create-a-tidb-serverless-cluster) で取得した接続文字列を実行してください。

   ```shell
   mysql --connect-timeout 15 -u '<prefix>.root' -h <host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p
   ```

<CustomContent platform="tidb">

> **Note:**
>
> - TiDB Serverless クラスタに接続する際は、[TLS 接続を使用する必要があります](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)。
> - TiDB Serverless クラスタに接続する際に問題が発生した場合は、[Secure Connections to TiDB Serverless Clusters](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> - TiDB Serverless クラスタに接続する際は、[TLS 接続を使用する必要があります](/tidb-cloud/secure-connections-to-serverless-clusters.md)。
> - TiDB Serverless クラスタに接続する際に問題が発生した場合は、[Secure Connections to TiDB Serverless Clusters](/tidb-cloud/secure-connections-to-serverless-clusters.md) を参照してください。

</CustomContent>

3. パスワードを入力してサインインしてください。

## ステップ 3. SQL ステートメントを実行する {#step-3-execute-a-sql-statement}

TiDB Cloud 上で最初の SQL ステートメントを実行してみましょう。

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

実際の出力が期待される出力に似ている場合、おめでとうございます。あなたは TiDB Cloud 上で SQL ステートメントを成功裏に実行しました。"
