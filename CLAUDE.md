# MU TALK チーム開発のルール（AI 向け）

講演会で匿名の質問・感想を集めるアプリ。3人チーム・Issue 駆動・残り約10時間。
コードを書く前に **README.md の「7. みんなで守るルール」「8. 設計図」** を読むこと。目指す完成形はプロトタイプ（バージョン10）: https://claude.ai/artifact/LyYXL8LSSM2NpAekLbBFkh

## 使う技術（この範囲で書く）
- HTML / Tailwind（CDN）/ JavaScript / PHP / Laravel / MySQL（Docker）
- 範囲外の技術を使うときは、README「2. 使う道具」の表に理由を書く

## 絶対に守ること
- **CSS を書かない**。`<style>` も `.css` ファイルも作らず、Tailwind の class だけで見た目を作る
- ユーザーが書いた文字は **`textContent`**（Blade なら `{{ }}`）で出す。`innerHTML` に投稿の本文を入れない（XSS 対策）
- **API の JSON の形と名前を変えない**（README「8. 設計図」が約束。`body` を `text` にする、などの変更は禁止）
- HTML の `id` / `data-*` / `aria-*` を勝手に変えない（JS がそれを目印に探している）
- **担当ファイル以外は変更しない**。必要なら変更せずに「◯◯さんに頼むこと」として伝える
- `hidden` 属性をつける要素に `flex` などの display の class をつけない（`hidden` が効かなくなる）

## ファイルの持ち主
| ファイル | 担当 |
| --- | --- |
| `routes/web.php`、`app/`、`database/` | スクリーン＋裏側担当 |
| `resources/views/screen.blade.php`、`public/js/screen.js` | スクリーン＋裏側担当 |
| `resources/views/participant.blade.php`、`public/js/participant.js`、`mockups/participant.html` | 参加者担当 |
| `resources/views/speaker.blade.php`、`public/js/speaker.js` | 講演者＋まとめ役 |
| `resources/views/layouts/app.blade.php`、`public/js/api.js`、`README.md`、`docs/` | 講演者＋まとめ役 |

## 書き方
- fetch は `public/js/api.js` の `getState()` / `post()` / `startPolling()` を使う（各画面で fetch を書き直さない）
- 更新間隔：参加者 3000ms、スクリーン・講演者 2000ms。`document.hidden` のときは取りに行かない
- 描き直すときは `scrollTop` を保存・復元して、読んでいる位置をずらさない
- POST の返事が `!res.ok` なら `getState()` で取り直して描き直す
- 初心者が読むコードなので、何をしているかを日本語のコメントで短く書く
- 手順書: `docs/laravel-setup.md`（環境）、`docs/api-guide.md`（API）、`docs/issues.md`（タスク）

## 終わったら
- 変更したファイルと、確認のしかた（ブラウザで何を見ればいいか）を伝える
- 何を変えたかを3行で説明する（PR の「やったこと」にそのまま使う）
- commit は担当者が動作を確認してから。`.env` と `vendor/` は commit しない
