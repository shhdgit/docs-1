---
title: TLS Connections to TiDB Serverless
summary: Introduce TLS connections in TiDB Serverless.
aliases: ['/tidbcloud/secure-connections-to-serverless-tier-clusters']
---

# TLS Connections to TiDB Serverless {#tls-connections-to-tidb-serverless}

クライアントとTiDB Serverlessクラスター間の安全なTLS接続を確立することは、データベースに接続するための基本的なセキュリティ慣行の1つです。TiDB Serverlessのサーバー証明書は、独立した第三者証明書プロバイダーによって発行されます。サーバーサイドのデジタル証明書をダウンロードせずに、簡単にTiDB Serverlessクラスターに接続できます。

## 前提条件 {#prerequisites}

- [パスワード認証](/tidb-cloud/tidb-cloud-password-authentication.md)または[SSO認証](/tidb-cloud/tidb-cloud-sso-authentication.md)を使用して、TiDB Cloudにログインします。
- [TiDB Serverlessクラスターを作成](/tidb-cloud/tidb-cloud-quickstart.md)します。

## TiDB ServerlessクラスターへのTLS接続 {#tls-connection-to-a-tidb-serverless-cluster}

[TiDB Cloudコンソール](https://tidbcloud.com/)では、さまざまな接続方法の例と、次のようにTiDB Serverlessクラスターに接続する方法が示されます。

1. プロジェクトの[**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、クラスターの名前をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。ダイアログが表示されます。

3. ダイアログで、エンドポイントタイプを`Public`のままにし、希望の接続方法とオペレーティングシステムを選択します。

4. パスワードをまだ設定していない場合は、**パスワードを生成**をクリックして、TiDB Serverlessクラスターに接続するためのサンプル接続文字列にランダムなパスワードを生成します。パスワードは自動的にクラスターに簡単に接続するためのサンプル接続文字列に埋め込まれます。

   > **Note:**
   >
   > - ランダムなパスワードは、大文字と小文字のアルファベット、数字、特殊文字を含む16文字で構成されています。
   > - このダイアログを閉じると、生成されたパスワードは再表示されないため、安全な場所に保存する必要があります。忘れた場合は、このダイアログで**パスワードをリセット**をクリックしてリセットできます。
   > - TiDB Serverlessクラスターはインターネット経由でアクセスできます。他の場所でパスワードを使用する必要がある場合は、データベースのセキュリティを確保するためにリセットすることをお勧めします。

5. 接続文字列を使用してクラスターに接続します。

   > **Note:**
   >
   > TiDB Serverlessクラスターに接続する際は、ユーザー名にクラスターの接頭辞を含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名の接頭辞](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

## ルート証明書の管理 {#root-certificate-management}

### ルート証明書の発行と有効性 {#root-certificate-issuance-and-validity}

TiDB Serverlessは、クライアントとTiDB Serverlessクラスター間のTLS接続のための証明書として[Let's Encrypt](https://letsencrypt.org/)の証明機関（CA）を使用します。TiDB Serverless証明書が期限切れになると、クラスターの正常な操作と確立されたTLSセキュア接続に影響を与えることなく、自動的にローテーションされます。

クライアントがデフォルトでシステムのルートCAストアを使用する場合（例：JavaやGoなど）、CAルートのパスを指定せずに簡単にTiDB Serverlessクラスターに安全に接続できます。ただし、一部のドライバーやORMはシステムのルートCAストアを使用しない場合があります。その場合は、ドライバーやORMのCAルートパスをシステムのルートCAストアに構成する必要があります。たとえば、macOS上のPythonで[TiDB Serverlessクラスターに接続するためにmysqlclient](https://github.com/PyMySQL/mysqlclient)を使用する場合、`ssl`引数に`ca: /etc/ssl/cert.pem`を設定する必要があります。

証明書ファイルに複数の証明書が含まれることを受け入れないGUIクライアント（DBeaverなど）を使用している場合は、[ISRG Root X1](https://letsencrypt.org/certs/isrgrootx1.pem)証明書をダウンロードする必要があります。

### ルート証明書のデフォルトパス {#root-certificate-default-path}

異なるオペレーティングシステムでは、ルート証明書のデフォルトの保存パスは次のとおりです：

**MacOS**

```
/etc/ssl/cert.pem
```

**Debian / Ubuntu / Arch**

```
/etc/ssl/certs/ca-certificates.crt
```

**RedHat / Fedora / CentOS / Mageia**

```
/etc/pki/tls/certs/ca-bundle.crt
```

**Alpine**

```
/etc/ssl/cert.pem
```

**OpenSUSE**

```
/etc/ssl/ca-bundle.pem
```

**Windows**

WindowsにはCAルートへの特定のパスが用意されていません。代わりに、[レジストリ](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/local-machine-and-current-user-certificate-stores)を使用して証明書を保存します。このため、WindowsでCAルートパスを指定するには、次の手順を実行します：

1. [ISRG Root X1証明書](https://letsencrypt.org/certs/isrgrootx1.pem)をダウンロードし、`<path_to_ca>`などの好きなパスに保存します。
2. TiDB Serverlessクラスターに接続する際に、このパス（`<path_to_ca>`）をCAルートパスとして使用します。

## FAQ {#faqs}

### TiDB Serverlessクラスターに接続するためにサポートされているTLSバージョンはどれですか？ {#which-tls-versions-are-supported-to-connect-to-my-tidb-serverless-cluster}

セキュリティ上の理由から、TiDB ServerlessはTLS 1.2およびTLS 1.3のみをサポートし、TLS 1.0およびTLS 1.1のバージョンはサポートしていません。詳細については、IETFの[Deprecating TLS 1.0 and TLS 1.1](https://datatracker.ietf.org/doc/rfc8996/)を参照してください。

### 接続クライアントとTiDB Serverless間の双方向TLS認証はサポートされていますか？ {#is-two-way-tls-authentication-between-my-connection-client-and-tidb-serverless-supported}

いいえ。

TiDB Serverlessは、クライアントが公開鍵を使用してTiDB Cloudクラスター証明書の秘密鍵の署名を検証する一方向TLS認証のみをサポートしています。つまり、クラスターはクライアントを検証しません。

### TiDB Serverlessは安全な接続を確立するためにTLSを構成する必要がありますか？ {#does-tidb-serverless-have-to-configure-tls-to-establish-a-secure-connection}

標準接続の場合、TiDB ServerlessはTLS接続のみを許可し、非SSL/TLS接続を禁止しています。これは、インターネット経由でTiDB Serverlessクラスターに接続する際にデータ露出のリスクを減らすための基本的なセキュリティ対策の1つであるためです。

プライベートエンドポイント接続の場合、高度に安全で一方向のTiDB Cloudサービスへのアクセスをサポートし、データを公共のインターネットに公開しないため、TLSの構成は任意です。
