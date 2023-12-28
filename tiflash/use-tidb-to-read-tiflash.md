---
title: Use TiDB to Read TiFlash Replicas
summary: Learn how to use TiDB to read TiFlash replicas.
---

# TiDBを使用してTiFlashレプリカを読み取る {#use-tidb-to-read-tiflash-replicas}

このドキュメントでは、TiDBを使用してTiFlashレプリカを読み取る方法について説明します。

TiDBには、TiFlashレプリカを読み取るための3つの方法があります。エンジンの設定を行わずにTiFlashレプリカを追加した場合、CBO（コストベースの最適化）モードがデフォルトで使用されます。

## スマート選択 {#smart-selection}

TiFlashレプリカを持つテーブルでは、TiDBオプティマイザーがコストの推定に基づいてTiFlashレプリカを使用するかどうかを自動的に決定します。`desc`または`explain analyze`ステートメントを使用して、TiFlashレプリカが選択されたかどうかを確認できます。例えば：

```sql
desc select count(*) from test.t;
```

    +--------------------------+---------+--------------+---------------+--------------------------------+
    | id                       | estRows | task         | access object | operator info                  |
    +--------------------------+---------+--------------+---------------+--------------------------------+
    | StreamAgg_9              | 1.00    | root         |               | funcs:count(1)->Column#4       |
    | └─TableReader_17         | 1.00    | root         |               | data:TableFullScan_16          |
    |   └─TableFullScan_16     | 1.00    | cop[tiflash] | table:t       | keep order:false, stats:pseudo |
    +--------------------------+---------+--------------+---------------+--------------------------------+
    3 rows in set (0.00 sec)

```sql
explain analyze select count(*) from test.t;
```

    +--------------------------+---------+---------+--------------+---------------+----------------------------------------------------------------------+--------------------------------+-----------+------+
    | id                       | estRows | actRows | task         | access object | execution info                                                       | operator info                  | memory    | disk |
    +--------------------------+---------+---------+--------------+---------------+----------------------------------------------------------------------+--------------------------------+-----------+------+
    | StreamAgg_9              | 1.00    | 1       | root         |               | time:83.8372ms, loops:2                                              | funcs:count(1)->Column#4       | 372 Bytes | N/A  |
    | └─TableReader_17         | 1.00    | 1       | root         |               | time:83.7776ms, loops:2, rpc num: 1, rpc time:83.5701ms, proc keys:0 | data:TableFullScan_16          | 152 Bytes | N/A  |
    |   └─TableFullScan_16     | 1.00    | 1       | cop[tiflash] | table:t       | tiflash_task:{time:43ms, loops:1, threads:1}, tiflash_scan:{...}     | keep order:false, stats:pseudo | N/A       | N/A  |
    +--------------------------+---------+---------+--------------+---------------+----------------------------------------------------------------------+--------------------------------+-----------+------+

`cop[TiFlash]`は、タスクが処理されるためにTiFlashに送信されることを意味します。TiFlashのレプリカを選択していない場合は、`analyze table`ステートメントを使用して統計情報を更新し、`explain analyze`ステートメントを使用して結果を確認することができます。

ただし、テーブルに単一のTiFlashレプリカしかなく、関連するノードがサービスを提供できない場合、CBOモードでのクエリは繰り返しリトライされます。このような状況では、エンジンを指定するか、手動のヒントを使用してTiKVレプリカからデータを読み取る必要があります。

## エンジンの分離 {#engine-isolation}

エンジンの分離は、対応する変数を設定することで、すべてのクエリが指定されたエンジンのレプリカを使用するように指定することです。オプションのエンジンは「tikv」、「tidb」（TiDBの内部メモリテーブル領域を示し、一部のTiDBシステムテーブルを格納し、ユーザーがアクティブに使用することはできません）、「tiflash」です。

<CustomContent platform="tidb">

次の2つの設定レベルでエンジンを指定できます。

- TiDBインスタンスレベル、つまり、INSTANCEレベル。TiDBの設定ファイルに次の設定項目を追加します。

      [isolation-read]
      engines = ["tikv", "tidb", "tiflash"]

  **INSTANCEレベルのデフォルト設定は`["tikv", "tidb", "tiflash"]`です。**

