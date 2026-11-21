# AI にコードを書いてもらうときの頼み方

3人とも、この手順と見本で AI に頼みます。だれの AI でも同じ前提で書くので、パズルのピースがずれにくくなります。
**Claude Code** を使う場合は、リポジトリの `CLAUDE.md` を自動で読むので、下の「毎回渡すもの」の 1〜3 は省略できます。

---

## 1タスクの進め方（毎回これをくり返す）

```
① 終わりの条件を決める      例：送信すると一覧の一番下に出る
② AI に頼む                 下の「頼み方の型」に、タスクと今のファイルを入れて送る
③ ファイル全体で受け取る    どこに貼るか迷わないように
④ 目でチェック              id・名前が設計図とちがわないか / 知らない書き方がないか / innerHTML に本文を入れていないか
⑤ 貼って動かす              ブラウザで①ができるか / 開発者ツールの Console に赤いエラーがないか
⑥ 動いたら commit           動かなければ、エラー文をそのまま AI に貼って「直して」と頼む
⑦ 3行の説明をもらう         PR の「やったこと」に貼る（発表の準備にもなる）
```

**1タスク＝目で見てわかる動き1つ** にします（「参加者画面を全部作って」は大きすぎます）。

---

## 頼み方の型（コピーして使う）

```
MU TALK（講演会で匿名の質問・感想を集めるアプリ）の【◯◯画面】を作っています。

【タスク】
（1つだけ書く）

【終わりの条件】
（ブラウザで何が見えたら完成か）

【守ること】
- CSS は書かない（<style> も .css も禁止）。見た目は Tailwind の class だけ
- 投稿の本文は textContent で出す（innerHTML に入れない）
- HTML の id / data-* / aria-* は変えない
- API の JSON の形・名前は下の設計図のとおり。変えない
- fetch は api.js の getState() / post() / startPolling() を使う
- 初心者が読むので、日本語のコメントを短く入れる

【設計図】
（README の「8. 設計図」の API の表と JSON を貼る）

【今のファイル】
（自分の Blade と JS を、それぞれ全部貼る。まだなければ「まだ空です」）

【見た目の手本】
（プロトタイプの画面のスクリーンショットを貼る）

ファイル全体で返してください。
最後に、何を変えたかを3行で、確認のしかたを箇条書きで書いてください。
```

---

## タスクの順番（役割ごと）

### スクリーン＋裏側担当

| # | タスク | 終わりの条件 | 手順書 |
| --- | --- | --- | --- |
| 1 | 3人で環境を作る（操作する役） | 3人のパソコンで `localhost:8080` が開く | `docs/laravel-setup.md` 0〜8 |
| 2 | 表とサンプルデータ | `migrate:fresh --seed` で投稿5件・反応61件 | `docs/laravel-setup.md` 9〜11 |
| 3 | `GET /api/state` | `/api/state` の項目名が README と全部一致 | `docs/api-guide.md` 1 |
| 4 | `public/js/api.js`（みんなが使う fetch の部品。下の見本） | どの画面でも Console で `await getState()` を打つと JSON が出る。マージしたらチャットで2人に知らせる | このページ下の見本 |
| 5 | `POST /api/posts` | Console のテストで 201、空なら 422 | `docs/api-guide.md` 2・5 |
| 6 | `answer` / `done` | `current_id` が変わる | `docs/api-guide.md` 3・5 |
| 7 | `reactions` | 2回押すと戻る。シークレットウィンドウで別の人になる | `docs/api-guide.md` 4・5 |
| 8 | スクリーンの見た目 | 紺の背景・回答中のバナー・説明の札 | Issue #5 |
| 9 | 新しい投稿を流す | 参加者で投稿すると数秒で流れる。質問は半透明の青い帯 | Issue #12 |
| 10 | ⭐ 👍❤️ がふわっと上がる | 数が増えると右下から上がる | Issue #12 |

API を作った人が「注文のしかた」（api.js）も書くので、URL や送るものの形がずれません。他の2人が画面の JS を書き始める前（3時間目まで）にマージします。

API（3・5〜7）は `docs/api-guide.md` にコードがあります。AI には「このコードを今のファイルに入れて、意味を説明して」と頼みます。

### 参加者担当

