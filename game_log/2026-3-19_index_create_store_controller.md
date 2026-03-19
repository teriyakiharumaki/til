# 一覧、作成、保存画面実装の際のコントローラーについて

## ✅ 行ったこと

- 一覧、作成、保存画面実装の際のコントローラーについてまとめた

## コントローラーを作成するコマンド

```
php artisan make:controller GameController
```
これで、app/Http/Controllers/GameController.phpが生成される

実行後
```
root@020990d08807:/var/www/game_log# php artisan make:controller GameController

Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0

   INFO  Controller [app/Http/Controllers/GameController.php] created successfully.  
```
作成完了

## コントローラーの中身

```
<?php

namespace App\Http\Controllers;

use App\Models\Game;
use Illuminate\Http\Request;

class GameController extends Controller
{
    public function index()
    {
        $games = Game::orderByDesc('id')->get();
        return view('games.index', compact('games'));
    }

    public function create()
    {
        return view('games.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => ['required', 'string', 'max:255'],
            'platform' => ['nullable', 'string', 'max:255'],
            'status' => ['required', 'in:unplayed,playing,cleared'],
            'rating' => ['nullable', 'integer', 'min:1', 'max:5'],
            'review' => ['nullable', 'string'],
            'price' => ['nullable', 'integer', 'min:0'],
            'condition' => ['nullable', 'in:new,used'],
            'play_time_minutes' => ['nullable', 'integer', 'min:0'],
            'purchased_at' => ['nullable', 'date'],
            'cleared_at' => ['nullable', 'date'],
        ]);

        Game::create($validated);

        return redirect()->route('games.index');
    }
}
```

### use部分
```
use App\Models\Game;
use Illuminate\Http\Request;
```

- **Game**

  DB（gamesテーブル）とやり取りするため

- **Request**

  フォームの入力データを受け取るため

***

### `class GameController extends Controller`

**Controllerクラスを継承している**

これにより、以下が可能になる
- バリデーション
- リダイレクト
- view表示

***

### index()（一覧表示）
```
public function index()
{
    $games = Game::orderByDesc('id')->get();
    return view('games.index', compact('games'));
}
```

#### ①データ取得
```
$games = Game::orderByDesc('id')->get();
```
意味は、
- idの降順（新しい順）
- 全件取得

**`疑問?：$gamesの$ってなんだ`**

`$` は 「これは変数」っていうマーク

PHPのルールとして`$変数名`と書くのがルール

**`疑問?：compact()ってなんだ`**

Controllerの変数は、そのままだとViewでは使えないので、「$games を、Viewでも $games という名前で使えるように渡す」

compactの中身
```
['games' => $games]
```
「games」という名前の“変数”をViewに作っている
```
Controller
  $games = データ

↓ 渡す

View
  $games ← 新しく用意される
```

- 'games' → Viewでの変数名になる
- $games → その中身

**`疑問?：gamesがいっぱいで紛らわしい`**

別例で理解
```
return view('test', [
  'abc' => $games
]);
```
この場合、Viewでは
```
{{ $abc }} ← これで使う
{{ $games }} ← これは使えない
```

| Controller | View |
| ---- | ---- |
| $games | 渡されない |
| 'games' => $games | $gamesとして使える |
| 'abc' => $games | $abcとして使える |

**Viewに渡すときに「変数名が決まる」**

**`疑問?：Game::orderByDesc('id')->get();の、->get();ってなんだ`**

「クエリを実行して、データを取得する」という意味

```
Game::orderByDesc('id')
```
これは、まだ実行していない

「こういうSQL作りたい」という設計図

```
->get();
```
これをつけることで、初めてDBにアクセスして実際にデータを取りに行く

get() の戻り値は、コレクション（配列みたいなもの）

***

#### ②viewに渡す
```
return view('games.index', compact('games'));
```
games.index に $games を渡している

compact('games') は、
```
['games' => $games]
```
と同じ

***

### create()（フォーム表示）
```
public function create()
{
    return view('games.create');
}
```
ただ画面を返しているだけ

***

### store()（保存処理）
```
public function store(Request $request)
```

#### ①バリデーション
```
$validated = $request->validate([
```
以下をチェックしている
| 項目 | 意味 |
| ---- | ---- |
| required | 必須 |
| string | 文字列 |
| max:255 | 最大255文字 |
| nullable | 空白OK |
| integer | 数字 |
| min / max | 範囲 |
| in | 選択肢制限 |

#### ②保存
```
Game::create($validated);
```
DBにINSERTしている、中では、
```
INSERT INTO games ...
```
が実行されている

これが実行されるのは $fillable があるから

#### ③リダイレクト
```
return redirect()->route('games.index');
```
保存後に一覧画面に戻る

***

## 全体の流れ
```
① /games/create にアクセス
↓
create() → フォーム表示

② フォーム送信（POST /games）
↓
store() 実行
↓
バリデーション
↓
DB保存
↓
一覧へリダイレクト

③ /games
↓
index() → 一覧表示
```

## 参考サイト
- コントローラー作成について（https://readouble.com/laravel/10.x/ja/controllers.html）<br>
