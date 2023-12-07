---
title: Keywords
summary: Keywords and Reserved Words
---

# キーワード {#keywords}

この記事では、TiDBのキーワード、予約語と非予約語の違い、およびクエリのすべてのキーワードについてまとめて紹介します。

キーワードとは、SQLステートメントで特別な意味を持つ単語のことです。例えば、`SELECT`、`UPDATE`、`DELETE`などがあります。これらのうち、**非予約キーワード**と呼ばれる識別子として直接使用できるものと、特別な処理が必要な**予約キーワード**と呼ばれるものがあります。ただし、特別な処理が必要な非予約キーワードもあります。これらは予約キーワードとして扱うことをお勧めします。

予約キーワードを識別子として使用するには、バッククォート `` ` `` で囲む必要があります。

```sql
CREATE TABLE select (a INT);
```

    ERROR 1105 (HY000): line 0 column 19 near " (a INT)" (total length 27)

```sql
CREATE TABLE `select` (a INT);
```

    Query OK, 0 rows affected (0.09 sec)

非予約キーワードはバッククォートを必要としません。例えば `BEGIN` や `END` のようなキーワードは、次の文で識別子として成功裏に使用できます。

```sql
CREATE TABLE `select` (BEGIN int, END int);
```

    Query OK, 0 rows affected (0.09 sec)

特別な場合、予約キーワードは `.` デリミタと一緒に使用される場合、バッククォートは必要ありません。

```sql
CREATE TABLE test.select (BEGIN int, END int);
```

    Query OK, 0 rows affected (0.08 sec)

## キーワードリスト {#keyword-list}

以下のリストはTiDBのキーワードを示しています。 予約済みキーワードは`(R)`でマークされています。 [Window Functions](/functions-and-operators/window-functions.md)の予約済みキーワードは`(R-Window)`でマークされています。 バッククォート`` ` ``でエスケープする必要がある特別な非予約キーワードは`(S)`でマークされています。

<TabsPanel letters="ABCDEFGHIJKLMNOPQRSTUVWXYZ" />

<a id="A" class="letter" href="#A">A</a>

-   ACCOUNT
-   ACTION
-   ADD (R)
-   ADMIN
-   ADVISE
-   AFTER
-   AGAINST
-   AGO
-   ALGORITHM
-   ALL (R)
-   ALTER (R)
-   ALWAYS
-   ANALYZE (R)
-   AND (R)
-   ANY
-   ARRAY (R)
-   AS (R)
-   ASC (R)
-   ASCII
-   ATTRIBUTE
-   ATTRIBUTES
-   AUTO\_ID\_CACHE
-   AUTO\_INCREMENT
-   AUTO\_RANDOM
-   AUTO\_RANDOM\_BASE
-   AVG
-   AVG\_ROW\_LENGTH

<a id="B" class="letter" href="#B">B</a>

-   BACKEND
-   BACKUP
-   BACKUPS
-   BATCH
-   BDR
-   BEGIN
-   BERNOULLI
-   BETWEEN (R)
-   BIGINT (R)
-   BINARY (R)
-   BINDING
-   BINDINGS
-   BINDING\_CACHE
-   BINLOG
-   BIT
-   BLOB (R)
-   BLOCK
-   BOOL
-   BOOLEAN
-   BOTH (R)
-   BTREE
-   BUCKETS
-   BUILTINS
-   BY (R)
-   BYTE

<a id="C" class="letter" href="#C">C</a>

