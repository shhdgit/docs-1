---
title: TLS Connections to TiDB Serverless
summary: Introduce TLS connections in TiDB Serverless.
aliases: ['/tidbcloud/secure-connections-to-serverless-tier-clusters']
---

# TiDB ServerlessへのTLS接続 {#tls-connections-to-tidb-serverless}

クライアントとTiDB Serverlessクラスターとの間で安全なTLS接続を確立することは、データベースに接続するための基本的なセキュリティ慣行の1つです。TiDB Serverlessのサーバー証明書は独立した第三者証明書プロバイダーによって発行されます。サーバーサイドのデジタル証明書をダウンロードせずに、簡単にTiDB Serverlessクラスターに接続することができます。

## 前提条件 {#prerequisites}

-   [パスワード認証](/tidb-cloud/tidb-cloud-password-authentication.md)または[SSO認証](/tidb-cloud/tidb-cloud-sso-authentication.md)を使用してTiDB Cloudにログインします。
-   [TiDB Serverlessクラスターを作成](/tidb-cloud/tidb-cloud-quickstart.md)します。

## TiDB ServerlessクラスターへのTLS接続 {#tls-connection-to-a-tidb-serverless-cluster}

[TiDB Cloudコンソール](https://tidbcloud.com/)では、異なる接続方法の例とともに、TiDB Serverlessクラスターに接続する方法が以下のようになります：

1.  プロジェクトの[**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、クラスターの名前をクリックして概要ページに移動します。

2.  右上隅の**接続**をクリックします。ダイアログが表示されます。

3.  ダイアログで、エンドポイントタイプを`Public`のままにして、希望の接続方法とオペレーティングシステムを選択します。

4.  まだパスワードを設定していない場合は、TiDB Serverlessクラスター用にランダムなパスワードを生成するために**パスワードを生成**をクリックします。生成されたパスワードは、クラスターに簡単に接続するためのサンプル接続文字列に自動的に埋め込まれます。

    > **注意:**
    >
    > -   ランダムなパスワードは、大文字と小文字のアルファベット、数字、特殊文字を含む16文字で構成されています。
    > -   このダイアログを閉じると、生成されたパスワードは再表示されないため、安全な場所に保存する必要があります。忘れた場合は、このダイアログで**パスワードをリセット**をクリックしてリセットすることができます。
    > -   TiDB Serverlessクラスターはインターネット経由でアクセスできます。他でパスワードを使用する必要がある場合は、データベースのセキュリティを確保するためにリセットすることをお勧めします。

5.  接続文字列でクラスターに接続します。

    > **注意:**
    >
    > TiDB Serverlessクラスターに接続する際は、ユーザー名にクラスターの接頭辞を含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名の接頭辞](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

## ルート証明書の管理 {#root-certificate-management}

### ルート証明書の発行と有効性 {#root-certificate-issuance-and-validity}

TiDB Serverlessは、クライアントとTiDB Serverlessクラスター間のTLS接続のための証明書機関（CA）として[Let's Encrypt](https://letsencrypt.org/)の証明書を使用しています。TiDB Serverless証明書が期限切れになると、クラスターの正常な運用と確立されたTLS安全な接続に影響を与えることなく、自動的にローテーションされます。

クライアントがデフォルトでシステムのルートCAストアを使用している場合（JavaやGoなど）、CAルートのパスを指定せずに簡単にTiDB Serverlessクラスターに安全に接続することができます。ただし、一部のドライバーやORMはシステムのルートCAストアを使用していない場合があります。その場合は、ドライバーやORMのCAルートパスをシステムのルートCAストアに設定する必要があります。たとえば、macOS上のPythonでTiDB Serverlessクラスターに接続するために[mysqlclient](https://github.com/PyMySQL/mysqlclient)を使用する場合、`ssl`引数で`ca: /etc/ssl/cert.pem`を設定する必要があります。

証明書ファイルに複数の証明書が含まれている場合、DBeaverなどのGUIクライアントを使用している場合は、[ISRG Root X1](https://letsencrypt.org/certs/isrgrootx1.pem.txt)証明書をダウンロードする必要があります。

### ルート証明書のデフォルトパス {#root-certificate-default-path}

異なるオペレーティングシステムでは、ルート証明書のデフォルトの保存パスは次のようになります：

**MacOS**

    /etc/ssl/cert.pem

**Debian / Ubuntu / Arch**

    /etc/ssl/certs/ca-certificates.crt

**RedHat / Fedora / CentOS / Mageia**

    /etc/pki/tls/certs/ca-bundle.crt

**アルパイン**

    /etc/ssl/cert.pem

**OpenSUSE**

    /etc/ssl/ca-bundle.pem

**Windows**

WindowsはCAルートへの特定のパスを提供していません。代わりに、証明書を保存するために[レジストリ](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/local-machine-and-current-user-certificate-stores)を使用しています。そのため、WindowsでCAルートのパスを指定するには、次の手順を実行してください：

1.  [ISRG Root X1証明書](https://letsencrypt.org/certs/isrgrootx1.pem.txt)をダウンロードし、任意のパス（たとえば`<path_to_ca>`）に保存します。
2.  TiDB Serverlessクラスタに接続する際に、そのパス（`<path_to_ca>`）をCAルートパスとして使用します。

## よくある質問 {#faqs}

### TiDB Serverlessクラスタに接続するためにサポートされているTLSバージョンは何ですか？ {#which-tls-versions-are-supported-to-connect-to-my-tidb-serverless-cluster}

セキュリティ上の理由から、TiDB ServerlessはTLS 1.2およびTLS 1.3のみをサポートし、TLS 1.0およびTLS 1.1のバージョンはサポートしていません。詳細については、IETFの[Deprecating TLS 1.0 and TLS 1.1](https://datatracker.ietf.org/doc/rfc8996/)を参照してください。

### 接続クライアントとTiDB Serverlessの間での双方向TLS認証はサポートされていますか？ {#is-two-way-tls-authentication-between-my-connection-client-and-tidb-serverless-supported}

いいえ。

TiDB Serverlessは単方向TLS認証のみをサポートしており、つまりクライアントはTiDB Cloudクラスタ証明書の秘密鍵の署名を検証するために公開鍵を使用しますが、クラスタはクライアントを検証しません。

### TiDB Serverlessは安全な接続を確立するためにTLSを構成する必要がありますか？ {#does-tidb-serverless-have-to-configure-tls-to-establish-a-secure-connection}

標準接続では、TiDB ServerlessはSSL/TLS接続のみを許可し、非SSL/TLS接続を禁止しています。その理由は、インターネット経由でTiDB Serverlessクラスタに接続する際にデータ露出のリスクを減らすための基本的なセキュリティ対策の1つであるためです。

プライベートエンドポイント接続では、TiDB Cloudサービスへの高度に安全な単方向アクセスをサポートし、データを公共のインターネットに公開しないため、TLSの構成は任意です。
