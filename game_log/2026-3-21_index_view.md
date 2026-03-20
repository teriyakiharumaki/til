# 一覧画面のviewについて

## ✅ 行ったこと

- 一覧画面のviewについてまとめた

## viewを作成
game_log/resources/views/games/index.blade.php
```
@extends('layouts.app')

@section('content')
  <h2>ゲーム一覧</h2>
  <p><a href="{{ route('games.create') }}">＋ 新規登録</a></p>

  @if ($games->isEmpty())
    <p>まだ登録がありません。</p>
  @else
    <ul>
      @foreach ($games as $game)
        <li>
          <strong>{{ $game->title }}</strong>
          @if($game->platform)（{{ $game->platform }}）@endif

          @if($game->rating)
            - {{ str_repeat('★', $game->rating) }}{{ str_repeat('☆', 5 - $game->rating) }}
          @endif

          - {{ ['unplayed'=>'未プレイ','playing'=>'プレイ中','cleared'=>'クリア済み'][$game->status] ?? $game->status }}
        </li>
      @endforeach
    </ul>
  @endif
@endsection
```

### ①`@extends('layouts.app')`
共通レイアウト（app.blade.php）を使う

`layouts/app.blade.php`の中身がはめ込まれる

### ②`@section('content')`
layoutの「content部分」に差し込む

`@yield('content')`にこの中身が入る

### ③タイトル表示
```
<h2>ゲーム一覧</h2>
```
そのまま

### ④新規登録リンク
```
<a href="{{ route('games.create') }}">＋ 新規登録</a>
```

**`{{ }} の意味`**

PHPの値を表示する

**`route('games.create')`**

→ /games/create に変換される

```
<a href="/games/create">
```
と同じ

### ⑤データが空かチェック
```
@if ($games->isEmpty())
```

意味：**`$games が空なら`**

`$games は Collection(配列のようなもの)`

```
<p>まだ登録がありません。</p>
```
データ0件のとき表示

### ⑥データがある場合
```
@else
```

### ⑦ループ処理
```
@foreach ($games as $game)
```

意味：**`$games の中身を1件ずつ取り出して、$game に入れて処理する`**

asの部分
```
$games as $game
```
「$gamesの中身を1つずつ$gameとして使う」

**疑問？：`$game`は配列？**

配列ではなく、「Gameモデルのオブジェクト」(Gameクラスから作られた「オブジェクト」)

配列との違い:

❌ 配列だった場合
`$game['title']`

✅ 今回（オブジェクト）
`$game->title`

中身
```
$game = new Game();
$game->title = "FF7";
$game->platform = "PS5";
$game->rating = 5;
```

なぜ？
```
$games = Game::orderByDesc('id')->get();
```
**これがDBの1行 = Gameオブジェクトに変換**

### ⑧ ゲーム名
```
<strong>{{ $game->title }}</strong>
```

意味：**titleカラムを表示**

### ⑨ platform（条件付き表示）
```
@if($game->platform)
  （{{ $game->platform }}）
@endif
```
意味：**platformがあるときだけ表示**

### ⓾星評価
```
@if($game->rating)
  - {{ str_repeat('★', $game->rating) }}{{ str_repeat('☆', 5 - $game->rating) }}
@endif
```

str_repeatで、繰り返し

参考サイト（https://www.php.net/manual/ja/function.str-repeat.php）

### ⑪ ステータス変換
```
{{ ['unplayed'=>'未プレイ','playing'=>'プレイ中','cleared'=>'クリア済み'][$game->status] ?? $game->status }}
```

**やっていること**
```
[
  'unplayed' => '未プレイ',
  'playing'  => 'プレイ中',
  'cleared'  => 'クリア済み'
]
```
この配列から、
```
[$game->status]
```
で取り出す

```
$game->status = 'playing'
```
の時、「プレイ中」

**?? の意味**
```
?? $game->status
```
もし見つからなかったら、元の値をそのまま表示する