-   CACHE
-   CALIBRATE
-   CALL (R)
-   CANCEL
-   CAPTURE
-   CARDINALITY
-   CASCADE (R)
-   CASCADED
-   CASE (R)
-   CAUSAL
-   CHAIN
-   CHANGE (R)
-   CHAR (R)
-   CHARACTER (R)
-   CHARSET
-   CHECK (R)
-   CHECKPOINT
-   CHECKSUM
-   CIPHER
-   CLEANUP
-   CLIENT
-   CLIENT\_ERRORS\_SUMMARY
-   CLOSE
-   CLUSTER
-   CLUSTERED
-   CMSKETCH
-   COALESCE
-   COLLATE (R)
-   COLLATION
-   COLUMN (R)
-   COLUMN\_FORMAT
-   COLUMN\_STATS\_USAGE
-   COLUMNS
-   COMMENT
-   COMMIT
-   COMMITTED
-   COMPACT
-   COMPRESSED
-   COMPRESSION
-   CONCURRENCY
-   CONFIG
-   CONNECTION
-   CONSISTENCY
-   CONSISTENT
-   CONSTRAINT (R)
-   CONTEXT
-   CONTINUE (R)
-   CONVERT (R)
-   CORRELATION
-   CPU
-   CREATE (R)
-   CROSS (R)
-   CSV\_BACKSLASH\_ESCAPE
-   CSV\_DELIMITER
-   CSV\_HEADER
-   CSV\_NOT\_NULL
-   CSV\_NULL
-   CSV\_SEPARATOR
-   CSV\_TRIM\_LAST\_SEPARATORS
-   CUME\_DIST (R-Window)
-   CURRENT
-   CURRENT\_DATE (R)
-   CURRENT\_ROLE (R)
-   CURRENT\_TIME (R)
-   CURRENT\_TIMESTAMP (R)
-   CURRENT\_USER (R)
-   CURSOR (R)
-   CYCLE

<a id="D" class="letter" href="#D">D</a>

-   データ
-   データベース (R)
-   データベース (R)
-   日付
-   日時
-   日
-   日\_時間 (R)
-   日\_マイクロ秒 (R)
-   日\_分 (R)
-   日\_秒 (R)
-   DDL
-   DEALLOCATE
-   DECIMAL (R)
-   DECLARE
-   DEFAULT (R)
-   DEFINER
-   DELAY\_KEY\_WRITE
-   DELAYED (R)
-   DELETE (R)
-   DENSE\_RANK (R-Window)
-   DEPENDENCY
-   DEPTH
-   DESC (R)
-   DESCRIBE (R)
-   DIGEST
-   DIRECTORY
-   DISABLE
-   DISABLED
-   DISCARD
-   DISK
-   DISTINCT (R)
-   DISTINCTROW (R)
-   DIV (R)
-   DO
-   DOUBLE (R)
-   DRAINER
-   DROP (R)
-   DRY
-   DUAL (R)
-   DUPLICATE
-   DYNAMIC

<a id="E" class="letter" href="#E">E</a>

-   ELSE (R)
-   ELSEIF (R)
-   ENABLE
-   ENABLED
-   ENCLOSED (R)
-   ENCRYPTION
-   END
-   ENFORCED
-   ENGINE
-   ENGINES
-   ENUM
-   ERROR
-   ERRORS
-   ESCAPE
-   ESCAPED (R)
-   EVENT
-   EVENTS
-   EVOLVE
-   EXCEPT (R)
-   EXCHANGE
-   EXCLUSIVE
-   EXECUTE
-   EXISTS (R)
-   EXIT (R)
-   EXPANSION
-   EXPIRE
-   EXPLAIN (R)
-   EXTENDED

<a id="F" class="letter" href="#F">F</a>

-   FAILED\_LOGIN\_ATTEMPTS
-   FALSE (R)
-   FAULTS
-   FETCH (R)
-   FIELDS
-   FILE
-   FIRST
-   FIRST\_VALUE (R-Window)
-   FIXED
-   FLOAT (R)
-   FLUSH
-   FOLLOWING
-   FOR (R)
-   FORCE (R)
-   FOREIGN (R)
-   FORMAT
-   FOUND
-   FROM (R)
-   FULL
-   FULLTEXT (R)
-   FUNCTION

<a id="G" class="letter" href="#G">G</a>

-   GENERAL
-   GENERATED (R)
-   GLOBAL
-   GRANT (R)
-   GRANTS
-   GROUP (R)
-   GROUPS (R-Window)

