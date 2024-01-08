---
title: Connect to TiDB Serverless via Public Endpoint
summary: Learn how to connect to your TiDB Serverless cluster via public endpoint.
---

# パブリックエンドポイントを介してTiDB Serverlessに接続 {#connect-to-tidb-serverless-via-public-endpoint}

このドキュメントでは、パブリックエンドポイントを介してTiDB Serverlessクラスタに接続する方法について説明します。パブリックエンドポイントを使用すると、ラップトップからSQLクライアントを使用してTiDB Serverlessクラスタに接続できます。

> **ヒント：**
>
> TiDB Dedicatedクラスタにパブリックエンドポイントを介して接続する方法については、[標準接続を介したTiDB Dedicatedに接続](/tidb-cloud/connect-via-standard-connection.md)を参照してください。

パブリックエンドポイントを介してTiDB Serverlessクラスタに接続するには、次の手順を実行します。

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、その概要ページに移動するためにターゲットクラスタの名前をクリックします。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. ダイアログで、エンドポイントタイプのデフォルト設定を`Public`のままにし、希望の接続方法とオペレーティングシステムを選択して、対応する接続文字列を取得します。

   > **Note:**
   >
   > - エンドポイントタイプを`Public`のままにすると、接続は標準TLS接続経由になります。詳細については、[TiDB ServerlessへのTLS接続](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。
   > - **エンドポイントタイプ**のドロップダウンリストで**Private**を選択すると、接続はプライベートエンドポイント経由になります。詳細については、[プライベートエンドポイントを介したTiDB Serverlessへの接続](/tidb-cloud/set-up-private-endpoint-connections-serverless.md)を参照してください。

4. TiDB Serverlessでは、クラスタに対して[ブランチ](/tidb-cloud/branch-overview.md)を作成できます。ブランチが作成されたら、**ブランチ**のドロップダウンリストからブランチに接続することができます。`main`はクラスタそのものを表します。

5. まだパスワードを設定していない場合は、**パスワードを生成**をクリックしてランダムなパスワードを生成します。生成されたパスワードは再表示されないため、安全な場所にパスワードを保存してください。

6. 接続文字列でクラスタに接続します。

   > **Note:**
   >
   > TiDB Serverlessクラスタに接続する際は、ユーザー名にクラスタの接頭辞を含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名の接頭辞](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

## 次は何ですか {#what-s-next}

TiDBクラスタに正常に接続した後、[TiDBでSQLステートメントを探索](/basic-sql-operations.md)できます。"
