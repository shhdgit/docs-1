---
title: Import Data into TiDB Cloud via MySQL CLI
summary: Learn how to import Data into TiDB Cloud via MySQL CLI.
---

# TiDB Cloudを使用してMySQL CLI経由でデータをインポートする {#import-data-into-tidb-cloud-via-mysql-cli}

このドキュメントでは、[MySQLコマンドラインクライアント](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)を使用してTiDB Cloudにデータをインポートする方法について説明します。SQLファイルまたはCSVファイルからデータをインポートできます。次のセクションでは、それぞれのファイルタイプからデータをインポートする手順がステップバイステップで提供されます。

## 前提条件 {#prerequisites}

TiDB CloudにMySQL CLIを介してデータをインポートする前に、次の前提条件が必要です。

- TiDB Cloudクラスタにアクセスできること。TiDBクラスタを持っていない場合は、[TiDB Serverlessクラスタを構築する](/develop/dev-guide-build-cluster-in-cloud.md)の手順に従って作成します。
- ローカルコンピュータにMySQL CLIをインストールします。

## ステップ1. TiDB Cloudクラスタに接続する {#step-1-connect-to-your-tidb-cloud-cluster}

TiDB Cloudクラスタに接続するには、選択したTiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が環境に一致することを確認します。

   - **Endpoint Type**が`Public`に設定されていること。
   - **Connect With**が`MySQL CLI`に設定されていること。
   - **Operating System**が環境に一致していること。

4. ランダムなパスワードを作成するには、**Generate Password**をクリックします。

   > **ヒント:**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するには**Reset Password**をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックします。

   接続文字列を取得する詳細については、[標準接続を介してTiDB Dedicatedに接続する](/tidb-cloud/connect-via-standard-connection.md)を参照してください。

</div>
</SimpleTab>

## ステップ2. テーブルを定義し、サンプルデータを挿入する {#step-2-define-the-table-and-insert-sample-data}

データをインポートする前に、テーブル構造を準備し、実際のサンプルデータを挿入する必要があります。以下は、テーブルを作成しサンプルデータを挿入するための例のSQLファイル（`product_data.sql`）です。

```sql
-- TiDBデータベースにテーブルを作成
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(255),
    price DECIMAL(10, 2)
);

-- テーブルにサンプルデータを挿入
INSERT INTO products (product_id, product_name, price) VALUES
    (1, 'Laptop', 999.99),
    (2, 'Smartphone', 499.99),
    (3, 'Tablet', 299.99);
```

## ステップ3. SQLファイルまたはCSVファイルからデータをインポートする {#step-3-import-data-from-a-sql-or-csv-file}

SQLファイルまたはCSVファイルからデータをインポートできます。次のセクションでは、それぞれのファイルタイプからデータをインポートする手順がステップバイステップで提供されます。

<SimpleTab>
<div label="SQLファイルから">

次の手順でSQLファイルからデータをインポートします。

1. インポートしたい実際のSQLファイル（例：`product_data.sql`）を提供します。このSQLファイルには、実際のデータを含む`INSERT`ステートメントが含まれている必要があります。

2. 次のコマンドを使用してSQLファイルからデータをインポートします。

   ```bash
   mysql --comments --connect-timeout 150 -u '<your_username>' -h <your_cluster_host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=<your_ca_path> -p <your_password> < product_data.sql
   ```

> **Note:**
>
> ここで使用されているデフォルトのデータベース名は`test`です。手動でデータベースを作成するか、SQLファイルで`CREATE DATABASE`コマンドを使用できます。

</div>
<div label="CSVファイルから">

次の手順でCSVファイルからデータをインポートします。

1. TiDBでデータのインポートに必要なデータベースとスキーマを作成します。

2. インポートしたい実際のCSVファイル（例：`product_data.csv`）を提供します。以下はCSVファイルの例です。

   **product\_data.csv:**

   ```csv
   product_id,product_name,price
   4,Laptop,999.99
   5,Smartphone,499.99
   6,Tablet,299.99
   ```

3. 次のコマンドを使用してCSVファイルからデータをインポートします。

   ```bash
   mysql --comments --connect-timeout 150 -u '<your_username>' -h <your_host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=<your_ca_path> -p<your_password> -e "LOAD DATA LOCAL INFILE '<your_csv_path>' INTO TABLE products
   FIELDS TERMINATED BY ','
   LINES TERMINATED BY '\n'
   IGNORE 1 LINES (product_id, product_name, price);"
   ```

4. パス、テーブル名（この例では`products`）、`<your_username>`、`<your_host>`、`<your_password>`、`<your_csv_path>`、`<your_ca_path>`、および他のプレースホルダを実際の情報で置き換え、必要に応じてサンプルCSVデータを実際のデータセットで置き換えてください。

> **Note:**
>
> `LOAD DATA LOCAL INFILE`の構文の詳細については、[`LOAD DATA`](/sql-statements/sql-statement-load-data.md)を参照してください。

</div>
</SimpleTab>
