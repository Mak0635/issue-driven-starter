# API の作り方（コントローラーの中身）

Issue **#7〜#9** の手順書です。[Laravel 初期設定マニュアル](laravel-setup.md) の手順 11 まで（表とサンプルデータができた状態）が終わってから進めます。
返す JSON の形は README「8. 設計図」が **約束** です。名前や形を変えるときは、先に README を直してチームに伝えます。

---

## 0. API のしくみ（全体の流れ）

```
① 画面の JS            fetch('/api/posts', { method: 'POST', body: … })
        │
② routes/web.php       「POST /api/posts が来たら PostController の store へ」
        │
③ コントローラー        受け取る → チェック（バリデーション）→ モデルで保存・読み出し → JSON を返す
        │
④ モデル（Post など）   データベースの表を読み書きする係
        │
⑤ データベース          posts 表 / reactions 表
```

3 つの画面は **おたがいに直接やり取りしません**。参加者が投稿すると ③ でデータベースに保存され、スクリーンと講演者は `GET /api/state` を数秒ごとに読みに行くので、そこに新しい投稿が入っている、というしくみです。

| URL | メソッド | 作る順 | Issue |
| --- | --- | --- | --- |
| `GET /api/state` | `PostController@state` | **1（最優先。3画面すべてがこれを待っている）** | #7 |
| `POST /api/posts` | `PostController@store` | 2 | #7 |
| `POST /api/posts/{post}/answer` | `PostController@answer` | 3 | #9 |
| `POST /api/posts/{post}/done` | `PostController@done` | 3 | #9 |
| `POST /api/posts/{post}/reactions` | `ReactionController@toggle` | 4 | #8 |

ルートはマニュアルの手順 5 で、もう `routes/web.php` に書いてあります。ここではコントローラーの中身だけを書きます。

---

## 1. `GET /api/state`（いまの状態を全部返す）

### やること
1. 「だれが見ているか」をセッション ID で知る（名前は使わない＝匿名）
2. 投稿を古い順に全部読む
3. リアクションの数を **1 回の SQL でまとめて** 数える
4. 「自分が押したもの」を読む
5. README の形に組み立てて JSON で返す

### コード（`app/Http/Controllers/PostController.php` の `state`）

ファイルの上の `use` に 2 行足します。

```php
use App\Models\Post;
use App\Models\Reaction;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
```

```php
// GET /api/state
public function state(Request $request)
{
    // 1. 見ている人のしるし（ブラウザごとにちがう。名前は保存しない）
    $visitorId = $request->session()->getId();

    // 2. 投稿を古い順に全部
    $posts = Post::orderBy('id')->get();

    // 3. リアクションの数：「どの投稿の・どの種類が・何個」を 1 回の SQL で数える
    //    （投稿ごとに数えると、投稿が 100 件なら SQL も 100 回になって重い＝N+1 問題）
    $counts = Reaction::select('post_id', 'kind', DB::raw('COUNT(*) as total'))
        ->groupBy('post_id', 'kind')
        ->get()
        ->groupBy('post_id');

    // 4. 自分が押したもの（これも 1 回の SQL）
    $mine = Reaction::where('visitor_id', $visitorId)
        ->get(['post_id', 'kind'])
        ->groupBy('post_id');

    // 5. README の形に組み立てる
    $list = $posts->map(function (Post $post) use ($counts, $mine) {
        $kinds   = $counts->get($post->id, collect())->pluck('total', 'kind'); // ['🤔' => 3, 'like' => 18, …]
        $myKinds = $mine->get($post->id, collect())->pluck('kind');           // ['👀', 'like', …]

        return [
            'id'           => $post->id,
            'type'         => $post->type,
            'body'         => $post->body,
            'status'       => $post->status,
            'reactions'    => (object) $kinds->except(['like', 'heart'])->map(fn ($n) => (int) $n)->all(),
            'my_reactions' => $myKinds->reject(fn ($k) => in_array($k, ['like', 'heart']))->values(),
            'like'         => (int) $kinds->get('like', 0),
            'heart'        => (int) $kinds->get('heart', 0),
            'my_like'      => $myKinds->contains('like'),
            'my_heart'     => $myKinds->contains('heart'),
            'created_at'   => $post->created_at,
        ];
    });

    return response()->json([
        'current_id' => Post::where('status', 'answering')->value('id'), // なければ null
        'posts'      => $list,
    ]);
}
```

| 書き方 | 意味 |
| --- | --- |
| `$request->session()->getId()` | ブラウザごとのセッション ID。同じブラウザなら同じ値なので「同じ人」とわかる |
| `groupBy('post_id', 'kind')` + `COUNT(*)` | SQL で「投稿×種類」ごとに数える |
| `->get()->groupBy('post_id')` | 取ってきた結果を、PHP で投稿ごとにまとめ直す |
| `pluck('total', 'kind')` | `['🤔' => 3, '👀' => 4]` の形にする |
| `except(['like', 'heart'])` | 絵文字だけ残す（👍❤️ は別の項目にする約束） |
| `(object)` | 空のときに `[]` ではなく `{}` で返すため（README の形に合わせる） |
| `->value('id')` | 1 件目の id だけを取る。見つからなければ `null` |

### 確認
ブラウザで http://localhost:8080/api/state を開きます。
- サンプルの 5 件が古い順に出ている
- 1 件目が `"status": "answering"`、`"current_id": 1`、`"like": 18`、`"reactions": {"🤔": 3, "👀": 4}`

README の JSON と **項目の名前を 1 つずつ見比べます**。ここがずれると 3 画面すべてが動きません。

---

