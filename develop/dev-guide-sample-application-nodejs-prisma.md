---
title: Connect to TiDB with Prisma
summary: Learn how to connect to TiDB using Prisma. This tutorial gives Node.js sample code snippets that work with TiDB using Prisma.
---

# TiDBとPrismaを使用して接続する {#connect-to-tidb-with-prisma}

TiDBはMySQL互換のデータベースであり、[Prisma](https://github.com/prisma/prisma)はNode.js向けの人気のあるオープンソースORMフレームワークです。

このチュートリアルでは、TiDBとPrismaを使用して次のタスクを達成する方法を学ぶことができます：

-   環境をセットアップする。
-   Prismaを使用してTiDBクラスタに接続する。
-   アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です：

-   あなたのマシンにインストールされている[Node.js](https://nodejs.org/en) >= 16.x。
-   あなたのマシンにインストールされている[Git](https://git-scm.com/downloads)。
-   実行中のTiDBクラスタ。

**TiDBクラスタを持っていない場合は、次のように作成できます：**

<CustomContent platform="tidb">

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ1：サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-nodejs-prisma-quickstart.git
cd tidb-nodejs-prisma-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

以下のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`prisma`を含む）をインストールしてください：

```shell
npm install
```

<details>
<summary><b>既存のプロジェクトに依存関係をインストールする</b></summary>

既存のプロジェクトに対して、以下のコマンドを実行してパッケージをインストールしてください：

```shell
npm install prisma typescript ts-node @types/node --save-dev
```

</details>

### ステップ3：接続パラメータを提供する {#step-3-provide-connection-parameters}

TiDBクラスタに接続するには、選択したTiDB展開オプションに応じて行います。

<SimpleTab>
<div label="TiDBサーバーレス">

1.  [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの名前をクリックして概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成が操作環境に一致することを確認します。

    -   **エンドポイントタイプ** が `Public` に設定されていること。
    -   **Branch** が `main` に設定されていること。
    -   **Connect With** が `Prisma` に設定されていること。
    -   **Operating System** がアプリケーションを実行するオペレーティングシステムと一致していること。

4.  まだパスワードを設定していない場合は、ランダムなパスワードを生成するために **Generate Password** をクリックします。

5.  次のコマンドを実行して、`.env.example` をコピーして `.env` に名前を変更します：

    ```shell
    cp .env.example .env
    ```

6.  `.env` ファイルを編集し、環境変数 `DATABASE_URL` を次のように設定し、接続ダイアログの接続文字列で対応するプレースホルダ `{}` を置き換えます：

    ```dotenv
    DATABASE_URL={connection_string}
    ```

    > **注意**
    >
    > TiDBサーバーレスの場合、接続ダイアログでダウンロードしたCA証明書のファイルパスを `sslcert=/path/to/ca.pem` を介して指定することで、TLS接続を有効にする必要があります。

7.  `.env` ファイルを保存します。

8.  `prisma/schema.prisma` で、`mysql` を接続プロバイダとして設定し、`env("DATABASE_URL")` を接続URLとして設定します：

    ```prisma
    datasource db {
      provider = "mysql"
      url      = env("DATABASE_URL")
    }
    ```

</div>
<div label="TiDB専用">

1.  [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの名前をクリックして概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4.  次のコマンドを実行して、`.env.example` をコピーして `.env` に名前を変更します：

    ```shell
    cp .env.example .env
    ```

5.  `.env` ファイルを編集し、環境変数 `DATABASE_URL` を次のように設定し、接続ダイアログの接続パラメータで対応するプレースホルダ `{}` を置き換えます：

    ```dotenv
    DATABASE_URL=mysql://{user}:{password}@{host}:4000/test?sslaccept=strict&sslcert={downloaded_ssl_ca_path}
    ```

    > **注意**
    >
    > TiDBサーバーレスの場合、パブリックエンドポイントを使用する場合は、`sslaccept=strict` を設定してTLS接続を有効にすることが**推奨**されます。`sslaccept=strict` を設定してTLS接続を有効にする場合、接続ダイアログからダウンロードしたCA証明書のファイルパスを `sslcert=/path/to/ca.pem` を介して指定する必要があります。

6.  `.env` ファイルを保存します。

7.  `prisma/schema.prisma` で、`mysql` を接続プロバイダとして設定し、`env("DATABASE_URL")` を接続URLとして設定します：

    ```prisma
    datasource db {
      provider = "mysql"
      url      = env("DATABASE_URL")
    }
    ```

</div>
<div label="TiDBセルフホスト">

1.  次のコマンドを実行して、`.env.example` をコピーして `.env` に名前を変更します：

    ```shell
    cp .env.example .env
    ```

2.  `.env` ファイルを編集し、環境変数 `DATABASE_URL` を次のように設定し、TiDBクラスタの接続パラメータで対応するプレースホルダ `{}` を置き換えます：

    ```dotenv
    DATABASE_URL=mysql://{user}:{password}@{host}:4000/test
    ```

    もしTiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3.  `.env` ファイルを保存します。

4.  `prisma/schema.prisma` で、`mysql` を接続プロバイダとして設定し、`env("DATABASE_URL")` を接続URLとして設定します：

    ```prisma
    datasource db {
      provider = "mysql"
      url      = env("DATABASE_URL")
    }
    ```

</div>
</SimpleTab>

### ステップ4. データベーススキーマを初期化する {#step-4-initialize-the-database-schema}

以下のコマンドを実行して、[Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate)を呼び出し、`prisma/prisma.schema`で定義されたデータモデルでデータベースを初期化してください。

```shell
npx prisma migrate dev
```

**`prisma.schema` で定義されたデータモデル:**

```prisma
// Define a Player model, which represents the `players` table.
model Player {
  id        Int      @id @default(autoincrement())
  name      String   @unique(map: "uk_player_on_name") @db.VarChar(50)
  coins     Decimal  @default(0)
  goods     Int      @default(0)
  createdAt DateTime @default(now()) @map("created_at")
  profile   Profile?

  @@map("players")
}

// Define a Profile model, which represents the `profiles` table.
model Profile {
  playerId  Int    @id @map("player_id")
  biography String @db.Text

  // Define a 1:1 relation between the `Player` and `Profile` models with foreign key.
  player    Player @relation(fields: [playerId], references: [id], onDelete: Cascade, map: "fk_profile_on_player_id")

  @@map("profiles")
}
```

Prismaでデータモデルを定義する方法を学ぶには、[Data model](https://www.prisma.io/docs/concepts/components/prisma-schema/data-model) のドキュメントを参照してください。

**期待される実行結果:**

    Your database is now in sync with your schema.

    ✔ Generated Prisma Client (5.1.1 | library) to ./node_modules/@prisma/client in 54ms

このコマンドは、`prisma/prisma.schema`に基づいてTiDBデータベースにアクセスするための[Prisma Client](https://www.prisma.io/docs/concepts/components/prisma-client)も生成します。

### ステップ5：コードを実行する {#step-5-run-the-code}

以下のコマンドを実行して、サンプルコードを実行します：

```shell
npm start
```

**サンプルコードのメインロジック：**

```typescript
// Step 1. Import the auto-generated `@prisma/client` package.
import {Player, PrismaClient} from '@prisma/client';

async function main(): Promise<void> {
  // Step 2. Create a new `PrismaClient` instance.
  const prisma = new PrismaClient();
  try {

    // Step 3. Perform some CRUD operations with Prisma Client ...

  } finally {
    // Step 4. Disconnect Prisma Client.
    await prisma.$disconnect();
  }
}

void main();
```

**実行結果の予想:**

接続が成功した場合、ターミナルはTiDBクラスターのバージョンを以下のように出力します:

    🔌 Connected to TiDB cluster! (TiDB version: 5.7.25-TiDB-v6.6.0-serverless)
    🆕 Created a new player with ID 1.
    ℹ️ Got Player 1: Player { id: 1, coins: 100, goods: 100 }
    🔢 Added 50 coins and 50 goods to player 1, now player 1 has 150 coins and 150 goods.
    🚮 Player 1 has been deleted.

## サンプルコードスニペット {#sample-code-snippets}

以下のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-prisma-quickstart](https://github.com/tidb-samples/tidb-nodejs-prisma-quickstart) リポジトリをチェックしてください。

### データの挿入 {#insert-data}

以下のクエリは単一の `Player` レコードを作成し、TiDBによって生成された `id` フィールドを含む作成された `Player` オブジェクトを返します。

```javascript
const player: Player = await prisma.player.create({
   data: {
      name: 'Alice',
      coins: 100,
      goods: 200,
      createdAt: new Date(),
   }
});
```

[データの挿入](/develop/dev-guide-insert-data.md)に関する詳細は、こちらを参照してください。

### データのクエリ {#query-data}

次のクエリは、IDが`101`の単一の`Player`オブジェクトを返します。レコードが見つからない場合は`null`を返します。

```javascript
const player: Player | null = prisma.player.findUnique({
   where: {
      id: 101,
   }
});
```

詳細については、[クエリデータ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、IDが`101`の`Player`に`50`コインと`50`商品を追加します。

```javascript
await prisma.player.update({
   where: {
      id: 101,
   },
   data: {
      coins: {
         increment: 50,
      },
      goods: {
         increment: 50,
      },
   }
});
```

[データの更新](/develop/dev-guide-update-data.md)に関する詳細は、こちらを参照してください。

### データの削除 {#delete-data}

以下のクエリは、IDが`101`の`Player`を削除します。

```javascript
await prisma.player.delete({
   where: {
      id: 101,
   }
});
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md) を参照してください。

## 便利なノート {#useful-notes}

### 外部キー制約 vs Prisma 関連モード {#foreign-key-constraints-vs-prisma-relation-mode}

[参照整合性](https://en.wikipedia.org/wiki/Referential_integrity?useskin=vector) を確認するために、外部キー制約または Prisma 関連モードを使用できます。

-   [外部キー](https://docs.pingcap.com/tidb/stable/foreign-key) は TiDB v6.6.0 からサポートされている実験的な機能で、関連するデータのクロステーブル参照とデータ整合性を維持するための外部キー制約を可能にします。

    > **警告:**
    >
    > **外部キーは小規模および中規模のデータシナリオに適しています。** 大規模なデータ量で外部キーを使用すると、重大なパフォーマンスの問題が発生する可能性があり、システムに予測できない影響を与える可能性があります。外部キーを使用する場合は、まず徹底的な検証を行い、注意して使用してください。

-   [Prisma 関連モード](https://www.prisma.io/docs/concepts/components/prisma-schema/relations/relation-mode) は Prisma Client サイドでの参照整合性のエミュレーションです。ただし、参照整合性を維持するために追加のデータベースクエリが必要なため、パフォーマンスへの影響があることに注意する必要があります。

## 次のステップ {#next-steps}

-   [Prisma のドキュメント](https://www.prisma.io/docs) から ORM フレームワーク Prisma ドライバのさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md) の章を通じて TiDB アプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md) などです。
-   プロフェッショナルな [TiDB 開発者コース](https://www.pingcap.com/education/) を通じて学び、試験に合格して [TiDB 認定資格](https://www.pingcap.com/education/certification/) を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX) で質問するか、[サポートチケットを作成](https://support.pingcap.com/) してください。
