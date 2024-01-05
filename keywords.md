---
title: Keywords
summary: Keywords and Reserved Words
---

# キーワード {#keywords}

この記事では、TiDBのキーワード、予約語と非予約語の違い、およびクエリのすべてのキーワードについてまとめて紹介します。

キーワードとは、SQLステートメントで特別な意味を持つ単語のことです。例えば、`SELECT`、`UPDATE`、`DELETE`などがあります。そのうちのいくつかは直接識別子として使用できる非予約キーワードと呼ばれます。その他のいくつかは、識別子として使用する前に特別な処理が必要な予約キーワードと呼ばれます。ただし、特別な非予約キーワードも、特別な処理が必要な場合があります。これらは予約キーワードとして扱うことをお勧めします。

予約キーワードを識別子として使用するには、バッククォート `` ` `` で囲む必要があります。

```sql
CREATE TABLE select (a INT);
```

```
ERROR 1105 (HY000): line 0 column 19 near " (a INT)" (total length 27)
```

```sql
CREATE TABLE `select` (a INT);
```

```
Query OK, 0 rows affected (0.09 sec)
```

非予約キーワードはバッククォートを必要としません。たとえば、`BEGIN`や`END`などは、次の文で識別子として正常に使用できます。

```sql
CREATE TABLE `select` (BEGIN int, END int);
```

```
Query OK, 0 rows affected (0.09 sec)
```

特別な場合、予約語は、`.`デリミタと一緒に使用される場合、バッククォートは必要ありません。

```sql
CREATE TABLE test.select (BEGIN int, END int);
```

```
Query OK, 0 rows affected (0.08 sec)
```

## キーワードリスト {#keyword-list}

以下のリストは、TiDBのキーワードを示しています。予約キーワードは`(R)`でマークされています。[ウィンドウ関数](/functions-and-operators/window-functions.md)の予約キーワードは、`(R-Window)`でマークされています。バッククォート `` ` `` でエスケープする必要のある特別な非予約キーワードは、`(S)`でマークされています。

<TabsPanel letters="ABCDEFGHIJKLMNOPQRSTUVWXYZ" />

<a id="A" class="letter" href="#A">A</a>

- ACCOUNT
- ACTION
- ADD (R)
- ADMIN
- ADVISE
- AFTER
- AGAINST
- AGO
- ALGORITHM
- ALL (R)
- ALTER (R)
- ALWAYS
- ANALYZE (R)
- AND (R)
- ANY
- ARRAY (R)
- AS (R)
- ASC (R)
- ASCII
- ATTRIBUTE
- ATTRIBUTES
- AUTO\_ID\_CACHE
- AUTO\_INCREMENT
- AUTO\_RANDOM
- AUTO\_RANDOM\_BASE
- AVG
- AVG\_ROW\_LENGTH

<a id="B" class="letter" href="#B">B</a>

- BACKEND
- BACKUP
- BACKUPS
- BATCH
- BDR
- BEGIN
- BERNOULLI
- BETWEEN (R)
- BIGINT (R)
- BINARY (R)
- BINDING
- BINDINGS
- BINDING\_CACHE
- BINLOG
- BIT
- BLOB (R)
- BLOCK
- BOOL
- BOOLEAN
- BOTH (R)
- BTREE
- BUCKETS
- BUILTINS
- BY (R)
- BYTE

<a id="C" class="letter" href="#C">C</a>

- CACHE
- CALIBRATE
- CALL (R)
- CANCEL
- CAPTURE
- CARDINALITY
- CASCADE (R)
- CASCADED
- CASE (R)
- CAUSAL
- CHAIN
- CHANGE (R)
- CHAR (R)
- CHARACTER (R)
- CHARSET
- CHECK (R)
- CHECKPOINT
- CHECKSUM
- CIPHER
- CLEANUP
- CLIENT
- CLIENT\_ERRORS\_SUMMARY
- CLOSE
- CLUSTER
- CLUSTERED
- CMSKETCH
- COALESCE
- COLLATE (R)
- COLLATION
- COLUMN (R)
- COLUMN\_FORMAT
- COLUMN\_STATS\_USAGE
- COLUMNS
- COMMENT
- COMMIT
- COMMITTED
- COMPACT
- COMPRESSED
- COMPRESSION
- CONCURRENCY
- CONFIG
- CONNECTION
- CONSISTENCY
- CONSISTENT
- CONSTRAINT (R)
- CONTEXT
- CONTINUE (R)
- CONVERT (R)
- CORRELATION
- CPU
- CREATE (R)
- CROSS (R)
- CSV\_BACKSLASH\_ESCAPE
- CSV\_DELIMITER
- CSV\_HEADER
- CSV\_NOT\_NULL
- CSV\_NULL
- CSV\_SEPARATOR
- CSV\_TRIM\_LAST\_SEPARATORS
- CUME\_DIST (R-Window)
- CURRENT
- CURRENT\_DATE (R)
- CURRENT\_ROLE (R)
- CURRENT\_TIME (R)
- CURRENT\_TIMESTAMP (R)
- CURRENT\_USER (R)
- CURSOR (R)
- CYCLE