- SESSIONレベル。次のステートメントを使用して設定します。

  ```sql
  set @@session.tidb_isolation_read_engines = "カンマで区切られたエンジンのリスト";
  ```

  または

  ```sql
  set SESSION tidb_isolation_read_engines = "カンマで区切られたエンジンのリスト";
  ```

  SESSIONレベルのデフォルト設定は、TiDB INSTANCEレベルの設定を継承します。

最終的なエンジンの設定は、セッションレベルの設定であり、セッションレベルの設定はインスタンスレベルの設定を上書きします。たとえば、INSTANCEレベルで「tikv」を設定し、SESSIONレベルで「tiflash」を設定した場合、TiFlashレプリカが読み取られます。最終的なエンジンの設定が「tikv」と「tiflash」である場合、TiKVとTiFlashの両方のレプリカが読み取られ、オプティマイザーが自動的により良いエンジンを選択して実行します。

> **Note:**
>
> [TiDB Dashboard](/dashboard/dashboard-intro.md)などのコンポーネントは、TiDBメモリテーブル領域に格納されている一部のシステムテーブルを読み取る必要があるため、常にインスタンスレベルのエンジン設定に「tidb」エンジンを追加することをお勧めします。

</CustomContent>

<CustomContent platform="tidb-cloud">

次のステートメントを使用してエンジンを指定できます。

// The output content start

`cop[TiFlash]`は、タスクが処理されるためにTiFlashに送信されることを意味します。TiFlashのレプリカを選択していない場合は、`analyze table`ステートメントを使用して統計情報を更新し、`explain analyze`ステートメントを使用して結果を確認することができます。

ただし、テーブルに単一のTiFlashレプリカしかなく、関連するノードがサービスを提供できない場合、CBOモードでのクエリは繰り返しリトライされます。このような状況では、エンジンを指定するか、手動のヒントを使用してTiKVレプリカからデータを読み取る必要があります。

## エンジンの分離 {#manual-hint}

エンジンの分離は、対応する変数を設定することで、すべてのクエリが指定されたエンジンのレプリカを使用するように指定することです。オプションのエンジンは「tikv」、「tidb」（TiDBの内部メモリテーブル領域を示し、一部のTiDBシステムテーブルを格納し、ユーザーがアクティブに使用することはできません）、「tiflash」です。

<CustomContent platform="tidb">

次の2つの設定レベルでエンジンを指定できます。

- TiDBインスタンスレベル、つまり、INSTANCEレベル。TiDBの設定ファイルに次の設定項目を追加します。

      [isolation-read]
      engines = ["tikv", "tidb", "tiflash"]

  **INSTANCEレベルのデフォルト設定は`["tikv", "tidb", "tiflash"]`です。**

- SESSIONレベル。次のステートメントを使用して設定します。

  ```sql
  set @@session.tidb_isolation_read_engines = "カンマで区切られたエンジンのリスト";
  ```

  または

  ```sql
  set SESSION tidb_isolation_read_engines = "カンマで区切られたエンジンのリスト";
  ```

  SESSIONレベルのデフォルト設定は、TiDB INSTANCEレベルの設定を継承します。

最終的なエンジンの設定は、セッションレベルの設定であり、セッションレベルの設定はインスタンスレベルの設定を上書きします。たとえば、INSTANCEレベルで「tikv」を設定し、SESSIONレベルで「tiflash」を設定した場合、TiFlashレプリカが読み取られます。最終的なエンジンの設定が「tikv」と「tiflash」である場合、TiKVとTiFlashの両方のレプリカが読み取られ、オプティマイザーが自動的により良いエンジンを選択して実行します。

> **Note:**
>
> [TiDB Dashboard](/dashboard/dashboard-intro.md)などのコンポーネントは、TiDBメモリテーブル領域に格納されている一部のシステムテーブルを読み取る必要があるため、常にインスタンスレベルのエンジン設定に「tidb」エンジンを追加することをお勧めします。

</CustomContent>

<CustomContent platform="tidb-cloud">

次のステートメントを使用してエンジンを指定できます。

```sql
set @@session.tidb_isolation_read_engines = "engine list separated by commas";
```

// The output content start

また、**Note:** このコンポーネントはstorageのメモリを使用します。また、TiFlashはカラム指向のストレージを提供します。また、IはYに比べてメモリ使用量が少ないため、storageのメモリをより効率的に使用することができます。

// The output content end