## 2. `POST /api/posts`（投稿を保存）

```php
// POST /api/posts
public function store(Request $request)
{
    // チェック：おかしい値なら、Laravel が自動で 422 エラーとエラー内容を返す
    $data = $request->validate([
        'type' => 'required|in:question,comment',
        'body' => 'required|string|max:120',
    ]);

    $post = Post::create($data); // status はマイグレーションの default('open') が入る

    return response()->json($post, 201); // 201 =「作りました」
}
```

| チェック | 意味 |
| --- | --- |
| `required` | 空はダメ |
| `in:question,comment` | この 2 つ以外はダメ |
| `max:120` | 120 文字まで（画面の `maxlength="120"` と同じ。画面のチェックはごまかせるので、サーバーでも必ずチェックする） |

---

## 3. `POST /api/posts/{post}/answer` と `/done`（講演者のボタン）

```php
// POST /api/posts/{post}/answer 「回答する」
public function answer(Post $post)
{
    abort_if($post->type !== 'question', 422, '質問だけ回答中にできます');

    // 2 つの書きかえを「全部成功」か「全部なし」にする（途中で止まって回答中が 0 件や 2 件にならないように）
    DB::transaction(function () use ($post) {
        Post::where('status', 'answering')->update(['status' => 'answered']); // 前の質問を回答済みに
        $post->update(['status' => 'answering']);                            // この質問を回答中に
    });

    return response()->json(['current_id' => $post->id]);
}

// POST /api/posts/{post}/done 「回答済みにする」
public function done(Post $post)
{
    $post->update(['status' => 'answered']);

    return response()->json(['current_id' => null]);
}
```

- `Post $post` と書くと、URL の `{post}` の番号の投稿を Laravel が探して渡してくれます（ルートモデルバインディング）。**ない番号なら自動で 404** を返すので、自分でチェックしなくて大丈夫です。
- `DB::transaction` は「まとめて 1 つの作業」にする書き方です。

---

## 4. `POST /api/posts/{post}/reactions`（絵文字・👍・❤️ を付ける／外す）

`app/Http/Controllers/ReactionController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Database\QueryException;
use Illuminate\Http\Request;

class ReactionController extends Controller
{
    // POST /api/posts/{post}/reactions
    public function toggle(Request $request, Post $post)
    {
        $data = $request->validate([
            'kind' => 'required|in:👀,💡,🙌,🤔,💯,😂,👏,🔥,like,heart',
        ]);

        $visitorId = $request->session()->getId();

        $existing = $post->reactions()
            ->where('kind', $data['kind'])
            ->where('visitor_id', $visitorId)
            ->first();

        if ($existing) {
            $existing->delete();                 // もう押してある → 取り消し
            $on = false;
        } else {
            try {
                $post->reactions()->create([     // まだ → 追加
                    'kind'       => $data['kind'],
                    'visitor_id' => $visitorId,
                ]);
            } catch (QueryException $e) {
                // ほぼ同時に 2 回押すと、2 回目はユニーク制約で失敗する → すでに付いているので OK とする
            }
            $on = true;
        }

        return response()->json([
            'kind'  => $data['kind'],
            'on'    => $on,
            'count' => $post->reactions()->where('kind', $data['kind'])->count(),
        ]);
    }
}
```

- 「同じ人・同じ投稿・同じ種類」は、表の `unique` で 1 つしか作れないようにしてあります（マイグレーション）。
- 100 人いると連打は必ず起きるので、`try / catch` で 500 エラーにならないようにしています。

---

## 5. POST をブラウザで試す方法

POST は URL を開くだけでは試せません（CSRF トークンが必要）。次の手順で、開発者ツールの Console から試します。

1. http://localhost:8080/ を開く（ページの `<meta name="csrf-token">` にトークンが入る）
2. 開発者ツール → **Console** に貼って Enter

```js
const token = document.querySelector('meta[name="csrf-token"]').content;
const send = (url, data = {}) => fetch(url, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'Accept': 'application/json', 'X-CSRF-TOKEN': token },
  body: JSON.stringify(data),
}).then(async r => console.log(r.status, await r.json()));

send('/api/posts', { type: 'question', body: 'テストの質問' }); // 201
send('/api/posts', { type: 'question', body: '' });             // 422（空はダメ）
send('/api/posts/2/reactions', { kind: '👀' });                  // on: true
send('/api/posts/2/reactions', { kind: '👀' });                  // on: false（取り消し）
send('/api/posts/2/answer');                                     // current_id: 2
send('/api/posts/999/answer');                                   // 404
```

3. もう一度 http://localhost:8080/api/state を開いて、結果が反映されているか見る
4. **シークレットウィンドウ** で同じことをすると「別の人」になるので、👀 の数が 2 になる

| 出た数字 | 意味 | よくある原因 |
| --- | --- | --- |
| 200 / 201 | 成功 | |
| 404 | 見つからない | ない番号の投稿、URL の打ち間違い |
| 419 | CSRF トークンがない・古い | `X-CSRF-TOKEN` を付けていない、ページを開き直していない |
| 422 | チェックで弾かれた | 空の本文、`type` や `kind` の値がちがう |
| 500 | サーバーのエラー | `storage/logs/laravel.log` の一番下を見る |

---

## 6. 終わったら

- [ ] `/api/state` の項目名が README と 1 つずつ一致している
- [ ] 5 の Console のテストが全部、表のとおりの数字になる
- [ ] シークレットウィンドウで、別の人として数が増える
- [ ] PR（`closes #7` など）→ マージしたらチャットで「API できたよ！ ニセのデータを fetch に差し替えてね」と伝える
