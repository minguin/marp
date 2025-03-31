---
marp: true
theme: default
paginate: true
size: 16:9
style: |
  /* シンプルでベーシックなスタイル */
  section {
    font-family: 'Helvetica', 'Arial', 'Hiragino Sans', 'Meiryo', sans-serif;
    padding: 30px;
    background-color: white;
  }
  /* タイトルスライド用 */
  section.title-slide {
    text-align: center;
    display: flex;
    flex-direction: column;
    justify-content: center;
    background-color: #f5f5f5;
  }
  section.title-slide h1 {
    font-size: 2.5em;
    margin-bottom: 0.3em;
  }
  section.title-slide h2 {
    font-size: 1.5em;
    font-weight: normal;
    color: #555;
  }
  /* 通常スライド用 */
  h1 {
    font-size: 1.8em;
    color: #333;
    border-bottom: 1px solid #ccc;
    padding-bottom: 0.2em;
    margin-bottom: 0.6em;
  }
  h2 {
    font-size: 1.3em;
    color: #444;
    margin-top: 0.2em;
    margin-bottom: 0.4em;
  }
  h3 {
    font-size: 1.1em;
    color: #555;
  }
  ul, ol {
    margin: 0.4em 0;
    padding-left: 1.2em;
    line-height: 1.4;
  }
  li {
    margin: 0.2em 0;
    font-size: 0.95em;
  }
  code {
    background-color: #f5f5f5;
    padding: 0.1em 0.3em;
    border-radius: 3px;
    font-size: 0.85em;
  }
  .columns {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1.5em;
  }
  .box {
    background-color: #f8f8f8;
    border: 1px solid #eee;
    border-radius: 5px;
    padding: 10px;
    margin: 8px 0;
    font-size: 0.9em;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    margin: 0.7em 0;
    font-size: 0.85em;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 6px 10px;
    text-align: left;
  }
  th {
    background-color: #f5f5f5;
  }
  .caption {
    font-size: 0.7em;
    color: #666;
    text-align: center;
    margin-top: 3px;
  }
  .code-block {
    font-size: 0.75em;
    line-height: 1.3;
    max-height: 350px;
    overflow: auto;
  }
  pre {
    font-size: 0.75em;
    line-height: 1.3;
  }
---

<!-- _class: title-slide -->

# Claude の Model Context Protocol (MCP)
## AIと外部システムの連携を可能にする標準プロトコル

---

# MCPとは何か？

<div class="columns">
<div>

- Anthropicが開発した**オープンプロトコル**
- AIアシスタントと外部システムを接続する**標準インターフェース**
- 「AIアプリケーション向けのUSB-Cポート」のような役割
- AIが様々なツールやデータソースと安全に対話できる仕組み

</div>
<div>

**概念図**: Claude AIと外部システム（ファイルシステム、データベース、APIなど）が
MCPを介して双方向に通信する標準インターフェース

<div class="box">
<b>MCPの主な特徴</b><br>
✅ 標準化されたインターフェース<br>
✅ 双方向通信をサポート<br>
✅ セキュリティを確保<br>
✅ 拡張性と相互運用性
</div>

</div>
</div>

---

# MCPの仕組み

<div class="columns">
<div>

## MCPの基本構成
- **MCP Server**: 外部システムとの接続を提供
- **MCP Client**: AIアシスタント側（Claude）
- **Tool**: MCPサーバーが提供する機能単位
- **Transport**: 通信方式（標準入出力、HTTP等）

## ツールとコンテキスト
- AIからツールを呼び出し、結果を受け取る
- 複数のツールを組み合わせて複雑なタスクを実行
- ツールは型付きスキーマで定義される

</div>
<div>

**アーキテクチャ概要**:
- Claude AI → MCP Client → Transport → MCP Server
- MCP Serverは各種ツール（ファイル操作、データベース、API）を提供
- 各ツールは対応する外部システムと連携

<div class="caption">MCPは多層構造で安全な通信を実現</div>

</div>
</div>

---

# MCPで接続可能なシステム・サービス

<table>
  <tr>
    <th>カテゴリ</th>
    <th>具体例</th>
    <th>主な用途</th>
  </tr>
  <tr>
    <td>ローカルシステム</td>
    <td>ファイルシステム、SQLite/PostgreSQL、実行環境</td>
    <td>ローカルファイルの読み書き、データベース操作、コード実行</td>
  </tr>
  <tr>
    <td>クラウドサービス</td>
    <td>Google Drive, Slack, GitHub, Google Calendar</td>
    <td>ドキュメント管理、コミュニケーション、コード管理、予定管理</td>
  </tr>
  <tr>
    <td>開発ツール</td>
    <td>Git, Puppeteer, VS Code</td>
    <td>バージョン管理、ウェブ自動化、コード開発</td>
  </tr>
  <tr>
    <td>その他</td>
    <td>Figma, YouTube, Pandoc, Blender</td>
    <td>デザイン連携、動画分析、文書変換、3Dモデリング</td>
  </tr>
</table>

**拡張性**: 新しいシステムやサービスに対応するMCPサーバーを自作することも可能

---

# MCPサーバーの実装例

<div class="columns">
<div>

