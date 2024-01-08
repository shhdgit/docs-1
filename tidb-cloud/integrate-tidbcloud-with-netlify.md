---
title: Integrate TiDB Cloud with Netlify
summary: Learn how to connect your TiDB Cloud clusters to Netlify projects.
---

# NetlifyとTiDB Cloudの統合 {#integrate-tidb-cloud-with-netlify}

[Netlify](https://netlify.com/)は、モダンなWebプロジェクトを自動化するためのオールインワンプラットフォームです。ホスティングインフラストラクチャ、継続的な統合、およびデプロイパイプラインを単一のワークフローに置き換え、サーバーレス関数、ユーザー認証、およびフォーム処理などの動的機能をプロジェクトが成長するにつれて統合します。

このドキュメントでは、TiDB Cloudをデータベースバックエンドとして使用してNetlifyでフルスタックアプリをデプロイする方法について説明します。また、TiDB Cloudのサーバーレスドライバを使用してNetlifyエッジ関数を使用する方法も学ぶことができます。

## 前提条件 {#prerequisites}

デプロイの前に、次の前提条件を満たしていることを確認してください。

### NetlifyアカウントとCLI {#a-netlify-account-and-cli}

NetlifyアカウントとCLIを持っていることが期待されます。持っていない場合は、次のリンクを参照して作成してください。

- [Netlifyアカウントにサインアップ](https://app.netlify.com/signup)。
- [Netlify CLIを取得](https://docs.netlify.com/cli/get-started/)。

### TiDB CloudアカウントとTiDBクラスタ {#a-tidb-cloud-account-and-a-tidb-cluster}

TiDB Cloudのアカウントとクラスタを持っていることが期待されます。持っていない場合は、次のリンクを参照して作成してください。

- [TiDB Serverlessクラスタを作成](/tidb-cloud/create-tidb-cluster-serverless.md)
- [TiDB Dedicatedクラスタを作成](/tidb-cloud/create-tidb-cluster.md)

1つのTiDB Cloudクラスタは複数のNetlifyサイトに接続できます。

### TiDB Cloudでトラフィックフィルタに許可されたすべてのIPアドレス {#all-ip-addresses-allowed-for-traffic-filter-in-tidb-cloud}

TiDB Dedicatedクラスタの場合、クラスタのトラフィックフィルタが接続のためにすべてのIPアドレス（`0.0.0.0/0`に設定）を許可していることを確認してください。これは、Netlifyのデプロイが動的IPアドレスを使用するためです。

TiDB Serverlessクラスタはデフォルトで接続のためにすべてのIPアドレスを許可しているため、トラフィックフィルタを構成する必要はありません。

## ステップ1. 例のプロジェクトと接続文字列を取得 {#step-1-get-the-example-project-and-the-connection-string}

迅速に始めるために、TiDB Cloudは、ReactとPrisma Clientを使用したNext.jsで作成されたTypeScriptのフルスタック例のアプリを提供しています。これは、自分のブログを投稿したり削除したりできるシンプルなブログサイトです。すべてのコンテンツはPrismaを介してTiDB Cloudに保存されています。

### 例のプロジェクトをフォークして自分のスペースにクローンする {#fork-the-example-project-and-clone-it-to-your-own-space}

1. [Next.jsとPrismaを使用したフルスタックの例](https://github.com/tidbcloud/nextjs-prisma-example)リポジトリを自分のGitHubリポジトリにフォークします。

2. フォークしたリポジトリを自分のスペースにクローンします：

   ```shell
   git clone https://github.com/${your_username}/nextjs-prisma-example.git
   cd nextjs-prisma-example/
   ```

### TiDB Cloudの接続文字列を取得 {#get-the-tidb-cloud-connection-string}

TiDB Serverlessクラスタの場合、[TiDB Cloud CLI](/tidb-cloud/cli-reference.md)または[TiDB Cloudコンソール](https://tidbcloud.com/)から接続文字列を取得できます。

TiDB Dedicatedクラスタの場合、接続文字列はTiDB Cloudコンソールからのみ取得できます。

<SimpleTab>
<div label="TiDB Cloud CLI">

> **ヒント:**
>
> Cloud CLIをインストールしていない場合は、次の手順を実行する前に[TiDB Cloud CLIクイックスタート](/tidb-cloud/get-started-with-cli.md)を参照してクイックインストールしてください。

1. インタラクティブモードでクラスタの接続文字列を取得します：

   ```shell
   ticloud cluster connect-info
   ```

2. クラスタ、クライアント、およびオペレーティングシステムを選択するためにプロンプトに従ってください。このドキュメントで使用されるクライアントは`Prisma`です。

   ```
   クラスタを選択
   > [x] Cluster0(13796194496)
   クライアントを選択
   > [x] Prisma
   オペレーティングシステムを選択
   > [x] macOS/Alpine (Detected)
   ```

   次のような出力があり、`url`の値でPrismaの接続文字列を見つけることができます。

   ```shell
   datasource db {
   provider = "mysql"
   url      = "mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict"
   }
   ```

   > **注意:**
   >
   > 後で接続文字列を使用する場合は、次の点に注意してください：
   >
   > - 接続文字列内のパラメータを実際の値で置き換えてください。
   > - このドキュメントの例のアプリでは新しいデータベースが必要なので、`<Database>`を一意の新しい名前に置き換える必要があります。

</div>
<div label="TiDB Cloudコンソール">

1. [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタの概要ページに移動して、右上隅の**Connect**をクリックします。表示されるダイアログで、接続文字列から次の接続パラメータを取得できます。

   - `${host}`
   - `${port}`
   - `${user}`
   - `${password}`

2. 次の接続文字列に接続パラメータを入力してください：

   ```shell
   mysql://<User>:<Password>@<Host>:<Port>/<Database>?sslaccept=strict
   ```

   > **注意:**
   >
   > 後で接続文字列を使用する場合は、次の点に注意してください：
   >
   > - 接続文字列内のパラメータを実際の値で置き換えてください。
   > - このドキュメントの例のアプリでは新しいデータベースが必要なので、`<Database>`を一意の新しい名前に置き換える必要があります。

</div>
</SimpleTab>

## ステップ2. Netlifyに例のアプリをデプロイする {#step-2-deploy-the-example-app-to-netlify}

1. Netlify CLIで、Netlifyアカウントに認証し、アクセストークンを取得します。

   ```shell
   netlify login
   ```

2. 自動セットアップを開始します。このステップでは、継続的なデプロイのためにリポジトリを接続するため、Netlify CLIがデプロイキーとリポジトリ上のウェブフックを作成するためのアクセスが必要です。

   ```shell
   netlify init
   ```

   プロンプトが表示されたら、**新しいサイトを作成および構成**を選択し、GitHubにアクセスを許可します。その他のオプションについてはデフォルト値を使用します。

   ```shell
   ローカルの .netlify フォルダを .gitignore ファイルに追加中...
   ? 何をしたいですか？ +  新しいサイトを作成および構成
   ? チーム: your_username’s team
   ? サイト名（ランダムな名前にするには空白のままにしてください；後で変更できます）:

   サイトが作成されました

   管理者URL: https://app.netlify.com/sites/mellow-crepe-e2ca2b
   URL:       https://mellow-crepe-e2ca2b.netlify.app
   サイトID:   b23d1359-1059-49ed-9d08-ed5dba8e83a2

   mellow-crepe-e2ca2b にリンクされました


   ? Netlify CLIがGitHubアカウントにアクセスしてWebhooksとデプロイキーを構成する必要があります。何をしますか？ app.netlify.comを介してGitHubを承認
   Next.jsランタイムを構成中...

   ? ビルドコマンド（hugo build/yarn run build/etc）: npm run netlify-build
   ? デプロイするディレクトリ（現在のディレクトリの場合は空白）: .next

   リポジトリにデプロイキーを追加中...
   (node:36812) ExperimentalWarning: The Fetch API is an experimental feature. This feature could change at any time
   (警告が作成された場所を表示するには `node --trace-warnings ...` を使用してください)
   デプロイキーが追加されました！

   Netlify GitHub通知フックを作成中...
   Netlify通知フックが構成されました！

   成功！Netlify CI/CDが構成されました！

   このサイトは、githubのブランチとプルリクエストから自動的にデプロイするように構成されています

   次のステップ:

   git push       gitリポジトリにプッシュして新しいサイトをビルドする
   netlify open   サイトのNetlify管理URLを開く
   ```

3. 環境変数を設定します。自分のスペースとNetlifyスペースからTiDB Cloudクラスタに接続するために、[ステップ1](#step-1-get-the-example-project-and-the-connection-string)で取得した接続文字列を `DATABASE_URL` として設定する必要があります。

   ```shell
   # 自分のスペースの環境変数を設定
   export DATABASE_URL='mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict'

   # Netlifyスペースの環境変数を設定
   netlify env:set DATABASE_URL 'mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict'
   ```

   環境変数を確認します。

   ```shell
   # 自分のスペースの環境変数を確認
   env | grep DATABASE_URL

   # Netlifyスペースの環境変数を確認
   netlify env:list
   ```

4. アプリをローカルでビルドし、スキーマをTiDB Cloudクラスタにマイグレーションします。

   > **ヒント:**
   >
   > ローカルデプロイをスキップしてアプリを直接Netlifyにデプロイしたい場合は、ステップ6に進んでください。

   ```shell
   npm install .
   npm run netlify-build
   ```

5. アプリケーションをローカルで実行します。ローカル開発サーバーを起動してサイトをプレビューできます。

   ```shell
   netlify dev
   ```

   次に、ブラウザで `http://localhost:3000/` にアクセスしてUIを確認します。

6. アプリをNetlifyにデプロイします。ローカルプレビューで満足したら、次のコマンドを使用してサイトをNetlifyにデプロイできます。`--trigger` はローカルファイルをアップロードせずにデプロイを行うことを意味します。ローカルで変更を加えた場合は、GitHubリポジトリにコミットしていることを確認してください。

   ```shell
   netlify deploy --prod --trigger
   ```

   Netlifyコンソールに移動して、デプロイ状態を確認できます。デプロイが完了すると、アプリのサイトにはNetlifyによって提供されるパブリックIPアドレスが付与されます。

## エッジ関数の使用 {#use-the-edge-function}

上記のセクションで言及されている例のアプリは、Netlifyサーバーレス関数で実行されます。このセクションでは、[TiDB Cloudサーバーレスドライバ](/tidb-cloud/serverless-driver.md)を使用したエッジ関数の使用方法を示します。エッジ関数は、Netlifyが提供する機能で、Netlify CDNのエッジでサーバーレス関数を実行できるようにします。

エッジ関数を使用するには、次の手順を実行します:

1. プロジェクトのルートディレクトリに `netlify/edge-functions` という名前のディレクトリを作成します。

2. ディレクトリに `hello.ts` という名前のファイルを作成し、次のコードを追加します:

   ```typescript
   import { connect } from 'https://esm.sh/@tidbcloud/serverless'

   export default async () => {
     const conn = connect({url: Netlify.env.get('DATABASE_URL')})
     const result = await conn.execute('show databases')
     return new Response(JSON.stringify(result));
   }

   export const config = { path: "/api/hello" };
   ```

3. `DATABASE_URL` 環境変数を設定します。接続情報は[TiDB Cloudコンソール](https://tidbcloud.com/)から取得できます。

   ```shell
   netlify env:set DATABASE_URL 'mysql://<username>:<password>@<host>/<database>'
   ```

4. エッジ関数をNetlifyにデプロイします。

   ```shell
   netlify deploy --prod --trigger
   ```

その後、Netlifyコンソールに移動してデプロイの状態を確認できます。デプロイが完了すると、`https://<netlify-host>/api/hello` URLを介してエッジ関数にアクセスできます。
