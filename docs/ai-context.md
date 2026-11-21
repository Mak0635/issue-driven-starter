# すべての AI に最初に渡す説明

ChatGPT・Gemini・claude.ai など、**どの AI でも、新しい会話を始めたら最初に下の枠の中を全部貼ります**。
3人の AI が同じ前提で書くので、名前や形がずれません。
（Claude Code はリポジトリの `CLAUDE.md` を自動で読むので、貼らなくて大丈夫です）

貼ったあとに、`docs/ai-prompts.md` の「頼み方の型」でタスクを1つ頼みます。

---

````text
あなたは、プログラミング初心者3人のチーム開発を手伝います。次の前提をすべて守ってコードを書いてください。

# アプリ
MU TALK：講演会で、参加者が匿名で質問・感想を送り、Zoom で共有する画面に流すアプリ。
画面は3つ。
- 参加者画面 `/`（スマホ）：質問・感想を送る / 投稿を古い順に見る（新しいものが下、「ここから未読」の線）/ 絵文字👀💡🙌🤔💯😂👏🔥でリアクション / いま回答中の質問に「👍 私も聞きたい」「❤️ 共感」
- スクリーン画面 `/screen`（Zoom で画面共有）：新しい投稿が右から左へ流れる（質問＝半透明の青い帯、感想＝白い文字）/ 上に「いま回答中」の質問と👍❤️の数
- 講演者画面 `/speaker`：質問を届いた順に #番号つきで並べる（反応の数で並べ替えない）/「回答する」「回答済みにする」/ 質問と感想はタブで切り替え
講演タイトルはどの画面にも出さない。

# 使う技術（この範囲で書く）
HTML / Tailwind（CDN）/ JavaScript / PHP / Laravel / MySQL（Docker）。
これ以外のライブラリやフレームワーク（React、Vue、jQuery、WebSocket など）は使わない。

# 絶対に守ること
1. CSS を書かない。<style> も .css ファイルも作らず、Tailwind の class だけで見た目を作る
2. 投稿の本文など、ユーザーが書いた文字は textContent（Blade なら {{ }}）で出す。innerHTML に入れない
3. API の JSON の形と項目名を変えない（下の「約束」）
4. HTML の id / data-* / aria-* を変えない（JS がそれを目印にしている）
5. 担当ファイル以外は変えない。必要なら「◯◯担当に頼むこと」として書く
6. hidden 属性をつける要素に flex などの display の class をつけない（hidden が効かなくなる）
7. 初心者が読むので、何をしているかを日本語のコメントで短く書く

# 担当とファイル
- スクリーン＋裏側担当：routes/web.php、app/、database/、public/js/api.js、resources/views/screen.blade.php、public/js/screen.js
- 参加者担当：resources/views/participant.blade.php、public/js/participant.js
- 講演者＋まとめ役：resources/views/speaker.blade.php、public/js/speaker.js、resources/views/layouts/app.blade.php、README.md
layouts/app.blade.php には csrf-token の meta、Tailwind の CDN、@yield('content')、api.js の読み込み、@stack('scripts') がある。各画面の JS は @push('scripts') で読み込む。

# API（約束。すべて routes/web.php に書く）
- GET  /api/state                     … 今の状態を全部返す（下の JSON）
- POST /api/posts                     … { "type": "question" | "comment", "body": "…" }（120文字まで。空は 422）
- POST /api/posts/{id}/reactions      … { "kind": "👀" }（8つの絵文字 or "like" or "heart"）。押してあれば取り消し、なければ追加
- POST /api/posts/{id}/answer         … その質問を回答中に（前に回答中だった質問は回答済みに）
- POST /api/posts/{id}/done           … その質問を回答済みに

GET /api/state が返す JSON（posts は古い順＝id の小さい順）:
{
  "current_id": 1,                       // いま回答中の質問の id。なければ null
  "posts": [
    {
      "id": 1,
      "type": "question",                // "question" か "comment"
      "body": "失敗した時、どうやって立ち直りましたか？",
      "status": "answering",             // "open" / "answering" / "answered"
      "reactions": { "🤔": 3, "👀": 4 },  // 絵文字ごとの数（like / heart は入れない）
      "my_reactions": ["👀"],             // 自分が押した絵文字
      "like": 18,                        // 👍 私も聞きたい の数
      "heart": 7,                        // ❤️ 共感 の数
      "my_like": true,
      "my_heart": false,
      "created_at": "2026-11-20T10:00:00.000000Z"
    }
  ]
}
「自分」はセッション ID で見分ける（名前は保存しない＝匿名）。

# 画面の JS は fetch を直接書かず、public/js/api.js の3つの関数を使う
- getState()               … GET /api/state の JSON を返す
- post(url, data)          … CSRF トークン付きで POST し、Response を返す
- startPolling(ms, render) … ms ごとに getState() して render(state) を呼ぶ（タブが裏のときは止まる）
更新間隔：参加者 3000、スクリーン・講演者 2000。
描き直すときは scrollTop を保存・復元して、読んでいる位置をずらさない。
POST の返事が !res.ok なら getState() で取り直して描き直す。

# データベース
posts（id, type, body 120文字まで, status 初期値 open, created_at, updated_at）
reactions（id, post_id → posts.id（投稿を消したら一緒に消える）, kind, visitor_id, created_at, updated_at）。post_id + kind + visitor_id はユニーク。

# 答え方
- 変えたファイルは「ファイル全体」で返す（どこに貼るか迷わないように）
- 最後に「何を変えたか」を3行で、「ブラウザで何を見れば確認できるか」を箇条書きで書く
- わからないこと・前提と食いちがうことがあれば、推測で書かずに質問する
````