## 基本的なMCPサーバー
```typescript
import { Server } from "@modelcontextprotocol/sdk/server";
import { StdioServerTransport } from 
  "@modelcontextprotocol/sdk/server/stdio";

const server = new Server({
  name: "hello-world-server",
  version: "1.0.0",
});

// ツールの定義
server.tool(
  "hello", // ツール名
  { name: { type: "string" } }, // 入力スキーマ
  async ({ name }) => {
    return {
      content: [{ type: "text", text: `Hello, ${name}!` }]
    };
  }
);

// サーバー起動
const transport = new StdioServerTransport();
await server.connect(transport);
```

</div>
<div>

## ファイル操作MCPサーバー
```typescript
import { Server } from "@modelcontextprotocol/sdk/server";
import { StdioServerTransport } from 
  "@modelcontextprotocol/sdk/server/stdio";
import fs from "fs/promises";

const server = new Server({
  name: "filesystem-server",
  version: "1.0.0",
});

server.tool(
  "read_file",
  { path: { type: "string" } },
  async ({ path }) => {
    const content = await fs.readFile(path, "utf-8");
    return {
      content: [{ type: "text", text: content }]
    };
  }
);

// サーバー起動
const transport = new StdioServerTransport();
await server.connect(transport);
```

</div>
</div>

---

# MCPの設定と使用方法

<div class="columns">
<div>

## 設定方法
1. MCPサーバーのコードを作成・実行
2. Claude for Desktopの設定ファイルを編集
   ```json
   {
     "mcpServers": {
       "myserver": {
         "command": "node",
         "args": ["path/to/your/mcp-server.js"]
       }
     }
   }
   ```
3. Claudeを再起動

## 利用方法
- Claudeに直接指示するだけで連携が可能
- 例：「ローカルのファイル一覧を表示して」
- APIキーなどの認証情報はMCPサーバーで管理

</div>
<div>

## 実際の活用例
- ローカルファイルの分析と要約
- データベースからの情報取得と可視化
- 外部APIを使った情報収集と整理
- 複数のシステム間でのワークフロー自動化

<div class="box">
<b>MCPの主なメリット</b><br>
✅ データをクラウドに送信せずローカル処理<br>
✅ 複数のシステムを統合的に利用可能<br>
✅ カスタムツールで独自機能を拡張<br>
✅ セキュリティとプライバシーの確保
</div>

</div>
</div>

---

# 外部API連携のMCPサーバー例

```typescript
import { Server } from "@modelcontextprotocol/sdk/server";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio";
import axios from "axios";

const server = new Server({
  name: "weather-server",
  version: "1.0.0",
});

// 天気情報取得ツール
server.tool(
  "get_weather",
  { city: { type: "string" } },
  async ({ city }) => {
    const apiKey = process.env.WEATHER_API_KEY;
    const url = `http://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}`;
    
    const response = await axios.get(url);
    const weather = response.data.weather[0].description;
    const temp = response.data.main.temp - 273.15; // Convert to Celsius
    
    return {
      content: [{ type: "text", text: `${city}の天気は${weather}で、気温は${temp.toFixed(1)}°Cです。` }]
    };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

---

# MCPの将来性と発展

<div class="columns">
<div>

## 今後の展望
- **エコシステムの拡大**: より多くのサービス・ツールとの連携
- **複雑なワークフロー**: 複数のツールを組み合わせた高度な処理
- **標準化の進展**: AIエージェント間の相互運用性向上
- **セキュリティ強化**: より安全な認証・権限管理の仕組み

## 課題
- MCPサーバー開発の敷居の低減
- パフォーマンスと安定性の向上
- ユーザーのプライバシー保護と透明性確保

</div>
<div>

<div class="box">
<b>MCPの応用領域</b>

- **企業内システム連携**: 社内ツールとAIの橋渡し
- **パーソナルアシスタント**: 個人のデータと連携した支援
- **開発者支援**: コード生成・テスト・デプロイの自動化
- **データ分析**: 複数ソースからのデータ統合と分析
</div>

**エコシステム図**: Claudeが様々なシステムと連携可能
- **ローカルシステム**: ファイルシステム、データベース、コード実行環境
- **クラウドサービス**: Google Drive、GitHub、Slack
- **独自システム**: カスタムAPI、社内システム

</div>
</div>

---

# まとめ

<div class="columns">
<div>

## MCPの主要ポイント
- AIと外部システムを**標準化されたプロトコル**で接続
- ローカルシステム、外部API、各種サービスとの連携が可能
- **JavaScript/TypeScript**を使った簡単なサーバー実装
- Claude for Desktopでの利用がすぐに可能

## 学習リソース
- [Anthropic MCP公式ドキュメント](https://docs.anthropic.com/docs/build-with-claude/mcp)
- [Model Context Protocol SDK](https://github.com/anthropics/anthropic-model-context-protocol)
- [Claude for Desktop設定ガイド](https://docs.anthropic.com/docs/claude-for-desktop)

</div>
<div>

<div class="box">
<b>MCPの本質</b><br><br>
MCPは単なる技術仕様ではなく、AIと既存システムの共存・連携の基盤として、AIの実用性と有用性を大きく高める可能性を秘めています。<br><br>
オープンなプロトコルとして公開されることで、エコシステム全体の発展に貢献し、AIの適用範囲を大きく広げるでしょう。
</div>

<div style="text-align: center; margin-top: 20px;">
<b>「MCPはAIアプリケーション向けのUSB-Cポート」</b><br>
- Anthropic より -
</div>

</div>
</div>