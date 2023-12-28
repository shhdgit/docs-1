---
title: Integrate TiDB Cloud with Netlify
summary: Learn how to connect your TiDB Cloud clusters to Netlify projects.
---

# NetlifyとTiDB Cloudを統合する {#integrate-tidb-cloud-with-netlify}

[Netlify](https://netlify.com/)は、モダンなWebプロジェクトを自動化するためのオールインワンプラットフォームです。ホスティングインフラストラクチャー、継続的インテグレーション、デプロイパイプラインを1つのワークフローに置き換え、プロジェクトが成長するにつれてサーバーレス関数、ユーザー認証、フォーム処理などの動的機能を統合します。

このドキュメントでは、TiDB Cloudをデータベースバックエンドとして使用してNetlifyにフルスタックアプリをデプロイする方法について説明します。また、TiDB Cloudサーバーレスドライバーを使用してNetlifyエッジ関数を使用する方法も学べます。

## 前提条件 {#prerequisites}

デプロイする前に、次の前提条件を満たしていることを確認してください。

### NetlifyアカウントとCLI {#a-netlify-account-and-cli}

NetlifyアカウントとCLIを持っていることが期待されます。持っていない場合は、次のリンクを参照して作成してください。

- [Netlifyアカウントにサインアップする](https://app.netlify.com/signup).
- [Netlify CLIを取得する](https://docs.netlify.com/cli/get-started/).

### TiDB CloudアカウントとTiDBクラスター {#a-tidb-cloud-account-and-a-tidb-cluster}

TiDB Cloudのアカウントとクラスターを持っていることが期待されます。持っていない場合は、次のリンクを参照して作成してください。

- [TiDB Serverlessクラスターを作成する](/tidb-cloud/create-tidb-cluster-serverless.md)
- [TiDB Dedicatedクラスターを作成する](/tidb-cloud/create-tidb-cluster.md)

1つのTiDB Cloudクラスターは、複数のNetlifyサイトに接続できます。

### TiDB Cloudでトラフィックフィルターに許可されているすべてのIPアドレス {#all-ip-addresses-allowed-for-traffic-filter-in-tidb-cloud}

TiDB Dedicatedクラスターの場合、クラスターのトラフィックフィルターが接続にすべてのIPアドレス（`0.0.0.0/0`に設定）を許可していることを確認してください。これは、Netlifyデプロイメントが動的IPアドレスを使用するためです。

TiDB Serverlessクラスターは、接続にすべてのIPアドレスをデフォルトで許可するため、トラフィックフィルターを構成する必要はありません。

## ステップ1. サンプルプロジェクトと接続文字列を取得する {#step-1-get-the-example-project-and-the-connection-string}

すばやく始めるために、TiDB Cloudは、ReactとPrisma Clientを使用したNext.jsのTypeScriptでフルスタックのサンプルアプリを提供しています。これは、自分のブログを投稿して削除できるシンプルなブログサイトです。すべてのコンテンツは、Prismaを介してTiDB Cloudに保存されます。

### サンプルプロジェクトをフォークして、自分のスペースにクローンする {#fork-the-example-project-and-clone-it-to-your-own-space}

1. [Next.jsとPrismaを使用したフルスタックのサンプル](https://github.com/tidbcloud/nextjs-prisma-example)リポジトリを自分のGitHubリポジトリにフォークします。

2. フォークしたリポジトリを自分のスペースにクローンします。

   ```shell
   git clone https://github.com/${your_username}/nextjs-prisma-example.git
   cd nextjs-prisma-example/
   ```

### TiDB Cloudの接続文字列を取得する {#get-the-tidb-cloud-connection-string}

TiDB Serverlessクラスターの場合、[TiDB Cloud CLI](/tidb-cloud/cli-reference.md)または[TiDB Cloudコンソール](https://tidbcloud.com/)から接続文字列を取得できます。

TiDB Dedicatedクラスターの場合、TiDB Cloudコンソールからのみ接続文字列を取得できます。

<SimpleTab>
<div label="TiDB Cloud CLI">

> **ヒント:**
>
> Cloud CLIをインストールしていない場合は、次の手順を実行する前に、[TiDB Cloud CLIクイックスタート](/tidb-cloud/get-started-with-cli.md)を参照して、クイックインストールを行ってください。

1. インタラクティブモードでクラスターの接続文字列を取得します。

   ```shell
   ticloud cluster connect-info
   ```

2. プロンプトに従って、クラスター、クライアント、およびオペレーティングシステムを選択します。このドキュメントで使用されるクライアントは`Prisma`です。

       クラスターを選択
       > [x] Cluster0(13796194496)
       クライアントを選択
       > [x] Prisma
       オペレーティングシステムを選択
       > [x] macOS/Alpine (Detected)

   出力は次のようになります。ここで、`url`の値にPrismaの接続文字列が見つかります。

   ```shell
   datasource db {
   provider = "mysql"
   url      = "mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict"
   }
   ```

   > **注意:**
   >
   > 後で接続文字列を使用する場合は、次のことに注意してください。
   >
   > - 接続文字列のパラメーターを実際の値で置き換えます。
   > - このドキュメントのサンプルアプリでは、新しいデータベースが必要なので、`<Database>`を一意の新しい名前に置き換える必要があります。

</div>
<div label="TiDB Cloudコンソール">

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動します。ターゲットクラスタの名前をクリックして、概要ページに移動し、右上の**Connect**をクリックします。表示されるダイアログから、接続文字列から次の接続パラメータを取得できます。

   - `${host}`
   - `${port}`
   - `${user}`
   - `${password}`

2. 次の接続文字列に接続パラメータを入力します。

   ```shell
   mysql://<User>:<Password>@<Host>:<Port>/<Database>?sslaccept=strict
   ```

   > **Note:**
   >
   > 接続文字列を後で使用する場合は、次の点に注意してください。
   >
   > - 接続文字列内のパラメータを実際の値で置き換えます。
   > - このドキュメントの例のアプリケーションでは、新しいデータベースが必要なので、`<Database>`を一意の新しい名前に置き換える必要があります。

</div>
</SimpleTab>

## ステップ2. Netlifyに例のアプリケーションをデプロイする {#step-2-deploy-the-example-app-to-netlify}

1. Netlify CLIで、Netlifyアカウントに認証し、アクセストークンを取得します。

   ```shell
   netlify login
   ```

2. 自動セットアップを開始します。このステップでは、継続的デプロイメントのためにリポジトリを接続するため、Netlify CLIはリポジトリ上にデプロイキーとWebhookを作成するためのアクセスが必要です。

   ```shell
   netlify init
   ```

   プロンプトが表示されたら、**Create & configure a new site**を選択し、GitHubアクセスを許可します。その他のオプションはすべてデフォルト値を使用します。

   ```shell
   Adding local .netlify folder to .gitignore file...
   ? What would you like to do? +  Create & configure a new site
   ? Team: your_username’s team
   ? Site name (leave blank for a random name; you can change it later):

   Site Created

   Admin URL: https://app.netlify.com/sites/mellow-crepe-e2ca2b
   URL:       https://mellow-crepe-e2ca2b.netlify.app
   Site ID:   b23d1359-1059-49ed-9d08-ed5dba8e83a2

   Linked to mellow-crepe-e2ca2b


   ? Netlify CLI needs access to your GitHub account to configure Webhooks and Deploy Keys. What would you like to do? Authorize with GitHub through app.netlify.com
   Configuring Next.js runtime...

   ? Your build command (hugo build/yarn run build/etc): npm run netlify-build
   ? Directory to deploy (blank for current dir): .next

   Adding deploy key to repository...
   (node:36812) ExperimentalWarning: The Fetch API is an experimental feature. This feature could change at any time
   (Use `node --trace-warnings ...` to show where the warning was created)
   Deploy key added!

   Creating Netlify GitHub Notification Hooks...
   Netlify Notification Hooks configured!

   Success! Netlify CI/CD Configured!

   This site is now configured to automatically deploy from github branches & pull requests

   Next steps:

   git push       Push to your git repository to trigger new site builds
   netlify open   Open the Netlify admin URL of your site
   ```

3. 環境変数を設定します。自分のスペースとNetlifyスペースからTiDB Cloudクラスタに接続するために、[Step 1](#step-1-get-the-example-project-and-the-connection-string)で取得した接続文字列を`DATABASE_URL`として設定する必要があります。

   ```shell
   # 自分のスペースの環境変数を設定します
   export DATABASE_URL='mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict'

   # Netlifyスペースの環境変数を設定します
   netlify env:set DATABASE_URL 'mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict'
   ```

   環境変数を確認します。

   ```shell
   # 自分のスペースの環境変数を確認します
   env | grep DATABASE_URL

   # Netlifyスペースの環境変数を確認します
   netlify env:list
   ```

4. アプリをローカルでビルドし、スキーマをTiDB Cloudクラスタにマイグレーションします。

   > **Tips:**
   >
   > ローカルデプロイメントをスキップし、アプリを直接Netlifyにデプロイしたい場合は、ステップ6に進んでください。

   ```shell
   npm install .
   npm run netlify-build
   ```

5. アプリケーションをローカルで実行します。ローカル開発サーバーを起動してサイトをプレビューできます。

   ```shell
   netlify dev
   ```

   次に、ブラウザで`http://localhost:3000/`にアクセスしてUIを確認します。

6. アプリをNetlifyにデプロイします。ローカルプレビューで満足したら、次のコマンドを使用してサイトをNetlifyにデプロイできます。`--trigger`は、ローカルファイルをアップロードせずにデプロイすることを意味します。ローカルで変更を加えた場合は、GitHubリポジトリにコミットしたことを確認してください。

   ```shell
   netlify deploy --prod --trigger
   ```

   Netlifyコンソールに移動して、デプロイ状態を確認します。デプロイが完了すると、アプリ用のサイトにはNetlifyによって提供されるパブリックIPアドレスがあり、誰でもアクセスできるようになります。

## エッジ関数を使用する {#use-the-edge-function}

前述のセクションで説明した例のアプリは、Netlifyサーバーレス関数で実行されます。このセクションでは、[TiDB Cloudサーバーレスドライバー](/tidb-cloud/serverless-driver.md)を使用してエッジ関数を使用する方法を説明します。エッジ関数は、Netlifyが提供する機能で、Netlify CDNのエッジでサーバーレス関数を実行できるようにします。

エッジ関数を使用するには、次の手順を実行します：

1. プロジェクトのルートディレクトリに `netlify/edge-functions` という名前のディレクトリを作成します。

2. ディレクトリに `hello.ts` という名前のファイルを作成し、以下のコードを追加します：

   ```typescript
   import { connect } from 'https://esm.sh/@tidbcloud/serverless'

   export default async () => {
     const conn = connect({url: Netlify.env.get('DATABASE_URL')})
     const result = await conn.execute('show databases')
     return new Response(JSON.stringify(result));
   }

   export const config = { path: "/api/hello" };
   ```

3. `DATABASE_URL` 環境変数を設定します。接続情報は [TiDB Cloud コンソール](https://tidbcloud.com/) から取得できます。

   ```shell
   netlify env:set DATABASE_URL 'mysql://<username>:<password>@<host>/<database>'
   ```

4. エッジ関数を Netlify にデプロイします。

   ```shell
   netlify deploy --prod --trigger
   ```

デプロイが完了したら、Netlify コンソールでデプロイの状態を確認できます。デプロイが完了したら、`https://<netlify-host>/api/hello` の URL を通じてエッジ関数にアクセスできます。
