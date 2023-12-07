---
title: Connect to TiDB with TypeORM
summary: Learn how to connect to TiDB using TypeORM. This tutorial gives Node.js sample code snippets that work with TiDB using TypeORM.
---

# TypeORMを使用してTiDBに接続する {#connect-to-tidb-with-typeorm}

TiDBはMySQL互換のデータベースであり、[TypeORM](https://github.com/TypeORM/TypeORM)はNode.js向けの人気のあるオープンソースのORMフレームワークです。

このチュートリアルでは、TiDBとTypeORMを使用して次のタスクを実行する方法を学ぶことができます:

-   環境をセットアップする。
-   TypeORMを使用してTiDBクラスターに接続する。
-   アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です:

-   あなたのマシンにインストールされている[Node.js](https://nodejs.org/en) >= 16.x。
-   あなたのマシンにインストールされている[Git](https://git-scm.com/downloads)。
-   実行中のTiDBクラスター。

**TiDBクラスターがない場合は、次のように作成できます:**

<CustomContent platform="tidb">

-   (推奨) [TiDB Serverlessクラスターを作成する](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスターを作成します。
-   [ローカルテストTiDBクラスターをデプロイする](/quick-start-with-tidb.md#deploy-a-local-test-cluster)か[本番用TiDBクラスターをデプロイする](/production-deployment-using-tiup.md)を参照して、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

-   (推奨) [TiDB Serverlessクラスターを作成する](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスターを作成します。
-   [ローカルテストTiDBクラスターをデプロイする](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)か[本番用TiDBクラスターをデプロイする](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を参照して、ローカルクラスターを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-nodejs-typeorm-quickstart.git
cd tidb-nodejs-typeorm-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

以下のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`typeorm`および`mysql2`を含む）をインストールしてください：

```shell
npm install
```

<details>
<summary><b>既存のプロジェクトに依存関係をインストールする</b></summary>

既存のプロジェクトに対して、以下のコマンドを実行してパッケージをインストールしてください:

-   `typeorm`: Node.js向けのORMフレームワーク。
-   `mysql2`: Node.js向けのMySQLドライバ。`mysql`ドライバも利用できます。
-   `dotenv`: `.env`ファイルから環境変数を読み込みます。
-   `typescript`: TypeScriptコードをJavaScriptにコンパイルします。
-   `ts-node`: コンパイルせずに直接TypeScriptコードを実行します。
-   `@types/node`: Node.jsのTypeScript型定義を提供します。

```shell
npm install typeorm mysql2 dotenv --save
npm install @types/node ts-node typescript --save-dev
```

</details>

### ステップ3：接続情報を構成する {#step-3-configure-connection-information}

TiDBクラスタに接続し、選択したTiDBデプロイメントオプションに応じて設定します。

<SimpleTab>
<div label="TiDB Serverless">

1.  [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成が操作環境に一致することを確認します。

    -   **エンドポイントタイプ** が `Public` に設定されていること。
    -   **Branch** が `main` に設定されていること。
    -   **Connect With** が `General` に設定されていること。
    -   **Operating System** がアプリケーションを実行するオペレーティングシステムと一致していること。

4.  まだパスワードを設定していない場合は、ランダムなパスワードを生成するために **Generate Password** をクリックします。

5.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

6.  `.env` ファイルを編集し、接続ダイアログの接続パラメータで対応するプレースホルダ `{}` を置き換えて、環境変数を次のように設定します：

    ```dotenv
    TIDB_HOST={host}
    TIDB_PORT=4000
    TIDB_USER={user}
    TIDB_PASSWORD={password}
    TIDB_DATABASE=test
    TIDB_ENABLE_SSL=true
    ```

    > **注意**
    >
    > TiDB Serverlessの場合、パブリックエンドポイントを使用する場合は、`TIDB_ENABLE_SSL` を介してTLS接続を有効にする必要があります。

7.  `.env` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1.  [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

5.  `.env` ファイルを編集し、接続ダイアログの接続パラメータで対応するプレースホルダ `{}` を置き換えて、環境変数を次のように設定します：

    ```dotenv
    TIDB_HOST={host}
    TIDB_PORT=4000
    TIDB_USER={user}
    TIDB_PASSWORD={password}
    TIDB_DATABASE=test
    TIDB_ENABLE_SSL=true
    TIDB_CA_PATH={downloaded_ssl_ca_path}
    ```

    > **注意**
    >
    > TiDB Dedicatedの場合、パブリックエンドポイントを使用する場合は、`TIDB_ENABLE_SSL` を介してTLS接続を有効にすることが**推奨**されます。`TIDB_ENABLE_SSL=true` を設定する場合は、接続ダイアログからダウンロードしたCA証明書のパスを `TIDB_CA_PATH=/path/to/ca.pem` で指定する必要があります。

6.  `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1.  次のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします：

    ```shell
    cp .env.example .env
    ```

2.  `.env` ファイルを編集し、TiDBクラスタの接続パラメータで対応するプレースホルダ `{}` を置き換えて、環境変数を次のように設定します：

    ```dotenv
    TIDB_HOST={host}
    TIDB_PORT=4000
    TIDB_USER=root
    TIDB_PASSWORD={password}
    TIDB_DATABASE=test
    ```

    もしTiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3.  `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：データベーススキーマを初期化する {#step-4-initialize-the-database-schema}

次のコマンドを実行して、TypeORM CLIを呼び出し、`src/migrations` フォルダに記述されたSQLステートメントでデータベースを初期化します：

```shell
npm run migration:run
```

<details>
<summary><b>期待される実行結果</b></summary>

以下のSQLステートメントは、`players`テーブルと`profiles`テーブルを作成し、そして2つのテーブルは外部キーを介して関連付けられています。

```sql
query: SELECT VERSION() AS `version`
query: SELECT * FROM `INFORMATION_SCHEMA`.`COLUMNS` WHERE `TABLE_SCHEMA` = 'test' AND `TABLE_NAME` = 'migrations'
query: CREATE TABLE `migrations` (`id` int NOT NULL AUTO_INCREMENT, `timestamp` bigint NOT NULL, `name` varchar(255) NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB
query: SELECT * FROM `test`.`migrations` `migrations` ORDER BY `id` DESC
0 migrations are already loaded in the database.
1 migrations were found in the source code.
1 migrations are new migrations must be executed.
query: START TRANSACTION
query: CREATE TABLE `profiles` (`player_id` int NOT NULL, `biography` text NOT NULL, PRIMARY KEY (`player_id`)) ENGINE=InnoDB
query: CREATE TABLE `players` (`id` int NOT NULL AUTO_INCREMENT, `name` varchar(50) NOT NULL, `coins` decimal NOT NULL, `goods` int NOT NULL, `created_at` datetime NOT NULL, `profilePlayerId` int NULL, UNIQUE INDEX `uk_players_on_name` (`name`), UNIQUE INDEX `REL_b9666644b90ccc5065993425ef` (`profilePlayerId`), PRIMARY KEY (`id`)) ENGINE=InnoDB
query: ALTER TABLE `players` ADD CONSTRAINT `fk_profiles_on_player_id` FOREIGN KEY (`profilePlayerId`) REFERENCES `profiles`(`player_id`) ON DELETE NO ACTION ON UPDATE NO ACTION
query: INSERT INTO `test`.`migrations`(`timestamp`, `name`) VALUES (?, ?) -- PARAMETERS: [1693814724825,"Init1693814724825"]
Migration Init1693814724825 has been  executed successfully.
query: COMMIT
```

</details>

マイグレーションファイルは、`src/entities`フォルダで定義されたエンティティから生成されます。TypeORMでエンティティを定義する方法については、[TypeORM: Entities](https://typeorm.io/entities)を参照してください。

### ステップ5：コードを実行して結果を確認する {#step-5-run-the-code-and-check-the-result}

次のコマンドを実行して、サンプルコードを実行します：

```shell
npm start
```

**実行結果の期待値：**

接続が成功した場合、ターミナルはTiDBクラスタのバージョンを以下のように出力します：

    🔌 Connected to TiDB cluster! (TiDB version: 5.7.25-TiDB-v7.1.0)
    🆕 Created a new player with ID 2.
    ℹ️ Got Player 2: Player { id: 2, coins: 100, goods: 100 }
    🔢 Added 50 coins and 50 goods to player 2, now player 2 has 100 coins and 150 goods.
    🚮 Deleted 1 player data.

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-typeorm-quickstart](https://github.com/tidb-samples/tidb-nodejs-typeorm-quickstart) リポジトリをチェックしてください。

### 接続オプションを使用して接続する {#connect-with-connection-options}

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続を確立します。

```typescript
// src/dataSource.ts

// Load environment variables from .env file to process.env.
require('dotenv').config();

export const AppDataSource = new DataSource({
  type: "mysql",
  host: process.env.TIDB_HOST || '127.0.0.1',
  port: process.env.TIDB_PORT ? Number(process.env.TIDB_PORT) : 4000,
  username: process.env.TIDB_USER || 'root',
  password: process.env.TIDB_PASSWORD || '',
  database: process.env.TIDB_DATABASE || 'test',
  ssl: process.env.TIDB_ENABLE_SSL === 'true' ? {
    minVersion: 'TLSv1.2',
    ca: process.env.TIDB_CA_PATH ? fs.readFileSync(process.env.TIDB_CA_PATH) : undefined
  } : null,
  synchronize: process.env.NODE_ENV === 'development',
  logging: false,
  entities: [Player, Profile],
  migrations: [__dirname + "/migrations/**/*{.ts,.js}"],
});
```

**注意**

TiDB Serverlessを使用する場合、パブリックエンドポイントを使用する際にはTLS接続を有効にする必要があります。このサンプルコードでは、`.env`ファイルで環境変数`TIDB_ENABLE_SSL`を`true`に設定してください。

ただし、デフォルトでNode.jsは組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、`TIDB_CA_PATH`を介してSSL CA証明書を指定する必要はありません。この証明書はTiDB Serverlessによって信頼されています。

### データの挿入 {#insert-data}

以下のクエリは単一の`Player`レコードを作成し、TiDBによって生成された`id`フィールドを含む作成された`Player`オブジェクトを返します。

```typescript
const player = new Player('Alice', 100, 100);
await this.dataSource.manager.save(player);
```

[Insert data](/develop/dev-guide-insert-data.md) を参照してください。

### データのクエリ {#query-data}

次のクエリは、IDが101の単一の `Player` オブジェクトを返します。レコードが見つからない場合は `null` を返します。

```typescript
const player: Player | null = await this.dataSource.manager.findOneBy(Player, {
  id: id
});
```

詳細については、[クエリデータ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

次のクエリは、IDが`101`の`Player`に`50`の商品を追加します。

```typescript
const player = await this.dataSource.manager.findOneBy(Player, {
  id: 101
});
player.goods += 50;
await this.dataSource.manager.save(player);
```

[データの更新](/develop/dev-guide-update-data.md)に関する詳細は、こちらを参照してください。

### データの削除 {#delete-data}

以下のクエリは、IDが`101`の`Player`を削除します。

```typescript
await this.dataSource.manager.delete(Player, {
  id: 101
});
```

[データの削除](/develop/dev-guide-delete-data.md)に関する詳細は、詳細情報を参照してください。

### 生のSQLクエリの実行 {#execute-raw-sql-queries}

次のクエリは生のSQLステートメント（`SELECT VERSION() AS tidb_version;`）を実行し、TiDBクラスタのバージョンを返します。

```typescript
const rows = await dataSource.query('SELECT VERSION() AS tidb_version;');
console.log(rows[0]['tidb_version']);
```

For more information, refer to [TypeORM: DataSource API](https://typeorm.io/data-source-api).

## 有用なノート {#useful-notes}

### 外部キー制約 {#foreign-key-constraints}

[外部キー制約](https://docs.pingcap.com/tidb/stable/foreign-key)（実験的）を使用すると、データベース側でチェックを追加することでデータの[参照整合性](https://en.wikipedia.org/wiki/Referential_integrity)を確保できます。ただし、これは大容量のデータのシナリオで深刻なパフォーマンスの問題を引き起こす可能性があります。

`createForeignKeyConstraints`オプション（デフォルト値は`true`）を使用することで、エンティティ間の関係を構築する際に外部キー制約が作成されるかどうかを制御できます。

```typescript
@Entity()
export class ActionLog {
    @PrimaryColumn()
    id: number

    @ManyToOne((type) => Person, {
        createForeignKeyConstraints: false,
    })
    person: Person
}
```

For more information, refer to the [TypeORM FAQ](https://typeorm.io/relations-faq#avoid-foreign-key-constraint-creation) and [Foreign key constraints](https://docs.pingcap.com/tidbcloud/foreign-key#foreign-key-constraints).

## 次のステップ {#next-steps}

-   [TypeORMのドキュメント](https://typeorm.io/)からTypeORMのさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)など。
-   [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
