---
title: Control Execution Plan
---

# 実行計画の制御 {#control-execution-plan}

SQLチューニングの最初の2つの章では、TiDBの実行計画の理解方法とTiDBが実行計画を生成する方法について紹介します。この章では、実行計画の問題を特定する際に実行計画の生成を制御するために使用できる方法について紹介します。この章には主に次の3つの側面が含まれています。

- [Optimizer Hints](/optimizer-hints.md)では、ヒントを使用してTiDBに実行計画の生成をガイドする方法を学びます。
- ただし、ヒントはSQLステートメントを侵害的に変更します。一部のシナリオでは、ヒントを単純に挿入することはできません。[SQL Plan Management](/sql-plan-management.md)では、TiDBが実行計画の生成を非侵害的に制御するための別の構文を使用する方法、およびバックグラウンドでの自動実行計画進化の方法について説明します。この方法は、バージョンのアップグレードによって引き起こされる実行計画の不安定さやクラスタのパフォーマンスの低下などの問題に対処するのに役立ちます。
- 最後に、[Blocklist of Optimization Rules and Expression Pushdown](/blocklist-control-plan.md)でブロックリストを使用する方法を学びます。

<CustomContent platform="tidb">

前述の方法に加えて、実行計画はいくつかのシステム変数の影響を受けます。これらの変数をシステムレベルまたはセッションレベルで変更することで、実行計画の生成を制御することができます。v7.1.0から、TiDBは比較的特別な変数[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v657-and-v710)を導入しました。この変数は、オプティマイザの動作をより細かく制御して、クラスタのアップグレード後の動作変更によるパフォーマンスの低下を防ぐために複数の制御項目を受け入れることができます。詳細については、[Optimizer Fix Controls](/optimizer-fix-controls.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

前述の方法に加えて、実行計画はいくつかのシステム変数の影響を受けます。これらの変数をシステムレベルまたはセッションレベルで変更することで、実行計画の生成を制御することができます。v7.1.0から、TiDBは比較的特別な変数[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v657-and-v710)を導入しました。この変数は、オプティマイザの動作をより細かく制御して、クラスタのアップグレード後の動作変更によるパフォーマンスの低下を防ぐために複数の制御項目を受け入れることができます。詳細については、[Optimizer Fix Controls](https://docs.pingcap.com/tidb/v7.1/optimizer-fix-controls)を参照してください。

</CustomContent>