<a id="H" class="letter" href="#H">H</a>

-   HANDLER
-   HASH
-   HAVING (R)
-   HELP
-   HIGH\_PRIORITY (R)
-   HISTOGRAM
-   HISTOGRAMS\_IN\_FLIGHT
-   HISTORY
-   HOSTS
-   HOUR
-   HOUR\_MICROSECOND (R)
-   HOUR\_MINUTE (R)
-   HOUR\_SECOND (R)

<a id="I" class="letter" href="#I">I</a>

-   IDENTIFIED
-   IF (R)
-   IGNORE (R)
-   ILIKE (R)
-   IMPORT
-   IMPORTS
-   IN (R)
-   INCREMENT
-   INCREMENTAL
-   INDEX (R)
-   INDEXES
-   INFILE (R)
-   INNER (R)
-   INOUT (R)
-   INSERT (R)
-   INSERT\_METHOD
-   INSTANCE
-   INT (R)
-   INT1 (R)
-   INT2 (R)
-   INT3 (R)
-   INT4 (R)
-   INT8 (R)
-   INTEGER (R)
-   INTERSECT (R)
-   INTERVAL (R)
-   INTO (R)
-   INVISIBLE
-   INVOKER
-   IO
-   IPC
-   IS (R)
-   ISOLATION
-   ISSUER
-   ITERATE (R)

<a id="J" class="letter" href="#J">J</a>

-   JOB
-   JOBS
-   JOIN (R)
-   JSON

<a id="K" class="letter" href="#K">K</a>

-   キー (R)
-   キー (R)
-   キー\_ブロック\_サイズ
-   キル (R)

<a id="L" class="letter" href="#L">L</a>

-   ラベル
-   ラグ (R-Window)
-   言語
-   最後
-   最終\_バックアップ
-   最終\_値 (R-Window)
-   ラストバル
-   先行 (R-Window)
-   先頭 (R)
-   退出 (R)
-   左 (R)
-   より小さい
-   レベル
-   と同じ (R)
-   制限 (R)
-   線形 (R)
-   行 (R)
-   リスト
-   読み込み (R)
-   ローカル
-   ローカルのみ
-   ローカル時間 (R)
-   ローカルタイムスタンプ (R)
-   場所
-   ロック (R)
-   ロックされた
-   ログ
-   長い (R)
-   長いバイナリ (R)
-   長いテキスト (R)
-   低い優先度 (R)

<a id="M" class="letter" href="#M">M</a>

-   マスター
-   一致 (R)
-   最大値 (R)
-   時間あたりの最大接続数
-   最大\_インデックス番号
-   最大\_分
-   時間あたりの最大クエリ数
-   最大行数
-   時間あたりの最大更新数
-   最大ユーザ接続数
-   MB
-   中型バイナリ (R)
-   中型整数 (R)
-   中型テキスト (R)
-   メンバー
-   メモリ
-   マージ
-   マイクロ秒
-   分
-   分\_マイクロ秒 (R)
-   分\_秒 (R)
-   最小値
-   最小行数
-   MOD (R)
-   モード
-   修正
-   月

<a id="N" class="letter" href="#N">N</a>

-   名前
-   ナショナル
-   ナチュラル (R)
-   NCHAR
-   決して
-   次の
-   次の値
-   いいえ
-   キャッシュなし
-   サイクルなし
-   ノードグループ
-   ノードID
-   ノード状態
-   最大値なし
-   最小値なし
-   ノンクラスタ化
-   なし
-   ない (R)
-   待たない
-   バイナリログに書き込まない (R)
-   NTH\_VALUE (R-Window)
-   NTILE (R-Window)
-   NULL (R)
-   NULLS
-   数値 (R)
-   NVARCHAR

<a id="O" class="letter" href="#O">O</a>

