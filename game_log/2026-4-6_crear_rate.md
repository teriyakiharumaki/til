# 一覧にクリア率を表示

## ✅ 行ったこと

- 一覧にクリア率を表示させた
- クリア率の割合を、バーで表示できるようにした
- クリアしたゲームに背景が変わるようにした

## ①コントローラーでクリア率の計算
game_log/app/Http/Controllers/GameController.php
```
$total = Game::count();
$cleared = Game::where('status','cleared')->count();

$clearRate = $total ? round($cleared / $total * 100) : 0;

return view('games.index', compact(
    'games',
    'totalMinutes',
    'statusCounts',
    'clearRate'
));
```

### `$total = Game::count();`

総ゲーム数を数えている（games テーブルの件数を数えている）

### `$cleared = Game::where('status','cleared')->count();`

クリア済みの数を数えている（status = cleared のものだけ数える）

上と同じcountについて（https://gyas.co.jp/knowledge_blog/laravel-eloquent-count/）

### `$clearRate = $total ? round($cleared / $total * 100) : 0;`

クリア率の計算をしている

- `$cleared / $total * 100`

  この部分で、割合をパーセントに変換

- `round(...)`

  小数を四捨五入

  (https://www.sejuku.net/blog/48961#index_id0)

- `$total ? A : B`

  totalが0じゃなければA、0ならB<br>
  (ゲームが0件だと、0/0となりエラーになる)

  これは**三項演算子**<br>
  条件式を一文で書ける

  (https://qiita.com/sho91/items/c88c3418f777385f962c)
  (https://qiita.com/shonansurvivors/items/1e3194cf3eb2ea089039)

## ②一覧に表示
game_log/resources/views/games/index.blade.php
```
<div style="margin-bottom:10px;">
  🏆 クリア率：{{ $clearRate }}%
</div>
```

## ③クリア率の進捗バーを実装
game_log/resources/views/games/index.blade.php
```
<div style="margin-bottom:15px;">
  <div style="margin-bottom:6px;">
    🏆 クリア率：{{ $clearRate }}%
  </div>

  <div style="width: 320px; max-width: 100%; background: #e5e7eb; border-radius: 9999px; overflow: hidden;">
    <div style="width: {{ $clearRate }}%; background: #22c55e; padding: 6px 0;"></div>
  </div>
</div>
```

### 進捗バーの土台
```
<div style="width: 320px; max-width: 100%; background: #e5e7eb; border-radius: 9999px; overflow: hidden;">
```

- `width: 320px;`
  
  バー全体の横幅を 320px

- `max-width: 100%;`

  親要素より大きくなりすぎないようにする

  (スマホ画面みたいに横幅が狭いときでも、はみ出さないようにするため) 

  **普段は 320px、でも画面が小さければ、その画面幅に収まるように縮む**

- `background: #e5e7eb;`

  バーの土台の背景色

- `border-radius: 9999px;`

  角をめちゃくちゃ丸くしている

  **9999px？**：十分大きい数を指定すると、左右が完全に丸まってカプセル型になる。

  **普通の四角いバーではなく、丸っこい進捗バーにするため**

- `overflow: hidden;`

  内側の緑バーが外にはみ出したときに、外枠からはみ出さないようにする

  これがないと、中の緑バーの角が、外側の丸みと合わずにはみ出して見えることがある

  **外枠の丸みに合わせて、中身もきれいに切り取る**

### バーの中身（進捗部分）

```
<div style="width: {{ $clearRate }}%; background: #22c55e; padding: 6px 0;"></div>
```

**実際に緑で埋まる部分**

- `width: {{ $clearRate }}%;`

  **`$clearRate` の値に応じて、バーの長さを変えている**

  ```
  $clearRate = 25 → width: 25%
  $clearRate = 50 → width: 50%
  $clearRate = 100 → width: 100%
  ```

- `background: #22c55e;`¥

  **進捗部分の色を緑**

  灰色 → 未達成部分<br>
  緑色 → クリア済み部分

## クリア率に応じた称号の表示
game_log/resources/views/games/index.blade.php

```
@php
  if ($clearRate < 20) {
    $title = '積みゲーマー';
  } elseif ($clearRate < 60) {
    $title = '冒険中ゲーマー';
  } elseif ($clearRate < 90) {
    $title = 'クリア職人';
  } else {
    $title = 'コンプリート勢';
  }
@endphp

<div style="margin-top:8px; font-weight:bold; color:#f59e0b;">
  称号：{{ $title }}
</div>
```

### `@php部分`
```
@php
  if ($clearRate < 20) {
    $title = '積みゲーマー';
  } elseif ($clearRate < 60) {
    $title = '冒険中ゲーマー';
  } elseif ($clearRate < 90) {
    $title = 'クリア職人';
  } else {
    $title = 'コンプリート勢';
  }
@endphp
```

if文で条件分けをして、各条件ごとに変数`$title`に文字を代入

### 称号の表示部分
```
<div style="margin-top:8px; font-weight:bold; color:#f59e0b;">
  称号：{{ $title }}
</div>
```

`{{}}`内で、称号が代入された変数`$title`の内容を表示

## クリアしたゲームに背景色を追加
game_log/resources/views/games/index.blade.php

```
 <ul>
    @foreach ($games as $game)
      <li>
        <div style="
            padding:10px;
            margin-bottom:10px;
            border:1px solid #ddd;
            background: {{ $game->status === 'cleared' ? '#ecfdf5' : '#ffffff' }};
        ">
          <a href="{{ route('games.show', $game) }}">
            {{ $game->title }}
          </a>
        </div>
```

### 背景色をつけている部分

```
background: {{ $game->status === 'cleared' ? '#ecfdf5' : '#ffffff' }};
```

- `$game->status === 'cleared'`

  `===` は 完全一致を確認（型も値も一致するかを見る）

- `'cleared' ? '#ecfdf5' : '#ffffff' }`

  `条件 ? A : B`の形で三項演算子の形（条件がtrueならA、falseならB）
