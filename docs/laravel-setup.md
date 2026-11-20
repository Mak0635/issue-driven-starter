# Laravel 初期設定マニュアル（Docker 版）

Issue **#2**（Laravel プロジェクトを作る）と **#3**（データベースの表を作る）の手順書です。
**A さんが 1 回だけ** やります。B さん・C さんは、最後の「[あとから参加する人の手順](#あとから参加する人の手順b--c)」だけ読めば OK です。

> 📝 このマニュアルのコマンドに出てくる `app` は、**`docker-compose.yml` の `services:` の下に書いてある PHP のコンテナ名** です。`php` や `laravel` などになっていたら、そこを置きかえてください。
> `db` も同じで、データベースのコンテナ名です。

---

## 目次

| # | やること | Issue |
| --- | --- | --- |
| 0 | [準備：Docker の設定ファイルを置く](#0-準備docker-の設定ファイルを置く) | #2 |
| 1 | [コンテナを起動する](#1-コンテナを起動する) | #2 |
| 2 | [Laravel を作る](#2-laravel-を作る) | #2 |
| 3 | [.env（データベースの設定）](#3-envデータベースの設定) | #2 |
| 4 | [コントローラーを作る（2個）](#4-コントローラーを作る2個) | #2 |
| 5 | [routes/web.php を書く](#5-routeswebphp-を書く) | #2 |
| 6 | [画面のファイル（Blade）を作る](#6-画面のファイルbladeを作る) | #2 |
| 7 | [JS ファイルを空で作る](#7-js-ファイルを空で作る) | #2 |
| 8 | [動くか確認 → PR](#8-動くか確認--pr) | #2 |
| 9 | [モデルと表（マイグレーション）を作る](#9-モデルと表マイグレーションを作る) | #3 |
| 10 | [サンプルデータ（シーダー）を作る](#10-サンプルデータシーダーを作る) | #3 |
| 11 | [表を作って確認 → PR](#11-表を作って確認--pr) | #3 |
| ー | [あとから参加する人の手順（B・C）](#あとから参加する人の手順b--c) | ー |
| ー | [こまったとき](#こまったとき) | ー |

---

## 0. 準備：Docker の設定ファイルを置く

このリポジトリには、まだ Docker の設定ファイルがありません。
**授業で使っている Docker 環境のファイル**（`docker-compose.yml`、`docker/` フォルダなど）を、このリポジトリの一番上にコピーします。

```
issue-driven-starter/
├── docker-compose.yml   ← 授業の環境からコピー
├── docker/              ← あればコピー（Dockerfile や nginx の設定）
├── README.md
├── docs/
└── practice/
```

`docker-compose.yml` を開いて、次の 3 つをメモしておきます。

| 見るところ | 例 | 何に使う？ |
| --- | --- | --- |
| PHP のサービス名 | `app` | コマンドの `docker compose exec app ...` |
| DB のサービス名と `MYSQL_DATABASE` / `MYSQL_USER` / `MYSQL_PASSWORD` | `db` / `laravel` / `user` / `password` | 手順 3 の `.env` |
| `volumes:` の左側 | `./src:/var/www/html` | Laravel ができるフォルダ（この例なら `src/`） |

> 💡 `volumes:` が `./:/var/www/html` のように **`./` だけ** なら、リポジトリの一番上に Laravel ができます。`./src` なら `src/` の中にできます。以降の説明の「Laravel のフォルダ」はこの場所のことです。

---

## 1. コンテナを起動する

```bash
docker compose up -d
```

- **やること**: `docker-compose.yml` どおりに PHP・DB・Web サーバーの「部屋」を作って、裏で動かしたままにする（`-d`）。
- **これだけでは Laravel のファイルはできません。** 部屋を用意しただけです。

動いているか確認します。全部 `running`（または `Up`）なら OK です。

```bash
docker compose ps
```

---

## 2. Laravel を作る

```bash
docker compose exec app composer create-project laravel/laravel .
```

| よくある間違い | 正しい書き方 |
| --- | --- |
| `laravel/laraver` | `laravel/laravel` |
| 最後の `.` がない | `.` は「今いるフォルダに作る」という意味。ないと `laravel/` という別フォルダができてしまう |

> ⚠️ **Laravel のフォルダにファイルが入っていると失敗します**（`Project directory is not empty`）。
> 一番上に Laravel を作る場合は README などがあるので、次のどちらかにします。
> - **別の空のフォルダで作ってから、中身をコピーする**（Laravel が作った `README.md` はコピーしない。チームの README が消えるため）
> - `docker-compose.yml` の `volumes:` を `./src:/var/www/html` にして、`src/` に作る（おすすめ。ぶつからない）

できたか確認します。`artisan`・`app/`・`routes/`・`resources/` があれば成功です。

```bash
docker compose exec app ls
```

この時点で `routes/web.php` も自動でできています（中身は `/` でようこそ画面を出す 1 行だけ）。

---

## 3. .env（データベースの設定）

Laravel のフォルダにある `.env` を開いて、`DB_` で始まる行を `docker-compose.yml` に合わせます。

```
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=user
DB_PASSWORD=password
```

- `DB_HOST` は `localhost` ではなく **DB のサービス名**（例: `db`）。いちばんよくある間違いです。
- `DB_CONNECTION=sqlite` になっていて、その下の行が `#` でコメントになっていたら、`#` を外して書きかえます。
- `.env` は **Git に入りません**（パスワードが入るため）。同じ内容を `.env.example` にも書いておくと、B・C さんが楽になります。

アプリの鍵は `create-project` が作ってくれます。`.env` の `APP_KEY=` が空のときだけ、次を実行します。

```bash
docker compose exec app php artisan key:generate
```

設定を変えたら、古い設定の記憶を消しておきます。

```bash
docker compose exec app php artisan config:clear
```

---

## 4. コントローラーを作る（2個）

**`web.php` より先に作ります。** 存在しないコントローラーを `web.php` に書くと、エラーになるためです。

```bash
docker compose exec app php artisan make:controller PostController
```

```bash
docker compose exec app php artisan make:controller ReactionController
```

`app/Http/Controllers/` に 2 つのファイルができます。

| コントローラー | メソッド | やること | 中身を書く Issue |
| --- | --- | --- | --- |
| PostController | `state` | 投稿・反応の数・回答中の質問を JSON で返す | #7 |
| | `store` | 質問・感想を保存する | #7 |
| | `answer` | 質問を「回答中」にする | #9 |
| | `done` | 質問を「回答済み」にする | #9 |
| ReactionController | `toggle` | 絵文字・👍・❤️ を付ける／外す | #8 |

今はメソッドの**形だけ**入れておきます（中身は #7〜#9 で書きます）。

`app/Http/Controllers/PostController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    // GET /api/state（#7）
    public function state(Request $request)
    {
        return response()->json(['todo' => '#7 で作る']);
    }

    // POST /api/posts（#7）
    public function store(Request $request)
    {
        //
    }

    // POST /api/posts/{post}/answer（#9）
    public function answer(Post $post)
    {
        //
    }

    // POST /api/posts/{post}/done（#9）
    public function done(Post $post)
    {
        //
    }
}
```

`app/Http/Controllers/ReactionController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class ReactionController extends Controller
{
    // POST /api/posts/{post}/reactions（#8）
    public function toggle(Request $request, Post $post)
    {
        //
    }
}
```

> `Post $post` を使っているので、手順 9 で `Post` モデルを作るまでは API を呼ぶとエラーになります。**今はそれで正常** です。

---

## 5. routes/web.php を書く

`routes/web.php` の中身を、全部これに置きかえます。

```php
<?php

use App\Http\Controllers\PostController;
use App\Http\Controllers\ReactionController;
use Illuminate\Support\Facades\Route;

// 画面（Blade を返すだけなのでコントローラーはいらない）
Route::view('/', 'participant');       // 参加者画面
Route::view('/screen', 'screen');      // スクリーン画面
Route::view('/speaker', 'speaker');    // 講演者画面

// API（JavaScript から fetch で呼ぶ）
Route::get('/api/state', [PostController::class, 'state']);
Route::post('/api/posts', [PostController::class, 'store']);
Route::post('/api/posts/{post}/answer', [PostController::class, 'answer']);
Route::post('/api/posts/{post}/done', [PostController::class, 'done']);
Route::post('/api/posts/{post}/reactions', [ReactionController::class, 'toggle']);
```

- **API も `web.php` に書きます。** セッション（匿名のしるし）と CSRF 対策をそのまま使うためです。`routes/api.php` は作りません。
- `{post}` と書くと、Laravel がその番号の投稿を探して `Post $post` に入れてくれます（ルートモデルバインディング）。

---

## 6. 画面のファイル（Blade）を作る

`resources/views/` に 4 つのファイルを作ります。

```
resources/views/
├── layouts/
│   └── app.blade.php     ← 3 画面で共通の外側
├── participant.blade.php ← B さんが作る
├── screen.blade.php      ← C さんが作る
└── speaker.blade.php     ← C さんが作る
```

`resources/views/layouts/app.blade.php`

```blade
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>@yield('title', 'MU TALK')</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="@yield('body_class', 'bg-slate-50 text-slate-900')">
    @yield('content')
    @stack('scripts')
</body>
</html>
```

| 行 | 意味 |
| --- | --- |
| `csrf-token` | POST するときに JS が読む合言葉（README「共通の fetch のしかた」で使う） |
| `cdn.tailwindcss.com` | Tailwind を使えるようにする（CSS ファイルは書かない） |
| `@yield('content')` | 各画面の中身が入る場所 |
| `@stack('scripts')` | 各画面の `<script>` が入る場所 |

`resources/views/participant.blade.php`（`screen` と `speaker` も同じ形で、文字と JS のファイル名だけ変える）

```blade
@extends('layouts.app')

@section('title', '参加者 | MU TALK')

@section('content')
    <p class="p-8 text-xl">参加者画面（準備中）</p>
@endsection

@push('scripts')
    <script src="{{ asset('js/participant.js') }}"></script>
@endpush
```

| ファイル | 表示する文字 | 読み込む JS |
| --- | --- | --- |
| `participant.blade.php` | 参加者画面（準備中） | `js/participant.js` |
| `screen.blade.php` | スクリーン画面（準備中） | `js/screen.js` |
| `speaker.blade.php` | 講演者画面（準備中） | `js/speaker.js` |

---

## 7. JS ファイルを空で作る

`public/js/` フォルダを作って、空のファイルを 3 つ置きます。

```bash
docker compose exec app mkdir -p public/js
```

```bash
docker compose exec app touch public/js/participant.js public/js/screen.js public/js/speaker.js
```

B さんは `participant.js`、C さんは `screen.js` と `speaker.js` に書きます（担当ファイルが分かれているので、コンフリクトしにくい）。

---

## 8. 動くか確認 → PR

**① 画面が開くか**（ポートは `docker-compose.yml` の `ports:` に合わせる）

| URL | 出ればOK |
| --- | --- |
| http://localhost:8080/ | 参加者画面（準備中） |
| http://localhost:8080/screen | スクリーン画面（準備中） |
| http://localhost:8080/speaker | 講演者画面（準備中） |

**② ルートが 8 本あるか**

```bash
docker compose exec app php artisan route:list
```

`/`・`screen`・`speaker`・`api/state`・`api/posts`・`.../answer`・`.../done`・`.../reactions` が並んでいれば OK です（Laravel が自動で足すルートも少し出ます）。

**③ README の「起動のしかた」を、実際に打ったコマンドに書き直す**

**④ Git に入れてはいけないものが入っていないか**

```bash
git status
```

`.env` と `vendor/` が **出てこなければ OK**（Laravel の `.gitignore` が外してくれています）。出てきたら commit しないでチャットで相談します。

**⑤ commit → push → PR**（`closes #2`）→ マージしたら、チャットで「Laravel できたよ！pull してね」と伝えます。

---

## 9. モデルと表（マイグレーション）を作る

ここから Issue #3 です。新しいブランチで作業します。

```bash
docker compose exec app php artisan make:model Post -m
```

```bash
docker compose exec app php artisan make:model Reaction -m
```

`-m` をつけると、モデル（`app/Models/`）と表の設計図（`database/migrations/`）が一緒にできます。
**Post を先に作ります。** reactions は posts を参照するので、posts の表が先にないとエラーになるためです。

### マイグレーション

`database/migrations/xxxx_create_posts_table.php` の `up()`

```php
public function up(): void
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('type');                    // question / comment
        $table->string('body', 120);               // 120文字まで
        $table->string('status')->default('open'); // open / answering / answered
        $table->timestamps();
    });
}
```

`database/migrations/xxxx_create_reactions_table.php` の `up()`

```php
public function up(): void
{
    Schema::create('reactions', function (Blueprint $table) {
        $table->id();
        $table->foreignId('post_id')->constrained()->cascadeOnDelete(); // 投稿が消えたら反応も消える
        $table->string('kind');        // 絵文字 / like / heart
        $table->string('visitor_id');  // セッションID（名前は保存しない＝匿名）
        $table->timestamps();

        $table->unique(['post_id', 'kind', 'visitor_id']); // 同じ人が同じ反応を2回できない
    });
}
```

### モデル

`app/Models/Post.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    protected $fillable = ['type', 'body', 'status'];

    // 1つの投稿は、たくさんの反応を持つ
    public function reactions(): HasMany
    {
        return $this->hasMany(Reaction::class);
    }
}
```

`app/Models/Reaction.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Reaction extends Model
{
    protected $fillable = ['post_id', 'kind', 'visitor_id'];

    // 1つの反応は、1つの投稿のもの
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

`$fillable` は「`create()` でまとめて入れてよい列のリスト」です。書かないと保存のときにエラーになります。

---

## 10. サンプルデータ（シーダー）を作る

```bash
docker compose exec app php artisan make:seeder PostSeeder
```

`database/seeders/PostSeeder.php` の `run()`

```php
public function run(): void
{
    $posts = [
        ['type' => 'question', 'body' => '学生のうちにやっておくべきことは何ですか？'],
        ['type' => 'question', 'body' => '失敗したときはどうやって立ち直りましたか？'],
        ['type' => 'question', 'body' => 'エンジニアとして一番大事にしていることは？'],
        ['type' => 'comment',  'body' => 'チームの話がとても参考になりました！'],
        ['type' => 'comment',  'body' => 'スライドが見やすいです'],
    ];

    foreach ($posts as $post) {
        \App\Models\Post::create($post);
    }
}
```

> サンプルの文はプロトタイプの投稿に合わせて自由に変えて OK です。

`database/seeders/DatabaseSeeder.php` の `run()` の中を、これだけにします（最初から入っているユーザー作成は消して OK）。

```php
public function run(): void
{
    $this->call(PostSeeder::class);
}
```

---

## 11. 表を作って確認 → PR

```bash
docker compose exec app php artisan migrate:fresh --seed
```

`migrate:fresh` は **表を全部消して作り直す** コマンドです。開発中は何回やっても大丈夫ですが、本番データがあるときは使いません。

データが入ったか確認します。

```bash
docker compose exec app php artisan tinker --execute="echo App\Models\Post::count();"
```

`5` と出れば成功です。→ commit → push → PR（`closes #3`）。

---

## あとから参加する人の手順（B・C）

A さんの PR がマージされたら、**Laravel を作り直す必要はありません。** 次の手順で自分のパソコンでも動かします。

```bash
git switch main
```

```bash
git pull origin main
```

```bash
docker compose up -d
```

```bash
docker compose exec app composer install
```

```bash
docker compose exec app cp .env.example .env
```

```bash
docker compose exec app php artisan key:generate
```

```bash
docker compose exec app php artisan migrate:fresh --seed
```

| コマンド | なぜ必要？ |
| --- | --- |
| `composer install` | `vendor/`（Laravel 本体の部品）は Git に入っていないので、自分で取ってくる |
| `cp .env.example .env` | `.env` も Git に入っていないので、見本からコピーする |
| `key:generate` | 自分用の鍵を作る |
| `migrate:fresh --seed` | 自分のパソコンの DB に表とサンプルを作る |

http://localhost:8080/ が開けば準備完了です。

---

## こまったとき

| こまったこと | 原因と直し方 |
| --- | --- |
| `Project directory is not empty` | Laravel を作るフォルダにファイルがある → [手順 2](#2-laravel-を作る) の ⚠️ を見る |
| `service "app" is not running` | コンテナが止まっている → `docker compose up -d` |
| `no such service: app` | サービス名がちがう → `docker-compose.yml` の `services:` の名前に置きかえる |
| `SQLSTATE[HY000] [2002] Connection refused` | `.env` の `DB_HOST` が `localhost` / `127.0.0.1` になっている → DB のサービス名（例: `db`）に |
| `Access denied for user` | `.env` の `DB_USERNAME` / `DB_PASSWORD` が `docker-compose.yml` とちがう |
| `.env` を直したのに変わらない | `docker compose exec app php artisan config:clear` |
| `No application encryption key` | `docker compose exec app php artisan key:generate` |
| `Class "App\Http\Controllers\PostController" not found` | コントローラーを作る前に `web.php` に書いた／名前の打ち間違い → [手順 4](#4-コントローラーを作る2個) |
| `View [participant] not found` | Blade のファイル名か置き場所がちがう → `resources/views/participant.blade.php` |
| `/api/state` がエラー | 手順 9 より前なら正常（`Post` モデルがまだない）。中身は #7 で作る |
| `migrate` で `reactions` のところだけ失敗 | posts より先に reactions の表を作ろうとしている → ファイル名の日付が posts の方が古いか確認 |
| `Permission denied`（`storage` や `logs`） | `docker compose exec app chmod -R 777 storage bootstrap/cache` |
| ポート 8080 が使用中 | ほかのアプリを止めるか、`docker-compose.yml` の `ports:` の左側の番号を変える |
| 全部うまくいかない | `docker compose down` → `docker compose up -d` でやり直す（DB のデータは残る。`down -v` にすると DB も消える） |
