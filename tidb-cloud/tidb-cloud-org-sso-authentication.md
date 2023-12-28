---
title: Organization SSO Authentication
summary: Learn how to log in to the TiDB Cloud console via your customized organization authentication.
---

# 組織SSO認証 {#organization-sso-authentication}

シングルサインオン（SSO）は、メンバーが電子メールアドレスとパスワードの代わりに、アイデンティティプロバイダ（IdP）からのアイデンティティを使用してTiDB Cloudにログインできるようにする認証スキームです。

TiDB Cloudでは、次の2つのタイプのSSO認証をサポートしています。

- [基本的なSSO](/tidb-cloud/tidb-cloud-sso-authentication.md)：メンバーは、GitHub、Google、またはMicrosoftの認証方法を使用して[TiDB Cloudコンソール](https://tidbcloud.com/)にログインできます。基本的なSSOは、TiDB Cloudのすべての組織でデフォルトで有効になっています。

- クラウド組織SSO：メンバーは、組織で指定された認証方法を使用して、TiDB Cloudのカスタムログインページにログインできます。クラウド組織SSOはデフォルトで無効になっています。

基本的なSSOと比較すると、クラウド組織SSOはより柔軟性とカスタマイズ性を提供し、組織のセキュリティとコンプライアンス要件をよりよく満たすことができます。例えば、ログインページに表示される認証方法を指定したり、ログインに許可される電子メールアドレスのドメインを制限したり、[OpenID Connect（OIDC）](https://openid.net/connect/)や[Security Assertion Markup Language（SAML）](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language)のアイデンティティプロトコルを使用するアイデンティティプロバイダ（IdP）でTiDB Cloudにログインできるようにしたりすることができます。

このドキュメントでは、組織の認証スキームを基本的なSSOからクラウド組織SSOに移行する方法について説明します。

> **Note:**
>
> クラウド組織SSO機能は、有料の組織のみで利用可能です。

## 開始する前に {#before-you-begin}

クラウド組織SSOに移行する前に、このセクションの項目を確認して組織について確認してください。

> **Note:**
>
> - クラウド組織SSOが有効になると、無効にすることはできません。
> - クラウド組織SSOを有効にするには、TiDB Cloud組織の`Organization Owner`ロールである必要があります。ロールについての詳細は、[ユーザーロール](/tidb-cloud/manage-user-access.md#user-roles)を参照してください。

### 組織のカスタムURLを決定する {#decide-a-custom-url-for-the-tidb-cloud-login-page-of-your-organization}

クラウド組織SSOが有効になると、メンバーはTiDB CloudにログインするためにパブリックログインURL（`https://tidbcloud.com`）の代わりにカスタムURLを使用する必要があります。

カスタムURLは有効になった後は変更できないため、事前に使用するURLを決定する必要があります。

カスタムURLの形式は`https://tidbcloud.com/enterprise/signin/your-company-name`であり、会社名をカスタマイズすることができます。

### 組織メンバーの認証方法を決定する {#decide-authentication-methods-for-your-organization-members}

TiDB Cloudでは、組織SSOに次の認証方法を提供しています。

- ユーザー名とパスワード
- Google
- GitHub
- Microsoft
- OIDC
- SAML

クラウド組織SSOを有効にすると、最初の4つの方法がデフォルトで有効になります。組織でSSOの使用を強制する場合は、ユーザー名とパスワードの認証方法を無効にすることができます。

有効になっているすべての認証方法は、カスタムTiDB Cloudログインページに表示されるため、事前に有効または無効にする認証方法を決定する必要があります。

### 自動プロビジョニングを有効にするかどうかを決定する {#decide-whether-to-enable-auto-provision}

自動プロビジョニングは、`Organization Owner`または`Project Owner`からの招待を必要とせずにメンバーが組織に自動的に参加できるようにする機能です。TiDB Cloudでは、サポートされているすべての認証方法でデフォルトで無効になっています。

- 自動プロビジョニングが認証方法で無効になっている場合、カスタムURLにログインするために招待された`Organization Owner`または`Project Owner`のみがログインできます。
- 自動プロビジョニングが認証方法で有効になっている場合、この認証方法を使用するすべてのユーザーがカスタムURLにログインできます。ログイン後、彼らは自動的に組織内のデフォルトの**メンバー**ロールが割り当てられます。

セキュリティ上の考慮事項として、自動プロビジョニングを有効にする場合は、[認証方法の詳細を設定する](#step-2-configure-authentication-methods)際に認証に許可される電子メールドメインを制限することをお勧めします。

### クラウド組織SSOの移行計画についてメンバーに通知する {#notify-your-members-about-the-cloud-organization-sso-migration-plan}

クラウド組織SSOを有効にする前に、メンバーに次のことを通知してください。

- TiDB CloudのカスタムログインURL
- ログインに`https://tidbcloud.com`の代わりにカスタムログインURLを使用する時期
- 利用可能な認証方法
- メンバーがカスタムログインURLにログインするために招待が必要かどうか

## ステップ1. クラウド組織SSOを有効にする {#step-1-enable-cloud-organization-sso}

クラウド組織SSOを有効にするには、次の手順を実行します：

1. `Organization Owner` 権限を持つユーザーとして[TiDB Cloudコンソール](https://tidbcloud.com)にログインします。

2. TiDB Cloudコンソールの左下隅にある <MDSvgIcon name="icon-top-organization" /> をクリックし、**Organization Settings**をクリックします。

3. **Organization Settings**ページで、**Authentication**タブをクリックし、**Enable**をクリックします。

4. ダイアログに、TiDB Cloud内で一意である必要がある組織のカスタムURLを入力します。

   > **Note:**
   >
   > Cloud Organization SSOが有効になると、URLを変更することはできません。組織のメンバーは、カスタムURLを使用してTiDB Cloudにログインすることしかできません。後で設定したURLを変更する必要がある場合は、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してサポートを受ける必要があります。

5. **I understand and confirm**チェックボックスをクリックし、**Enable**をクリックします。

   > **Note:**
   >
   > ダイアログにCloud Organization SSOの再招待および再参加するユーザーのリストが含まれている場合、TiDB CloudはCloud Organization SSOを有効にした後、自動的にこれらのユーザーに招待メールを送信します。招待メールを受け取った後、各ユーザーはメール内のリンクをクリックして自分のアイデンティティを確認し、カスタムログインページが表示されます。

## ステップ2. 認証方法を設定する {#step-2-configure-authentication-methods}

TiDB Cloudで認証方法を有効にすると、その方法を使用してログインするメンバーは、カスタムURLを使用してTiDB Cloudにログインできます。

### ユーザー名とパスワード、Google、GitHub、またはMicrosoftの認証方法を設定する {#configure-username-and-password-google-github-or-microsoft-authentication-methods}

Cloud Organization Cloudを有効にした後、次のようにしてユーザー名とパスワード、Google、GitHub、またはMicrosoftの認証方法を設定できます。

1. **Organization Settings**ページで、必要に応じてGoogle、GitHub、またはMicrosoftの認証方法を有効または無効にします。

2. 有効になっている認証方法では、<svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 20H21M3.00003 20H4.67457C5.16376 20 5.40835 20 5.63852 19.9447C5.84259 19.8957 6.03768 19.8149 6.21663 19.7053C6.41846 19.5816 6.59141 19.4086 6.93732 19.0627L19.5001 6.49998C20.3285 5.67156 20.3285 4.32841 19.5001 3.49998C18.6716 2.67156 17.3285 2.67156 16.5001 3.49998L3.93729 16.0627C3.59139 16.4086 3.41843 16.5816 3.29475 16.7834C3.18509 16.9624 3.10428 17.1574 3.05529 17.3615C3.00003 17.5917 3.00003 17.8363 3.00003 18.3255V20Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg> をクリックして、方法の詳細を設定できます。

3. 方法の詳細では、次のように設定できます。

   - [**Auto-provision Accounts**](#decide-whether-to-enable-auto-provision)

     デフォルトでは無効になっています。必要に応じて有効にできます。セキュリティ上の考慮事項として、自動プロビジョニングを有効にする場合は、認証に許可されるメールドメインを制限することをお勧めします。

   - **Allowed Email Domains**

     このフィールドを設定すると、この認証方法の指定されたメールドメインのみが、カスタムURLを使用してTiDB Cloudにログインできるようになります。ドメイン名を入力する際は、`@`記号を除外し、コンマで区切る必要があります。例えば、`company1.com,company2.com`です。

     > **Note:**
     >
     > メールドメインを設定した場合、設定を保存する前に、現在ログインに使用しているメールドメインを追加しておくようにしてください。そうしないと、TiDB Cloudによってロックアウトされる可能性があります。

4. **Save**をクリックします。

### OIDC認証方法を設定する {#configure-the-oidc-authentication-method}

OIDCアイデンティティプロトコルを使用するアイデンティティプロバイダーがある場合、TiDB CloudログインにOIDC認証方法を有効にできます。

TiDB Cloudでは、OIDC認証方法はデフォルトで無効になっています。Cloud Organization Cloudを有効にした後、次のようにしてOIDC認証方法を有効にし、設定できます：

1. TiDB Cloud組織SSOのために、次の情報をあなたのアイデンティティプロバイダから取得してください：

   - 発行者URL
   - クライアントID
   - クライアントシークレット

2. **組織設定**ページで、**認証**タブをクリックし、**認証方法**エリアのOIDCの行を見つけ、<svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 20H21M3.00003 20H4.67457C5.16376 20 5.40835 20 5.63852 19.9447C5.84259 19.8957 6.03768 19.8149 6.21663 19.7053C6.41846 19.5816 6.59141 19.4086 6.93732 19.0627L19.5001 6.49998C20.3285 5.67156 20.3285 4.32841 19.5001 3.49998C18.6716 2.67156 17.3285 2.67156 16.5001 3.49998L3.93729 16.0627C3.59139 16.4086 3.41843 16.5816 3.29475 16.7834C3.18509 16.9624 3.10428 17.1574 3.05529 17.3615C3.00003 17.5917 3.00003 17.8363 3.00003 18.3255V20Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg>をクリックして、OIDCの詳細を表示します。

3. 認証方法の詳細では、次のように設定できます：

   - **名前**

     カスタムログインページに表示されるOIDC認証方法の名前を指定します。

   - **発行者URL**、**クライアントID**、および**クライアントシークレット**

     あなたのIdPから取得した対応する値を貼り付けてください。

   - [**アカウントの自動プロビジョニング**](#自動プロビジョニングを有効にするかどうかを決定する)

     デフォルトでは無効になっています。必要に応じて有効にすることができます。セキュリティ上の考慮事項として、自動プロビジョニングを有効にする場合は、認証に許可されるメールドメインを制限することをお勧めします。

   - **許可されたメールドメイン**

     このフィールドが設定されると、この認証方法の指定されたメールドメインのみがカスタムURLを使用してTiDB Cloudにログインできます。ドメイン名を入力する際は、`@`記号を除外し、コンマで区切ってください。例えば、`company1.com,company2.com`。

     > **Note:**
     >
     > メールドメインを設定した場合、設定を保存する前に、現在ログインに使用しているメールドメインを追加しておくことを忘れないでください。そうしないと、TiDB Cloudによってロックアウトされる可能性があります。

4. **保存**をクリックします。

### SAML認証方法を設定する {#configure-the-saml-authentication-method}

SAMLアイデンティティプロトコルを使用するアイデンティティプロバイダがある場合、TiDB CloudログインのためにSAML認証方法を有効にすることができます。

> **Note:**
>
> TiDB Cloudは、異なるユーザーの一意の識別子としてメールアドレスを使用します。そのため、組織メンバーの`email`属性があなたのアイデンティティプロバイダで設定されていることを確認してください。

TiDB Cloudでは、SAML認証方法はデフォルトで無効になっています。Cloud Organization Cloudを有効にした後、次のようにSAML認証方法を有効にして設定することができます：

1. TiDB Cloud組織SSOのために、次の情報をあなたのアイデンティティプロバイダーから取得してください：

   - サインオンURL
   - 署名証明書

2. **組織設定**ページで、**認証**タブをクリックし、**認証方法**エリアのSAMLの行を見つけ、<svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 20H21M3.00003 20H4.67457C5.16376 20 5.40835 20 5.63852 19.9447C5.84259 19.8957 6.03768 19.8149 6.21663 19.7053C6.41846 19.5816 6.59141 19.4086 6.93732 19.0627L19.5001 6.49998C20.3285 5.67156 20.3285 4.32841 19.5001 3.49998C18.6716 2.67156 17.3285 2.67156 16.5001 3.49998L3.93729 16.0627C3.59139 16.4086 3.41843 16.5816 3.29475 16.7834C3.18509 16.9624 3.10428 17.1574 3.05529 17.3615C3.00003 17.5917 3.00003 17.8363 3.00003 18.3255V20Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg>をクリックして、SAMLメソッドの詳細を表示します。

3. メソッドの詳細では、次のように設定できます：

   - **名前**

     カスタムログインページに表示されるSAML認証方法の名前を指定します。

   - **サインオンURL**

     アイデンティティプロバイダーから取得したURLを貼り付けます。

   - **署名証明書**

     アイデンティティプロバイダーから取得した署名証明書全体を、`---begin certificate---`から始まる行と`---end certificate---`で終わる行を含めて貼り付けます。

   - [**アカウントの自動プロビジョニング**](#自動プロビジョニングを有効にするかどうかを決定する)

     デフォルトでは無効になっています。必要に応じて有効にすることができます。セキュリティ上の考慮から、自動プロビジョニングを有効にする場合は、認証に許可されるメールドメインを制限することをお勧めします。

   - **許可されたメールドメイン**

     このフィールドが設定されると、カスタムURLを使用してTiDB Cloudにログインできるのは、この認証方法の指定されたメールドメインのみになります。ドメイン名を入力する際は、`@`記号を除外し、コンマで区切ってください。例：`company1.com,company2.com`。

     > **Note:**
     >
     > メールドメインを設定した場合、設定を保存する前に、現在ログインに使用しているメールドメインを追加することを忘れないでください。そうしないと、TiDB Cloudによってロックアウトされる可能性があります。

   - **SCIMプロビジョニングアカウント**

     デフォルトでは無効になっています。アイデンティティプロバイダーからTiDB Cloud組織のユーザーとグループのプロビジョニング、デプロビジョニング、およびアイデンティティ管理を自動化する場合は、これを有効にすることができます。詳細な設定手順については、[SCIMプロビジョニングの設定](#scimプロビジョニングの設定)を参照してください。

4. **保存**をクリックします。

#### SCIMプロビジョニングの設定 {#configure-scim-provisioning}

[System for Cross-domain Identity Management (SCIM)](https://www.rfc-editor.org/rfc/rfc7644)は、アイデンティティドメインとITシステム間でユーザーのアイデンティティ情報を自動的に交換するオープンスタンダードです。SCIMプロビジョニングを設定することで、アイデンティティプロバイダーからのユーザーグループが自動的にTiDB Cloudに同期され、TiDB Cloudでこれらのグループの役割を一元管理することができます。

> **Note:**
>
> SCIMプロビジョニングは、[SAML認証方法](#saml認証方法の設定)でのみ有効にすることができます。

1. TiDB Cloudでは、[SAML認証方法](#configure-the-saml-authentication-method)の**SCIM Provisioning Accounts**オプションを有効にし、後で使用するために以下の情報を記録してください。

   - SCIMコネクタのベースURL
   - ユーザーの一意の識別子フィールド
   - 認証モード

2. アイデンティティプロバイダーで、TiDB CloudのSCIMプロビジョニングを設定してください。

   1. アイデンティティプロバイダーで、SAMLアプリ統合にTiDB Cloudの組織のSCIMプロビジョニングを追加してください。

      例えば、アイデンティティプロバイダーがOktaの場合、[アプリ統合にSCIMプロビジョニングを追加する](https://help.okta.com/en-us/content/topics/apps/apps_app_integration_wizard_scim.htm)を参照してください。

   2. アイデンティティプロバイダーの所定のグループにSAMLアプリ統合を割り当て、グループのメンバーがアプリ統合にアクセスして使用できるようにしてください。

      例えば、アイデンティティプロバイダーがOktaの場合、[グループにアプリ統合を割り当てる](https://help.okta.com/en-us/content/topics/provisioning/lcm/lcm-assign-app-groups.htm)を参照してください。

   3. アイデンティティプロバイダーからTiDB Cloudにユーザーグループをプッシュしてください。

      例えば、アイデンティティプロバイダーがOktaの場合、[グループプッシュの管理](https://help.okta.com/en-us/content/topics/users-groups-profiles/usgp-group-push-main.htm)を参照してください。

3. TiDB Cloudで、アイデンティティプロバイダーからプッシュされたグループを表示してください。

   1. [TiDB Cloudコンソール](https://tidbcloud.com)の左下隅で、<MDSvgIcon name="icon-top-organization" />をクリックし、**Organization Settings**をクリックしてください。
   2. **Organization Settings**ページで、**Groups**タブをクリックしてください。アイデンティティプロバイダーから同期されたグループが表示されます。
   3. グループ内のユーザーを表示するには、**View**をクリックしてください。

4. TiDB Cloudで、アイデンティティプロバイダーからプッシュされたグループにロールを付与してください。

   > **Note:**
   >
   > グループにロールを付与すると、グループのすべてのメンバーがそのロールを取得します。グループにすでにTiDB Cloud組織のメンバーが含まれている場合、これらのメンバーもグループの新しいロールを取得します。

   1. グループに組織ロールを付与するには、**By organization**をクリックし、**Organization Role**列でロールを設定してください。組織ロールの権限については、[組織ロール](/tidb-cloud/manage-user-access.md#organization-roles)を参照してください。
   2. グループにプロジェクトロールを付与するには、**By project**をクリックし、**Project Role**列でロールを設定してください。プロジェクトロールの権限については、[プロジェクトロール](/tidb-cloud/manage-user-access.md#project-roles)を参照してください。

5. アイデンティティプロバイダーでプッシュされたグループのメンバーを変更すると、これらの変更はTiDB Cloudの対応するグループに動的に同期されます。

   - アイデンティティプロバイダーのグループに新しいメンバーが追加されると、これらのメンバーは対応するグループのロールを取得します。
   - アイデンティティプロバイダーのグループから一部のメンバーが削除されると、これらのメンバーはTiDB Cloudの対応するグループからも削除されます。
