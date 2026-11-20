# Issue の本文まとめ

GitHub で **New issue → 「タスク」** を選び、下の各カードの「タイトル」と本文をコピーして立ててください。
立てた順に番号がつくので、**#1 から順番に** 立てると README の番号と一致します。

---

## #1 チームのルールを決めて Issue を全部立てる
担当: 全員 / 目安 30分 / ラベル: `MVP`

### 概要
3人が同じ地図を見て動けるようにする。

### やること
- [ ] README を全員で読む（特に「4. 時間割」「7. ルール」「8. 設計図」）
- [ ] A・B・C をだれがやるか決めて、README の表に名前を書く
- [ ] ラベルを作る: `担当:A` `担当:B` `担当:C` `MVP` `おまけ` `バグ`
- [ ] このファイルの #2〜#15 を Issue として立て、Assignees とラベルをつける
- [ ] Settings → Branches で main を保護する（できれば。PRなしで push できないように）
- [ ] 連絡用のチャット（Slack / Discord / LINE など）を決める

### 完了条件
- Issue が #15 まで立っていて、全部に担当者がついている

---

## #2 Laravel プロジェクトを作って main に置く
担当: A / 目安 1時間 / ラベル: `担当:A` `MVP`

### 概要
📘 手順は [docs/laravel-setup.md](laravel-setup.md) の 0〜8 を見ながら進める。

みんなが作業する土台（Laravel）を用意する。これが終わるまで B・C は `mockups/` の HTML で作業する。

### やること
- [ ] 授業と同じ方法で Laravel プロジェクトを作る
  - このリポジトリにはすでにファイルがあるので、**別のフォルダで作ってから中身をこのリポジトリの一番上にコピー** する
  - Laravel が作る `README.md` は **コピーしない**（チームの README が消えるため）
- [ ] `resources/views/layouts/app.blade.php` を作る
  - `<meta name="csrf-token" content="{{ csrf_token() }}">`
  - `<script src="https://cdn.tailwindcss.com"></script>`
  - `@yield('content')` と `@stack('scripts')`
- [ ] `participant.blade.php` / `screen.blade.php` / `speaker.blade.php` を、中身「準備中」だけで作る
- [ ] `routes/web.php` に `/` `/screen` `/speaker` を作る
- [ ] `public/js/participant.js` `screen.js` `speaker.js` を空で作る
- [ ] README の「起動のしかた」を実際の手順に書き直す
- [ ] PR → マージしたらチャットで「Laravel できたよ！ pull してね」と伝える

### 変更する場所
リポジトリ全体（Laravel の新規ファイル）, `README.md` の「起動のしかた」

### 完了条件
- B と C が clone → README の手順で起動し、3つの URL に「準備中」が表示される

---

## #3 データベースの表（posts / reactions）とサンプルデータを作る
担当: A / 目安 1時間 / ラベル: `担当:A` `MVP`

### 概要
📘 手順は [docs/laravel-setup.md](laravel-setup.md) の 9〜11 を見ながら進める。

質問・感想・リアクションを保存する場所（黒板）を作る。

### やること
- [ ] `php artisan make:model Post -m` で posts の表を作る（列は README「8. 設計図」どおり）
- [ ] `php artisan make:model Reaction -m` で reactions の表を作る
  - `post_id` は `foreignId('post_id')->constrained()->cascadeOnDelete()`
  - `unique(['post_id', 'kind', 'visitor_id'])`
- [ ] モデルに `$fillable` とリレーション（`Post hasMany Reaction` / `Reaction belongsTo Post`）を書く
- [ ] シーダーで、プロトタイプと同じサンプル投稿を5件ほど入れる
- [ ] `php artisan migrate:fresh --seed` で作れることを確認

### 変更する場所
`database/migrations/*`, `database/seeders/*`, `app/Models/*`

### 完了条件
- `migrate:fresh --seed` でエラーなく表とサンプルデータができる

---

## #4 参加者画面の見た目を作る
担当: B / 目安 2時間 / ラベル: `担当:B` `MVP`

### 概要
スマホで見る投稿画面の形を作る（まだ動かなくてよい）。

