---
title: Tune Region Performance
summary: Learn how to tune Region performance by adjusting the Region size and how to use buckets to optimize concurrent queries when the Region size is large.
---

# リージョンのパフォーマンスを調整する {#tune-region-performance}

このドキュメントでは、リージョンのサイズを調整してリージョンのパフォーマンスを調整する方法と、リージョンのサイズが大きい場合に同時クエリを最適化するためにバケットを使用する方法について説明します。

## 概要 {#overview}

TiKVは、自動的に[下層のデータをシャーディング](/best-practices/tidb-best-practices.md#data-sharding)します。データはキーの範囲に基づいて複数のリージョンに分割されます。リージョンのサイズが閾値を超えると、TiKVは2つ以上のリージョンに分割します。

大規模なデータセットを扱うシナリオでは、リージョンのサイズが比較的小さいと、TiKVにはリージョンが多すぎるため、より多くのリソースを消費し、[パフォーマンスの低下](/best-practices/massive-regions-best-practices.md#performance-problem)が発生する可能性があります。v6.1.0以降、TiDBはリージョンのサイズをカスタマイズすることができます。リージョンのデフォルトサイズは96 MiBです。リージョンの数を減らすために、リージョンをより大きなサイズに調整することができます。

多数のリージョンのパフォーマンスオーバーヘッドを減らすために、[ハイバネートリージョン](/best-practices/massive-regions-best-practices.md#method-4-increase-the-number-of-tikv-instances)または[`リージョンマージ`](/best-practices/massive-regions-best-practices.md#method-5-adjust-raft-base-tick-interval)を有効にすることもできます。

## `region-split-size`を使用してリージョンのサイズを調整する {#use-region-split-size-to-adjust-region-size}

> **Note:**
>
> リージョンサイズの推奨範囲は\[48MiB、258MiB]です。一般的に使用されるサイズには、96 MiB、128 MiB、256 MiBがあります。リージョンサイズを1 GiBを超えるように設定することはお勧めしません。10 GiBを超えるサイズに設定しないでください。極端に大きなリージョンサイズは、次の副作用を引き起こす可能性があります。
>
> - パフォーマンスのジッター
> - クエリパフォーマンスの低下、特に大量のデータを扱うクエリの場合
> - リージョンのスケジューリングが遅くなる

リージョンのサイズを調整するには、[`coprocessor.region-split-size`](/tikv-configuration-file.md#region-split-size)構成項目を使用できます。TiFlashを使用する場合、リージョンのサイズは256 MiBを超えてはいけません。

Dumplingツールを使用する場合、リージョンのサイズは1 GiBを超えてはいけません。この場合、リージョンのサイズを増やした後、並行性を減らす必要があります。そうしないと、TiDBがメモリ不足になる可能性があります。

## バケットを使用して並行性を増やす {#use-bucket-to-increase-concurrency}

> **Warning:**
>
> 現在、これはTiDB v6.1.0で導入された実験的な機能です。本番環境では使用しないでください。

リージョンをより大きなサイズに設定した後、クエリの並行性をさらに向上させたい場合は、[`coprocessor.enable-region-bucket`](/tikv-configuration-file.md#enable-region-bucket-new-in-v610)を`true`に設定して、クエリの並行性を増やすことができます。この構成を使用すると、リージョンはバケットに分割されます。バケットはリージョン内の小さな範囲であり、スキャンの並行性を向上させるための並行クエリの単位として使用されます。バケットのサイズは[`coprocessor.region-bucket-size`](/tikv-configuration-file.md#region-bucket-size-new-in-v610)を使用して制御できます。デフォルト値は`96MiB`です。
