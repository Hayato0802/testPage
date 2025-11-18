# ServiceNow Virtual Agent Test Page

このリポジトリは、ServiceNow Virtual Agentの埋め込みウィジェットをテストするためのシンプルなWebページです。

## 🚀 GitHub Pagesでの公開

このページは GitHub Pages で公開されています: https://hayato0802.github.io/testPage/

## 📚 ドキュメント

**[完全なセットアップ手順書を見る](SETUP_GUIDE.md)**

以下の内容を含む包括的なガイド：
- ServiceNow側の設定手順（CORS、システムプロパティなど）
- 外部ページへの埋め込み方法
- トラブルシューティング
- カスタマイズオプション
- **外部チャットボタンの非アクティブ化方法**（5つの方法を紹介）

## 📝 クイックスタート

### 1. ServiceNow側の設定

詳細は[SETUP_GUIDE.md](SETUP_GUIDE.md#servicenow側の設定手順)を参照してください。

- Portable Virtual Agent Web Clientを有効化
- CORS設定を構成
- システムプロパティを設定

### 2. 外部サイトに埋め込み

```html
<link rel="stylesheet" href="style.css">

<script type="module">
    import ServiceNowChat from "https://YOUR-INSTANCE.service-now.com/uxasset/externals/now-requestor-chat-popover-app/index.jsdbx?sysparm_substitute=false";

    const chat = new ServiceNowChat({
        instance: "https://YOUR-INSTANCE.service-now.com/",
    });
</script>
```

## 🎛️ チャットボタンの制御

外部サイトに埋め込んだチャットボタンをServiceNow側から非アクティブ化する方法：

1. **即時停止**: Portable Web Clientを無効化
2. **ドメイン単位**: CORS設定を変更
3. **柔軟な制御**: カスタムAPIで動的制御（推奨）

詳細は[SETUP_GUIDE.md - 外部チャットボタンの非アクティブ化方法](SETUP_GUIDE.md#外部チャットボタンの非アクティブ化方法)を参照してください。

## 🔧 カスタマイズ

- `index.html`: HTMLコンテンツ
- `style.css`: スタイル定義
- スクリプト内の`branding`オブジェクトで色や位置を調整

## 📁 ファイル構成

```
testPage/
├── index.html          # メインHTMLファイル
├── style.css           # スタイル定義
├── SETUP_GUIDE.md      # 完全なセットアップガイド
└── README.md           # このファイル
```

## 🔗 参考リンク

- [ServiceNow公式ドキュメント（日本語）](https://www.servicenow.com/docs/ja-JP/bundle/zurich-conversational-interfaces/page/administer/virtual-agent/task/add-portable-va-client-website.html)
- [ServiceNow Community - Virtual Agent Forum](https://www.servicenow.com/community/virtual-agent-nlu-forum/bd-p/virtual-agent-nlu-forum)

## 📄 ライセンス

MIT License