### やること
- [ ] プロトタイプの「参加者」画面（スマホ幅）をまねて、Tailwind の class だけで作る
  - 上: タイトル、「投稿は匿名で表示されます」
  - 「いま回答中」の箱（👍 私も聞きたい / ❤️ 共感 ボタン）
  - タブ: 一覧 / 質問 / 感想
  - 投稿カード（質問・感想バッジ、本文、絵文字チップ、＋ボタン）
  - 下に固定: 「質問する／感想を書く」切り替え、入力欄、送信ボタン
- [ ] #2 が終わるまでは `mockups/participant.html` で作り、終わったら `participant.blade.php` に移す
- [ ] カードやボタンに、JS から探しやすい `id` を付けておく（例: `id="list"` `id="form"`）

### 変更する場所
`mockups/participant.html` → `resources/views/participant.blade.php`

### 完了条件
- スマホ幅（開発者ツールで iPhone 表示）でプロトタイプに近い見た目になっている
- `<style>` を使っていない

---

## #5 スクリーン画面の見た目を作る
担当: C / 目安 1時間 / ラベル: `担当:C` `MVP`

### やること
- [ ] 画面いっぱいの紺色の背景（Zoom の画面共有のイメージ）
- [ ] 上に「いま回答中」バナー（質問文・👍◯人が聞きたい・❤️◯人が共感）
- [ ] 文字が流れる場所（`id="sky"`、`relative overflow-hidden`）
- [ ] 左下に「● 青い帯＝質問 / 白い文字＝感想」の説明
- [ ] まず `mockups/screen.html`、#2 のあと `screen.blade.php` へ

### 完了条件
- PC の全画面で、プロトタイプのスクリーンに近い見た目

---

## #6 講演者画面の見た目を作る
担当: C / 目安 1時間 / ラベル: `担当:C` `MVP`

### やること
- [ ] 左: 質問の一覧（反応の数・質問文・「回答する」ボタン）、いま回答中の行は濃い青
- [ ] 下に「回答済み」の一覧（うすく表示）
- [ ] 右: 感想（新しい順）
- [ ] 上: 「いまの質問を聞きたい ◯ / 共感 ◯」
- [ ] まず `mockups/speaker.html`、#2 のあと `speaker.blade.php` へ

### 完了条件
- PC 画面でプロトタイプの講演者画面に近い見た目

---

## #7 投稿の一覧・保存 API を作る
担当: A / 目安 1.5時間 / ラベル: `担当:A` `MVP`
**B・C がこれを待っているので、A の API の中で一番先にやる**

### やること
- [ ] `GET /api/state` … README「8. 設計図」の JSON の形どおりに返す
  - 自分のしるし: `$request->session()->getId()` を `visitor_id` として使う
  - `reactions`・`my_reactions`・`like`・`heart`・`my_like`・`my_heart` を計算する
- [ ] `POST /api/posts` … バリデーション
  - `type` は `required|in:question,comment`
  - `body` は `required|string|max:120`
- [ ] 作ったらブラウザで `/api/state` を開いて JSON が出るか確認

### 変更する場所
`routes/web.php`, `app/Http/Controllers/PostController.php`（など）

### 完了条件
- `/api/state` を開くと設計図どおりの JSON が出る
- 空の投稿は 422 エラーになる

---

## #8 リアクション（絵文字・聞きたい・共感）API を作る
担当: A / 目安 1時間 / ラベル: `担当:A` `MVP`

### やること
- [ ] `POST /api/posts/{post}/reactions` に `{ "kind": "👀" }` を受け取る
- [ ] `kind` は 8つの絵文字か `like` / `heart` だけOK（`in:` でバリデーション）
- [ ] 同じ人・同じ投稿・同じ kind があれば削除、なければ追加（トグル）

### 完了条件
- 2回押すと元に戻る。別のブラウザ（シークレットウィンドウ）からは別の人として数が増える

---

## #9 「回答する」「回答済み」API を作る
担当: A / 目安 1時間 / ラベル: `担当:A` `MVP`

### やること
- [ ] `POST /api/posts/{post}/answer` … 今 `answering` の質問を `answered` にしてから、この質問を `answering` に
- [ ] `POST /api/posts/{post}/done` … この質問を `answered` に
- [ ] `/api/state` の `current_id` に反映されることを確認

