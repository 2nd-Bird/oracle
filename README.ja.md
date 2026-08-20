# oracle 🧿 — fork: ChatGPT の過去チャットを知識グラフに取り込む

> これは [steipete/oracle](https://github.com/steipete/oracle)（MIT, © Peter Steinberger）のフォーク [2nd-Bird/oracle](https://github.com/2nd-Bird/oracle) です。追加したのは `oracle conversation export` の1機能だけで、それ以外は upstream の Oracle そのものです。upstream には [PR #402](https://github.com/steipete/oracle/pull/402) として提出済みで、マージされるまではこのフォークを `npm install -g github:2nd-Bird/oracle#feat/chatgpt-conversation-export` で入れてください。英語版 README は [こちら](README.md)。

## 課題

Codex や Claude Code のセッションログを Obsidian に入れて知識グラフ化すると、判断・失敗・動いたスニペットが全部リンク可能なノードになり、生産性が積み上がっていきます。ところが、エージェント登場**以前**に ChatGPT で積み上げてきた何年分もの思考は、このグラフの外に取り残されたままです。マージしようとすると、長いスレッドはとっくにコンテキストから落ちていて、公式エクスポートは巨大な JSON ひとつ、手作業でのコピペは数百ターンになると人間がもちません。

## 解決

エージェントにやらせます。`oracle conversation export` は、Oracle が既に持っている「ログイン済みの ChatGPT タブ」を使って、既存の会話を**読み取り専用**で（プロンプトは送らない・クリックしない・遷移しない）取得し、**クエリと回答 1 組につき 1 つの Markdown ノート**として Git 管理下のフォルダに書き出します。frontmatter・wikilink・会話ごとの `INDEX.md` 付きなので、Obsidian はそのまま読み込めます。エージェント以前の ChatGPT 資産が、エージェントのログと同じグラフの一部になります。

Obsidian は必須ではありません。単に、ChatGPT の会話を**原文のまま**（SHA-256 付き）自分のリポジトリに保存する、grep も diff もできるアーカイブとしても使えます。

**Codex からでも Claude Code からでも**（ただの CLI なので）自分のシェルからでも同じように動きます。

```bash
# 1. 初回のみ: DevTools リモートデバッグを有効にした Chrome を起動し、
#    ChatGPT にログインしておく（docs/browser-mode.md の "Remote Chrome Sessions"）。

# 2. 会話を vault の inbox に書き出す。ログイン済みの ChatGPT タブが 1 つあればよく、
#    その会話自体を開いている必要はない。
oracle conversation export "https://chatgpt.com/c/<conversation-id>" \
  --format obsidian --out ./00_Inbox --timezone Asia/Tokyo

# 3. ほかのノートと同じように commit する。
git add 00_Inbox/ChatGPT-* && git commit -m "archive: chatgpt conversation"
```

生成物:

```
00_Inbox/ChatGPT-685b5c1d/
├── INDEX.md                         # 要約と全ノートへの wikilink
├── 001-2025-06-25-turn-001.md       # クエリ 1 件 + その回答、原文のまま
├── 002-2025-06-25-turn-003.md
└── …
```

各ノートには `conversation_id`・`source_url`・元のタイムスタンプ（指定タイムゾーンの日付）・turn id・`query_sha256` / `answer_sha256` が入るので、後からでも「改変されていない」ことを証明できます。取り込み時には一切要約しません。**raw first, organize later** — これが一次資料として信頼できるアーカイブにする条件です。

内部的には ChatGPT の UI 自身が描画に使っている `/backend-api/conversation/<id>` の JSON を読むので、スレッド全体を一発で取得できます。分岐、回答ごとのモデル、canvas ドキュメント、UI には表示されない「思考のみ」のターンも見えます。従来の DOM スクロール方式は `--source dom` で残してあります。`--format` で JSON / Markdown / バックエンドの生 JSON も出せます。詳細は [CLI reference](docs/cli-reference.md#conversation-export)。

エージェントに任せるときの指示例:

> この ChatGPT の会話を raw-first で保存して。`oracle conversation export <url> --format obsidian --out ./00_Inbox` を実行し、書き出されたものは要約も編集もせず、`00_Inbox` の下に置いたままにすること。vault の他の場所への昇格は別の工程で後からやる。

## ライセンス

upstream と同じ MIT。著作権表示は [LICENSE](LICENSE) のとおり Peter Steinberger のものを保持しています。