<a id="D" class="letter" href="#D">D</a>

- DATA
- DATABASE (R)
- DATABASES (R)
- DATE
- DATETIME
- DAY
- DAY\_HOUR (R)
- DAY\_MICROSECOND (R)
- DAY\_MINUTE (R)
- DAY\_SECOND (R)
- DDL
- DEALLOCATE
- DECIMAL (R)
- DECLARE
- DEFAULT (R)
- DEFINER
- DELAY\_KEY\_WRITE
- DELAYED (R)
- DELETE (R)
- DENSE\_RANK (R-Window)
- DEPENDENCY
- DEPTH
- DESC (R)
- DESCRIBE (R)
- DIGEST
- DIRECTORY
- DISABLE
- DISABLED
- DISCARD
- DISK
- DISTINCT (R)
- DISTINCTROW (R)
- DIV (R)
- DO
- DOUBLE (R)
- DRAINER
- DROP (R)
- DRY
- DUAL (R)
- DUPLICATE
- DYNAMIC

<a id="E" class="letter" href="#E">E</a>

- ELSE (R)
- ELSEIF (R)
- ENABLE
- ENABLED
- ENCLOSED (R)
- ENCRYPTION
- END
- ENFORCED
- ENGINE
- ENGINES
- ENUM
- ERROR
- ERRORS
- ESCAPE
- ESCAPED (R)
- EVENT
- EVENTS
- EVOLVE
- EXCEPT (R)
- EXCHANGE
- EXCLUSIVE
- EXECUTE
- EXISTS (R)
- EXIT (R)
- EXPANSION
- EXPIRE
- EXPLAIN (R)
- EXTENDED

<a id="F" class="letter" href="#F">F</a>

- FAILED\_LOGIN\_ATTEMPTS
- FALSE (R)
- FAULTS
- FETCH (R)
- FIELDS
- FILE
- FIRST
- FIRST\_VALUE (R-Window)
- FIXED
- FLOAT (R)
- FLUSH
- FOLLOWING
- FOR (R)
- FORCE (R)
- FOREIGN (R)
- FORMAT
- FOUND
- FROM (R)
- FULL
- FULLTEXT (R)
- 関数

<a id="G" class="letter" href="#G">G</a>

- GENERAL
- GENERATED (R)
- GLOBAL
- GRANT (R)
- GRANTS
- GROUP (R)
- GROUPS (R-Window)

<a id="H" class="letter" href="#H">H</a>

- HANDLER
- HASH
- HAVING (R)
- HELP
- HIGH\_PRIORITY (R)
- HISTOGRAM
- HISTOGRAMS\_IN\_FLIGHT
- HISTORY
- HOSTS
- HOUR
- HOUR\_MICROSECOND (R)
- HOUR\_MINUTE (R)
- HOUR\_SECOND (R)

<a id="I" class="letter" href="#I">I</a>

- IDENTIFIED
- IF (R)
- IGNORE (R)
- ILIKE (R)
- IMPORT
- IMPORTS
- IN (R)
- INCREMENT
- INCREMENTAL
- INDEX (R)
- INDEXES
- INFILE (R)
- INNER (R)
- INOUT (R)
- INSERT (R)
- INSERT\_METHOD
- INSTANCE
- INT (R)
- INT1 (R)
- INT2 (R)
- INT3 (R)
- INT4 (R)
- INT8 (R)
- INTEGER (R)
- INTERSECT (R)
- INTERVAL (R)
- INTO (R)
- INVISIBLE
- INVOKER
- IO
- IPC
- IS (R)
- ISOLATION
- ISSUER
- ITERATE (R)

<a id="J" class="letter" href="#J">J</a>

- JOB
- JOBS
- JOIN (R)
- JSON

<a id="K" class="letter" href="#K">K</a>

- KEY (R)
- KEYS (R)
- KEY\_BLOCK\_SIZE
- KILL (R)

<a id="L" class="letter" href="#L">L</a>