```sql
set SESSION tidb_isolation_read_engines = "engine list separated by commas";
```

</CustomContent>

</CustomContent>

クエリされたテーブルに指定されたエンジンのレプリカが存在しない場合（例えば、エンジンが "tiflash" として設定されているが、テーブルに TiFlash レプリカが存在しない場合）、クエリはエラーを返します。

## マニュアルヒント {#the-relationship-of-smart-selection-engine-isolation-and-manual-hint}

マニュアルヒントを使用すると、エンジンの分離を満たす前提で、特定のテーブルに対して指定されたレプリカを TiDB が強制的に使用することができます。以下はマニュアルヒントを使用する例です：

```sql
select /*+ read_from_storage(tiflash[table_name]) */ ... from table_name;
```

クエリ文でテーブルにエイリアスを設定した場合、ヒントを含む文でそのエイリアスを使用しなければ、ヒントは効果を発揮しません。例えば：

```sql
select /*+ read_from_storage(tiflash[alias_a,alias_b]) */ ... from table_name_1 as alias_a, table_name_2 as alias_b where alias_a.column_1 = alias_b.column_2;
```

上記の文では、`tiflash[]`を使用することで、オプティマイザーにTiFlashレプリカを読み取るよう促すことができます。必要に応じて、`tikv[]`を使用してオプティマイザーにTiKVレプリカを読み取るよう促すこともできます。ヒントの構文の詳細については、[READ\_FROM\_STORAGE](/optimizer-hints.md#read_from_storagetiflasht1_name--tl_name--tikvt2_name--tl_name-)を参照してください。

ヒントで指定されたテーブルに指定されたエンジンのレプリカがない場合、ヒントは無視され、警告が報告されます。また、ヒントはエンジンの分離の前提条件でのみ有効です。ヒントで指定されたエンジンがエンジンの分離リストにない場合、ヒントは無視され、警告が報告されます。

> **Note:**
>
> 5.7.7以前のバージョンのMySQLクライアントは、デフォルトでオプティマイザーヒントをクリアします。これらの早期バージョンでヒント構文を使用するには、`--comments`オプションを使用してクライアントを起動します。例えば、`mysql -h 127.0.0.1 -P 4000 -uroot --comments`です。

## スマート選択、エンジン分離、および手動ヒントの関係

上記の3つの方法でTiFlashレプリカを読み取る場合、エンジン分離はエンジンの利用可能なレプリカの全体範囲を指定します。この範囲内で、手動ヒントはより細かい粒度のステートメントレベルおよびテーブルレベルのエンジン選択を提供します。最後に、CBOは指定されたエンジンリスト内のコスト推定に基づいて、レプリカを選択し決定します。

> **Note:**
>
> - v4.0.3より前のバージョンでは、非読み取り専用SQLステートメント（例：`INSERT INTO ... SELECT`、`SELECT ... FOR UPDATE`、`UPDATE ...`、`DELETE ...`）でのTiFlashレプリカからの読み取りの振る舞いは未定義です。
> - v4.0.3からv6.2.0までのバージョンでは、内部的にTiDBはデータの正確性を保証するため、非読み取り専用SQLステートメントのTiFlashレプリカを無視します。つまり、[スマート選択](#smart-selection)の場合、TiDBは自動的に非TiFlashレプリカを選択します。[エンジン分離](#engine-isolation)がTiFlashレプリカのみを指定する場合、TiDBはエラーを報告します。[手動ヒント](#manual-hint)の場合、TiDBはヒントを無視します。
> - v6.3.0からv7.0.0までのバージョンでは、TiFlashレプリカが有効な場合、[`tidb_enable_tiflash_read_for_write_stmt`](/system-variables.md#tidb_enable_tiflash_read_for_write_stmt-new-in-v630)変数を使用して、TiDBが非読み取り専用SQLステートメントでTiFlashレプリカを使用するかどうかを制御できます。
> - v7.1.0以降、TiFlashレプリカが有効で、現在のセッションの[SQLモード](/sql-mode.md)が厳密でない場合（つまり、`sql_mode`の値に`STRICT_TRANS_TABLES`または`STRICT_ALL_TABLES`が含まれない場合）、TiDBはコスト推定に基づいて非読み取り専用SQLステートメントでTiFlashレプリカを使用するかどうかを自動的に決定します。
