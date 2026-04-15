# JVN CVE Fetcher

Microsoft Security Copilot 用カスタムプラグインです。\
[JPCERT/CC](https://www.jpcert.or.jp/) および [JVN iPedia（JVNDB）](https://jvndb.jvn.jp/) から CVE 脆弱性情報を取得します。

[MyJVN API](https://jvndb.jvn.jp/apis/index.html)（IPA/JPCERT/CC が無償提供）を通じて、Security Copilot のプロンプトから直接 JVN 脆弱性情報や IPA セキュリティ警戒情報を検索できます。

---

## 機能

| スキル | 説明 |
|---|---|
| **GetJVNDBVulnerabilities** | JVN iPedia（JVNDB）に登録された CVE 脆弱性情報を取得。キーワード・深刻度・期間でフィルタリング可能。 |
| **GetIPASecurityAlerts** | IPA および JPCERT/CC が発行するセキュリティ警戒情報（製品アップデート通知・緊急勧告）を取得。 |

### GetJVNDBVulnerabilities パラメータ

| パラメータ | 説明 | 値 | デフォルト |
|---|---|---|---|
| `rangeDatePublic` | 発見日（公表日）の期間フィルタ | `w`=過去1週間 / `m`=過去1ヶ月 / `n`=無制限 | `m` |
| `rangeDatePublished` | 更新日の期間フィルタ | `w` / `m` / `n` | `n` |
| `rangeDateFirstPublished` | JVN iPedia 登録日の期間フィルタ | `w` / `m` / `n` | `n` |
| `keyword` | キーワード検索（製品名、ベンダ名、CVE番号など） | 任意の文字列 | — |
| `severity` | CVSS 深刻度フィルタ（CVSSv3 基準） | `c`=緊急 / `h`=重要 / `m`=警告 / `l`=注意 / `n`=なし | `n` |
| `startItem` | ページネーション開始位置 | 整数（1〜） | `1` |
| `maxCountItem` | 取得件数（最大50件） | 1〜50 | `50` |
| `lang` | レスポンス言語 | `ja`=日本語 / `en`=英語 | `ja` |

> **日付の絶対指定について：** MyJVN API の期間パラメータは「過去1週間/1ヶ月/無制限」の相対指定のみサポートされています。  
> 「2025年1月1日以降」のような絶対日付で絞り込む場合は `rangeDatePublic=n` で取得し、レスポンスの `dc:date` フィールドを Security Copilot に参照させて絞り込んでください。

### GetIPASecurityAlerts パラメータ

| パラメータ | 説明 | デフォルト |
|---|---|---|
| `startItem` | ページネーション開始位置 | `1` |
| `maxCountItem` | 取得件数（最大50件） | `50` |
| `lang` | レスポンス言語（`ja` / `en`） | `ja` |

---

## プロンプト例

```
直近1ヶ月の CVE 脆弱性情報を取得してください
```
```
過去1週間に公開された Windows に関する脆弱性を取得してください
```
```
深刻度が重要（High）な脆弱性を過去1ヶ月分取得してください
```
```
Apache の重大な CVE 脆弱性を検索してください
```
```
最新の IPA セキュリティ警戒情報を取得してください
```
```
JPCERT/CC が発行した最近のセキュリティ勧告を確認してください
```

---

## ファイル一覧

```
JVNCVEFetcher/
├── JVNCVEFetcher.yaml                  # プラグインマニフェスト
├── JVNCVEFetcher_vuln_openapi.yaml     # GetJVNDBVulnerabilities 用 OpenAPI 仕様
├── JVNCVEFetcher_alert_openapi.yaml    # GetIPASecurityAlerts 用 OpenAPI 仕様
├── JVNCVEFetcher_card.html             # Plugin Card（ブラウザで視覚確認用）
└── README.md                           # このファイル
```

---

## デプロイ手順

### 1. OpenAPI ファイルをホスティング

`JVNCVEFetcher_vuln_openapi.yaml` と `JVNCVEFetcher_alert_openapi.yaml` を、Security Copilot からアクセスできる公開 URL にホスティングします。

**GitHub を使う場合（推奨）：**

このリポジトリを公開リポジトリとして GitHub に Push すると、以下の形式の Raw URL が使えます。

```
https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_vuln_openapi.yaml
https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_alert_openapi.yaml
```

### 2. マニフェストの URL を更新

`JVNCVEFetcher.yaml` の `<YOUR-HOST>` を実際の URL に書き換えます。

```yaml
SkillGroups:
  - Format: API
    Settings:
      OpenApiSpecUrl: https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_vuln_openapi.yaml

  - Format: API
    Settings:
      OpenApiSpecUrl: https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_alert_openapi.yaml
```

### 3. Security Copilot にアップロード

1. [Microsoft Security Copilot](https://securitycopilot.microsoft.com/) を開く
2. 右上の「Sources」→「Custom plugins」→「Add plugin」を選択
3. Upload から `JVNCVEFetcher.yaml`（更新済み）をアップロード
4. プラグインが有効化されたことを確認

---

## データソース

| ソース | URL | 提供元 |
|---|---|---|
| JVN iPedia（JVNDB） | https://jvndb.jvn.jp/ | IPA（独立行政法人 情報処理推進機構） |
| JPCERT/CC | https://www.jpcert.or.jp/ | JPCERT/CC |
| MyJVN API | https://jvndb.jvn.jp/apis/index.html | IPA / JPCERT/CC（無償） |

本プラグインが使用する MyJVN API は IPA と JPCERT/CC が共同運営する無償の公開 API です。\
ご利用の際は [MyJVN 利用上の規約](https://jvndb.jvn.jp/apis/termsofuse.html) をご確認ください。

---

## 制限事項

- MyJVN API の 1 回のリクエストあたりの取得上限は **50件** です。51件以降は `startItem` でページネーションしてください。
- 期間指定は相対期間（`w`=過去1週間, `m`=過去1ヶ月, `n`=無制限）のみサポートされています。
- Security Copilot は XML レスポンスを受け取り解析します。大量データ（数百件以上）の取得にはページネーションが必要です。
- 本プラグインは読み取り専用です（書き込みアクションはありません）。

---

## 関連リンク

- [MyJVN API ドキュメント](https://jvndb.jvn.jp/apis/index.html)
- [MyJVN API よくある質問](https://jvndb.jvn.jp/apis/myjvnapi_faq.html)
- [JVN iPedia](https://jvndb.jvn.jp/)
- [JPCERT/CC](https://www.jpcert.or.jp/)
- [Security Copilot カスタムプラグイン ドキュメント](https://learn.microsoft.com/ja-jp/copilot/security/custom-plugins)