- LABELS
- LAG (R-Window)
- LANGUAGE
- LAST
- LAST\_BACKUP
- LAST\_VALUE (R-Window)
- LASTVAL
- LEAD (R-Window)
- LEADING (R)
- LEAVE (R)
- LEFT (R)
- LESS
- LEVEL
- LIKE (R)
- LIMIT (R)
- LINEAR (R)
- LINES (R)
- LIST
- LOAD (R)
- LOCAL
- LOCAL\_ONLY
- LOCALTIME (R)
- LOCALTIMESTAMP (R)
- LOCATION
- LOCK (R)
- LOCKED
- LOGS
- LONG (R)
- LONGBLOB (R)
- LONGTEXT (R)
- LOW\_PRIORITY (R)

<a id="M" class="letter" href="#M">M</a>

- MASTER
- MATCH (R)
- MAXVALUE (R)
- MAX\_CONNECTIONS\_PER\_HOUR
- MAX\_IDXNUM
- MAX\_MINUTES
- MAX\_QUERIES\_PER\_HOUR
- MAX\_ROWS
- MAX\_UPDATES\_PER\_HOUR
- MAX\_USER\_CONNECTIONS
- MB
- MEDIUMBLOB (R)
- MEDIUMINT (R)
- MEDIUMTEXT (R)
- MEMBER
- MEMORY
- MERGE
- MICROSECOND
- MINUTE
- MINUTE\_MICROSECOND (R)
- MINUTE\_SECOND (R)
- MINVALUE
- MIN\_ROWS
- MOD (R)
- MODE
- MODIFY
- MONTH

<a id="N" class="letter" href="#N">N</a>

- 名前
- 国民
- 自然（R）
- NCHAR
- 決して
- 次
- 次の値
- いいえ
- NOCACHE
- NOCYCLE
- NODEGROUP
- NODE\_ID
- NODE\_STATE
- NOMAXVALUE
- NOMINVALUE
- NONCLUSTERED
- なし
- NOT（R）
- NOWAIT
- NO\_WRITE\_TO\_BINLOG（R）
- NTH\_VALUE（R-Window）
- NTILE（R-Window）
- NULL（R）
- NULLS
- 数値（R）
- NVARCHAR

<a id="O" class="letter" href="#O">O</a>

- OF（R）
- オフ
- オフセット
- OLTP\_READ\_ONLY
- OLTP\_READ\_WRITE
- OLTP\_WRITE\_ONLY
- ON（R）
- ON\_DUPLICATE
- オンライン
- のみ
- 開く
- 楽観的
- 最適化（R）
- オプション（R）
- オプション
- オプションで（R）
- OR（R）
- 注文（R）
- OUT（R）
- OUTER（R）
- OUTFILE（R）
- OVER（R-Window）

<a id="P" class="letter" href="#P">P</a>

- PACK\_KEYS
- PAGE
- PARSER
- 部分的
- PARTITION（R）
- PARTITIONING
- PARTITIONS
- PASSWORD
- PASSWORD\_LOCK\_TIME
- 一時停止
- パーセント
- PERCENT\_RANK（R-Window）
- PER\_DB
- PER\_TABLE
- 悲観的
- 配置（S）
- プラグイン
- ポイント
- ポリシー
- 先行
- 精度（R）
- 準備
- 保存
- PRE\_SPLIT\_REGIONS
- PRIMARY（R）
- 特権
- PROCEDURE（R）
- プロセス
- PROCESSLIST
- プロファイル
- プロファイル
- 代理
- ポンプ
- 廃棄

<a id="Q" class="letter" href="#Q">Q</a>

- 四半期
- クエリ
- クエリ
- クイック

<a id="R" class="letter" href="#R">R</a>

- RANGE（R）
- RANK（R-Window）
- RATE\_LIMIT
- READ（R）
- REAL（R）
- 再構築
- 回復
- 再帰的（R）
- 冗長
- 参照（R）
- 正規表現（R）
- 地域
- 地域
- リリース（R）
- リロード
- 削除
- 名前を変更（R）
- 再編成
- 修理
- 繰り返す（R）
- REPEATABLE
- 置換（R）
- レプリカ
- レプリカ
- レプリケーション
- 必要（R）
- 必須
- リセット
- リソース
- 尊重
- 再起動
- 復元
- 復元
- 制限（R）
- 再開
- 再利用
- 逆
- 取り消し（R）
- 右（R）
- RLIKE（R）
- 役割
- ロールバック
- ルーチン
- 行（R）
- 行数
- ROW\_FORMAT
- ROW\_NUMBER（R-Window）
- ROWS（R-Window）
- RTREE
- 実行

<a id="S" class="letter" href="#S">S</a>

