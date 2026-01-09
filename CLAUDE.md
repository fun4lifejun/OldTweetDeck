# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。
このリポジトリは dimdenGD/OldTweetDeck の fork です。fork元の最新を取り込むには `git pull upstream main` を実行してください。

## ビルドコマンド

```bash
# 依存関係のインストール
npm install

# 拡張機能パッケージのビルド（OldTweetDeckChrome.zip と OldTweetDeckFirefox.zip を生成）
npm run build
```

## アーキテクチャ概要

OldTweetDeck は、Twitter の新しいインターフェースをインターセプトし、旧版 TweetDeck アプリケーションに置き換えるブラウザ拡張機能です。

### コア実行フロー

1. **destroyer.js** (MAIN world, document_start) - `__SCRIPTS_LOADED__` をフリーズし、`Array.prototype.push` をハイジャックして webpack チャンク注入を阻止、エラー報告を抑制することで、Twitter の新しい React アプリの読み込みをブロック
2. **content.js** (document_start) - `window.postMessage` を介して拡張機能コンテキストとページコンテキストを橋渡しし、Cookie/トークンをリレー
3. **injection.js** (MAIN world) - ローカルファイルまたは GitHub から旧版 TweetDeck リソースを読み込み、旧 UI をレンダリングし、Twitter の DOM を削除

### 主要ファイル

| ファイル | 役割 |
|---------|------|
| `src/interception.js` | Twitter の GraphQL API レスポンスを旧版 REST API 形式に変換（2,900行以上のデータマッピング） |
| `src/injection.js` | 旧版 TweetDeck リソースの読み込みと DOM 置換を統括 |
| `src/destroyer.js` | Twitter の新しい JavaScript を妨害し、新 UI の読み込みを阻止 |
| `src/background3.js` | Chrome/Edge 用 Service Worker - x.com と twitter.com 間の Cookie 同期を処理 |
| `src/background.js` | Firefox 用バックグラウンドスクリプト |
| `src/challenge.js` | iframe ベースの CAPTCHA チャレンジソルバー |
| `files/bundle.js` | 旧版 TweetDeck アプリケーションの完全なコード |
| `files/decider.json` | 旧版 TweetDeck 機能に必要なフィーチャーフラグ |
| `ruleset.json` | CSP 削除とドメインリダイレクト用の Declarative Net Request ルール |

### ビルドシステム (pack.js)

ビルドスクリプトは2つのパッケージを生成:
- **Chrome**: Manifest v3 を使用、`destroyer.js` と declarativeNetRequest を含む
- **Firefox**: Manifest v2 に変換、webRequest 権限を追加、`destroyer.js` を削除（Firefox では不要）

### 外部依存関係

- **GitHub**: `https://raw.githubusercontent.com/dimdenGD/OldTweetDeck/main/` から最新ファイルの自動更新チェック
- **oldtd.org**: 追加スクリプトと通知 API

### localStorage キー

localStorage に保存される主要データ:
- `OTDalwaysUseLocalFiles`: リモート更新の代わりにローカルファイルを強制使用
- `OTDfollowsData`: キャッシュされたフォロイーデータ（6時間 TTL）
- `OTDtoken`: OTD API トークン