### 完了条件
- answer を呼ぶと `current_id` が変わり、前の質問は `answered` になる

---

## #10 参加者画面: 投稿を送る・一覧を自動更新する
担当: B / 目安 2.5時間 / ラベル: `担当:B` `MVP`

### やること
- [ ] 「質問する／感想を書く」の切り替え（押した方の色が変わる）
- [ ] 送信ボタンで `POST /api/posts`。空なら送信ボタンを押せないようにする
- [ ] Enter で送信、Shift+Enter で改行（日本語変換中の Enter は送らない: `e.isComposing`）
- [ ] 2秒ごとに `GET /api/state` → 一覧を新しい順で描き直す
- [ ] 本文は **`textContent`** で表示する
- [ ] ⭐ タブ（一覧／質問／感想）と件数
- [ ] ⭐ 送信後に「質問を送りました（匿名）」と小さく表示

### 変更する場所
`public/js/participant.js`, `resources/views/participant.blade.php`

### 完了条件
- 2つのブラウザで開き、片方で投稿すると、もう片方に数秒で出る

---

## #11 参加者画面: 絵文字リアクションと「聞きたい／共感」
担当: B / 目安 2時間 / ラベル: `担当:B` `MVP`（絵文字は `おまけ`）

### やること
- [ ] `current_id` があれば「いま回答中」の箱に質問を出す。なければ「講演者が質問に答え始めると…」と出す
- [ ] 👍 私も聞きたい / ❤️ 共感 ボタン → `POST /api/posts/{id}/reactions`（`like` / `heart`）。押したらボタンの見た目を変える
- [ ] ⭐ カードの絵文字チップ（数つき）。押すとトグル。自分が押したものは色が変わる
- [ ] ⭐ ＋ボタンで8つの絵文字を選ぶ小さい箱を開く

### 完了条件
- 講演者が「回答する」を押すと数秒で箱に出て、👍❤️ を押すと数が増える

---

## #12 スクリーン: 新しい投稿を流す・回答中バナー
担当: C / 目安 2.5時間 / ラベル: `担当:C` `MVP`

### やること
- [ ] 2秒ごとに `GET /api/state`
- [ ] 前回より大きい id の投稿を、右から左へ流す（README のお手本 `flow()` を使う）
  - 質問: 青い丸い帯＋「Q」/ 感想: 白い太字
  - 行（高さ）が重ならないように少しずらす
  - 最初に開いたときは、今ある投稿を全部流さない（最後の数件だけ、などでOK）
- [ ] `current_id` があれば上のバナーに質問文と 👍・❤️ の数を出す
- [ ] ⭐ 👍❤️ の数が増えたら、右下からアイコンがふわっと上がる（`animate()` で）

### 完了条件
- 参加者画面で投稿すると数秒以内にスクリーンに流れる

---

## #13 講演者画面: 質問を並べる・「回答する」ボタン
担当: C / 目安 2時間 / ラベル: `担当:C` `MVP`

### やること
- [ ] 2秒ごとに `GET /api/state`
- [ ] 質問を「回答中」→「まだ（リアクション合計の多い順）」→「回答済み」に分けて表示
- [ ] 「回答する」→ `POST /api/posts/{id}/answer`、「回答済みにする」→ `/done`
- [ ] 右に感想を新しい順で
- [ ] 上にいまの質問の 👍・❤️ の数

### 完了条件
- 「回答する」を押すと、参加者画面とスクリーンに数秒で「いま回答中」が出る

---

## #14 通し確認とバグ直し
担当: 全員 / 目安 1時間 / ラベル: `MVP`

### やること
- [ ] PC 1台でスクリーンと講演者、スマホ2台以上で参加者を開き、本番のつもりで10分動かす
- [ ] README「10. 提出前チェックリスト」を全部試す
- [ ] 見つけたバグは、その場で `バグ` ラベルの Issue を立ててから直す

---

## #15 技術の説明資料をまとめて提出する
担当: 全員 / 目安 30分 / ラベル: `MVP`

### やること
- [ ] README「2. 使う道具」の **範囲外の技術と理由** を最終確認
- [ ] README「起動のしかた」が正しいか、新しく clone して確認
- [ ] 提出物（リポジトリURL・説明など）を課題の指示どおりに提出