| # | タスク | 終わりの条件 | Issue |
| --- | --- | --- | --- |
| 1 | モックを Blade に移す | `localhost:8080/` でモックと同じ見た目 | #4 |
| 2 | 一覧を `/api/state` から描く（古い順） | サンプル5件が出る。本文は textContent | #10 |
| 3 | 3秒ごとに更新・位置を守る | 別のブラウザで投稿すると一番下に増える。読んでいる位置が動かない | #10 |
| 4 | 記入欄を出す・閉じる | 「質問する」で記入欄が出る。もう一度押すと閉じる | #10 |
| 5 | 投稿を送る | 送信すると一番下に出る。空なら押せない。Enter で送信、変換中は送らない | #10 |
| 6 | いま回答中の箱と 👍❤️ | 講演者が「回答する」と数秒で出る。押すと数が増え、もう一度で戻る | #11 |
| 7 | タブ（一覧・質問・感想） | タブで絞り込める。件数が出る | #10 |
| 8 | ⭐ 絵文字チップと＋ボタン | 押すとトグル。＋で8つから選べる | #11 |
| 9 | ⭐ 「ここから未読」の線とタブの「未読N」 | 新着の上に線。全部見たらバッジが消える | #10 |

### 講演者＋まとめ役

| # | タスク | 終わりの条件 | Issue |
| --- | --- | --- | --- |
| 1 | `layouts/app.blade.php` に `api.js` の読み込みを入れる（api.js 本体はスクリーン＋裏側担当） | 3画面すべてで、api.js のマージ後に `getState` が使える | #2 |
| 2 | 講演者の見た目（タブ・小さい行） | プロトタイプの講演者画面に近い | #6 |
| 3 | 質問を届いた順・#番号で並べる | サンプルが #1〜#3 で出る。回答中は一番上 | #13 |
| 4 | 「回答する」「回答済みにする」 | 押すと参加者とスクリーンに数秒で出る | #13 |
| 5 | 感想タブ・⭐「+N 新着」 | 質問タブを見ている間に増えた感想の数が出る | #13 |
| 6 | 通し確認・Zoom 確認 | README「10. 提出前チェックリスト」が全部OK | #14 |
| 7 | README の仕上げ・提出 | 新しく clone して起動できる | #15 |

---

## `public/js/api.js` の見本（スクリーン＋裏側担当のタスク4）

3画面が共通で使う部品です。作ったあとは、変える前にチャットで相談します。

```js
// みんなが使う fetch の部品（3画面共通）
const csrfToken = document.querySelector('meta[name="csrf-token"]').content;

// いまの状態（投稿全部・回答中の質問）をもらう
async function getState() {
  const res = await fetch('/api/state', { headers: { 'Accept': 'application/json' } });
  return res.json();
}

// 書きこむ（投稿・リアクション・回答する など）。返事（Response）をそのまま返す
async function post(url, data = {}) {
  return fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'Accept': 'application/json', 'X-CSRF-TOKEN': csrfToken },
    body: JSON.stringify(data),
  });
}

// ms ミリ秒ごとに getState して render(state) を呼ぶ。最初の1回はすぐ呼ぶ
// タブが裏にあるとき（document.hidden）は取りに行かない（100人で使っても重くならないように）
function startPolling(ms, render) {
  const tick = async () => {
    if (document.hidden) return;
    try { render(await getState()); } catch (e) { console.error(e); }
  };
  tick();
  setInterval(tick, ms);
}
```

各画面の JS での使い方:

```js
startPolling(3000, render);                 // 参加者（スクリーン・講演者は 2000）

const res = await post('/api/posts', { type: 'question', body: '…' });
if (!res.ok) { render(await getState()); }  // うまくいかなかったら取り直す
```

`layouts/app.blade.php` では、各画面の JS より **先に** 読み込みます。

```blade
<script src="{{ asset('js/api.js') }}"></script>
@stack('scripts')
```

---

## エラーが出たときの頼み方

```
次のエラーが出ました。原因と直し方を教えてください。直したファイルは全体で返してください。

【やったこと】（何を押したら / どの URL を開いたら）
【エラー】（Console の赤い文字、または画面の文字をそのまま貼る）
【今のファイル】（関係するファイルを全部貼る）
```

API のエラー（500）のときは、`storage/logs/laravel.log` の一番下の数十行も貼ります。
