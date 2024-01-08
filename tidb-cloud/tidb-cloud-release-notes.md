---
title: TiDB Cloud Release Notes in 2024
summary: Learn about the release notes of TiDB Cloud in 2024.
aliases: ['/tidbcloud/supported-tidb-versions','/tidbcloud/release-notes']
---

# TiDB Cloud Release Notes in 2024 {#tidb-cloud-release-notes-in-2024}

このページは、2024年の[TiDB Cloud](https://www.pingcap.com/tidb-cloud/)のリリースノートをリストアップしています。

## 2024年1月3日 {#january-3-2024}

**一般的な変更**

- [Organization SSO](https://tidbcloud.com/console/preferences/authentication)のサポートを追加し、企業の認証プロセスを効率化します。

  この機能により、[Security Assertion Markup Language (SAML)](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language)や[OpenID Connect (OIDC)](https://openid.net/developers/how-connect-works/)を使用して、TiDB Cloudを任意のアイデンティティプロバイダ（IdP）にシームレスに統合できます。

  詳細については、[Organization SSO Authentication](/tidb-cloud/tidb-cloud-org-sso-authentication.md)を参照してください。

- 新しい[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタのデフォルトのTiDBバージョンを[v7.1.1](https://docs.pingcap.com/tidb/v7.1/release-7.1.1)から[v7.5.0](https://docs.pingcap.com/tidb/v7.5/release-7.5.0)にアップグレードしました。

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)のデュアルリージョンバックアップ機能が一般利用可能（GA）になりました。

  この機能を使用すると、AWSやGoogle Cloud内の地理的なリージョン間でバックアップをレプリケートできます。この機能により、データ保護とディザスタリカバリの追加のレイヤーが提供されます。

  詳細については、[Dual region backup](/tidb-cloud/backup-and-restore.md#turn-on-dual-region-backup)を参照してください。