-   OF (R)
-   オフ
-   オフセット
-   OLTP\_READ\_ONLY
-   OLTP\_READ\_WRITE
-   OLTP\_WRITE\_ONLY
-   ON (R)
-   ON\_DUPLICATE
-   オンライン
-   のみ
-   開く
-   楽観的
-   最適化 (R)
-   オプション (R)
-   オプション
-   オプションで (R)
-   または (R)
-   順序 (R)
-   外部 (R)
-   OUTFILE (R)
-   OVER (R-Window)

<a id="P" class="letter" href="#P">P</a>

-   キーをパック
-   ページ
-   パーサー
-   部分的
-   パーティション (R)
-   パーティショニング
-   パーティション
-   パスワード
-   パスワードロック時間
-   一時停止
-   パーセント
-   パーセントランク (R-Window)
-   データベースごと
-   テーブルごと
-   悲観的
-   配置 (S)
-   プラグイン
-   ポイント
-   ポリシー
-   先行
-   精度 (R)
-   準備
-   保存
-   事前分割領域
-   主要 (R)
-   特権
-   手続き (R)
-   プロセス
-   プロセスリスト
-   プロファイル
-   プロファイル
-   代理
-   ポンプ
-   廃棄

<a id="Q" class="letter" href="#Q">Q</a>

-   四半期
-   クエリ
-   問い合わせ
-   速い

<a id="R" class="letter" href="#R">R</a>

-   RANGE（R）
-   RANK（R-Window）
-   RATE\_LIMIT
-   READ（R）
-   REAL（R）
-   REBUILD
-   RECOVER
-   RECURSIVE（R）
-   REDUNDANT
-   REFERENCES（R）
-   REGEXP（R）
-   REGION
-   REGIONS
-   RELEASE（R）
-   RELOAD
-   REMOVE
-   RENAME（R）
-   REORGANIZE
-   REPAIR
-   REPEAT（R）
-   REPEATABLE
-   REPLACE（R）
-   REPLICA
-   REPLICAS
-   REPLICATION
-   REQUIRE（R）
-   REQUIRED
-   RESET
-   RESOURCE
-   RESPECT
-   RESTART
-   RESTORE
-   RESTORES
-   RESTRICT（R）
-   RESUME
-   REUSE
-   REVERSE
-   REVOKE（R）
-   RIGHT（R）
-   RLIKE（R）
-   ROLE
-   ROLLBACK
-   ROUTINE
-   ROW（R）
-   ROW\_COUNT
-   ROW\_FORMAT
-   ROW\_NUMBER（R-Window）
-   ROWS（R-Window）
-   RTREE
-   RUN

<a id="S" class="letter" href="#S">S</a>

