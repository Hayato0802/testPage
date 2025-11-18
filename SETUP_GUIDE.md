# ServiceNow Virtual Agent 外部ページ埋め込み手順書

このドキュメントは、ServiceNow Virtual Agentのポータブルウェブクライアントを外部ウェブサイトに埋め込むための完全な手順をまとめたものです。

---

## 目次

1. [概要](#概要)
2. [前提条件](#前提条件)
3. [ServiceNow側の設定手順](#servicenow側の設定手順)
4. [外部ページへの埋め込み手順](#外部ページへの埋め込み手順)
5. [トラブルシューティング](#トラブルシューティング)
6. [カスタマイズオプション](#カスタマイズオプション)

---

## 概要

ServiceNow Virtual Agentのポータブルウェブクライアントを使用すると、ServiceNowインスタンス外の任意のウェブサイトにチャットウィジェットを統合できます。

### 主な機能
- **簡単な統合**: JavaScriptのインポート1つで実装可能
- **カスタマイズ可能**: 色、位置、サイズなどを自由に調整
- **レスポンシブ対応**: モバイルデバイスでも動作

---

## 前提条件

### 必要な権限
- **ServiceNow側**: `virtual_agent_admin` または `admin` ロール
- **外部サイト側**: HTMLファイルの編集権限

### 必要な情報
- ServiceNowインスタンスURL（例: `https://your-instance.service-now.com`）
- 外部ウェブサイトのドメイン（例: `https://example.com`）

---

## ServiceNow側の設定手順

### 1. Portable Virtual Agent Web Clientの有効化

1. ServiceNowインスタンスにログイン
2. ナビゲーションフィルタで **Virtual Agent** を検索
3. **Virtual Agent > Designer** に移動
4. 使用したいVirtual Agentトピックを選択
5. **「Portable」** タブをクリック
6. **「Website」** オプションを選択
7. **「Enable」** をクリックして有効化

### 2. CORS設定の構成

外部ドメインからServiceNow APIへのアクセスを許可するため、CORSルールを設定します。

#### 手順:

1. **All > System Web Services > REST > CORS Rules** に移動
2. **「New」** をクリック
3. 以下の情報を入力:

| フィールド | 値 |
|----------|-----|
| **Name** | `External Website VA Access` (任意の名前) |
| **REST API** | 空欄（すべてのREST APIを許可） |
| **Domain** | `https://example.com` (外部サイトのドメイン) |
| **Max age** | `3600` (推奨) |

4. **HTTP methods** タブで以下を有効化:
   - ☑ GET
   - ☑ POST
   - ☑ PUT
   - ☑ DELETE
   - ☑ PATCH

5. **HTTP headers** タブ:
   - **Allowed request headers**: `*` (すべて許可)
   - **Allowed response headers**: `*` (すべて許可)

6. **「Submit」** をクリック

#### 複数ドメインの場合:
複数の外部サイトで使用する場合は、それぞれのドメインに対してCORSルールを作成します。

---

### 3. システムプロパティの設定

Content Security Policy (CSP) とフレーム埋め込み設定を構成します。

#### 手順:

1. **All > System Properties** に移動
2. 以下のプロパティを検索して設定:

##### プロパティ 1: `com.glide.cs.embed.csp_frame_ancestors`

| フィールド | 値 |
|----------|-----|
| **Name** | `com.glide.cs.embed.csp_frame_ancestors` |
| **Value** | `https://example.com` (外部サイトのドメイン) |
| **Description** | `Allow embedding in external site` |

**複数ドメインの場合**: スペース区切りで指定
```
https://example.com https://another-site.com
```

##### プロパティ 2: `com.glide.cs.embed.xframe_options`

| フィールド | 値 |
|----------|-----|
| **Name** | `com.glide.cs.embed.xframe_options` |
| **Value** | `ALLOWALL` または `SAMEORIGIN` |
| **Description** | `X-Frame-Options setting` |

3. **「Save」** をクリック

---

### 4. 埋め込みコードの取得方法

#### オプション A: ServiceNow公式ドキュメントから取得

**情報源**: [ServiceNow公式ドキュメント](https://www.servicenow.com/docs/ja-JP/bundle/zurich-conversational-interfaces/page/administer/virtual-agent/task/add-portable-va-client-website.html)

公式ドキュメントに記載されている標準的な実装コード：

```html
<script type="module">
    import ServiceNowChat from "https://site1.example.com/uxasset/externals/now-requestor-chat-popover-app/index.jsdbx?sysparm_substitute=false";
    var chat = new ServiceNowChat({
        instance: "https://site1.example.com/",
    });
</script>
```

**重要**: `site1.example.com` を実際のServiceNowインスタンスURLに置き換えてください。

---

#### オプション B: ServiceNow Designerから生成（可能な場合）

一部のServiceNowバージョンでは、Designer画面からコードを生成できる機能があります：

1. **Virtual Agent > Designer** に移動
2. 対象のVirtual Agentトピックを選択
3. **「Portable」** タブを確認
4. コード生成機能があれば使用（バージョンにより異なる）

**注意**:
- この機能はServiceNowのバージョンや設定により利用できない場合があります
- オプションAの公式ドキュメントのコードを使用することを推奨します

---

## 外部ページへの埋め込み手順

### 基本的な実装

外部ウェブサイトのHTMLファイルに以下のコードを追加します。

#### 1. `<head>` セクションにスクリプトを追加

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Your Page Title</title>

    <!-- ServiceNow Virtual Agent Portable Web Client -->
    <script type="module">
        import ServiceNowChat from "https://YOUR-INSTANCE.service-now.com/uxasset/externals/now-requestor-chat-popover-app/index.jsdbx?sysparm_substitute=false";

        // チャットインスタンスを作成して初期化
        const chat = new ServiceNowChat({
            instance: "https://YOUR-INSTANCE.service-now.com/",
        });
    </script>
</head>
<body>
    <!-- ページコンテンツ -->
</body>
</html>
```

**重要**: `YOUR-INSTANCE` を実際のServiceNowインスタンス名に置き換えてください。

---

### カスタマイズ付き実装例

より詳細な設定を行う場合の実装例:

```html
<script type="module">
    import ServiceNowChat from "https://YOUR-INSTANCE.service-now.com/uxasset/externals/now-requestor-chat-popover-app/index.jsdbx?sysparm_substitute=false";

    const chat = new ServiceNowChat({
        // ServiceNowインスタンスURL（必須）
        instance: "https://YOUR-INSTANCE.service-now.com/",

        // コンテキスト設定
        context: {
            skip_load_history: 1  // チャット履歴の読み込みをスキップ
        },

        // ブランディング設定
        branding: {
            bgColor: '#0066cc',        // ボタンの背景色
            primaryColor: '#ffffff',   // テキスト色
            hoverColor: '#0052a3',     // ホバー時の背景色
            activeColor: '#003d7a',    // アクティブ時の背景色
            openIcon: 'custom-open.svg',   // カスタム開くアイコン（オプション）
            closeIcon: 'custom-close.svg', // カスタム閉じるアイコン（オプション）
            sizeMultiplier: 1.0        // サイズの倍率（1.0がデフォルト）
        },

        // 位置設定
        offsetX: 20,           // X軸方向のオフセット（ピクセル）
        offsetY: 20,           // Y軸方向のオフセット（ピクセル）
        position: 'right',     // 'left' または 'right'

        // 多言語対応（オプション）
        translations: {
            'Open dialog': 'チャットを開く',
            'Open Chat. {0} unread message(s)': 'クリックして開く',
            'Close chat.': 'チャットを閉じる'
        }
    });
</script>
```

---

## トラブルシューティング

### 問題: チャットウィジェットが表示されない

#### 原因と解決策:

1. **CORS エラー**
   - **症状**: ブラウザコンソールに `Access to fetch ... has been blocked by CORS policy` エラー
   - **解決策**: ServiceNow側のCORS設定を確認
     - CORSルールが正しく作成されているか確認
     - ドメインが正確に一致しているか確認（`http` vs `https`、末尾の `/` など）

2. **CSP (Content Security Policy) エラー**
   - **症状**: `Framing '...' violates the following Content Security Policy directive` エラー
   - **解決策**: システムプロパティ `com.glide.cs.embed.csp_frame_ancestors` を確認
     - 外部サイトのドメインが含まれているか確認

3. **スクリプト読み込みエラー**
   - **症状**: `Failed to load module script` エラー
   - **解決策**:
     - ServiceNowインスタンスURLが正しいか確認
     - `type="module"` が指定されているか確認
     - インスタンスがアクセス可能か確認

4. **Portable Web Client が有効化されていない**
   - **症状**: 404エラーまたはスクリプトが見つからない
   - **解決策**: Virtual Agent Designer で Portable Web Client を有効化

---

### デバッグ方法

#### 1. ブラウザコンソールの確認

1. ブラウザで外部ページを開く
2. 開発者ツールを開く（F12）
3. **Console** タブでエラーメッセージを確認

#### 2. ネットワークタブの確認

1. 開発者ツールの **Network** タブを開く
2. ページをリロード
3. `index.jsdbx` へのリクエストを確認
   - **ステータス 200**: 正常
   - **ステータス 404**: スクリプトが見つからない（Portable Web Client未有効化）
   - **ステータス 403/CORS エラー**: CORS設定の問題

#### 3. ServiceNow インスタンスの確認

以下のURLに直接アクセスして、スクリプトが存在するか確認:
```
https://YOUR-INSTANCE.service-now.com/uxasset/externals/now-requestor-chat-popover-app/index.jsdbx?sysparm_substitute=false
```

正常な場合はJavaScriptコードが表示されます。

---

## カスタマイズオプション

### 設定パラメータ一覧

| パラメータ | 型 | 必須 | 説明 | デフォルト値 |
|-----------|-----|------|------|-------------|
| `instance` | string | ✓ | ServiceNowインスタンスURL | - |
| `context` | object | - | sys_parm変数 | `{}` |
| `context.skip_load_history` | number | - | 履歴読み込みスキップ（1=スキップ） | `0` |
| `branding.bgColor` | string | - | ボタン背景色（RGB or HEX） | - |
| `branding.primaryColor` | string | - | テキスト色（RGB or HEX） | - |
| `branding.hoverColor` | string | - | ホバー時の色（RGB or HEX） | - |
| `branding.activeColor` | string | - | アクティブ時の色（RGB or HEX） | - |
| `branding.openIcon` | string | - | カスタム開くアイコン | - |
| `branding.closeIcon` | string | - | カスタム閉じるアイコン | - |
| `branding.sizeMultiplier` | number | - | サイズ倍率 | `1.0` |
| `offsetX` | number | - | X軸オフセット（ピクセル） | `0` |
| `offsetY` | number | - | Y軸オフセット（ピクセル） | `0` |
| `position` | string | - | 位置（'left' or 'right'） | `'right'` |
| `translations` | object | - | ラベル翻訳マッピング | `{}` |

---

### 色の指定方法

色は以下の形式で指定できます:

#### HEX形式（推奨）
```javascript
bgColor: '#0066cc'     // 6桁HEX
bgColor: '#06c'        // 3桁HEX（短縮形）
```

#### RGB形式
```javascript
bgColor: '0,102,204'   // カンマ区切りのRGB値
```

---

### よく使うカスタマイズ例

#### 例1: 左下に配置

```javascript
const chat = new ServiceNowChat({
    instance: "https://YOUR-INSTANCE.service-now.com/",
    position: 'left',
    offsetX: 20,
    offsetY: 20
});
```

#### 例2: カスタムカラーテーマ

```javascript
const chat = new ServiceNowChat({
    instance: "https://YOUR-INSTANCE.service-now.com/",
    branding: {
        bgColor: '#ff6600',      // オレンジ
        primaryColor: '#ffffff',  // 白
        hoverColor: '#e65c00',   // ダークオレンジ
        activeColor: '#cc5200'   // より暗いオレンジ
    }
});
```

#### 例3: 大きめのウィジェット

```javascript
const chat = new ServiceNowChat({
    instance: "https://YOUR-INSTANCE.service-now.com/",
    branding: {
        sizeMultiplier: 1.5  // 1.5倍のサイズ
    }
});
```

#### 例4: 日本語ラベル

```javascript
const chat = new ServiceNowChat({
    instance: "https://YOUR-INSTANCE.service-now.com/",
    translations: {
        'Open dialog': 'チャットを開く',
        'Open Chat. {0} unread message(s)': '未読メッセージ {0} 件',
        'Close chat.': 'チャットを閉じる'
    }
});
```

---

## セキュリティに関する考慮事項

### 1. CORS設定の範囲

本番環境では、CORSルールで許可するドメインを必要最小限に制限してください。

**推奨しない設定:**
```javascript
Domain: *  // すべてのドメインを許可（セキュリティリスク）
```

**推奨する設定:**
```javascript
Domain: https://example.com  // 特定のドメインのみ許可
```

### 2. HTTPS の使用

本番環境では必ずHTTPSを使用してください。HTTPでは多くの機能が制限される可能性があります。

### 3. システムプロパティの定期確認

システムプロパティの設定を定期的に確認し、不要なドメインが含まれていないか確認してください。

---

## 外部チャットボタンの非アクティブ化方法

外部サイトに埋め込んだVirtual Agentチャットボタンを、ServiceNow側から制御・非アクティブ化する方法をいくつか紹介します。

### 方法1: Portable Web Clientの無効化（即時・完全停止）

**最も簡単で確実な方法**

#### 手順:
1. **Virtual Agent > Designer** に移動
2. 対象のVirtual Agentトピックを選択
3. **「Portable」** タブを開く
4. **「Disable」** または **「Deactivate」** をクリック

#### 効果:
- すべての外部サイトでチャットウィジェットが即座に読み込まれなくなる
- スクリプトのリクエストが404エラーになる
- 再度有効化すれば復旧可能

#### メリット:
✅ 即座に全サイトで無効化
✅ ServiceNow側の操作のみで完結
✅ 外部サイトのコード変更不要

#### デメリット:
⚠️ すべての外部サイトで無効化される（個別制御不可）
⚠️ 完全停止のため、部分的な制御はできない

---

### 方法2: CORS設定の削除・変更（ドメイン単位の制御）

**特定のドメインのみ非アクティブ化したい場合**

#### 手順:
1. **System Web Services > REST > CORS Rules** に移動
2. 該当するCORSルールを検索
3. オプション:
   - **削除**: ルールを完全に削除
   - **無効化**: Activeフラグをfalseに設定
   - **ドメイン変更**: 許可リストから特定ドメインを削除

#### 効果:
- 指定したドメインからのAPIリクエストがブロックされる
- チャットボタンは表示されるが、機能しない（CORSエラー）

#### メリット:
✅ ドメイン単位で個別制御可能
✅ 他のドメインには影響しない

#### デメリット:
⚠️ チャットボタンは表示されたまま（エラーになる）
⚠️ ユーザー体験が悪い（クリックしてもエラー）

---

### 方法3: システムプロパティによる制御

**CSP設定でフレーム埋め込みをブロック**

#### 手順:
1. **System Properties** に移動
2. `com.glide.cs.embed.csp_frame_ancestors` を検索
3. 値から対象ドメインを削除、または値を空にする

#### 効果:
- Content Security Policyでフレーム埋め込みがブロックされる
- チャットウィンドウが開かない

#### メリット:
✅ セキュリティレベルで制御
✅ ドメイン単位で調整可能

#### デメリット:
⚠️ 完全な無効化にはCORS設定も変更が必要

---

### 方法4: カスタムAPI/設定ファイルによる動的制御（推奨：柔軟な制御）

**最も柔軟で高度な方法**

#### 実装アイデア:

外部サイト側で、ServiceNowのAPIまたは設定ファイルをチェックしてから、チャットを初期化します。

#### ServiceNow側の準備:

1. **カスタムREST APIを作成**（推奨）
   - エンドポイント: `/api/now/v1/va/availability`
   - レスポンス例:
   ```json
   {
     "enabled": true,
     "maintenance": false,
     "message": "Virtual Agent is available"
   }
   ```

2. **または、Public設定ファイルを用意**
   - ServiceNowで静的ファイルをホスト
   - 例: `/va-config.json`

#### 外部サイト側の実装:

```html
<script type="module">
    // 1. ServiceNowからチャット有効状態を取得
    async function checkVAAvailability() {
        try {
            const response = await fetch('https://YOUR-INSTANCE.service-now.com/api/now/v1/va/availability');
            const config = await response.json();
            return config.enabled && !config.maintenance;
        } catch (error) {
            console.error('Failed to check VA availability:', error);
            return false; // エラー時は無効化
        }
    }

    // 2. 有効な場合のみチャットを初期化
    const isAvailable = await checkVAAvailability();

    if (isAvailable) {
        import ServiceNowChat from "https://YOUR-INSTANCE.service-now.com/uxasset/externals/now-requestor-chat-popover-app/index.jsdbx?sysparm_substitute=false"
            .then(module => {
                const ServiceNowChat = module.default;
                const chat = new ServiceNowChat({
                    instance: "https://YOUR-INSTANCE.service-now.com/",
                });
            });
    } else {
        console.log('Virtual Agent is currently unavailable');
        // オプション: ユーザーに代替手段を表示
    }
</script>
```

#### ServiceNow側でのカスタムREST API作成手順:

1. **Scripted REST API** を作成
   - **Name**: `Virtual Agent Availability API`
   - **API ID**: `va_availability`

2. **Resource** を追加
   - **Name**: `Check Availability`
   - **HTTP method**: `GET`
   - **Path**: `/availability`

3. **Script**:
```javascript
(function process(/*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {
    // システムプロパティで制御
    var enabled = gs.getProperty('custom.va.external.enabled', 'true') === 'true';
    var maintenanceMode = gs.getProperty('custom.va.maintenance.mode', 'false') === 'true';

    var result = {
        enabled: enabled && !maintenanceMode,
        maintenance: maintenanceMode,
        message: enabled ? 'Virtual Agent is available' : 'Virtual Agent is disabled',
        timestamp: new GlideDateTime().getDisplayValue()
    };

    response.setStatus(200);
    response.setBody(result);
})(request, response);
```

4. **システムプロパティを作成**:
   - `custom.va.external.enabled`: `true` / `false`
   - `custom.va.maintenance.mode`: `true` / `false`

#### メリット:
✅ ServiceNow側で簡単にON/OFF切り替え可能
✅ メンテナンスモードなど柔軟な制御
✅ 外部サイトでエラーが発生しない
✅ カスタムメッセージを表示可能
✅ ログ取得や分析が可能

#### デメリット:
⚠️ 初期実装に手間がかかる
⚠️ 外部サイト側のコード変更が必要
⚠️ APIレスポンス待ちの遅延がわずかに発生

---

### 方法5: Virtual Agentトピックの非アクティブ化

**Virtual Agent自体を停止**

#### 手順:
1. **Virtual Agent > Designer** に移動
2. 対象のVirtual Agentトピックを選択
3. **「Deactivate」** をクリック

#### 効果:
- Virtual Agent全体が停止
- ポータブルウェブクライアント、サービスポータル両方で使用不可

#### メリット:
✅ 完全停止
✅ 簡単

#### デメリット:
⚠️ サービスポータル側のVirtual Agentも停止
⚠️ 外部サイトのみの制御ができない

---

## 推奨アプローチ

### シナリオ別の推奨方法:

| シナリオ | 推奨方法 | 理由 |
|---------|---------|------|
| **緊急停止が必要** | 方法1: Portable Web Client無効化 | 即座に全停止、最も確実 |
| **特定ドメインのみ停止** | 方法2: CORS削除 | ドメイン単位で制御可能 |
| **定期メンテナンス** | 方法4: カスタムAPI | ユーザー体験が良い、柔軟 |
| **本番環境での運用** | 方法4: カスタムAPI | 最も柔軟で運用しやすい |
| **テスト環境** | 方法1または方法2 | シンプルで十分 |

---

### ベストプラクティス:

1. **段階的な停止**
   - まず方法4でメンテナンスモードを有効化
   - ユーザーに通知
   - その後、必要に応じて方法1で完全停止

2. **複数の制御レイヤー**
   - 通常: カスタムAPIで制御（方法4）
   - 緊急時: Portable Web Client無効化（方法1）

3. **モニタリング**
   - APIアクセスログを監視
   - エラー率を追跡

---

## チェックリスト

埋め込み前に以下を確認してください:

### ServiceNow側

- [ ] Portable Virtual Agent Web Clientが有効化されている
- [ ] CORSルールが作成され、外部ドメインが登録されている
- [ ] HTTPメソッド（GET、POSTなど）が許可されている
- [ ] システムプロパティ `com.glide.cs.embed.csp_frame_ancestors` が設定されている
- [ ] システムプロパティ `com.glide.cs.embed.xframe_options` が設定されている
- [ ] Virtual Agentのトピックが正しく構成されている

### 外部サイト側

- [ ] HTMLファイルに`<script type="module">`が追加されている
- [ ] インスタンスURLが正しく指定されている
- [ ] カスタマイズ設定が適切に行われている（オプション）
- [ ] HTTPSでホスティングされている（本番環境の場合）

### テスト

- [ ] ブラウザコンソールでエラーが出ていない
- [ ] チャットウィジェットが表示される
- [ ] チャットボタンをクリックできる
- [ ] チャットウィンドウが開く
- [ ] Virtual Agentとの会話が可能

---

## 参考リンク

### 公式ドキュメント

- **[ServiceNow公式 - Portable VA Client埋め込み手順（日本語）](https://www.servicenow.com/docs/ja-JP/bundle/zurich-conversational-interfaces/page/administer/virtual-agent/task/add-portable-va-client-website.html)**
  - このドキュメントの主要な情報源
  - 埋め込みコードサンプル、設定スキーマ、パラメータ説明を含む

- **[ServiceNow Docs - Portable Virtual Agent Web Client](https://docs.servicenow.com/en-US/bundle/vancouver-servicenow-platform/page/administer/virtual-agent/task/add-portable-va-client-website.html)**
  - 英語版の公式ドキュメント
  - 最新バージョンの情報

### コミュニティリソース

- **[ServiceNow Community - Virtual Agent Forum](https://www.servicenow.com/community/virtual-agent-nlu-forum/bd-p/virtual-agent-nlu-forum)**
  - コミュニティフォーラムで質問・回答を検索

- **[How to embed Virtual Agent in an external site](https://www.servicenow.com/community/virtual-agent-nlu-articles/how-to-embed-virtual-agent-in-an-external-site-updated-for-tokyo/ta-p/2308092)**
  - コミュニティ記事：外部サイトへの埋め込み方法

### 参考にした情報源

- ServiceNow公式ドキュメント（Zurich Release）
- ServiceNowコミュニティフォーラムの事例
- 実装テストの結果（blueshipcoltddemo4インスタンス）

---

## サポート

問題が解決しない場合は、以下を確認してください:

1. ServiceNowのバージョンが最新か
2. ブラウザが最新版か
3. ServiceNowコミュニティフォーラムで類似の問題が報告されていないか

---

## バージョン履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|---------|
| 1.0 | 2025-01-XX | 初版作成 |

---

## ライセンス

このドキュメントはMITライセンスのもとで公開されています。
