---
title: tiup dm upgrade
---

# tiup dm upgrade {#tiup-dm-upgrade}

`tiup dm upgrade`コマンドは、指定されたクラスターを特定のバージョンにアップグレードするために使用されます。

## 構文 {#syntax}

```shell
tiup dm upgrade <cluster-name> <version> [flags]
```

- `<cluster-name>`は操作するクラスターの名前です。クラスター名を忘れた場合は、[`tiup dm list`](/tiup/tiup-component-dm-list.md)コマンドを使用して確認できます。
- `<version>`はアップグレードするターゲットバージョンです。例えば、`v7.1.3`です。現在、後のバージョンにのみアップグレードすることができ、前のバージョンにはアップグレードできません。つまり、ダウングレードは許可されません。また、ナイトリーバージョンへのアップグレードも許可されません。

## オプション {#options}

### --offline {#offline}

- 現在のクラスターがオフラインであることを宣言します。このオプションが指定された場合、TiUP DMはサービスを再起動せずにクラスターコンポーネントのバイナリファイルを現在の場所に置き換えます。

### -h, --help {#h-help}

- ヘルプ情報を出力します。
- データタイプ：`BOOLEAN`
- このオプションは、`false`値でデフォルトで無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

## 出力 {#output}

サービスアップグレードプロセスのログ。

[<< 前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)