-   SAMPLERATE
-   SAMPLES
-   SAN
-   SAVEPOINT
-   SECOND
-   SECOND\_MICROSECOND（R）
-   SECONDARY
-   SECONDARY\_ENGINE
-   SECONDARY\_LOAD
-   SECONDARY\_UNLOAD
-   SECURITY
-   SELECT（R）
-   SEND\_CREDENTIALS\_TO\_TIKV
-   SEPARATOR
-   SEQUENCE
-   SERIAL
-   SERIALIZABLE
-   SESSION
-   SESSION\_STATES
-   SET（R）
-   SETVAL
-   SHARD\_ROW\_ID\_BITS
-   SHARE
-   SHARED
-   SHOW（R）
-   SHUTDOWN
-   SIGNED
-   SIMPLE
-   SKIP
-   SKIP\_SCHEMA\_FILES
-   SLAVE
-   SLOW
-   SMALLINT（R）
-   SNAPSHOT
-   SOME
-   SOURCE
-   SPATIAL（R）
-   SPLIT
-   SQL（R）
-   SQL\_BIG\_RESULT（R）
-   SQL\_BUFFER\_RESULT
-   SQL\_CACHE
-   SQL\_CALC\_FOUND\_ROWS（R）
-   SQL\_NO\_CACHE
-   SQL\_SMALL\_RESULT（R）
-   SQL\_TSI\_DAY
-   SQL\_TSI\_HOUR
-   SQL\_TSI\_MINUTE
-   SQL\_TSI\_MONTH
-   SQL\_TSI\_QUARTER
-   SQL\_TSI\_SECOND
-   SQL\_TSI\_WEEK
-   SQL\_TSI\_YEAR
-   SQLEXCEPTION（R）
-   SQLSTATE（R）
-   SQLWARNING（R）
-   SSL（R）
-   START
-   STARTING（R）
-   STATISTICS
-   STATS
-   STATS\_AUTO\_RECALC
-   STATS\_BUCKETS
-   STATS\_COL\_CHOICE
-   STATS\_COL\_LIST
-   STATS\_EXTENDED（R）
-   STATS\_HEALTHY
-   STATS\_HISTOGRAMS
-   STATS\_LOCKED
-   STATS\_META
-   STATS\_OPTIONS
-   STATS\_PERSISTENT
-   STATS\_SAMPLE\_PAGES
-   STATS\_SAMPLE\_RATE
-   STATS\_TOPN
-   STATUS
-   STORAGE
-   STORED（R）
-   STRAIGHT\_JOIN（R）
-   STRICT\_FORMAT
-   SUBJECT
-   SUBPARTITION
-   SUBPARTITIONS
-   SUPER
-   SWAPS
-   SWITCHES
-   SYSTEM
-   SYSTEM\_TIME

<a id="T" class="letter" href="#T">T</a>

-   テーブル（R）
-   テーブル
-   TABLESAMPLE（R）
-   TABLESPACE
-   TABLE\_CHECKSUM
-   テレメトリ
-   TELEMETRY\_ID
-   一時的な
-   一時テーブル
-   終了（R）
-   テキスト
-   よりも
-   その後（R）
-   TiDB
-   TiDB\_CURRENT\_TSO（R）
-   TIFLASH
-   TIKV\_IMPORTER
-   時間
-   タイムスタンプ
-   TINYBLOB（R）
-   TINYINT（R）
-   TINYTEXT（R）
-   TO（R）
-   トークン発行者
-   TOPN
-   TPCC
-   トレース
-   伝統的な
-   末尾（R）
-   トランザクション
-   トリガー（R）
-   トリガー
-   真（R）
-   切り捨てる
-   TSO
-   TTL
-   TTL\_ENABLE
-   TTL\_JOB\_INTERVAL
-   タイプ

<a id="U" class="letter" href="#U">U</a>

-   UNBOUNDED
-   UNCOMMITTED
-   未定義
-   UNICODE
-   UNION（R）
-   ユニーク（R）
-   UNKNOWN
-   アンロック（R）
-   UNSIGNED（R）
-   UNTIL（R）
-   UPDATE（R）
-   USAGE（R）
-   USE（R）
-   ユーザー
-   USING（R）
-   UTC\_DATE（R）
-   UTC\_TIME（R）
-   UTC\_TIMESTAMP（R）

<a id="V" class="letter" href="#V">V</a>

-   検証
-   値
-   VALUES（R）
-   VARBINARY（R）
-   VARCHAR（R）
-   VARCHARACTER（R）
-   変数
-   VARYING（R）
-   ビュー
-   VIRTUAL（R）
-   VISIBLE

<a id="W" class="letter" href="#W">W</a>

-   待つ
-   警告
-   週
-   WEIGHT\_STRING
-   WHEN（R）
-   WHERE（R）
-   WHILE（R）
-   幅
-   ウィンドウ（R-Window）
-   WITH（R）
-   WITHOUT
-   WORKLOAD
-   WRITE（R）

<a id="X" class="letter" href="#X">X</a>

-   X509
-   XOR（R）

<a id="Y" class="letter" href="#Y">Y</a>

-   年
-   YEAR\_MONTH（R）

<a id="Z" class="letter" href="#Z">Z</a>

-   ZEROFILL（R）
