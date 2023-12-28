---
title: Slow Queries Page of TiDB Dashboard
summary: Learn the Slow Queries page of TiDB Dashboard.
---

# TiDB ダッシュボードの遅いクエリページ {#slow-queries-page-of-tidb-dashboard}

TiDB ダッシュボードの遅いクエリページでは、クラスター内のすべての遅いクエリを検索して表示することができます。

デフォルトでは、実行時間が 300 ミリ秒を超える SQL クエリは遅いクエリと見なされ、[遅いクエリログ](/identify-slow-queries.md)に記録され、TiDB ダッシュボードで検索することができます。遅いクエリの閾値は、[`tidb_slow_log_threshold`](/system-variables.md#tidb_slow_log_threshold)セッション変数または[`instance.tidb_slow_log_threshold`](/tidb-configuration-file.md#tidb_slow_log_threshold) TiDB パラメータを介して調整することができます。

> **Note:**
>
> 遅いクエリログが無効になっている場合、この機能は利用できません。遅いクエリログはデフォルトで有効になっており、システム変数[`tidb_enable_slow_log`](/system-variables.md#tidb_enable_slow_log)を介して有効または無効にすることができます。

## ページにアクセスする {#access-the-page}

遅いクエリページにアクセスするには、次のいずれかの方法を使用できます。

- TiDB ダッシュボードにログインした後、左側のナビゲーションメニューで**遅いクエリ**をクリックします。

- ブラウザで <http://127.0.0.1:2379/dashboard/#/slow_query> を訪問します。`127.0.0.1:2379`を実際の PD アドレスとポートに置き換えます。

遅いクエリページに表示されるすべてのデータは、TiDB 遅いクエリシステムテーブルと遅いクエリログから取得されます。詳細については、[遅いクエリログ](/identify-slow-queries.md)を参照してください。

### フィルターを変更する {#change-filters}

時間範囲、関連するデータベース、SQL キーワード、SQL タイプ、表示する遅いクエリの数に基づいて、遅いクエリをフィルタリングすることができます。下の画像では、デフォルトで直近 30 分間の 100 件の遅いクエリが表示されます。

![リストのフィルターを変更する](/media/dashboard/dashboard-slow-queries-list1-v620.png)

### より多くのカラムを表示する {#display-more-columns}

ページの**カラム**をクリックすると、より多くのカラムを選択することができます。カラム名の右側の\*\*(i)\*\*アイコンにマウスを移動すると、このカラムの説明を表示することができます。

![より多くのカラムを表示する](/media/dashboard/dashboard-slow-queries-list2-v620.png)

### 遅いクエリをローカルにエクスポートする {#export-slow-queries-locally}

ページの右上隅にある☰ (**その他**)をクリックすると、**エクスポート**オプションが表示されます。**エクスポート**をクリックすると、TiDB ダッシュボードは現在のリストの遅いクエリを CSV ファイルとしてエクスポートします。

![遅いクエリをローカルにエクスポートする](/media/dashboard/dashboard-slow-queries-export-v651.png)

### カラムでソートする {#sort-by-column}

デフォルトでは、リストは**終了時間**で降順にソートされます。カラムの見出しをクリックすると、カラムでソートしたり、ソート順を切り替えることができます。

![ソート基準を変更する](/media/dashboard/dashboard-slow-queries-list3-v620.png)

## 実行の詳細を表示する {#view-execution-details}

リスト内の任意のアイテムをクリックすると、遅いクエリの詳細な実行情報が表示されます。これには、次のものが含まれます。

- クエリ：SQL ステートメントのテキスト（図の領域 1）
- プラン：遅いクエリの実行プラン（図の領域 2）
- その他のソートされた SQL 実行情報（図の領域 3）

![実行の詳細を表示する](/media/dashboard/dashboard-slow-queries-detail1-v620.png)

### SQL {#sql}

> **Note:**
>
> `Query`カラムに記録されるクエリの最大長さは、[`tidb_stmt_summary_max_sql_length`](/system-variables.md#tidb_stmt_summary_max_sql_length-new-in-v40)システム変数によって制限されます。

**展開**ボタンをクリックすると、アイテムの詳細情報が表示されます。**コピー**ボタンをクリックすると、詳細情報がクリップボードにコピーされます。

### 実行プラン {#execution-plans}

TiDB ダッシュボードでは、2 つの方法で実行プランを表示することができます。グラフとテキストです。視覚的な実行プランを使用すると、ステートメントの各オペレーターと詳細情報を直感的に学習することができます。実行プランの読み方については、[クエリ実行プランを理解する](/explain-overview.md)を参照してください。

#### 視覚的な実行プラン {#visual-execution-plans}

次の図は、視覚的な実行プランを示しています。

![視覚的な実行プラン](/media/dashboard/dashboard-visual-plan-2.png)

- グラフは左から右、上から下に実行を示します。
- 上のノードは親オペレーターであり、下のノードは子オペレーターです。
- タイトルバーの色は、オペレーターが実行されるコンポーネントを示します。黄色は TiDB、青は TiKV、ピンクは TiFlash を表します。
- タイトルバーにはオペレーター名が表示され、下に表示されるテキストはオペレーターの基本情報です。

ノード領域をクリックすると、右側のサイドバーに詳細なオペレーター情報が表示されます。

![視覚的な実行プラン - サイドバー](/media/dashboard/dashboard-visual-plan-popup.png)

### SQL 実行の詳細 {#sql-execution-details}

対応するタブのタイトルをクリックすると、SQL 実行の情報を切り替えることができます。

![異なる実行情報を表示する](/media/dashboard/dashboard-slow-queries-detail2-v620.png)
