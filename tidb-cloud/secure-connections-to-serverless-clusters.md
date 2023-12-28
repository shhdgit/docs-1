---
title: TLS Connections to TiDB Serverless
summary: Introduce TLS connections in TiDB Serverless.
aliases: ['/tidbcloud/secure-connections-to-serverless-tier-clusters']
---

# TiDB ServerlessへのTLS接続 {#tls-connections-to-tidb-serverless}

クライアントとTiDB Serverlessクラスター間の安全なTLS接続を確立することは、データベースに接続するための基本的なセキュリティ手法の1つです。TiDB Serverlessのサーバー証明書は、独立した第三者の証明書プロバイダーによって発行されます。サーバー側のデジタル証明書をダウンロードすることなく、簡単にTiDB Serverlessクラスターに接続することができます。

## 前提条件 {#prerequisites}

- [パスワード認証](/tidb-cloud/tidb-cloud-password-authentication.md)または[SSO認証](/tidb-cloud/tidb-cloud-sso-authentication.md)を使用して、TiDB Cloudにログインします。
- [TiDB Serverlessクラスターを作成します](/tidb-cloud/tidb-cloud-quickstart.md)。

## TiDB ServerlessクラスターへのTLS接続 {#tls-connection-to-a-tidb-serverless-cluster}

[TiDB Cloudコンソール](https://tidbcloud.com/)では、さまざまな接続方法の例を取得し、次のようにTiDB Serverlessクラスターに接続することができます。

1. プロジェクトの[**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、クラスターの名前をクリックして、概要ページに移動します。

2. 右上隅の**接続**をクリックします。ダイアログが表示されます。

3. ダイアログで、エンドポイントタイプのデフォルト設定を`Public`に保持し、お好みの接続方法とオペレーティングシステムを選択します。

4. パスワードをまだ設定していない場合は、**パスワードの生成**をクリックして、TiDB Serverlessクラスターに接続するためのサンプル接続文字列に自動的に埋め込まれるランダムなパスワードを生成します。

   > **Note:**
   >
   > - ランダムなパスワードは、大文字と小文字のアルファベット、数字、および特殊文字を含む16文字で構成されます。
   > - このダイアログを閉じると、生成されたパスワードは再表示されないため、安全な場所にパスワードを保存する必要があります。忘れた場合は、このダイアログで**パスワードのリセット**をクリックしてリセットすることができます。
   > - TiDB Serverlessクラスターはインターネット経由でアクセスできます。パスワードを他の場所で使用する必要がある場合は、データベースのセキュリティを確保するためにリセットすることをお勧めします。

5. 接続文字列を使用してクラスターに接続します。

   > **Note:**
   >
   > TiDB Serverlessクラスターに接続する場合、ユーザー名にクラスターの接頭辞を含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名の接頭辞](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

## ルート証明書の管理 {#root-certificate-management}

### ルート証明書の発行と有効期限 {#root-certificate-issuance-and-validity}

TiDB Serverlessは、クライアントとTiDB Serverlessクラスター間のTLS接続のための証明書機関（CA）として[Let's Encrypt](https://letsencrypt.org/)の証明書を使用します。TiDB Serverless証明書が期限切れになると、クラスターの正常な操作や確立されたTLSセキュア接続に影響を与えることなく、自動的にローテーションされます。

クライアントがJavaやGoなどのシステムのルートCAストアをデフォルトで使用している場合、CAルートのパスを指定せずに簡単にTiDB Serverlessクラスターに安全に接続することができます。ただし、一部のドライバーやORMはシステムのルートCAストアを使用しない場合があります。その場合は、ドライバーやORMのCAルートパスをシステムのルートCAストアに設定する必要があります。たとえば、macOS上のPythonでTiDB Serverlessクラスターに接続する場合は、[mysqlclient](https://github.com/PyMySQL/mysqlclient)を使用して`ssl`引数で`ca: /etc/ssl/cert.pem`を設定する必要があります。

[ISRG Root X1](https://letsencrypt.org/certs/isrgrootx1.pem.txt)証明書をダウンロードする必要があるGUIクライアント（DBeaverなど）では、複数の証明書が含まれる証明書ファイルを受け入れないため、証明書をダウンロードする必要があります。

### ルート証明書のデフォルトパス {#root-certificate-default-path}

異なるオペレーティングシステムでは、ルート証明書のデフォルトの保存パスは次のとおりです：

**MacOS**

    /etc/ssl/cert.pem

**Debian / Ubuntu / Arch**

**Debian / Ubuntu / Arch**

    /etc/ssl/certs/ca-certificates.crt

**RedHat / Fedora / CentOS / Mageia**

**RedHat / Fedora / CentOS / Mageia**

    /etc/pki/tls/certs/ca-bundle.crt

**アルパイン**

    /etc/ssl/cert.pem

**OpenSUSE**

**OpenSUSE**

    /etc/ssl/ca-bundle.pem

**Windows**

WindowsにはCAルートへの特定のパスがありません。代わりに、[レジストリ](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/local-machine-and-current-user-certificate-stores)を使用して証明書を保存します。そのため、WindowsでCAルートパスを指定するには、次の手順を実行します。

1. [ISRG Root X1証明書](https://letsencrypt.org/certs/isrgrootx1.pem.txt)をダウンロードし、任意のパス（例：\<path\_to\_ca>）に保存します。
2. TiDB Serverlessクラスタに接続する際に、パス（\<path\_to\_ca>）をCAルートパスとして使用します。

## FAQ {#faqs}

### TiDB Serverlessクラスタに接続するためにサポートされているTLSバージョンはどれですか？ {#which-tls-versions-are-supported-to-connect-to-my-tidb-serverless-cluster}

セキュリティ上の理由から、TiDB ServerlessはTLS 1.2とTLS 1.3のみをサポートし、TLS 1.0とTLS 1.1のバージョンはサポートしていません。詳細については、IETF [Deprecating TLS 1.0 and TLS 1.1](https://datatracker.ietf.org/doc/rfc8996/)を参照してください。

### 接続クライアントとTiDB Serverlessの間での双方向TLS認証はサポートされていますか？ {#is-two-way-tls-authentication-between-my-connection-client-and-tidb-serverless-supported}

いいえ。

TiDB Serverlessは片方向のTLS認証のみをサポートしており、クライアントはTiDB Cloudクラスタ証明書の秘密鍵の署名を検証するために公開鍵を使用しますが、クラスタはクライアントを検証しません。

### TiDB Serverlessは安全な接続を確立するためにTLSを設定する必要がありますか？ {#does-tidb-serverless-have-to-configure-tls-to-establish-a-secure-connection}

標準接続では、TiDB ServerlessはTLS接続のみを許可し、非SSL/TLS接続を禁止します。その理由は、SSL/TLSがインターネット経由でTiDB Serverlessクラスタに接続する際にデータの漏洩リスクを減らすための最も基本的なセキュリティ対策の1つだからです。

プライベートエンドポイント接続では、高度に安全で片方向のアクセスをTiDB Cloudサービスに提供し、データを公共のインターネットに公開しないため、TLSの設定は任意です。