- サンプルレート
- サンプル
- SAN
- セーブポイント
- セカンド
- SECOND\_MICROSECOND（R）
- セカンダリ
- SECONDARY\_ENGINE
- SECONDARY\_LOAD
- SECONDARY\_UNLOAD
- セキュリティ
- 選択（R）
- SEND\_CREDENTIALS\_TO\_TIKV
- セパレータ
- シーケンス
- シリアル
- シリアライズ可能
- セッション
- セッション状態
- SET（R）
- SETVAL
- SHARD\_ROW\_ID\_BITS
- 共有
- 共有
- 表示（R）
- シャットダウン
- 署名済み
- シンプル
- スキップ
- SKIP\_SCHEMA\_FILES
- スレーブ
- 遅い
- SMALLINT（R）
- スナップショット
- いくつか
- ソース
- 空間（R）
- 分割
- SQL（R）
- SQL\_BIG\_RESULT（R）
- SQL\_BUFFER\_RESULT
- SQL\_CACHE
- SQL\_CALC\_FOUND\_ROWS（R）
- SQL\_NO\_CACHE
- SQL\_SMALL\_RESULT（R）
- SQL\_TSI\_DAY
- SQL\_TSI\_HOUR
- SQL\_TSI\_MINUTE
- SQL\_TSI\_MONTH
- SQL\_TSI\_QUARTER
- SQL\_TSI\_SECOND
- SQL\_TSI\_WEEK
- SQL\_TSI\_YEAR
- SQLEXCEPTION（R）
- SQLSTATE（R）
- SQLWARNING（R）
- SSL（R）
- 開始
- 開始（R）
- 統計
- 統計
- STATS\_AUTO\_RECALC
- STATS\_BUCKETS
- STATS\_COL\_CHOICE
- STATS\_COL\_LIST
- STATS\_EXTENDED（R）
- STATS\_HEALTHY
- STATS\_HISTOGRAMS
- STATS\_LOCKED
- STATS\_META
- STATS\_OPTIONS
- STATS\_PERSISTENT
- STATS\_SAMPLE\_PAGES
- STATS\_SAMPLE\_RATE
- STATS\_TOPN
- ステータス
- ストレージ
- 格納済み（R）
- STRAIGHT\_JOIN（R）
- 厳密なフォーマット
- 主題
- サブパーティション
- サブパーティション
- スーパー
- スワップ
- スイッチ
- システム
- システム時間

<a id="T" class="letter" href="#T">T</a>

- テーブル（R）
- テーブル
- TABLESAMPLE（R）
- TABLESPACE
- TABLE\_CHECKSUM
- TELEMETRY
- TELEMETRY\_ID
- 一時的
- 一時テーブル
- 終了（R）
- テキスト
- よりも
- その後（R）
- TIDB
- TiDB\_CURRENT\_TSO（R）
- TIFLASH
- TIKV\_IMPORTER
- 時間
- タイムスタンプ
- TINYBLOB（R）
- TINYINT（R）
- TINYTEXT（R）
- TO（R）
- TOKEN\_ISSUER
- TOPN
- TPCC
- トレース
- 伝統的
- トレイリング（R）
- トランザクション
- トリガー（R）
- トリガー
- TRUE（R）
- 切り捨て
- TSO
- TTL
- TTL\_ENABLE
- TTL\_JOB\_INTERVAL
- タイプ

<a id="U" class="letter" href="#U">U</a>

- UNBOUNDED
- UNCOMMITTED
- UNDEFINED
- UNICODE
- UNION（R）
- UNIQUE（R）
- UNKNOWN
- UNLOCK（R）
- UNSIGNED（R）
- UNTIL（R）
- UPDATE（R）
- USAGE（R）
- USE（R）
- USER
- USING（R）
- UTC\_DATE（R）
- UTC\_TIME（R）
- UTC\_TIMESTAMP（R）

<a id="V" class="letter" href="#V">V</a>

- 検証
- 値
- VALUES（R）
- VARBINARY（R）
- VARCHAR（R）
- VARCHARACTER（R）
- 変数
- VARYING（R）
- VIEW
- VIRTUAL（R）
- VISIBLE

<a id="W" class="letter" href="#W">W</a>

- 待つ
- 警告
- 週
- WEIGHT\_STRING
- WHEN（R）
- WHERE（R）
- WHILE（R）
- 幅
- WINDOW（R-Window）
- WITH（R）
- WITHOUT
- WORKLOAD
- WRITE（R）

<a id="X" class="letter" href="#X">X</a>

- X509
- XOR（R）

<a id="Y" class="letter" href="#Y">Y</a>

- 年
- YEAR\_MONTH（R）

<a id="Z" class="letter" href="#Z">Z</a>

- ZEROFILL（R）"
